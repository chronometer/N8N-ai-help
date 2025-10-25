# Your First AI Workflow: Text Summarization

Learn by doing! Build a simple AI workflow that summarizes text using OpenAI.

## 🎯 What We'll Build

A workflow that:
1. Accepts text input
2. Sends it to OpenAI
3. Gets a summary back
4. Displays the result

**Estimated time:** 15-20 minutes  
**Prerequisites:** OpenAI API key

## 🔑 Get Your OpenAI API Key

1. Go to [OpenAI API](https://platform.openai.com/api/keys)
2. Log in to your account (create one if needed)
3. Click "Create new secret key"
4. Copy the key (you won't see it again!)
5. Keep it safe - never share it

## 🔨 Step-by-Step Building

### Step 1: Create Workflow

1. Open n8n
2. Click **New Workflow**
3. Name it: "Text Summarizer with AI"
4. Click **Create**

### Step 2: Add Manual Trigger

1. Click **Add Node**
2. Search for "Manual Trigger"
3. Click to add it
4. This node lets you start the workflow manually

Configuration:
- Leave defaults as-is
- Click **Save** (bottom right)

### Step 3: Add Code Node (Input Preparation)

1. Click **+** on the Manual Trigger
2. Search for "Code"
3. Click to add Code node
4. Rename it to "Prepare Input" (optional)

In the code editor, replace everything with:

```javascript
return [
  {
    text: "Artificial Intelligence is transforming industries by automating tasks, improving decision-making, and enabling new possibilities. Machine learning models can now analyze vast amounts of data faster than humans, identify patterns, and make predictions. Natural language processing allows computers to understand and generate human language. Computer vision enables machines to interpret images and video. These technologies are creating new opportunities in healthcare, finance, education, and many other sectors."
  }
];
```

Click **Save**.

### Step 4: Add OpenAI Node

1. Click **+** on the Code node
2. Search for "OpenAI"
3. Click to add
4. Select "Completion" as the operation

Configuration:
- **Credentials**: 
  - Click "Create new credential"
  - Choose "OpenAI API"
  - Paste your API key from Step 0
  - Click "Create"
- **Model**: Select `gpt-3.5-turbo` (faster and cheaper) or `gpt-4`
- **Message**: Click in the field and enter:

```
Summarize this text in 2-3 sentences:

{{ $json.text }}
```

- **Temperature**: 0.5
- Click **Save**

### Step 5: Add Code Node (Format Output)

1. Click **+** on OpenAI node
2. Search for "Code"
3. Add Code node
4. Rename to "Format Output"

In code editor:

```javascript
const summary = $input.item().message;

return [
  {
    original_length: $input.all()[0].json.text.length,
    summary: summary,
    word_count: summary.split(' ').length,
    timestamp: new Date().toISOString()
  }
];
```

Click **Save**.

### Step 6: Test the Workflow

1. Click **Execute** button at top
2. Watch the workflow run
3. Data flows through each node
4. In right sidebar, see the final output
5. Should show original text length, summary, and word count

### Step 7: Save Workflow

1. Press `Cmd+S` (Mac) or `Ctrl+S` (Windows)
2. Workflow saved!

---

## 📊 What Just Happened

```
Manual Trigger
    ↓
    Creates { text: "..." }
    ↓
Prepare Input
    ↓
    Passes to OpenAI API
    ↓
OpenAI
    ↓
    Returns { message: "Summary..." }
    ↓
Format Output
    ↓
    Final { summary: "...", word_count: ... }
```

---

## 🎮 Try These Variations

### Variation 1: Web Form Input

Instead of hardcoded text, use a form:

1. Delete the "Prepare Input" code node
2. Add "n8n Form Trigger" node
3. Add a text input field
4. Connect to OpenAI node

The form will appear in n8n, let you enter text, and submit to the workflow.

### Variation 2: Multiple Operations

Add branches for different operations:

```
       ├─→ [Code] Summarize
       ├─→ [Code] Extract Keywords  
Trigger→
       ├─→ [Code] Sentiment Analysis
       └─→ [Merge] Combine Results
```

### Variation 3: Save Results to Spreadsheet

Add a final node:

1. Click **+** on Format Output
2. Search for "Google Sheets"
3. Add node and authenticate
4. Configure to append results
5. Results auto-save to your spreadsheet

### Variation 4: Schedule to Run Daily

Instead of Manual Trigger:

1. Delete Manual Trigger
2. Add "Schedule Trigger"
3. Set to daily at specific time
4. Activate workflow
5. Runs automatically

---

## 🔗 Working with Node Outputs

### Access Data from Previous Nodes

```
In expressions:
{{ $node.OpenAI.data.message }}   // Get summary from OpenAI
{{ $json.text }}                  // Get current item's text

In Code node:
const openaiResult = $input.all()[0].json.message;
const text = $json.text;
```

### Multiple Items

```javascript
// Process multiple items
$input.all().forEach((item) => {
  console.log(item.json.text);
});

// Return multiple outputs
return $input.all().map(item => ({
  original: item.json.text,
  processed: item.json.text.toUpperCase()
}));
```

---

## 🐛 Common Issues and Solutions

### "API Key Invalid"
- Check API key is copied correctly
- Verify API key has permissions
- Create new key if needed

### "Timeout Error"
- Check internet connection
- OpenAI API might be slow
- Try simpler prompt

### "No Output Data"
- Check each node has input data
- Click nodes to see input/output in right panel
- Verify expressions reference correct fields

### "Authentication Failed"
- Re-create credentials with correct API key
- Test key on OpenAI website first

---

## ✨ Next Steps to Level Up

### Expand Your Skills

1. **Use Different Models** - Try GPT-4, text-davinci-003
2. **Add More Nodes** - Send summaries via Slack, email, or save to database
3. **Handle Errors** - Add error handling branches
4. **Add Conditions** - Different actions based on summary length
5. **Loop Over Multiple** - Process multiple texts in one run

### Try These Patterns

- **Document Q&A** - Ask questions about documents
- **Email Processing** - Summarize incoming emails
- **Content Generation** - Create blog posts or social content
- **Data Analysis** - Analyze survey responses

### Learn More

- [Core Concepts](../core-concepts/) - Understand the fundamentals
- [AI Workflows](../ai-workflows/) - Advanced AI patterns
- [LangChain Integration](../ai-workflows/langchain-integration.md) - Build intelligent chains
- [Examples](../examples/) - More workflow examples

---

## 🎓 Key Takeaways

✅ **Workflows** = Sequences of connected nodes  
✅ **Nodes** = Individual operations (triggers, actions, data processing)  
✅ **Data** = Flows as JSON between nodes  
✅ **Expressions** = Reference and transform data with `{{ }}`  
✅ **Credentials** = Store API keys securely  
✅ **Execute** = Run workflow to test and debug  

---

## 📝 Checkpoint

- [ ] Workflow created and named
- [ ] Manual Trigger added
- [ ] Code node with input created
- [ ] OpenAI node configured with API key
- [ ] Format Output code node added
- [ ] Workflow executed successfully
- [ ] Output shows summary and word count
- [ ] Workflow saved

---

**Completion Time:** 15-20 minutes  
**Difficulty:** Beginner  
**Next:** [Explore AI Workflows](../ai-workflows/) or [Build More Complex Workflows](../patterns/)

---

## 🔗 Resources

- [OpenAI Documentation](https://platform.openai.com/docs)
- [n8n Code Node](../key-nodes/code-node.md)
- [n8n Expression Editor](../code-in-n8n/expressions-reference.md)
- [More Examples](../examples/)
