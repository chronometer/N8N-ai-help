# Linjateras Powder Coating RFP Automation

Automated quote generation system for Linjateras Oy using n8n AI workflows.

---

## 📁 Files in This Project

### ⭐ START HERE - Phase 1 (Proof of Concept)

**`IMPLEMENTATION_PLAN.md`**
- Overview of 2-phase approach
- Why we split it this way
- Decision points and timeline

**`PHASE1_PROOF_OF_CONCEPT.md`**  
- Quick overview of Phase 1
- High-level architecture
- What to expect

**`PHASE1_IMPLEMENTATION_GUIDE.md`** ← **USE THIS TO BUILD**
- Complete step-by-step build instructions
- All 5 nodes with exact configuration
- Code for all 3 tools (copy-paste ready)
- Testing checklist and success criteria
- Model comparison framework

### 📚 Reference Materials

**`linjateras_agent_guide_corrected_openai.md`**
- OpenAI Agents platform implementation (comparison)
- Original business requirements
- Detailed pricing matrices

**`linjateras_n8n_ai_agents_workflow.md`**
- Full 21-node production workflow (reference)
- Comprehensive but complex
- Use after Phase 1 succeeds

**`linjateras_n8n_workflow_guide.md`** [DEPRECATED]
- Old HTTP Request approach
- Don't use - kept for reference only

---

## 🚀 Quick Start

### For Immediate Implementation

1. Read `IMPLEMENTATION_PLAN.md` (5 min)
2. Follow `PHASE1_IMPLEMENTATION_GUIDE.md` step-by-step
3. Build 5-node workflow in n8n
4. Test with 10-20 drawings
5. Compare Claude 3.5 Sonnet vs GPT-4o
6. Pick winner based on accuracy

**Timeline**: 2 days to working prototype

### What You Need

- n8n instance (Cloud or self-hosted)
- API key: Anthropic (Claude) or OpenAI (GPT-4o) or OpenRouter (both)
- 10-20 sample technical drawings for testing
- 2 days of focused development time

---

## 📊 Phase 1 vs Phase 2

| Feature | Phase 1 (POC) | Phase 2 (Production) |
|---------|---------------|---------------------|
| **Interface** | n8n Chat only | Webhook + Form + Chat |
| **Nodes** | 5 nodes | 20+ nodes |
| **AI Agents** | 1 agent | 5-7 agents |
| **Tools** | 3 simple tools | 10+ tools |
| **Integrations** | None | Email, Slack, DB |
| **Complexity** | Minimal | Full featured |
| **Timeline** | 2 days | 1-2 weeks |
| **Cost** | ~$1-2 testing | €50-200/month |
| **Goal** | Prove it works | Production ready |

---

## ✅ Success Criteria

### Phase 1 Must Achieve

- ✅ Extract dimensions from drawings (>90% accuracy)
- ✅ Validate constraints (100% accuracy)
- ✅ Calculate surface area (±10%)
- ✅ Estimate price (±20%)
- ✅ Response time <10 seconds
- ✅ No crashes on valid inputs

### If Phase 1 Succeeds

→ Proceed to Phase 2 implementation  
→ Add RFP handling, integrations, persistence  
→ Production deployment in 1-2 weeks  

### If Phase 1 Fails (<85% accuracy)

→ Test different models  
→ Refine prompts  
→ Consider hybrid approach (AI + manual review)  

---

## 🔧 Current Status

**Phase 1**: Implementation guide complete ✅  
**Phase 2**: Plan ready, build after Phase 1 validated  

**Next Action**: Build Phase 1 workflow and test!

---

## 💡 Model Recommendations

**For Tool Calling**:
1. Claude 3.5 Sonnet (best)
2. GPT-4o (good)
3. GPT-4o-mini (fast but less capable)

**For Vision (Technical Drawings)**:
1. GPT-4o (potentially best for CAD/technical)
2. Claude 3.5 Sonnet (good, test to compare)
3. Gemini 1.5 Pro (alternative option)

**Strategy**: Test both Claude 3.5 Sonnet AND GPT-4o with same 10 drawings, pick winner.

---

## 📞 Questions?

Check the implementation guides or review the n8n documentation:
- https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent/
- https://docs.n8n.io/advanced-ai/

---

**Last Updated**: October 2025  
**Status**: Phase 1 ready to build  
**Repository**: https://github.com/chronometer/N8N-ai-help
