# Phase 1: n8n-native Chat Quote Flow (No Code Nodes)

Goal: Minimal, production-style n8n flow to prove end-to-end quoting from a drawing via chat. No JavaScript nodes, no HTTP calls, no external notifiers. n8n-native only.

---

## Flow (5 nodes)

Chat Trigger → Edit Fields → AI Agent (Claude 4.5 via OpenRouter) → AI Structured Output → Respond to Chat

---

## Node 1: Chat Trigger

Type: Chat Trigger

Settings:
- Allow file uploads: ON
- File types: image/png, image/jpeg, application/pdf
- Max size: 10MB
- Optional user fields: quantity (number), material (select: steel/aluminum/galvanized), urgency (select: standard/fast/rush)

---

## Node 2: Edit Fields (constants)

Type: Edit Fields (Set)

Fields:
- max_width_mm = 1150
- max_height_mm = 2100
- max_length_mm = 6600
- base_rate_per_m2 = 40
- minimum_charge_eur = 500
- vat_rate = 0.24

---

## Node 3: AI Agent

Type: AI Agent

Provider: OpenRouter
Model: anthropic/claude-4.5 (or latest Claude 4.5 variant)
Tools: Vision ON, Code Interpreter OFF, Web Search OFF
Temperature: 0.2

System Prompt:
You are a technical drawing analyzer for Linjateras (powder coating only).

Constraints:
- Max width: 1,150 mm
- Max height: 2,100 mm
- Max length: 6,600 mm

Task:
1) Read dimensions (mm) from the drawing.
2) Decide feasibility against constraints.
3) Calculate paintable surface area for box geometry: 2(LW + LH + WH).
4) Estimate a price range (min charge €500, VAT 24%) using base rate from context.
5) Return JSON that strictly matches the schema provided by the next node.
6) If units are not mm, convert to mm and note confidence.

Rules:
- Be deterministic and concise.
- Do not output prose; only produce content designed to satisfy the schema.

User Message:
Analyze the uploaded technical drawing to create a powder coating quote. Use limits and pricing from context. If the geometry is not a rectangular box, approximate conservatively and set a note in the output.

Attachments:
- Forward the uploaded file from Chat Trigger.

Context Values (pass-through):
- Use `max_width_mm`, `max_height_mm`, `max_length_mm`, `base_rate_per_m2`, `minimum_charge_eur`, `vat_rate` from Edit Fields.

---

## Node 4: AI Structured Output

Type: AI Structured Output

Schema:
{
  "type": "object",
  "properties": {
    "dimensions": {
      "type": "object",
      "properties": {
        "width_mm": { "type": "number" },
        "height_mm": { "type": "number" },
        "length_mm": { "type": "number" },
        "confidence": { "type": "number" },
        "units_detected": { "type": "string" }
      },
      "required": ["width_mm", "height_mm", "length_mm"]
    },
    "feasibility": {
      "type": "object",
      "properties": {
        "status": { "type": "string", "enum": ["FEASIBLE", "FEASIBLE_WITH_RESERVATION", "INFEASIBLE"] },
        "violations": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["status", "violations"]
    },
    "surface": {
      "type": "object",
      "properties": {
        "area_m2": { "type": "number" },
        "method": { "type": "string" }
      },
      "required": ["area_m2"]
    },
    "pricing": {
      "type": "object",
      "properties": {
        "base_rate_per_m2": { "type": "number" },
        "subtotal_excl_vat": { "type": "number" },
        "vat_24_percent": { "type": "number" },
        "total_incl_vat": { "type": "number" },
        "price_range": { "type": "string" },
        "minimum_applied": { "type": "boolean" }
      },
      "required": ["total_incl_vat", "price_range"]
    },
    "notes": { "type": "string" }
  },
  "required": ["dimensions", "feasibility", "surface", "pricing"]
}

Validation: Reject and re-ask model on schema mismatch.

---

## Node 5: Respond to Chat

Type: Respond to Chat

Message template (Markdown):
Dimensions: {{$json.dimensions.width_mm}}×{{$json.dimensions.height_mm}}×{{$json.dimensions.length_mm}} mm
Status: {{$json.feasibility.status}}
Surface Area: {{$json.surface.area_m2}} m²
Estimate: {{$json.pricing.price_range}} (incl. VAT)
Notes: {{$json.notes}}

---

## Test Scenarios (no dev code required)

1) Valid part: 5000×1000×2000 → Feasible, ~34 m², ~€1,360–€1,700
2) Borderline length: 6601×1000×2000 → Reservation, note special handling
3) Too large: 7000×1500×2500 → Infeasible, list violations
4) Small part: 500×500×500 → Minimum charge note
5) Blurry drawing: Low confidence + ask for clearer image

Success Criteria:
- Dimensions parsed (>90% of tests)
- Feasibility correct (100% of tests)
- Area approximated (±10%)
- Quote within expected range (±20%)
- <10s response

---

## Model Notes

- Use Claude 4.5 via OpenRouter for best tool-following and concise JSON. If vision extraction underperforms on your drawings, A/B test GPT-4o and select empirically.


