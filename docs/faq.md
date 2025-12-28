# FAQ

Frequently Asked Questions about nkit framework.

## General Questions

### What is nkit?

nkit is a Python framework for building intelligent agents and multi-agent systems. It provides tools for:
- Creating autonomous agents with reasoning capabilities
- Orchestrating multiple agents (crews)
- Managing tasks and workflows
- Integrating with LLMs and external tools
- Tracking memory and conversation history

### How is nkit different from other frameworks?

| Feature | nkit | LangChain | Crew AI |
|---------|------|-----------|---------|
| **Async First** | ✅ Native async | ⚠️ Partial | ✅ Full |
| **ReAct Pattern** | ✅ Built-in | ⚠️ Chains | ✅ Tasks |
| **Task DAGs** | ✅ Full support | ⚠️ Chains | ✅ Full support |
| **Multi-Agent** | ✅ Crews | ✅ Agents | ✅ Full |
| **Memory Types** | ✅ Multiple | ✅ Vector | ✅ Basic |
| **Event System** | ✅ Hooks | ❌ No | ⚠️ Limited |

### Is nkit production-ready?

Yes, nkit is designed for production use with:
- Comprehensive error handling
- Resource limits and timeouts
- Token counting and cost tracking
- Audit logging capabilities
- Scalable architecture

### Can I use nkit with local models?

Yes! nkit supports:
- **Ollama** - Run models locally
- **Custom LLM adapters** - Bring your own model
- **Hugging Face** - Via custom adapters

```python
agent = Agent(
    name="LocalBot",
    model="ollama:llama2"
)
```

## Getting Started

### How do I install nkit?

```bash
pip install nkit
```

For LLM support:
```bash
pip install nkit openai anthropic  # Your choice of providers
```

### What's the simplest agent example?

```python
from nkit import Agent

agent = Agent(name="MyBot")
result = agent.process([], "Hello, world!")
print(result)
```

### How do I add tools to an agent?

```python
from nkit.tools import Tool

def calculator(a: float, b: float) -> float:
    return a + b

tool = Tool(
    name="add",
    description="Add two numbers",
    callback=calculator
)

agent.add_tool(tool)
```

### How do I use async?

```python
import asyncio

async def main():
    result = await agent.aprocess(steps, "Your task")
    return result

asyncio.run(main())
```

## Agent Configuration

### What's the difference between temperature settings?

```python
# Low temperature = deterministic (good for analysis)
agent = Agent(temperature=0.2)  # More consistent

# High temperature = creative (good for generation)
agent = Agent(temperature=0.8)  # More varied
```

### How many iterations should I set?

It depends on task complexity:

```python
# Simple tasks
agent = Agent(max_iterations=3)

# Complex tasks
agent = Agent(max_iterations=10)

# Very complex reasoning
agent = Agent(max_iterations=20)
```

### Should I enable memory?

Enable memory if:
- ✅ Building conversational agents
- ✅ Tasks need context from previous interactions
- ❌ Processing independent queries

```python
# Conversational
agent = Agent(enable_memory=True)

# Single queries
agent = Agent(enable_memory=False)
```

## Tools and Integration

### How do I create a custom tool?

```python
from nkit.tools import Tool

def my_tool(input_param: str) -> str:
    """Do something useful"""
    return f"Processed: {input_param}"

tool = Tool(
    name="my_tool",
    description="What this tool does",
    callback=my_tool,
    input_schema={...}
)

agent.add_tool(tool)
```

### Can tools be async?

Yes! Use async functions:

```python
async def fetch_data(url: str) -> dict:
    """Fetch data from API"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.json()

tool = Tool(
    name="fetch",
    description="Fetch data from URL",
    callback=fetch_data,
    is_async=True
)
```

### How do I validate tool inputs?

Use JSON Schema:

```python
Tool(
    name="tool",
    description="...",
    callback=my_func,
    input_schema={
        "type": "object",
        "properties": {
            "email": {
                "type": "string",
                "pattern": "^[^@]+@[^@]+\\.[^@]+$"
            }
        },
        "required": ["email"]
    }
)
```

### Can I limit tool access?

Yes! Use conditions:

```python
def requires_admin(context: dict) -> bool:
    return context.get("user_role") == "admin"

tool = Tool(
    name="delete_all",
    description="Delete everything (admin only)",
    callback=delete_all,
    condition=requires_admin
)
```

## Tasks and Workflows

### How do I create task dependencies?

```python
task_a = Task(title="Step 1", agent=agent)
task_b = Task(title="Step 2", agent=agent, depends_on=[task_a])
task_c = Task(title="Step 3", agent=agent, depends_on=[task_b])

manager = TaskManager()
results = await manager.execute([task_a, task_b, task_c])
```

### Can tasks run in parallel?

Yes! Independent tasks run in parallel:

```python
task_a = Task(title="Fetch A", agent=agent)
task_b = Task(title="Fetch B", agent=agent)
task_c = Task(title="Fetch C", agent=agent)
# All run simultaneously if no dependencies

merge = Task(title="Merge", agent=agent, depends_on=[task_a, task_b, task_c])
```

### How do I handle task failures?

```python
task = Task(
    title="Risky Operation",
    agent=agent,
    retries=3,  # Retry 3 times
    timeout=30,  # 30 seconds max
    on_error="continue"  # Continue on error
)
```

## Crews and Multi-Agent

### How do I create a crew?

```python
from nkit.crews import Crew

crew = Crew(
    name="TeamA",
    agents=[agent1, agent2, agent3],
    process_type="sequential"
)

result = await crew.aprocess("Your task")
```

### What execution strategies are available?

```python
# Sequential - one after another
Crew(agents=[...], process_type="sequential")

# Hierarchical - manager coordinates
Crew(agents=[...], manager=manager, process_type="hierarchical")

# Parallel - all at once
Crew(agents=[...], process_type="parallel")
```

### Can crews share memory?

Yes! Enable shared memory:

```python
crew = Crew(
    agents=[...],
    shared_memory=True
)

# Agent 1 stores fact
crew.agents[0].memory.add("key", "value")

# Agent 2 retrieves it
value = crew.agents[1].memory.get("key")
```

## Memory and State

### What memory types are available?

```python
# Conversation history
from nkit.memory import ConversationMemory
agent = Agent(memory=ConversationMemory())

# Vector embeddings
from nkit.memory import VectorMemory
agent = Agent(memory=VectorMemory())

# File-based persistence
from nkit.memory import FileMemory
agent = Agent(memory=FileMemory("./memory.json"))

# Custom implementation
agent = Agent(memory=CustomMemory())
```

### How do I clear memory?

```python
agent.memory.clear()  # Clear all

agent.memory.prune(keep_last=10)  # Keep last 10 messages

agent.memory.compress(strategy="summarize")  # Compress old messages
```

### Does memory consume tokens?

Yes, memory is included in LLM context:

```python
# Check token usage
tokens = agent.memory.get_token_count()

# Monitor and clear periodically
if tokens > 2000:
    agent.memory.prune(keep_last=5)
```

## Monitoring and Debugging

### How do I debug agent behavior?

```python
agent = Agent(
    name="Bot",
    verbose=True  # Enable debug output
)

# Or use hooks
@agent.hook("on_iteration_start")
def debug_iter(step, iteration):
    print(f"Iteration {iteration}: {step.title}")
```

### How do I track costs?

```python
result = await agent.aprocess([step], "task")

tokens = result["tokens_used"]
cost = (tokens["prompt"] * 0.0015 + tokens["completion"] * 0.002) / 1000
print(f"Cost: ${cost:.4f}")
```

### Can I log all events?

```python
@agent.hook("on_iteration_start")
def log_event(step, iteration):
    logger.info(f"Start iteration {iteration}")

@agent.hook("on_tool_execute")
def log_tool(tool, input_data, output):
    logger.info(f"Tool: {tool.name} = {output}")

@agent.hook("on_process_complete")
def log_complete(result):
    logger.info(f"Status: {result['status']}")
```

## Performance and Optimization

### Agent loops forever. What's wrong?

```python
# Set max iterations limit
agent = Agent(
    name="Bot",
    max_iterations=5  # Stop after 5 iterations
)
```

### Tool isn't being called. Why?

```python
# Problem: Unclear description
Tool(name="search", description="Searches stuff")

# Solution: Specific description
Tool(
    name="search",
    description="Search the web for current information about topics",
    callback=search
)
```

### High token usage. How do I reduce it?

```python
# 1. Disable memory if not needed
agent = Agent(enable_memory=False)

# 2. Use cheaper model
agent = Agent(model="gpt-3.5-turbo")

# 3. Limit iterations
agent = Agent(max_iterations=3)

# 4. Clear memory periodically
agent.memory.prune(keep_last=10)
```

### How do I speed up execution?

```python
# 1. Use parallel tasks
crew = Crew(agents=[...], process_type="parallel")

# 2. Reduce max_iterations
agent = Agent(max_iterations=3)

# 3. Use async/await
result = await agent.aprocess([step], "task")

# 4. Optimize tools
# Use fast, cached tools instead of API calls
```

## Troubleshooting

### Agent times out. What to do?

```python
# Increase timeout
agent = Agent(timeout=600)

# Or increase per-tool
task = Task(title="...", agent=agent, timeout=300)

# Check tool speed
@agent.hook("on_tool_execute")
def check_speed(tool, input_data, output):
    print(f"Tool {tool.name} time: ...")
```

### "API key not found" error

```bash
# Set environment variable
export OPENAI_API_KEY="sk-..."

# Or in code
import os
os.environ["OPENAI_API_KEY"] = "sk-..."
```

### ImportError for optional dependencies

Install required packages:

```bash
# For OpenAI
pip install openai

# For Claude
pip install anthropic

# For embeddings
pip install sentence-transformers
```

## Advanced Questions

### Can I use custom LLM providers?

Yes! Implement the adapter:

```python
from nkit.llms import BaseLLMAdapter

class MyLLMAdapter(BaseLLMAdapter):
    def __init__(self, api_key):
        self.api_key = api_key
    
    async def agenerate(self, prompt, **kwargs):
        # Call your LLM
        return response

agent = Agent(
    name="Bot",
    llm=MyLLMAdapter(api_key="...")
)
```

### How do I scale to many agents?

```python
# Use process pools
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(agent.process, steps, task) 
               for task in tasks]
    results = [f.result() for f in futures]
```

### Can I deploy to cloud?

Yes! nkit works with:
- **Docker** - Containerize your agent
- **AWS Lambda** - Serverless agents
- **Google Cloud** - Cloud Functions
- **Azure** - Function Apps

## Getting Help

### Where's the documentation?

- 📚 [Full Documentation](../index.md)
- 🤖 [API Reference](../api-reference/index.md)
- 💡 [Examples](../examples/index.md)
- 🎓 [Tutorials](../getting-started/index.md)

### I found a bug. What do I do?

Report it on [GitHub Issues](https://github.com/yourusername/nkit/issues)

Include:
- Python version
- nkit version
- Error message
- Minimal reproduction code

### How do I contribute?

See [Contributing Guide](contributing.md)

### I have another question

Feel free to reach out:
- 📧 Email: support@nkit.dev
- 💬 Discord: [Community Server](https://discord.gg/nkit)
- 🐦 Twitter: [@nkitdev](https://twitter.com/nkitdev)
