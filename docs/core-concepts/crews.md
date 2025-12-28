# Crews

## What are Crews?

A **Crew** is a group of agents working together toward a common goal. Crews coordinate:

- Multiple specialized agents
- Task distribution
- Execution strategies (sequential, hierarchical, parallel)
- Context sharing
- Results aggregation

## Crew Architecture

```
┌──────────────────────┐
│      Crew            │
├──────────────────────┤
│ Manager Agent        │ (Optional - delegates work)
├──────────────────────┤
│ Worker Agents        │
│ ├── ResearchBot      │
│ ├── AnalysisBot      │
│ ├── WriterBot        │
│ └── ReviewBot        │
└──────────────────────┘
```

## Creating a Crew

### Basic Crew

```python
from nkit.crews import Crew
from nkit import Agent

# Create specialized agents
researcher = Agent(
    name="ResearchBot",
    description="Conducts research on topics"
)

analyzer = Agent(
    name="AnalysisBot",
    description="Analyzes and synthesizes information"
)

writer = Agent(
    name="WriterBot",
    description="Writes clear, well-structured content"
)

# Create crew
crew = Crew(
    name="ResearchCrew",
    description="Research and content creation team",
    agents=[researcher, analyzer, writer]
)
```

### Crew with Manager

```python
crew = Crew(
    name="SmartCrew",
    description="Research crew with intelligent management",
    agents=[researcher, analyzer, writer],
    
    # Manager agent coordinates work
    manager=Agent(
        name="ProjectManager",
        description="Coordinates research and writing tasks"
    ),
    
    # Execution strategy
    process_type="hierarchical"  # Manager delegates to workers
)
```

## Execution Strategies

### 1. Sequential

Agents work one after another:

```python
crew = Crew(
    name="PipelineCrew",
    agents=[researcher, analyzer, writer],
    process_type="sequential"
)

# Executes in order: researcher → analyzer → writer
```

```
Step 1: ResearchBot
   ↓
Step 2: AnalysisBot
   ↓
Step 3: WriterBot
```

### 2. Hierarchical

Manager delegates and coordinates:

```python
crew = Crew(
    name="ManagedCrew",
    agents=[researcher, analyzer, writer],
    manager=Agent(name="Manager"),
    process_type="hierarchical"
)

# Manager decides which agent works on what
```

```
       Manager
      /   |   \
     /    |    \
ResearchBot AnalysisBot WriterBot
```

### 3. Parallel

All agents work simultaneously:

```python
crew = Crew(
    name="ParallelCrew",
    agents=[agent1, agent2, agent3],
    process_type="parallel"
)

# All agents execute at the same time
```

```
ResearchBot ─┐
AnalysisBot ─┤─→ Results
WriterBot ──┘
```

## Using Crews

### Execute Crew

```python
import asyncio

crew = Crew(
    name="ResearchCrew",
    agents=[researcher, analyzer, writer]
)

# Run async
result = await crew.aprocess("Research machine learning trends")

# Run sync
result = crew.process("Research machine learning trends")
```

### With Tasks

```python
from nkit.tasks import Task

tasks = [
    Task(
        title="Research",
        description="Research the topic",
        agent=researcher
    ),
    Task(
        title="Analyze",
        description="Analyze findings",
        agent=analyzer
    ),
    Task(
        title="Write",
        description="Write report",
        agent=writer
    )
]

result = await crew.execute_tasks(tasks)
```

## Crew Configuration

```python
crew = Crew(
    name="ProductTeam",
    description="Team for product development",
    
    # Team composition
    agents=[pm, designer, developer, tester],
    manager=Agent(name="Lead"),
    
    # Execution
    process_type="hierarchical",  # sequential, hierarchical, parallel
    max_iterations=10,
    timeout=3600,
    
    # Communication
    verbose=True,
    
    # Memory
    shared_memory=True  # All agents share memory
)
```

## Context Sharing

### Shared Memory

```python
# All agents access same memory
crew = Crew(
    name="Crew",
    agents=[agent1, agent2, agent3],
    shared_memory=True
)

# Agent1 stores fact
agent1.memory.add("key", "value")

# Agent2 can access it
fact = agent2.memory.get("key")
```

### Context Passing

```python
# Pass context between agents
context = {
    "project": "ResearchProject",
    "deadline": "2024-01-15",
    "budget": 10000,
    "stakeholders": ["CEO", "Board"]
}

result = await crew.aprocess("Your task", context=context)
```

## Agent Roles

Define what each agent specializes in:

```python
crew = Crew(
    name="TeamA",
    agents=[
        Agent(
            name="Researcher",
            description="Gathers and synthesizes information",
            role="information_gatherer"
        ),
        Agent(
            name="Analyst",
            description="Analyzes data and identifies patterns",
            role="analyst"
        ),
        Agent(
            name="Strategist",
            description="Develops recommendations and strategies",
            role="strategist"
        )
    ]
)
```

## Advanced Features

### Custom Process

```python
from nkit.crews import Crew

class CustomCrew(Crew):
    """Custom crew with special logic"""
    
    async def execute_custom(self, task):
        # Custom execution logic
        results = []
        for agent in self.agents:
            # Custom coordination
            result = await agent.aprocess([task], ...)
            results.append(result)
        return results

crew = CustomCrew(name="Custom", agents=[...])
```

### Callback on Completion

```python
def on_crew_complete(crew, result):
    print(f"Crew {crew.name} completed")
    print(f"Result: {result}")
    # Notify stakeholders, save to database, etc.

crew = Crew(
    name="NotifyCrew",
    agents=[...],
    on_complete=on_crew_complete
)
```

## Crew Monitoring

### Track Crew Metrics

```python
result = await crew.aprocess("task")

print(f"Status: {result['status']}")
print(f"Agents used: {len(result['agents_used'])}")
print(f"Total iterations: {result['total_iterations']}")
print(f"Total cost: ${result['total_cost']}")
print(f"Duration: {result['duration']}s")
```

### Crew Hooks

```python
@crew.hook("on_agent_switch")
def log_switch(from_agent, to_agent):
    print(f"Switched from {from_agent.name} to {to_agent.name}")

@crew.hook("on_task_assign")
def log_assignment(agent, task):
    print(f"Assigned {task.title} to {agent.name}")
```

## Best Practices

### 1. Clear Specialization

```python
# Good - each agent has specific role
crew = Crew(
    name="Crew",
    agents=[
        Agent(name="Researcher", description="Researches topics"),
        Agent(name="Analyst", description="Analyzes data"),
        Agent(name="Writer", description="Writes reports")
    ]
)

# Avoid - overlapping roles
crew = Crew(
    name="Crew",
    agents=[
        Agent(name="Bot1", description="Does everything"),
        Agent(name="Bot2", description="Does everything")
    ]
)
```

### 2. Appropriate Strategy

```python
# Sequential for dependent tasks
sequential_crew = Crew(
    name="Pipeline",
    agents=[researcher, analyzer, writer],
    process_type="sequential"
)

# Parallel for independent tasks
parallel_crew = Crew(
    name="Independent",
    agents=[agent1, agent2, agent3],
    process_type="parallel"
)

# Hierarchical for complex coordination
hierarchical_crew = Crew(
    name="Managed",
    agents=[...],
    manager=manager,
    process_type="hierarchical"
)
```

### 3. Resource Management

```python
# Limit concurrent agents
crew = Crew(
    name="Crew",
    agents=[...],
    max_concurrent=2  # Max 2 agents at once
)

# Timeout per agent
for agent in crew.agents:
    agent.timeout = 300
```

### 4. Error Handling

```python
try:
    result = await crew.aprocess("task")
    if result["status"] == "failed":
        # Handle failure
        print(f"Failed agents: {result['failed_agents']}")
except Exception as e:
    print(f"Crew error: {e}")
```

## Troubleshooting

### Crew Deadlocked

```python
# Check for circular dependencies
crew.validate_dependencies()

# Increase timeout
crew.timeout = 600

# Reduce complexity
crew.max_iterations = 5
```

### Uneven Work Distribution

```python
# Check agent workload
for agent in crew.agents:
    print(f"{agent.name}: {agent.task_count} tasks")

# Rebalance tasks
crew.rebalance_tasks()
```

### Communication Issues

```python
# Enable debug mode
crew.verbose = True

# Check shared memory
crew.shared_memory.debug()
```

## Examples

- **[Crew Example](../examples/crew.md)** - Complete crew implementation
- **[Multi-Agent RAG](../examples/rag.md)** - Crew for information retrieval
- **[Task Orchestration](../examples/tasks.md)** - Complex workflows

## See Also

- **[Agents](agents.md)** - Individual agent design
- **[Tasks](tasks.md)** - Task management
- **[Examples](../examples/crew.md)** - Working examples
