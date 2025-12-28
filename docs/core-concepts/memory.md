# Memory

## What is Memory?

**Memory** allows agents to persist and recall information across interactions:

- **Conversation Memory**: Track chat history
- **Entity Memory**: Remember important facts
- **Long-term Memory**: Vector embeddings for semantic search
- **Custom Memory**: Database-backed persistence

## Memory Types

### 1. Conversation Memory

Track interaction history:

```python
from nkit import Agent
from nkit.memory import ConversationMemory

agent = Agent(
    name="ChatBot",
    enable_memory=True,
    memory_type="conversation"
)

# Agent remembers previous messages
result1 = await agent.aprocess([step], "My name is Alice")
result2 = await agent.aprocess([step], "What's my name?")
# Agent recalls: "Your name is Alice"
```

### 2. Entity Memory

Store important facts:

```python
memory = agent.memory

# Add facts
memory.add_entity("user", {
    "name": "Alice",
    "role": "engineer",
    "preferences": ["Python", "async", "clean code"]
})

# Retrieve facts
user_info = memory.get_entity("user")
```

### 3. Summary Memory

Compress long conversations:

```python
from nkit.memory import SummaryMemory

agent = Agent(
    name="Bot",
    memory=SummaryMemory()
)

# Automatically summarizes old messages
```

### 4. Vector Memory

Semantic search across conversations:

```python
from nkit.memory import VectorMemory

agent = Agent(
    name="Bot",
    memory=VectorMemory(embedding_model="openai")
)

# Can search semantically similar past messages
```

## Using Memory

### Access Memory

```python
agent = Agent(name="Bot", enable_memory=True)

# Get memory object
memory = agent.memory

# Add to memory
memory.add("fact", "important information")

# Retrieve from memory
facts = memory.get("fact")

# Clear memory
memory.clear()
```

### Memory Structure

```python
memory_state = {
    "messages": [
        {"role": "user", "content": "Hello"},
        {"role": "assistant", "content": "Hi there!"}
    ],
    "entities": {
        "user": {"name": "Alice", "age": 30},
        "context": {"topic": "Python"}
    },
    "facts": [
        "User is a Python developer",
        "Interested in async programming"
    ]
}
```

### Hooks for Memory Events

```python
@agent.hook("on_memory_add")
def log_memory_write(key, value):
    print(f"Storing: {key} = {value}")

@agent.hook("on_memory_retrieve")
def log_memory_read(key, value):
    print(f"Retrieved: {key}")
```

## Custom Memory Backend

### File-Based Memory

```python
from nkit.memory import FileMemory

agent = Agent(
    name="Bot",
    memory=FileMemory(filepath="./memory.json")
)

# Persists across sessions
```

### Database Memory

```python
from nkit.memory import DatabaseMemory
import sqlite3

db = sqlite3.connect("agent_memory.db")

agent = Agent(
    name="Bot",
    memory=DatabaseMemory(db=db)
)

# Persist to database
```

### Custom Implementation

```python
from nkit.memory import BaseMemory

class RedisMemory(BaseMemory):
    """Custom Redis-backed memory"""
    
    def __init__(self, redis_client):
        self.client = redis_client
    
    def add(self, key: str, value):
        """Store in Redis"""
        self.client.set(key, json.dumps(value))
    
    def get(self, key: str):
        """Retrieve from Redis"""
        data = self.client.get(key)
        return json.loads(data) if data else None
    
    def clear(self):
        """Clear all keys"""
        self.client.flushdb()

# Use custom memory
import redis
redis_client = redis.Redis(host='localhost')
agent = Agent(
    name="Bot",
    memory=RedisMemory(redis_client)
)
```

## Memory Management

### Token Optimization

```python
agent = Agent(
    name="Bot",
    enable_memory=True
)

# Memory uses tokens - monitor usage
tokens_used = agent.memory.get_token_count()
print(f"Memory tokens: {tokens_used}")

# Clear old messages to save tokens
agent.memory.prune(keep_last=10)  # Keep last 10 messages
```

### Memory Compression

```python
# Summarize old messages automatically
agent.memory.compress_before(
    timestamp=time.time() - 3600,  # Older than 1 hour
    strategy="summarize"
)
```

### Selective Retention

```python
# Remember important messages only
@agent.hook("on_message_add")
def select_important(message):
    if message["importance"] > 0.7:
        return True  # Keep
    return False  # Discard
```

## Memory Size Limits

```python
from nkit.memory import BoundedMemory

agent = Agent(
    name="Bot",
    memory=BoundedMemory(
        max_tokens=2000,  # Max 2000 tokens
        max_messages=50,  # Max 50 messages
        strategy="oldest"  # Remove oldest when full
    )
)
```

## Multi-Agent Memory

### Shared Memory

```python
# Multiple agents share same memory
shared_memory = ConversationMemory()

agent1 = Agent(name="Bot1", memory=shared_memory)
agent2 = Agent(name="Bot2", memory=shared_memory)

# Both agents access same facts
```

### Memory Isolation

```python
# Each agent has own memory
agent1 = Agent(name="Bot1", enable_memory=True)
agent2 = Agent(name="Bot2", enable_memory=True)

# Separate memories
```

## Best Practices

### 1. Clear Periodically

```python
# Prevent unbounded growth
async def periodic_cleanup():
    while True:
        await asyncio.sleep(3600)  # 1 hour
        agent.memory.prune(keep_last=20)
```

### 2. Monitor Token Usage

```python
@agent.hook("on_process_complete")
def monitor_memory(result):
    memory_tokens = agent.memory.get_token_count()
    if memory_tokens > 1500:
        print("⚠️ Memory approaching limit")
        agent.memory.clear()
```

### 3. Selective Storage

```python
# Only store important information
@agent.hook("on_message_add")
def filter_messages(message):
    # Don't store debug messages
    if message["level"] == "debug":
        return False
    return True
```

### 4. Secure Storage

```python
from nkit.memory import EncryptedMemory

agent = Agent(
    name="Bot",
    memory=EncryptedMemory(
        filepath="./memory.encrypted",
        encryption_key="secret-key"
    )
)
```

## Troubleshooting

### Memory Not Working

```python
# Ensure memory is enabled
agent = Agent(
    name="Bot",
    enable_memory=True  # Required!
)
```

### High Token Usage

```python
# Reduce memory size
agent.memory.clear()

# Or use summary memory
from nkit.memory import SummaryMemory
agent.memory = SummaryMemory()
```

### Memory Persistence Issues

```python
# Check file permissions
import os
os.chmod("./memory.json", 0o644)

# Or use database
from nkit.memory import DatabaseMemory
agent.memory = DatabaseMemory(db=db)
```

## See Also

- **[Agents](agents.md)** - Agent configuration
- **[Tasks](tasks.md)** - Task-based workflows
- **[Advanced Memory](../advanced/custom-memory.md)** - Deep customization
