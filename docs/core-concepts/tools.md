# Tools

## What are Tools?

**Tools** are capabilities that agents can use to interact with the external world:

- 🔍 Search the web
- 📄 Read/write files
- 🧮 Perform calculations
- 🌐 Call APIs
- 💾 Query databases
- 📧 Send emails
- And more!

## Tool Structure

Every tool has three components:

```
Tool
├── Name: Unique identifier
├── Description: What it does (agent uses this)
└── Callback: Python function to execute
```

## Creating Tools

### Simple Tool

```python
from nkit.tools import Tool

def add_numbers(a: float, b: float) -> float:
    """Add two numbers together"""
    return a + b

# Create tool
calculator = Tool(
    name="add",
    description="Add two numbers together",
    callback=add_numbers
)
```

### Tool with Input Validation

```python
from nkit.tools import Tool

def search_web(query: str, limit: int = 5) -> list:
    """Search the web for information"""
    # Validate inputs
    if not query or len(query) > 200:
        raise ValueError("Query must be 1-200 characters")
    
    if limit < 1 or limit > 20:
        raise ValueError("Limit must be 1-20")
    
    # Perform search...
    return ["result1", "result2", ...]

search_tool = Tool(
    name="web_search",
    description="Search the web for information about any topic. Returns list of relevant results.",
    callback=search_web,
    input_schema={
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "Search query (1-200 characters)"
            },
            "limit": {
                "type": "integer",
                "description": "Max results to return (1-20)",
                "default": 5
            }
        },
        "required": ["query"]
    }
)
```

### Async Tools

```python
import asyncio
from nkit.tools import Tool

async def fetch_data(url: str) -> dict:
    """Fetch data from an API endpoint"""
    # Simulate async operation
    await asyncio.sleep(1)
    return {"status": "ok", "data": [...]}

fetch_tool = Tool(
    name="api_fetch",
    description="Fetch data from a URL",
    callback=fetch_data,
    is_async=True
)
```

## Tool Types

### 1. **Data Retrieval**

```python
Tool(
    name="database_query",
    description="Query the database with SQL",
    callback=lambda query: db.execute(query)
)
```

### 2. **Data Modification**

```python
Tool(
    name="save_file",
    description="Save content to a file",
    callback=lambda path, content: open(path, 'w').write(content)
)
```

### 3. **Computation**

```python
Tool(
    name="mathematical_analysis",
    description="Perform complex mathematical calculations",
    callback=perform_analysis
)
```

### 4. **External API Integration**

```python
Tool(
    name="weather_api",
    description="Get current weather for a location",
    callback=lambda location: requests.get(f"https://api.weather.io/{location}").json()
)
```

### 5. **Content Generation**

```python
Tool(
    name="code_generator",
    description="Generate Python code for a given task",
    callback=generate_code
)
```

## Input Schemas

Define expected inputs using JSON Schema:

```python
from nkit.tools import Tool

def calculate(operation: str, a: float, b: float) -> float:
    """Perform math operation"""
    ops = {"add": lambda x, y: x + y, "subtract": lambda x, y: x - y}
    return ops[operation](a, b)

tool = Tool(
    name="calculator",
    description="Perform mathematical operations",
    callback=calculate,
    input_schema={
        "type": "object",
        "properties": {
            "operation": {
                "type": "string",
                "description": "Math operation: 'add' or 'subtract'",
                "enum": ["add", "subtract"]
            },
            "a": {
                "type": "number",
                "description": "First number"
            },
            "b": {
                "type": "number",
                "description": "Second number"
            }
        },
        "required": ["operation", "a", "b"]
    }
)
```

## Using Tools with Agents

### Add Single Tool

```python
from nkit import Agent
from nkit.tools import Tool

agent = Agent(name="Bot")

calculator = Tool(
    name="add",
    description="Add two numbers",
    callback=lambda a, b: a + b
)

agent.add_tool(calculator)
```

### Add Multiple Tools

```python
tools = [
    Tool(name="add", description="Add numbers", callback=lambda a, b: a + b),
    Tool(name="subtract", description="Subtract numbers", callback=lambda a, b: a - b),
    Tool(name="multiply", description="Multiply numbers", callback=lambda a, b: a * b),
]

agent.add_tools(tools)

# Now agent can use any of these tools
result = await agent.aprocess([step], "What is 10 + 5?")
```

## Built-in Tools

nkit provides common tools:

```python
from nkit.tools import builtin_tools

# Use built-in tools
math_tools = builtin_tools.get_math_tools()
file_tools = builtin_tools.get_file_tools()
web_tools = builtin_tools.get_web_tools()

agent.add_tools(math_tools)
agent.add_tools(file_tools)
agent.add_tools(web_tools)
```

Available built-in tools:
- **Math**: add, subtract, multiply, divide, power, sqrt
- **File I/O**: read_file, write_file, list_files, delete_file
- **Web**: fetch_url, parse_html, extract_json
- **Text**: text_similarity, string_match, regex_match
- **Data**: json_parse, csv_read, csv_write

## Error Handling

### Tool Errors

```python
def divide(a: float, b: float) -> float:
    """Divide two numbers"""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

tool = Tool(
    name="divide",
    description="Divide one number by another",
    callback=divide
)

# Agent will catch and handle the error gracefully
```

### Timeout Protection

```python
def slow_operation():
    import time
    time.sleep(100)  # Very slow

tool = Tool(
    name="slow_op",
    description="A slow operation",
    callback=slow_operation,
    timeout=5  # Will timeout after 5 seconds
)
```

## Tool Monitoring

### Log Tool Usage

```python
@agent.hook("on_tool_execute")
def log_tool(tool, input, output):
    print(f"Tool: {tool.name}")
    print(f"Input: {input}")
    print(f"Output: {output}")
```

### Validate Tool Output

```python
from nkit.tools import Tool

def validate_output(output):
    """Ensure output meets requirements"""
    if not isinstance(output, (str, dict, list)):
        raise TypeError("Output must be string, dict, or list")
    return output

tool = Tool(
    name="api_call",
    description="Call an API",
    callback=some_api_call,
    output_validator=validate_output
)
```

## Advanced Features

### Tool Dependencies

```python
# Tool A depends on output from Tool B
tool_b = Tool(name="fetch_data", ...)
tool_a = Tool(
    name="process_data",
    description="Process fetched data",
    callback=process_data,
    depends_on=[tool_b]
)
```

### Conditional Tool Use

```python
def should_use_tool(context: dict) -> bool:
    """Decide whether to use this tool"""
    return context.get("has_permission", False)

tool = Tool(
    name="delete_file",
    description="Delete a file (admin only)",
    callback=delete_file,
    condition=should_use_tool
)
```

### Tool Metadata

```python
tool = Tool(
    name="expensive_operation",
    description="Runs an expensive operation",
    callback=expensive_fn,
    metadata={
        "cost": 0.10,  # Monetary cost
        "latency": "high",
        "rarity": "use only when necessary",
        "category": "expensive"
    }
)
```

## Best Practices

### 1. **Clear Descriptions**

Agent uses descriptions to decide when to call tools. Be specific!

```python
# Bad
Tool(name="data", description="Gets data")

# Good
Tool(
    name="user_database_query",
    description="Query the user database by ID, email, or name. Returns user record with id, name, email, and created_date."
)
```

### 2. **Input Validation**

```python
def send_email(to: str, subject: str, body: str) -> bool:
    """Send an email"""
    # Validate inputs
    if not re.match(r"^[\w\.-]+@[\w\.-]+\.\w+$", to):
        raise ValueError(f"Invalid email: {to}")
    
    if len(subject) > 200:
        raise ValueError("Subject too long (max 200 chars)")
    
    # Send email...
    return True
```

### 3. **Meaningful Return Values**

```python
# Good - clear success/failure
Tool(
    name="operation",
    description="Perform operation",
    callback=lambda: {"success": True, "message": "Done", "result": {...}}
)

# Avoid - ambiguous
Tool(
    name="operation",
    description="Perform operation",
    callback=lambda: "OK"
)
```

### 4. **Limit Tool Scope**

```python
# Bad - too broad
Tool(name="database", description="Do anything with database")

# Good - focused
Tool(name="get_user_by_id", description="Retrieve user by ID")
Tool(name="list_all_users", description="List all users")
Tool(name="update_user_email", description="Update a user's email address")
```

### 5. **Document Side Effects**

```python
Tool(
    name="create_backup",
    description="Create database backup (saves to /backups/ directory, takes 30 seconds)",
    callback=backup_db
)
```

## Troubleshooting

### Tool Not Being Called

```python
# Problem: Vague description
Tool(name="search", description="Searches stuff")

# Solution: Specific description
Tool(
    name="web_search",
    description="Search the internet for information. Use when you need to find facts, recent news, or data about a topic."
)
```

### Agent Misuses Tool

```python
# Problem: Unclear input schema
tool = Tool(
    name="query",
    description="Query the database",
    callback=query_db
)

# Solution: Clear schema
tool = Tool(
    name="query",
    description="Query the database",
    callback=query_db,
    input_schema={
        "type": "object",
        "properties": {
            "table": {
                "type": "string",
                "description": "Table name: 'users', 'products', or 'orders'"
            },
            "filter": {
                "type": "string",
                "description": "WHERE clause for filtering results"
            }
        },
        "required": ["table"]
    }
)
```

### High Latency

```python
# Use async for I/O-bound operations
async def fetch_large_data():
    # Use async HTTP client
    async with aiohttp.ClientSession() as session:
        return await session.get(url)

tool = Tool(
    name="fetch",
    description="Fetch data from API",
    callback=fetch_large_data,
    is_async=True
)
```

## See Also

- **[Agents](agents.md)** - How agents use tools
- **[Examples](../examples/basic-agent.md)** - Tools in action
- **[API Reference](../api-reference/tools.md)** - Complete tool API
