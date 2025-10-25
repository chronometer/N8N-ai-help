# Linjateras Powder Coating RFP Analysis - n8n AI Agent Workflow

## Executive Summary

This guide provides a complete n8n workflow using **native AI Agent nodes, OpenRouter integration, and n8n's tool ecosystem** to automate RFP analysis, technical drawing interpretation, quote calculation, and proposal generation for Linjateras Oy.

**Platform**: n8n with AI Agent nodes  
**AI Provider**: OpenRouter (multi-model support) + direct OpenAI  
**Timeline**: 2-3 weeks to production  
**Cost**: €20-150/month  

---

## 1. Company Context

### Linjateras Oy - Powder Coating Specialist
- **Location**: Tampere, Finland
- **History**: 60+ years in powder coating
- **Capacity**: 6,000 m² per day on 200m automated line
- **Service**: POWDER COATING ONLY (not wet painting)

### Equipment Constraints (ABSOLUTE LIMITS)
- **Max Width**: 1,150 mm
- **Max Height**: 2,100 mm
- **Max Length**: 6,600 mm (9,100 mm with special handling)

### Certifications
- ISO 9001, ISO 14001, ISO 45001
- Avainlippu (Finnish Quality Mark)

---

## 2. n8n Workflow Architecture Using AI Agent Nodes

### Complete Workflow Flow

```
[n8n Form Trigger]
    ↓
[Edit Fields: Prepare State]
    ↓
[AI Agent 1: RFP Requirements Extractor]
  Model: Claude 3.5 Sonnet (via OpenRouter)
  Tools: Vision, Code Interpreter
    ↓
[Code: Parse JSON Response]
    ↓
[Switch: Route Based on Document Type]
    ├─ Has Drawings → [AI Agent 2: Technical Drawing Analyzer]
    │                   Model: GPT-4o (via OpenRouter)
    │                   Tools: Vision, Code Interpreter
    │                     ↓
    ├─ Text Only    → [Edit Fields: Use Text Dimensions]
    │                     ↓
    └─ Invalid      → [Send Email: Request Clarification] → END
                          ↓
[Merge: Combine Drawing/Text Data]
    ↓
[AI Agent 3: Dimensional Validator]
  Model: GPT-4o-mini (via OpenRouter)
  Tools: Code Interpreter
    ↓
[IF: Passes Equipment Constraints?]
    ├─ NO  → [AI Agent: Rejection Letter Generator]
    │          ↓
    │       [Send Email: Polite Rejection]
    │          ↓
    │       [Slack: Notify Sales - Rejected]
    │          ↓
    │       [END]
    └─ YES → Continue
              ↓
[AI Agent 4: Surface Area & Complexity Calculator]
  Model: Claude 3.5 Sonnet (via OpenRouter)
  Tools: Code Interpreter, Web Search
    ↓
[AI Agent 5: Pricing Calculator]
  Model: GPT-4o (via OpenRouter)
  Tools: Code Interpreter
    ↓
[Code: Apply Minimum Charge & Business Rules]
    ↓
[AI Agent 6: Professional Quote Generator]
  Model: Claude 3.5 Sonnet (via OpenRouter)
  Tools: None
    ↓
[AI Agent 7: Quote Quality Validator]
  Model: GPT-4o-mini (via OpenRouter)
  Tools: Code Interpreter
    ↓
[Switch: Quote Value Category]
    ├─ High (>€5,000)   → [Wait: Approval Form] → Manual Review
    ├─ Medium (€1-5k)   → Auto-approve
    └─ Low (<€1,000)    → Auto-approve
                            ↓
[Send Email (Gmail): Deliver Quote]
    ↓
[Supabase / Postgres: Save Quote Record]
    ↓
[Slack: Notify Sales Team - New Quote]
    ↓
[Webhook: Update CRM (optional)]
    ↓
END
```

---

## 3. Node-by-Node Implementation

### Node 1: n8n Form Trigger

**Node**: `n8n Form Trigger`

**Form Configuration**:

**Fields**:
1. **Company Name** (Text, Required)
2. **Contact Person** (Text, Required)
3. **Contact Email** (Email, Required)
4. **Phone** (Text, Optional)
5. **Project Description** (Textarea, Required, placeholder: "Describe the parts and coating requirements")
6. **RFP Document** (File Upload, Required, accept: `.pdf,.png,.jpg,.jpeg`, max: 10MB)
7. **Technical Drawings** (File Upload, Optional, Multiple files, accept: `.pdf,.dwg,.png,.jpg`)
8. **Urgency** (Dropdown, Options: "Standard (5 days)", "Fast (3 days)", "Rush (1-2 days)")
9. **Quantity** (Number, Optional, placeholder: "Number of parts")
10. **Special Requirements** (Textarea, Optional)

**Form Settings**:
- Completion Message: "Thank you! Your RFP is being analyzed. You'll receive a quote within 4 hours."
- Redirect URL: `https://linjateras.fi/thank-you`

---

### Node 2: Edit Fields - Prepare Workflow State

**Node**: `Edit Fields (Set)`

**Fields to Set**:
```javascript
rfp_id = LT-{{ $now.format('yyyyMMdd-HHmmss') }}
customer_company = {{ $json["Company Name"] }}
customer_contact = {{ $json["Contact Person"] }}
customer_email = {{ $json["Contact Email"] }}
customer_phone = {{ $json["Phone"] }}
project_description = {{ $json["Project Description"] }}
urgency_level = {{ $json["Urgency"] }}
quantity_input = {{ $json["Quantity"] || 1 }}
special_requirements = {{ $json["Special Requirements"] }}
rfp_document = {{ $json["RFP Document"] }}
technical_drawings = {{ $json["Technical Drawings"] }}
has_technical_drawings = {{ $json["Technical Drawings"] ? true : false }}
workflow_start_time = {{ $now.toISO() }}
```

---

### Node 3: AI Agent - RFP Requirements Extractor

**Node**: `AI Agent`

**Credentials**: OpenRouter API or OpenAI API

**Model**: `anthropic/claude-3.5-sonnet` (via OpenRouter) or `gpt-4o`

**Why Claude 3.5 Sonnet**: Better at following JSON structure instructions, excellent vision capabilities

**System Prompt**:
```
You are an expert RFP analyst for Linjateras Oy, a Finnish powder coating specialist.

CONTEXT:
Linjateras provides POWDER COATING ONLY (not wet painting). They coat steel, aluminum, and galvanized steel products with maximum dimensions: 1,150mm W × 2,100mm H × 6,600mm L.

YOUR TASK:
Analyze the RFP document/images provided and extract all key information needed for quoting.

EXTRACT:
1. Customer details (already provided in form, verify consistency)
2. Part/product description
3. Material type (steel, aluminum, galvanized, other)
4. Coating specifications (type, color RAL codes, finish)
5. Dimensions if mentioned (width, height, length in mm)
6. Quantity of parts
7. Surface preparation needs (sandblasting, rust removal, etc.)
8. Special coating requirements (UV, chemical resistant, food-safe, etc.)
9. Delivery timeline needs

CRITICAL CHECKS:
- If customer requests "wet painting" or "liquid paint" → FLAG as wrong service type
- If dimensions exceed limits → FLAG for dimensional check
- If material is unusual (not steel/aluminum/galvanized) → FLAG for review

OUTPUT:
Return ONLY a valid JSON object (no markdown, no explanation):

{
  "customer_verified": {"company": "", "contact": "", "email": ""},
  "parts_description": "",
  "material_type": "steel|aluminum|galvanized|other",
  "coating_type": "standard|premium|specialty",
  "color_spec": {"type": "RAL|custom", "codes": [], "count": 0},
  "finish": "glossy|semi-gloss|matte|supermatt|textured",
  "special_coatings": [],
  "dimensions_mentioned": {"width_mm": 0, "height_mm": 0, "length_mm": 0, "found": false},
  "quantity": 0,
  "surface_prep_needed": "none|light|sandblasting|heavy",
  "delivery_urgency": "standard|fast|rush",
  "has_technical_drawings": true/false,
  "confidence_score": 0.0,
  "flags": [],
  "missing_info": []
}
```

**User Prompt** (built from form data):
```javascript
{{ $json.project_description }}

Company: {{ $json.customer_company }}
Contact: {{ $json.customer_contact }}
Urgency: {{ $json.urgency_level }}
Quantity: {{ $json.quantity_input }}
Special Requirements: {{ $json.special_requirements }}

Please analyze the attached RFP document and technical drawings (if provided) and extract all requirements.
```

**Tools Enabled**:
- ✅ **Vision** (for analyzing images/PDFs)
- ✅ **Code Interpreter** (for data processing)
- ✅ **File Search** (if multiple documents)

**Input**:
- Attach: `{{ $json.rfp_document }}`
- Attach: `{{ $json.technical_drawings }}`

**Options**:
- Temperature: 0.2 (for consistency)
- Max Tokens: 4000

---

### Node 4: Code - Parse Requirements JSON

**Node**: `Code`

**Code**:
```javascript
// Get AI agent output
const agent_output = $input.first().json.output || $input.first().json.text;

// Parse JSON response
let requirements;
try {
  requirements = JSON.parse(agent_output);
} catch (e) {
  // Try extracting from code blocks
  const json_block = agent_output.match(/```json\s*([\s\S]*?)\s*```/);
  const json_obj = agent_output.match(/\{[\s\S]*\}/);
  
  if (json_block) {
    requirements = JSON.parse(json_block[1]);
  } else if (json_obj) {
    requirements = JSON.parse(json_obj[0]);
  } else {
    throw new Error('Cannot parse agent response as JSON');
  }
}

// Merge with form data
return [{
  ...requirements,
  rfp_id: $node["Edit Fields"].json.rfp_id,
  customer_email_verified: $node["Edit Fields"].json.customer_email,
  workflow_state: 'requirements_extracted'
}];
```

---

### Node 5: Switch - Route Based on Drawing Availability

**Node**: `Switch`

**Mode**: Rules

**Rules**:
1. **Has Drawings** → `{{ $json.has_technical_drawings === true }}` → Output 0
2. **No Drawings** → `{{ $json.has_technical_drawings === false }}` → Output 1
3. **Invalid/Unclear** → `{{ $json.flags.includes('INVALID') }}` → Output 2

---

### Node 6: AI Agent - Technical Drawing Analyzer

**Node**: `AI Agent`

**Model**: `openai/gpt-4o` (via OpenRouter) - best vision capabilities

**System Prompt**:
```
You are a manufacturing engineer analyzing technical drawings for powder coating quotes.

YOUR TASK:
Extract precise dimensional information and calculate paintable surface areas from technical drawings.

ANALYSIS STEPS:

1. VISUAL INSPECTION:
   - Identify drawing type (orthographic, isometric, assembly)
   - Check units (mm, inches, cm) - Convert all to mm
   - Identify scale (1:1, 1:2, etc.)
   - Assess drawing quality (clear, readable, poor)

2. TITLE BLOCK EXTRACTION:
   - Part number
   - Part description/name
   - Material specification
   - Quantity (if shown)
   - Revision number

3. DIMENSION EXTRACTION:
   - Overall length (mm)
   - Overall width (mm)
   - Overall height (mm)
   - Diameter (if cylindrical) with Ø symbol
   - Critical features (holes, cutouts, bends)

4. GEOMETRY IDENTIFICATION:
   - Shape type: Rectangular Box / Cylinder / I-Beam / Tube / Complex Assembly
   - Number of distinct surfaces
   - Accessibility (easy / moderate / difficult to coat)

5. SURFACE AREA CALCULATION:
   Use Code Interpreter to calculate:
   
   RECTANGULAR BOX:
   Surface_Area = 2 × (length×width + length×height + width×height)
   
   CYLINDER:
   Lateral_Surface = 2 × π × radius × height
   Total_Surface = 2 × π × radius × (height + radius)
   
   COMPLEX SHAPES:
   - Decompose into geometric primitives
   - Calculate each surface separately
   - Sum total areas
   - Add complexity factor: 10-20% extra

6. COMPLEXITY ASSESSMENT:
   - Simple flat surfaces: Factor = 1.0
   - Tubes, angles, bends: Factor = 1.15
   - Complex assemblies, hard-to-reach: Factor = 1.30

OUTPUT:
Return ONLY valid JSON:

{
  "drawing_info": {
    "part_number": "",
    "part_name": "",
    "material_spec": "",
    "units": "mm",
    "scale": "",
    "quality": "clear|readable|poor"
  },
  "dimensions": {
    "length_mm": 0,
    "width_mm": 0,
    "height_mm": 0,
    "diameter_mm": 0,
    "geometry_type": "box|cylinder|i-beam|tube|complex"
  },
  "surface_area": {
    "per_part_m2": 0.0,
    "calculation_method": "formula used",
    "calculation_steps": "show work",
    "complexity_factor": 1.0,
    "final_area_per_part_m2": 0.0
  },
  "features": {
    "holes_count": 0,
    "bends_count": 0,
    "welds": false,
    "accessibility": "easy|moderate|difficult"
  },
  "confidence": 0.0,
  "missing_dimensions": [],
  "assumptions": [],
  "requires_clarification": false
}

VALIDATION:
- Confidence < 0.85 → requires human review
- Missing critical dimensions → set requires_clarification = true
- Calculations must use Code Interpreter for accuracy
```

**Tools Enabled**:
- ✅ **Vision** (analyze drawings)
- ✅ **Code Interpreter** (calculate surface areas with formulas)
- ❌ File Search (not needed)

**Input**:
```javascript
Analyze these technical drawings and extract dimensions:

{{ $json.technical_drawings }}

Context from RFP:
- Parts: {{ $json.parts_description }}
- Material: {{ $json.material_type }}
- Quantity: {{ $json.quantity }}
```

**Options**:
- Temperature: 0.1 (precision)
- Max Tokens: 8000
- Response Format: JSON

---

### Node 7: Code - Parse Drawing Analysis

**Node**: `Code`

**Code**:
```javascript
const agent_response = $input.first().json.output;

// Parse JSON
let drawing_data;
try {
  drawing_data = JSON.parse(agent_response);
} catch (e) {
  const match = agent_response.match(/\{[\s\S]*\}/);
  if (match) {
    drawing_data = JSON.parse(match[0]);
  } else {
    // Fallback: Use text dimensions from requirements
    drawing_data = {
      dimensions: $node.ParseRequirements.json.dimensions_mentioned,
      surface_area: { per_part_m2: 0, final_area_per_part_m2: 0 },
      confidence: 0.5,
      requires_clarification: true
    };
  }
}

return [{
  dimensions: drawing_data.dimensions,
  surface_area_per_part: drawing_data.surface_area.final_area_per_part_m2,
  complexity_factor: drawing_data.surface_area.complexity_factor || 1.0,
  confidence: drawing_data.confidence,
  features: drawing_data.features,
  needs_review: drawing_data.requires_clarification || drawing_data.confidence < 0.85
}];
```

---

### Node 8: Merge Drawing Data with Requirements

**Node**: `Merge`

**Mode**: Combine by Position

**Inputs**:
- Input 1: ParseRequirements output
- Input 2: ParseDrawingAnalysis output

---

### Node 9: AI Agent - Dimensional Validator

**Node**: `AI Agent`

**Model**: `openai/gpt-4o-mini` (via OpenRouter) - fast and cheap for validation

**System Prompt**:
```
You are a production constraint validator for Linjateras powder coating equipment.

EQUIPMENT LIMITS (ABSOLUTE, NON-NEGOTIABLE):
- Maximum Width: 1,150 mm
- Maximum Height: 2,100 mm  
- Maximum Length: 6,600 mm (standard) OR 9,100 mm (special handling)

YOUR TASK:
Check if the parts can physically fit through our coating line.

INPUT DATA:
You'll receive dimensions in mm for width, height, and length.

VALIDATION LOGIC:
1. Compare each dimension to limits
2. If ANY dimension exceeds maximum → INFEASIBLE
3. If length is 6,601-9,100 mm → FEASIBLE_WITH_RESERVATION (special handling)
4. If all within limits → FEASIBLE

OUTPUT JSON:
{
  "status": "FEASIBLE|FEASIBLE_WITH_RESERVATION|INFEASIBLE",
  "checks": {
    "width_ok": true/false,
    "height_ok": true/false,
    "length_ok": true/false
  },
  "violations": [],
  "special_notes": "",
  "rejection_message": "" (if infeasible, explain professionally)
}

If INFEASIBLE, generate a professional rejection explaining:
- Which dimension(s) exceeded limits
- Actual vs. maximum dimensions
- Suggestion to redesign or split into smaller components
```

**Tools Enabled**:
- ✅ **Code Interpreter** (for validation calculations)

**Input**:
```javascript
Validate these dimensions against our equipment constraints:

Width: {{ $json.dimensions.width_mm }} mm (limit: 1,150 mm)
Height: {{ $json.dimensions.height_mm }} mm (limit: 2,100 mm)
Length: {{ $json.dimensions.length_mm }} mm (limit: 6,600 mm standard, 9,100 mm special)

Use Code Interpreter to check each dimension.
```

---

### Node 10: IF - Passes Dimensional Constraints?

**Node**: `IF`

**Condition**:
```javascript
{{ $json.status !== "INFEASIBLE" }}
```

**True**: Continue to pricing  
**False**: Generate rejection

---

### Node 11: AI Agent - Rejection Letter Generator (If Fails)

**Node**: `AI Agent`

**Model**: `anthropic/claude-3.5-sonnet` (via OpenRouter) - excellent at writing

**System Prompt**:
```
You are a customer service specialist for Linjateras Oy writing polite rejection letters.

Generate a professional, empathetic email explaining that the parts exceed our equipment capacity.

Be helpful and suggest alternatives:
1. Redesigning to fit within constraints
2. Splitting assemblies into smaller components  
3. Alternative service providers if appropriate

Tone: Professional, helpful, apologetic but firm.

Include: Specific dimension violations, our equipment limits, next steps for customer.

Format: Professional email in English (Linjateras serves international clients).
```

**Input**:
```javascript
Customer: {{ $json.customer_company }}
Violation details: {{ $json.violations }}
Dimensions: {{ $json.dimensions.width_mm }}mm W × {{ $json.dimensions.height_mm }}mm H × {{ $json.dimensions.length_mm }}mm L
Our limits: 1,150mm W × 2,100mm H × 6,600mm L (9,100mm special)
```

---

### Node 12: AI Agent - Surface Area & Complexity Calculator

**Node**: `AI Agent`

**Model**: `anthropic/claude-3.5-sonnet` (via OpenRouter) - excellent at calculations

**System Prompt**:
```
You are a surface area calculation specialist for powder coating quotes.

YOUR TASK:
Calculate total quotable surface area considering quantity, waste factors, and complexity.

INPUTS YOU'LL RECEIVE:
- Surface area per part (m²)
- Quantity of parts
- Complexity factor (1.0-1.5)
- Part geometry type

CALCULATION STEPS (use Code Interpreter):

Step 1: Verify Per-Part Area
If not provided, calculate from dimensions using geometry type

Step 2: Apply Quantity
Total_Base_Area = surface_area_per_part × quantity

Step 3: Add Waste Buffer
- Small parts (<1 m² each): +15% waste
- Medium parts (1-10 m²): +10% waste  
- Large parts (>10 m²): +5% waste

Step 4: Calculate Quotable Area
Quotable_Area = Total_Base_Area × (1 + waste_buffer)

Step 5: Validate
- Check if exceeds daily capacity (6,000 m²)
- Estimate coating days needed
- Flag if multi-day job

OUTPUT JSON:
{
  "per_part_area_m2": 0.0,
  "quantity": 0,
  "total_base_area_m2": 0.0,
  "waste_buffer_percent": 0,
  "quotable_area_m2": 0.0,
  "estimated_coating_days": 0,
  "multi_day_job": false,
  "calculation_breakdown": "show steps"
}

Use Code Interpreter for all math. Show your calculations in calculation_breakdown.
```

**Tools Enabled**:
- ✅ **Code Interpreter** (REQUIRED for calculations)
- ✅ **Web Search** (optional - check current material prices)

**Input**:
```javascript
Calculate quotable surface area for:

Surface area per part: {{ $json.surface_area_per_part }} m²
Quantity: {{ $json.quantity }}
Complexity factor: {{ $json.complexity_factor }}
Geometry: {{ $json.dimensions.geometry_type }}
```

---

### Node 13: AI Agent - Pricing Calculator

**Node**: `AI Agent`

**Model**: `openai/gpt-4o` (via OpenRouter)

**System Prompt**:
```
You are a powder coating pricing specialist for Linjateras Oy using Finnish/European market rates.

PRICING TIERS (EUR per m²):
BASE RATES:
- Standard (stock RAL colors, steel/aluminum): €35-45/m²
- Premium (custom colors, specialty finishes): €50-65/m²
- Specialty (anti-bacterial, heat-resistant, ESD): €70-90/m²

ADJUSTMENTS:
Material:
- Steel: 1.0×
- Aluminum: 1.0×
- Galvanized: 1.15×
- Other: 1.20×

Color:
- Standard RAL in stock: 1.0×
- Custom color match: 1.20× + €75 setup
- Multi-color (2-3): 1.30× + €50/color
- Multi-color (4+): 1.40× + €50/color

Surface Prep:
- Clean/new: €0
- Light cleaning: €0 (included)
- Sandblasting: +€25/m²
- Heavy rust removal: +€40/m²
- Paint stripping: +€30/m²

Complexity:
- Simple flat: 1.0×
- Moderate (tubes, angles): 1.15×
- Complex (assemblies): 1.30×

Volume Discounts:
- 100-500 m²: 10% off
- 500-1000 m²: 20% off
- 1000+ m²: 25% off

Urgency Premiums:
- Standard (5 days): 1.0×
- Fast (3 days): 1.25×
- Rush (1-2 days): 1.50×

CALCULATION (use Code Interpreter):
```python
base_cost = quotable_area_m2 * base_rate_per_m2
material_adjusted = base_cost * material_factor
color_adjusted = material_adjusted * color_factor + setup_fees
prep_added = color_adjusted + (quotable_area_m2 * prep_rate)
complexity_adjusted = prep_added * complexity_factor
volume_discounted = complexity_adjusted * (1 - discount_rate)
urgency_adjusted = volume_discounted * urgency_factor
subtotal = urgency_adjusted
vat = subtotal * 0.24  # Finnish VAT
total = subtotal + vat

# Apply €500 minimum if needed
if subtotal < 500:
    subtotal = 500
    vat = 120
    total = 620
```

OUTPUT JSON:
{
  "tier": "standard|premium|specialty",
  "base_rate_per_m2": 0,
  "quotable_area_m2": 0,
  "base_cost": 0,
  "adjustments": {
    "material": {"factor": 0, "amount": 0},
    "color": {"factor": 0, "amount": 0, "setup": 0},
    "surface_prep": {"rate_per_m2": 0, "amount": 0},
    "complexity": {"factor": 0, "amount": 0},
    "volume_discount": {"rate": 0, "savings": 0},
    "urgency": {"factor": 0, "amount": 0}
  },
  "subtotal_excl_vat": 0,
  "vat_24_percent": 0,
  "total_incl_vat": 0,
  "minimum_charge_applied": false,
  "pricing_notes": [],
  "calculation_steps": "detailed breakdown"
}

Use Code Interpreter for all calculations. Be precise.
```

**Tools Enabled**:
- ✅ **Code Interpreter** (REQUIRED)
- ✅ **Web Search** (optional - verify current market rates)

**Input**:
```javascript
Calculate pricing for this powder coating project:

Quotable Surface Area: {{ $json.quotable_area_m2 }} m²
Material: {{ $json.material_type }}
Coating Type: {{ $json.coating_type }}
Color: {{ $json.color_spec.type }} - {{ $json.color_spec.codes.length }} color(s)
Surface Prep: {{ $json.surface_prep_needed }}
Complexity: {{ $json.complexity_factor }}
Urgency: {{ $json.delivery_urgency }}
```

---

### Node 14: Code - Apply Business Rules & Minimum Charge

**Node**: `Code`

**Code**:
```javascript
const pricing = $input.first().json;

// Parse if still JSON string
let price_data;
try {
  price_data = typeof pricing === 'string' ? JSON.parse(pricing) : pricing;
} catch (e) {
  const match = pricing.match(/\{[\s\S]*\}/);
  price_data = match ? JSON.parse(match[0]) : pricing;
}

// Apply absolute minimum charge
let final_total = price_data.total_incl_vat;
let minimum_applied = false;

if (price_data.subtotal_excl_vat < 500) {
  price_data.subtotal_excl_vat = 500;
  price_data.vat_24_percent = 120;
  final_total = 620;
  minimum_applied = true;
}

// Determine review requirement
const requires_review = final_total > 5000 || 
                       $node.ParseDrawingAnalysis.json.needs_review ||
                       $node.ParseRequirements.json.confidence_score < 0.85;

return [{
  pricing_breakdown: price_data,
  final_quote_eur: final_total,
  minimum_charge_applied: minimum_applied,
  requires_human_review: requires_review,
  quote_category: final_total > 5000 ? 'high' : (final_total > 1000 ? 'medium' : 'low')
}];
```

---

### Node 15: AI Agent - Professional Quote Generator

**Node**: `AI Agent`

**Model**: `anthropic/claude-3.5-sonnet` (via OpenRouter) - excellent at structured writing

**System Prompt**:
```
You are a professional quotation document writer for Linjateras Oy.

Generate a complete, customer-ready quote document in professional Markdown format.

COMPANY DETAILS TO INCLUDE:
Linjateras Oy
Sileesuonkatu 38, 33330 Tampere, Finland
Tel: +358 3 342 4600
Email: linjateras@linjateras.fi
Website: www.linjateras.fi

QUOTE STRUCTURE:

# POWDER COATING QUOTATION

**Quote Number:** [from data]  
**Date:** [today's date]  
**Valid Until:** [30 days from today]

---

## CUSTOMER INFORMATION
[Customer details from data]

---

## PROJECT DESCRIPTION
[Brief description of parts and coating requirements]

**Scope of Work:**
- Parts: [description]
- Quantity: [X] pieces
- Material: [material type]
- Coating Type: [powder coating specification]
- Color: [RAL code or description]
- Finish: [glossy/matte/etc.]
- Surface Preparation: [included services]

**Technical Specifications:**
- Dimensions: [L × W × H] mm
- Surface Area: [X.X] m² total
- Pre-Treatment: 7-stage automated washing (65m)
- Coating: Electrostatic powder application
- Curing: 30m oven, 160-220°C

---

## QUALITY ASSURANCE
- ISO 9001:2015 Quality Management
- ISO 14001:2015 Environmental Standards
- ISO 45001 Occupational Health & Safety
- Real-time tracking via customer portal
- Video documentation (2-month retention)

---

## PRICING

| Description | Quantity/Area | Rate | Amount (EUR) |
|-------------|---------------|------|--------------|
| Powder Coating - [Spec] | [X.X] m² | €[X]/m² | €[X,XXX] |
| [Surface Prep if applicable] | [X.X] m² | €[X]/m² | €[XXX] |
| [Setup fees if applicable] | - | - | €[XX] |
| **Subtotal (excl. VAT)** | | | **€[X,XXX]** |
| **VAT 24%** | | | €[XXX] |
| **TOTAL (incl. VAT)** | | | **€[X,XXX]** |

[If volume discount applied, note it]
[If minimum charge applied, note it]

---

## DELIVERY & LEAD TIME
- Turnaround: [based on urgency level]
- Delivery: Customer pickup or delivery service available
- Drop-off Hours: Mon-Fri, 07:00-15:30

[If special handling: Note extended length handling]

---

## TERMS & CONDITIONS
- Payment: Net 30 days from invoice
- Quote Validity: 30 days
- Setup charges on first run only
- Standard quality control included

---

## CONTACT
For questions or to proceed:

**Sales Team:**
- Ari Kesti, Sales Director: ari.kesti@linjateras.fi, +358 40 586 7964
- Lauri Suontausta: lauri.suontausta@linjateras.fi, +358 44 579 4377
- Emmi Seppänen: emmi.seppanen@linjateras.fi, +358 40 123 9612

---

**Linjateras Oy - 60 Years of Powder Coating Excellence**

Format professionally. Use clear tables. Include all pricing details from input data.
```

**Tools Enabled**: None (pure text generation)

**Input**:
```javascript
Generate professional quote for:

Quote Number: {{ $json.rfp_id }}
Customer: {{ $json.customer_company }}
Contact: {{ $json.customer_contact }}
Email: {{ $json.customer_email_verified }}

Project: {{ $json.project_description }}
Parts: {{ $json.parts_description }}
Material: {{ $json.material_type }}
Dimensions: {{ $json.dimensions.length_mm }} × {{ $json.dimensions.width_mm }} × {{ $json.dimensions.height_mm }} mm
Surface Area: {{ $json.quotable_area_m2 }} m²
Quantity: {{ $json.quantity }}

Coating Spec:
- Type: {{ $json.coating_type }}
- Color: {{ $json.color_spec.codes }}
- Finish: {{ $json.finish }}

Pricing Breakdown:
{{ JSON.stringify($json.pricing_breakdown, null, 2) }}

TOTAL: €{{ $json.final_quote_eur }}

Urgency: {{ $json.delivery_urgency }}
Special Notes: {{ $json.special_notes }}
```

---

### Node 16: AI Agent - Quote Quality Validator

**Node**: `AI Agent`

**Model**: `openai/gpt-4o-mini` (via OpenRouter) - fast validation

**System Prompt**:
```
You are a quality control validator checking quote documents.

REQUIRED CHECKS:
✓ All fields populated (no [TBD], [X], TODO, etc.)
✓ Customer name and email present
✓ Total cost is reasonable (€20-100/m² range)
✓ Math is correct (line items sum properly, VAT = 24%)
✓ All sections present (header, pricing table, terms, contact)
✓ Professional formatting (no typos)
✓ Lead time matches urgency level

OUTPUT JSON:
{
  "validation_passed": true/false,
  "issues_found": [],
  "confidence": 0.0-1.0,
  "recommendation": "APPROVE|REVIEW|REJECT",
  "quality_score": 0-100
}
```

**Tools Enabled**:
- ✅ **Code Interpreter** (for math validation)

**Input**:
```javascript
Validate this quote document:

{{ $node.GenerateQuote.json.output }}

Expected total: €{{ $json.final_quote_eur }}
```

---

### Node 17: Switch - Quote Value Category

**Node**: `Switch`

**Mode**: Rules

**Rules**:
1. **High Value** → `{{ $json.final_quote_eur > 5000 }}` → Output 0 (Human Approval)
2. **Medium Value** → `{{ $json.final_quote_eur > 1000 }}` → Output 1 (AI Quality Check)
3. **Low Value** → `{{ $json.final_quote_eur <= 1000 }}` → Output 2 (Auto-approve)

---

### Node 18: Wait - Human Approval (High-Value Projects)

**Node**: `Wait`

**Mode**: Webhook

**Configuration**:
- Wait for incoming webhook with approval/rejection
- Timeout: 24 hours
- Resume URL sent to sales team via Slack

**Alternative**: Use `n8n Form` for approval interface

---

### Node 19: Send Email - Deliver Quote

**Node**: `Gmail` or `Send Email`

**Configuration**:
- **To**: `{{ $json.customer_email_verified }}`
- **CC**: `quotes@linjateras.fi`
- **Subject**: `Linjateras Powder Coating Quote {{ $json.rfp_id }} - {{ $json.customer_company }}`
- **Body**: `{{ $node.GenerateQuote.json.output }}`
- **Format**: HTML (convert markdown to HTML first with Markdown node)

---

### Node 20: Save Quote to Database

**Node**: `Supabase` or `Postgres`

**Operation**: Insert

**Table**: `quotes`

**Data to Insert**:
```javascript
{
  quote_id: "{{ $json.rfp_id }}",
  created_at: "{{ $now.toISO() }}",
  customer_company: "{{ $json.customer_company }}",
  customer_email: "{{ $json.customer_email_verified }}",
  customer_contact: "{{ $json.customer_contact }}",
  project_description: "{{ $json.project_description }}",
  material_type: "{{ $json.material_type }}",
  dimensions_json: {{ JSON.stringify($json.dimensions) }},
  surface_area_m2: {{ $json.quotable_area_m2 }},
  quantity: {{ $json.quantity }},
  coating_specifications_json: {{ JSON.stringify($json.coating_requirements) }},
  pricing_breakdown_json: {{ JSON.stringify($json.pricing_breakdown) }},
  subtotal_excl_vat: {{ $json.pricing_breakdown.subtotal_excl_vat }},
  vat_amount: {{ $json.pricing_breakdown.vat_24_percent }},
  total_incl_vat: {{ $json.final_quote_eur }},
  urgency: "{{ $json.delivery_urgency }}",
  status: "sent",
  quote_document: "{{ $node.GenerateQuote.json.output }}",
  confidence_score: {{ $json.confidence }},
  required_manual_review: {{ $json.requires_human_review }},
  approved_by: "{{ $json.approved_by || 'auto' }}",
  flags_json: {{ JSON.stringify($json.flags) }}
}
```

---

### Node 21: Notify Sales Team

**Node**: `Slack`

**Operation**: Send Message

**Channel**: `#sales-quotes` or similar

**Message**:
```
🎯 New Quote Generated: {{ $json.rfp_id }}

**Customer:** {{ $json.customer_company }}
**Project:** {{ $json.parts_description }}
**Total:** €{{ $json.final_quote_eur }}
**Status:** {{ $json.requires_human_review ? '⚠️ Needs Review' : '✅ Auto-Approved' }}

📊 Details:
- Surface Area: {{ $json.quotable_area_m2 }} m²
- Material: {{ $json.material_type }}
- Urgency: {{ $json.delivery_urgency }}
- Confidence: {{ Math.round($json.confidence * 100) }}%

🔗 View: [Link to quote in database]
```

---

## 4. OpenRouter Configuration in n8n

### Setting Up OpenRouter Credentials

**Node**: Create credential in n8n

**Credential Type**: `HTTP Header Auth` or custom `OpenRouter API`

**Configuration**:
- **Header Name**: `Authorization`
- **Header Value**: `Bearer YOUR_OPENROUTER_API_KEY`
- **Additional Headers**:
  - `HTTP-Referer`: `https://linjateras.fi`
  - `X-Title`: `Linjateras RFP Analyzer`

### OpenRouter Model Selection

**Available Models via OpenRouter**:

| Model | Use Case | Cost/1M tokens | Speed |
|-------|----------|----------------|-------|
| `anthropic/claude-3.5-sonnet` | Requirements extraction, writing | $3-15 | Fast |
| `openai/gpt-4o` | Vision analysis, general tasks | $5-15 | Fast |
| `openai/gpt-4o-mini` | Validation, simple tasks | $0.15-0.60 | Very Fast |
| `google/gemini-pro-1.5` | Alternative vision | $3.50 | Fast |
| `meta-llama/llama-3.1-70b` | Open source option | $0.60 | Medium |

**Recommended Distribution**:
- Agent 1 (Requirements): Claude 3.5 Sonnet
- Agent 2 (Drawing Analysis): GPT-4o  
- Agent 3 (Validation): GPT-4o-mini
- Agent 4 (Surface Calc): Claude 3.5 Sonnet
- Agent 5 (Pricing): GPT-4o
- Agent 6 (Quote Writing): Claude 3.5 Sonnet
- Agent 7 (Quality Check): GPT-4o-mini

---

## 5. n8n Tools & Community Nodes

### Built-in Tools Available in AI Agent Nodes

✅ **Code Interpreter**
- Execute Python code in sandbox
- Use for: Surface calculations, pricing math
- Enable: Check box in AI Agent node

✅ **Vision** 
- Analyze images/PDFs (automatic with gpt-4o/claude-3.5-sonnet)
- Use for: Technical drawings, RFP scans
- Enable: Automatic when using vision-capable model

✅ **Web Search**
- Real-time internet search
- Use for: Current powder coating prices, material costs
- Enable: Check box in AI Agent node

✅ **File Search / RAG**
- Query uploaded documents
- Use for: Multi-page RFPs, multiple drawings
- Enable: Upload to vector store, check box

### Community Nodes to Install

**Install these for enhanced functionality**:

1. **@n8n/n8n-nodes-langchain** (if not built-in)
   - Advanced AI chains
   - Memory management
   - Tool calling

2. **n8n-nodes-document-processor**
   - PDF extraction
   - Image preprocessing
   - OCR capabilities

3. **n8n-nodes-supabase**
   - Vector storage
   - Postgres database
   - Real-time subscriptions

4. **n8n-nodes-browserless**
   - Web scraping (if needed for competitor analysis)
   - Screenshot generation

### Native n8n Nodes to Use

✅ **Edit Fields (Set)** - State management  
✅ **IF / Switch** - Conditional routing  
✅ **Code** - JavaScript processing  
✅ **Merge** - Combine data branches  
✅ **Filter** - Data filtering  
✅ **Aggregate** - Group calculations  
✅ **HTTP Request** - API calls (fallback)  
✅ **Gmail** - Send emails  
✅ **Slack** - Notifications  
✅ **Postgres / Supabase** - Database  
✅ **Markdown** - Convert to HTML  
✅ **Wait** - Human approval  
✅ **Webhook** - External triggers  

---

## 6. Alternative: Using n8n's LangChain Nodes

### LangChain-Based Architecture

```
[n8n Form Trigger]
    ↓
[OpenAI Chat Model] (with Memory)
    ↓
[Vector Store Tool] (for document search)
    ↓
[Python Function Tool] (for calculations)
    ↓
[Agent Executor]
    ↓
[Output Parser]
```

**When to use LangChain nodes**:
- Need conversation memory
- Building chatbot interface
- Complex tool chaining
- RAG requirements

**When to use AI Agent nodes**:
- Simpler workflow
- Less setup overhead
- Direct model access
- Better for production

---

## 7. Error Handling & Retry Logic

### Per-Agent Error Handling

**Add after each AI Agent node**:

**Node**: `IF - Check Agent Success`

**Condition**:
```javascript
{{ $json.error === undefined && $json.output !== "" }}
```

**False Branch** → Retry Logic:

**Node**: `Code - Implement Retry`

```javascript
const retry_count = $node["Edit Fields"].json.retry_count || 0;
const MAX_RETRIES = 3;

if (retry_count < MAX_RETRIES) {
  return [{
    retry: true,
    retry_count: retry_count + 1,
    wait_seconds: Math.pow(2, retry_count) * 5 // Exponential backoff
  }];
} else {
  return [{
    retry: false,
    failed: true,
    error_message: 'AI Agent failed after 3 retries',
    escalate_to_human: true
  }];
}
```

### Workflow-Level Error Catch

**Node**: `Error Trigger` (catches workflow errors)

**Connected to**:

**Node**: `Code - Log Error`

```javascript
return [{
  workflow_id: $execution.id,
  error_message: $json.error.message,
  failed_node: $json.error.node.name,
  timestamp: $now.toISO(),
  customer_email: $('Edit Fields').item.json.customer_email,
  rfp_id: $('Edit Fields').item.json.rfp_id
}];
```

**Then**:

**Node**: `Send Email - Notify Support`

```javascript
Subject: ⚠️ Workflow Error - {{ $json.rfp_id }}
Body: Workflow failed at {{ $json.failed_node }}
Error: {{ $json.error_message }}
Customer: {{ $json.customer_email }}
```

---

## 8. Complete Workflow Implementation Checklist

### Phase 1: Setup (Week 1)

- [ ] Install n8n (self-hosted or cloud)
- [ ] Get OpenRouter API key (https://openrouter.ai/)
- [ ] Get OpenAI API key (https://platform.openai.com/)
- [ ] Set up credentials in n8n
- [ ] Install community nodes if needed

### Phase 2: Build Core Workflow (Week 1-2)

- [ ] Create n8n Form Trigger with file upload
- [ ] Add Edit Fields node for state management
- [ ] Add AI Agent 1: RFP Requirements Extractor
  - [ ] Configure OpenRouter credential
  - [ ] Select model (Claude 3.5 Sonnet)
  - [ ] Enable Code Interpreter tool
  - [ ] Test with sample RFP
- [ ] Add Code node: Parse Requirements JSON
- [ ] Add Switch node: Route by drawing availability
- [ ] Add AI Agent 2: Technical Drawing Analyzer
  - [ ] Select GPT-4o model
  - [ ] Enable Vision + Code Interpreter
  - [ ] Test with sample drawings
- [ ] Add Code node: Parse Drawing JSON
- [ ] Add Merge node: Combine data paths

### Phase 3: Add Validation & Pricing (Week 2)

- [ ] Add AI Agent 3: Dimensional Validator
  - [ ] Select GPT-4o-mini
  - [ ] Enable Code Interpreter
  - [ ] Test with various dimensions
- [ ] Add IF node: Check feasibility
- [ ] Add AI Agent: Rejection Letter (for infeasible)
- [ ] Add AI Agent 4: Surface Area Calculator
  - [ ] Enable Code Interpreter
  - [ ] Test calculations
- [ ] Add AI Agent 5: Pricing Calculator
  - [ ] Enable Code Interpreter + Web Search
  - [ ] Test pricing scenarios
- [ ] Add Code node: Apply business rules

### Phase 4: Quote Generation (Week 2-3)

- [ ] Add AI Agent 6: Quote Generator
  - [ ] Select Claude 3.5 Sonnet
  - [ ] Test quote formatting
- [ ] Add AI Agent 7: Quality Validator
  - [ ] Select GPT-4o-mini
  - [ ] Test validation
- [ ] Add Switch: Value category routing
- [ ] Add Wait node: Human approval flow
- [ ] Test end-to-end quote generation

### Phase 5: Integration & Delivery (Week 3)

- [ ] Add Gmail node: Send quote email
  - [ ] Configure Gmail credentials
  - [ ] Test email delivery
- [ ] Add Supabase/Postgres node: Save quote
  - [ ] Create database schema
  - [ ] Test data insertion
- [ ] Add Slack node: Sales notification
  - [ ] Configure Slack webhook
  - [ ] Test notifications
- [ ] Add error handling to all nodes
- [ ] Add retry logic where needed

### Phase 6: Testing & Optimization (Week 3-4)

- [ ] Test with 20+ diverse RFPs
- [ ] Measure accuracy (dimensions, pricing)
- [ ] Optimize prompts based on failures
- [ ] Test error scenarios
- [ ] Load test (10+ concurrent workflows)
- [ ] Validate cost per execution

### Phase 7: Production (Week 4)

- [ ] Final end-to-end testing
- [ ] Create user documentation
- [ ] Train sales team
- [ ] Soft launch with monitoring
- [ ] Full production deployment
- [ ] Set up monitoring dashboard

---

## 9. Cost Analysis

### Monthly Operational Costs

**n8n Hosting**:
- Self-hosted (DigitalOcean): €20-50/month
- n8n Cloud: €50-100/month (depending on executions)

**OpenRouter API Costs** (per quote):
- Agent 1 (Requirements): ~€0.02-0.05
- Agent 2 (Drawing Analysis): ~€0.05-0.15
- Agent 3 (Validation): ~€0.005-0.01
- Agent 4 (Surface Calc): ~€0.02-0.05
- Agent 5 (Pricing): ~€0.02-0.05
- Agent 6 (Quote Gen): ~€0.03-0.08
- Agent 7 (Quality Check): ~€0.005-0.01

**Total per quote**: €0.15-0.40

**Other Services**:
- Gmail: Free (within limits)
- Supabase: Free tier or €25/month
- Slack: Free tier

**TOTAL: €50-200/month for 200-300 quotes/month**

---

## 10. Monitoring & Metrics

### Add Monitoring Node at End

**Node**: `Code - Log Metrics`

```javascript
const metrics = {
  rfp_id: $json.rfp_id,
  customer: $json.customer_company,
  workflow_duration_ms: new Date() - new Date($node["Edit Fields"].json.workflow_start_time),
  total_quote_eur: $json.final_quote_eur,
  required_review: $json.requires_human_review,
  confidence_score: $json.confidence,
  agent_calls: {
    requirements_extraction: true,
    drawing_analysis: $json.has_technical_drawings,
    validation: true,
    surface_calc: true,
    pricing: true,
    quote_gen: true,
    quality_check: true
  },
  status: 'completed',
  timestamp: $now.toISO()
};

// Send to analytics
return [metrics];
```

### Success Metrics to Track

- **Processing Time**: Target < 5 minutes
- **Automation Rate**: Target 80%
- **Dimension Accuracy**: Target ±5%
- **Pricing Accuracy**: Target ±10%
- **Quote Completeness**: Target 100%
- **Customer Satisfaction**: Survey after quote

---

## Conclusion

This **n8n AI Agent workflow** provides a production-ready solution for Linjateras using:

✅ **Native AI Agent nodes** - No HTTP requests needed  
✅ **OpenRouter integration** - Multi-model flexibility  
✅ **Built-in tools** - Code Interpreter, Vision, Web Search  
✅ **Community nodes** - Extended functionality  
✅ **Visual workflow** - Easy to understand and modify  
✅ **Scalable architecture** - Handle 300+ quotes/month  
✅ **Cost-effective** - €50-200/month operational cost  

**Expected Outcomes**:
- 3-5× faster quote generation
- 80% automation (20% human review for complex cases)
- Consistent pricing based on market rates
- Professional quote documents every time

**Implementation Timeline**: 2-4 weeks to production

