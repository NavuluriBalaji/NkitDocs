# Quick Start

Get your first NKit agent initialized and autonomously executing tasks in minutes.

## 1. Installation

Install the framework and its base dependencies:

```bash
pip install nkit-agents
```

If you plan on hooking up local models via LM Studio, ensure you have the required provider dependency:

```bash
pip install lmstudio
```

## 2. Connect an LLM Provider

NKit operates through a unified dependency injection model. You first select your LLM hardware provider:

```python
import os
from NKit.llms import GeminiLLM, LMStudioLLM

# Option A: Cloud Provider (Google Gemini)
os.environ["GEMINI_API_KEY"] = "your-api-key"
google_backend = GeminiLLM(model="gemini-2.5-flash")

# Option B: Local Provider (LM Studio)
# Bypasses API keys and connects to your local inference server
local_backend = LMStudioLLM(base_url="http://localhost:1234")
```

## 3. Initialize the Core Engine

The `Agent` orchestrates your selected configurations. You can run tasks synchronously using `.run()` or gracefully in async loops using `.run_async()`.

```python
import asyncio
from NKit import Agent

# Initialize the God Agent
agent = Agent(llm=local_backend)

async def main():
    print("Initiating sequence...")
    result = await agent.run_async("What is the current time and current working directory?")
    print("Result:", result)

asyncio.run(main())
```

*(Note: NKit comes pre-equipped with foundational utilities like `get_time`, `list_files`, and `web_search` natively, allowing your agent to dynamically accomplish basic tasks without any custom tool setup.)*

## 4. Enabling Observability (Event Streaming)

Agents can operate like a black box if configured poorly. To establish complete transparency, NKit includes the `LiveObserver` class. 

By injecting an observer, you can pipe real-time streams of agent reasoning directly to your CLI, frontend server, or graphical IDE.

```python
from NKit import Agent, LiveObserver
from NKit.llms import LMStudioLLM

observer = LiveObserver()

# Subscribe to reasoning events
@observer.on("agent.reasoning")
def print_thoughts(event):
    print(f"[REASONING STREAM]: {event.get('thought', '')}")

# Subscribe to tool execution events
@observer.on("tool.before")
def print_action(event):
    print(f"[EXECUTION STREAM]: Calling '{event['tool_name']}'")

agent = Agent(
    llm=LMStudioLLM(), 
    observer=observer
)

agent.run("What files are currently around you?")
```

## 5. Security & Auditing

For enterprise applications spanning multiple autonomous actions, it is heavily recommended to inject our safety and tracing parameters:

```python
from NKit.safety import SafetyGate
from NKit.audit import WhyLog

# The SafetyGate validates all tool permissions pre-execution
safety_filter = SafetyGate()

# WhyLog tracks un-tamperable audit logs of every iteration
audit = WhyLog(path="./logs/agent_audit.jsonl")

agent = Agent(
    llm=local_backend,
    safety_gate=safety_filter,
    why_log=audit
)
```

## Next Steps

Now that your Agent framework is initialized, read out our architecture documents to learn how to inject the powerful CodeACT sandbox or deploy hierarchical, multi-agent frameworks.
