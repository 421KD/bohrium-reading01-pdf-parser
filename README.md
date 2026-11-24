# literature-pdf-parser

A Skill + companion script that turns a scientific PDF into structured, evidence-mapped output.

## What it does

1. Extracts text with page and bounding-box provenance.
2. Corrects multi-column reading order (two-column and spanning blocks).
3. Reconstructs the section hierarchy (heading levels).
4. Extracts tables to Markdown (using PyMuPDF's table finder when available).
5. Detects figure captions and formula/special-character spans.
6. Flags pages that have no text layer and therefore need OCR.
7. Emits `parsed.json`, `parsed.md`, and a `result.json` skeleton.

## Files

| Path | Purpose |
|---|---|
| `SKILL.md` | Skill definition (name/description frontmatter + instructions) |
| `src/parse_pdf.py` | Layout-aware parser |
| `src/requirements.txt` | Runtime dependency (PyMuPDF) |
| `README.md` | This document |

## Install and run

```bash
pip install -r src/requirements.txt
python3 src/parse_pdf.py /path/to/paper.pdf /path/to/outdir
```

If PyMuPDF is not available the script falls back to a pure-text extraction that still keeps page provenance but has coarser layout information (no column-aware reordering, no table reconstruction).

## Inputs and outputs

Input: a single PDF.

Outputs in the target directory:

- `parsed.json` — blocks, sections, tables, figures, formulas, reading order, uncertainties.
- `parsed.md` — readable Markdown with `[pN]` page markers.
- `result.json` — summary skeleton (status / sources / uncertainty / change_records).

## Example

```bash
python3 src/parse_pdf.py paper.pdf out
cat out/parsed.md
```

## Limitations

- Column detection targets the common one/two-column layout; exotic layouts may need manual review.
- Table extraction depends on ruled lines or PyMuPDF's detector; borderless tables are best-effort.
- Formula handling preserves the text layer and flags math-looking spans; it does not reconstruct LaTeX from images.
- Scanned PDFs (no text layer) are flagged for OCR, which must be supplied by the caller's OCR tool.

## Self-check

After a run, confirm that: adjacent columns are not merged, headers/footers are excluded from body text, section numbering is intact, tables render as valid Markdown, and every block carries a page reference.
