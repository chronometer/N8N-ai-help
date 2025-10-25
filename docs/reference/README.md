# Quick Reference

Fast lookup guides and cheat sheets.

## 📖 Contents

### [Cheat Sheet](cheat-sheet.md)
Quick reference for common operations and syntax.

### [Glossary](glossary.md)
n8n terminology and definitions.

### [API Reference](api-reference.md)
n8n API basics and endpoints.

### [Keyboard Shortcuts](keyboard-shortcuts.md)
Keyboard shortcuts for the n8n UI.

---

## 🔍 Quick Lookup

### I need to...

**Reference data**: See [Cheat Sheet - Expressions](cheat-sheet.md#expressions)

**Understand a term**: Check [Glossary](glossary.md)

**Call the n8n API**: Read [API Reference](api-reference.md)

**Work faster**: Use [Keyboard Shortcuts](keyboard-shortcuts.md)

---

## 📚 By Topic

### Data Operations

| Task | Reference |
|------|-----------|
| Get field value | [Cheat Sheet](cheat-sheet.md#accessing-data) |
| Transform string | [Cheat Sheet](cheat-sheet.md#string-functions) |
| Math operations | [Cheat Sheet](cheat-sheet.md#math) |
| Array operations | [Cheat Sheet](cheat-sheet.md#arrays) |
| Date/time | [Cheat Sheet](cheat-sheet.md#datetime) |

### Node Operations

| Task | Reference |
|------|-----------|
| Get other node data | [Cheat Sheet](cheat-sheet.md#accessing-nodes) |
| Handle errors | [Cheat Sheet](cheat-sheet.md#error-handling) |
| Loop items | [Cheat Sheet](cheat-sheet.md#loops) |
| Conditional logic | [Cheat Sheet](cheat-sheet.md#conditionals) |

### Workflow Operations

| Task | Reference |
|------|-----------|
| Execute workflow | [Keyboard Shortcuts](keyboard-shortcuts.md) |
| Save workflow | [Keyboard Shortcuts](keyboard-shortcuts.md) |
| Run single node | [Keyboard Shortcuts](keyboard-shortcuts.md) |
| Debug workflow | [Glossary - Debugging](glossary.md#debugging) |

---

## 🚀 Getting Answers Fast

### Most Used Expressions

```javascript
{{ $json.field }}                  // Get field
{{ $node.NodeName.data }}         // Get other node
{{ DateTime.now() }}               // Current time
{{ $json.price * 1.1 }}           // Math
{{ $json.name.toUpperCase() }}    // Uppercase
{{ $json.items.length }}          // Array length
{{ $json.active ? 'Yes' : 'No' }} // Conditional
```

### Most Used Functions

| Function | Purpose |
|----------|---------|
| `map()` | Transform array items |
| `filter()` | Keep/remove items |
| `split()` | Split string |
| `join()` | Join array to string |
| `substring()` | Extract part of string |
| `toLowerCase()` | Make lowercase |
| `toUpperCase()` | Make uppercase |

### Most Common Nodes

| Node | Purpose |
|------|---------|
| Code | Execute JavaScript |
| Filter | Keep/remove items |
| Switch | Branch logic |
| Merge | Combine branches |
| HTTP Request | Call API |
| Edit Fields | Rename/remove fields |

---

## 📝 Popular Tasks

### Task 1: Split Array into Multiple Items
Use "Split Out" node or:
```javascript
return $input.all();
```

### Task 2: Combine Multiple Items
Use "Merge" node or:
```javascript
return [{
  items: $input.all(),
  count: $input.all().length
}];
```

### Task 3: Filter Data
Use "Filter" node or:
```javascript
return $input.all().filter(item => item.value > 100);
```

### Task 4: Transform Each Item
```javascript
return $input.all().map(item => ({
  ...item,
  processed: true,
  timestamp: new Date().toISOString()
}));
```

### Task 5: Get Previous Node Data
```javascript
const data = $node.PreviousNodeName.data;
```

---

## 🔗 Related Sections

- [Code in n8n](../code-in-n8n/) - Complete coding guide
- [Core Concepts](../core-concepts/) - Fundamental concepts
- [Best Practices](../best-practices/) - Guidelines

---

## 💡 Tips

- Bookmark [Cheat Sheet](cheat-sheet.md) for quick access
- Check [Glossary](glossary.md) when unsure of terms
- Review [Keyboard Shortcuts](keyboard-shortcuts.md) to work faster
- Use [API Reference](api-reference.md) for workflow automation

---

**Level:** Beginner to Advanced  
**Purpose:** Quick lookup reference  
**Time:** Varies by lookup
