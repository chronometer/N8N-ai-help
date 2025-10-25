# AI Service Integrations

Connect to AI providers and services in your workflows.

## 📖 Contents

### [OpenAI Integration](openai.md)
GPT-4, GPT-3.5-turbo, and text generation models.

### [Anthropic Integration](anthropic.md)
Claude and Anthropic models.

### [Google AI Integration](google-ai.md)
Gemini, Vertex AI, and Google services.

### [Pinecone Integration](pinecone.md)
Vector database for semantic search and RAG.

### [Supabase Integration](supabase.md)
PostgreSQL with vector support.

### [LangSmith Integration](langsmith.md)
Monitor and debug LangChain workflows.

---

## 🎯 Quick Selection

### For Text Generation
- **Best performance:** OpenAI GPT-4
- **Best value:** OpenAI GPT-3.5-turbo
- **Good alternative:** Anthropic Claude
- **Google option:** Gemini

### For Vector Database
- **Managed service:** Pinecone
- **Self-hosted option:** Supabase (PostgreSQL with pgvector)
- **Open source:** Weaviate

### For Monitoring
- **LangChain workflows:** LangSmith
- **n8n workflows:** Built-in execution logs

---

## 🔑 Prerequisites

All integrations require:
1. Account with the service
2. API key or credentials
3. n8n credentials configured

---

## ⚙️ Common Setup Steps

### Step 1: Create Account
Sign up for the service (usually free tier available).

### Step 2: Get API Key
Navigate to API keys or credentials section and create one.

### Step 3: Add to n8n
1. In workflow, add node for the service
2. Click credentials dropdown
3. Select "Create new credential"
4. Paste API key
5. Name the credential
6. Click "Create"

### Step 4: Use in Workflow
- Select the credential from dropdown
- Configure node parameters
- Test the connection

---

## 🏆 Most Popular Integrations

### OpenAI
**Best for:** Text generation, summarization, classification  
**Models:** GPT-4, GPT-3.5-turbo  
**Cost:** Pay per token  
**Setup time:** 5 minutes

### Pinecone
**Best for:** Semantic search, RAG workflows  
**Type:** Vector database  
**Cost:** Free tier available  
**Setup time:** 10 minutes

### Anthropic
**Best for:** Long-form text, analysis  
**Models:** Claude  
**Cost:** Competitive pricing  
**Setup time:** 5 minutes

### Supabase
**Best for:** Vector storage, queries  
**Type:** PostgreSQL + vectors  
**Cost:** Free tier available  
**Setup time:** 15 minutes

---

## 💡 Use Case Examples

### Use Case 1: Document Summarization
- **Integration:** OpenAI
- **Flow:** Extract text → OpenAI → Store summary
- **Time:** 10 minutes

### Use Case 2: Document Q&A
- **Integration:** OpenAI + Pinecone
- **Flow:** Upload → Generate embeddings → Store in Pinecone → Query
- **Time:** 30 minutes

### Use Case 3: Content Analysis
- **Integration:** OpenAI
- **Flow:** Get content → Multiple AI operations → Combine results
- **Time:** 20 minutes

### Use Case 4: Data Classification
- **Integration:** OpenAI or Anthropic
- **Flow:** Input data → AI classify → Store results
- **Time:** 15 minutes

---

## 🔐 Security Best Practices

1. **Never hardcode API keys** - Use credentials
2. **Rotate keys regularly** - Change keys every 3-6 months
3. **Limit permissions** - Use keys with minimal required permissions
4. **Monitor usage** - Watch for unusual activity
5. **Use HTTPS** - Always use secure connections
6. **Store securely** - n8n encrypts credentials

---

## 💰 Cost Optimization

### OpenAI
- GPT-3.5-turbo is 5-10x cheaper than GPT-4
- Cache results to avoid re-processing
- Use streaming for long responses

### Pinecone
- Free tier: 1M vectors
- Scale pricing as needed
- Use efficient embedding models

### Anthropic
- Similar pricing to OpenAI
- Good for long documents
- Consider batch processing

---

## 🐛 Common Issues

### Issue 1: Invalid API Key
**Solution:** Verify key is correct, try regenerating it

### Issue 2: Rate Limit Exceeded
**Solution:** Implement delays, use cheaper models, batch requests

### Issue 3: Timeout Errors
**Solution:** Check internet, increase timeout, simplify requests

### Issue 4: Quota Exceeded
**Solution:** Check account limits, upgrade plan if needed

---

## 🔄 Integration Patterns

### Pattern 1: Simple API Call
```
Trigger → Prepare Input → API Call → Format Output
```

### Pattern 2: With Error Handling
```
Trigger → Try {API Call} → Catch {Handle Error} → Output
```

### Pattern 3: Batch Processing
```
Trigger → Loop {API Call} → Combine Results → Output
```

### Pattern 4: Multiple Services
```
Trigger → API Call 1 → API Call 2 → Combine → Output
```

---

## 📚 Learning Path

### Beginner
1. Start with OpenAI
2. Build simple text generation workflow
3. Add error handling

### Intermediate
1. Add vector database (Pinecone/Supabase)
2. Build RAG workflow
3. Combine multiple services

### Advanced
1. Optimize costs and performance
2. Add monitoring and logging
3. Build production systems

---

## 🔗 Related Sections

- [AI Workflows](../ai-workflows/) - AI concepts
- [Patterns](../patterns/) - Integration patterns
- [Examples](../examples/) - Real-world examples
- [Best Practices](../best-practices/) - Guidelines

---

## 📝 Integration Checklist

- [ ] Account created
- [ ] API key generated
- [ ] Credentials added to n8n
- [ ] Connection tested
- [ ] Workflow built
- [ ] Error handling added
- [ ] Costs monitored
- [ ] Ready for production

---

## 📚 Resources

- [OpenAI Docs](https://platform.openai.com/docs)
- [Anthropic Docs](https://docs.anthropic.com)
- [Google AI Docs](https://ai.google/tools/)
- [Pinecone Docs](https://docs.pinecone.io)
- [Supabase Docs](https://supabase.com/docs)

---

**Level:** Beginner to Intermediate  
**Time:** 5-15 minutes per integration  
**Prerequisites:** n8n basics
