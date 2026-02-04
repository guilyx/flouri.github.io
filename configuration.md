---
layout: default
title: Configuration
nav_order: 3
description: "Configuration options and settings for Flouri.sh"
permalink: /configuration/
---

# Configuration

Flouri can be configured through environment variables, configuration files, and command-line options.

## Environment Variables

### Required

- **`API_KEY`**: Your LLM API key from your chosen provider
- **`MODEL`**: The model to use (e.g., `gpt-4o-mini`, `anthropic/claude-3-sonnet`)

### Optional

- **`API_BASE`**: Custom API base URL (for custom endpoints or local models)
- **`DEFAULT_ALLOWLIST`**: Comma-separated list of allowed commands
- **`DEFAULT_BLACKLIST`**: Comma-separated list of blacklisted commands
- **`APP_NAME`**: Application name (default: `flouri`)
- **`USER_ID`**: User identifier (default: `user`)
- **`SESSION_ID`**: Session identifier (default: `session`)

### Example `.env` File

```bash
# LLM API Key
API_KEY=your-api-key-here

# LLM Model
MODEL=gpt-4o-mini

# Optional: API Base URL
# API_BASE=https://api.openai.com/v1

# Optional: Default Command Lists
# DEFAULT_ALLOWLIST=
# DEFAULT_BLACKLIST=rm,dd,format,mkfs,chmod 777
```

## Configuration Files

### Global Configuration

Flouri stores configuration in `~/.config/flouri/`:

```
~/.config/flouri/
├── config.json          # Command allowlist/blacklist
├── history/             # Command history
└── logs/                # Session logs
```

### Project Configuration

You can also place a `config/` directory in your project root:

```
your-project/
├── config/
│   └── config.json      # Project-specific allowlist/blacklist
└── .env                 # Project-specific environment variables
```

### Config File Format

The `config.json` file manages command allowlists and blacklists:

```json
{
  "allowlist": ["ls", "cd", "git", "find", "grep"],
  "blacklist": ["rm", "dd", "format", "mkfs", "chmod 777"]
}
```

## Command-Line Options

### TUI Mode

```bash
flouri
# or
flouri tui
```

No additional options for TUI mode.

### CLI Agent Mode

```bash
flouri agent "your prompt" [OPTIONS]
```

**Options:**

- **`--allowlist`, `-a`**: Comma-separated list of allowed commands
  ```bash
  flouri agent --allowlist "ls,cd,git" "Check git status"
  ```

- **`--blacklist`, `-b`**: Comma-separated list of blacklisted commands
  ```bash
  flouri agent --blacklist "rm,dd" "Help me organize files"
  ```

- **`--stream`, `-s`**: Enable live streaming output
  ```bash
  flouri agent --stream "Explain Docker networking"
  ```

## Allowlist and Blacklist

### How They Work

- **Allowlist**: If set, only commands in the allowlist can be executed
- **Blacklist**: Commands in the blacklist are always blocked
- **Priority**: Blacklist takes precedence over allowlist

### Managing Lists

The agent can manage allowlists and blacklists automatically through tools:

- Commands not in the allowlist will prompt for confirmation
- The agent can add safe commands to the allowlist automatically
- Dangerous commands are automatically added to the blacklist

### Best Practices

1. **Start with a restrictive allowlist**:
   ```json
   {
     "allowlist": ["ls", "cd", "pwd", "cat", "grep"]
   }
   ```

2. **Always blacklist dangerous commands**:
   ```json
   {
     "blacklist": ["rm -rf", "dd", "format", "mkfs", "chmod 777", "sudo rm"]
   }
   ```

3. **Use project-specific configs** for different environments

4. **Review and update regularly** based on your workflow

## Configuration Precedence

Configuration is loaded in this order (later overrides earlier):

1. Default values
2. Global config file (`~/.config/flouri/config.json`)
3. Project config file (`config/config.json`)
4. Environment variables
5. Command-line options (highest priority)

## Advanced Configuration

### Custom Completion Scripts

Place completion scripts in:
- `completions/` (project directory)
- `~/.config/flouri/completions/` (user directory)

See [Plugins]({{ site.baseurl }}/plugins/) for more information.

### Logging Configuration

Logs are stored in `~/.config/flouri/logs/` with timestamps. Each session creates a new log file.

### Session Management

Sessions are tracked by `SESSION_ID`. You can customize this for different contexts:

```bash
export SESSION_ID=work-session
flouri
```

## Examples

### Development Environment

```bash
# .env
API_KEY=dev-key
MODEL=gpt-4o-mini
DEFAULT_ALLOWLIST=ls,cd,git,npm,python,pip
DEFAULT_BLACKLIST=rm,dd,format
```

### Production Environment

```bash
# .env
API_KEY=prod-key
MODEL=anthropic/claude-3-opus
DEFAULT_ALLOWLIST=ls,cd,git,deploy
DEFAULT_BLACKLIST=rm,dd,format,mkfs,chmod,chown,sudo
```

### Project-Specific Config

```json
// config/config.json
{
  "allowlist": ["ls", "cd", "git", "npm", "node"],
  "blacklist": ["rm", "npm install", "npm uninstall"]
}
```

## Troubleshooting

### Config Not Loading

- Check file permissions: `ls -la ~/.config/flouri/`
- Verify JSON syntax: `python3 -m json.tool ~/.config/flouri/config.json`
- Check environment variables: `env | grep FLOURI`

### Commands Being Blocked

- Review your allowlist/blacklist configuration
- Check command-line overrides
- Verify the command format matches exactly

### API Issues

- Verify `API_KEY` is set: `echo $API_KEY`
- Check model name format matches provider
- Review logs in `~/.config/flouri/logs/`
