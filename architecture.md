---
layout: default
title: Architecture
nav_order: 5
description: "System architecture and design of Flouri.sh"
permalink: /architecture/
---

# Architecture

Flouri is an AI-powered terminal environment enhancement tool designed to provide an intelligent assistant directly within the terminal. This document describes the system architecture and design.

## Overview

Flouri leverages Large Language Models (LLMs) to understand user requests, execute bash commands, and manage a secure environment through allowlists and blacklists. The architecture is modular, allowing for easy extension and integration with various LLM providers via LiteLLM.

## Core Components

### 1. Configuration (`flouri.config`)

**Purpose**: Manages application settings, environment variables, and persistent configuration.

**Key Features**:
- Loads settings from `.env` files
- Manages `config/commands.json` for dynamic command restrictions
- Provides settings access throughout the application

**Key Classes**:
- `Settings`: Pydantic-based settings management
- `ConfigManager`: Configuration file management

### 2. Agent (`flouri.agent`)

**Purpose**: Defines the AI agent's behavior, system instructions, and interaction logic with the LLM.

**Key Features**:
- Constructs system prompts with dynamic allowlist/blacklist rules
- Uses LiteLLM to communicate with various LLM providers
- Integrates with Google ADK for agent orchestration

**Key Functions**:
- `get_agent()`: Creates and configures the agent
- `build_agent_instruction()`: Generates system instructions

### 3. Tools (`flouri.tools`)

**Purpose**: Provides custom Python functions that the AI agent can call to interact with the bash environment.

**Organization**: Tools are organized into modular subdirectories by context:

- **`tools/bash/`**: Bash execution tools
  - `execute_bash`: Execute shell commands
  - `set_cwd`: Change working directory
  - `get_user`: Get current user

- **`tools/config/`**: Configuration management
  - `add_to_allowlist`: Add command to allowlist
  - `add_to_blacklist`: Add command to blacklist
  - `list_allowlist`: List allowed commands
  - `list_blacklist`: List blacklisted commands
  - `is_in_allowlist`: Check if command is allowed
  - `is_in_blacklist`: Check if command is blacklisted

- **`tools/history/`**: History management
  - `read_bash_history`: Read bash command history
  - `read_conversation_history`: Read conversation history
  - `get_tool_call_stats`: Get tool usage statistics

- **`tools/system/`**: System information
  - `get_current_datetime`: Get current date/time

- **`tools/ros2/`**: ROS2-specific tools (optional)
  - Various ROS2 command wrappers

- **`tools/tool_manager/`**: Tool management
  - `enable_tool`: Enable a tool
  - `disable_tool`: Disable a tool
  - `list_enabled_tools`: List enabled tools

**Security**: All tools incorporate pre-execution validation against the allowlist/blacklist.

### 4. Runner (`flouri.runner`)

**Purpose**: Orchestrates the interaction between the user, the AI agent, and the tools.

**Key Features**:
- Handles sending user prompts to the agent
- Processes streaming responses
- Logs conversations and tool calls
- Acts as the bridge between UI and core agent logic

**Key Functions**:
- `run_agent()`: Run agent asynchronously
- `run_agent_sync()`: Run agent synchronously
- `run_agent_live_sync()`: Run agent with streaming output

### 5. UI (`flouri.ui`)

**Purpose**: Implements the Terminal User Interface (TUI) and command-line interface (CLI).

**Components**:

- **TUI** (`tui.py`):
  - Interactive terminal environment
  - Built with `prompt-toolkit` for rich interactions
  - Features:
    - Command completion
    - Command history
    - Syntax highlighting
    - AI assistance via `?` prefix or `Ctrl+A`

- **CLI** (`cli.py`):
  - Non-interactive command-line interface
  - Supports one-off agent runs
  - Streaming output support

### 6. Completions (`flouri.completions`)

**Purpose**: Provides bash-style command completion system with support for custom completion scripts.

**Features**:
- Completion registry and loader
- Discovers completion scripts from project and user directories
- Built-in completions (e.g., `cd` with nested directory support, `git` command completions)
- Extensible custom completions

**Locations**:
- `completions/` (project directory)
- `~/.config/flouri/completions/` (user directory)

### 7. Plugins (`flouri.plugins`)

**Purpose**: Extend functionality with custom plugins.

**Types**:
- **Command Handlers**: Intercept and handle commands before standard execution
- **Command Enhancers**: Enhance command output after execution
- **Completion Plugins**: Provide enhanced tab completion

See [Plugins Documentation]({{ site.baseurl }}/plugins/) for details.

### 8. Logging (`flouri.logging`)

**Purpose**: Manages structured logging for sessions, conversations, and tool executions.

**Features**:
- Creates timestamped log files in `~/.config/flouri/logs/`
- Records detailed events for debugging and auditing
- Session-based logging

## Data Flow

### 1. Initialization

```
Application Start
  ↓
Load Configuration (flouri.config)
  ↓
Initialize Logging (flouri.logging)
  ↓
Load Completions (flouri.completions)
  ↓
Register Plugins (flouri.plugins)
  ↓
Ready for User Input
```

### 2. User Input Processing

**Terminal Mode**:
```
User Types Command
  ↓
Check Plugins (Command Handlers)
  ↓
If Handled: Execute Plugin
  ↓
If Not Handled: Execute via Subprocess
  ↓
Enhance Output (Command Enhancers)
  ↓
Display Result
```

**AI Request** (`?` prefix or `Ctrl+A`):
```
User Types "? question"
  ↓
Send to Runner (flouri.runner)
  ↓
Send to Agent (flouri.agent)
  ↓
LLM Processing (via LiteLLM)
  ↓
Tool Calls (if needed)
  ↓
Return Response
  ↓
Display Result
```

### 3. Tool Execution Flow

```
Agent Decides to Call Tool
  ↓
Validate Command (Allowlist/Blacklist Check)
  ↓
If Not Allowed: Request Confirmation
  ↓
If Confirmed: Execute Tool
  ↓
Log Execution
  ↓
Return Result to Agent
  ↓
Agent Processes Result
  ↓
Return Final Response
```

## Security Model

### Allowlist/Blacklist System

- **Blacklist**: Commands explicitly forbidden (hard-blocked)
- **Allowlist**: Commands explicitly permitted (if active, only these can execute)
- **Pre-execution Validation**: All commands validated before execution
- **User Confirmation**: Required for commands not in allowlist
- **Automated Management**: Agent can add safe commands to allowlist

### Validation Flow

```
Command Requested
  ↓
Check Blacklist → If Found: BLOCK
  ↓
Check Allowlist → If Active and Not Found: REQUEST CONFIRMATION
  ↓
If Confirmed: EXECUTE
  ↓
Log Execution
```

## Extensibility

### Adding New Tools

1. Create tool function in appropriate subdirectory
2. Create `Tool` class or use `FunctionToolWrapper`
3. Register in skill module
4. Export in `tools/__init__.py`

### Adding New Plugins

1. Create plugin class inheriting from `Plugin` or `CommandEnhancer`
2. Implement required methods
3. Register in `flouri/ui/tui.py`
4. Export in `flouri/plugins/__init__.py`

### Adding New Completions

1. Create completion script following the interface
2. Place in `completions/` or `~/.config/flouri/completions/`
3. System will auto-discover and load

## Technology Stack

- **Python 3.10+**: Core language
- **LiteLLM**: Multi-provider LLM integration
- **Google ADK**: Agent framework and orchestration
- **prompt-toolkit**: Rich terminal UI
- **Pydantic**: Settings and configuration management
- **Click**: CLI framework
- **Rich**: Terminal formatting and output

## Design Principles

1. **Modularity**: Clear separation of concerns
2. **Extensibility**: Easy to add new tools, plugins, and completions
3. **Security**: Allowlist/blacklist system with validation
4. **User Experience**: Rich TUI with completion and history
5. **Flexibility**: Support for multiple LLM providers
6. **Observability**: Comprehensive logging

## Future Enhancements

- System-level command filtering
- Sandboxed execution environment
- Enhanced plugin system
- More built-in tools
- Performance optimizations
