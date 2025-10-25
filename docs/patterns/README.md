# Common Workflow Patterns

Ready-to-use patterns for common scenarios.

## 📖 Contents

### [RAG Implementation](rag-pattern.md)
Build retrieval-augmented generation systems.

### [Agent Workflow](agent-pattern.md)
Create autonomous agents that take actions.

### [Human-in-Loop](human-in-loop.md)
Add human approval and decision-making.

### [API Integration](api-integration.md)
Integrate external APIs into workflows.

### [Data Enrichment](data-enrichment.md)
Enhance data with external sources.

---

## 🎯 Pattern Selection Guide

### By Use Case

**Need to query documents?**
→ [RAG Implementation](rag-pattern.md)

**Need autonomous decision-making?**
→ [Agent Workflow](agent-pattern.md)

**Need human review?**
→ [Human-in-Loop](human-in-loop.md)

**Need external data?**
→ [API Integration](api-integration.md) or [Data Enrichment](data-enrichment.md)

---

## 💡 Pattern Types

### Type 1: RAG (Retrieval Augmented Generation)

**Use when:** You need to answer questions about specific documents or data.

**Flow:**
```
Query
  ↓
Retrieve relevant documents
  ↓
Generate embeddings
  ↓
Search vector database
  ↓
Build context from results
  ↓
OpenAI generates answer based on context
  ↓
Response
```

**Example:** Company policy Q&A system

### Type 2: Agent Workflow

**Use when:** You need the AI to make decisions and take actions.

**Flow:**
```
Task
  ↓
Agent reads task
  ↓
Agent selects tool/action
  ↓
Execute tool
  ↓
Analyze result
  ↓
Decide next step
  ↓
Loop or finish
  ↓
Result
```

**Example:** Customer service bot that can look up info and create tickets

### Type 3: Human-in-Loop

**Use when:** You need human approval for critical decisions.

**Flow:**
```
Data arrives
  ↓
AI analyzes
  ↓
AI recommends action
  ↓
Wait for human approval
  ↓
Execute approved action
  ↓
Done
```

**Example:** Invoice approval workflow

### Type 4: API Integration

**Use when:** You need to pull data from external services.

**Flow:**
```
Trigger
  ↓
Prepare request
  ↓
Call API
  ↓
Process response
  ↓
Transform data
  ↓
Save result
```

**Example:** Sync data from Salesforce to Google Sheets

### Type 5: Data Enrichment

**Use when:** You need to add context or information to data.

**Flow:**
```
Raw data
  ↓
Lookup additional info
  ↓
Call multiple APIs
  ↓
Combine results
  ↓
Enrich record
  ↓
Save enriched data
```

**Example:** Add company info, website, and social profiles to contact

---

## 🛠️ Key Components

### Error Handling

Every pattern should handle errors:

```
Action
  ↓
Error? → Retry or alternate path
  ↓
Success → Continue
```

### Data Validation

Validate data before processing:

```javascript
if (!$json.email || !$json.email.includes('@')) {
  return [{ error: 'Invalid email' }];
}
```

### Logging

Log important steps for debugging:

```javascript
console.log('Processing:', $json.id);
console.log('Result:', result);
```

### Rate Limiting

Respect API limits:

```javascript
// Wait between requests
await new Promise(r => setTimeout(r, 1000));
```

---

## 📊 Common Architecture

Most patterns follow this structure:

```
Trigger
  ↓
Input Validation
  ↓
Data Preparation
  ↓
Core Logic (API, AI, DB, etc.)
  ↓
Response Processing
  ↓
Error Handling
  ↓
Output/Save
```

---

## 🚀 Building Your Own Pattern

### 1. Identify the Problem
- What are you trying to automate?
- What data do you have?
- What's the desired output?

### 2. Design the Flow
- Main steps needed
- Where errors can occur
- How to validate data

### 3. Implement Step by Step
- Start with skeleton
- Add main logic
- Add error handling
- Add logging

### 4. Test Thoroughly
- Test happy path
- Test edge cases
- Test error scenarios

### 5. Monitor and Adjust
- Check logs regularly
- Monitor error rates
- Optimize bottlenecks

---

## 📚 Pattern Examples by Industry

### E-commerce
- Product catalog sync
- Order processing
- Inventory updates
- Customer notifications

### Finance
- Invoice processing
- Expense approval
- Payment reconciliation
- Financial reporting

### HR
- Employee onboarding
- Leave requests
- Performance reviews
- Payroll integration

### Marketing
- Lead scoring
- Email campaigns
- Social media posting
- Content distribution

### Support
- Ticket routing
- Customer inquiry response
- Knowledge base search
- Survey distribution

---

## ⚡ Performance Tips

### 1. Batch Operations
Process multiple items together instead of one-by-one.

### 2. Cache Results
Store results to avoid repeated calls.

### 3. Parallel Processing
Run independent steps simultaneously.

### 4. Selective Logging
Log only important data to save space.

### 5. Optimize Queries
Minimize API calls, use efficient filters.

---

## 🔗 Related Sections

- [AI Workflows](../ai-workflows/) - AI-specific patterns
- [Examples](../examples/) - Real-world implementations
- [Best Practices](../best-practices/) - General guidelines

---

## 📝 Checklist for Each Pattern

- [ ] Input validation working
- [ ] Main logic implemented
- [ ] Error handling in place
- [ ] Logging added
- [ ] Tested with sample data
- [ ] Performance acceptable
- [ ] Documentation clear
- [ ] Ready for production

---

## 🚀 Next Steps

1. **Pick a pattern** matching your use case
2. **Read the guide** for that pattern
3. **Follow the steps** to build it
4. **Test thoroughly** before deploying
5. **Monitor performance** and iterate

---

## 📚 Resources

- [n8n Patterns Documentation](https://docs.n8n.io/)
- [Community Templates](https://n8n.io/templates)
- [n8n Forum](https://community.n8n.io/)

---

**Level:** Intermediate  
**Time:** Varies by pattern (30 min - 2 hours)  
**Prerequisites:** Understand [Core Concepts](../core-concepts/)
