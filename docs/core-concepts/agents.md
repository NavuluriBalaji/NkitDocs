# Agents

## What is an Agent?

An **Agent** is an autonomous software component that:
1. **Perceives** the environment (receives input)
2. **Reasons** about what to do (planning)
3. **Acts** by executing tools
4. **Reflects** on outcomes and iterates

## ReAct Loop

nkit uses the **ReAct** (Reasoning + Acting) pattern:

```
┌─────────────────────────────────────────┐
│         Receive Input/Task               │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│  THINK: Generate Reasoning               │
│  - Analyze the task                      │
│  - Check memory for context              │
│  - Plan approach                         │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│  ACT: Choose & Execute Tool              │
│  - Select appropriate tool               │
│  - Prepare inputs                        │
│  - Execute tool                          │
│  - Get output                            │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│  OBSERVE: Process Result                 │
│  - Evaluate tool output                  │
│  - Update memory                         │
│  - Emit events                           │
└──────────────────┬──────────────────────┘
                   │
              ┌────▼─────┐
              │ Continue? │
              └────┬──────┘
         YES   ┌───┘
              │
         NO   └──→ Return Result
                      │
                      ▼
              ┌─────────────────────┐
              │   Final Output      │
              └─────────────────────┘
```

## Creating an Agent

### Basic Agent

```python
from nkit import Agent

agent = Agent(
    name="MyBot",
    description="A helpful assistant"
)
```

### With Custom Configuration

```python
agent = Agent(
    name="ResearchBot",
    description="Researches and analyzes topics",
    
    # Model settings
    model="gpt-4",  # Default: gpt-3.5-turbo
    temperature=0.3,  # Lower = more deterministic
    
    # Execution settings
    max_iterations=10,  # Max reasoning steps
    timeout=300,  # Max seconds per execution
    
    # Features
    enable_memory=True,  # Remember conversation
    verbose=True  # Debug output
)
```

### Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `name` | str | Required | Agent identifier |
| `description` | str | "" | Agent purpose |
| `model` | str | "gpt-3.5-turbo" | LLM to use |
| `temperature` | float | 0.7 | Creativity (0=deterministic, 1=random) |
| `max_iterations` | int | 15 | Max thinking steps |
| `timeout` | int | 300 | Timeout in seconds |
| `enable_memory` | bool | False | Track conversation history |
| `memory_type` | str | "conversation" | Memory backend |
| `verbose` | bool | False | Debug output |
| `system_prompt` | str | None | Custom system instructions |

## Agent Methods

### Processing Input

#### Async (Non-blocking)

```python
import asyncio

result = await agent.aprocess(
    steps=[Step(...)],
    input="Your task or question"
)
```

#### Sync (Blocking)

```python
result = agent.process(
    steps=[Step(...)],
    input="Your task or question"
)
```

### Tool Management

```python
# Add a single tool
agent.add_tool(tool)

# Add multiple tools
agent.add_tools([tool1, tool2, tool3])

# Get available tools
tools = agent.get_tools()

# Remove a tool
agent.remove_tool("tool_name")
```

### Memory Management

```python
# Access memory
memory = agent.memory

# Add to memory
agent.memory.add("fact", "information")

# Clear memory
agent.memory.clear()

# Get memory state
state = agent.memory.get_state()
```

### Event Hooks

```python
# Register event handler
@agent.hook("on_iteration_start")
def on_iteration(step, iteration):
    print(f"Starting iteration {iteration}")

# Available events:
# - on_iteration_start
# - on_iteration_end
# - on_tool_select
# - on_tool_execute
# - on_tool_error
# - on_reasoning_complete
# - on_process_complete
```

## Agent States

An agent transitions through states during execution:

```
IDLE
  │
  ├─→ INITIALIZED → REASONING → ACTING → OBSERVING ─┐
  │                                           │      │
  └─────────────────────────────────────────←──COMPLETE
```

## Output Format

Agent output includes:

```python
{
    "status": "success",  # success, error, timeout
    "reasoning": [
        {
            "iteration": 1,
            "thought": "I need to...",
            "action": "use_tool",
            "tool_name": "search",
            "tool_input": {"query": "..."},
            "observation": "Tool returned..."
        },
        ...
    ],
    "final_answer": "The answer is...",
    "tool_calls": 3,
    "iterations": 4,
    "tokens_used": {
        "prompt": 450,
        "completion": 280,
        "total": 730
    }
}
```

## Advanced Features

### Custom System Prompt

```python
custom_prompt = """
You are an expert research assistant.
You have access to tools for searching and analyzing.
Always cite your sources.
Be concise but thorough.
"""

agent = Agent(
    name="ResearchBot",
    system_prompt=custom_prompt
)
```

### Stopping Conditions

Define when agent should stop:

```python
agent = Agent(
    name="Bot",
    max_iterations=10,  # Stop after 10 iterations
    stop_on_success=True,  # Stop when task succeeds
    stop_on_error=False  # Continue on errors
)
```

### Token Management

```python
# Track token usage
result = await agent.aprocess([step], "task")

tokens = result.get("tokens_used")
print(f"Prompt tokens: {tokens['prompt']}")
print(f"Completion tokens: {tokens['completion']}")
print(f"Total cost: ${tokens['total'] * 0.0015}")
```

### Parallel Agent Execution

```python
import asyncio

agents = [
    Agent(name="Bot1"),
    Agent(name="Bot2"),
    Agent(name="Bot3")
]

# Run all agents simultaneously
results = await asyncio.gather(*[
    agent.aprocess([step], "same task")
    for agent in agents
])
```

## Best Practices

### 1. Clear Agent Descriptions

```python
# Good
agent = Agent(
    name="ResearchBot",
    description="Conducts thorough research using search and analysis tools"
)

# Avoid
agent = Agent(
    name="Bot",
    description="Does stuff"
)
```

### 2. Appropriate Temperature

```python
# For analytical tasks
agent = Agent(model="gpt-4", temperature=0.2)

# For creative tasks
agent = Agent(model="gpt-4", temperature=0.8)
```

### 3. Tool Descriptions

```python
# Clear tool descriptions help agent choose correctly
Tool(
    name="math_calculator",
    description="Performs mathematical calculations including addition, subtraction, multiplication, and division",
    callback=calculate
)
```

### 4. Memory Management

```python
# Only enable if needed (uses tokens)
agent = Agent(
    name="ConversationBot",
    enable_memory=True
)

# Clear periodically to manage costs
agent.memory.clear()
```

### 5. Error Handling

```python
try:
    result = await agent.aprocess([step], "task")
    if result["status"] == "error":
        print(f"Agent error: {result['error']}")
except TimeoutError:
    print("Agent timed out")
```

## Troubleshooting

### Agent Loops Indefinitely

```python
# Solution: Set max_iterations
agent = Agent(
    name="Bot",
    max_iterations=5  # Limit iterations
)
```

### Tool Not Being Called

```python
# Problem: Unclear description
Tool(name="search", description="Looks up stuff")

# Solution: Be specific
Tool(
    name="search",
    description="Searches the web for information using Google Search API. Returns relevant links and summaries."
)
```

### High Token Usage

```python
# Disable memory if not needed
agent = Agent(name="Bot", enable_memory=False)

# Use cheaper model
agent = Agent(model="gpt-3.5-turbo")

# Reduce iterations
agent = Agent(max_iterations=5)
```

## See Also

- **[Tools](tools.md)** - Add capabilities to agents
- **[Memory](memory.md)** - Persistent state
- **[Tasks](tasks.md)** - Structured workflows
- **[Crews](crews.md)** - Multi-agent coordination
- **[Examples](../examples/basic-agent.md)** - Complete working examples
