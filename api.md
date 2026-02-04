---
layout: default
title: API Reference
nav_order: 6
has_children: true
description: "Complete API reference for Flouri.sh"
permalink: /api/
---

# API Reference

Complete API documentation for Flouri.sh, including CLI commands, Python API, tools, and agent functions.

## CLI API

### `flouri`

Launch the interactive TUI (default when run without arguments).

```bash
flouri
# or
flouri tui
```

**Options**: None

**Behavior**: Launches an interactive terminal where you can:
- Type commands normally
- Prefix with `?` to ask the AI
- Use `Ctrl+A` as an alternative to `?`
- Use `Tab` for completion, `↑/↓` for history

### `flouri agent`

Run the agent in CLI mode (non-interactive).

```bash
flouri agent "your prompt" [OPTIONS]
```

**Arguments**:
- `prompt` (required): The question or task for the agent

**Options**:
- `--allowlist`, `-a`: Comma-separated list of allowed commands
  ```bash
  flouri agent --allowlist "ls,cd,git" "Check git status"
  ```

- `--blacklist`, `-b`: Comma-separated list of blacklisted commands
  ```bash
  flouri agent --blacklist "rm,dd" "Help me organize files"
  ```

- `--stream`, `-s`: Enable live streaming output
  ```bash
  flouri agent --stream "Explain Docker networking"
  ```

**Examples**:
```bash
# Basic usage
flouri agent "List files and show git status"

# With streaming
flouri agent --stream "Explain how Docker networking works"

# With custom allowlist
flouri agent --allowlist "ls,cd,git" "Check git status"

# With blacklist
flouri agent --blacklist "rm,dd" "Help me organize files"
```

## Python API

### Runner Functions

#### `run_agent()`

Run the agent asynchronously.

```python
from flouri.runner import run_agent

response = await run_agent(
    prompt: str,
    allowed_commands: list[str] | None = None,
    blacklisted_commands: list[str] | None = None,
) -> str
```

**Parameters**:
- `prompt` (str): The user's prompt
- `allowed_commands` (list[str] | None): Optional list of allowed commands
- `blacklisted_commands` (list[str] | None): Optional list of blacklisted commands

**Returns**: The agent's response text (str)

**Example**:
```python
import asyncio
from flouri.runner import run_agent

async def main():
    response = await run_agent("List files in current directory")
    print(response)

asyncio.run(main())
```

#### `run_agent_sync()`

Synchronous wrapper for `run_agent()`.

```python
from flouri.runner import run_agent_sync

response = run_agent_sync(
    prompt: str,
    allowed_commands: list[str] | None = None,
    blacklisted_commands: list[str] | None = None,
) -> str
```

**Example**:
```python
from flouri.runner import run_agent_sync

response = run_agent_sync("List files in current directory")
print(response)
```

#### `run_agent_live()`

Run the agent with live streaming output (async).

```python
from flouri.runner import run_agent_live

async def stream_callback(text: str):
    print(text, end="", flush=True)

response = await run_agent_live(
    prompt: str,
    allowed_commands: list[str] | None = None,
    blacklisted_commands: list[str] | None = None,
    stream_callback: Callable[[str], None] | None = None,
) -> str
```

**Parameters**:
- `prompt` (str): The user's prompt
- `allowed_commands` (list[str] | None): Optional list of allowed commands
- `blacklisted_commands` (list[str] | None): Optional list of blacklisted commands
- `stream_callback` (Callable[[str], None] | None): Optional callback for each text chunk

**Returns**: The complete agent's response text (str)

#### `run_agent_live_sync()`

Synchronous wrapper for `run_agent_live()`.

```python
from flouri.runner import run_agent_live_sync

def stream_callback(text: str):
    print(text, end="", flush=True)

response = run_agent_live_sync(
    prompt: str,
    allowed_commands: list[str] | None = None,
    blacklisted_commands: list[str] | None = None,
    stream_callback: Callable[[str], None] | None = None,
) -> str
```

### Agent Functions

#### `get_agent()`

Create and return the agent with code execution capabilities.

```python
from flouri.agent import get_agent

agent = get_agent(
    allowed_commands: list[str] | None = None,
    blacklisted_commands: list[str] | None = None,
) -> LlmAgent
```

**Parameters**:
- `allowed_commands` (list[str] | None): Optional list of allowed commands
- `blacklisted_commands` (list[str] | None): Optional list of blacklisted commands

**Returns**: `LlmAgent` instance

### Configuration Functions

#### `get_settings()`

Get application settings.

```python
from flouri.config import get_settings

settings = get_settings() -> Settings
```

**Returns**: `Settings` object with:
- `api_key` (str): LLM API key
- `model` (str): Model name
- `app_name` (str): Application name
- `user_id` (str): User identifier
- `session_id` (str): Session identifier

### UI Functions

#### `run_tui()`

Launch the interactive TUI.

```python
from flouri.ui import run_tui

run_tui()
```

## Tools API

Tools are functions that the AI agent can call. They are organized by skill category.

### Bash Tools

#### `execute_bash()`

Execute a shell command.

```python
from flouri.tools import execute_bash

result = execute_bash(
    cmd: str,
    tool_context: ToolContext | None = None,
) -> dict[str, Any]
```

**Parameters**:
- `cmd` (str): The shell command to execute
- `tool_context` (ToolContext | None): Tool context (optional)

**Returns**: Dictionary with:
- `status` (str): "success" or "error"
- `stdout` (str): Standard output
- `stderr` (str): Standard error
- `exit_code` (int): Exit code
- `cmd` (str): The executed command

**Security**: Command is validated against allowlist/blacklist before execution.

#### `set_cwd()`

Set the global working directory.

```python
from flouri.tools import set_cwd

result = set_cwd(path: str) -> str
```

**Parameters**:
- `path` (str): Absolute path to the new working directory

**Returns**: Confirmation message (str)

**Raises**: `ValueError` if path is not a valid directory

#### `get_user()`

Get current user information.

```python
from flouri.tools import get_user

result = get_user() -> dict[str, Any]
```

**Returns**: Dictionary with:
- `username` (str): Current username
- `home_directory` (str): Home directory path
- `current_working_directory` (str): Current working directory

### Config Tools

#### `add_to_allowlist()`

Add a command to the allowlist.

```python
from flouri.tools import add_to_allowlist

result = add_to_allowlist(
    command: str,
    tool_context: ToolContext | None = None,
) -> dict
```

**Parameters**:
- `command` (str): Base command to add (e.g., "ls", "git")
- `tool_context` (ToolContext | None): Tool context (optional)

**Returns**: Dictionary with status, message, and updated allowlist

#### `remove_from_allowlist()`

Remove a command from the allowlist.

```python
from flouri.tools import remove_from_allowlist

result = remove_from_allowlist(
    command: str,
    tool_context: ToolContext | None = None,
) -> dict
```

#### `add_to_blacklist()`

Add a command to the blacklist.

```python
from flouri.tools import add_to_blacklist

result = add_to_blacklist(
    command: str,
    tool_context: ToolContext | None = None,
) -> dict
```

#### `remove_from_blacklist()`

Remove a command from the blacklist.

```python
from flouri.tools import remove_from_blacklist

result = remove_from_blacklist(
    command: str,
    tool_context: ToolContext | None = None,
) -> dict
```

#### `list_allowlist()`

List all commands in the allowlist.

```python
from flouri.tools import list_allowlist

result = list_allowlist() -> dict
```

**Returns**: Dictionary with status, allowlist, and count

#### `list_blacklist()`

List all commands in the blacklist.

```python
from flouri.tools import list_blacklist

result = list_blacklist() -> dict
```

**Returns**: Dictionary with status, blacklist, and count

#### `is_in_allowlist()`

Check if a command is in the allowlist.

```python
from flouri.tools import is_in_allowlist

result = is_in_allowlist(command: str) -> dict
```

**Returns**: Dictionary with status, command, base_command, in_allowlist, and matched_entry

#### `is_in_blacklist()`

Check if a command is in the blacklist.

```python
from flouri.tools import is_in_blacklist

result = is_in_blacklist(command: str) -> dict
```

**Returns**: Dictionary with status, command, base_command, in_blacklist, and matched_entry

### History Tools

#### `read_bash_history()`

Read bash command history.

```python
from flouri.tools import read_bash_history

result = read_bash_history(
    limit: int = 50,
    tool_context: ToolContext | None = None,
) -> dict
```

**Parameters**:
- `limit` (int): Maximum number of history entries to return
- `tool_context` (ToolContext | None): Tool context (optional)

**Returns**: Dictionary with status and history entries

#### `read_conversation_history()`

Read conversation history.

```python
from flouri.tools import read_conversation_history

result = read_conversation_history(
    limit: int = 50,
    tool_context: ToolContext | None = None,
) -> dict
```

**Parameters**:
- `limit` (int): Maximum number of history entries to return
- `tool_context` (ToolContext | None): Tool context (optional)

**Returns**: Dictionary with status and conversation entries

#### `get_tool_call_stats()`

Get tool usage statistics.

```python
from flouri.tools import get_tool_call_stats

result = get_tool_call_stats(
    tool_context: ToolContext | None = None,
) -> dict
```

**Returns**: Dictionary with tool usage statistics

### System Tools

#### `get_current_datetime()`

Get current date and time.

```python
from flouri.tools import get_current_datetime

result = get_current_datetime(
    tool_context: ToolContext | None = None,
) -> dict
```

**Returns**: Dictionary with current datetime information

### Tool Manager Tools

#### `get_available_tools()`

Get list of all available tools.

```python
from flouri.tools import get_available_tools

result = get_available_tools(
    tool_context: ToolContext | None = None,
) -> dict
```

**Returns**: Dictionary with list of available tools

#### `list_enabled_tools()`

List currently enabled tools.

```python
from flouri.tools import list_enabled_tools

result = list_enabled_tools(
    tool_context: ToolContext | None = None,
) -> dict
```

**Returns**: Dictionary with list of enabled tools

#### `enable_tool()`

Enable a tool.

```python
from flouri.tools import enable_tool

result = enable_tool(
    tool_name: str,
    tool_context: ToolContext | None = None,
) -> dict
```

**Parameters**:
- `tool_name` (str): Name of the tool to enable
- `tool_context` (ToolContext | None): Tool context (optional)

**Returns**: Dictionary with status and message

#### `disable_tool()`

Disable a tool.

```python
from flouri.tools import disable_tool

result = disable_tool(
    tool_name: str,
    tool_context: ToolContext | None = None,
) -> dict
```

**Parameters**:
- `tool_name` (str): Name of the tool to disable
- `tool_context` (ToolContext | None): Tool context (optional)

**Returns**: Dictionary with status and message

## Plugin API

### Base Classes

#### `Plugin`

Base class for command handler plugins.

```python
from flouri.plugins import Plugin

class MyPlugin(Plugin):
    def name(self) -> str:
        return "my_plugin"

    def should_handle(self, command: str) -> bool:
        return command.startswith("mycommand")

    async def execute(self, command: str, cwd: str) -> dict[str, Any]:
        return {
            "handled": True,
            "output": "Plugin output",
            "exit_code": 0,
        }
```

#### `CommandEnhancer`

Base class for command output enhancers.

```python
from flouri.plugins.enhancers import CommandEnhancer

class MyEnhancer(CommandEnhancer):
    def name(self) -> str:
        return "my_enhancer"

    def should_enhance(self, command: str) -> bool:
        return command.startswith("mycommand")

    def enhance_output(
        self,
        command: str,
        stdout: str,
        stderr: str,
        exit_code: int,
        cwd: str,
    ) -> dict[str, Any]:
        return {
            "enhanced": True,
            "stdout": enhanced_stdout,
            "stderr": stderr,
            "hints": [],
        }
```

See [Plugins Documentation]({{ site.baseurl }}/plugins/) for more details.

## Configuration API

### `ConfigManager`

Manage configuration files.

```python
from flouri.config.config_manager import ConfigManager

config_manager = ConfigManager()

# Add to allowlist
config_manager.add_to_allowlist("ls")

# Remove from allowlist
config_manager.remove_from_allowlist("ls")

# Add to blacklist
config_manager.add_to_blacklist("rm")

# Remove from blacklist
config_manager.remove_from_blacklist("rm")

# Get allowlist
allowlist = config_manager.get_allowlist()

# Get blacklist
blacklist = config_manager.get_blacklist()
```

## Error Handling

All functions may raise exceptions. Common exceptions:

- `ValueError`: Invalid input (e.g., invalid directory path)
- `RuntimeError`: Runtime errors during agent execution
- `KeyError`: Missing configuration keys

Always wrap API calls in try/except blocks:

```python
try:
    response = run_agent_sync("your prompt")
except Exception as e:
    print(f"Error: {e}")
```
