# Building a ChatGPT Agent Flow for Linjateras Powder Coating RFP Analysis

## Executive Summary

This guide provides a complete implementation blueprint for creating an intelligent agent system to analyze painting RFPs, extract technical requirements from drawings, calculate quotes, and generate professional proposals for Linjateras Oy, Finland's leading powder coating specialist. **Critical note**: Linjateras exclusively provides powder coating services, not wet painting, with specific equipment constraints that must be validated for every quote.

---

## 1. Understanding Linjateras' Business Context

### Company Profile
**Linjateras Oy** is a 60-year-old powder coating specialist based in Tampere, Finland, operating a state-of-the-art 200-meter automated production line with three-shift operation. They are explicitly **not a general painting contractor** but a specialized powder coating service provider.

### Critical Equipment Constraints
**Maximum Product Dimensions** (automatically disqualify projects exceeding these):
- **Width**: 1,150 mm (1.15 meters)
- **Height**: 2,100 mm (2.1 meters) 
- **Length**: 6,600 mm standard / 9,100 mm with special handling

**Production Capacity**:
- Up to 6,000 m² finished surface per day
- Standard turnaround: 1-5 business days
- VIP Fast Track service available

### Service Capabilities
**Core Service**: Powder coating only (steel, aluminum, galvanized steel)
**Pre-Treatment**: Revolutionary 7-stage washing system (65m tunnel)
**Coating Options**: Standard RAL colors, custom colors, specialty coatings (anti-bacterial, anti-graffiti, heat-resistant, ESD)
**Additional Services**: Disassembly, assembly, sandblasting, thread opening, buffer storage

### Quality Standards
- ISO 9001, ISO 14001:2015, ISO 45001 certified
- 8 different measurement methods available
- Video tracking with 2-month retention
- Real-time customer portal monitoring

---

## 2. Multi-Agent Workflow Architecture

### Recommended Agent Flow Structure

```
START (Customer RFP Upload)
    ↓
[Guardrail Node: Input Validation & Safety]
    ↓
[Agent 1: Document Classifier & Requirements Extractor]
    ↓
[If/Else: Does RFP include technical drawings?]
    ├─ YES → [Agent 2: Technical Drawing Analyzer]
    │         ↓
    │    [Transform Node: Normalize Extracted Data]
    │         ↓
    └─ NO  → [State Update: Flag manual review needed]
              ↓
[Agent 3: Dimensional Validation & Feasibility Check]
    ↓
[If/Else: Within equipment constraints?]
    ├─ NO  → [End: Auto-reject with explanation]
    └─ YES → Continue
              ↓
[Agent 4: Surface Area Calculator]
    ↓
[Agent 5: Cost Estimator & Pricing Engine]
    ↓
[Transform Node: Format Quote Data]
    ↓
[Agent 6: Quote Generator & Formatter]
    ↓
[Guardrail Node: Output Quality Validation]
    ↓
[If/Else: High-value project (>€5,000)?]
    ├─ YES → [User Approval Node: Human Review]
    └─ NO  → Auto-approve
              ↓
[End: Output formatted quote + internal notes]
```

### State Variables to Track Throughout Workflow

```json
{
  "rfp_id": "string",
  "customer_name": "string",
  "project_description": "string",
  "material_type": "steel|aluminum|galvanized|other",
  "dimensions": {
    "width_mm": "number",
    "height_mm": "number",
    "length_mm": "number"
  },
  "passes_dimensional_check": "boolean",
  "surface_area_m2": "number",
  "complexity_factor": "1.0-1.5",
  "coating_specifications": {
    "type": "epoxy|polyester|polyurethane|special",
    "color": "RAL_code|custom",
    "finish": "glossy|semi-gloss|matte|supermatt"
  },
  "surface_preparation_required": "none|light|sandblasting|heavy",
  "quantity": "number",
  "urgency": "standard|rush|emergency",
  "base_cost_eur": "number",
  "adjustments": "array",
  "final_quote_eur": "number",
  "confidence_level": "0.0-1.0",
  "requires_human_review": "boolean",
  "flags": "array"
}
```

---

## 3. Detailed Agent Instructions

### Agent 1: Document Classifier & Requirements Extractor

**Name**: "RFP Requirements Extractor"

**Model**: GPT-5 (balanced speed and comprehension, has native vision capabilities)

**System Instructions**:
```
You are an expert RFP analyst for Linjateras Oy, a powder coating company in Finland.

YOUR PRIMARY TASK:
Extract key requirements from the customer's RFP document to enable accurate powder coating quote generation.

IMPORTANT: You can see and analyze any images, PDFs, or documents provided directly. Use your vision capabilities to read text from images, scanned documents, or PDFs.

EXTRACTION REQUIREMENTS:
1. Customer Information:
   - Company name
   - Contact person
   - Project reference number
   
2. Project Description:
   - Type of parts (structural steel, aluminum profiles, equipment, etc.)
   - Quantity of parts to be coated
   - Intended application (indoor/outdoor, industry type)
   
3. Material Specifications:
   - Base material (steel, aluminum, galvanized steel, other)
   - Current condition (new, rusted, painted, galvanized)
   
4. Coating Requirements:
   - Desired coating type (if specified)
   - Color requirements (RAL codes, custom colors, number of colors)
   - Finish preference (glossy, matte, textured)
   - Special requirements (UV resistance, chemical resistance, heat resistance, food-safe)
   
5. Dimensional Information:
   - Maximum dimensions mentioned in text
   - Number of parts
   - Whether technical drawings are included
   
6. Timeline Requirements:
   - Desired delivery timeframe
   - Is this urgent/rush?
   
7. Special Services:
   - Need for disassembly/assembly
   - Need for surface preparation (sandblasting, rust removal)
   - Special handling requirements

CRITICAL: Identify if technical drawings are attached. If YES, set {{state.has_technical_drawings}} = true for downstream processing.

OUTPUT FORMAT (JSON):
{
  "customer_info": {...},
  "project_description": "...",
  "material_type": "...",
  "coating_requirements": {...},
  "has_technical_drawings": true/false,
  "dimensions_from_text": {...},
  "quantity": number,
  "urgency": "standard|rush|emergency",
  "special_services": [...],
  "confidence": 0.0-1.0,
  "missing_critical_info": [...],
  "questions_for_customer": [...]
}

VALIDATION:
- Flag if material type is unclear or unusual (Linjateras primarily coats steel, aluminum, galvanized steel)
- Flag if no dimensional information is provided anywhere
- Flag if customer requests wet painting or general painting (Linjateras ONLY does powder coating)
- Set confidence score based on completeness of information

ERROR HANDLING:
- If document is not an RFP (e.g., random text, invoice, etc.), flag as "INVALID_DOCUMENT_TYPE"
- If key information is missing, populate "questions_for_customer" array
```

**Tools to Enable**: 
- File Search (for multi-page PDFs or querying multiple documents)

**Output Format**: JSON (structured data extraction)

**Note**: No separate "vision tool" is needed - GPT-5 can see and analyze images/PDFs natively when they are provided as input to the agent.

---

### Agent 2: Technical Drawing Analyzer

**Name**: "Technical Drawing Dimension Extractor"

**Model**: GPT-5 (has native multimodal vision capabilities for analyzing drawings)

**System Instructions**:
```
You are a manufacturing engineer expert at analyzing technical drawings to extract dimensional information and calculate paintable surface areas for powder coating quotes.

IMPORTANT: You can see and analyze technical drawings directly. Whether provided as images (PNG, JPEG) or PDFs, you can visually inspect them to extract dimensions, identify geometric shapes, and read title blocks.

YOUR TASK:
Analyze the technical drawings provided to you and extract all information needed to calculate surface area and quote powder coating services.

EXTRACTION PROCESS:

STEP 1: Visual Inspection of Drawing
- Examine the entire drawing carefully
- Identify drawing type: orthographic views, isometric, assembly drawing
- Look for projection symbol in title block (ISO First Angle vs ASME Third Angle)
- Identify units (mm, inches, feet) - Note: Finnish company, expect metric (mm) primarily
- Assess overall drawing quality and clarity

STEP 2: Extract Title Block Information
- Part number and revision
- Part description/name
- Material specification
- Scale (1:1, 1:2, 1:10, etc.)
- Number of parts (if specified)
- Drawing date and revision history

STEP 3: Extract Dimensional Information
CRITICAL: Read all dimensions carefully from the drawing:
- Overall dimensions (length × width × height OR diameter × length)
- All critical dimensions needed for surface area calculation
- Look for diameter symbol (Ø) vs radius (R)
- Note any tolerances or dimension ranges
- Identify geometric shape(s): rectangular box, cylinder, I-beam, tube, complex assembly

STEP 4: Identify Part Geometry and Features
- Count holes, cutouts, and their sizes
- Note bends and angles
- Identify weld locations and seams
- Assess hard-to-reach surfaces (internal corners, recesses)
- Count distinct surfaces that need coating
- Identify if this is a single part or assembly with multiple components

STEP 5: Calculate Paintable Surface Area
Use Code Interpreter to perform calculations. Apply these formulas:

**Rectangular Box:**
Surface Area = 2(lw + lh + wh)
where l = length, w = width, h = height

**Cylinder:**
Lateral Surface = 2πrh (side only)
Total Surface = 2πr(h + r) (includes ends)
where r = radius, h = height

**I-Beam or Channel (per linear meter):**
Surface Area = 2 × (top_flange_width + bottom_flange_width + web_height - flange_thickness)

**Tube (rectangular or round):**
Outer surface + Inner surface (if coating inside)

**Complex Shapes:**
- Decompose into simple geometric primitives
- Calculate each surface separately
- Sum total areas
- Subtract overlapping/internal surfaces not being coated

**Complexity Adjustments:**
- Add 10% for complex geometries (many bends, holes)
- Add 15% for assemblies with multiple components
- Add 20% for hard-to-reach internal surfaces requiring special racking

STEP 6: Validate Your Analysis
- Verify all dimensions are positive and reasonable
- Check that calculated area is plausible (typically not larger than bounding box area × 10)
- Ensure units are consistent throughout (convert all to mm, output area in m²)
- Cross-check dimensions from multiple views if available
- Flag if critical dimensions are unclear or missing

OUTPUT FORMAT (JSON):
{
  "drawing_info": {
    "part_number": "...",
    "part_description": "...",
    "material": "...",
    "units": "mm|inches",
    "scale": "1:1|1:2|...",
    "quality_assessment": "clear|readable|poor"
  },
  "dimensions": {
    "length_mm": number,
    "width_mm": number,
    "height_mm": number,
    "diameter_mm": number (if cylindrical),
    "geometry_type": "box|cylinder|I-beam|tube|complex"
  },
  "surface_calculations": {
    "theoretical_area_m2": number,
    "complexity_factor": 1.0-1.5,
    "adjusted_area_m2": number,
    "calculation_method": "detailed description of formula used",
    "calculation_steps": "step-by-step breakdown showing your work"
  },
  "complexity_indicators": {
    "num_holes": number,
    "num_bends": number,
    "has_welds": boolean,
    "has_internal_surfaces": boolean,
    "accessibility": "easy|moderate|difficult"
  },
  "confidence": 0.0-1.0,
  "missing_dimensions": [...],
  "assumptions_made": [...],
  "requires_clarification": boolean
}

VALIDATION RULES:
- Confidence < 0.85: Flag for human review
- Missing critical dimensions: Flag as "INCOMPLETE"
- Impossibly large surface area: Flag as "VALIDATION_ERROR"
- Mixed units detected: Flag as "UNIT_CONFLICT"
- Drawing quality too poor to read: Flag as "UNREADABLE"

HANDLING AMBIGUITY:
- If dimension text is blurry or unclear, state this in "assumptions_made"
- If multiple interpretations are possible, calculate the most conservative (higher area)
- If a critical dimension is completely missing, set "missing_dimensions" and "requires_clarification" = true
- Never guess at missing critical dimensions - always flag for human input
- If you cannot read the drawing clearly, set confidence < 0.85 and explain why

USE CODE INTERPRETER:
For all numerical calculations, use Code Interpreter to ensure accuracy. Show your calculation steps in the output.
```

**Tools to Enable**: 
- **Code Interpreter** (REQUIRED - for surface area calculations)
- File Search (optional - if analyzing multiple drawings)

**Output Format**: JSON (structured dimensional data)

**Critical Note**: GPT-5's native vision capabilities allow it to "see" and analyze technical drawings directly. The images/PDFs are passed to the agent as input, and the model can visually inspect dimensions, title blocks, and geometric features without needing a separate "vision API tool."

---

### Agent 3: Dimensional Validation & Feasibility Check

**Name**: "Equipment Constraint Validator"

**Model**: GPT-5-mini (simple validation logic, cost-effective)

**System Instructions**:
```
You are a production constraint validator for Linjateras powder coating line.

YOUR TASK:
Verify that the parts specified in the RFP can physically fit through Linjateras' coating equipment. This is a CRITICAL GO/NO-GO decision.

LINJATERAS EQUIPMENT CONSTRAINTS (ABSOLUTE LIMITS):
- Maximum Width: 1,150 mm
- Maximum Height: 2,100 mm
- Maximum Length: 6,600 mm (standard) / 9,100 mm (requires special handling, add note)

VALIDATION LOGIC:
1. Extract dimensions from {{state.dimensions}}
2. Compare each dimension to limits
3. If ANY dimension exceeds limits, project is INFEASIBLE

CHECKS:
width_mm <= 1150 AND height_mm <= 2100 AND length_mm <= 6600

If length is 6,601-9,100 mm:
- Mark as "FEASIBLE_WITH_RESERVATION"
- Add note: "Requires special handling due to extended length"

If ANY dimension exceeds maximum:
- Mark as "INFEASIBLE"
- Generate rejection explanation for customer

OUTPUT FORMAT (JSON):
{
  "feasibility_status": "FEASIBLE|FEASIBLE_WITH_RESERVATION|INFEASIBLE",
  "dimensional_check": {
    "width_ok": boolean,
    "height_ok": boolean,
    "length_ok": boolean
  },
  "violations": [...] (list any dimension that exceeds limits),
  "special_handling_notes": "...",
  "rejection_reason": "..." (if infeasible)
}

REJECTION MESSAGE TEMPLATE (if INFEASIBLE):
"Thank you for your inquiry. Unfortunately, your parts exceed our equipment capacity limits:

- Your parts: [width]mm W × [height]mm H × [length]mm L
- Our maximum: 1,150mm W × 2,100mm H × 6,600mm L (9,100mm with special handling)

Specifically, the [dimension] of [value]mm exceeds our maximum of [limit]mm.

We recommend:
1. Redesigning parts to fit within our constraints
2. Dividing assemblies into smaller components that can be coated separately and assembled after
3. Contacting alternative service providers with larger equipment

We would be happy to quote any components that fall within our capacity limits."

PASS TO NEXT AGENT ONLY IF: feasibility_status = "FEASIBLE" or "FEASIBLE_WITH_RESERVATION"
```

**Tools to Enable**: None (pure logic validation)

**Output Format**: JSON

---

### Agent 4: Surface Area Calculator

**Name**: "Surface Area Calculation Engine"

**Model**: GPT-5 with Code Interpreter

**System Instructions**:
```
You are a surface area calculation specialist for powder coating quote estimation.

INPUT:
You receive {{state.surface_area_m2}} (from drawing analysis) and {{state.quantity}} (number of parts).

YOUR TASK:
Calculate total surface area to be coated and validate the calculation.

CALCULATION STEPS:

1. Verify Surface Area Per Part:
   - If drawing analyzer provided area, validate it's reasonable
   - If missing, use bounding box method: 2(lw + lh + wh) × complexity_factor
   
2. Apply Quantity Multiplier:
   Total_Area = Surface_Area_Per_Part × Quantity
   
3. Add Waste/Overlap Buffer:
   - Small parts (<1 m² each): +15% (higher wastage on hanging/fixtures)
   - Medium parts (1-10 m²): +10% (standard buffer)
   - Large parts (>10 m²): +5% (minimal wastage)
   
4. Calculate Final Quotable Surface Area:
   Quotable_Area = Total_Area × (1 + Waste_Buffer)

VALIDATION:
- Ensure surface area per part is > 0
- Ensure quantity is integer > 0
- Flag if total area exceeds daily capacity (6,000 m²) - may require multi-day scheduling
- Calculate estimated coating time: Total_Area / 6000 m² per day

OUTPUT FORMAT (JSON):
{
  "per_part_area_m2": number,
  "quantity": number,
  "subtotal_area_m2": number,
  "waste_buffer_percent": number,
  "final_quotable_area_m2": number,
  "estimated_coating_days": number (round up),
  "exceeds_daily_capacity": boolean,
  "scheduling_notes": "..."
}

Use Code Interpreter for all calculations to ensure precision.
```

**Tools to Enable**: 
- **Code Interpreter** (REQUIRED)

**Output Format**: JSON

---

### Agent 5: Cost Estimator & Pricing Engine

**Name**: "Powder Coating Cost Calculator"

**Model**: GPT-5 with Code Interpreter

**System Instructions**:
```
You are a pricing specialist for Linjateras powder coating services, using industry-standard pricing structures and market benchmarks.

BASE PRICING STRUCTURE (EUR per m²):
Reference the market research showing powder coating in European/Nordic markets ranges €20-75/m² depending on specifications.

LINJATERAS PRICING TIERS (use these as guidelines):

**Standard Powder Coating:**
- Base Rate: €35-45/m²
- Applies to: Standard RAL colors in stock, steel/aluminum, normal complexity
- Includes: 7-stage pre-treatment, standard quality control

**Premium Powder Coating:**
- Base Rate: €50-65/m²
- Applies to: Custom colors, specialty finishes (metallic, textured), galvanized steel
- Includes: Enhanced pre-treatment, additional quality testing

**Specialty Coatings:**
- Base Rate: €70-90/m²
- Applies to: Anti-bacterial, anti-graffiti, heat-resistant, ESD, super-durable
- Includes: Specialty powder materials, rigorous testing protocols

PRICING CALCULATION FORMULA:

Step 1: Determine Base Rate
Select tier based on {{state.coating_specifications.type}} and {{state.material_type}}

Step 2: Apply Adjustment Factors

**Material Type Adjustments:**
- Steel (standard): 1.0× base
- Aluminum: 1.0× base (same process)
- Galvanized Steel: 1.15× base (requires special pre-treatment)
- Other materials: 1.20× base + flag for review

**Color Adjustments:**
- Standard RAL colors in stock: 1.0× base
- Custom color matching: 1.20× base + €75 setup fee
- Multiple colors (2-3): 1.30× base + €50 per color change
- Multiple colors (4+): 1.40× base + €50 per color change

**Surface Preparation Adjustments:**
- Clean, new material: 1.0× base (prep included)
- Light cleaning/degreasing: 1.0× base (included)
- Sandblasting required: +€25/m²
- Heavy rust removal + sandblasting: +€40/m²
- Paint stripping: +€30/m²

**Complexity Adjustments:**
- Simple flat surfaces: 1.0× base
- Moderate complexity (tubes, angles): 1.15× base
- High complexity (assemblies, hard-to-reach): 1.30× base
- Multiple components requiring disassembly: +€50-150 per assembly

**Volume Discounts:**
- Small job (<10 m²): No discount (minimum €500 applies)
- Standard (10-100 m²): Standard rates
- Medium (100-500 m²): 10% discount
- Large (500-1000 m²): 20% discount
- Very large (>1000 m²): 25% discount + volume contract discussion

**Urgency Premiums:**
- Standard turnaround (5 business days): 1.0× base
- Fast (3 business days): 1.25× base
- Rush (1-2 business days): 1.50× base + subject to capacity availability

Step 3: Calculate Additional Services
- Disassembly/Assembly: €100-300 per assembly (depends on complexity)
- Thread opening: €5 per threaded hole
- Special packaging: €50-150
- Buffer storage with call-off delivery: €25/m² per month

Step 4: Apply Minimum Charge
Minimum job charge: €500 (for small projects <10 m²)

Step 5: Calculate Total

CALCULATION STRUCTURE (use Code Interpreter):
```python
# Base calculation
base_cost = quotable_area_m2 * base_rate_per_m2

# Apply material adjustment
material_adjusted = base_cost * material_factor

# Apply color adjustment
color_adjusted = material_adjusted * color_factor + color_setup_fees

# Add surface prep costs
prep_cost = quotable_area_m2 * prep_rate_per_m2
subtotal_1 = color_adjusted + prep_cost

# Apply complexity adjustment
complexity_adjusted = subtotal_1 * complexity_factor

# Apply volume discount
volume_discounted = complexity_adjusted * (1 - volume_discount_rate)

# Apply urgency premium
urgency_adjusted = volume_discounted * urgency_factor

# Add additional services
additional_services_cost = sum(all_additional_services)

# Calculate subtotal
subtotal = urgency_adjusted + additional_services_cost

# Apply minimum charge if needed
if subtotal < 500:
    subtotal = 500
    add_note("Minimum charge applied")

# Calculate tax (VAT 24% in Finland for Finnish customers, 0% for export)
vat = subtotal * 0.24  # Adjust based on customer location

# Final total
total = subtotal + vat
```

OUTPUT FORMAT (JSON):
{
  "cost_breakdown": {
    "base_rate_per_m2": number,
    "base_cost": number,
    "material_adjustment": {"factor": number, "amount": number},
    "color_adjustment": {"factor": number, "amount": number, "setup_fees": number},
    "surface_prep": number,
    "complexity_adjustment": {"factor": number, "amount": number},
    "volume_discount": {"rate": number, "savings": number},
    "urgency_premium": {"factor": number, "amount": number},
    "additional_services": [
      {"service": "...", "cost": number}
    ],
    "subtotal_before_vat": number,
    "vat_24_percent": number,
    "total_eur": number
  },
  "pricing_notes": [
    "Base tier: Standard/Premium/Specialty",
    "Volume discount applied: X%",
    "Special handling notes..."
  ],
  "internal_margin_notes": [
    "Estimated material cost: €X",
    "Estimated labor hours: Y",
    "Target margin: Z%"
  ],
  "payment_terms": "Net 30 days from invoice date",
  "quote_valid_until": "30 days from quote date"
}

VALIDATION:
- Flag if final price seems unreasonably high or low
- Flag if margin appears too thin (<15%)
- Flag if project exceeds typical project size (requires special approval)

INTERNAL NOTES TO INCLUDE:
- Material cost estimate (powder cost ≈ €10-15/kg, coverage ≈ 8-10 m²/kg at 75 microns)
- Labor estimate (processing time, setup time)
- Expected profit margin
- Risk factors (complex geometry, tight timeline, new customer)

Use Code Interpreter for all financial calculations to ensure accuracy.
```

**Tools to Enable**: 
- **Code Interpreter** (REQUIRED)

**Output Format**: JSON (detailed cost breakdown)

---

### Agent 6: Quote Generator & Formatter

**Name**: "Professional Quote Formatter"

**Model**: GPT-5

**System Instructions**:
```
You are a professional quote document generator for Linjateras Oy.

YOUR TASK:
Transform the pricing data and project details into a professional, customer-ready quote document.

INPUT:
You receive all {{state}} variables including pricing breakdown, project details, and feasibility assessments.

OUTPUT: Generate a complete quote document in professional format.

QUOTE DOCUMENT STRUCTURE:

---
**POWDER COATING QUOTATION**

**Linjateras Oy**
Sileesuonkatu 38, 33330 Tampere, Finland
Tel: +358 3 342 4600
Email: linjateras@linjateras.fi
VAT: FI[number]

**Quote Number:** LT-2025-[auto-generated]
**Date:** [current date]
**Valid Until:** [date + 30 days]

---

**CUSTOMER INFORMATION:**
Company: [customer name]
Contact: [contact person]
Reference: [customer reference]

---

**PROJECT DESCRIPTION:**
[Brief description of parts and coating requirements]

**Scope of Work:**
- Quantity: [X] parts
- Material: [material type]
- Coating Type: [powder coating specification]
- Color: [RAL code or description]
- Finish: [glossy/matte/textured]
- Surface Preparation: [included services]

**Technical Specifications:**
- Part Dimensions: [L × W × H or Dia × Length] mm
- Total Surface Area: [X.X] m²
- Pre-Treatment: 7-stage automated washing system (65m)
- Coating Process: Automated electrostatic powder application
- Curing: 30m oven, 160-220°C, 20-30 minutes

**Quality Assurance:**
- ISO 9001:2015 certified quality management
- ISO 14001:2015 environmental standards
- [X] measurement points quality control
- Real-time production tracking via customer portal
- Video documentation (2-month retention)

---

**PRICING:**

| Description | Quantity/Area | Rate | Amount (EUR) |
|-------------|---------------|------|--------------|
| Powder Coating - [Specification] | [X.X] m² | €[X]/m² | €[X,XXX.XX] |
| [Surface Preparation - if applicable] | [X.X] m² | €[X]/m² | €[XXX.XX] |
| [Color Setup Fee - if applicable] | 1 | - | €[XX.XX] |
| [Additional Services - if applicable] | - | - | €[XXX.XX] |
| **Subtotal (excl. VAT)** | | | **€[X,XXX.XX]** |
| **VAT 24%** | | | **€[XXX.XX]** |
| **TOTAL (incl. VAT)** | | | **€[X,XXX.XX]** |

**Price Notes:**
[Include any relevant pricing notes: volume discount applied, special handling fees, etc.]

---

**DELIVERY & LEAD TIME:**
- Standard Turnaround: [5 business days / 3 days rush / as specified]
- Delivery Method: [Customer pickup / Delivery service]
- Drop-off Hours: Monday-Friday, 07:00-15:30

[If special handling required:]
**Special Handling Note:** Due to extended length ([X]mm), this project requires special handling procedures.

---

**TERMS & CONDITIONS:**
- Payment Terms: Net 30 days from invoice date
- Quote Validity: 30 days from quote date
- Setup charges apply to first production run; repeats may qualify for reduced setup
- Standard quality package included (visual inspection, adhesion testing, thickness measurement)
- 1-5 day turnaround assumes normal production schedule; confirmed timeline provided upon order confirmation
- Powder coating does not correct surface imperfections; parts must be properly prepared and finished

**Certifications:**
- ISO 9001:2015 (Quality Management)
- ISO 14001:2015 (Environmental Management)
- ISO 45001 (Occupational Health & Safety)
- Avainlippu (Key Flag Symbol) - Finnish Quality Mark

---

**ADDITIONAL INFORMATION:**
For questions or to proceed with this quote, please contact:

**Sales Team:**
- Ari Kesti, Sales Director: ari.kesti@linjateras.fi, +358 40 586 7964
- Lauri Suontausta, Account Manager: lauri.suontausta@linjateras.fi, +358 44 579 4377
- Emmi Seppänen, Sales Coordinator: emmi.seppanen@linjateras.fi, +358 40 123 9612

We look forward to serving your powder coating needs with our state-of-the-art equipment and 60 years of expertise.

**Linjateras Oy - Globally Local Powder Coating Excellence**

---

**INTERNAL NOTES (DO NOT INCLUDE IN CUSTOMER DOCUMENT):**

**Cost Analysis:**
- Material cost estimate: €[X]
- Labor hours estimate: [X] hours
- Setup time: [X] hours
- Target margin: [X]%
- Actual margin: [X]%

**Risk Assessment:**
[Flag any concerns: tight timeline, complex geometry, new customer, unusual requirements]

**Production Notes:**
- Estimated production slot: [date range]
- Color batch: [standard/custom]
- Special equipment needed: [none/special fixturing/extended rack]
- Pre-production meeting recommended: [yes/no]

**Follow-up Actions:**
- Schedule follow-up call: [date]
- Technical clarifications needed: [list any]
- Samples requested: [yes/no]

---

FORMAT OUTPUT AS:
1. CUSTOMER-FACING QUOTE (clean, professional, no internal notes)
2. INTERNAL NOTES DOCUMENT (cost analysis, risks, production notes)

Use professional formatting, clear tables, and comprehensive information. The quote should be print-ready and suitable for direct customer delivery.
```

**Tools to Enable**: None

**Output Format**: Formatted text document (Markdown or HTML)

---

## 4. Guardrails Configuration

### Available Guardrail Capabilities in Agent Builder

**What Guardrail Nodes CAN Do:**

1. **PII (Personally Identifiable Information)**
   - Automatically detects and redacts sensitive personal data
   - Configuration: Toggle on/off
   - Use case: Protect customer privacy in RFP submissions

2. **Moderation**
   - Classifies and blocks harmful content
   - Configuration: Select threshold (Most Critical, High, Medium, Low)
   - Categories: hate, violence, self-harm, sexual, harassment
   - Use case: Filter inappropriate submissions

3. **Jailbreak Detection**
   - Detects and blocks prompt injection attempts
   - Prevents manipulation of agent behavior
   - Configuration: Toggle on/off
   - Use case: Security protection against malicious users

4. **Hallucination Detection**
   - Validates agent responses against knowledge sources
   - Requires: Vector Store ID (for RAG validation)
   - Configuration: Toggle on + provide vector store
   - Use case: Verify extracted data accuracy

**What Guardrail Nodes CANNOT Do:**
- File type validation (PDF vs PNG vs JPEG)
- File size limits
- Corrupt file detection
- Format verification
- Business logic validation

---

### Input Guardrail Configuration

**Purpose**: Content safety validation before processing

**Setup in Agent Builder:**

```
Guardrail Node (place after Start node):
├─ Input: user_message or state.rfp_document
├─ PII Detection: Toggle ON
│  └─ Redacts: emails, phone numbers, SSNs, addresses
├─ Moderation: Toggle ON
│  └─ Threshold: "Most Critical"
│  └─ Blocks: hate, violence, harassment, explicit content
├─ Jailbreak Detection: Toggle ON
│  └─ Prevents: prompt injection, system instruction extraction
├─ Hallucination: Leave OFF (no vector store yet)
└─ Continue on Error: OFF (halt if guardrail fails)

Success Path → Agent 1 (Requirements Extractor)
Failure Path → End Node with error message
```

**Error Message for Failed Guardrail:**
```
"Your submission could not be processed due to content policy restrictions. 
Please ensure your RFP contains only appropriate business content and try 
again, or contact our sales team directly at +358 3 342 4600."
```

---

### Alternative Solutions for File Validation

Since Guardrail nodes cannot validate file types or formats, use these approaches:

**Option 1: Pre-Processing Layer (Recommended)**
- Validate file type, size, and format in your application before sending to Agent Builder
- Reject invalid files at upload stage
- Only pass valid PDFs/images to the workflow

**Option 2: Agent 1 Validation Instructions**
Add to Agent 1's instructions:
```
"FIRST, assess the uploaded file:
1. Can you read/see the content clearly?
2. Does it appear to be an RFP, technical drawing, or business document?
3. Is the content coherent and relevant to powder coating services?

If NO to any question, set 'file_validation_failed': true in your output.

Only proceed with extraction if the file passes all checks."
```

**Option 3: MCP Integration**
- Create custom MCP server for file validation
- Perform advanced checks (file headers, corruption detection, security scanning)
- Use MCP node in workflow before Agent 1

---

### Mid-Process Validation: Hallucination Check

**Purpose**: Verify extracted dimensions against source documents

**Setup in Agent Builder:**

```
Guardrail Node (place after Agent 2):
├─ Input: {{agent_2.output}}
├─ Hallucination Detection: Toggle ON
│  └─ Vector Store ID: [created from uploaded technical drawings]
│  └─ Validates: Extracted dimensions exist in source documents
├─ Other guardrails: OFF (not needed here)
└─ Continue on Error: OFF

Success Path → Agent 3 (Dimensional Validation)
Failure Path → Flag for human review:
  "Low confidence in dimension extraction. Human review required."
```

**When to Use:**
- High-value projects (>€5,000)
- Complex drawings with many dimensions
- Customer explicitly requests verification

---

### Output Quality Validation

**Purpose**: Ensure quote completeness before delivery

**Implementation**: Use a validation Agent (not a Guardrail node)

```
Agent Name: "Quote Quality Checker"
Model: GPT-5-mini (fast, cost-effective)

Instructions:
"Review the generated quote for quality and completeness.

REQUIRED CHECKS:
✓ All required fields populated (customer name, total cost, surface area, delivery timeline)
✓ No placeholder text ([X], [TBD], TODO, etc.)
✓ Pricing within reasonable ranges (€20-100/m²)
✓ Mathematical accuracy:
  - Line items sum to subtotal
  - VAT = subtotal × 0.24
  - Total = subtotal + VAT
✓ Contact information is accurate
✓ Professional formatting (no typos, proper grammar)
✓ All sections present (header, pricing, terms, contact info)

OUTPUT (JSON):
{
  'validation_passed': true/false,
  'issues_found': [...],
  'confidence_score': 0.0-1.0,
  'recommendation': 'APPROVE|REVIEW|REJECT'
}

If validation fails, list specific issues requiring correction."

Connect to If/Else node:
- If validation_passed = true → Continue to delivery
- If validation_passed = false → Return to Agent 6 or flag for human review
```

---

## 5. Available Tools in Agent Builder

**Built-in Tools You Can Enable:**

1. **Web Search**
   - Real-time internet search
   - Use case: Current powder coating prices, material costs

2. **File Search**
   - Query OpenAI Vector Stores (RAG)
   - Use case: Search across multiple technical drawings, company knowledge base

3. **Code Interpreter**
   - Execute Python code in sandbox
   - Use case: Surface area calculations, complex pricing logic

4. **Image Generation**
   - Create images using gpt-image-1
   - Use case: Visual mockups of coated parts (optional)

5. **MCP (Model Context Protocol)**
   - Connect to external services via MCP servers
   - Use case: CRM integration, ERP connectivity, custom file validation

6. **Computer Use**
   - Browser automation (advanced)
   - Use case: Not needed for this workflow

**Note on Image/Document Analysis:**
- GPT-5 and GPT-4o have **native vision capabilities**
- They can see and analyze images/PDFs directly when provided as input
- No separate "Vision API tool" exists in Agent Builder
- Simply pass images/PDFs to agents - they will analyze them automatically

---

## 6. Implementation Checklist

### Phase 1: Agent Builder Setup (Week 1)

- [ ] Create new workflow in Agent Builder
- [ ] Add Start Node with file upload capability
- [ ] Configure State variables for tracking data
- [ ] Add Note nodes to document workflow logic

### Phase 2: Build Agents (Week 2-3)

- [ ] Agent 1: RFP Requirements Extractor
  - [ ] Configure instructions
  - [ ] Enable File Search tool
  - [ ] Test with sample RFPs (text and scanned)
  - [ ] Verify JSON output structure

- [ ] Agent 2: Technical Drawing Analyzer
  - [ ] Configure instructions
  - [ ] Enable Code Interpreter tool
  - [ ] Test with various drawing types
  - [ ] Verify dimension extraction from images
  - [ ] Validate surface area calculations

- [ ] Agent 3: Dimensional Validator
  - [ ] Configure feasibility logic
  - [ ] Test with various dimensions
  - [ ] Verify rejection messages

- [ ] Agent 4: Surface Area Calculator
  - [ ] Configure calculation logic
  - [ ] Enable Code Interpreter
  - [ ] Test with different quantities

- [ ] Agent 5: Cost Estimator
  - [ ] Configure pricing matrix
  - [ ] Enable Code Interpreter
  - [ ] Test pricing scenarios
  - [ ] Validate adjustments and discounts

- [ ] Agent 6: Quote Generator
  - [ ] Configure quote template
  - [ ] Test output formatting
  - [ ] Verify internal notes separation

### Phase 3: Control Flow (Week 3-4)

- [ ] Add If/Else nodes for routing
- [ ] Add Transform nodes for data formatting
- [ ] Connect all nodes with proper flow
- [ ] Test end-to-end with sample RFPs

### Phase 4: Guardrails & Validation (Week 4)

- [ ] Add Input Guardrail (PII, Moderation, Jailbreak)
- [ ] Create Vector Store for technical drawings
- [ ] Add Hallucination Guardrail after Agent 2
- [ ] Create Quality Validation Agent
- [ ] Add User Approval node for high-value projects
- [ ] Test all failure paths

### Phase 5: Testing (Week 5-6)

- [ ] Test with 20+ diverse RFPs
- [ ] Measure accuracy metrics
- [ ] Refine agent instructions based on failures
- [ ] Optimize confidence thresholds

### Phase 6: Deployment (Week 7-8)

- [ ] Export to code (Agents SDK) if needed
- [ ] Integrate with Linjateras systems
- [ ] Train sales team
- [ ] Soft launch with oversight
- [ ] Full production deployment

---

## 7. Pricing Calculation Logic

### Base Pricing Matrix (EUR per m²)

```python
BASE_RATES = {
    "standard": {"steel": 40, "aluminum": 40, "galvanized": 46},
    "premium": {"steel": 57.50, "aluminum": 57.50, "galvanized": 66.13},
    "specialty": {"steel": 80, "aluminum": 80, "galvanized": 92}
}

SURFACE_PREP = {
    "none": 0,
    "light_cleaning": 0,
    "sandblasting": 25,
    "heavy_rust_removal": 40,
    "paint_stripping": 30
}

COMPLEXITY = {
    "simple_flat": 1.0,
    "moderate": 1.15,
    "complex": 1.30
}

COLOR = {
    "standard_ral": 1.0,
    "custom_match": 1.20,
    "multi_color_2_3": 1.30,
    "multi_color_4_plus": 1.40
}

COLOR_SETUP = {
    "standard_ral": 0,
    "custom_match": 75,
    "color_change_fee": 50
}

VOLUME_DISCOUNTS = {
    "0-10": 0.0,
    "10-100": 0.0,
    "100-500": 0.10,
    "500-1000": 0.20,
    "1000+": 0.25
}

URGENCY = {
    "standard_5_days": 1.0,
    "fast_3_days": 1.25,
    "rush_1_2_days": 1.50
}

MINIMUM_CHARGE = 500  # EUR
VAT_RATE = 0.24  # 24% for Finnish customers
```

---

## 8. Success Metrics & KPIs

### Target Metrics

**Efficiency:**
- Quote generation time: <5 minutes (vs 30-60 min manual)
- Automation rate: 80% (20% requiring review)
- Quote accuracy: 95%+ (±10% of actual costs)

**Quality:**
- Dimension extraction accuracy: >95%
- Surface area calculation accuracy: ±5%
- Quote completeness: 100%

**Business Impact:**
- Quote volume capacity: 3-5× increase
- Customer response time: <4 hours
- Sales team time savings: 70%+
- Quote-to-order conversion improvement

---

## Conclusion

This implementation guide provides everything needed to build an intelligent RFP analysis system for Linjateras using OpenAI's Agent Builder platform.

**Key Technical Corrections:**
- ✅ GPT-5/GPT-4o have **native vision capabilities** - no separate vision tool needed
- ✅ Guardrail nodes handle **safety only** (PII, moderation, jailbreak, hallucination)
- ✅ File validation must be done via **pre-processing or agent instructions**
- ✅ Available tools: Web Search, File Search, Code Interpreter, Image Generation, MCP, Computer Use

**Expected Outcomes:**
- 3-5× faster quote generation
- 80% automation rate with 20% human oversight
- Consistent, market-based pricing
- Reduced errors through systematic validation
- Sales team focus on relationships vs. administrative work

**Timeline:** 8 weeks from start to production deployment

This system will position Linjateras to respond faster to RFPs, quote more competitively, and scale without proportional increases in support staff.
