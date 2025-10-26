# Building a Linjateras Powder Coating RFP Analysis Workflow in n8n

## Executive Summary

This guide provides a complete implementation blueprint for creating an intelligent n8n workflow using **AI Agent nodes, OpenRouter integration, and n8n tools** to analyze painting RFPs, extract technical requirements from drawings, calculate quotes, and generate professional proposals for Linjateras Oy, Finland's leading powder coating specialist.

**Critical note**: Linjateras exclusively provides powder coating services, not wet painting, with specific equipment constraints that must be validated for every quote.

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

## 2. n8n AI Agent Workflow Architecture

### Overall Workflow Structure

```
START (RFP Document Upload)
    ↓
[n8n Form Trigger / Webhook]
    ↓
[Edit Fields: Prepare Input Data]
    ↓
[AI Agent: RFP Requirements Extractor]
  Tools: Code Interpreter, File Search
    ↓
[Switch: Document Type & Drawings Available?]
    ├─ Has Drawings → [AI Agent: Technical Drawing Analyzer]
    │                   Tools: Code Interpreter, Vision
    │                     ↓
    │                [Code: Validate Surface Calculations]
    │                     ↓
    ├─ No Drawings  → [Edit Fields: Flag Manual Review]
    │                     ↓
    └─ Invalid Doc  → [Send Email: Request Clarification]
                          ↓
[AI Agent: Dimensional Validator & Feasibility]
  Tools: Code Interpreter
    ↓
[IF: Within Equipment Constraints?]
    ├─ NO  → [AI Agent: Generate Rejection Letter]
    │          ↓
    │       [Send Email: Rejection]
    │          ↓
    │       [END]
    └─ YES → Continue
              ↓
[AI Agent: Pricing Calculator]
  Tools: Code Interpreter, Web Search (current prices)
    ↓
[Code: Apply Business Rules & Minimums]
    ↓
[AI Agent: Professional Quote Generator]
  Tools: None (pure generation)
    ↓
[Code: Validate Quote Quality]
    ↓
[Switch: Quote Value Category]
    ├─ High (>€5k) → [Wait: Human Approval]
    ├─ Medium      → [AI Agent: Quality Check]
    └─ Low         → Auto-approve
                      ↓
[Gmail / Send Email: Deliver Quote]
    ↓
[Postgres / Supabase: Save Quote Record]
    ↓
[Slack: Notify Sales Team]
    ↓
END
```

### n8n Workflow State Management

Use **Edit Fields (Set)** nodes to manage state throughout workflow:

```json
{
  "rfp_id": "LT-{{ $now.format('yyyyMMdd-HHmmss') }}",
  "customer_name": "string",
  "customer_email": "string",
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
  "quantity": "number",
  "urgency": "standard|rush|emergency",
  "base_cost_eur": "number",
  "final_quote_eur": "number",
  "confidence_level": "0.0-1.0",
  "requires_human_review": "boolean",
  "flags": "array"
}
```

---

## 3. Detailed n8n Workflow Components with AI Agent Nodes

### Node 1: RFP Document Submission

**Node Type**: `n8n Form Trigger` or `Webhook`

**Configuration** (if using n8n Form):
- **Form Fields**:
  - Company Name (text, required)
  - Contact Email (email, required)
  - Project Description (textarea)
  - RFP Document Upload (file, accept: .pdf, .png, .jpg, .jpeg)
  - Technical Drawings (file, optional, multiple)
  - Urgency (dropdown: Standard / Fast / Rush)

**Node Type**: `Edit Fields (Set)`

**Purpose**: Prepare data for AI agents

**Fields to Set**:
```javascript
rfp_id = LT-{{ $now.format('yyyyMMdd-HHmmss') }}
customer_name = {{ $json.company_name }}
customer_email = {{ $json.contact_email }}
project_description = {{ $json.project_description }}
urgency = {{ $json.urgency }}
has_drawings = {{ $json.technical_drawings ? true : false }}
uploaded_files = {{ $json }}
```

---

### Node 2: AI Agent - RFP Requirements Extractor

**Node Type**: `AI Agent` (OpenRouter / OpenAI integration)

**Model Selection**: 
- Via OpenRouter: `anthropic/claude-3.5-sonnet` or `openai/gpt-4o`
- Direct OpenAI: `gpt-4o`

**System Prompt**:
```
You are an expert RFP analyst for Linjateras Oy, a powder coating company in Finland.

YOUR TASK:
Extract all key requirements from the customer's RFP document to enable accurate powder coating quote generation.

EXTRACTION REQUIREMENTS:
1. Customer Information:
   - Company name
   - Contact person  
   - Project reference number
   - Email address

2. Project Description:
   - Type of parts (structural steel, aluminum profiles, equipment, etc.)
   - Quantity of parts to be coated
   - Intended application (indoor/outdoor, industry type)

3. Material Specifications:
   - Base material (steel, aluminum, galvanized steel, other)
   - Current condition (new, rusted, painted, galvanized)

4. Coating Requirements:
   - Desired coating type (epoxy, polyester, polyurethane, special)
   - Color requirements (RAL codes, custom colors, number of colors)
   - Finish preference (glossy, semi-gloss, matte, supermatt, textured)
   - Special requirements (UV resistant, chemical resistant, heat resistant, food-safe, anti-bacterial)

5. Dimensional Information:
   - Maximum dimensions mentioned in text (width, height, length in mm)
   - Number of parts
   - Whether technical drawings are included

6. Timeline Requirements:
   - Desired delivery timeframe
   - Is this urgent/rush?

7. Special Services:
   - Need for disassembly/assembly
   - Need for surface preparation (sandblasting, rust removal)
   - Special handling requirements

CRITICAL VALIDATIONS:
- Flag if customer requests "wet painting" or "liquid painting" (Linjateras ONLY does powder coating)
- Flag if material type is unclear or unusual (we primarily coat steel, aluminum, galvanized steel)
- Flag if no dimensional information is provided
- Set confidence score based on completeness of information (0.0-1.0)

OUTPUT FORMAT:
Return a JSON object with this exact structure:

{
  "customer_info": {
    "company": "string",
    "contact": "string", 
    "email": "string",
    "reference": "string"
  },
  "project_description": "string",
  "material_type": "steel|aluminum|galvanized|other",
  "coating_requirements": {
    "type": "epoxy|polyester|polyurethane|special",
    "color": "RAL code or custom",
    "finish": "glossy|semi-gloss|matte|supermatt",
    "special": []
  },
  "has_technical_drawings": true/false,
  "dimensions_from_text": {
    "width_mm": 0,
    "height_mm": 0,
    "length_mm": 0
  },
  "quantity": 0,
  "urgency": "standard|rush|emergency",
  "special_services": [],
  "confidence": 0.0-1.0,
  "missing_critical_info": [],
  "flags": []
}
```

**Tools to Enable**:
- ✅ **Code Interpreter** - For parsing data
- ✅ **File Search** - If multiple documents uploaded
- ✅ **Vision** (automatic with gpt-4o) - For image/PDF analysis

**Input to Agent**:
```javascript
{{ $json.project_description }}

Files attached: {{ $json.uploaded_files }}
```

**Response Mode**: Text (will output JSON)

---

### Node 3: Parse Agent Response

**Node Type**: `Code`

**Purpose**: Extract JSON from AI agent response

**Code**:
```javascript
// Get agent response
const agent_response = $input.first().json.output;

// Parse JSON from response
let extracted_data;
try {
  // Try parsing directly
  extracted_data = JSON.parse(agent_response);
} catch (e) {
  // Extract JSON from markdown code blocks
  const json_match = agent_response.match(/```json\n([\s\S]*?)\n```/) || 
                     agent_response.match(/\{[\s\S]*\}/);
  if (json_match) {
    extracted_data = JSON.parse(json_match[1] || json_match[0]);
  } else {
    throw new Error('Could not extract JSON from agent response');
  }
}

return [extracted_data];
```

---

### Step 3: Decision - Has Technical Drawings?

**Node Type**: IF/ELSE

**Condition**:
```
$node.ExtractRequirements.json.has_technical_drawings === true
```

**If YES**: Continue to Drawing Analysis  
**If NO**: Set flag for manual review

---

### Step 4: Technical Drawing Analysis (If Present)

**Node Type**: HTTP Request (OpenAI Vision API)

**Configuration**:
- **Method**: POST
- **URL**: `https://api.openai.com/v1/chat/completions`

**Body**:
```javascript
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": `You are a manufacturing engineer expert at analyzing technical drawings.

TASK: Extract dimensional information and calculate paintable surface areas.

EXTRACTION PROCESS:
1. Visual Inspection: Examine drawing type, projection, units, scale
2. Title Block: Extract part number, material, scale, revision
3. Dimensions: Extract length, width, height, diameter - ALL critical dimensions
4. Geometry: Identify shape (box, cylinder, I-beam, tube, complex)
5. Surface Area Calculation:

For Rectangular Box: Surface Area = 2(lw + lh + wh)
For Cylinder: Lateral = 2πrh, Total = 2πr(h + r)
For Complex: Decompose into primitives, sum areas, add 10-20% for complexity

OUTPUT (JSON):
{
  "drawing_info": {
    "part_number": "...",
    "part_description": "...",
    "material": "...",
    "units": "mm|inches",
    "scale": "1:1|1:2"
  },
  "dimensions": {
    "length_mm": 0,
    "width_mm": 0,
    "height_mm": 0,
    "geometry_type": "box|cylinder|complex"
  },
  "surface_calculations": {
    "theoretical_area_m2": 0,
    "complexity_factor": 1.0,
    "adjusted_area_m2": 0,
    "calculation_steps": "..."
  },
  "confidence": 0.0-1.0,
  "missing_dimensions": [],
  "requires_clarification": false
}`
    },
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Extract dimensions and calculate surface area from this technical drawing"
        },
        {
          "type": "image_url",
          "image_url": {
            "url": "data:image/jpeg;base64,{{$node.FileUpload.json.file_data}}"
          }
        }
      ]
    }
  ],
  "temperature": 0.2
}
```

---

### Step 5: Parse Dimensions & Calculate Surface Area

**Node Type**: Code

**Code**:
```javascript
// Get drawing analysis from OpenAI
const analysis = $node.DrawingAnalysis.json;
const response_text = analysis.choices[0].message.content;

// Parse JSON response
let drawing_data;
try {
  drawing_data = JSON.parse(response_text);
} catch (e) {
  // If response includes markdown, extract JSON
  const json_match = response_text.match(/\{[\s\S]*\}/);
  if (json_match) {
    drawing_data = JSON.parse(json_match[0]);
  } else {
    throw new Error('Could not parse drawing analysis');
  }
}

// Extract dimensions
const dimensions = drawing_data.dimensions;
const calculations = drawing_data.surface_calculations;

return [{
  dimensions: dimensions,
  surface_area_m2: calculations.adjusted_area_m2,
  complexity_factor: calculations.complexity_factor,
  confidence: drawing_data.confidence,
  flags: drawing_data.requires_clarification ? ['MANUAL_REVIEW_NEEDED'] : []
}];
```

---

### Step 6: Validate Dimensions vs Equipment Constraints

**Node Type**: Code

**Code**:
```javascript
// Equipment constraints
const MAX_WIDTH = 1150;
const MAX_HEIGHT = 2100;
const MAX_LENGTH = 6600;
const MAX_LENGTH_SPECIAL = 9100;

const dims = $node.ParseDimensions.json.dimensions;

let status = 'FEASIBLE';
let violations = [];
let notes = '';

if (dims.width_mm > MAX_WIDTH) {
  violations.push(`Width ${dims.width_mm}mm exceeds limit ${MAX_WIDTH}mm`);
  status = 'INFEASIBLE';
}

if (dims.height_mm > MAX_HEIGHT) {
  violations.push(`Height ${dims.height_mm}mm exceeds limit ${MAX_HEIGHT}mm`);
  status = 'INFEASIBLE';
}

if (dims.length_mm > MAX_LENGTH) {
  if (dims.length_mm <= MAX_LENGTH_SPECIAL) {
    status = 'FEASIBLE_WITH_RESERVATION';
    notes = 'Requires special handling due to extended length';
  } else {
    violations.push(`Length ${dims.length_mm}mm exceeds maximum ${MAX_LENGTH_SPECIAL}mm`);
    status = 'INFEASIBLE';
  }
}

return [{
  feasibility_status: status,
  dimensional_check: {
    width_ok: dims.width_mm <= MAX_WIDTH,
    height_ok: dims.height_mm <= MAX_HEIGHT,
    length_ok: dims.length_mm <= MAX_LENGTH
  },
  violations: violations,
  special_handling_notes: notes,
  passes_check: (status !== 'INFEASIBLE')
}];
```

---

### Step 7: Check if Feasible

**Node Type**: IF/ELSE

**Condition**:
```
$node.ValidateDimensions.json.passes_check === true
```

**If NO**: Send rejection  
**If YES**: Continue to pricing

---

### Step 8: Calculate Pricing

**Node Type**: HTTP Request (OpenAI API)

**Body**:
```javascript
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": `You are a powder coating pricing specialist for Linjateras Oy.

PRICING TIERS (EUR per m²):
- Standard: €35-45/m² (stock colors, normal complexity)
- Premium: €50-65/m² (custom colors, specialty finishes)
- Specialty: €70-90/m² (anti-bacterial, heat-resistant, ESD)

ADJUSTMENT FACTORS:
- Galvanized Steel: ×1.15
- Custom Color: ×1.20 + €75 setup
- Complexity (tubes, angles): ×1.15
- High Complexity (assemblies): ×1.30

VOLUME DISCOUNTS:
- 100-500 m²: 10% discount
- 500-1000 m²: 20% discount
- 1000+ m²: 25% discount

URGENCY PREMIUMS:
- Standard (5 days): ×1.0
- Fast (3 days): ×1.25
- Rush (1-2 days): ×1.50

OUTPUT (JSON):
{
  "base_rate_per_m2": 0,
  "material_adjustment": {"factor": 1.0, "amount": 0},
  "color_adjustment": {"factor": 1.0, "amount": 0},
  "complexity_adjustment": {"factor": 1.0, "amount": 0},
  "volume_discount": {"rate": 0, "savings": 0},
  "urgency_premium": {"factor": 1.0, "amount": 0},
  "subtotal_before_vat": 0,
  "vat_24_percent": 0,
  "total_eur": 0
}`
    },
    {
      "role": "user",
      "content": `Calculate pricing for:
- Surface area: {{$node.ParseDimensions.json.surface_area_m2}} m²
- Material: {{$node.ExtractRequirements.json.material_type}}
- Coating type: {{$node.ExtractRequirements.json.coating_requirements.type}}
- Color: {{$node.ExtractRequirements.json.coating_requirements.color}}
- Quantity: {{$node.ExtractRequirements.json.quantity}}
- Urgency: {{$node.ExtractRequirements.json.urgency}}
- Complexity: {{$node.ParseDimensions.json.complexity_factor}}`
    }
  ],
  "temperature": 0.2
}
```

---

### Step 9: Parse Pricing & Format Quote Data

**Node Type**: Code

**Code**:
```javascript
// Parse pricing response
const pricing_response = $node.CalculatePricing.json;
const pricing_text = pricing_response.choices[0].message.content;

let pricing_data;
try {
  pricing_data = JSON.parse(pricing_text);
} catch (e) {
  const json_match = pricing_text.match(/\{[\s\S]*\}/);
  if (json_match) {
    pricing_data = JSON.parse(json_match[0]);
  } else {
    throw new Error('Could not parse pricing');
  }
}

// Calculate final minimum
let final_total = pricing_data.total_eur;
if (final_total < 500) {
  pricing_data.subtotal_before_vat = 500;
  pricing_data.vat_24_percent = 500 * 0.24;
  final_total = 500 + (500 * 0.24);
  pricing_data.minimum_charge_applied = true;
}

return [{
  pricing: pricing_data,
  final_quote_eur: final_total,
  requires_review: final_total > 5000
}];
```

---

### Step 10: Generate Professional Quote

**Node Type**: HTTP Request (OpenAI API)

**Body**:
```javascript
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": `You are a professional quote document generator for Linjateras Oy.

Generate a complete, professional quote in Markdown format.

REQUIRED SECTIONS:
1. Header (Company info, Quote number, Date, Valid until)
2. Customer Information
3. Project Description & Scope
4. Technical Specifications
5. Quality Assurance
6. Pricing Table
7. Delivery & Lead Time
8. Terms & Conditions
9. Contact Information
10. Internal Notes (separate section)

REQUIRED FIELDS TO INCLUDE:
- All pricing line items and total
- Dimensions and surface area
- Lead time based on urgency
- Special handling notes if applicable
- Company certifications
- Sales contact information`
    },
    {
      "role": "user",
      "content": `Generate a professional quote with this data:
      
Customer: {{$node.ExtractRequirements.json.customer_info.company}}
Project: {{$node.ExtractRequirements.json.project_description}}
Surface Area: {{$node.ParseDimensions.json.surface_area_m2}} m²
Material: {{$node.ExtractRequirements.json.material_type}}
Color: {{$node.ExtractRequirements.json.coating_requirements.color}}
Quantity: {{$node.ExtractRequirements.json.quantity}}

Pricing Details:
{{JSON.stringify($node.ParsePricing.json.pricing, null, 2)}}

Total Quote: €{{$node.ParsePricing.json.final_quote_eur}}

Special Handling: {{$node.ValidateDimensions.json.special_handling_notes}}`
    }
  ],
  "temperature": 0.3
}
```

---

### Step 11: Format & Validate Quote Quality

**Node Type**: Code

**Code**:
```javascript
// Extract quote from response
const quote_response = $node.GenerateQuote.json;
const quote_text = quote_response.choices[0].message.content;

// Validate quote completeness
const required_sections = [
  'Quote Number',
  'Date',
  'Customer',
  'Project',
  'Pricing',
  'Total',
  'Lead Time',
  'Contact'
];

const missing = [];
for (const section of required_sections) {
  if (!quote_text.toLowerCase().includes(section.toLowerCase())) {
    missing.push(section);
  }
}

const validation_passed = missing.length === 0;
const confidence = validation_passed ? 1.0 : 0.7;

return [{
  quote_document: quote_text,
  validation_passed: validation_passed,
  missing_sections: missing,
  confidence: confidence,
  recommendation: validation_passed ? 'APPROVE' : 'REVIEW'
}];
```

---

### Step 12: High-Value Project Check

**Node Type**: IF/ELSE

**Condition**:
```
$node.ParsePricing.json.final_quote_eur > 5000
```

**If YES**: Send to manual approval  
**If NO**: Auto-approve and send

---

### Step 13: Manual Approval (If High-Value)

**Node Type**: Wait For Webhook / Manual Approval

**Configuration**:
- Set timeout to 24 hours
- Require approval before continuing
- Link to sales team for review

---

### Step 14: Send Quote to Customer

**Node Type**: Gmail Send Email

**Configuration**:
- **To**: {{$node.ExtractRequirements.json.customer_info.email}}
- **Subject**: Linjateras Powder Coating Quote - {{$node.GenerateQuote.json.quote_number}}
- **Body**: {{$node.FormatQuote.json.quote_document}}
- **Attachments**: Quote as PDF (optional)

---

### Step 15: Save Quote Record

**Node Type**: Database Insert (PostgreSQL / MySQL)

**Query**:
```sql
INSERT INTO quotes (
  quote_number,
  customer_name,
  customer_email,
  project_description,
  surface_area_m2,
  material_type,
  total_quote_eur,
  urgency,
  created_at,
  quote_document,
  internal_notes
) VALUES (
  $1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11
)
```

**Parameters**:
```javascript
[
  `LT-2025-${Date.now()}`,
  $node.ExtractRequirements.json.customer_info.company,
  $node.ExtractRequirements.json.customer_info.email,
  $node.ExtractRequirements.json.project_description,
  $node.ParseDimensions.json.surface_area_m2,
  $node.ExtractRequirements.json.material_type,
  $node.ParsePricing.json.final_quote_eur,
  $node.ExtractRequirements.json.urgency,
  new Date().toISOString(),
  $node.GenerateQuote.json.quote_text,
  $node.ParsePricing.json.pricing
]
```

---

## 4. Error Handling Strategy

### Node-Level Error Handling

**Implement for each HTTP Request node:**

1. **Add Error Branch** to HTTP Request
2. **Catch Blocks**:

```javascript
// If OpenAI API fails
if (error.response?.status === 401) {
  // Invalid API key
  return [{
    error: 'Authentication failed',
    action: 'Check OPENAI_API_KEY environment variable'
  }];
}

if (error.response?.status === 429) {
  // Rate limited
  return [{
    error: 'Rate limited - retry after 60 seconds',
    retry_after: 60
  }];
}

if (error.response?.status === 500) {
  // Server error
  return [{
    error: 'OpenAI API error - please retry',
    should_retry: true
  }];
}
```

### Workflow-Level Error Handling

**Add Error Handler Node** at end:

```javascript
// Catches all unhandled errors
const error_message = $error.message;
const workflow_execution = $execution;

return [{
  error_logged: true,
  error_message: error_message,
  execution_id: workflow_execution.id,
  timestamp: new Date().toISOString(),
  notify_support: true
}];
```

---

## 5. Environment Variables

**Create in n8n Credentials:**

```bash
OPENAI_API_KEY=sk-xxx...
OPENAI_ORGANIZATION_ID=org-xxx...
LINJATERAS_EMAIL=quotes@linjateras.fi
GMAIL_APP_PASSWORD=xxx...
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=linjateras_db
DATABASE_USER=n8n_user
DATABASE_PASSWORD=xxx...
```

---

## 6. n8n Workflow Testing Checklist

### Phase 1: Individual Node Testing

- [ ] File Upload node - test with PDF and images
- [ ] Extract Requirements node - verify JSON output
- [ ] Drawing Analysis node - test dimension extraction
- [ ] Dimension Validator node - test feasible/infeasible cases
- [ ] Pricing Calculator node - verify cost calculations
- [ ] Quote Generator node - check formatting

### Phase 2: Flow Testing

- [ ] Happy path (valid RFP with drawings) → complete quote
- [ ] RFP without drawings → flag for manual review
- [ ] Oversized parts → rejection email
- [ ] High-value projects → approval wait
- [ ] API failures → proper error handling

### Phase 3: Data Quality Testing

- [ ] Surface area accuracy ±5%
- [ ] Pricing consistency across scenarios
- [ ] Quote document completeness 100%
- [ ] Email delivery success rate 100%

### Phase 4: Performance Testing

- [ ] End-to-end time < 5 minutes
- [ ] Concurrent execution (5+ workflows)
- [ ] API rate limit compliance

---

## 7. Deployment Steps

### 1. Create n8n Workflow

- Start with blank workflow
- Add nodes according to architecture
- Configure credentials
- Set environment variables

### 2. Test with Sample RFPs

- Download 5-10 sample RFPs
- Test workflow end-to-end
- Validate output accuracy
- Refine as needed

### 3. Set Up Webhook

- Enable webhook trigger
- Create public URL
- Share with Linjateras team
- Test submission

### 4. Configure Email Integration

- Connect Gmail credentials
- Test email sending
- Configure templates

### 5. Database Setup

- Create quotes table
- Set up credentials
- Test data insertion

### 6. Production Deployment

- Activate workflow
- Monitor execution logs
- Set up alerts
- Train sales team

---

## 8. Key Differences: n8n vs OpenAI Agents

| Feature | OpenAI Agents | n8n Workflow |
|---------|--------------|-------------|
| **Setup** | Agent Builder UI | Drag-drop canvas |
| **Vision** | Native (no tool needed) | HTTP → OpenAI API |
| **State** | Automatic | Manual Set nodes |
| **Pricing** | Per token + base fee | Per execution |
| **Error Handling** | Built-in guardrails | Code + branches |
| **Database** | Manual SQL nodes | Native integrations |
| **Email** | No native support | Gmail integration |
| **Scalability** | Works but limited | Excellent |
| **Customization** | Less flexible | Highly flexible |

---

## 9. Cost Estimation

### n8n Costs (Monthly)
- Self-hosted n8n: €0-100 (infrastructure)
- OpenAI API: ~€0.10-0.50 per quote
- Email service: €0-20 (Gmail free with limits)
- Database: €10-50 (PostgreSQL on cloud)

**Total: €20-100/month for 100-200 quotes**

### OpenAI Agents Costs
- Agents API: €15/month base + usage
- Token costs: Similar to n8n
- No additional integrations needed

---

## 10. Monitoring & Optimization

### Key Metrics to Track

```javascript
// Add this to final node
const metrics = {
  workflow_duration_seconds: $execution.stopTime - $execution.startTime,
  api_calls: 4, // Requirements + Drawing + Pricing + Quote
  total_tokens_used: estimate_tokens(),
  quote_total_eur: $node.ParsePricing.json.final_quote_eur,
  required_manual_review: $node.ParsePricing.json.requires_review,
  timestamp: new Date().toISOString()
};

// Save to analytics
```

### Optimization Tips

1. **Cache RFP templates** - avoid re-processing similar requests
2. **Use GPT-3.5-turbo** for simple extraction, GPT-4o for complex analysis
3. **Batch process** during off-hours to avoid rate limits
4. **Set temperature lower** (0.2-0.3) for consistent results

---

## Conclusion

This n8n workflow implementation provides:

✅ **Full automation** of RFP → Quote pipeline  
✅ **Flexible architecture** using HTTP nodes for any LLM  
✅ **Scalable design** for 100+ quotes/month  
✅ **Cost-effective** compared to OpenAI Agents  
✅ **Easy customization** with n8n's visual interface  

**Expected outcomes:**
- 3-5× faster quote generation
- 80% automation rate
- €20-100/month operational cost
- Consistent, market-based pricing

**Timeline to production:** 2-3 weeks with existing n8n instance
