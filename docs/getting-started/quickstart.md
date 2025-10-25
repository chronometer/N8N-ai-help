# Quick Start Guide

Get familiar with n8n basics in 10 minutes.

## 🎯 Core Concepts

### Workflows
A **workflow** is an automated sequence of actions. Think of it as a recipe:
- Start with a trigger (e.g., "run on schedule")
- Execute a series of steps (nodes)
- Each step processes data and passes it to the next
- End with an action (e.g., send an email)

### Nodes
**Nodes** are building blocks. Examples:
- **Trigger nodes**: Manual Trigger, Schedule Trigger, Webhook
- **Action nodes**: Send Email, HTTP Request, OpenAI
- **Data nodes**: Filter, Merge, Aggregate, Code
- **Integration nodes**: Google Sheets, Slack, CRM systems

### Data Structure
Data flows as **JSON objects** between nodes:

```json
{
  "name": "John",
  "email": "john@example.com",
  "status": "active"
}
```

### Expressions
Use **expressions** to reference and transform data:

```
{{ $node.PreviousNode.data.fieldName }}
{{ $json.myField }}
{{ DateTime.now().toFormat('yyyy-MM-dd') }}
```

---

## 🎮 The Interface

### Top Bar
- **Workflow name** - Click to rename
- **Execute** button - Run workflow manually
- **Activate/Deactivate** toggle - Enable/disable scheduling
- **Save** - Save workflow

### Left Sidebar
- **Nodes panel** - Add nodes to workflow
- **Search** - Find nodes
- **Collections** - Organize workflows

### Canvas
- **Central area** - Build workflows by dragging nodes
- **Connections** - Drag from output of one node to input of another
- **Node details** - Click a node to configure it

### Right Sidebar
- **Node configuration** - Set up selected node
- **Parameters** - Input fields for node
- **Connections** - View connected nodes

---

## 📋 Building Your First Workflow

### Step 1: Create Workflow
1. Click **New Workflow** or **Create Workflow**
2. Give it a name (e.g., "Hello World")

### Step 2: Add Trigger Node
1. Click **Add Node** or drag from left panel
2. Search for "Manual Trigger"
3. Select and add it

### Step 3: Add Action Node
1. Click **Add Node** next to the Manual Trigger
2. Search for "Code"
3. Select the Code node
4. In the code editor, enter:

```javascript
return [{ message: "Hello, World!" }];
```

### Step 4: Execute Workflow
1. Click **Execute** button
2. Watch the data flow through nodes
3. See output in right sidebar

### Step 5: Save
Click **Save** to persist your workflow

---

## 🔌 Connecting Nodes

### Basic Connection
1. Hover over output of first node (right side)
2. Drag to input of second node (left side)
3. Connection appears as a line

### Multiple Inputs/Outputs
- **Merge mode**: Combine multiple inputs
- **Input type selector**: Choose how to handle multiple connections

### Error Handling
- **Continue on error**: Keep workflow running on errors
- **Error branch**: Handle errors separately

---

## 🛠️ Common Node Types

### Trigger Nodes (Start Workflow)
- **Manual Trigger** - Run manually (click Execute)
- **Schedule Trigger** - Run on schedule (daily, hourly, etc.)
- **Webhook** - Run when external service calls
- **Email Trigger** - Run on incoming email

### Action Nodes (Do Something)
- **HTTP Request** - Call APIs
- **Send Email** - Send emails
- **Code** - Execute JavaScript
- **Merge** - Combine data from multiple branches

### Data Processing Nodes
- **Filter** - Keep/remove items based on conditions
- **Aggregate** - Group and summarize data
- **Split Out** - Process items individually
- **Rename Keys** - Rename fields

### Integration Nodes
- **OpenAI** - AI text generation
- **Google Sheets** - Read/write spreadsheets
- **Slack** - Send messages
- **Airtable** - Database operations

---

## 🔑 Adding Credentials

### For APIs (OpenAI, Slack, etc.)

1. Click on node that needs credentials (e.g., OpenAI)
2. Look for credentials dropdown
3. Click "Create new credential"
4. Enter API key or credentials
5. Name the credential
6. Click "Create"

### Storing Securely
- n8n encrypts credentials
- Never expose API keys in code
- Use credentials dropdown instead

---

## 🔄 Data Flow

### Understanding Input/Output

```
[Trigger] ---> [Filter] ---> [Code] ---> [Email]
   (1)            (2)          (3)        (4)
```

1. **Trigger** generates initial data
2. **Filter** receives data, passes subset forward
3. **Code** transforms data
4. **Email** receives and sends

### Accessing Previous Node Data

In expressions or Code node:

```javascript
// In Code node, access input data
const input = $input.item();
console.log(input.data);

// In expressions, reference other nodes
{{ $node.MyNodeName.data.fieldName }}
```

---

## 🧪 Testing and Debugging

### Run Single Node
1. Right-click a node
2. Select "Execute node"
3. See only that node's output

### View Execution Data
1. Execute workflow
2. Click on node in canvas
3. Right sidebar shows input/output data

### Enable Debugging
1. In node configuration, enable debugging
2. Additional data appears in execution

### Common Issues

| Issue | Solution |
|-------|----------|
| Node shows red error | Check configuration, API keys |
| No data appearing | Check connections, node outputs |
| Wrong data | Use Filter node to clean data |
| API fails | Verify credentials, rate limits |

---

## 💾 Saving and Activating

### Save Workflow
- Press `Ctrl+S` (Windows) or `Cmd+S` (Mac)
- Or click Save button
- Auto-saves during editing

### Activate Workflow
- For scheduled workflows, click "Activate" toggle
- Workflow runs on schedule after activation
- Manual workflows don't need activation

### Export Workflow
1. Click menu (three dots)
2. Select "Export"
3. Save as JSON file
4. Share or backup

### Import Workflow
1. Click menu
2. Select "Import"
3. Choose JSON file
4. Workflow imported

---

## 🚀 Next Steps

1. **Build a real workflow** - Follow [Your First AI Workflow](first-ai-workflow.md)
2. **Learn expressions** - See [Expressions Reference](../code-in-n8n/expressions-reference.md)
3. **Explore nodes** - Browse [Key Nodes](../key-nodes/)
4. **Understand data** - Read [Data Structure](../core-concepts/data-structure.md)

---

## 📝 Quick Reference

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Cmd/Ctrl + S | Save |
| Cmd/Ctrl + X | Cut node |
| Cmd/Ctrl + C | Copy node |
| Cmd/Ctrl + V | Paste node |
| Delete | Remove selected node |
| Escape | Deselect |
| Space | Search nodes |

### Common Expressions

```
{{ $json.fieldName }}           // Access data field
{{ $node.NodeName.data }}       // Access other node's data
{{ DateTime.now() }}             // Current time
{{ $json.value * 2 }}           // Do math
{{ $json.name.toUpperCase() }}  // Transform string
```

---

**Time:** 10 minutes  
**Difficulty:** Beginner  
**Next:** [Your First AI Workflow](first-ai-workflow.md)
