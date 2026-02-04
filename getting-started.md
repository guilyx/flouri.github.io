---
layout: default
title: Getting Started
nav_order: 2
has_children: true
description: "Installation and basic usage guide for Flouri.sh"
permalink: /getting-started/
---

# Getting Started

This guide will help you install and start using Flouri.sh, an AI-powered terminal environment.

## Installation

### From PyPI (Recommended)

```bash
pip install flouri
```

### From Source

```bash
git clone https://github.com/guilyx/flouri.git
cd flouri
pip install -e ".[dev]"
```

### Requirements

- Python 3.10 or higher
- An API key from one of the supported LLM providers:
  - OpenAI: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
  - Anthropic: [console.anthropic.com](https://console.anthropic.com/)
  - Google: [aistudio.google.com/apikey](https://aistudio.google.com/apikey)

## Configuration

### Environment Variables

Create a `.env` file in your project directory or set environment variables:

```bash
# Required: Your LLM API key
API_KEY=your-api-key-here

# Required: Model to use
MODEL=gpt-4o-mini   # or anthropic/claude-3-sonnet, gemini/gemini-2.0-flash, etc.

# Optional: API Base URL (for custom endpoints)
# API_BASE=https://api.openai.com/v1

# Optional: Default command lists
# DEFAULT_ALLOWLIST=
# DEFAULT_BLACKLIST=rm,dd,format,mkfs,chmod 777
```

### Supported Models

Flouri supports any model available through [LiteLLM](https://docs.litellm.ai/docs/providers). Common examples:

- `gpt-4o-mini` (OpenAI)
- `gpt-4o` (OpenAI)
- `anthropic/claude-3-sonnet` (Anthropic)
- `anthropic/claude-3-opus` (Anthropic)
- `gemini/gemini-2.0-flash` (Google)
- `gemini/gemini-pro` (Google)

See the [LiteLLM documentation](https://docs.litellm.ai/docs/providers) for the full list.

## Quick Start

### Interactive TUI Mode

Launch the interactive terminal:

```bash
flouri
# or
flouri tui
```

In the TUI:
- Type commands normally (e.g., `ls`, `cd`, `git status`)
- Prefix with `?` to ask the AI: `? How do I check disk usage?`
- Use `Ctrl+A` as an alternative to `?`
- Use `Tab` for command completion
- Use `↑/↓` for command history
- Use `Ctrl+R` to search history
- Use `Ctrl+D` to exit

### CLI Mode

Run one-off commands:

```bash
# Basic usage
flouri agent "List files and show git status"

# With streaming output
flouri agent --stream "Explain how Docker networking works"

# With custom allowlist
flouri agent --allowlist "ls,cd,git" "Check git status"

# With blacklist
flouri agent --blacklist "rm,dd" "Help me organize files"
```

## First Steps

1. **Set up your API key**:
   ```bash
   export API_KEY=your-key
   export MODEL=gpt-4o-mini
   ```

2. **Launch Flouri**:
   ```bash
   flouri
   ```

3. **Try a command**:
   ```bash
   $ ls -la
   ```

4. **Ask for help**:
   ```bash
   $ ? What files are taking up the most space?
   ```

5. **Explore features**:
   - Try tab completion: `cd ~/pro<Tab>`
   - Use history: `↑` to see previous commands
   - Ask complex questions: `? How do I set up a Python virtual environment?`

## Next Steps

- Learn about [Configuration]({{ site.baseurl }}/configuration/) options
- Explore the [API Reference]({{ site.baseurl }}/api/) for programmatic usage
- Read about [Security]({{ site.baseurl }}/security/) best practices
- Check out [Plugins]({{ site.baseurl }}/plugins/) to extend functionality

## Troubleshooting

### API Key Issues

If you see authentication errors:
- Verify your API key is set: `echo $API_KEY`
- Check that the key is valid for your chosen provider
- Ensure the model name matches your provider (e.g., `anthropic/claude-3-sonnet` for Anthropic)

### Command Execution Issues

If commands aren't executing:
- Check your allowlist/blacklist configuration
- Review the [Security]({{ site.baseurl }}/security/) documentation
- Check logs in `~/.config/flouri/logs/`

### Installation Issues

If installation fails:
- Ensure Python 3.10+ is installed: `python3 --version`
- Try upgrading pip: `pip install --upgrade pip`
- Use a virtual environment: `python3 -m venv venv && source venv/bin/activate`

## Getting Help

- Check the [GitHub Issues](https://github.com/guilyx/flouri/issues)
- Review the [Architecture]({{ site.baseurl }}/architecture/) documentation
- Open a [Discussion](https://github.com/guilyx/flouri/discussions)
