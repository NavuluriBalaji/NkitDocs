# nkit - Professional AI Agent Framework

Welcome to **nkit**, a production-grade Python framework for building sophisticated AI agents and multi-agent systems.

## 🚀 Key Features

- **🤖 Intelligent Agents** - ReAct-style agents with iterative reasoning
- **👥 Multi-Agent Coordination** - Sequential, hierarchical, and parallel execution
- **📋 Task Management** - Dependency resolution with DAG execution
- **🧠 Knowledge Base** - RAG system with embedding support
- **🔌 Plugin Architecture** - Extensible LLM, memory, and tool backends
- **📡 Event System** - Pub/sub coordination between agents
- **🎣 Lifecycle Hooks** - Pre/post execution customization
- **📊 Observability** - Built-in metrics, tracing, and cost tracking
- **🔒 Security** - Input validation and sanitization

## ⚡ Quick Example

```python
from nkit import Agent

# Create a simple agent
agent = Agent(llm=your_llm_function)

# Add a custom tool
@agent.tool("search", "Search the web")
def web_search(query: str) -> str:
    return f"Search results for: {query}"

# Run the agent
result = agent.run("Find information about AI agents")
print(result)
```

## 🎯 Use Cases

- **Research & Analysis** - Multi-step information gathering
- **Automation** - Complex workflow orchestration
- **Q&A Systems** - Tool-augmented question answering
- **Data Processing** - Iterative data analysis and transformation
- **Content Generation** - Multi-stage content creation pipelines

## 📚 Learn More

- [Installation](getting-started/installation.md) - Get started in minutes
- [Quick Start](getting-started/quick-start.md) - First agent example
- [Architecture](core-concepts/architecture.md) - Understand the design
- [Examples](examples/basic-agent.md) - Real-world examples
- [API Reference](api-reference/agent/agent.md) - Complete API docs

## 🏗️ What's Inside

### Core Components

- **Agent** - ReAct-style reasoning and tool execution
- **Tasks** - Task management with dependency resolution
- **Crews** - Multi-agent orchestration
- **LLMs** - Unified interface for multiple LLM providers
- **Memory** - Persistent and in-memory storage
- **Knowledge Base** - RAG with embeddings
- **Events** - Event bus for coordination
- **Hooks** - Lifecycle customization
- **Telemetry** - Metrics and monitoring

### Professional Features

- ✅ Full type hints
- ✅ Comprehensive logging
- ✅ Async/sync support
- ✅ Error handling & retries
- ✅ Security validation
- ✅ Cost tracking
- ✅ Distributed tracing

## 🤝 Contributing

We welcome contributions! See our [contribution guidelines](CONTRIBUTING.md) for details.

## 📄 License

MIT License - See LICENSE file for details.

## 💬 Support

- 📖 Check the [FAQ](faq.md)
- 🐛 Report issues on GitHub
- 💡 Discuss ideas in discussions

---

**Ready to build?** Start with [Installation](getting-started/installation.md) →
