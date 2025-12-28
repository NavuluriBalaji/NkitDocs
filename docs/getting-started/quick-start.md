# Quick Start

Get your first agent running in **5 minutes** ⚡

## 1. Install nkit

```bash
pip install nkit openai
```

## 2. Set Your API Key

```bash
export OPENAI_API_KEY="sk-..."
```

Or set it in Python:

```python
import os
os.environ["OPENAI_API_KEY"] = "sk-..."
```

## 3. Create Your First Agent

```python
from nkit import Agent, Step
import asyncio

# Define agent behavior
agent = Agent(
    name="ResearchBot",
    description="An agent that researches topics"
)

# Create a step (unit of work)
research_step = Step(
    title="Research the topic",
    description="Search for information about the given topic",
)

# Run the agent
async def main():
    result = await agent.aprocess([research_step], "What is machine learning?")
    print(result)

# Execute
asyncio.run(main())
```

**Output:**
```
Agent ResearchBot processed your query...
Machine learning is a subset of artificial intelligence...
```

## 4. Add Tools

Give your agent the ability to use external services:

```python
from nkit import Agent, Step
from nkit.tools import Tool
import asyncio

# Create a simple tool
def calculate(operation: str, a: float, b: float) -> float:
    """Perform basic math operations"""
    if operation == "add":
        return a + b
    elif operation == "multiply":
        return a * b
    return 0

# Register with agent
agent = Agent(name="MathBot")
agent.add_tool(Tool(
    name="calculator",
    description="Perform math operations",
    callback=calculate
))

# Use it
async def main():
    result = await agent.aprocess(
        [Step(title="Calculate", description="5 + 3")],
        "Calculate 5 + 3"
    )
    print(result)

asyncio.run(main())
```

## 5. Create a Multi-Step Workflow

```python
from nkit import Agent, Step

agent = Agent(name="WorkflowBot")

steps = [
    Step(title="Gather Data", description="Collect information"),
    Step(title="Analyze", description="Analyze the data"),
    Step(title="Report", description="Generate report")
]

# Sequential execution
import asyncio
result = asyncio.run(agent.aprocess(steps, "Analyze quarterly sales"))
```

## 6. Next Steps

✨ **Learn More:**
- **[Your First Agent](first-agent.md)** - Detailed step-by-step guide
- **[Tools Guide](../core-concepts/tools.md)** - How to create custom tools
- **[Tasks & Workflows](../core-concepts/tasks.md)** - Build complex workflows
- **[Examples](../examples/basic-agent.md)** - Ready-to-run examples

## Common Patterns

### Synchronous Execution

```python
# Use process() instead of aprocess()
result = agent.process(steps, "Your task")
```

### With Memory

```python
agent = Agent(
    name="Bot",
    enable_memory=True  # Remembers conversation history
)
```

### Custom Settings

```python
agent = Agent(
    name="CustomBot",
    model="gpt-4",  # Use GPT-4
    temperature=0.7,  # More creative
    max_iterations=10  # Limit reasoning steps
)
```

## Troubleshooting

**Agent not responding?**
- Check your API key is set
- Verify internet connection
- See [Troubleshooting](../advanced/troubleshooting.md)

**Want to learn more?**
- See [Core Concepts](../core-concepts/index.md)
- Check [API Reference](../api-reference/index.md)

---

**Ready? Let's build something! → [Your First Agent](first-agent.md)**
