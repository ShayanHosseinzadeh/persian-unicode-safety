# Mojibake Diagnosis and Repair

Use this reference when Persian text is already corrupted or display looks suspicious.

## First Response

1. Stop broad editing.
2. Preserve evidence:
   - Check Git history if available.
   - Copy the file before rewriting if it is not versioned.
   - Inspect raw bytes before guessing.
3. Determine whether the problem is display-only or persisted corruption.

## Common Signals

- Replacement character `�`: bytes were decoded with errors and information may be lost.
- Latin-1-looking sequences such as `Ø§`, `Ù`, `Û`: UTF-8 bytes were likely decoded as Windows-1252 or Latin-1.
- Escaped octal path fragments in Git such as `\330\264`: Git path quoting, often display-only.
- Question marks `????`: text may have been encoded through a code page that cannot represent Persian; original data is often unrecoverable without a backup.

## Byte Inspection

Python:

```sh
python3 - <<'PY' path/to/file
from pathlib import Path
import sys
path = Path(sys.argv[1])
data = path.read_bytes()
print(data[:200].hex(" "))
for enc in ("utf-8", "utf-8-sig", "cp1252", "latin-1"):
    try:
        print(f"\n--- {enc} ---")
        print(data.decode(enc)[:500])
    except UnicodeDecodeError as exc:
        print(f"{enc}: {exc}")
PY
```

PowerShell:

```powershell
$bytes = [System.IO.File]::ReadAllBytes($path)
($bytes[0..([Math]::Min($bytes.Length - 1, 120))] | ForEach-Object { $_.ToString('X2') }) -join ' '
[System.Text.Encoding]::UTF8.GetString($bytes)
```

## Repair Patterns

If UTF-8 bytes were decoded as Latin-1/Windows-1252 and then saved as Unicode text, this may recover:

```python
bad = "Ø³Ù„Ø§Ù…"
fixed = bad.encode("latin-1").decode("utf-8")
print(fixed)
```

For Windows-1252 specifically:

```python
fixed = bad.encode("cp1252").decode("utf-8")
```

If replacement characters or literal `?` replaced original bytes, repair from backups, Git history, source data, or user-provided original text. Do not invent Persian content unless asked.

## Safe Repair Workflow

1. Reproduce the corruption on a small sample.
2. Write a reversible repair script that reads the original file and writes a new output file.
3. Compare before/after on a small diff.
4. Validate JSON/CSV/source syntax after repair.
5. Confirm Persian probe text and representative real text display correctly.

## Do Not

- Do not run bulk transcodes over the whole repo without file-type filtering.
- Do not combine encoding repair with unrelated formatting.
- Do not assume all files have the same encoding.
- Do not treat Git-escaped path output as file content corruption.
