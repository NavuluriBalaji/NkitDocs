# Architecture

## Overview

nkit follows a **modular, plugin-based architecture** designed for extensibility and clean separation of concerns.

```
┌─────────────────────────────────────────────────────┐
│                    Application                       │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                    Crews                            │
│         (Multi-agent Orchestration)                 │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                    Tasks                            │
│         (Unit of Work with Dependencies)            │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                   Agent                             │
│      (ReAct-style Reasoning Loop)                   │
└──────────────────────┬──────────────────────────────┘
       ┌──────────────┬──────────────┬────────────────┐
       │              │              │                │
   ┌───▼───┐  ┌─────▼─────┐  ┌────▼────┐  ┌──────▼─────┐
   │ Tools │  │  Memory   │  │   LLMs  │  │  Events    │
   └───────┘  │(History)  │  │(OpenAI) │  │(Hooks)     │
              └───────────┘  └─────────┘  └────────────┘
```

## Core Components

### 1. **Agent** 🤖

The intelligent executor using **ReAct** (Reasoning + Acting) pattern:

```python
Agent
├── Reasoning Step
│   └── Analyze task, decide action
├── Tool Selection
│   └── Choose appropriate tool
├── Tool Execution
│   └── Run selected tool
└── Reflection
    └── Evaluate result, continue or stop
```

**Key Features:**
- Async/sync execution
- Tool orchestration
- Memory integration
- Token counting
- Iterative reasoning

### 2. **Tasks** 📋

Discrete units of work with:
- Dependency management (DAG)
- Priority levels
- Retry policies
- Timeout handling

```python
Task A → Task B → Task C  # Sequential
    ↓
Task D → Task E           # Parallel after A
```

### 3. **Crews** 👥

Multi-agent coordination:
- **Sequential**: Agents work one after another
- **Hierarchical**: Manager delegates to workers
- **Parallel**: Independent concurrent execution
- Context sharing between agents

### 4. **Tools** 🔧

Extensible capability system:
- Built-in tools (math, file I/O, web)
- Custom tool registration
- Async/sync support
- Input validation via schemas
- Error handling

### 5. **Memory** 🧠

Persistent state management:
- **Conversation memory**: Chat history
- **Entity memory**: Important facts
- **Long-term memory**: Vector embeddings
- **Custom backends**: Database integration

### 6. **LLMs** 🧠

Pluggable language models:
- OpenAI (GPT-3.5, GPT-4)
- Anthropic (Claude)
- Ollama (local models)
- Custom providers

### 7. **Events** 📡

Pub/sub system for:
- Agent lifecycle events
- Tool execution tracking
- Error notifications
- Metrics collection

## Module Organization

```
nkit/
├── agent/              # Core Agent class
│   └── core.py
├── tasks/              # Task management
├── crews/              # Multi-agent coordination
├── tools/              # Tool system
├── memory/             # State management
├── llms/               # LLM adapters
├── knowledge/          # RAG & embeddings
├── events/             # Event system
├── hooks/              # Plugin points
├── chain/              # Execution chains
├── telemetry/          # Metrics & tracing
└── utils.py            # Helper functions
```

## Design Patterns

### 1. **Dependency Injection**

Components receive dependencies instead of creating them:

```python
agent = Agent(
    name="Bot",
    llm=OpenAIAdapter(...),
    memory=ConversationMemory(...),
    tools=[...],
    event_bus=EventBus()
)
```

### 2. **Plugin System**

Extend via hooks without modifying core:

```python
@agent.hook("on_tool_execute")
def log_tool_use(tool, input, output):
    print(f"Tool {tool.name} executed")
```

### 3. **Strategy Pattern**

Swap implementations at runtime:

```python
# Use GPT-4 for quality
agent1 = Agent(llm=GPT4Adapter())

# Use local model for speed
agent2 = Agent(llm=OllamaAdapter())
```

### 4. **Builder Pattern**

Fluent configuration:

```python
agent = (Agent("Bot")
    .with_llm(GPT4Adapter())
    .with_memory(ConversationMemory())
    .with_tools([tool1, tool2])
    .with_max_iterations(10)
)
```

## Data Flow

### Agent Processing Flow

```
Input Query
    │
    ▼
Plan Reasoning
    │
    ├─→ Analyze task
    ├─→ Check memory
    └─→ Determine approach
    │
    ▼
Tool Selection
    │
    ├─→ Available tools
    ├─→ Tool descriptions
    └─→ Choose best tool
    │
    ▼
Tool Execution
    │
    ├─→ Validate inputs
    ├─→ Execute tool
    ├─→ Capture output
    └─→ Emit event
    │
    ▼
Reflection
    │
    ├─→ Evaluate result
    ├─→ Update memory
    ├─→ Check stop condition
    └─→ Continue or finish
    │
    ▼
Final Response
```

### Task Execution (DAG)

```
Start
  │
  ├─→ Task A ─────┐
  │               │
  ├─→ Task B ─────┤─→ Task D
  │               │
  ├─→ Task C ─────┘
  │
  └─→ End
```

## Execution Models

### 1. **Synchronous**

```python
result = agent.process(steps, "Task")  # Blocks
```

### 2. **Asynchronous**

```python
result = await agent.aprocess(steps, "Task")  # Non-blocking
```

### 3. **Batch Processing**

```python
results = await crew.process_batch(
    [(step1, input1), (step2, input2)]
)
```

## Configuration Hierarchy

```
Global Config
    │
    ├─→ Crew Config
    │   └─→ Agent Config
    │       └─→ Tool Config
    │
    └─→ Environment Variables
```

## Performance Considerations

### 1. **Caching**

- Tool results cached by default
- Memory lookups optimized with indexing
- LLM response caching available

### 2. **Concurrency**

- Parallel task execution
- Async/await throughout
- Connection pooling for APIs

### 3. **Resource Limits**

- Token limits per request
- Max iterations per agent
- Timeout per tool execution

## Security

### 1. **Input Validation**

All tool inputs validated via JSON schemas:

```python
Tool(
    name="execute",
    input_schema={
        "type": "object",
        "properties": {"cmd": {"type": "string"}},
        "required": ["cmd"]
    }
)
```

### 2. **Sandboxing**

Tools execute in isolated context:
- No direct file system access
- Network requests controlled
- Memory limits enforced

### 3. **Audit Logging**

All actions logged:
- Tool execution
- Memory access
- LLM calls

## Next Steps

- **[Agents](agents.md)** - Deep dive into agent design
- **[Tools](tools.md)** - Building custom tools
- **[Memory](memory.md)** - State management strategies
- **[Tasks & Workflows](tasks.md)** - Complex task orchestration
- **[Crews](crews.md)** - Multi-agent systems
