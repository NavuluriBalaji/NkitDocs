# Contributing

Thank you for your interest in contributing to nkit! Here's how you can help.

## Getting Started

### Fork and Clone

```bash
git clone https://github.com/yourusername/nkit
cd nkit
pip install -e ".[dev]"
```

### Development Setup

```bash
# Install dev dependencies
pip install pytest pytest-asyncio black flake8 mypy

# Run tests
pytest

# Format code
black .

# Check linting
flake8 .

# Type checking
mypy .
```

## Contributing Code

### Code Style

We follow PEP 8 with these conventions:

- **Line length**: 100 characters
- **Indentation**: 4 spaces
- **Type hints**: Required for all public functions
- **Docstrings**: Google-style docstrings

Example:

```python
async def process_agent(
    agent: Agent,
    steps: list[Step],
    input_text: str
) -> dict:
    """Process agent execution.
    
    Args:
        agent: The agent to execute
        steps: List of workflow steps
        input_text: Input task description
    
    Returns:
        Dictionary with execution results containing:
        - status: Execution status (success/error/timeout)
        - final_answer: Agent's final response
        - iterations: Number of reasoning iterations
        - tool_calls: Number of tool invocations
    
    Raises:
        TimeoutError: If execution exceeds agent.timeout
        ValueError: If invalid inputs provided
    """
    # Implementation
    pass
```

### Testing

Write tests for your changes:

```python
import pytest
from nkit import Agent

@pytest.mark.asyncio
async def test_agent_execution():
    agent = Agent(name="TestBot")
    result = await agent.aprocess([], "test input")
    assert result["status"] == "success"

def test_agent_configuration():
    agent = Agent(
        name="Bot",
        temperature=0.5,
        max_iterations=10
    )
    assert agent.temperature == 0.5
    assert agent.max_iterations == 10
```

Run tests:

```bash
pytest -v
pytest tests/test_agent.py -v
pytest tests/test_agent.py::test_agent_execution -v
```

### Pull Request Process

1. **Create a branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**
   - Update code following style guide
   - Add/update tests
   - Update documentation

3. **Commit with clear messages**
   ```bash
   git commit -m "Add agent timeout feature"
   ```

4. **Push and create PR**
   ```bash
   git push origin feature/your-feature-name
   ```

5. **PR Description**
   Include:
   - What changes you made
   - Why you made them
   - Related issues
   - Testing notes

### Commit Messages

Follow conventional commits:

```
feat: add new feature
fix: fix a bug
docs: update documentation
test: add/update tests
refactor: refactor code
perf: improve performance
```

Examples:
- `feat: add memory pruning to reduce token usage`
- `fix: resolve agent timeout on network errors`
- `docs: add RAG example to examples/`

## Contributing Documentation

### Writing Docs

Documentation lives in `docs/docs/`:

```
docs/docs/
├── index.md                 # Homepage
├── getting-started/         # Quick start guides
├── core-concepts/           # Deep dives
├── api-reference/           # Auto-generated from code
├── examples/                # Working examples
├── advanced/                # Expert topics
└── faq.md                   # Common questions
```

### Style Guide

- **Headings**: Use H1 (#), H2 (##), H3 (###)
- **Code blocks**: Language-specific (python, bash)
- **Links**: Use relative links `[text](path.md)`
- **Line length**: ~80 characters for readability

Example:

```markdown
# Feature Title

Brief description of the feature.

## How It Works

Explanation with diagrams if helpful.

## Example

```python
# Code example
```

## Best Practices

- Keep explanations clear and concise
- Include working code examples
- Add diagrams for complex flows
- Link to related sections
```

### Building Docs Locally

```bash
pip install mkdocs mkdocs-material pymdown-extensions

cd docs
mkdocs serve  # http://localhost:8000
```

### Updating Examples

When adding new examples:

1. Create file in `docs/docs/examples/`
2. Include full working code
3. Add "Running the Example" section
4. Link from relevant concept pages
5. Update `docs/mkdocs.yml` nav

## Reporting Issues

### Bug Reports

Include:

```markdown
**Describe the bug**
Clear description of what went wrong

**Reproduce**
```python
# Minimal code to reproduce
```

**Expected behavior**
What should happen

**Actual behavior**
What actually happens

**Environment**
- Python version: 3.10
- nkit version: 0.1.0
- OS: Windows/Mac/Linux
```

### Feature Requests

```markdown
**Is your feature request related to a problem?**
Description

**Describe the solution you'd like**
What should be added

**Describe alternatives**
Other approaches considered

**Additional context**
Any other context
```

## Code Review

### What Reviewers Look For

- ✅ Code follows style guide
- ✅ Tests added/updated
- ✅ Documentation updated
- ✅ No breaking changes
- ✅ Performance considered
- ✅ Error handling included

### Responding to Feedback

- Ask for clarification if needed
- Implement suggestions
- Push updates to branch
- Mark conversations resolved after changes

## Development Workflows

### Adding a Feature

1. Check existing issues to avoid duplicates
2. Discuss design (for large features)
3. Create feature branch
4. Implement with tests
5. Update documentation
6. Submit PR
7. Address feedback

### Fixing a Bug

1. Create minimal reproduction
2. Write failing test
3. Implement fix
4. Verify test passes
5. Update changelog
6. Submit PR

### Documentation Changes

1. Fork repository
2. Edit relevant files
3. Build docs locally to verify
4. Submit PR
5. No code review needed, auto-merge

## Release Process

Maintainers handle releases:

1. Merge PRs to main
2. Update version in `setup.py`
3. Update `CHANGELOG.md`
4. Create GitHub release
5. Package and publish to PyPI

Release versions follow [Semantic Versioning](https://semver.org/):
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes

## Areas We Need Help

### High Priority

- [ ] GPU/CUDA support documentation
- [ ] Cloud deployment guides (AWS, GCP, Azure)
- [ ] Performance optimization guides
- [ ] More LLM provider integrations

### Medium Priority

- [ ] Expanded examples (finance, healthcare, etc.)
- [ ] Benchmark suite
- [ ] Integration tests
- [ ] Visual debugging tools

### Low Priority

- [ ] VSCode extension
- [ ] Web UI dashboard
- [ ] Mobile support
- [ ] Language bindings (Go, Rust, etc.)

## Community Guidelines

### Be Respectful

- Treat everyone with respect
- No harassment or discrimination
- Assume good intentions
- Give constructive feedback

### Be Helpful

- Help answer questions
- Share knowledge
- Review PRs from newcomers
- Mentor new contributors

### Be Collaborative

- Share ideas openly
- Discuss design decisions
- Credit contributions
- Give credit to others

## Resources

- **Code of Conduct**: [CODE_OF_CONDUCT.md](../code-of-conduct.md)
- **Architecture**: [ARCHITECTURE.md](../architecture.md)
- **API Reference**: [API Docs](../api-reference/index.md)
- **Examples**: [Examples](../examples/index.md)

## Questions?

- 📖 Check existing docs
- 💬 Ask in discussions
- 🐛 Search existing issues
- 📧 Contact maintainers

## Thank You!

We appreciate your contributions! 🙏
