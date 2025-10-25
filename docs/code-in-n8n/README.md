# Code in n8n

Master coding and scripting in n8n workflows.

## 📖 Contents

### [Code Node Guide](code-node.md)
Learn how to use the Code node for JavaScript execution.

### [Expressions Reference](expressions-reference.md)
Complete guide to expressions and dynamic data references.

### [Built-in Methods](built-in-methods.md)
Methods and variables available in n8n.

### [Custom Variables](custom-variables.md)
Create and use custom variables in workflows.

### [Data Transformation](data-transformation.md)
Transform and manipulate data with built-in functions.

---

## 🎯 Quick Start

### Where Can I Write Code?

1. **Code Node** - Full JavaScript execution
2. **Expressions** - Simple `{{ }}` syntax
3. **Node Configuration** - In input fields with `{{ }}`

### When to Use What?

| Need | Use |
|------|-----|
| Simple field reference | Expressions: `{{ $json.name }}` |
| Data transformation | Code Node |
| Conditional logic | If/Switch node or Expression |
| Complex logic | Code Node with JavaScript |

---

## 🔑 Key Concepts

### Code Node
- Full JavaScript environment
- Access to input data
- Return array of objects
- No size limits on code

### Expressions
- Lightweight syntax
- Used in node parameters
- `{{ code_here }}`
- Quick transformations

### Built-in Variables
- `$json` - Current item data
- `$input` - Input data object
- `$node` - Other node data
- `$workflow` - Workflow information

### Data Types

n8n works with JSON:

```javascript
// String
"hello"

// Number
42

// Array
[1, 2, 3]

// Object
{ name: "John", age: 30 }

// Boolean
true / false

// Null
null
```

---

## 💡 Common Tasks

### Read Data from Input

```javascript
// In Code node
const input = $input.item();
console.log(input.fieldName);

// In expression
{{ $json.fieldName }}
```

### Transform Data

```javascript
return [
  {
    ...item,
    processed: true,
    timestamp: new Date().toISOString()
  }
];
```

### Access Other Nodes

```javascript
// In Code node
const previousData = $node.PreviousNodeName.data;

// In expression
{{ $node.PreviousNodeName.data.field }}
```

### Loop Over Items

```javascript
const items = $input.all();
return items.map(item => ({
  ...item,
  doubled: item.value * 2
}));
```

### Conditional Logic

```javascript
return $input.all().map(item => {
  if (item.value > 100) {
    return { ...item, category: 'high' };
  } else {
    return { ...item, category: 'low' };
  }
});
```

---

## 📚 Code Node Basics

### Input Data

```javascript
// Get current item
const item = $input.item();

// Get all items
const items = $input.all();

// Get specific item
const first = $input.all()[0];
```

### Output Data

```javascript
// Return single item (wrapped in array)
return [{ name: "John", age: 30 }];

// Return multiple items
return [
  { id: 1, name: "John" },
  { id: 2, name: "Jane" }
];

// No output
return [];
```

### Error Handling

```javascript
try {
  // Your code
  return [result];
} catch (error) {
  // Handle error
  return [{ error: error.message }];
}
```

---

## 🔗 Expressions Guide

### Basic Syntax

```
{{ expression_code }}
```

### Common Patterns

```
{{ $json.field }}                           // Get field
{{ $node.NodeName.data }}                   // Get node data
{{ $json.price * 1.1 }}                     // Math
{{ $json.name.toUpperCase() }}              // String methods
{{ DateTime.now() }}                         // Date/time
{{ $json.items.length }}                    // Array length
{{ $json.active ? 'Yes' : 'No' }}           // Conditional
```

---

## ⚙️ Environment

### Available in Code Node

- JavaScript (ES6+)
- Async/await
- Common libraries
- n8n helper functions

### Available Methods

See [Built-in Methods](built-in-methods.md) for complete list.

---

## 🐛 Debugging

### Console Logging

```javascript
console.log('Debug:', $input.item());
```

See output in execution logs.

### Test with Sample Data

1. Click "Test" on Code node
2. Provide sample input
3. See output immediately

### Inspect Data

1. Execute workflow
2. Click node in canvas
3. See input/output in right sidebar

---

## 🚀 Next Steps

1. **Learn Code Node** - [Code Node Guide](code-node.md)
2. **Master Expressions** - [Expressions Reference](expressions-reference.md)
3. **Use Built-ins** - [Built-in Methods](built-in-methods.md)
4. **Transform Data** - [Data Transformation](data-transformation.md)
5. **Custom Variables** - [Custom Variables](custom-variables.md)

---

## 📝 Best Practices

1. **Test incrementally** - Test each step
2. **Use descriptive names** - Clear variable names
3. **Add comments** - Explain complex logic
4. **Handle errors** - Try-catch blocks
5. **Keep it simple** - Don't over-complicate
6. **Optimize performance** - Avoid loops, use native methods

---

## 🔗 Resources

- [Official Code Node Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/)
- [Expressions Docs](https://docs.n8n.io/code-in-n8n/expressions/)
- [JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

---

**Level:** Beginner to Advanced  
**Time:** Varies by topic  
**Prerequisites:** None (though coding basics help)
