# EPUB → PDF Converter

Client-side EPUB to PDF converter. Runs entirely in the browser — no server, no uploads.

**Live**: https://ravmike.github.io/epub-to-pdf/

## Architecture

Single-file HTML app (`index.html`) using:
- **JSZip** — unpack EPUB (which is a ZIP)
- **jsPDF** — generate PDF
- **Noto Sans** — Unicode font loaded from Google Fonts CDN at runtime

Flow: `EPUB (ZIP) → parse OPF spine → extract chapter XHTML → toBlocks() → render to PDF`

## Key functions

- `parseEpub(arrayBuffer)` — unzips EPUB, reads OPF, resolves spine order, returns chapters as HTML strings
- `toBlocks(html)` — parses chapter HTML into an array of typed blocks: `{T:'h'}` (heading), `{T:'p'}` (paragraph), `{T:'br'}` (break), `{T:'hr'}`, `{T:'img'}`, `{T:'t'}` (bare text)
- `wr(text, bold, fontSize, after, firstIndent)` — writes text to PDF with word-wrap, page breaks, and spacing
- Main render loop — iterates blocks, applies formatting per type, collects outline entries

## Chapter detection

Many EPUBs (especially FB2→EPUB conversions) lack semantic `<h1>`–`<h6>` tags. The converter detects chapters by:
1. `<p>` elements consisting entirely of `<strong>` text (length > 3 chars, not just a dash)
2. Text matching patterns like "Глава N.", "Chapter N.", "Часть", "Part", etc.

## PDF features

- Paragraph first-line indent (7mm)
- Chapters on new pages with 1.5× font size, bold
- Empty-line breaks preserved from `<p class="empty-line"/>`
- PDF outline/bookmarks for chapter navigation
- Configurable page size, font size, margins, line height

## Testing

Sample EPUB files go in `samples/` (gitignored). Use Playwright via conda env `epub-test`:
```bash
conda run -n epub-test python /tmp/test_deployed.py
```

## Deployment

GitHub Pages via Actions workflow (`.github/workflows/pages.yml`). Pushes to `main` auto-deploy.
