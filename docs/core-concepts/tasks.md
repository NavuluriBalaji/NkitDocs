# Tasks

## What are Tasks?

**Tasks** are discrete units of work that agents perform. They can be organized into workflows with:

- Dependencies between tasks
- Parallel execution
- Retry policies
- Timeout handling
- Priority levels

## Task Structure

```
Task
├── Title: What to do
├── Description: How to do it
├── Agent: Who executes it
├── Dependencies: Which tasks must complete first
├── Retries: How many times to retry on failure
├── Timeout: Max execution time
└── Priority: Execution priority
```

## Creating Tasks

### Simple Task

```python
from nkit.tasks import Task
from nkit import Agent

agent = Agent(name="Worker")

task = Task(
    title="Analyze Data",
    description="Analyze the provided dataset and generate insights",
    agent=agent,
)
```

### Task with Configuration

```python
task = Task(
    title="Process Data",
    description="Clean and transform raw data",
    agent=agent,
    expected_output="Cleaned data in CSV format",
    
    # Execution settings
    timeout=300,  # 5 minutes max
    retries=3,  # Retry 3 times on failure
    priority=1,  # Higher = execute first
    
    # Dependencies
    depends_on=[],  # No dependencies
    
    # Validation
    output_validator=lambda x: "csv" in x.lower()
)
```

## Task Execution

### Sequential Tasks

```python
from nkit.tasks import TaskManager, Task

# Create tasks
gather_task = Task(
    title="Gather Data",
    description="Collect raw data",
    agent=agent
)

analyze_task = Task(
    title="Analyze Data",
    description="Analyze gathered data",
    agent=agent,
    depends_on=[gather_task]  # Wait for gather_task
)

report_task = Task(
    title="Generate Report",
    description="Create final report",
    agent=agent,
    depends_on=[analyze_task]  # Wait for analyze_task
)

# Execute
manager = TaskManager()
results = await manager.execute([gather_task, analyze_task, report_task])
```

### Parallel Tasks

```python
# These run in parallel (no dependencies)
task_a = Task(title="Task A", agent=agent)
task_b = Task(title="Task B", agent=agent)
task_c = Task(title="Task C", agent=agent)

# Execute all at once
manager = TaskManager()
results = await manager.execute([task_a, task_b, task_c])
```

### Mixed Workflow (DAG)

```python
# Task A runs first
task_a = Task(title="Gather", agent=agent)

# Tasks B and C run in parallel after A
task_b = Task(title="Analyze Path 1", agent=agent, depends_on=[task_a])
task_c = Task(title="Analyze Path 2", agent=agent, depends_on=[task_a])

# Task D runs after both B and C complete
task_d = Task(
    title="Merge Results",
    agent=agent,
    depends_on=[task_b, task_c]
)

# Task E runs last
task_e = Task(
    title="Report",
    agent=agent,
    depends_on=[task_d]
)

# Execute
manager = TaskManager()
results = await manager.execute([task_a, task_b, task_c, task_d, task_e])
```

## Task Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `title` | str | Required | Task name |
| `description` | str | Required | What to do |
| `agent` | Agent | Required | Agent to execute task |
| `expected_output` | str | "" | Expected result format |
| `depends_on` | list | [] | Prerequisite tasks |
| `timeout` | int | 300 | Max seconds |
| `retries` | int | 1 | Retry attempts |
| `priority` | int | 0 | Higher = execute first |
| `output_validator` | callable | None | Validate output |
| `on_error` | str | "fail" | "fail", "continue", "retry" |

## Task Callbacks

### On Completion

```python
def on_complete(task, result):
    print(f"Task {task.title} completed")
    print(f"Result: {result}")

task = Task(
    title="Work",
    description="Do something",
    agent=agent,
    on_complete=on_complete
)
```

### On Error

```python
def handle_error(task, error):
    print(f"Task {task.title} failed: {error}")
    # Send alert, rollback, etc.

task = Task(
    title="Critical Task",
    description="Must succeed",
    agent=agent,
    on_error=handle_error
)
```

## Task Status

```python
# Check task status
status = task.status  # "pending", "running", "completed", "failed"

# Get task result
result = task.result

# Get execution time
duration = task.duration
```

## Advanced Features

### Conditional Execution

```python
def should_continue(previous_result):
    """Decide whether to execute next task"""
    return previous_result.get("quality_score", 0) > 0.7

task = Task(
    title="Advanced Processing",
    description="Only run if quality is good",
    agent=agent,
    condition=should_continue
)
```

### Context Passing

```python
# Share data between tasks
task_a = Task(title="Gather", agent=agent)
task_b = Task(
    title="Process",
    agent=agent,
    depends_on=[task_a],
    context={
        "input": task_a.result  # Pass result from task_a
    }
)
```

### Custom Task Types

```python
from nkit.tasks import Task

class DataValidationTask(Task):
    """Custom task for data validation"""
    
    def validate(self):
        """Run validation"""
        # Custom logic
        pass
    
    async def execute(self):
        self.validate()
        return await super().execute()
```

## Best Practices

### 1. Clear Descriptions

```python
# Good
Task(
    title="Validate Email",
    description="Check if email format is valid using regex, return True/False"
)

# Avoid
Task(
    title="Check",
    description="Do validation"
)
```

### 2. Appropriate Timeouts

```python
# Quick task - short timeout
quick_task = Task(
    title="Simple Calculation",
    agent=agent,
    timeout=10
)

# Long-running task - longer timeout
long_task = Task(
    title="Data Processing",
    agent=agent,
    timeout=600  # 10 minutes
)
```

### 3. Retry Strategy

```python
# Unreliable API - more retries
api_task = Task(
    title="API Call",
    agent=agent,
    retries=5
)

# Deterministic task - fewer retries
compute_task = Task(
    title="Math Operation",
    agent=agent,
    retries=1
)
```

### 4. Efficient Parallelization

```python
# Good - truly independent tasks
tasks = [
    Task(title="Fetch Data Source 1", agent=agent),
    Task(title="Fetch Data Source 2", agent=agent),
    Task(title="Fetch Data Source 3", agent=agent)
]

# Avoid - unnecessary dependencies
tasks = [
    Task(title="Step 1", agent=agent),
    Task(title="Step 2", agent=agent, depends_on=[tasks[0]]),
    Task(title="Step 3", agent=agent, depends_on=[tasks[1]])
]
```

## Monitoring Tasks

### Task Hooks

```python
@task.hook("on_start")
def log_start():
    print(f"Starting {task.title}")

@task.hook("on_complete")
def log_complete():
    print(f"Completed {task.title}")

@task.hook("on_error")
def log_error(error):
    print(f"Error in {task.title}: {error}")
```

### TaskManager Events

```python
manager = TaskManager()

@manager.hook("on_task_complete")
def track_completion(task):
    print(f"Task {task.title} done in {task.duration}s")

results = await manager.execute(tasks)
```

## Troubleshooting

### Task Hangs

```python
# Check dependencies
print(task.depends_on)

# Set appropriate timeout
task.timeout = 30

# Check for circular dependencies
manager.validate_dag(tasks)
```

### Task Fails Repeatedly

```python
# Increase retries
task.retries = 5

# Increase timeout
task.timeout = 600

# Check error
try:
    result = await task.execute()
except Exception as e:
    print(f"Error: {e}")
```

## See Also

- **[Agents](agents.md)** - Task execution by agents
- **[Crews](crews.md)** - Multi-agent task coordination
- **[Examples](../examples/tasks.md)** - Task workflows
