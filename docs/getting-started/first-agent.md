# Your First Agent

In this guide, you'll build a **Weather Research Agent** that can look up weather information.

## Step 1: Setup

### Install Dependencies

```bash
pip install nkit openai
```

### Set Your API Key

```bash
export OPENAI_API_KEY="sk-..."
```

## Step 2: Create the Agent

Create a file called `weather_agent.py`:

```python
from nkit import Agent, Step

# Create your agent
weather_agent = Agent(
    name="WeatherBot",
    description="An intelligent agent that researches weather conditions and provides insights",
    model="gpt-3.5-turbo",  # or "gpt-4" for better reasoning
    temperature=0.3  # Low temperature = more deterministic
)

if __name__ == "__main__":
    import asyncio
    
    # Test it
    step = Step(
        title="Research Weather",
        description="Find weather information about a location"
    )
    
    result = asyncio.run(
        weather_agent.aprocess([step], "What's the weather like in New York?")
    )
    print(result)
```

## Step 3: Add a Tool

Tools let your agent interact with external services. Let's add a weather lookup tool:

```python
from nkit import Agent, Step
from nkit.tools import Tool
import asyncio

# Simulate a weather API
def get_weather(location: str) -> dict:
    """Get current weather for a location"""
    # In reality, this would call a real weather API
    weather_data = {
        "New York": {"temp": 72, "condition": "Sunny", "humidity": 65},
        "London": {"temp": 55, "condition": "Cloudy", "humidity": 80},
        "Tokyo": {"temp": 68, "condition": "Clear", "humidity": 50}
    }
    return weather_data.get(location, {"error": "Location not found"})

# Create agent
weather_agent = Agent(
    name="WeatherBot",
    description="Researches weather and climate information",
    model="gpt-3.5-turbo"
)

# Add the weather tool
weather_agent.add_tool(Tool(
    name="get_weather",
    description="Get current weather for a location. Returns temperature, conditions, and humidity.",
    callback=get_weather,
    input_schema={
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "City name (e.g., 'New York', 'London')"
            }
        },
        "required": ["location"]
    }
))

if __name__ == "__main__":
    # Use the agent
    step = Step(
        title="Get Weather",
        description="Look up current weather"
    )
    
    result = asyncio.run(
        weather_agent.aprocess([step], "What's the weather in New York and London?")
    )
    print(result)
```

## Step 4: Add Multiple Steps

Let's create a workflow:

```python
from nkit import Agent, Step
from nkit.tools import Tool
import asyncio

def get_weather(location: str) -> dict:
    """Get current weather for a location"""
    weather_data = {
        "New York": {"temp": 72, "condition": "Sunny", "humidity": 65},
        "London": {"temp": 55, "condition": "Cloudy", "humidity": 80},
    }
    return weather_data.get(location, {"error": "Location not found"})

def get_forecast(location: str) -> str:
    """Get 5-day weather forecast"""
    return f"5-day forecast for {location}: Mostly sunny, high 75°F, low 60°F"

# Create agent with multiple tools
weather_agent = Agent(
    name="WeatherBot",
    description="Comprehensive weather research agent",
    model="gpt-3.5-turbo"
)

# Add tools
weather_agent.add_tool(Tool(
    name="get_weather",
    description="Get current weather",
    callback=get_weather
))

weather_agent.add_tool(Tool(
    name="get_forecast",
    description="Get weather forecast",
    callback=get_forecast
))

# Create workflow steps
steps = [
    Step(
        title="Get Current Weather",
        description="Look up current weather conditions"
    ),
    Step(
        title="Get Forecast",
        description="Retrieve the weather forecast"
    ),
    Step(
        title="Summarize",
        description="Provide a comprehensive weather summary"
    )
]

if __name__ == "__main__":
    result = asyncio.run(
        weather_agent.aprocess(steps, "What's the weather in New York? Will it be good for outdoor activities?")
    )
    print(result)
```

## Step 5: Add Memory

Let your agent remember the conversation:

```python
from nkit import Agent, Step
from nkit.tools import Tool
import asyncio

def get_weather(location: str) -> dict:
    weather_data = {
        "New York": {"temp": 72, "condition": "Sunny"},
        "London": {"temp": 55, "condition": "Cloudy"},
    }
    return weather_data.get(location, {"error": "Not found"})

# Create agent with memory
weather_agent = Agent(
    name="WeatherBot",
    description="Weather research agent with memory",
    model="gpt-3.5-turbo",
    enable_memory=True,  # Enable conversation memory
    max_iterations=5
)

weather_agent.add_tool(Tool(
    name="get_weather",
    description="Get current weather",
    callback=get_weather
))

async def main():
    # First query
    result1 = await weather_agent.aprocess(
        [Step(title="Query 1", description="Weather lookup")],
        "What's the weather in New York?"
    )
    print("First query:", result1)
    
    # Second query - agent remembers the context!
    result2 = await weather_agent.aprocess(
        [Step(title="Query 2", description="Follow-up question")],
        "Is it good for outdoor activities?" # Agent remembers we were talking about New York
    )
    print("Follow-up:", result2)

asyncio.run(main())
```

## Running Your Agent

```bash
python weather_agent.py
```

## Next Steps

**Congratulations!** 🎉 You've created your first agent!

### Learn More:

- 📚 **[Tools Guide](../core-concepts/tools.md)** - Create sophisticated tools
- 🔄 **[Tasks Guide](../core-concepts/tasks.md)** - Build complex workflows
- 👥 **[Crews Guide](../core-concepts/crews.md)** - Coordinate multiple agents
- 📊 **[Memory Systems](../core-concepts/memory.md)** - Persistent state management
- 🧪 **[Examples](../examples/index.md)** - More complete examples

### Common Extensions:

1. **Add Real APIs**: Replace mock functions with real API calls
2. **Error Handling**: Add try-catch for tool execution
3. **Logging**: Enable debug logging for troubleshooting
4. **Custom Memory**: Use database instead of in-memory storage
5. **Webhooks**: Trigger agents from web events

## Troubleshooting

**Agent loops indefinitely?**
```python
agent = Agent(
    name="Bot",
    max_iterations=5  # Limit iterations
)
```

**Tools not being called?**
```python
# Check tool description is clear
# Agent uses description to decide when to call tools
Tool(
    name="function_name",
    description="Very clear description of what this does",
    callback=function
)
```

**Memory not working?**
```python
agent = Agent(
    name="Bot",
    enable_memory=True  # Must enable explicitly
)
```

## Get Help

- 💬 See [Core Concepts](../core-concepts/index.md) for deeper understanding
- 🔍 Check [API Reference](../api-reference/agent.md) for detailed docs
- ❓ Visit [FAQ](../faq.md) for common questions
