# windows-shell Plugin

Windows shell compatibility layer for Claude Code - ensures proper path handling, output redirection, and command patterns on Windows.

## Overview

The `windows-shell` plugin provides a proactive skill that prevents common Windows shell errors by teaching Claude Code the correct patterns for:
- Windows path quoting
- Output redirection (`nul` vs `/dev/null`)
- Avoiding problematic pipe patterns
- PowerShell vs Bash selection

## Installation

This plugin is part of the `claude-windows-shell` marketplace.

```bash
/plugin marketplace add https://github.com/nicoforclaude/claude-windows-shell
/plugin install windows-shell@claude-windows-shell
```

## Components

### Skills

#### `windows-shell`
**Activation:** Proactive - automatically when running on Windows and executing shell commands

**Purpose:** Prevents bash errors by ensuring correct Windows command patterns

**Key behaviors:**
- Quotes paths with backslashes: `"C:\KolyaRepositories\project"`
- Uses `nul` for output redirection instead of `/dev/null`
- Avoids `findstr` and `Where-Object` pipes after `cd`
- Recommends PowerShell for Windows-specific tasks
- Recommends Bash for Unix-style operations (git, npm, etc.)

**When it activates:**
- Executing shell commands (git, npm, docker, etc.)
- Scanning repositories
- Reading files via bash
- Running slash commands like `/commit`, `/startup`

**Examples:**

```bash
# ✅ Skill ensures this pattern
cd "C:\KolyaRepositories\repo" && git status

# ❌ Skill prevents this pattern
cd C:\KolyaRepositories\repo && git status --short | findstr /R "."
```

## Usage

Once installed, the skill works automatically. No manual activation required!

### What Changes

**Before (common errors):**
```bash
# Unquoted paths
cd C:\KolyaRepositories\project
# Error: cd: C:KolyaRepositoriesproject: No such file or directory

# Unix output redirection
command >/dev/null 2>&1
# Error: /dev/null: No such file or directory

# Pipes after cd
cd "C:\repo" && git status | findstr /R "."
# Error: FINDSTR: Cannot open repo
```

**After (with windows-shell):**
```bash
# Quoted paths
cd "C:\KolyaRepositories\project"
# ✅ Works

# Windows output redirection
command >nul 2>&1
# ✅ Works

# Direct commands
cd "C:\repo" && git status
# ✅ Works
```

## Technical Details

### Common Pitfalls Prevented

1. **Unquoted backslash paths** - Bash strips backslashes
2. **Findstr after cd** - Interprets directory name as file argument
3. **PowerShell piping after bash cd** - Context loss
4. **Unix `/dev/null`** - Doesn't exist on Windows
5. **Unnecessary bash pipes** - Should use specialized tools (Read, Grep, Glob)

### Platform Detection

The skill activates when:
- Running on Windows OS (detected by Claude Code)
- Executing Bash commands via the Bash tool
- Working with Windows-style paths (`C:\...`)

### Best Practices Enforced

- **Paths:** Always quote paths with backslashes or spaces
- **Output:** Use `nul` for Windows, not `/dev/null`
- **Tools:** Prefer Read/Grep/Glob tools over bash for file operations
- **Multi-repo:** Use parallel tool calls for independent operations
- **Skills:** Use forward slashes in configuration files

## Compatibility

- Windows 10/11
- Git Bash
- WSL (Windows Subsystem for Linux)
- PowerShell 5.1+

## Troubleshooting

### Skill Not Activating?

Check if the skill is loaded:
```bash
/skills
```

You should see `windows-shell` in the list.

### Still Getting Path Errors?

The skill is proactive but may not catch 100% of cases. If you see:
```
cd: C:KolyaRepositories...: No such file or directory
```

Manually quote the path or report an issue.

### PowerShell Context Loss?

If you see:
```
The term 'C:\...' is not recognized
```

This happens when mixing bash `cd` with PowerShell piping. The skill prevents this by avoiding PowerShell pipes after bash directory changes.

## Examples

### Git Operations
```bash
# Multi-repo status check
cd "C:\KolyaRepositories\repo1" && git status --short
cd "C:\KolyaRepositories\repo2" && git status --short

# Or using git -C
git -C "C:\KolyaRepositories\repo1" status --short
```

### Build Commands
```bash
# Suppress output Windows-style
npm install >nul 2>&1
npm run build >nul 2>&1
```

### File Operations
```bash
# Prefer specialized tools
# Instead of: cat file.txt
Read tool: C:\KolyaRepositories\project\file.txt

# Instead of: find . -name "*.js"
Glob tool: **/*.js
```

## Contributing

Found a Windows path pattern that isn't handled? Please report it!

https://github.com/nicoforclaude/claude-windows-shell/issues

## License

MIT License - See root LICENSE file for details.

---

**Part of the claude-windows-shell marketplace by NicoForClaude**
