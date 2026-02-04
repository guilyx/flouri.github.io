---
layout: default
title: Security
nav_order: 4
description: "Security best practices and considerations for Flouri.sh"
permalink: /security/
---

# Security

Flouri can execute terminal commands on your system. This section covers security best practices and considerations.

## Security Model

### Command Execution

Flouri uses an **allowlist/blacklist** system to control which commands can be executed:

- **Allowlist**: Only commands in the allowlist can be executed (if allowlist is active)
- **Blacklist**: Commands in the blacklist are always blocked
- **Pre-execution Validation**: All commands are validated before execution
- **User Confirmation**: Commands not in the allowlist require explicit confirmation

### Permissions

- Flouri runs with the same permissions as your user account
- It does not elevate privileges automatically
- Never run Flouri as root unless absolutely necessary

## Best Practices

### 1. Use Allowlists

Always use allowlists in production or sensitive environments:

```bash
flouri agent --allowlist "ls,cd,git,find" "Safe task"
```

Or configure in `config.json`:

```json
{
  "allowlist": ["ls", "cd", "git", "find", "grep", "cat"]
}
```

### 2. Use Blacklists

Prevent dangerous commands:

```bash
flouri agent --blacklist "rm,dd,format,mkfs" "Task"
```

Or configure in `config.json`:

```json
{
  "blacklist": ["rm", "rm -rf", "dd", "format", "mkfs", "chmod 777", "sudo rm"]
}
```

### 3. Protect API Keys

- **Never commit `.env` files** to version control
- Use environment variables or secure secret management
- Rotate API keys regularly
- Use different keys for development and production

### 4. Review Before Execution

The agent will explain what it plans to do. Always review:
- What commands will be executed
- What files will be modified
- What network operations will occur

### 5. Use Project-Specific Configs

Different projects may need different security settings:

```json
// config/config.json (project-specific)
{
  "allowlist": ["ls", "cd", "git", "npm"],
  "blacklist": ["rm", "npm install", "npm uninstall"]
}
```

## Dangerous Commands

The following commands are particularly dangerous and should be blacklisted by default:

- **`rm -rf`**: Recursive deletion (can delete entire directories)
- **`dd`**: Disk operations (can overwrite disks)
- **`format` / `mkfs`**: Filesystem operations (can destroy data)
- **`chmod 777`**: Permission changes (can expose sensitive files)
- **`sudo` commands**: Elevated privileges (can modify system)
- **Network commands**: Can expose your system or send data externally
- **`curl | sh`**: Piping to shell (can execute arbitrary code)

### Recommended Default Blacklist

```json
{
  "blacklist": [
    "rm -rf",
    "rm -r",
    "dd",
    "format",
    "mkfs",
    "chmod 777",
    "chmod 666",
    "sudo rm",
    "sudo dd",
    "curl | sh",
    "wget | sh"
  ]
}
```

## Security Features

### Built-in Protections

1. **Pre-execution Validation**: All commands are checked before execution
2. **Automatic Blacklisting**: Dangerous commands are automatically added to blacklist
3. **Confirmation Flow**: Commands not in allowlist require user confirmation
4. **Logging**: All commands and tool calls are logged for auditing

### Limitations

- No sandboxing by default - commands run in your actual environment
- Allowlist/blacklist is enforced through agent instructions, not system-level restrictions
- The agent uses Python code execution which runs shell commands directly

## Reporting Vulnerabilities

If you discover a security vulnerability, please **DO NOT** open a public issue.

Instead:

1. Email the maintainers directly (if contact info is available)
2. Or create a private security advisory on GitHub
3. Provide:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if you have one)

We will respond within 48 hours and work with you to address the issue.

## Security Checklist

Before using Flouri in production:

- [ ] API keys are stored securely (not in version control)
- [ ] Allowlist is configured with minimal required commands
- [ ] Blacklist includes all dangerous commands
- [ ] Review logs regularly (`~/.config/flouri/logs/`)
- [ ] Use project-specific configs for different environments
- [ ] Never run as root
- [ ] Keep Flouri and dependencies updated
- [ ] Review agent actions before confirming execution

## Future Security Enhancements

Planned security improvements:

- [ ] System-level command filtering
- [ ] Sandboxed execution environment
- [ ] Enhanced command confirmation prompts
- [ ] Audit logging of executed commands
- [ ] Rate limiting for API calls
- [ ] Command timeout mechanisms

## Additional Resources

- [Architecture Documentation]({{ site.baseurl }}/architecture/) - Understanding the system
- [Configuration Guide]({{ site.baseurl }}/configuration/) - Setting up security configs
- [GitHub Security Policy](https://github.com/guilyx/flouri/security/policy)
