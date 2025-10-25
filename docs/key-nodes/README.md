# Key Nodes Documentation

Reference documentation for critical nodes used in n8n workflows.

## 📖 Contents

### [AI Transform Node](ai-transform.md)
Transform data using AI capabilities.

### [Code Node](code-node.md)
Execute JavaScript code in workflows.

### [HTTP Request Node](http-request.md)
Make HTTP requests to APIs.

### [Webhooks and Forms](webhook.md)
Receive data from external sources.

### [Schedule Trigger](schedule-trigger.md)
Run workflows on a schedule.

### [Aggregate Node](aggregate.md)
Group and aggregate data.

### [Filter and Split Nodes](filter-split.md)
Filter, split, and process data.

---

## 🎯 Quick Node Selection

### I need to...

**Execute code** → [Code Node](code-node.md)  
**Call an API** → [HTTP Request Node](http-request.md)  
**Process with AI** → [AI Transform Node](ai-transform.md)  
**Filter data** → [Filter Node](filter-split.md)  
**Run on schedule** → [Schedule Trigger](schedule-trigger.md)  
**Get external data** → [Webhook Node](webhook.md)  
**Group data** → [Aggregate Node](aggregate.md)  

---

## 📊 Node Categories

### Trigger Nodes (Start Workflows)
- Schedule Trigger
- Webhook
- Manual Trigger (basic)

### Data Processing Nodes
- Filter
- Aggregate
- Code
- Edit Fields
- Split Out

### Action Nodes (Do Something)
- HTTP Request
- Send Email
- Save to Database
- AI Transform

### Integration Nodes
- OpenAI
- Anthropic
- Google Sheets
- Slack

---

## 💡 Most Used Nodes

| Node | Purpose | Frequency |
|------|---------|-----------|
| Code | Execute logic | Daily |
| HTTP Request | API calls | Daily |
| Filter | Data filtering | Daily |
| Edit Fields | Rename/transform | Daily |
| Schedule Trigger | Schedule execution | Weekly |
| Merge | Combine branches | Frequently |
| Aggregate | Group data | Frequently |

---

## 🚀 Common Node Patterns

### Pattern 1: Trigger → Process → Output
```
Manual Trigger → Code (process) → HTTP Request (output)
```

### Pattern 2: Trigger → Filter → Multiple Paths
```
Trigger → Filter → {Path A, Path B} → Merge
```

### Pattern 3: Data → Aggregate → Send
```
Input → Aggregate → Format → HTTP Request
```

### Pattern 4: Schedule → Get Data → Process
```
Schedule → HTTP (get data) → Code → Save
```

---

## 🔧 Node Configuration Tips

### General Tips

1. **Name nodes clearly** - "Get Customers", not "Node 1"
2. **Test each node** - Right-click and execute single node
3. **Check output** - Click node to see output data
4. **Use expressions** - Dynamic values with `{{ }}`
5. **Error handling** - Set to "Continue on error" for important paths

### For Each Node

- **Trigger nodes**: No error handling needed (they start workflows)
- **Action nodes**: Add error handling
- **Data nodes**: Test output format
- **Integration nodes**: Verify credentials work

---

## 📚 Learning Path

### Beginner
1. Learn [Code Node](code-node.md) basics
2. Learn [HTTP Request Node](http-request.md)
3. Learn [Filter Node](filter-split.md)

### Intermediate
1. Master [Schedule Trigger](schedule-trigger.md)
2. Learn [Aggregate Node](aggregate.md)
3. Understand data transformation patterns

### Advanced
1. Optimize with [HTTP Request](http-request.md) batching
2. Advanced filtering and splitting
3. Complex error handling patterns

---

## 🐛 Common Issues

### Issue 1: Node Shows Red Error
**Solution:** Check configuration, API keys, input format

### Issue 2: No Output Data
**Solution:** Check connections, verify node returns data

### Issue 3: Unexpected Output Format
**Solution:** Use Debug node, check data transformation

### Issue 4: API Request Fails
**Solution:** Test API separately, check credentials, verify URL

---

## 🔐 Security Considerations

- **Never expose API keys** in node configuration (use credentials)
- **Validate all input data** before processing
- **Use HTTPS for HTTP requests** when possible
- **Limit node access** - Don't expose sensitive data nodes to all users
- **Audit node changes** - Track modifications

---

## 🔗 Related Sections

- [Code in n8n](../code-in-n8n/) - Coding reference
- [AI Workflows](../ai-workflows/) - AI node usage
- [Core Concepts](../core-concepts/) - Fundamental concepts
- [Best Practices](../best-practices/) - Guidelines

---

## 📝 Node Checklist

For each node in your workflow:

- [ ] Clear, descriptive name
- [ ] Configuration complete
- [ ] Credentials set (if needed)
- [ ] Output format verified
- [ ] Error handling configured
- [ ] Tested individually
- [ ] Documented purpose

---

## 💾 Quick Reference

### Code Node
```javascript
// Most common pattern
return $input.all().map(item => ({
  ...item,
  processed: true
}));
```

### HTTP Request
- Method: GET/POST/PUT/DELETE
- URL: Full endpoint URL
- Authentication: Basic, Bearer, OAuth
- Headers: Content-Type, Authorization

### Filter
- Property: Field to check
- Operator: ==, <, >, contains, etc.
- Value: What to compare

---

## 📚 Resources

- [Official Nodes Reference](https://docs.n8n.io/integrations/builtin/core-nodes/)
- [Code Node Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/)
- [HTTP Request Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)

---

**Level:** Beginner to Advanced  
**Time:** Varies by node  
**Prerequisites:** Understand [Core Concepts](../core-concepts/)
