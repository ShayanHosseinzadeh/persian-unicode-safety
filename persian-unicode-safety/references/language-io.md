# Language and File I/O Rules

Use this reference when writing or editing Python, Node.js, JSON, Markdown, CSV/TSV, config files, generated files, and test fixtures containing Persian text.

## Python

Always pass an encoding for text files:

```python
from pathlib import Path

probe = "فارسی، ی، ک، نیم‌فاصله"
path = Path("sample.txt")
path.write_text(probe, encoding="utf-8", newline="\n")
assert path.read_text(encoding="utf-8") == probe
```

For scripts launched in uncertain environments:

```sh
PYTHONUTF8=1 python script.py
python -X utf8 script.py
```

`PYTHONIOENCODING=utf-8` can control standard streams, but it is not a substitute for explicit file encodings.

Source: Python UTF-8 Mode, PEP 540: https://peps.python.org/pep-0540/

Avoid:

```python
open(path).read()
open(path, "w").write(text)
Path(path).write_text(text)
```

unless the project deliberately relies on a known UTF-8 default and that assumption is verified.

## Node.js

Use explicit UTF-8 for text:

```js
import { readFile, writeFile } from "node:fs/promises";

const probe = "فارسی، ی، ک، نیم‌فاصله";
await writeFile("sample.txt", probe, { encoding: "utf8" });
const actual = await readFile("sample.txt", "utf8");
if (actual !== probe) throw new Error("UTF-8 round trip failed");
```

Buffers are useful for byte inspection:

```js
const bytes = await readFile("sample.txt");
console.log(bytes.toString("hex"));
```

## JSON

- Parse and write with the platform JSON library.
- Write UTF-8 text.
- Do not repair JSON with regex when Persian strings are present.
- Escaped Unicode such as `\u0641` is valid JSON, but direct UTF-8 Persian text is usually easier to review.
- Preserve existing formatting if the repo has a formatter or stable style.

## Markdown and Source Files

- Prefer UTF-8 without BOM unless a tool requires otherwise.
- Keep Persian examples as literal UTF-8 when the file and toolchain support it.
- If source files include an encoding declaration, keep it accurate.

## CSV and TSV

- Use a CSV parser/writer, not string splitting.
- Confirm the expected encoding of downstream tools. Excel on older Windows setups may expect BOM-marked UTF-8 for easy opening, while cross-platform pipelines usually prefer UTF-8 without BOM.
- Include a small fixture row with Persian text in tests when CSV export/import is part of the task.

## Generated Files

- Make encoding an explicit parameter in generators.
- Add regression tests that compare decoded text, not just file existence.
- For binary formats such as `.docx`, `.xlsx`, `.pptx`, or PDF, use format-specific libraries and render/extract to verify Persian text survives.
