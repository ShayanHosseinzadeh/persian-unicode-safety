# macOS, Linux, WSL, Bash, Zsh, and CI

Use this reference for POSIX shells, Git Bash, WSL, Docker, CI, and remote Linux/macOS terminals.

## Preflight

```sh
locale
printf '%s\n' "$LANG" "$LC_ALL" "$LC_CTYPE"
printf '%s\n' 'فارسی، ی، ک، نیم‌فاصله'
```

Expect a UTF-8 locale such as `C.UTF-8`, `en_US.UTF-8`, or another `*.UTF-8` locale. If `LANG`, `LC_ALL`, or `LC_CTYPE` is `C`, `POSIX`, or non-UTF-8, fix the environment before processing Persian text.

If the active terminal is not UTF-8-safe, start a clean shell with a UTF-8 locale or export UTF-8 locale variables before reading or writing Persian text.

Common UTF-8 session preset:

```sh
export LANG=C.UTF-8
export LC_ALL=C.UTF-8
export PYTHONUTF8=1
export PYTHONIOENCODING=utf-8
```

On macOS, `en_US.UTF-8` is commonly available:

```sh
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

Do not hard-code one locale for all machines when writing reusable docs. Tell users to choose an installed UTF-8 locale.

## Round-Trip Probe

```sh
probe='فارسی، ی، ک، نیم‌فاصله'
tmp="$(mktemp)"
printf '%s' "$probe" > "$tmp"
python3 - <<'PY' "$tmp" "$probe"
from pathlib import Path
import sys
path = Path(sys.argv[1])
expected = sys.argv[2]
actual = path.read_text(encoding="utf-8")
if actual != expected:
    raise SystemExit("UTF-8 round trip failed")
print("UTF-8 round trip ok")
PY
rm -f "$tmp"
```

## Safe Shell Habits

- Use `printf` instead of `echo` for exact text.
- Quote variables and paths.
- Prefer tools that treat input as UTF-8 under the active locale.
- For machine parsing filenames, prefer NUL-delimited output such as `find -print0`, `xargs -0`, and `git -z`.
- In Docker/CI images, set UTF-8 locale variables explicitly because minimal images may default to `C`.

## Common Failure Modes

- A local terminal displays Persian correctly, but CI corrupts files because CI runs under `LANG=C`.
- Git Bash or MinTTY displays text correctly, but a Windows program invoked from it expects a different code page.
- A script uses `sed`, `awk`, or byte-counting commands in ways that assume single-byte characters.
