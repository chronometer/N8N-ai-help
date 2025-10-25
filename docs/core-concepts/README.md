# Core Concepts in n8n

Understand the fundamental principles and components of n8n.

## 📖 Contents

### [Workflow Basics](workflow-basics.md)
Learn what workflows are and how they work.

### [Data Structure](data-structure.md)
Understand how data is represented and flows through n8n.

### [Nodes and Connections](nodes-and-connections.md)
Learn about nodes, connections, and building blocks.

### [Expressions](expressions.md)
Master expression syntax for dynamic data handling.

### [Error Handling](error-handling.md)
Learn strategies for handling errors in workflows.

### [Flow Logic](flow-logic.md)
Understand conditionals, loops, merging, and branching.

---

## 🎯 Quick Reference

### Key Terms

| Term | Meaning |
|------|---------|
| **Workflow** | Automated sequence of actions |
| **Node** | Individual action or step |
| **Trigger** | Event that starts a workflow |
| **Connection** | Link between nodes passing data |
| **Expression** | Dynamic code like `{{ $json.field }}` |
| **Data** | Information flowing through workflow |
| **Execution** | Single run of a workflow |

---

## 🚀 Learning Path

### Beginner (Start Here)
1. [Workflow Basics](workflow-basics.md) - Understand core concepts
2. [Nodes and Connections](nodes-and-connections.md) - Learn building blocks
3. [Data Structure](data-structure.md) - How data flows

### Intermediate
1. [Expressions](expressions.md) - Dynamic data handling
2. [Flow Logic](flow-logic.md) - Conditionals and loops
3. [Error Handling](error-handling.md) - Robust workflows

### Advanced
- Combine all concepts into complex workflows
- Optimize performance and reliability
- Build reusable patterns

---

## 💡 Core Principles

### 1. Data-Driven Execution
Everything in n8n revolves around data:
- Trigger generates initial data
- Each node processes and transforms data
- Data flows left to right through connections

### 2. Visual Programming
- Build workflows without code (though code is available)
- Each operation is a visual node
- Connections show data flow clearly

### 3. Expressions for Flexibility
- Reference data with `{{ $json.field }}`
- Access other nodes with `{{ $node.NodeName.data }}`
- Transform data with expressions

### 4. Error Handling
- Workflows can continue on error
- Error branches for specific handling
- Robust automation requires error planning

### 5. Modular Design
- Break workflows into reusable steps
- Each node has clear input/output
- Compose complex workflows from simple parts

---

## 📊 Visual Workflow Example

```
Manual Trigger                   Trigger Node
      ↓                          (starts workflow)
   Input Data
      ↓
Filter Node                      Data Processing
 (remove invalid)
      ↓
   Filtered Data
      ↓
    If/Switch                    Flow Control
 (branch logic)                  (conditional)
    ↙        ↘
Option A    Option B
   ↓          ↓
Process    Process
   ↓          ↓
   └─→ Merge ←─┘               Combine branches
      ↓
   Final Data
      ↓
Output Node                      Action
(send/save)
```

---

## 🔑 Essential Concepts

### Workflows

A **workflow** is:
- Automated sequence of actions
- Triggered by events (manual, schedule, webhook)
- Processes data step-by-step
- Produces output/action

### Nodes

**Nodes** are:
- Individual operations or steps
- Have input (left side) and output (right side)
- Configured with parameters
- Combined to build workflows

**Node Types:**
- **Trigger nodes** - Start workflows
- **Action nodes** - Do something
- **Data processing** - Transform/filter
- **Connectors** - Integrate external services

### Data

**Data** is:
- JSON objects flowing through nodes
- Each item is a record
- Transformed by nodes
- Accessible via expressions

Example:
```json
{
  "name": "John",
  "email": "john@example.com",
  "age": 30
}
```

### Expressions

**Expressions** let you:
- Reference data fields: `{{ $json.name }}`
- Access other nodes: `{{ $node.PreviousNode.data }}`
- Transform data: `{{ $json.name.toUpperCase() }}`
- Calculate: `{{ $json.price * 1.1 }}`

---

## 🎮 Interface Overview

### Canvas
Central area where you build workflows.

### Left Sidebar
- Add nodes
- Search nodes
- Manage workflows

### Right Sidebar
- Configure selected node
- View node output
- Set parameters

### Top Bar
- Workflow name
- Execute button
- Save/Activate buttons

---

## 🔄 Execution Flow

### 1. Trigger Event
Workflow starts when trigger fires.

### 2. Initial Data
Trigger node provides initial data.

### 3. Processing
Each node processes data sequentially.

### 4. Data Transformation
Each node receives input, processes, outputs result.

### 5. Final Action
Last node performs final action (save, send, etc.).

---

## 📚 Next Steps

1. **Read Workflow Basics** - Understand fundamentals
2. **Learn Data Structure** - See how data flows
3. **Study Expressions** - Master dynamic data
4. **Practice Flow Logic** - Build complex flows
5. **Handle Errors** - Build robust workflows

---

## 🔗 Related Sections

- [Getting Started](../getting-started/) - Quick introduction
- [AI Workflows](../ai-workflows/) - AI-specific concepts
- [Code in n8n](../code-in-n8n/) - Advanced coding

---

## 💾 Tips

- Start simple, add complexity gradually
- Test workflows step-by-step
- Use Debug Helper node to inspect data
- Refer to expression syntax frequently
- Keep workflows organized with clear naming

---

**Section Level:** Beginner to Intermediate  
**Time to Complete:** 2-3 hours  
**Prerequisites:** None
