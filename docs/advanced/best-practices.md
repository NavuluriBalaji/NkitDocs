# Best Practices

Expert patterns and best practices for building production-grade nkit applications.

## Agent Design

### 1. Clear Agent Purpose

Define specific, focused agents:

```python
# Good - focused purpose
researcher = Agent(
    name="ResearchAgent",
    description="Conducts thorough research on specified topics using search tools",
    model="gpt-4"  # Better reasoning
)

# Avoid - too broad
agent = Agent(
    name="GenericBot",
    description="Does everything"
)
```

### 2. Appropriate Model Selection

```python
# For complex reasoning
agent = Agent(model="gpt-4", temperature=0.3)

# For balanced performance/cost
agent = Agent(model="gpt-3.5-turbo", temperature=0.5)

# For local/fast execution
agent = Agent(model="ollama:llama2", temperature=0.2)
```

### 3. Meaningful Tool Descriptions

Agents use descriptions to select tools:

```python
# Good - specific behavior
Tool(
    name="web_search",
    description="Search the internet for current information using Google Search. Returns relevant links and summaries with publication dates.",
    callback=search
)

# Avoid - vague
Tool(
    name="search",
    description="Searches for things",
    callback=search
)
```

### 4. Proper Error Boundaries

```python
agent = Agent(
    name="SafeBot",
    max_iterations=5,    # Prevent loops
    timeout=60,          # Prevent hangs
    enable_memory=False  # Stateless by default
)
```

## Tool Design

### 1. Single Responsibility

Each tool does one thing well:

```python
# Good - focused tools
Tool(name="fetch_url", description="Fetch HTML from URL", callback=fetch_url)
Tool(name="parse_html", description="Extract text from HTML", callback=parse_html)
Tool(name="rank_results", description="Rank results by relevance", callback=rank)

# Avoid - kitchen sink
Tool(
    name="process_web",
    description="Fetch, parse, rank, and summarize web content",
    callback=do_everything
)
```

### 2. Input Validation

```python
# Use JSON schemas
Tool(
    name="email_send",
    description="Send email to recipient",
    callback=send_email,
    input_schema={
        "type": "object",
        "properties": {
            "to": {
                "type": "string",
                "pattern": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
            },
            "subject": {
                "type": "string",
                "maxLength": 200
            },
            "body": {
                "type": "string",
                "maxLength": 10000
            }
        },
        "required": ["to", "subject", "body"]
    }
)
```

### 3. Graceful Error Handling

```python
def safe_api_call(endpoint: str) -> dict:
    """Call API with error handling"""
    try:
        response = requests.get(endpoint, timeout=5)
        response.raise_for_status()
        return response.json()
    except requests.Timeout:
        return {"error": "Request timeout"}
    except requests.HTTPError as e:
        return {"error": f"HTTP error: {e.status_code}"}
    except Exception as e:
        return {"error": str(e)}
```

### 4. Async for I/O

```python
async def fetch_multiple_urls(urls: list) -> list:
    """Fetch multiple URLs concurrently"""
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [await r.json() for r in responses]

tool = Tool(
    name="fetch_urls",
    description="Fetch multiple URLs in parallel",
    callback=fetch_multiple_urls,
    is_async=True
)
```

## Task Orchestration

### 1. Optimal Parallelization

```python
# Identify independent tasks
tasks = [
    Task(title="Fetch Source A", agent=agent),      # Independent
    Task(title="Fetch Source B", agent=agent),      # Independent
    Task(title="Fetch Source C", agent=agent),      # Independent
    Task(title="Merge Data", agent=agent, depends_on=[...]),  # Depends on all
    Task(title="Analyze", agent=agent, depends_on=[...]),     # Depends on merge
]

# This DAG maximizes parallelization
```

### 2. Timeout Strategy

```python
# Quick, reliable tasks - short timeout
quick_task = Task(
    title="Cache Lookup",
    agent=agent,
    timeout=5,
    retries=0
)

# Potentially slow tasks - longer timeout
slow_task = Task(
    title="Complex Analysis",
    agent=agent,
    timeout=300,
    retries=2
)

# External API calls - balanced
api_task = Task(
    title="API Call",
    agent=agent,
    timeout=30,
    retries=3
)
```

### 3. Dependency Management

```python
# Express dependencies clearly
task_a = Task(title="Gather Data", agent=agent)
task_b = Task(title="Analyze", agent=agent, depends_on=[task_a])
task_c = Task(title="Report", agent=agent, depends_on=[task_b])

# Validate DAG has no cycles
manager = TaskManager()
manager.validate_dag([task_a, task_b, task_c])
```

## Memory Management

### 1. Strategic Enabling

```python
# Enable for conversational agents
chat_agent = Agent(
    name="ChatBot",
    enable_memory=True  # Remember conversation
)

# Disable for stateless tasks
task_agent = Agent(
    name="TaskBot",
    enable_memory=False  # Process each independently
)
```

### 2. Proactive Cleanup

```python
async def agent_with_cleanup():
    agent = Agent(name="Bot", enable_memory=True)
    
    for query in queries:
        result = await agent.aprocess([step], query)
        
        # Periodic cleanup
        tokens = agent.memory.get_token_count()
        if tokens > 2000:
            agent.memory.prune(keep_last=10)

asyncio.run(agent_with_cleanup())
```

### 3. Appropriate Memory Type

```python
# Conversation history
agent = Agent(memory=ConversationMemory())

# Vector embeddings for semantic search
agent = Agent(memory=VectorMemory())

# Persistent storage
agent = Agent(memory=FileMemory("./memory.json"))

# Database for scaling
agent = Agent(memory=DatabaseMemory(db=connection))
```

## Performance Optimization

### 1. Async First

```python
# Always use async where possible
async def process_many(queries: list):
    agent = Agent(name="Bot")
    
    tasks = [
        agent.aprocess([step], query)
        for query in queries
    ]
    
    results = await asyncio.gather(*tasks)
    return results
```

### 2. Caching Results

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def cached_search(query: str) -> list:
    """Cache search results"""
    return perform_search(query)

tool = Tool(
    name="cached_search",
    description="Search with caching",
    callback=cached_search
)
```

### 3. Token Optimization

```python
agent = Agent(
    name="OptimizedBot",
    model="gpt-3.5-turbo",  # Cheaper
    max_iterations=3,       # Fewer iterations
    temperature=0.2,        # More deterministic
    enable_memory=False     # No memory overhead
)
```

### 4. Batch Processing

```python
async def batch_process(agents: list, tasks: list) -> list:
    """Process multiple tasks in parallel"""
    
    # Distribute tasks among agents
    batches = [
        tasks[i::len(agents)]
        for i in range(len(agents))
    ]
    
    results = await asyncio.gather(*[
        agent.aprocess([], task)
        for agent, task_batch in zip(agents, batches)
        for task in task_batch
    ])
    
    return results
```

## Monitoring & Observability

### 1. Comprehensive Logging

```python
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

@agent.hook("on_process_complete")
def log_completion(result):
    logger.info({
        "timestamp": datetime.now().isoformat(),
        "agent": result.get("agent_name"),
        "status": result.get("status"),
        "iterations": result.get("iterations"),
        "tokens": result.get("tokens_used", {}).get("total"),
        "duration": result.get("duration")
    })
```

### 2. Metrics Collection

```python
from collections import defaultdict
import time

class Metrics:
    def __init__(self):
        self.request_times = defaultdict(list)
        self.error_count = 0
    
    @property
    def avg_request_time(self):
        times = [t for times in self.request_times.values() for t in times]
        return sum(times) / len(times) if times else 0
    
    @property
    def error_rate(self):
        total = sum(len(times) for times in self.request_times.values())
        return self.error_count / total if total > 0 else 0

metrics = Metrics()

@agent.hook("on_process_complete")
def track_metrics(result):
    key = f"{result['agent_name']}:{result['status']}"
    metrics.request_times[key].append(result.get('duration', 0))
    if result['status'] == 'error':
        metrics.error_count += 1
```

### 3. Alert Thresholds

```python
@agent.hook("on_process_complete")
def check_health(result):
    # High iterations warning
    if result["iterations"] > 8:
        logger.warning(f"High iterations: {result['iterations']}")
    
    # High token usage warning
    tokens = result.get("tokens_used", {}).get("total", 0)
    if tokens > 3000:
        logger.warning(f"High token usage: {tokens}")
    
    # Slow execution warning
    if result.get("duration", 0) > 60:
        logger.warning(f"Slow execution: {result['duration']}s")
```

## Security

### 1. Input Sanitization

```python
import re

def sanitize_input(user_input: str) -> str:
    """Remove potentially harmful content"""
    # Remove SQL-like patterns
    dangerous_patterns = [
        r'DROP\s+TABLE',
        r'DELETE\s+FROM',
        r'INSERT\s+INTO',
    ]
    
    result = user_input
    for pattern in dangerous_patterns:
        result = re.sub(pattern, '', result, flags=re.IGNORECASE)
    
    return result.strip()
```

### 2. Permission Checks

```python
def check_permission(user_id: str, action: str) -> bool:
    """Verify user can perform action"""
    permissions = db.get_user_permissions(user_id)
    return action in permissions

Tool(
    name="sensitive_operation",
    description="Perform sensitive operation",
    callback=sensitive_op,
    condition=lambda: check_permission(current_user, "sensitive_op")
)
```

### 3. Secrets Management

```python
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    OPENAI_KEY = os.getenv("OPENAI_API_KEY")
    DB_PASSWORD = os.getenv("DB_PASSWORD")
    
    @classmethod
    def validate(cls):
        if not cls.OPENAI_KEY:
            raise ValueError("OPENAI_API_KEY not set")
        if not cls.DB_PASSWORD:
            raise ValueError("DB_PASSWORD not set")

Config.validate()
```

## Testing

### 1. Unit Tests

```python
import pytest
from nkit import Agent

@pytest.mark.asyncio
async def test_agent_basic():
    agent = Agent(name="TestBot")
    result = await agent.aprocess([], "test")
    assert result["status"] in ["success", "error"]

def test_tool_validation():
    tool = Tool(
        name="test",
        description="Test",
        callback=lambda x: x
    )
    assert tool.name == "test"
```

### 2. Integration Tests

```python
@pytest.mark.asyncio
async def test_agent_with_tools():
    agent = Agent(name="Bot")
    agent.add_tool(Tool(
        name="add",
        description="Add numbers",
        callback=lambda a, b: a + b
    ))
    
    result = await agent.aprocess([], "What is 2+3?")
    assert "5" in str(result["final_answer"])
```

### 3. Load Tests

```python
import asyncio
import time

async def load_test(num_requests=100):
    agent = Agent(name="LoadTestBot")
    
    start = time.time()
    
    tasks = [
        agent.aprocess([], f"Request {i}")
        for i in range(num_requests)
    ]
    
    results = await asyncio.gather(*tasks)
    
    duration = time.time() - start
    
    successful = sum(1 for r in results if r["status"] == "success")
    
    print(f"Processed {num_requests} in {duration:.2f}s")
    print(f"Success rate: {successful/num_requests*100:.1f}%")
    print(f"Throughput: {num_requests/duration:.1f} req/s")
```

## Documentation

### 1. Clear Comments

```python
async def process_agent_request(agent: Agent, task: str):
    """
    Execute agent with error handling and monitoring.
    
    Args:
        agent: The agent to execute
        task: The task description
    
    Returns:
        Result dictionary with:
        - status: success/error/timeout
        - final_answer: Agent's response
        - iterations: Number of reasoning steps
    
    Raises:
        TimeoutError: If execution exceeds timeout
        ValueError: If invalid inputs
    """
    # Implementation
    pass
```

### 2. Type Hints

```python
from typing import list, dict, Optional

async def batch_process(
    agents: list[Agent],
    tasks: list[str],
    timeout: Optional[int] = None
) -> list[dict]:
    """Process multiple tasks concurrently"""
    # Implementation
    pass
```

## Summary

Key practices:
- ✅ Single responsibility per agent/tool
- ✅ Clear descriptions for agent decision-making
- ✅ Input validation and error handling
- ✅ Async for I/O operations
- ✅ Proactive resource management
- ✅ Comprehensive monitoring
- ✅ Security and access control
- ✅ Thorough testing

See also:
- [Advanced Patterns](../advanced/patterns.md)
- [Performance Tuning](../advanced/performance.md)
- [Monitoring](../advanced/monitoring.md)
