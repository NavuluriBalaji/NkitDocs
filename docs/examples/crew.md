# Crew Example

A complete working example of multiple agents coordinating to research and create reports.

## Full Code

```python
"""
Crew Example: Research and Report Generation
"""

import asyncio
from nkit.crews import Crew
from nkit.tasks import Task, TaskManager
from nkit import Agent
from nkit.tools import Tool

# Tools
def search_topic(query: str) -> list:
    """Search for information about a topic"""
    knowledge = {
        "AI": ["AI is transforming industries", "Machine learning powers AI"],
        "ML": ["ML learns from data", "Deep learning is powerful"],
    }
    results = []
    for topic, facts in knowledge.items():
        if query.lower() in topic.lower():
            results.extend(facts)
    return results

# Create specialized agents
def create_crew():
    researcher = Agent(
        name="ResearchBot",
        description="Conducts research",
        model="gpt-3.5-turbo",
        temperature=0.3
    )
    researcher.add_tool(Tool(
        name="search",
        description="Search for information",
        callback=search_topic
    ))
    
    analyst = Agent(
        name="AnalysisBot",
        description="Analyzes findings",
        model="gpt-3.5-turbo"
    )
    
    writer = Agent(
        name="WriterBot",
        description="Creates reports",
        model="gpt-3.5-turbo"
    )
    
    # Create crew
    crew = Crew(
        name="ResearchCrew",
        description="Research team",
        agents=[researcher, analyst, writer],
        process_type="sequential",
        verbose=True
    )
    
    return crew

async def main():
    crew = create_crew()
    
    print("=" * 60)
    print("RESEARCH CREW EXAMPLE")
    print("=" * 60)
    
    topics = ["Artificial Intelligence", "Machine Learning"]
    
    for topic in topics:
        print(f"\n📚 Researching: {topic}\n")
        
        result = await crew.aprocess(f"Research {topic} and create a report")
        
        print(f"✅ Status: {result.get('status')}")
        print(f"📊 Agents: {len(crew.agents)}")
        print(f"📝 Result: {result.get('final_answer')}\n")

if __name__ == "__main__":
    asyncio.run(main())
```

## Running the Example

```bash
pip install nkit openai
export OPENAI_API_KEY="sk-..."
python crew_example.py
```

## Key Concepts

### Specialized Agents

```python
researcher = Agent(name="ResearchBot", description="Researches topics")
analyst = Agent(name="AnalysisBot", description="Analyzes data")
writer = Agent(name="WriterBot", description="Writes reports")
```

### Crew Coordination

```python
crew = Crew(
    agents=[researcher, analyst, writer],
    process_type="sequential"  # Execute one after another
)
```

### Sequential Workflow

```
ResearchBot → AnalysisBot → WriterBot
```

## Advanced Variations

### Parallel Execution

```python
crew = Crew(
    agents=[agent1, agent2, agent3],
    process_type="parallel"  # All run simultaneously
)
```

### Hierarchical with Manager

```python
crew = Crew(
    agents=[...],
    manager=Agent(name="ProjectManager"),
    process_type="hierarchical"  # Manager coordinates
)
```

## Next Steps

- **[Task Example](tasks.md)** - Task dependencies and workflows
- **[RAG Example](rag.md)** - Knowledge base integration
- **[Core Concepts](../core-concepts/crews.md)** - Detailed guide
