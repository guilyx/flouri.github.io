---
layout: default
title: Plugins
nav_order: 7
description: "Plugin system documentation for extending Flouri.sh"
permalink: /plugins/
---

# Plugins

Flouri has a powerful plugin system that allows you to extend the terminal environment with custom commands, aliases, and behaviors.

## Overview

Flouri supports three types of plugins:

1. **Command Handlers**: Plugins that completely handle command execution (e.g., zsh bindings)
2. **Command Enhancers**: Plugins that enhance/enrich command output without replacing execution (e.g., colored ls output)
3. **Completion Plugins**: Plugins that provide enhanced tab completion for commands

## Command Handlers

Command handlers are checked **before** standard command execution. If a plugin handles a command, it executes and returns a result. If no plugin handles it, the command falls through to standard bash execution.

### Creating a Command Handler

Create a new file in `flouri/plugins/` (e.g., `my_plugin.py`):

```python
"""My custom plugin for Flouri."""

from pathlib import Path
from typing import Any

from flouri.plugins import Plugin


class MyPlugin(Plugin):
    """Description of what your plugin does."""

    def name(self) -> str:
        """Return the plugin name."""
        return "my_plugin"

    def should_handle(self, command: str) -> bool:
        """Check if this plugin should handle the command."""
        # Return True if your plugin should handle this command
        return command.strip().startswith("mycommand")

    async def execute(self, command: str, cwd: str) -> dict[str, Any]:
        """Execute the command."""
        try:
            # Your plugin logic here
            result = "Plugin output"

            return {
                "handled": True,      # Must be True if plugin handled the command
                "output": result,     # Standard output (optional)
                "error": "",          # Error message if failed (optional)
                "exit_code": 0,       # Exit code (0 = success, non-zero = error)
                "new_cwd": cwd,       # New working directory if changed (optional)
            }
        except Exception as e:
            return {
                "handled": True,
                "output": "",
                "error": str(e),
                "exit_code": 1,
            }
```

### Registering a Command Handler

Add your plugin to `flouri/ui/tui.py`:

```python
from flouri.plugins import PluginManager, ZshBindingsPlugin, MyPlugin

# In TerminalApp.__init__:
self.plugin_manager = PluginManager()
self.plugin_manager.register(ZshBindingsPlugin())
self.plugin_manager.register(MyPlugin())  # Add your plugin
```

### Example: Zsh Bindings Plugin

The `ZshBindingsPlugin` demonstrates how to create a plugin:

```python
class ZshBindingsPlugin(Plugin):
    """Plugin that provides zsh-like command bindings."""

    def name(self) -> str:
        return "zsh_bindings"

    def should_handle(self, command: str) -> bool:
        cmd = command.strip()
        # Handle: cd (alone), cd with 3+ dots
        if cmd == "cd":
            return True
        if cmd.startswith("cd "):
            parts = cmd.split()
            if len(parts) == 2:
                path = parts[1]
                clean_path = path.replace("/", "")
                if clean_path and all(c == "." for c in clean_path) and len(clean_path) >= 3:
                    return True
        return False

    async def execute(self, command: str, cwd: str) -> dict[str, Any]:
        # Implementation handles:
        # - cd (alone) -> home directory
        # - cd ... (3+ dots) -> go back (dots - 1) directories
        ...
```

## Command Enhancers

Command enhancers intercept command output **after** execution and can:
- Add colors and formatting to output
- Provide helpful hints and suggestions
- Enhance readability without changing functionality

### Creating a Command Enhancer

Create a new file in `flouri/plugins/` (e.g., `my_enhancer.py`):

```python
from flouri.plugins.enhancers import CommandEnhancer
from typing import Any

class MyEnhancer(CommandEnhancer):
    """My custom command enhancer."""

    def name(self) -> str:
        return "my_enhancer"

    def should_enhance(self, command: str) -> bool:
        """Check if this enhancer should enhance the command."""
        return command.startswith("mycommand")

    def enhance_output(
        self,
        command: str,
        stdout: str,
        stderr: str,
        exit_code: int,
        cwd: str,
    ) -> dict[str, Any]:
        """Enhance the command output."""
        # Your enhancement logic here
        enhanced_stdout = stdout  # Modify stdout
        hints = []  # Optional hints to display

        return {
            "enhanced": True,  # True if output was modified
            "stdout": enhanced_stdout,
            "stderr": stderr,  # Can also enhance stderr
            "hints": hints,  # List of hint strings to display
        }
```

### Registering an Enhancer

Add your enhancer to `flouri/ui/tui.py`:

```python
from flouri.plugins.enhancers import MyEnhancer

# In TerminalApp.__init__:
self.enhancer_manager.register(MyEnhancer())
```

### Built-in Enhancers

#### LsColorEnhancer

Adds color coding to `ls` output:
- **Blue (bold)**: Directories
- **Cyan**: Symlinks
- **Green**: Executable files
- **Yellow**: Archive files (.zip, .tar, etc.)
- **Magenta**: Image/media files
- **Default**: Regular files

#### CdEnhancementPlugin

Provides helpful hints when `cd` fails:
- Suggests similar directory names when a directory doesn't exist
- Helps with typos and partial matches

## Completion Plugins

Completion plugins integrate with the `prompt-toolkit` completion system to provide enhanced tab completion for specific commands.

### Creating a Completion Plugin

Create a new file in `flouri/plugins/` (e.g., `my_completer.py`):

```python
from prompt_toolkit.completion import Completion, Completer
from prompt_toolkit.document import Document

class MyCompleter(Completer):
    """Enhanced completer for mycommand."""

    def __init__(self, cwd: Path | None = None):
        self.cwd = cwd or Path.cwd()

    def get_completions(self, document, complete_event):
        """Get completions for mycommand."""
        text = document.text_before_cursor
        # Your completion logic here
        yield Completion("completion1", start_position=0)
        yield Completion("completion2", start_position=0)
```

### Built-in Completion Plugins

#### CdCompleter

Enhanced completion for the `cd` command:
- **Nested directory completion**: Supports paths like `cd dev/proj` with smart completion
- **Absolute and relative paths**: Handles both `/path/to/dir` and `path/to/dir`
- **Home directory expansion**: Supports `~` and `~/path/to/dir`
- **Visual depth indicators**: Shows nested directory structure in completion menu

## Plugin Return Values

### Command Handler Return Values

Your plugin's `execute()` method must return a dictionary with these keys:

- **`handled`** (bool, required): `True` if the plugin handled the command, `False` to pass to next handler
- **`output`** (str, optional): Standard output to display
- **`error`** (str, optional): Error message if execution failed
- **`exit_code`** (int, optional): Exit code (0 = success, non-zero = error)
- **`new_cwd`** (str, optional): New working directory if the plugin changed it

### Command Enhancer Return Values

Your enhancer's `enhance_output()` method must return a dictionary with:

- **`enhanced`** (bool): `True` if output was modified, `False` otherwise
- **`stdout`** (str): Enhanced or original stdout
- **`stderr`** (str): Enhanced or original stderr
- **`hints`** (list[str]): Optional list of hint messages to display

## Examples

### Example 1: Simple Alias Plugin

```python
class AliasPlugin(Plugin):
    """Plugin that provides command aliases."""

    def __init__(self):
        self.aliases = {
            "ll": "ls -la",
            "la": "ls -a",
            "gst": "git status",
            "gco": "git checkout",
        }

    def name(self) -> str:
        return "alias"

    def should_handle(self, command: str) -> bool:
        cmd = command.strip().split()[0] if command.strip() else ""
        return cmd in self.aliases

    async def execute(self, command: str, cwd: str) -> dict[str, Any]:
        cmd = command.strip().split()[0]
        alias_cmd = self.aliases[cmd]
        # Execute the aliased command via subprocess
        import subprocess
        result = subprocess.run(
            alias_cmd,
            shell=True,
            capture_output=True,
            text=True,
            cwd=cwd,
        )
        return {
            "handled": True,
            "output": result.stdout,
            "error": result.stderr,
            "exit_code": result.returncode,
        }
```

### Example 2: Directory Navigation Plugin

```python
class QuickNavPlugin(Plugin):
    """Plugin for quick directory navigation."""

    def name(self) -> str:
        return "quick_nav"

    def should_handle(self, command: str) -> bool:
        return command.strip().startswith("goto ")

    async def execute(self, command: str, cwd: str) -> dict[str, Any]:
        parts = command.strip().split()
        if len(parts) != 2:
            return {"handled": False}

        target = parts[1]
        # Your navigation logic here
        target_path = Path.home() / "projects" / target

        if target_path.exists() and target_path.is_dir():
            return {
                "handled": True,
                "output": f"Navigated to {target_path}",
                "exit_code": 0,
                "new_cwd": str(target_path),
            }
        else:
            return {
                "handled": True,
                "output": "",
                "error": f"Directory not found: {target_path}",
                "exit_code": 1,
            }
```

### Example 3: Colorful Output Enhancer

```python
class ColorfulOutputEnhancer(CommandEnhancer):
    """Add colors to specific command outputs."""

    RESET = "\033[0m"
    GREEN = "\033[32m"
    RED = "\033[31m"

    def name(self) -> str:
        return "colorful_output"

    def should_enhance(self, command: str) -> bool:
        return command.startswith("echo ")

    def enhance_output(
        self,
        command: str,
        stdout: str,
        stderr: str,
        exit_code: int,
        cwd: str,
    ) -> dict[str, Any]:
        if exit_code == 0:
            enhanced_stdout = f"{self.GREEN}{stdout}{self.RESET}"
        else:
            enhanced_stderr = f"{self.RED}{stderr}{self.RESET}"
            return {
                "enhanced": True,
                "stdout": stdout,
                "stderr": enhanced_stderr,
                "hints": [],
            }

        return {
            "enhanced": True,
            "stdout": enhanced_stdout,
            "stderr": stderr,
            "hints": [],
        }
```

## Best Practices

1. **Be Specific**: Make `should_handle()` or `should_enhance()` as specific as possible to avoid conflicts
2. **Error Handling**: Always wrap execution in try/except and return proper error responses
3. **Documentation**: Add clear docstrings explaining what your plugin does
4. **Testing**: Test your plugin with various inputs and edge cases
5. **Performance**: Keep plugin execution fast - plugins are checked on every command
6. **Directory Changes**: If your plugin changes directories, return `new_cwd` in the result

## Plugin Ideas

Here are some ideas for plugins you could create:

1. **Alias Plugin**: Create command aliases (e.g., `ll` -> `ls -la`, `gst` -> `git status`)
2. **Directory Bookmarks Plugin**: Save and jump to bookmarked directories
3. **History Search Plugin**: Enhanced history search with fzf-like functionality
4. **Git Shortcuts Plugin**: Short git commands (e.g., `gst` -> `git status`, `gco` -> `git checkout`)
5. **Environment Variable Plugin**: Quick environment variable management
6. **Project Switcher Plugin**: Quickly switch between projects with custom commands

## Contributing Plugins

We welcome plugin contributions! To contribute a plugin:

1. **Create your plugin** following the structure above
2. **Add tests** for your plugin (if applicable)
3. **Update documentation** (this file or create a new section)
4. **Submit a PR** with:
   - Your plugin code
   - Tests (if applicable)
   - Documentation
   - Example usage

### Plugin Contribution Checklist

- [ ] Plugin follows the `Plugin` or `CommandEnhancer` base class interface
- [ ] `should_handle()` or `should_enhance()` is specific and efficient
- [ ] `execute()` or `enhance_output()` handles errors gracefully
- [ ] Plugin is registered in `flouri/ui/tui.py`
- [ ] Plugin is exported in `flouri/plugins/__init__.py`
- [ ] Documentation added/updated
- [ ] Code follows project style (black, ruff)
- [ ] No breaking changes to existing functionality

## Advanced: Plugin Configuration

Plugins can access configuration through the `ConfigManager`:

```python
from flouri.config.config_manager import ConfigManager

class MyPlugin(Plugin):
    def __init__(self):
        self.config = ConfigManager()
        # Access config as needed
```

## Plugin Execution Order

Plugins are executed in the order they are registered. The first plugin that returns `handled: True` wins. Make sure your plugin's `should_handle()` is specific enough to avoid conflicts.

## Questions?

- Check existing plugins in `flouri/plugins/` for examples
- Check existing enhancers in `flouri/plugins/enhancers.py` for enhancement examples
- Check `flouri/plugins/cd_completer.py` for completion plugin examples
- Open a [Discussion](https://github.com/guilyx/flouri/discussions) for questions
- Review [CONTRIBUTING.md](https://github.com/guilyx/flouri/blob/main/CONTRIBUTING.md) for general contribution guidelines

Happy plugin development! 🚀
