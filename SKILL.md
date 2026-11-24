---
name: literature-pdf-parser
description: Parse a scientific paper PDF into structured, evidence-mapped output. Corrects multi-column reading order, reconstructs section hierarchy and tables, preserves formulas and special characters, and tags every block with page and bounding-box provenance; flags pages that need OCR. Use whenever the task requires reading, extracting, answering questions about, or structuring content from a research-paper PDF.
---

# Literature PDF Parser

Convert a research PDF into reusable, verifiable structure. The goal is not "text that looks like the whole paper", but every paragraph, heading, table, figure caption and formula placed in the right order and tied back to a page and region of the original file.

## When to use

Use this Skill when the input is one or more scientific PDFs and the task asks for any of: full-text extraction, section/chapter structure, table extraction, formula or symbol preservation, question answering with citations, or knowledge structuring. Do **not** use it to summarise the science or to invent content.

## Inputs

At execution time the original material is under `/input` (typically `case.json` and `task.md`, plus the actual PDF). Read those files first. In particular:

1. Find the PDF path and any task-specific output contract in `case.json`.
2. Read `task.md` to understand what to return.
3. Never assume the paper's content from its filename or topic labels; read the actual file.

## Workflow

### 1. Inventory the input
List `/input`, read `case.json` and `task.md`, and record the output_contract fields (the keys the final `result.json` must contain) plus the `case_id`.

### 2. Extract text and layout
Prefer the environment's PDF tools when available (`pdf-text-tool`, `pdf-render-tool`). The core structured parsing is done by the bundled script:

```bash
python3 src/parse_pdf.py /input/<paper>.pdf /app/work/parsed
```

This produces, under `/app/work/parsed`:
- `parsed.json` — structured blocks, sections, tables, figures, formulas, reading order, uncertainties;
- `parsed.md` — human-readable Markdown with `[pN]` page markers;
- `result.json` — a skeleton summary (status / sources / uncertainty / change_records) to be completed in step 5.

### 3. Handle scanned pages
`parsed.json` → `uncertainties` lists every page with no usable text layer. For those pages, run OCR with the environment's `ocr-tool` (or `pdf-render-tool` to rasterise first), then merge the recognised text back into the affected page's blocks, keeping page and region provenance. If OCR is unavailable, record the page as unreadable instead of guessing.

### 4. Assemble the answer
Follow the task's output contract. For every key conclusion or extracted value, keep its provenance (page, paragraph, field, table cell, formula number). State explicitly what is *not reported*, what *cannot be determined*, and any *conflicts* in the material.

### 5. Write the final result
Write `/app/output/result.json` with:
- all original `output_contract` fields, verbatim keys;
- `case_id`;
- `status` (`ok` / `partial` / `unreadable`), plus `sources`, `uncertainty`, and `change_records`;
- references to `parsed.json` / `parsed.md` as the evidence layer.

Also copy the structured parse (`parsed.json`, `parsed.md`) and any editable artifacts into `/app/output`.

## Quality checks before finishing

- [ ] Reading order: for multi-column pages, no adjacent paragraphs from different columns are merged into one sentence.
- [ ] Headings: section hierarchy is intact and numbered correctly; headers/footers are not mixed into body text.
- [ ] Tables: row/column and merged-cell structure is preserved as Markdown and matches the source page.
- [ ] Formulas/symbols: subscripts, superscripts, Greek letters and math symbols are preserved, or explicitly marked uncertain.
- [ ] Evidence: every paragraph, table and figure carries a page (and, where available, bounding-box) reference.
- [ ] Honesty: unreadable, missing, or ambiguous parts are listed in `uncertainty`, never silently filled in.

## Failure handling

- Encrypted or locked PDF → record it and stop parsing that file; ask for a password only if the task allows.
- No text layer and no OCR tool → output the page as `unreadable` with a clear reason.
- Broken fonts / replacement glyphs → keep the raw text and flag the block as `uncertain`.
- Page limit or time budget → save partial results incrementally to `/app/work` and mark status `partial`.

## Notes

- The parser is a best-effort layout engine; it does not interpret the science.
- Bundled files are relative to the Skill directory; the only runtime dependency is PyMuPDF (`pip install pymupdf`), with a pure-text fallback if it is absent.
