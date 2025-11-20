# Claude Windows Shell

Windows shell utilities and PowerShell integration for Claude Code - eliminate bash errors with proper path handling, output redirection, and Windows-specific command patterns.

## Overview

This marketplace provides the `windows-shell` plugin that helps Claude Code work seamlessly on Windows by handling common pitfalls with paths, shell commands, and cross-platform compatibility.

## Problem Solved

When running Claude Code on Windows, you may encounter errors like:
- `FINDSTR: Cannot open [dirname]` - Piping issues after directory changes
- `cd: C:KolyaRepositories... No such file or directory` - Unquoted backslash paths
- `The term 'C:\...' is not recognized` - PowerShell context loss
- `/dev/null: No such file or directory` - Unix-style device names

The `windows-shell` skill proactively prevents these errors by teaching Claude the correct Windows command patterns.

## Installation

### 1. Add the Marketplace

```bash
/plugin marketplace add https://github.com/nicoforclaude/claude-windows-shell
```

Or for local testing:

```bash
/plugin marketplace add C:\KolyaRepositories\nicoforclaude\claude-windows-shell
```

### 2. Install the Plugin

```bash
/plugin install windows-shell@claude-windows-shell
```

### 3. Verify Installation

```bash
/plugin list
```

You should see `windows-shell` in your installed plugins list.

## Features

### `windows-shell` Plugin

Provides a proactive skill that activates when Claude Code:
- Executes shell commands (git, npm, docker, etc.)
- Scans repositories
- Performs file operations
- Runs slash commands like `/commit`, `/startup`

**Key capabilities:**
- ✅ Proper Windows path quoting (`"C:\Path\To\Dir"`)
- ✅ Output redirection to `nul` instead of `/dev/null`
- ✅ Avoiding problematic pipe patterns after `cd`
- ✅ PowerShell vs Bash decision guidance
- ✅ Cross-platform path format recommendations

## Usage Examples

Once installed, the skill activates automatically. Claude will:

### Quote Windows Paths Correctly

```bash
# ✅ Correct - paths are quoted
cd "C:\KolyaRepositories\project" && git status

# ❌ Wrong - backslashes stripped
cd C:\KolyaRepositories\project && git status
```

### Use Windows Output Redirection

```bash
# ✅ Correct - Windows nul device
npm install >nul 2>&1

# ❌ Wrong - Unix /dev/null
npm install >/dev/null 2>&1
```

### Avoid Problematic Pipe Patterns

```bash
# ✅ Correct - direct command
cd "C:\KolyaRepositories\repo" && git status --short

# ❌ Wrong - findstr after cd fails
cd "C:\KolyaRepositories\repo" && git status --short | findstr /R "."
```

### Choose PowerShell vs Bash Appropriately

The skill guides Claude to:
- Use **Bash** for: git, npm, standard Unix tools
- Use **PowerShell** for: Windows-specific tasks, .NET objects, registry operations

## How It Works

The `windows-shell` skill is **proactive** - it activates automatically when Claude Code detects it's running on Windows and about to execute shell commands. No manual intervention needed!

The skill teaches Claude to:
1. Detect Windows path patterns
2. Quote paths with backslashes or spaces
3. Use `nul` for output redirection
4. Avoid pipe patterns that break on Windows
5. Prefer specialized tools (Read, Glob, Grep) over bash for file operations

## Who Should Use This

- **Windows users** running Claude Code on Windows 10/11
- **Git Bash users** working with Windows paths
- **WSL users** who still need to interact with Windows filesystem
- **Anyone** experiencing shell command errors on Windows

## Compatibility

- ✅ Windows 10/11
- ✅ Git Bash
- ✅ WSL (Windows Subsystem for Linux)
- ✅ PowerShell 5.1+
- ✅ Claude Code 1.0+

## Related Plugins

Consider pairing with:
- **`claude-smart-git`** - Enhanced git workflows with proper Windows path handling
- **`claude-nico-dev`** - Development workflow improvements

## Contributing

Issues and suggestions welcome! Please report problems at:
https://github.com/nicoforclaude/claude-windows-shell/issues

## License

MIT License - See [LICENSE](./LICENSE) file for details.

## Author

**NicoForClaude**

Part of the NicoForClaude plugin collection for Claude Code.

---

**Stop fighting with Windows paths. Let Claude handle them correctly from the start.**
