# Linjateras Implementation Plan - Two-Phase Rollout

## Overview

**Phase 1**: Prove it works (2 days)  
**Phase 2**: Production system (1-2 weeks)  

---

## Phase 1: Proof of Concept (SIMPLE)

### Goal
Upload drawing image → AI extracts dimensions → validates → quotes → responds in chat

### Scope
- ✅ n8n Chat interface only
- ✅ Single AI Agent with 3 tools
- ✅ Vision-capable model (Claude 3.5 Sonnet or GPT-4o)
- ✅ 5 nodes total
- ❌ NO email, Slack, database, approvals, multi-agents

### Workflow
```
Chat Trigger → Edit Fields → AI Agent → Code → Respond to Chat
```

### AI Agent Configuration
**Model**: Claude 3.5 Sonnet (`claude-3-5-sonnet-20241022`)  
**Why**: Best tool calling, good vision, cost-effective  
**Alternative**: GPT-4o if vision accuracy is better  

**Tools** (n8n Code tool):
1. `validate_dimensions(width, height, length)` - Check equipment limits
2. `calculate_surface_area(L, W, H)` - Calculate m²
3. `estimate_price(area, material, urgency)` - Calculate quote

### Success Criteria
- ✅ Extracts dimensions from drawing (±5% accuracy)
- ✅ Validates constraints (100% accuracy)
- ✅ Calculates area (±10%)
- ✅ Returns price estimate
- ✅ < 10 second response

### Cost
~$1-10 for 100 test runs (basically free)

### Output File
`PHASE1_PROOF_OF_CONCEPT.md` - Complete implementation guide

---

## Phase 2: Production System (AFTER Phase 1 works)

### Additions
1. **RFP Text Extraction** - Handle PDF documents without drawings
2. **Multiple Geometry** - Cylinders, I-beams, tubes (not just boxes)
3. **Professional Quote Generator** - Full formatted document
4. **Persistence** - Save to Supabase/Postgres
5. **Email Delivery** - Gmail integration
6. **Slack Notifications** - Sales team alerts
7. **Approval Flow** - Human review for >€5k quotes
8. **Error Handling** - Retries, fallbacks, logging

### Additional Agents
- Quote Document Writer (Claude 3.5 Sonnet)
- Quality Validator (GPT-4o-mini)
- Rejection Letter Writer (Claude 3.5 Sonnet)

### Cost
€50-200/month for 200-300 quotes

### Timeline
1-2 weeks incremental build

---

## Model Selection Strategy

### For Phase 1 Testing

**Start with**: Claude 3.5 Sonnet
- Reason: Best tool calling
- Cost: Lower
- Speed: Fast

**If dimension extraction poor**: Test GPT-4o
- Reason: Better vision for drawings
- Cost: Slightly higher
- Speed: Fast

**Comparison test** (run both on same 10 drawings):
- Which extracts dimensions more accurately?
- Which follows tool calling better?
- Which is faster/cheaper?
- Pick winner for Phase 2

### For Phase 2 Production

**Drawing Analysis**: Winner from Phase 1 testing  
**Quote Writing**: Claude 3.5 Sonnet (excellent at structured writing)  
**Validation**: GPT-4o-mini (fast, cheap, accurate for simple checks)  

---

## Implementation Checklist

### Phase 1 (This Week)
- [ ] Day 1: Build 5-node workflow
- [ ] Day 1: Configure AI Agent with Claude 3.5 Sonnet
- [ ] Day 1: Create 3 Code tools
- [ ] Day 1: Test with 5 sample drawings
- [ ] Day 2: Test with 20 more drawings
- [ ] Day 2: Compare GPT-4o if needed
- [ ] Day 2: Document results and accuracy
- [ ] **Decision**: Proceed to Phase 2 if >85% accurate

### Phase 2 (Next 1-2 Weeks)
- [ ] Week 1: Add RFP text extraction
- [ ] Week 1: Add multiple geometry support
- [ ] Week 1: Add quote document generator
- [ ] Week 1: Add database persistence
- [ ] Week 2: Add email delivery
- [ ] Week 2: Add Slack notifications
- [ ] Week 2: Add approval flow
- [ ] Week 2: Add error handling
- [ ] Week 2: Production deployment

---

## Files in This Project

```
projects/linjateras/
├── IMPLEMENTATION_PLAN.md              ← This file (overview)
├── PHASE1_PROOF_OF_CONCEPT.md          ← Phase 1 guide (USE THIS FIRST)
├── PHASE2_PRODUCTION_SYSTEM.md         ← Phase 2 guide (after Phase 1 works)
├── linjateras_agent_guide_corrected_openai.md  (OpenAI Agents comparison)
└── linjateras_n8n_ai_agents_workflow.md        (Full complex version - reference only)
```

---

## Key Simplifications for Phase 1

1. **No bullshit about "effectiveness"** - we TEST it to see if it works
2. **Chat only** - fastest feedback, no email/Slack complexity
3. **3 simple tools** - validate, calculate, estimate (that's it)
4. **1 AI Agent** - not 7 agents, just one that does everything
5. **Prove it works** - then we add features

---

## Decision Points

### After Phase 1
**If accuracy >85%**: Proceed to Phase 2  
**If accuracy 70-85%**: Improve prompts, test other models  
**If accuracy <70%**: Rethink approach or add human-in-loop  

### Model Selection
**Test both** Claude 3.5 Sonnet and GPT-4o in Phase 1  
**Pick winner** based on actual results, not marketing claims  

---

**Next Step**: Read `PHASE1_PROOF_OF_CONCEPT.md` and build the 5-node workflow!

