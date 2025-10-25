# AI Chains in n8n

Learn how to build chains - structured sequences of AI operations that solve complex problems.

## What is a Chain?

A **chain** is a series of predefined steps that work together to accomplish a specific task. Each step processes the output of the previous step.

### Chain vs Single Call

```
Simple Call:
Input → OpenAI → Output

Chain:
Input → Step 1 (Extract) → Step 2 (Analyze) → Step 3 (Format) → Output
```

### When to Use Chains

- Breaking complex tasks into steps
- Ensuring consistent processing
- Reusing logic across workflows
- Building reproducible workflows
- Improving reliability with error handling

---

## Basic Chain Structure

### Components

1. **Input Processing** - Prepare the input
2. **Processing Steps** - Each transforms data
3. **Output Formatting** - Format final result

### Example: Document Analysis Chain

```
Document Text
    ↓
[Step 1] Extract Key Information
    ↓
[Step 2] Summarize Content
    ↓
[Step 3] Generate Keywords
    ↓
[Step 4] Combine Results
    ↓
Final Report
```

---

## Building a Chain in n8n

### Example: Multi-Step Content Analysis

```
Manual Trigger
    ↓
Code Node: Input
    ↓
OpenAI: Extract Summary
    ↓
OpenAI: Generate Keywords
    ↓
Code Node: Combine Results
    ↓
Format Output
```

### Step-by-Step Implementation

#### 1. Input Node

```javascript
// Code Node: "Prepare Input"
return [{
  title: "n8n Workflows",
  content: "n8n enables workflow automation...",
  language: "english"
}];
```

#### 2. Summary Step

Configure OpenAI node:
- **Prompt**: "Summarize this content in 3 sentences: {{ $json.content }}"
- **Model**: gpt-3.5-turbo

#### 3. Keywords Step

Configure second OpenAI node:
- **Prompt**: "Extract 5 key keywords from: {{ $json.content }}"
- **Model**: gpt-3.5-turbo

#### 4. Combine Results

```javascript
// Code Node: "Combine Results"
const previous = $input.all();

return [{
  title: $json.title,
  original_content: $json.content,
  summary: previous[1].json.message,
  keywords: previous[2].json.message.split(','),
  word_count: $json.content.split(' ').length,
  processed_at: new Date().toISOString()
}];
```

---

## Chain Patterns

### Sequential Chain
```
Step 1 → Step 2 → Step 3 → Result
```
Each step depends on previous. Best for linear processes.

### Parallel Chain with Merge
```
     ├→ Step 1A
Step 0 ├→ Step 1B → Merge → Result
     └→ Step 1C
```
Multiple branches processed in parallel. Use Merge node.

### Conditional Chain
```
Step 1 → [If/Switch] → Step 2A or Step 2B → Result
```
Different paths based on conditions.

### Looping Chain
```
Step 1 → Step 2 → [Check] → Loop back or Continue → Step 3
```
Repeat steps until condition met.

---

## Common Chain Types

### Type 1: Extraction Chain
Extract specific information from data.

```
Document
  ↓
Extract: Names
Extract: Dates
Extract: Amounts
  ↓
Combine
  ↓
Structured Data
```

### Type 2: Transformation Chain
Convert data from one format to another.

```
Raw Data
  ↓
Clean: Remove noise
Transform: Normalize
Enhance: Add context
  ↓
Structured Data
```

### Type 3: Analysis Chain
Analyze and generate insights.

```
Data
  ↓
Analyze: Sentiment
Analyze: Topics
Analyze: Patterns
  ↓
Report
```

### Type 4: Generation Chain
Generate new content based on input.

```
Input
  ↓
Generate: Draft
Refine: Improve writing
Format: Apply style
  ↓
Final Content
```

---

## Error Handling in Chains

### Try-Catch Pattern

```
Step 1 → Error? ─→ Handle Error → Continue
           ↓
         Success → Next Step
```

Implementation:

1. Set node to "Continue on error"
2. Check for error in next step
3. Branch accordingly

```javascript
// Detect error
if ($input.item()?.error) {
  // Handle error
  return [{ status: 'error', message: 'Processing failed' }];
}

// Continue normally
return [{ status: 'success', data: $json }];
```

### Retry Pattern

```
Step → Failed? → Wait → Retry → Success/Fail
```

Use Wait node + loop for retry logic.

---

## Advanced Chain Techniques

### 1. Caching Results

```javascript
// Check if already processed
const cache = $input.item().cache || {};

if (cache[key]) {
  return [{ result: cache[key] }];
}

// Process and cache
const result = callExpensiveOperation();
cache[key] = result;

return [{ result, cache }];
```

### 2. Streaming/Chunking

For large inputs, process in chunks:

```javascript
const chunk_size = 1000;
const chunks = [];

for (let i = 0; i < input.length; i += chunk_size) {
  chunks.push(input.slice(i, i + chunk_size));
}

return chunks.map((chunk, idx) => ({
  chunk_id: idx,
  data: chunk
}));
```

### 3. State Management

Maintain state across steps:

```javascript
// Store in static data
const state = $workflow.getStaticData('default');

state.step_count = (state.step_count || 0) + 1;
state.current_input = $json;

$workflow.setStaticData('default', state);

return [$json];
```

---

## Chain Best Practices

### 1. Keep Steps Independent
Each step should work with clear input/output.

### 2. Add Intermediate Checks
Validate data between steps.

### 3. Log Important Data
Debug by logging step outputs.

```javascript
console.log('Step X input:', $input.item());
console.log('Step X output:', result);
```

### 4. Handle Errors Explicitly
Don't ignore failures.

### 5. Optimize for Cost
- Use faster models where possible
- Cache expensive operations
- Batch similar requests

### 6. Document Steps
Add comments and clear naming.

### 7. Test Incrementally
Test each step independently before connecting.

---

## Chain Examples

### Example 1: Email Classification Chain

```javascript
// Step 1: Extract email content
// Step 2: Classify category (Support, Sales, Spam)
// Step 3: Extract priority level
// Step 4: Generate response template
// Step 5: Send or route appropriately
```

### Example 2: Data Enrichment Chain

```javascript
// Step 1: Receive customer data
// Step 2: Lookup additional info from APIs
// Step 3: Enrich with external data
// Step 4: Validate and clean
// Step 5: Save enriched record
```

### Example 3: Content Generation Chain

```javascript
// Step 1: Receive topic and outline
// Step 2: Generate initial draft with OpenAI
// Step 3: Improve writing quality
// Step 4: Add examples and references
// Step 5: Format for publication
```

---

## Performance Considerations

### Parallel vs Sequential

**Use Parallel when:**
- Steps are independent
- Reduce total execution time
- Can merge results later

**Use Sequential when:**
- Each step depends on previous
- Results need to be in order
- Dependencies are complex

### Execution Time

```
Sequential: Step1 (5s) + Step2 (5s) + Step3 (5s) = 15s
Parallel:   max(Step1, Step2, Step3) = 5s
```

### Cost Optimization

- Reuse results when possible
- Use cheaper models when suitable
- Batch multiple operations
- Implement caching

---

## Debugging Chains

### View Intermediate Results

1. Execute workflow
2. Click each node in canvas
3. Right sidebar shows input/output

### Add Debug Nodes

```javascript
// Code node for debugging
console.log('Debug:', $input.item());
return [$input.item()];
```

### Check Error Logs

- Look at execution logs
- Check node status (red = error)
- Review error messages

---

## Next Steps

- Learn [LangChain Integration](langchain-integration.md) for advanced chains
- Study [AI Agents](ai-agents.md) for autonomous decision-making
- Explore [Memory Management](memory-management.md) for context
- Build your own chain workflow

---

## Resources

- [Official n8n Chains Doc](https://docs.n8n.io/advanced-ai/)
- [LangChain Chains](https://docs.langchain.com/modules/chains)
- [n8n Code Examples](https://docs.n8n.io/code-in-n8n/)

---

**Level:** Intermediate  
**Time:** 30 minutes  
**Prerequisites:** Understand [Your First AI Workflow](../getting-started/first-ai-workflow.md)
