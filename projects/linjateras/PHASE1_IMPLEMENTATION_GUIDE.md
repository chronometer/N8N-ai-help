# Phase 1: Proof of Concept - n8n Chat Quote System

**Goal**: Prove it works - upload drawing → get dimensions + quote  
**Timeline**: 2 days  
**Nodes**: 5 total  
**Interface**: n8n Chat only  

---

## What You're Building

```
User uploads drawing image in chat
    ↓
AI analyzes drawing, extracts dimensions
    ↓
AI validates against equipment limits (1150×2100×6600mm)
    ↓
AI calculates surface area
    ↓
AI estimates price
    ↓
Chat responds: "✅ Feasible | 34m² | €1,360-€1,700 | 5 days"
```

**Time per quote**: 5-10 seconds  
**Accuracy target**: ±5% dimensions, ±10% surface area  

---

## Workflow Structure (5 Nodes)

### Node 1: Chat Trigger

**Type**: `Chat Trigger`

**Settings**:
- ✅ Allow file uploads
- ✅ Supported types: image/png, image/jpeg, application/pdf
- ✅ Public chat URL or embed in webpage

**What users see**:
```
💬 Linjateras Quote Bot

Upload your technical drawing to get an instant quote.

Supported: PNG, JPEG, PDF drawings
Max size: 10MB
```

---

### Node 2: Edit Fields - Set Business Rules

**Type**: `Edit Fields (Set)`

**Purpose**: Inject constants into workflow

**Fields to Set**:
```javascript
// Equipment constraints (Linjateras)
max_width_mm = 1150
max_height_mm = 2100
max_length_mm = 6600
max_length_special_mm = 9100

// Base pricing (EUR per m²)
base_rate_standard = 40
base_rate_premium = 57.5
base_rate_specialty = 80

// Material multipliers
material_factor_steel = 1.0
material_factor_aluminum = 1.0
material_factor_galvanized = 1.15

// Urgency multipliers
urgency_standard = 1.0
urgency_fast = 1.25
urgency_rush = 1.5

// Business rules
minimum_charge_eur = 500
vat_rate = 0.24

// Metadata
company_name = Linjateras Oy
timestamp = {{ $now.toISO() }}
```

---

### Node 3: AI Agent - Drawing Analyzer & Quoter

**Type**: `AI Agent`

**Credential**: Anthropic API (for Claude) or OpenRouter API

**Model**: `claude-3-5-sonnet-20241022`

**Why Claude 3.5 Sonnet**:
- ✅ Best-in-class tool calling (follows function schemas precisely)
- ✅ Good vision capabilities (can read technical drawings)
- ✅ Fast and cost-effective
- ✅ Reliable JSON output

**Alternative to test**: `gpt-4o` (may have better vision for complex drawings)

**System Prompt**:
```
You are a technical drawing analysis assistant for Linjateras Oy, a Finnish powder coating company.

YOUR TASK:
1. Analyze the uploaded technical drawing image
2. Extract key dimensions (width, height, length in mm)
3. Validate dimensions against equipment constraints
4. Calculate paintable surface area
5. Estimate powder coating price

PROCESS:
1. Examine the drawing carefully
2. Read dimension values from the drawing (look for numbers with mm, dimension lines, title block)
3. Identify the geometry (rectangular box, cylinder, etc.)
4. Use the validate_dimensions tool to check if parts fit our equipment
5. Use calculate_surface_area tool to get m²
6. Use estimate_price tool to calculate quote

EQUIPMENT LIMITS (call validate_dimensions to check):
- Max width: 1,150mm
- Max height: 2,100mm
- Max length: 6,600mm

IMPORTANT:
- Extract dimensions carefully (read labels, check units)
- If dimensions unclear, state your confidence level
- If infeasible, explain which dimension(s) exceed limits
- Be concise in your response

OUTPUT FORMAT:
Provide a brief summary like:

"📐 **Dimensions**: [W]×[H]×[L]mm
📊 **Surface Area**: [X]m² per part
💰 **Estimated Quote**: €[X]-€[Y] (incl. VAT)
⏱️ **Lead Time**: [X] days
✅ **Status**: Feasible / ⚠️ Special handling / ❌ Exceeds limits"
```

**User Message**:
```javascript
{{ $json.chatInput }}

[Technical drawing image is attached automatically by Chat Trigger]
```

**Tools Configuration**:

Enable these tools (click checkboxes in AI Agent node):
- ✅ **Code Interpreter** (for calculations)  
- ✅ **Vision** (automatic with Claude 3.5 Sonnet)

**Custom Tools** (define via n8n's tool system):

#### Tool 1: validate_dimensions

**Function Schema**:
```json
{
  "name": "validate_dimensions",
  "description": "Validate part dimensions against Linjateras equipment constraints",
  "parameters": {
    "type": "object",
    "properties": {
      "width_mm": {
        "type": "number",
        "description": "Width in millimeters"
      },
      "height_mm": {
        "type": "number",
        "description": "Height in millimeters"
      },
      "length_mm": {
        "type": "number",
        "description": "Length in millimeters"
      }
    },
    "required": ["width_mm", "height_mm", "length_mm"]
  }
}
```

**Implementation** (Code node):
```javascript
const { width_mm, height_mm, length_mm } = $json;

const MAX_W = $node["Edit Fields"].json.max_width_mm;
const MAX_H = $node["Edit Fields"].json.max_height_mm;
const MAX_L = $node["Edit Fields"].json.max_length_mm;
const MAX_L_SPECIAL = $node["Edit Fields"].json.max_length_special_mm;

let status = 'FEASIBLE';
const violations = [];

if (width_mm > MAX_W) {
  violations.push(`Width ${width_mm}mm exceeds ${MAX_W}mm`);
  status = 'INFEASIBLE';
}

if (height_mm > MAX_H) {
  violations.push(`Height ${height_mm}mm exceeds ${MAX_H}mm`);
  status = 'INFEASIBLE';
}

if (length_mm > MAX_L) {
  if (length_mm <= MAX_L_SPECIAL) {
    status = 'FEASIBLE_WITH_RESERVATION';
  } else {
    violations.push(`Length ${length_mm}mm exceeds maximum ${MAX_L_SPECIAL}mm`);
    status = 'INFEASIBLE';
  }
}

return {
  status: status,
  feasible: status !== 'INFEASIBLE',
  violations: violations,
  special_handling: status === 'FEASIBLE_WITH_RESERVATION',
  message: violations.length > 0 ? violations.join('; ') : 'All dimensions within limits'
};
```

#### Tool 2: calculate_surface_area

**Function Schema**:
```json
{
  "name": "calculate_surface_area",
  "description": "Calculate paintable surface area for rectangular box",
  "parameters": {
    "type": "object",
    "properties": {
      "length_mm": {"type": "number"},
      "width_mm": {"type": "number"},
      "height_mm": {"type": "number"},
      "complexity_factor": {
        "type": "number",
        "description": "Complexity multiplier (1.0 = simple, 1.15 = moderate, 1.3 = complex)",
        "default": 1.0
      }
    },
    "required": ["length_mm", "width_mm", "height_mm"]
  }
}
```

**Implementation**:
```javascript
const { length_mm, width_mm, height_mm, complexity_factor = 1.0 } = $json;

// Convert mm to meters
const L = length_mm / 1000;
const W = width_mm / 1000;
const H = height_mm / 1000;

// Surface area of rectangular box: 2(LW + LH + WH)
const base_area = 2 * (L*W + L*H + W*H);

// Apply complexity factor
const adjusted_area = base_area * complexity_factor;

return {
  length_m: L,
  width_m: W,
  height_m: H,
  base_area_m2: Math.round(base_area * 100) / 100,
  complexity_factor: complexity_factor,
  final_area_m2: Math.round(adjusted_area * 100) / 100,
  formula: '2(L×W + L×H + W×H)',
  calculation: `2(${L.toFixed(2)}×${W.toFixed(2)} + ${L.toFixed(2)}×${H.toFixed(2)} + ${W.toFixed(2)}×${H.toFixed(2)}) = ${base_area.toFixed(2)}m²`
};
```

#### Tool 3: estimate_price

**Function Schema**:
```json
{
  "name": "estimate_price",
  "description": "Estimate powder coating price with adjustments",
  "parameters": {
    "type": "object",
    "properties": {
      "area_m2": {"type": "number", "description": "Surface area in square meters"},
      "material": {
        "type": "string",
        "enum": ["steel", "aluminum", "galvanized"],
        "description": "Base material type"
      },
      "coating_type": {
        "type": "string",
        "enum": ["standard", "premium", "specialty"],
        "default": "standard"
      },
      "urgency": {
        "type": "string",
        "enum": ["standard", "fast", "rush"],
        "default": "standard"
      },
      "quantity": {
        "type": "number",
        "default": 1
      }
    },
    "required": ["area_m2", "material"]
  }
}
```

**Implementation**:
```javascript
const { 
  area_m2, 
  material, 
  coating_type = 'standard', 
  urgency = 'standard',
  quantity = 1 
} = $json;

// Get base rates from Edit Fields
const base_rates = {
  standard: $node["Edit Fields"].json.base_rate_standard,
  premium: $node["Edit Fields"].json.base_rate_premium,
  specialty: $node["Edit Fields"].json.base_rate_specialty
};

// Material factors
const material_factors = {
  steel: 1.0,
  aluminum: 1.0,
  galvanized: 1.15
};

// Urgency factors
const urgency_factors = {
  standard: 1.0,
  fast: 1.25,
  rush: 1.5
};

// Calculate total area
const total_area = area_m2 * quantity;

// Base cost
const base_rate = base_rates[coating_type];
const base_cost = total_area * base_rate;

// Apply adjustments
const material_adjusted = base_cost * material_factors[material];
const final_cost = material_adjusted * urgency_factors[urgency];

// Apply minimum
const subtotal = Math.max(final_cost, 500);
const vat = subtotal * 0.24;
const total = subtotal + vat;

// Price range (±15% for estimate)
const range_low = Math.round(total * 0.85);
const range_high = Math.round(total * 1.15);

return {
  base_rate_per_m2: base_rate,
  total_area_m2: total_area,
  base_cost: Math.round(base_cost),
  material_adjustment: material_factors[material],
  urgency_adjustment: urgency_factors[urgency],
  subtotal_excl_vat: Math.round(subtotal),
  vat_24_percent: Math.round(vat),
  total_incl_vat: Math.round(total),
  minimum_applied: final_cost < 500,
  price_range: `€${range_low}-€${range_high}`,
  lead_time_days: urgency === 'standard' ? 5 : urgency === 'fast' ? 3 : 2
};
```

**Agent Options**:
- Temperature: 0.2
- Max Tokens: 2000

---

### Node 4: Code - Format Chat Response

**Type**: `Code`

**Purpose**: Clean up agent response for chat display

**Code**:
```javascript
// Get agent output
const agent_response = $input.first().json;

// Agent response is already formatted in system prompt
// Just pass through or add minor formatting

const response = agent_response.output || agent_response.text || agent_response.content;

return [{
  chat_response: response,
  processed_at: new Date().toISOString()
}];
```

---

### Node 5: Respond to Chat

**Type**: `Respond to Chat`

**Message**:
```javascript
{{ $json.chat_response }}
```

**Options**:
- Format: Markdown
- Show typing indicator: Yes

---

## Setup Instructions

### Step 1: Create New Workflow

1. Open n8n
2. Create new workflow
3. Name it: "Linjateras Phase 1 - Drawing Quote POC"

### Step 2: Add Nodes

1. Add `Chat Trigger` node
2. Add `Edit Fields (Set)` node
3. Add `AI Agent` node
4. Add `Code` node
5. Add `Respond to Chat` node
6. Connect them in sequence

### Step 3: Configure Chat Trigger

- Enable file uploads
- Set max file size: 10MB
- Test chat URL

### Step 4: Configure Edit Fields

Copy the fields from Node 2 section above

### Step 5: Configure AI Agent

**Get API Key**:
- Claude: https://console.anthropic.com/
- OpenRouter: https://openrouter.ai/ (supports multiple models)

**Add Credential**:
- Settings → Credentials
- Create: Anthropic API or OpenRouter API
- Paste API key

**Select Model**:
- Anthropic direct: `claude-3-5-sonnet-20241022`
- OpenRouter: `anthropic/claude-3.5-sonnet`

**Add System Prompt**: Copy from Node 3 section above

**Add Tools**: Create the 3 Code tools (validate, calculate, estimate)

**Important**: In n8n, tools are defined as separate workflows or Code nodes that the agent can call. Follow n8n's tool definition pattern.

### Step 6: Configure Code + Respond Nodes

Copy code from sections above

### Step 7: Test

1. Click "Execute" or open Chat
2. Upload a technical drawing
3. See if agent extracts dimensions and quotes

---

## Testing Checklist

### Test Case 1: Standard Part (Should Pass)
**Drawing**: Rectangular part 5000×1000×2000mm
**Expected**:
- Dimensions extracted: ✅
- Status: Feasible ✅
- Area: ~34m²
- Quote: €1,360-€1,700
- Lead time: 5 days

### Test Case 2: Large Part (Should Flag)
**Drawing**: Part 8000×1200×2500mm
**Expected**:
- Dimensions extracted: ✅
- Status: Infeasible ❌
- Violations listed: Width, Height, Length all exceed
- No quote given

### Test Case 3: Borderline (Special Handling)
**Drawing**: Part 6500×1100×2000mm
**Expected**:
- Status: Feasible with reservation ⚠️
- Note: "Special handling for extended length"
- Quote provided

### Test Case 4: Small Part (Minimum Charge)
**Drawing**: Part 500×500×500mm
**Expected**:
- Area: ~1.5m²
- Quote: €500 minimum applied
- Note about minimum charge

### Test Case 5: Poor Quality Drawing
**Drawing**: Blurry or hand-sketched drawing
**Expected**:
- Agent states low confidence
- Requests clearer drawing or manual dimensions

---

## Model Comparison Testing

### Test Both Models on Same 10 Drawings

**Claude 3.5 Sonnet**:
```
Drawing 1: Extracted 5000×1000×2000 → Actual: 5000×1000×2000 ✅
Drawing 2: Extracted 3200×800×1500 → Actual: 3150×800×1500 (98% accurate)
...
```

**GPT-4o**:
```
Drawing 1: Extracted 5000×1000×2000 → Actual: 5000×1000×2000 ✅
Drawing 2: Extracted 3150×800×1500 → Actual: 3150×800×1500 ✅
...
```

**Scoring**:
- Dimension accuracy (average % error)
- Tool calling reliability (did it call tools correctly?)
- Response quality (concise, professional?)
- Cost per analysis
- Speed

**Pick winner** for Phase 2 based on results, not hype.

---

## Troubleshooting

### Issue 1: Agent doesn't extract dimensions

**Possible causes**:
- Drawing quality too poor
- No clear dimension labels
- Wrong units (inches vs mm)

**Solutions**:
- Test with clearer drawing
- Add to prompt: "If you cannot read dimensions, ask for clearer image"
- Test GPT-4o instead

### Issue 2: Agent doesn't call tools

**Possible causes**:
- Tool schemas not defined correctly
- Model not following instructions

**Solutions**:
- Verify tool schemas in n8n
- Simplify system prompt
- Add example tool call to prompt

### Issue 3: Wrong calculations

**Possible causes**:
- Tool code has bugs
- Agent passing wrong parameters

**Solutions**:
- Test tool code independently
- Add validation to tool code
- Log tool inputs/outputs

---

## Success Metrics for Phase 1

| Metric | Target | Pass/Fail |
|--------|--------|-----------|
| Dimension extraction accuracy | >90% | Required |
| Constraint validation accuracy | 100% | Required |
| Surface area accuracy | ±10% | Acceptable |
| Price estimate reasonable | ±20% | Acceptable |
| Response time | <10s | Required |
| Tool calling reliability | >95% | Required |
| No crashes on valid input | 100% | Required |

**If metrics met**: Proceed to Phase 2  
**If not**: Refine prompts, test other models, or adjust approach  

---

## Cost Analysis (Phase 1)

**Claude 3.5 Sonnet pricing**:
- Input: $3 per million tokens
- Output: $15 per million tokens
- Typical usage per analysis: ~1,500 tokens (input) + 500 tokens (output)
- **Cost per quote**: $0.01-0.02

**Testing cost (100 quotes)**: $1-2 total

**Phase 1 is essentially FREE to test!**

---

## Next Steps After Phase 1 Works

1. **Document results** - Which model worked better?
2. **Calculate accuracy** - Dimension extraction %, surface area %
3. **Decide on Phase 2** - If >85% accurate, proceed
4. **Plan additions** - RFP text, email, database, etc.

---

## Key Takeaways

✅ **Start simple** - 5 nodes, prove it works  
✅ **Test both models** - Claude 3.5 Sonnet vs GPT-4o  
✅ **No complexity** - Chat only, no integrations  
✅ **Real testing** - 10-20 actual drawings  
✅ **Data-driven decision** - Pick model based on results  
✅ **Fast iteration** - 2 days to working prototype  

**Then and only then, build Phase 2!**

---

**Ready to build?** Open n8n and start with Node 1! 🚀

