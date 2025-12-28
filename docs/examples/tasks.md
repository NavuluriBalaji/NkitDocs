# Task Example

Example of tasks with dependencies, parallel execution, and complex workflows.

## Full Code

```python
"""
Task Example: Workflow with Dependencies
Demonstrates task DAG (Directed Acyclic Graph) execution
"""

import asyncio
from nkit.tasks import Task, TaskManager
from nkit import Agent

# Create agents for different tasks
def create_agents():
    data_agent = Agent(
        name="DataAgent",
        description="Handles data gathering",
        model="gpt-3.5-turbo"
    )
    
    analysis_agent = Agent(
        name="AnalysisAgent",
        description="Performs analysis",
        model="gpt-3.5-turbo"
    )
    
    viz_agent = Agent(
        name="VisualizationAgent",
        description="Creates visualizations",
        model="gpt-3.5-turbo"
    )
    
    report_agent = Agent(
        name="ReportAgent",
        description="Generates reports",
        model="gpt-3.5-turbo"
    )
    
    return data_agent, analysis_agent, viz_agent, report_agent

async def main():
    print("=" * 70)
    print("TASK WORKFLOW EXAMPLE - DAG EXECUTION")
    print("=" * 70)
    
    data_agent, analysis_agent, viz_agent, report_agent = create_agents()
    
    # Create task DAG
    # gather_data runs first
    gather_task = Task(
        title="Gather Data",
        description="Collect raw data from sources",
        agent=data_agent,
        timeout=300,
        retries=2
    )
    
    # Both analysis and viz depend on gather_task
    # They run in parallel after gather completes
    analysis_task = Task(
        title="Analyze Data",
        description="Perform statistical analysis",
        agent=analysis_agent,
        depends_on=[gather_task],
        timeout=300,
        priority=1
    )
    
    viz_task = Task(
        title="Create Visualizations",
        description="Generate charts and graphs",
        agent=viz_agent,
        depends_on=[gather_task],
        timeout=300,
        priority=1
    )
    
    # Report depends on both analysis and viz
    report_task = Task(
        title="Generate Report",
        description="Compile final report",
        agent=report_agent,
        depends_on=[analysis_task, viz_task],
        timeout=300,
        expected_output="Comprehensive report with analysis and visualizations"
    )
    
    # Execute all tasks respecting dependencies
    manager = TaskManager()
    
    print("\n📋 Task Graph:")
    print("   gather_task")
    print("    ├─→ analysis_task ─┐")
    print("    └─→ viz_task ──────┤─→ report_task")
    print("\n")
    
    results = await manager.execute([
        gather_task,
        analysis_task,
        viz_task,
        report_task
    ])
    
    print("\n✅ Task Execution Complete\n")
    for i, result in enumerate(results, 1):
        print(f"Task {i}:")
        print(f"  Status: {result.get('status')}")
        print(f"  Duration: {result.get('duration', 'N/A')}s")
        print(f"  Result: {result.get('result', 'N/A')}\n")

if __name__ == "__main__":
    asyncio.run(main())
```

## Running the Example

```bash
pip install nkit openai
export OPENAI_API_KEY="sk-..."
python tasks_example.py
```

## Task Graph Visualization

```
Phase 1:
  ┌─────────────────┐
  │  gather_task    │
  └────────┬────────┘
           │
Phase 2: (Parallel)
   ┌───────┴───────┐
   │               │
┌──▼─────┐   ┌────▼───────┐
│ analysis│   │ visualize  │
└──┬──────┘   └────┬───────┘
   │               │
Phase 3:
   └───────┬───────┘
           │
       ┌───▼────────┐
       │  report    │
       └────────────┘
```

## Key Concepts

### Task Dependencies

```python
# Task A must complete before B starts
task_b = Task(
    title="Task B",
    description="...",
    agent=agent,
    depends_on=[task_a]
)
```

### Parallel Tasks

```python
# Both run simultaneously (no dependencies on each other)
task_a = Task(title="Task A", agent=agent)
task_b = Task(title="Task B", agent=agent)

# But both depend on task_0
# So: task_0 → (task_a | task_b) → task_c
```

### Task Configuration

```python
task = Task(
    title="Important Task",
    description="Do something critical",
    agent=agent,
    timeout=600,      # 10 minutes max
    retries=3,        # Retry 3 times
    priority=1,       # Higher priority = execute first
    depends_on=[...], # Prerequisites
    expected_output="CSV file with results"
)
```

## Advanced Patterns

### Conditional Task Execution

```python
def should_continue(prev_result):
    """Execute next task only if condition met"""
    return prev_result.get("quality_score", 0) > 0.8

conditional_task = Task(
    title="Premium Processing",
    description="Run expensive processing",
    agent=agent,
    condition=should_continue
)
```

### Task Retries

```python
task = Task(
    title="API Call",
    description="Call external API",
    agent=agent,
    retries=5,  # Retry 5 times on failure
    timeout=30   # Each attempt has 30s timeout
)
```

### Task Callbacks

```python
def on_complete(task, result):
    print(f"Task {task.title} completed!")
    # Update database, send notification, etc.

task = Task(
    title="Save to DB",
    description="...",
    agent=agent,
    on_complete=on_complete
)
```

## Complex Workflows

### Multi-Stage Pipeline

```python
# Stage 1: Prepare
stage1 = Task(title="Prepare", agent=agent1)

# Stage 2: Process (parallel)
stage2a = Task(title="Process A", agent=agent2, depends_on=[stage1])
stage2b = Task(title="Process B", agent=agent2, depends_on=[stage1])

# Stage 3: Aggregate
stage3 = Task(
    title="Aggregate",
    agent=agent3,
    depends_on=[stage2a, stage2b]
)

# Stage 4: Finalize
stage4 = Task(title="Finalize", agent=agent4, depends_on=[stage3])
```

### Branching Workflows

```python
# Main task
main_task = Task(title="Main", agent=agent)

# Branch A
branch_a = Task(title="Path A", agent=agent, depends_on=[main_task])
branch_a_cont = Task(title="Path A Cont", agent=agent, depends_on=[branch_a])

# Branch B
branch_b = Task(title="Path B", agent=agent, depends_on=[main_task])

# Merge
merge_task = Task(
    title="Merge",
    agent=agent,
    depends_on=[branch_a_cont, branch_b]
)
```

## Monitoring Tasks

### Track Execution

```python
manager = TaskManager()

@manager.hook("on_task_start")
def log_start(task):
    print(f"▶️  Started: {task.title}")

@manager.hook("on_task_complete")
def log_complete(task):
    print(f"✅ Completed: {task.title} ({task.duration}s)")

@manager.hook("on_task_error")
def log_error(task, error):
    print(f"❌ Failed: {task.title} - {error}")

results = await manager.execute(tasks)
```

## Best Practices

### 1. Clear Task Descriptions

```python
# Good
Task(
    title="Validate Email Format",
    description="Check if email matches pattern and is not already registered"
)

# Avoid
Task(
    title="Check",
    description="Do validation"
)
```

### 2. Appropriate Timeouts

```python
# Quick task
quick = Task(title="Calculate", agent=agent, timeout=5)

# Long task
long = Task(title="BigData Processing", agent=agent, timeout=600)
```

### 3. Effective Parallelization

Identify truly independent tasks:

```python
# Good - truly independent
independent_tasks = [
    Task(title="Fetch Data A", agent=agent),
    Task(title="Fetch Data B", agent=agent),
    Task(title="Fetch Data C", agent=agent)
]

# Then merge results
merge = Task(
    title="Merge Data",
    agent=agent,
    depends_on=independent_tasks
)
```

## Next Steps

- **[RAG Example](rag.md)** - Knowledge base integration
- **[Events Example](events.md)** - Event-driven workflows
- **[Core Concepts](../core-concepts/tasks.md)** - Detailed guide
- **[Advanced Patterns](../advanced/task-patterns.md)** - Expert techniques
