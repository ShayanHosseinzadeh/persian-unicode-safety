# Git, Editor, and Repository Display

Use this reference for Git status/diff/log, non-ASCII filenames, VS Code settings, line endings, and repository hygiene.

## Git Path Display

Git defaults to quoting path bytes above `0x80` in some command output. For human inspection of Persian filenames:

```sh
git -c core.quotepath=false status --short
git -c core.quotepath=false diff --name-status
```

For machine parsing, prefer NUL-delimited output:

```sh
git diff --name-only -z
git ls-files -z
```

Do not parse quoted Git path output as if it were the original filename.

Source: Git `core.quotePath` config: https://git-scm.com/docs/git-config

## Diff Inspection

Use UTF-8-capable terminal/editor output:

```sh
git -c core.quotepath=false diff -- path/to/file
```

If terminal display is suspicious, inspect bytes and decode through a known UTF-8 path:

```sh
python3 - <<'PY' path/to/file
from pathlib import Path
import sys
data = Path(sys.argv[1]).read_bytes()
print(data.decode("utf-8"))
PY
```

## VS Code

Useful settings:

```json
{
  "files.encoding": "utf8",
  "files.autoGuessEncoding": true,
  "files.eol": "\n"
}
```

For Windows PowerShell 5.1 `.ps1` files that contain Persian literals, use a language-specific override when required:

```json
{
  "[powershell]": {
    "files.encoding": "utf8bom",
    "files.autoGuessEncoding": true
  }
}
```

Use repository or workspace settings only when the project needs them. Avoid changing a user's global editor settings unless explicitly requested.

## Line Endings

Line-ending conversion is separate from text encoding, but both can confuse reviews. If the repo has `.gitattributes`, follow it. For new cross-platform text files:

```gitattributes
* text=auto
*.md text eol=lf
*.json text eol=lf
*.ps1 text eol=crlf
```

Only add or change `.gitattributes` when the task includes repository hygiene or repeated encoding/line-ending churn.

## Review Checklist

- `git -c core.quotepath=false status --short` shows expected paths.
- Diff content displays Persian text directly or the file bytes decode as UTF-8.
- No unintended file-wide rewrite happened from encoding or line-ending conversion.
- Editor settings are local to the repo when possible.
