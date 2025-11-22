---
name: windows-shell
description: Windows path handling and shell command guidelines for avoiding common pitfalls. Use proactively when running Claude on Windows and executing shell commands (git, file operations, scripts, etc), scanning repos, reading files, or inside slash commands implying such operations (e.g., /commit).
---

# Windows Shell Skill

This skill provides guidelines for handling Windows filesystem paths and commands to avoid common pitfalls when using Claude Code on Windows.

## Core Principles

### Path Formats

**In Skills and Configuration:**
- Always use **forward slashes** (Unix-style): `scripts/helper.py`
- Never use backslashes in Skills: ~~`scripts\helper.py`~~
- This ensures cross-platform compatibility

**In Bash Commands:**
- Always **quote paths with spaces or backslashes**: `cd "C:\KolyaRepositories\project"`
- Without quotes, backslashes are lost: ~~`cd C:\KolyaRepositories\project`~~
- Result: `/usr/bin/bash: line 1: cd: C:KolyaRepositoriesproject: No such file or directory`

### Shell Command Patterns

**Good patterns:**
```bash
# Quoted paths work reliably
cd "C:\KolyaRepositories\repo" && git status

# Multiple commands in sequence
cd "C:\KolyaRepositories\repo" && git rev-parse --abbrev-ref HEAD && git status --short

# Direct git commands without piping
git status --short
```

**Bad patterns:**
```bash
# Unquoted paths fail
cd C:\KolyaRepositories\repo && git status

# Piping to findstr after cd fails
cd "C:\KolyaRepositories\repo" && git status --short | findstr /R "."
# Error: FINDSTR: Cannot open repo

# PowerShell via -Command loses cd context
cd "C:\KolyaRepositories\repo" && powershell -Command "git status --short | Where-Object { $_ }"
# Error: The term 'C:\KolyaRepositories\repo' is not recognized
```

## Common Pitfalls

### 1. Findstr After Directory Change

**Problem:** `findstr` interprets the directory name as a file argument when piped after `cd`.

```bash
# Fails
cd "C:\KolyaRepositories\calc" && git status --short | findstr /R "."
# Error: FINDSTR: Cannot open calc
```

**Solution:** Use direct commands without unnecessary pipes.

```bash
# Works
cd "C:\KolyaRepositories\calc" && git status --short
```

### 2. PowerShell Context Loss

**Problem:** Using `powershell -Command` after `cd` causes the shell context to be misinterpreted.

```bash
# Fails
cd "C:\KolyaRepositories\calc" && powershell -Command "git status --short | Where-Object { $_ }"
# Error: The term 'C:\KolyaRepositories\calc' is not recognized
```

**Solution:** Either use PowerShell directly from the start, or avoid PowerShell piping after bash cd.

```bash
# Works - all in PowerShell
powershell -Command "cd 'C:\KolyaRepositories\calc'; git status --short"

# Works - no PowerShell piping
cd "C:\KolyaRepositories\calc" && git status --short
```

### 3. Unquoted Paths in Bash

**Problem:** Bash strips backslashes from unquoted Windows paths.

```bash
# Fails
cd C:\KolyaRepositories\calc
# Error: cd: C:KolyaRepositoriescalc: No such file or directory
```

**Solution:** Always quote paths containing backslashes.

```bash
# Works
cd "C:\KolyaRepositories\calc"
```

### 4. Output Redirection to Null Device

**Problem:** Using Unix-style `/dev/null` on Windows.

```bash
# Fails on Windows
command > /dev/null 2>&1
```

**Solution:** Use Windows `nul` device.

```bash
# Works on Windows
command >nul 2>&1

# Or in PowerShell
command | Out-Null
```

### 5. PowerShell Special Character Escaping

**Problem A - Negation Operator:**
When passing PowerShell commands via Bash, the `!` character gets escaped to `\!`.

```bash
# Fails - ! becomes \!
powershell -Command "if (!(Test-Path 'file.txt')) { Write-Output 'missing' }"
# Error: \! is not recognized
```

**Solution:** Use `-not` instead of `!` for negation.

```bash
# Works
powershell -Command "if (-not (Test-Path 'file.txt')) { Write-Output 'missing' }"
```

**Problem B - Variable Interpolation:**
PowerShell `$variable` syntax gets stripped when passed through Bash.

```bash
# Fails - $pdf and $sizeKB are empty
powershell -Command "$pdf = Get-Item 'file.pdf'; $sizeKB = [math]::Round($pdf.Length / 1KB, 1); Write-Output $sizeKB"
# Output: (empty or partial)
```

**Solution:** Use `.ps1` script files for complex operations, or pipe output between simple commands.

```bash
# Works - use script file
powershell -ExecutionPolicy Bypass -File "script.ps1" -Path "file.pdf"

# Works - simple piped commands
powershell -Command "Get-Item 'file.pdf' | Select-Object Length"
```

**Summary for inline PowerShell:**
- Use `-not` instead of `!`
- Use script files for complex logic with variables
- Keep inline commands simple (single operation, pipe output)

## Best Practices for Multi-Repo Operations

When iterating over multiple repositories (e.g., for status checks):

**Pattern 1 - Sequential commands with &&:**
```bash
cd "C:\KolyaRepositories\repo1" && git status --short
cd "C:\KolyaRepositories\repo2" && git status --short
```

**Pattern 2 - Self-contained commands:**
```bash
git -C "C:\KolyaRepositories\repo1" status --short
git -C "C:\KolyaRepositories\repo2" status --short
```

**Pattern 3 - Parallel tool calls:**
Use multiple Bash tool calls in a single message for independent operations:
- Bash tool 1: `cd "C:\repo1" && git status`
- Bash tool 2: `cd "C:\repo2" && git status`
- Bash tool 3: `cd "C:\repo3" && git status`

## PowerShell vs Bash on Windows

### When to Use PowerShell

Use PowerShell for:
- Windows-specific operations (registry, services, etc.)
- Complex filtering with `Where-Object`, `Select-Object`
- Working with .NET objects
- Windows system administration tasks

```powershell
# PowerShell example
powershell -Command "Get-ChildItem -Path 'C:\KolyaRepositories' -Directory | Select-Object Name"
```

### When to Use Bash (Git Bash/WSL)

Use Bash for:
- Git operations
- Standard Unix tools (grep, sed, awk)
- Cross-platform scripts
- Most development tooling (npm, docker, etc.)

```bash
# Bash example
cd "C:\KolyaRepositories\repo" && git status --short
```

## Tools to Prefer Over Bash

For file operations, use specialized Claude Code tools instead of bash commands:

| Operation | Use Tool | Not Bash |
|-----------|----------|----------|
| Read files | `Read` tool | ~~`cat`, `head`, `tail`~~ |
| Search files | `Glob` tool | ~~`find`, `ls`~~ |
| Search content | `Grep` tool | ~~`grep`, `rg`~~ |
| Edit files | `Edit` tool | ~~`sed`, `awk`~~ |
| Write files | `Write` tool | ~~`echo >`, `cat <<EOF`~~ |

Reserve Bash for actual terminal operations: git, npm, docker, build commands, etc.

## Summary Checklist

Before running Windows filesystem commands:

- [ ] Paths with backslashes are quoted: `"C:\Path\To\Dir"`
- [ ] Using `nul` instead of `/dev/null` for output redirection
- [ ] No unnecessary pipes after `cd` (especially `findstr`, `Where-Object`)
- [ ] Using specialized tools (Read, Glob, Grep) instead of bash for file ops
- [ ] Multiple independent operations use parallel tool calls
- [ ] Skills use forward slashes: `scripts/helper.py`
- [ ] PowerShell used for Windows-specific tasks, Bash for Unix-style operations
- [ ] Using `-not` instead of `!` for PowerShell negation
- [ ] Complex PowerShell logic in `.ps1` files, not inline commands

## Examples from Real Errors

**Common Error Patterns:**
1. `findstr` after `cd` → FINDSTR: Cannot open [dirname]
2. PowerShell pipe after `cd` → The term 'C:\...' is not recognized
3. Unquoted path → cd: C:KolyaRepositories... (backslashes stripped)
4. Using `/dev/null` → No such file or directory
5. Using `!` in PowerShell via Bash → `\!` is not recognized
6. Inline `$variable` in PowerShell → variables are empty/stripped

**Resolution:**
All resolved by following the patterns in this skill:
- Quote paths: `cd "C:\KolyaRepositories\repo"`
- Avoid pipes after cd: `git status --short` (no `| findstr`)
- Use `nul` not `/dev/null`: `command >nul 2>&1`
- Use `-not` instead of `!` for negation
- Use `.ps1` script files for complex PowerShell logic
