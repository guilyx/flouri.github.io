---
layout: default
title: Developer Guide
nav_order: 8
description: "Developer guide for contributing to Flouri.sh"
permalink: /developer/
---

# Developer Guide

This guide is for developers who want to contribute to Flouri.sh or extend its functionality.

## Development Setup

### Prerequisites

- Python 3.10 or higher
- Git
- [uv](https://github.com/astral-sh/uv) (recommended) or pip

### Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/guilyx/flouri.git
   cd flouri
   ```

2. **Install dependencies**:
   ```bash
   # Using uv (recommended)
   curl -LsSf https://astral.sh/uv/install.sh | sh
   uv sync --dev

   # Or using pip
   pip install -e ".[dev]"
   ```

3. **Install pre-commit hooks**:
   ```bash
   uv run pre-commit install
   # or
   pre-commit install
   ```

4. **Set up environment variables**:
   ```bash
   cp env.example .env
   # Edit .env with your API key
   ```

## Project Structure

```
flouri/
├── flouri/              # Main package
│   ├── __init__.py
│   ├── agent/            # Agent definitions
│   │   └── agents.py
│   ├── completions/      # Command completion system
│   │   ├── loader.py
│   │   ├── registry.py
│   │   ├── git.py
│   │   └── ros2.py
│   ├── config/           # Configuration management
│   │   ├── config.py
│   │   └── config_manager.py
│   ├── logging/          # Logging utilities
│   │   └── logger.py
│   ├── plugins/          # Plugin system
│   │   ├── base.py
│   │   ├── cd_completer.py
│   │   ├── enhancers.py
│   │   ├── zsh_bindings.py
│   │   └── __init__.py
│   ├── runner/           # Agent execution
│   │   └── runner.py
│   ├── tools/            # Agent tools
│   │   ├── base.py
│   │   ├── registry.py
│   │   ├── globals.py
│   │   ├── bash/         # Bash execution tools
│   │   ├── config/       # Configuration tools
│   │   ├── history/     # History tools
│   │   ├── system/      # System tools
│   │   ├── ros2/        # ROS2 tools (optional)
│   │   └── tool_manager/ # Tool management
│   └── ui/               # User interfaces
│       ├── cli.py
│       ├── tui.py
│       └── banner.py
├── config/               # Configuration files
│   └── config.json
├── docs/                  # Documentation
│   ├── architecture.md
│   ├── plugins.md
│   └── third-party-libraries.md
├── tests/                 # Test files
│   ├── unit/
│   └── test_*.py
├── pyproject.toml         # Project configuration
└── README.md
```

## Code Style

### Formatting

- Use `black` for formatting (line length: 100)
- Use `ruff` for linting
- Maximum line length: 100 characters

### Type Hints

- Use type hints where appropriate
- Use `typing` module for complex types
- Use `|` for union types (Python 3.10+)

### Example

```python
from typing import Any

def my_function(param: str, optional: int | None = None) -> dict[str, Any]:
    """Function with type hints."""
    return {"result": param}
```

## Running Tests

```bash
# Run all tests
uv run pytest
# or
pytest

# Run with coverage
uv run pytest --cov=flouri --cov-report=html

# Run specific test file
uv run pytest tests/test_tools.py

# Run specific test
uv run pytest tests/test_tools.py::test_execute_bash
```

## Code Quality Checks

```bash
# Run all checks
uv run ruff check .
uv run black --check .
uv run mypy flouri
uv run pytest

# Auto-fix issues
uv run ruff check --fix .
uv run black .
```

## Adding New Tools

### Step 1: Create Tool Function

Create a new file in the appropriate subdirectory (e.g., `flouri/tools/my_skill/my_tool.py`):

```python
"""My custom tool."""

import time
from typing import Any

from google.adk.tools import ToolContext

from ...logging import log_tool_call


def my_tool(param: str, tool_context: ToolContext | None = None) -> dict[str, Any]:
    """Description of what the tool does.

    Args:
        param: Description of parameter
        tool_context: Tool context (optional)

    Returns:
        Dictionary with tool results
    """
    t0 = time.perf_counter()

    try:
        # Your tool logic here
        result = {
            "status": "success",
            "data": "result data",
        }

        log_tool_call(
            "my_tool",
            {"param": param},
            result,
            success=True,
            duration_seconds=time.perf_counter() - t0,
        )
        return result
    except Exception as e:
        error_result = {
            "status": "error",
            "message": str(e),
        }
        log_tool_call(
            "my_tool",
            {"param": param},
            error_result,
            success=False,
            duration_seconds=time.perf_counter() - t0,
        )
        return error_result
```

### Step 2: Create Skill Module

Create `flouri/tools/my_skill/skill.py`:

```python
"""My skill module."""

from .my_tool import my_tool
from ..base import BaseSkill, FunctionToolWrapper


class MySkill(BaseSkill):
    """My custom skill."""

    def name(self) -> str:
        return "my_skill"

    def description(self) -> str:
        return "Description of my skill"

    def get_tools(self) -> list[FunctionToolWrapper]:
        return [
            FunctionToolWrapper(
                name="my_tool",
                description="Description of my tool",
                function=my_tool,
            ),
        ]
```

### Step 3: Register Skill

Add to `flouri/tools/registry.py`:

```python
from .my_skill.skill import MySkill

def _register_all_skills(registry: SkillRegistry):
    """Register all skills in the registry."""
    registry.register(BashSkill())
    registry.register(ConfigSkill())
    # ... other skills ...
    registry.register(MySkill())  # Add your skill
```

### Step 4: Export Tool

Add to `flouri/tools/__init__.py`:

```python
from .my_skill import my_tool

__all__ = [
    # ... other tools ...
    "my_tool",
]
```

## Adding New Plugins

See [Plugins Documentation]({{ site.baseurl }}/plugins/) for detailed information on creating plugins.

## Writing Tests

### Test Structure

```python
"""Tests for my module."""

import pytest
from flouri.my_module import my_function


def test_my_function_success():
    """Test successful execution."""
    result = my_function("test")
    assert result["status"] == "success"


def test_my_function_error():
    """Test error handling."""
    with pytest.raises(ValueError):
        my_function("")


@pytest.mark.asyncio
async def test_async_function():
    """Test async function."""
    result = await my_async_function("test")
    assert result is not None
```

### Running Tests

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run with coverage
pytest --cov=flouri --cov-report=html

# Run specific test
pytest tests/test_my_module.py::test_my_function
```

## Debugging

### Logging

Flouri uses structured logging. Logs are stored in `~/.config/flouri/logs/`.

```python
from flouri.logging import log_tool_call, log_conversation

# Log tool call
log_tool_call("tool_name", {"param": "value"}, result, success=True)

# Log conversation
log_conversation("user", "user message")
log_conversation("agent", "agent response")
```

### Debug Mode

Set environment variable for debug output:

```bash
export DEBUG=1
flouri
```

## Contributing

### Workflow

1. **Fork the repository**
2. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes**
4. **Run checks**:
   ```bash
   uv run ruff check .
   uv run black --check .
   uv run pytest
   ```
5. **Commit your changes**:
   ```bash
   git commit -m "Short commit message (max 70 chars)" \
              -m "Extended description of what and why"
   ```
6. **Push and create a Pull Request**

### Commit Message Guidelines

- First line: 70 characters max, imperative mood
- Second line: Extended description (unlimited)
- Reference issues: `Fixes #123` or `Closes #456`

Example:
```
Add new tool for file operations

Implements a new tool that allows the agent to perform
file operations with proper error handling and logging.
Includes comprehensive tests and documentation.

Fixes #123
```

### Pull Request Checklist

- [ ] Code follows style guidelines (black, ruff)
- [ ] Tests added/updated and passing
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
- [ ] Commit messages follow guidelines

## Architecture Decisions

### Why Google ADK?

Google ADK provides:
- Agent framework and orchestration
- Tool calling infrastructure
- Session management
- Multi-modal support

### Why LiteLLM?

LiteLLM provides:
- Unified interface for multiple LLM providers
- Easy switching between providers
- Consistent API across providers

### Why prompt-toolkit?

prompt-toolkit provides:
- Rich terminal UI capabilities
- Command completion
- History management
- Cross-platform support

## Performance Considerations

- **Tool Execution**: Tools should be fast - they're called frequently
- **Plugin Checks**: Plugin `should_handle()` should be efficient
- **Logging**: Use structured logging, avoid excessive logging
- **Async**: Use async/await for I/O operations

## Security Considerations

- **Command Validation**: Always validate commands before execution
- **Input Sanitization**: Sanitize user input in tools
- **Error Messages**: Don't expose sensitive information in errors
- **API Keys**: Never log or expose API keys

## Questions?

- Check [Architecture Documentation]({{ site.baseurl }}/architecture/)
- Review [Contributing Guidelines](https://github.com/guilyx/flouri/blob/main/CONTRIBUTING.md)
- Open a [Discussion](https://github.com/guilyx/flouri/discussions)
- Check existing code for examples
