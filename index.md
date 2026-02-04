---
layout: default
title: Home
nav_order: 1
description: "AI-powered terminal environment - Run normal bash commands and get help from an LLM"
permalink: /
---

# Flouri.sh

**AI-powered terminal environment** - Run normal bash commands and get help from an LLM with `?` or `flouri agent "..."`. Commands use an allowlist/blacklist; supports OpenAI, Anthropic, Google, and others via [LiteLLM](https://litellm.ai/).

[![PyPI version](https://img.shields.io/pypi/v/flouri.svg)](https://pypi.org/project/flouri/)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/flouri)](https://pypi.org/project/flouri/)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/flouri.svg)](https://pypi.org/project/flouri/)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://github.com/guilyx/flouri/blob/main/LICENSE)
[![CI](https://github.com/guilyx/flouri/actions/workflows/ci.yml/badge.svg)](https://github.com/guilyx/flouri/actions/workflows/ci.yml)

## Quick Start

### Installation

```bash
pip install flouri
```

### Configuration

Set your API key in `.env` or environment variables:

```bash
export API_KEY=your-key
export MODEL=gpt-4o-mini   # or anthropic/claude-3-sonnet, gemini/gemini-2.0-flash, etc.
```

### Usage

**Interactive TUI:**
```bash
flouri
# or: flouri tui
```

Then run shell commands as usual; use `? your question` or `Ctrl+A` for AI help.

**CLI Mode:**
```bash
flouri agent "List files and show git status"
flouri agent --stream "Explain how Docker networking works"
```

## Features

- 🤖 **AI-Powered Assistance**: Get help from LLMs directly in your terminal
- 🔒 **Security First**: Allowlist/blacklist system for command execution
- 🎨 **Rich TUI**: Interactive terminal with tab completion, history, and syntax highlighting
- 🔌 **Extensible**: Plugin system for custom commands and behaviors
- 🌐 **Multi-Provider**: Support for OpenAI, Anthropic, Google, and more via LiteLLM
- 📝 **Smart Completions**: Enhanced command completion with nested directory support
- 🛠️ **Tool System**: Modular tool architecture organized by skills

## Documentation

- **[Getting Started]({{ site.baseurl }}/getting-started/)** - Installation and basic usage
- **[API Reference]({{ site.baseurl }}/api/)** - Complete API documentation
- **[Architecture]({{ site.baseurl }}/architecture/)** - System design and components
- **[Plugins]({{ site.baseurl }}/plugins/)** - Extending Flouri with plugins
- **[Configuration]({{ site.baseurl }}/configuration/)** - Settings and customization
- **[Security]({{ site.baseurl }}/security/)** - Security best practices
- **[Developer Guide]({{ site.baseurl }}/developer/)** - Contributing and development

## Example Workflows

### Basic Terminal Operations
```bash
# In TUI, just type commands normally
$ ls -la
$ cd ~/projects
$ git status

# Ask for AI help
$ ? How do I check disk usage?
$ ? What's the difference between git merge and rebase?
```

### CLI Mode
```bash
# One-off questions
flouri agent "List all Python files in the current directory"

# With streaming output
flouri agent --stream "Explain Docker networking"

# With custom allowlist
flouri agent --allowlist "ls,cd,git" "Check git status and show recent commits"
```

## Links

- **PyPI**: [pypi.org/project/flouri](https://pypi.org/project/flouri/)
- **GitHub**: [github.com/guilyx/flouri](https://github.com/guilyx/flouri)
- **Issues**: [github.com/guilyx/flouri/issues](https://github.com/guilyx/flouri/issues)
- **LiteLLM**: [litellm.ai](https://litellm.ai/)

## License

Apache 2.0 - See [LICENSE](https://github.com/guilyx/flouri/blob/main/LICENSE)
