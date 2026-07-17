# Tectonic and Patch Tool Notes

## Tectonic (lightweight LaTeX)

The environment has a lightweight LaTeX compiler at `/opt/data/home/.local/bin/tectonic`.
It auto-downloads packages and is much faster than full TeX Live. Use it to verify `.tex` edits:

```bash
cd <paper_dir> && /opt/data/home/.local/bin/tectonic main.tex 2>&1 | grep -E "Error|Writing"
```

Output format: `note: Writing 'main.pdf' (130.70 KiB)` on success, `error:` lines on failure.

## Patch tool and LaTeX backslash doubling

The `patch` tool (both modes) doubles backslashes in LaTeX replacements. After any `patch`
call on a `.tex` file, check for doubled `\\begin`, `\\end`, `\\operatorname`, `\\neq`, etc.
Fix immediately with:

```bash
python3 -c "
import re
c = open('file.tex').read()
c = re.sub(r'\\\\\\\\', r'\\\\', c)
open('file.tex', 'w').write(c)
"
```

Prevention: prefer `write_file` for full-file LaTeX writes. Use `patch` only for content
that doesn't contain LaTeX commands, or always follow with the Python fix.
