# Events Example

Example of using the event system for monitoring and reactive programming.

## Full Code

```python
"""
Events Example: Event-Driven Agent Orchestration
Demonstrates event hooks and reactive agent patterns
"""

import asyncio
import time
from nkit import Agent, Step
from nkit.tools import Tool

# Metrics tracking
class MetricsCollector:
    def __init__(self):
        self.events = []
        self.start_time = None
        self.tool_times = {}
    
    def record_event(self, event_type, details):
        self.events.append({
            "type": event_type,
            "timestamp": time.time(),
            "details": details
        })
    
    def record_tool_time(self, tool_name, duration):
        if tool_name not in self.tool_times:
            self.tool_times[tool_name] = []
        self.tool_times[tool_name].append(duration)
    
    def print_summary(self):
        print("\n" + "=" * 70)
        print("EVENT METRICS SUMMARY")
        print("=" * 70)
        
        print(f"\n📊 Total Events: {len(self.events)}")
        
        event_types = {}
        for event in self.events:
            event_type = event["type"]
            event_types[event_type] = event_types.get(event_type, 0) + 1
        
        print("\n📈 Event Distribution:")
        for event_type, count in sorted(event_types.items()):
            print(f"   {event_type}: {count}")
        
        print("\n⏱️  Tool Execution Times:")
        for tool_name, times in sorted(self.tool_times.items()):
            avg_time = sum(times) / len(times)
            print(f"   {tool_name}: avg {avg_time:.2f}s, calls {len(times)}")

# Create tools
def create_tools():
    """Create tools for agent"""
    
    def calculate(operation: str, a: float, b: float) -> float:
        """Perform calculation"""
        time.sleep(0.1)  # Simulate work
        ops = {
            "add": lambda x, y: x + y,
            "subtract": lambda x, y: x - y,
            "multiply": lambda x, y: x * y
        }
        return ops.get(operation, 0)(a, b)
    
    def validate_result(result: float) -> bool:
        """Validate calculation result"""
        time.sleep(0.05)
        return isinstance(result, (int, float))
    
    return [
        Tool(
            name="calculate",
            description="Perform mathematical operations",
            callback=calculate
        ),
        Tool(
            name="validate",
            description="Validate calculation results",
            callback=validate_result
        )
    ]

# Create agent with event hooks
def create_event_aware_agent(metrics: MetricsCollector):
    """Create agent with event monitoring"""
    
    agent = Agent(
        name="EventAwareBot",
        description="Agent with event monitoring",
        model="gpt-3.5-turbo",
        temperature=0.3,
        enable_memory=True,
        verbose=True
    )
    
    # Add tools
    agent.add_tools(create_tools())
    
    # Register event hooks
    @agent.hook("on_iteration_start")
    def on_iter_start(step, iteration):
        msg = f"Starting iteration {iteration}: {step.title}"
        metrics.record_event("iteration_start", {"iteration": iteration, "step": step.title})
        print(f"▶️  {msg}")
    
    @agent.hook("on_iteration_end")
    def on_iter_end(step, iteration):
        msg = f"Completed iteration {iteration}"
        metrics.record_event("iteration_end", {"iteration": iteration})
        print(f"✅ {msg}")
    
    @agent.hook("on_tool_select")
    def on_tool_select(tool):
        msg = f"Selected tool: {tool.name}"
        metrics.record_event("tool_select", {"tool": tool.name})
        print(f"🔧 {msg}")
    
    @agent.hook("on_tool_execute")
    def on_tool_execute(tool, input_data, output):
        start = time.time()
        metrics.record_event("tool_execute", {
            "tool": tool.name,
            "input": str(input_data),
            "output": str(output)
        })
        duration = time.time() - start
        metrics.record_tool_time(tool.name, duration)
        print(f"⚙️  Tool {tool.name} executed: {output}")
    
    @agent.hook("on_tool_error")
    def on_tool_error(tool, error):
        msg = f"Tool {tool.name} error: {error}"
        metrics.record_event("tool_error", {"tool": tool.name, "error": str(error)})
        print(f"❌ {msg}")
    
    @agent.hook("on_reasoning_complete")
    def on_reasoning_complete(reasoning):
        metrics.record_event("reasoning_complete", {"reasoning": reasoning[:50]})
        print(f"🧠 Reasoning: {reasoning[:50]}...")
    
    @agent.hook("on_process_complete")
    def on_process_complete(result):
        metrics.record_event("process_complete", {
            "status": result.get("status"),
            "iterations": result.get("iterations")
        })
        print(f"🎯 Process complete - Status: {result.get('status')}")
    
    return agent

# Main execution
async def main():
    print("=" * 70)
    print("EVENT-DRIVEN AGENT EXAMPLE")
    print("=" * 70)
    
    # Create metrics collector
    metrics = MetricsCollector()
    
    # Create event-aware agent
    agent = create_event_aware_agent(metrics)
    
    print("\n✅ Created event-aware agent")
    print("✅ Registered event hooks\n")
    
    # Define workflow
    steps = [
        Step(
            title="Calculate Values",
            description="Perform mathematical calculations"
        ),
        Step(
            title="Validate Results",
            description="Validate calculated values"
        ),
        Step(
            title="Summary",
            description="Provide calculation summary"
        )
    ]
    
    # Execute agent
    print("=" * 70)
    print("EXECUTING AGENT WITH EVENT MONITORING")
    print("=" * 70 + "\n")
    
    result = await agent.aprocess(steps, "Calculate 10 + 5, then validate the result")
    
    # Print metrics summary
    metrics.print_summary()
    
    print("\n" + "=" * 70)
    print("FINAL RESULT")
    print("=" * 70)
    print(f"\nStatus: {result.get('status')}")
    print(f"Iterations: {result.get('iterations')}")
    print(f"Tool Calls: {result.get('tool_calls')}")
    print(f"\nAnswer: {result.get('final_answer')}\n")

if __name__ == "__main__":
    asyncio.run(main())
```

## Running the Example

```bash
pip install nkit openai
export OPENAI_API_KEY="sk-..."
python events_example.py
```

## Expected Output

```
======================================================================
EVENT-DRIVEN AGENT EXAMPLE
======================================================================

✅ Created event-aware agent
✅ Registered event hooks

======================================================================
EXECUTING AGENT WITH EVENT MONITORING
======================================================================

▶️  Starting iteration 1: Calculate Values
🔧 Selected tool: calculate
⚙️  Tool calculate executed: 15
✅ Completed iteration 1
▶️  Starting iteration 2: Validate Results
🔧 Selected tool: validate
⚙️  Tool validate executed: True
✅ Completed iteration 2

======================================================================
EVENT METRICS SUMMARY
======================================================================

📊 Total Events: 12
📈 Event Distribution:
   iteration_start: 2
   tool_select: 2
   tool_execute: 2
   iteration_end: 2
   process_complete: 1
```

## Available Events

### Iteration Events

- `on_iteration_start` - Start of reasoning iteration
- `on_iteration_end` - End of iteration
- `on_reasoning_complete` - Reasoning step finished

### Tool Events

- `on_tool_select` - Tool chosen
- `on_tool_execute` - Tool executed
- `on_tool_error` - Tool error occurred

### Process Events

- `on_process_start` - Agent processing started
- `on_process_complete` - Agent finished
- `on_memory_add` - Memory updated
- `on_memory_retrieve` - Memory accessed

### Agent Events

- `on_agent_error` - Error occurred
- `on_max_iterations` - Max iterations reached
- `on_timeout` - Timeout occurred

## Advanced Event Patterns

### Real-Time Monitoring

```python
@agent.hook("on_tool_execute")
def monitor_tool(tool, input_data, output):
    # Send to monitoring system
    send_to_monitoring_service({
        "tool": tool.name,
        "timestamp": time.time(),
        "status": "executed"
    })
```

### Error Alerting

```python
@agent.hook("on_tool_error")
def alert_on_error(tool, error):
    if "critical" in str(error).lower():
        send_alert(f"Critical error in {tool.name}: {error}")
```

### Metrics Collection

```python
@agent.hook("on_process_complete")
def record_metrics(result):
    metrics_db.insert({
        "agent": result["agent_name"],
        "iterations": result["iterations"],
        "token_usage": result["tokens_used"]["total"],
        "duration": result["duration"]
    })
```

### Logging

```python
@agent.hook("on_iteration_start")
def log_iteration(step, iteration):
    logger.info(f"Iteration {iteration}: {step.title}")

@agent.hook("on_tool_execute")
def log_tool_use(tool, input_data, output):
    logger.debug(f"Tool {tool.name} executed with output: {output}")
```

## Use Cases

### 1. **Performance Monitoring**

Track response times, iteration counts, token usage:

```python
@agent.hook("on_process_complete")
def track_performance(result):
    perf_data = {
        "iterations": result["iterations"],
        "tokens": result["tokens_used"]["total"],
        "duration": result["duration"]
    }
    analytics.record(perf_data)
```

### 2. **Cost Tracking**

Monitor API costs:

```python
@agent.hook("on_tool_execute")
def track_costs(tool, input_data, output):
    if tool.name == "expensive_api":
        cost = calculate_cost(input_data)
        billing.add_charge(cost)
```

### 3. **Audit Logging**

Log all actions for compliance:

```python
@agent.hook("on_tool_execute")
def audit_log(tool, input_data, output):
    audit_db.insert({
        "agent": agent.name,
        "tool": tool.name,
        "timestamp": time.time(),
        "user": current_user,
        "input_hash": hash(str(input_data))
    })
```

### 4. **Debug Tracing**

Detailed execution traces:

```python
@agent.hook("on_reasoning_complete")
def trace_reasoning(reasoning):
    debug_trace.append({"type": "reasoning", "content": reasoning})

@agent.hook("on_tool_select")
def trace_selection(tool):
    debug_trace.append({"type": "tool_selection", "tool": tool.name})
```

## Next Steps

- **[Core Concepts](../core-concepts/index.md)** - Deep dives
- **[Advanced Monitoring](../advanced/monitoring.md)** - Production monitoring
- **[Best Practices](../advanced/best-practices.md)** - Expert patterns
