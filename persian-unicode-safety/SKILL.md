---
name: persian-unicode-safety
description: Prevent and repair Persian, Arabic-script, RTL, Unicode, UTF-8, mojibake, and terminal/file encoding problems in Codex work. Use when editing or generating Persian text, Arabic-script text, filenames, JSON, Markdown, CSV, source files, shell commands, PowerShell commands, Git diffs/status, Python or Node scripts, VS Code settings, cross-platform terminal instructions, or when output contains mojibake such as ØŒ Ø§ Ù or replacement characters.
---

# Persian Unicode Safety

## Core Rule

Treat Persian/Arabic-script text as an encoding-sensitive workflow. Before editing, running shell commands, or writing files, verify the active shell, file I/O path, and Git/editor display path can preserve UTF-8.

If the current terminal cannot reliably read or write UTF-8, prepare a UTF-8-configured session or use a safer terminal/runtime before touching Persian text. Do not continue in a known-bad shell just because it is already open.

Use this sample for probes when possible:

```text
فارسی، ی، ک، نیم‌فاصله
```

## Workflow

1. Identify the risk surface:
   - Windows PowerShell, PowerShell 7+, cmd, Git Bash, WSL, macOS/Linux shell, CI, or editor.
   - File type: source, JSON, Markdown, CSV/TSV, config, generated artifact, or filename.
   - Operation: read, write, append, redirect, diff, parse, repair, or display.

2. Load only the needed reference:
   - Windows PowerShell or Windows terminal work: `references/windows-powershell.md`.
   - macOS, Linux, WSL, Bash, Zsh, CI, or locale work: `references/unix-shells.md`.
   - Python, Node, JSON, Markdown, CSV, or generated file I/O: `references/language-io.md`.
   - Git status/diff, filenames, VS Code, editor settings, or line endings: `references/git-editor.md`.
   - Existing corrupted text, mojibake diagnosis, or recovery: `references/repair.md`.

3. Prefer safe primitives:
   - Use a preconfigured UTF-8 terminal/session for read and write operations when the active shell is uncertain or misconfigured.
   - Use APIs that accept explicit UTF-8 encoding.
   - Avoid legacy shell redirection for non-ASCII text unless the shell's encoding behavior is known.
   - Avoid append operations that guess or change the existing file encoding.
   - Preserve the original file encoding when editing existing files unless a UTF-8 migration is intentional.
   - Keep Persian text in UTF-8 source files and use structured parsers for JSON/CSV/config.

4. Verify before and after:
   - Print the active terminal encoding or locale.
   - If needed, switch to or create a UTF-8 shell session and rerun the probe there.
   - Write the probe string through the same path that the task will use.
   - Read bytes back and confirm UTF-8 decoding reproduces the exact text.
   - Inspect diffs with commands that do not escape or corrupt non-ASCII paths/content.

5. If text already looks corrupted:
   - Stop making broad edits.
   - Save a byte-level backup or inspect Git history before repair.
   - Use `references/repair.md` to identify whether corruption is display-only, wrong decode, double-encoding, replacement-character loss, or mixed encodings.

## Safety Defaults

- Prefer PowerShell 7+ over Windows PowerShell 5.1 for new cross-platform commands.
- In Windows PowerShell 5.1, do not rely on default `Out-File`, `>`, `>>`, `Set-Content`, or `Add-Content` behavior for Persian text.
- In Python, pass `encoding="utf-8"` to `open()`, `Path.read_text()`, and `Path.write_text()`; for process-wide behavior use `PYTHONUTF8=1` or `python -X utf8` when appropriate.
- In Node.js, use `utf8` explicitly when reading and writing text if the code path is not obvious.
- In Git, use `-c core.quotepath=false` for human inspection of non-ASCII paths, or `-z` for machine parsing.
- In VS Code and cross-platform repos, prefer UTF-8 without BOM for general text files; use UTF-8 with BOM only when Windows PowerShell 5.1 must parse a `.ps1` containing non-ASCII literals.

## Acceptance Check

Before claiming the task is safe, verify all applicable items:

- Persian probe text survives write/read round trip.
- The edited file is UTF-8 or intentionally kept in its original encoding.
- Git diff/status displays Persian content or paths intelligibly.
- No mojibake sequences or replacement characters were introduced.
- Any platform-specific advice matches the user's shell and operating system.

## Example Prompts

- Edit this Persian JSON text safely.
- Fix mojibake in this Markdown file.
- Run a PowerShell command that writes Persian text to a file.
- Inspect a Git diff for Persian filenames.
- Make this Python or Node script preserve Persian output across Windows, macOS, and Linux.
