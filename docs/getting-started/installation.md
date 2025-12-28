# Installation

## Requirements

- Python 3.8+
- pip or conda

## Install from PyPI

```bash
pip install nkit
```

## Install for Development

Clone the repository and install in editable mode:

```bash
git clone https://github.com/yourusername/nkit
cd nkit
pip install -e .
```

## Optional Dependencies

### For LLM Support

```bash
# OpenAI
pip install openai

# Anthropic (Claude)
pip install anthropic

# Ollama (local models)
# Download from https://ollama.ai
```

### For Knowledge Base

```bash
# OpenAI Embeddings
pip install openai

# Hugging Face Embeddings
pip install sentence-transformers

# Vector Store Support
pip install pinecone-client  # Pinecone
pip install weaviate-client  # Weaviate
pip install chromadb         # Chroma
```

### For Documentation

```bash
pip install mkdocs mkdocs-material pymdown-extensions mkdocstrings[python]
```

## Verify Installation

```python
from nkit import Agent, Step
print("✓ nkit installed successfully!")
```

## Next Steps

- **[Quick Start](quick-start.md)** - Create your first agent
- **[First Agent](first-agent.md)** - Step-by-step tutorial
- **[Core Concepts](../core-concepts/architecture.md)** - Understand the framework

## Troubleshooting

### ImportError: No module named 'nkit'

Make sure you installed nkit:
```bash
pip install nkit
```

### LLM Provider Not Available

Install the specific provider:
```bash
pip install openai  # For OpenAI
pip install anthropic  # For Claude
```

### Issues with Async

Ensure you're using Python 3.8+:
```bash
python --version
```

## Getting Help

- 📖 Read the [documentation](../index.md)
- 🤔 Check the [FAQ](../faq.md)
- 🐛 Report issues on [GitHub](https://github.com/yourusername/nkit/issues)
