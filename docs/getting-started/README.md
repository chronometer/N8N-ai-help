# Getting Started with n8n

Welcome! This section will help you get n8n up and running and build your first AI workflow.

## 📖 Contents

### [Installation Guide](installation.md)
Choose your deployment method:
- **n8n Cloud** - Managed hosting (fastest way to start)
- **npm** - Local development
- **Docker** - Containerized deployment
- **Self-hosted** - Full control (various platforms)

### [Quick Start](quickstart.md)
A quick overview of n8n concepts and the interface before diving in.

### [Your First AI Workflow](first-ai-workflow.md)
Step-by-step tutorial: Build a simple AI workflow that uses OpenAI to summarize text.

## 🚀 Quick Start Path

### 1. Choose an Installation Method (5 min)
- **Fastest**: Use n8n Cloud (sign up and start immediately)
- **Development**: Use npm for local testing
- **Production**: Docker or self-hosted

See [Installation Guide](installation.md)

### 2. Understand the Basics (10 min)
- Workflows = automated sequences of actions
- Nodes = individual tasks/integrations
- Connections = flow of data between nodes
- Triggers = start your workflow

See [Quick Start](quickstart.md)

### 3. Build Your First Workflow (20 min)
Follow the tutorial to create a simple AI workflow.

See [Your First AI Workflow](first-ai-workflow.md)

### 4. Explore Further
Once you're comfortable, explore:
- [Core Concepts](../core-concepts/) - Understand the fundamentals
- [AI Workflows](../ai-workflows/) - Learn AI-specific features
- [Key Nodes](../key-nodes/) - Deep dive into important nodes
- [Examples](../examples/) - See more workflow examples

## 💡 Key Concepts

### Workflows
A workflow is a series of connected nodes that perform automated tasks. Workflows can be triggered manually, on a schedule, or by external events.

### Nodes
Nodes are the building blocks of workflows. Each node performs a specific action:
- **Trigger nodes** - Start the workflow
- **Action nodes** - Do something (API calls, code execution, etc.)
- **Data processing nodes** - Transform or filter data
- **Integration nodes** - Connect with external services

### Data Flow
Data flows through nodes from left to right. Each node receives input from previous nodes and passes output to the next.

### Connections
Connections link nodes together. The output of one node becomes the input to another.

## 🎯 Common First Steps

1. **Explore the UI** - Get familiar with the workflow editor
2. **Set up credentials** - Add API keys for services you'll use
3. **Create a test workflow** - Start simple
4. **Run and debug** - Execute and watch the data flow
5. **Iterate** - Modify and improve

## ⚙️ System Requirements

- **Browser**: Modern browser (Chrome, Firefox, Safari, Edge)
- **Node.js** (if running locally): v16 or higher
- **Internet connection**: For cloud or API-based workflows
- **Credentials**: API keys for services you want to integrate

## 🔗 Resources

- [Official Installation Docs](https://docs.n8n.io/hosting/)
- [n8n Community Slack](https://n8n.io/slack)
- [Video Tutorials](https://docs.n8n.io/getting-started/learning-path/video-courses/)

## 📝 Next Steps

Choose your starting point:
- **Ready to install?** → [Installation Guide](installation.md)
- **Want to learn first?** → [Quick Start](quickstart.md)
- **Let's build!** → [Your First AI Workflow](first-ai-workflow.md)

---

**Time to complete this section:** ~35 minutes  
**Prerequisites:** None  
**Next section:** [Core Concepts](../core-concepts/)
