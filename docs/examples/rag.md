# RAG Example

Example of building a Retrieval Augmented Generation (RAG) system with agents.

## Full Code

```python
"""
RAG Example: Knowledge Base Integration
Demonstrates retrieval of documents for context-aware agent responses
"""

import asyncio
from nkit import Agent, Step
from nkit.tools import Tool
from nkit.knowledge import KnowledgeBase, Document

# Create a simple knowledge base
def create_knowledge_base():
    """Create sample knowledge base"""
    kb = KnowledgeBase()
    
    documents = [
        Document(
            id="doc1",
            content="Python is a high-level programming language known for its simplicity and readability",
            source="tech-docs",
            metadata={"topic": "programming", "language": "python"}
        ),
        Document(
            id="doc2",
            content="Async/await in Python allows writing concurrent code that is easy to read",
            source="tech-docs",
            metadata={"topic": "programming", "feature": "async"}
        ),
        Document(
            id="doc3",
            content="Machine learning requires large amounts of quality training data",
            source="ai-docs",
            metadata={"topic": "machine-learning", "task": "training"}
        ),
        Document(
            id="doc4",
            content="Web frameworks like Django and Flask simplify Python web development",
            source="web-docs",
            metadata={"topic": "web", "frameworks": ["django", "flask"]}
        ),
    ]
    
    for doc in documents:
        kb.add_document(doc)
    
    return kb

# Create retrieval tool
def create_retrieval_tool(knowledge_base):
    """Create tool for retrieving documents"""
    
    def retrieve_documents(query: str, limit: int = 3) -> list:
        """Retrieve relevant documents from knowledge base"""
        results = knowledge_base.search(query, limit=limit)
        return [
            {
                "id": doc.id,
                "content": doc.content,
                "source": doc.source,
                "relevance": doc.relevance_score
            }
            for doc in results
        ]
    
    return Tool(
        name="retrieve_documents",
        description="Search knowledge base and retrieve relevant documents",
        callback=retrieve_documents,
        input_schema={
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Search query for knowledge base"
                },
                "limit": {
                    "type": "integer",
                    "description": "Max documents to return",
                    "default": 3
                }
            },
            "required": ["query"]
        }
    )

# Create RAG agent
def create_rag_agent(knowledge_base):
    """Create agent with knowledge base access"""
    
    agent = Agent(
        name="RAGAgent",
        description="Intelligent agent with access to knowledge base",
        model="gpt-3.5-turbo",
        temperature=0.3,
        enable_memory=True
    )
    
    # Add retrieval tool
    agent.add_tool(create_retrieval_tool(knowledge_base))
    
    # Custom system prompt for RAG
    agent.system_prompt = """
You are a helpful assistant with access to a knowledge base.
When answering questions:
1. Use the retrieve_documents tool to find relevant information
2. Base your answers on the retrieved documents
3. Cite the source of your information
4. If documents don't contain relevant info, say so clearly
"""
    
    return agent

# Main execution
async def main():
    print("=" * 70)
    print("RAG (RETRIEVAL AUGMENTED GENERATION) EXAMPLE")
    print("=" * 70)
    
    # Create knowledge base
    kb = create_knowledge_base()
    print("\n✅ Created knowledge base with 4 documents")
    
    # Create RAG agent
    agent = create_rag_agent(kb)
    print("✅ Created RAG agent with retrieval capability")
    
    # Define workflow step
    workflow = [
        Step(
            title="Retrieve Information",
            description="Search knowledge base for relevant documents"
        ),
        Step(
            title="Generate Answer",
            description="Use retrieved information to generate answer"
        )
    ]
    
    # Example queries
    queries = [
        "What is Python and why is it popular?",
        "How does async/await work in Python?",
        "What does machine learning need to be successful?"
    ]
    
    print("\n" + "=" * 70)
    print("QUERYING KNOWLEDGE BASE")
    print("=" * 70)
    
    for i, query in enumerate(queries, 1):
        print(f"\n📌 Query {i}: {query}")
        print("-" * 70)
        
        result = await agent.aprocess(workflow, query)
        
        print(f"\n✅ Status: {result.get('status')}")
        print(f"📚 Tools Used: {result.get('tool_calls', 0)}")
        
        answer = result.get('final_answer', '')
        print(f"\n📝 Answer:\n{answer}\n")

if __name__ == "__main__":
    asyncio.run(main())
```

## Running the Example

```bash
pip install nkit openai
export OPENAI_API_KEY="sk-..."
python rag_example.py
```

## Expected Output

```
======================================================================
RAG (RETRIEVAL AUGMENTED GENERATION) EXAMPLE
======================================================================

✅ Created knowledge base with 4 documents
✅ Created RAG agent with retrieval capability

======================================================================
QUERYING KNOWLEDGE BASE
======================================================================

📌 Query 1: What is Python and why is it popular?
──────────────────────────────────────────────────────────────────────

✅ Status: success
📚 Tools Used: 1

📝 Answer:
Based on the knowledge base, Python is a high-level programming language 
known for its simplicity and readability. This makes it popular for various 
applications including web development, data science, and automation.

Source: tech-docs
```

## Key Concepts

### Knowledge Base

```python
kb = KnowledgeBase()

# Add documents
kb.add_document(Document(
    id="doc1",
    content="Information about Python",
    source="tech-docs",
    metadata={"topic": "programming"}
))

# Search documents
results = kb.search("Python programming", limit=3)
```

### Retrieval Tool

```python
def retrieve_documents(query: str, limit: int = 3) -> list:
    """Retrieve relevant documents"""
    results = knowledge_base.search(query, limit=limit)
    return [{"id": doc.id, "content": doc.content} for doc in results]

tool = Tool(
    name="retrieve_documents",
    description="Search knowledge base",
    callback=retrieve_documents
)

agent.add_tool(tool)
```

### RAG Flow

```
User Query
    ↓
[retrieve_documents tool]
    ↓
Retrieved Documents
    ↓
[agent reasoning]
    ↓
Context-Aware Answer
    ↓
Response with Citations
```

## Advanced RAG Patterns

### Vector Embeddings

```python
from nkit.knowledge import VectorKnowledgeBase

# Use vector embeddings for semantic search
kb = VectorKnowledgeBase(
    embedding_model="openai",
    similarity_threshold=0.7
)

results = kb.semantic_search("machine learning basics")
```

### Chunking Documents

```python
from nkit.knowledge import Document

# Large documents are automatically chunked
doc = Document(
    id="large_doc",
    content="Very large document with lots of text...",
    chunk_size=500,  # Max 500 chars per chunk
    overlap=50  # 50 char overlap between chunks
)

kb.add_document(doc)
```

### Hybrid Search

```python
# Search by both keywords and semantics
results = kb.hybrid_search(
    query="Python async programming",
    keyword_weight=0.4,
    semantic_weight=0.6,
    limit=5
)
```

## Advanced Features

### Multiple Knowledge Bases

```python
kb_python = KnowledgeBase(name="python")
kb_ai = KnowledgeBase(name="ai")

# Multi-KB retrieval
def multi_retrieve(query):
    python_results = kb_python.search(query, limit=2)
    ai_results = kb_ai.search(query, limit=2)
    return python_results + ai_results

agent.add_tool(Tool(
    name="retrieve_all",
    description="Search all knowledge bases",
    callback=multi_retrieve
))
```

### Filtering Results

```python
# Filter by metadata
results = kb.search(
    query="machine learning",
    filters={"topic": "machine-learning"},
    limit=5
)

# Filter by date
results = kb.search(
    query="recent trends",
    filters={"created_after": "2024-01-01"}
)
```

### Re-ranking Results

```python
# Re-rank results by relevance or recency
results = kb.search("Python", limit=10)
results = kb.rerank(
    results,
    strategy="recency",  # or "relevance"
    top_k=3
)
```

## Best Practices

### 1. Quality Documents

```python
# Good - clear, focused documents
doc = Document(
    id="python_async",
    content="Async/await enables concurrent programming in Python...",
    source="official-docs",
    metadata={
        "topic": "python",
        "subtopic": "async",
        "difficulty": "intermediate"
    }
)

# Avoid - vague, long documents
doc = Document(
    id="stuff",
    content="Python is cool. It has many features. You can do things..."
)
```

### 2. Proper Metadata

```python
doc = Document(
    id="doc1",
    content="...",
    source="source-name",
    metadata={
        "topic": "machine-learning",
        "subtopic": "neural-networks",
        "difficulty": "advanced",
        "created_date": "2024-01-15",
        "author": "researcher"
    }
)
```

### 3. Effective Prompts

```python
# Guide agent to use retrieval
agent.system_prompt = """
Use the retrieve_documents tool for factual information.
Always cite sources from retrieved documents.
If information is not in knowledge base, say so clearly.
"""
```

## Troubleshooting

### Poor Retrieval Results

```python
# 1. Improve document quality
# 2. Add more relevant documents
# 3. Adjust query
# 4. Use better embeddings

kb = VectorKnowledgeBase(
    embedding_model="openai",  # Better embeddings
    similarity_threshold=0.5  # Lower threshold
)
```

### Agent Not Using Tool

```python
# Make sure description is clear
Tool(
    name="retrieve",
    description="Search knowledge base for documents about the topic",
    callback=retrieve_documents
)
```

## Next Steps

- **[Custom Tools](../core-concepts/tools.md)** - Create specialized tools
- **[Memory Systems](../core-concepts/memory.md)** - Persistent state
- **[Advanced RAG](../advanced/advanced-rag.md)** - Expert patterns
- **[Examples](../examples/index.md)** - More examples
