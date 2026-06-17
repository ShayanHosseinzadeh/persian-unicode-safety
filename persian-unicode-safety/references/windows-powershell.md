# Windows PowerShell and Terminal Encoding

Use this reference for Windows PowerShell 5.1, PowerShell 7+, Windows Terminal, cmd interop, and scripts that write Persian text on Windows.

## Facts to Remember

- PowerShell 7+ defaults to UTF-8 without BOM for text output.
- Windows PowerShell 5.1 has inconsistent defaults:
  - `>` and `>>` are based on `Out-File` and create UTF-16LE unless configured.
  - `Set-Content` and `Add-Content` can use the active ANSI code page for files without a BOM.
  - Reading a `.ps1` without a BOM can misinterpret non-ASCII literals as the legacy ANSI code page.
- UTF-8 with BOM can help Windows PowerShell 5.1 parse `.ps1` files that contain Persian literals, but BOMs can annoy Unix tools. Use BOM only for that compatibility need.

Sources:

- Microsoft PowerShell `about_Character_Encoding`: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_character_encoding
- Microsoft `Understanding file encoding in VS Code and PowerShell`: https://learn.microsoft.com/en-us/powershell/scripting/dev-cross-plat/vscode/understanding-file-encoding

## Preflight

```powershell
$PSVersionTable.PSVersion
[Console]::InputEncoding
[Console]::OutputEncoding
$OutputEncoding
chcp
```

If the active session is not UTF-8-safe, start or configure a UTF-8 session before reading or writing Persian text. Prefer PowerShell 7+ in Windows Terminal when available.

If using PowerShell 7+, prefer:

```powershell
[Console]::InputEncoding = [System.Text.UTF8Encoding]::new($false)
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
$OutputEncoding = [System.Text.UTF8Encoding]::new($false)
```

For Windows PowerShell 5.1, set console output explicitly for the session before testing:

```powershell
[Console]::InputEncoding = [System.Text.UTF8Encoding]::new($false)
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new($false)
$OutputEncoding = [System.Text.UTF8Encoding]::new($false)
```

`chcp 65001` can help legacy console programs, but do not treat it as sufficient by itself. Always run a round-trip test.

## UTF-8 Session Preset

Use this at the start of a PowerShell session that will handle Persian text:

```powershell
$utf8NoBom = [System.Text.UTF8Encoding]::new($false)
[Console]::InputEncoding = $utf8NoBom
[Console]::OutputEncoding = $utf8NoBom
$OutputEncoding = $utf8NoBom
$env:PYTHONUTF8 = '1'
$env:PYTHONIOENCODING = 'utf-8'
```

For external commands that still depend on the Windows console code page, run:

```powershell
chcp 65001
```

Use this preset as a session-level guardrail; still write files through explicit UTF-8 APIs.

To start a clean UTF-8 PowerShell 7 session from another shell:

```powershell
pwsh -NoLogo -NoProfile
```

Then apply the preset above and run the round-trip probe.

## Safe File Writes

Prefer .NET APIs when exact UTF-8 matters:

```powershell
$text = 'فارسی، ی، ک، نیم‌فاصله'
$utf8NoBom = [System.Text.UTF8Encoding]::new($false)
[System.IO.File]::WriteAllText($path, $text, $utf8NoBom)
$read = [System.IO.File]::ReadAllText($path, [System.Text.Encoding]::UTF8)
if ($read -ne $text) { throw 'UTF-8 round trip failed' }
```

PowerShell 7+ safe shorthand:

```powershell
Set-Content -LiteralPath $path -Value $text -Encoding utf8NoBOM
Get-Content -LiteralPath $path -Raw -Encoding utf8
```

Windows PowerShell 5.1 has no `utf8NoBOM` `-Encoding` value. Use .NET APIs for UTF-8 without BOM.

## Avoid

- Do not use `>` or `>>` for Persian text in Windows PowerShell 5.1.
- Do not append without confirming the target file encoding.
- Do not assume correct display means correct bytes; inspect bytes or read back with UTF-8.
- Do not use `-Encoding Default`, `Oem`, or `Ascii` for Persian text.

## Script Files

- Cross-platform `.ps1`: use UTF-8 without BOM if run by PowerShell 7+.
- Windows PowerShell 5.1 with Persian literals in the script source: save as UTF-8 with BOM, or avoid literals by loading UTF-8 data files through .NET APIs.
