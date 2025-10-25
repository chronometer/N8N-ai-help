# Linjateras n8n Workflow - Phase 1: Proof of Concept

**Goal**: Prove vision + tool calling works. Upload drawing → get dimensions + quote.  
**Timeline**: 2 days  
**Complexity**: Minimal (5 nodes)  

---

## What We're Building

**Input**: Technical drawing image (PNG/JPG)  
**Output**: "Dimensions: 5000×1000×2000mm, Area: 34m², Quote: €1,360-€1,700, Status: ✅ Feasible"  
**Interface**: n8n Chat (no email/Slack/database needed)  

---

## Workflow Architecture (5 Nodes)

```
1. Chat Trigger (user uploads image)
   ↓
2. Edit Fields (set equipment limits)
   ↓
3. AI Agent (analyze drawing + calculate quote)
   ↓
4. Code (format response)
   ↓
5. Respond to Chat (show result)
```

---

## Node Details

### Node 1: Chat Trigger

**Type**: `Chat Trigger`

**Configuration**:
- Enable file uploads
- Allow image types: PNG, JPEG, PDF
- Max file size: 10MB

**What user does**:
1. Opens n8n chat
2. Uploads technical drawing
3. (Optional) Types: "Quote for 50 parts, steel, standard coating"

---

### Node 2: Edit Fields - Set Constraints

**Type**: `Edit Fields (Set)`

**Fields**:
```javascript
// Equipment limits (Linjateras constraints)
max_width_mm = 1150
max_height_mm = 2100
max_length_mm = 6600

// Base pricing (EUR per m²)
base_price_standard = 40
base_price_premium = 57.5
base_price_specialty = 80

// Minimum charge
minimum_charge_eur = 500
```

---

### Node 3: AI Agent - Drawing Analyzer + Quoter

**Type**: `AI Agent`

**Model**: `claude-3-5-sonnet-20241022` (Anthropic) or `gpt-4o` (OpenAI)

**Why Claude 3.5 Sonnet**:
- ✅ Excellent tool calling (best in class)
- ✅ Good vision capabilities
- ✅ Follows JSON structure perfectly
- ✅ Cost-effective

**Why GPT-4o** (alternative):
- ✅ Excellent vision (may be better for complex drawings)
- ✅ Good tool calling
- ✅ Fast

**System Prompt**:
```
You are a powder coating quotation assistant for Linjateras Oy, Finland.

EQUIPMENT LIMITS:
- Max width: 1,150mm
- Max height: 2,100mm  
- Max length: 6,600mm

YOUR TASK:
1. Analyze the technical drawing image
2. Extract dimensions (width, height, length in mm)
3. Validate against equipment limits
4. Calculate surface area
5. Estimate price

Use the tools provided to:
- validate_dimensions() to check feasibility
- calculate_surface_area() to get m²
- estimate_price() to get quote

Be concise and accurate.
```

**User Message**:
```javascript
{{ $json.chatInput || "Analyze this drawing and provide a quote" }}

Image attached: {{ $json.files }}
```

**Tools to Define** (using n8n Code tool):

#### Tool 1: validate_dimensions

**Tool Name**: `validate_dimensions`

**Description**: Check if part dimensions fit in coating equipment

**Parameters**:
```json
{
  "type": "object",
  "properties": {
    "width_mm": {"type": "number", "description": "Width in millimeters"},
    "height_mm": {"type": "number", "description": "Height in millimeters"},
    "length_mm": {"type": "number", "description": "Length in millimeters"}
  },
  "required": ["width_mm", "height_mm", "length_mm"]
}
```

**Code** (JavaScript):
```javascript
const { width_mm, height_mm, length_mm } = $input.item().json;

const MAX_W = 1150;
const MAX_H = 2100;
const MAX_L = 6600;

const status = 
  width_mm > MAX_W || height_mm > MAX_H || length_mm > MAX_L
    ? 'INFEASIBLE'
    : length_mm > MAX_L ? 'FEASIBLE_WITH_RESERVATION' : 'FEASIBLE';

const violations = [];
if (width_mm > MAX_W) violations.push(`Width ${width_mm}mm > ${MAX_W}mm`);
if (height_mm > MAX_H) violations.push(`Height ${height_mm}mm > ${MAX_H}mm`);
if (length_mm > MAX_L) violations.push(`Length ${length_mm}mm > ${MAX_L}mm`);

return {
  status,
  feasible: status !== 'INFEASIBLE',
  violations,
  notes: status === 'FEASIBLE_WITH_RESERVATION' ? 'Special handling needed' : ''
};
```

#### Tool 2: calculate_surface_area

**Tool Name**: `calculate_surface_area`

**Description**: Calculate surface area for rectangular box

**Parameters**:
```json
{
  "type": "object",
  "properties": {
    "length_mm": {"type": "number"},
    "width_mm": {"type": "number"},
    "height_mm": {"type": "number"},
    "complexity_factor": {"type": "number", "description": "1.0-1.5", "default": 1.0}
  },
  "required": ["length_mm", "width_mm", "height_mm"]
}
```

**Code**:
```javascript
const { length_mm, width_mm, height_mm, complexity_factor = 1.0 } = $input.item().json;

// Convert mm to m
const l = length_mm / 1000;
const w = width_mm / 1000;
const h = height_mm / 1000;

// Rectangular box surface area: 2(lw + lh + wh)
const base_area = 2 * (l*w + l*h + w*h);

// Apply complexity factor
const adjusted_area = base_area * complexity_factor;

return {
  base_area_m2: Math.round(base_area * 100) / 100,
  complexity_factor,
  final_area_m2: Math.round(adjusted_area * 100) / 100
};
```

#### Tool 3: estimate_price

**Tool Name**: `estimate_price`

**Description**: Estimate powder coating price

**Parameters**:
```json
{
  "type": "object",
  "properties": {
    "area_m2": {"type": "number"},
    "material": {"type": "string", "enum": ["steel", "aluminum", "galvanized"]},
    "coating_type": {"type": "string", "enum": ["standard", "premium", "specialty"], "default": "standard"},
    "urgency": {"type": "string", "enum": ["standard", "fast", "rush"], "default": "standard"}
  },
  "required": ["area_m2", "material"]
}
```

**Code**:
```javascript
const { area_m2, material, coating_type = 'standard', urgency = 'standard' } = $input.item().json;

// Base rates
const base_rates = { standard: 40, premium: 57.5, specialty: 80 };
const base = base_rates[coating_type];

// Material adjustment
const material_adj = material === 'galvanized' ? 1.15 : 1.0;

// Urgency premium
const urgency_adj = { standard: 1.0, fast: 1.25, rush: 1.5 }[urgency];

// Calculate
const cost = area_m2 * base * material_adj * urgency_adj;
const total = Math.max(cost, 500); // Minimum €500
const with_vat = total * 1.24; // 24% Finnish VAT

return {
  base_rate_per_m2: base,
  subtotal_eur: Math.round(total),
  vat_eur: Math.round(total * 0.24),
  total_eur: Math.round(with_vat),
  minimum_applied: cost < 500,
  price_range: `€${Math.round(total)}-€${Math.round(total * 1.15)}`
};
```

**Agent Options**:
- Temperature: 0.2
- Max tokens: 2000
- Response format: Text (agent will use tools and summarize)

---

### Node 4: Code - Format Response

**Type**: `Code`

**Purpose**: Extract tool results and format for chat

**Code**:
```javascript
const agent_output = $input.first().json;

// Agent response includes tool call results
// Extract the final message
const response_text = agent_output.output || agent_output.text || '';

// Simple formatting for chat
return [{
  chat_response: response_text,
  timestamp: new Date().toISOString()
}];
```

---

### Node 5: Respond to Chat

**Type**: `Respond to Chat`

**Message**:
```javascript
{{ $json.chat_response }}
```

---

## Testing Phase 1

### Test Scenarios

1. **Valid dimensions** (5000×1000×2000mm)
   - Expected: Feasible, ~34m², ~€1,360-€1,564

2. **Borderline** (6500×1000×2000mm)
   - Expected: Special handling note

3. **Too large** (7000×1500×2500mm)
   - Expected: Infeasible, list violations

4. **Tiny part** (500×500×500mm)
   - Expected: Feasible, but minimum €500 charge

5. **Different materials** (aluminum, galvanized)
   - Expected: Correct pricing adjustments

### Success Criteria

- ✅ Extracts dimensions from drawing image correctly (±5% accuracy)
- ✅ Validates against limits correctly (100% accuracy on pass/fail)
- ✅ Calculates surface area correctly (±10% acceptable for Phase 1)
- ✅ Returns price estimate in range
- ✅ Response time < 10 seconds
- ✅ No crashes on bad input

---

## Setup Instructions

### 1. Install n8n (if needed)

```bash
# Via npm
npm install -g n8n
n8n start

# Or Docker
docker run -it --rm -p 5678:5678 n8nio/n8n
```

### 2. Get API Key

**Option A: Claude (Anthropic)**
- Go to https://console.anthropic.com/
- Get API key
- Model: `claude-3-5-sonnet-20241022`

**Option B: GPT-4o (OpenAI)**
- Go to https://platform.openai.com/
- Get API key
- Model: `gpt-4o`

**Option C: OpenRouter (supports both)**
- Go to https://openrouter.ai/
- Get API key
- Use model ID: `anthropic/claude-3.5-sonnet` or `openai/gpt-4o`

### 3. Create Credentials in n8n

- Settings → Credentials → Add Credential
- Type: Anthropic API or OpenAI API or OpenRouter API
- Paste API key

### 4. Build Workflow

Follow the 5-node structure above.

### 5. Test

Upload a technical drawing and see if it works!

---

## Expected Costs (Phase 1)

**Per quote analysis**:
- Claude 3.5 Sonnet: ~$0.01-0.05 (cheap, accurate tool calling)
- GPT-4o: ~$0.03-0.10 (better vision potentially)

**Testing (100 test runs)**: $1-10 total

**Phase 1 is basically free to prove it works!**

---

## What's Next (Phase 2)

Once Phase 1 works reliably:

- Add RFP text analysis agent
- Add cylinder/tube geometry calculations
- Add quote document generator
- Add email delivery
- Add database persistence
- Add Slack notifications
- Add approval workflow for high-value quotes

**But first: prove the core vision + tool calling works!**

---

## Notes on Models

**Claude 3.5 Sonnet** (current latest):
- Released: October 2024
- Version: `claude-3-5-sonnet-20241022`
- Best at: Tool calling, following instructions
- Vision: Good (may not be as strong as GPT-4o for drawings)

**GPT-4o**:
- Best vision model currently
- Excellent at technical drawing analysis
- Good tool calling
- Slightly more expensive

**Recommendation**: 
- **Start with Claude 3.5 Sonnet** for tool calling reliability
- **Test GPT-4o** if dimension extraction accuracy is poor
- **Compare both** in Phase 1 testing

---

**File**: PHASE1_PROOF_OF_CONCEPT.md  
**Status**: Ready to implement  
**Next**: Build it and test!

