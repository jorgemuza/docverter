---
name: md-converter
description: "Convert Markdown files to PDF or DOCX formats, and Excalidraw diagrams to SVG or PNG using the docverter CLI. Use this skill whenever the user wants to export documentation, create professional Word documents from markdown, generate printable PDFs, convert Excalidraw diagrams to images, or produce any formatted document from .md files. Trigger on requests like 'convert to PDF', 'export as DOCX', 'make a Word document', 'generate PDF from markdown', 'create a report', 'export this doc', 'I need a PDF', 'make this into a Word file', 'export excalidraw', or any task involving markdown-to-document conversion — even if the user just says 'export' or 'print-ready' without specifying the format."
---

# Docverter — Markdown to PDF/DOCX, Excalidraw to SVG/PNG

Convert markdown files to professional PDF and DOCX documents, and Excalidraw diagrams to SVG/PNG images using the `docverter` CLI.

## Prerequisites

1. `docverter` binary installed (`brew install jorgemuza/tap/docverter`)
2. For PDF: Google Chrome or Chromium
3. For DOCX: a template directory with `.docx` template files
4. For Excalidraw: Node.js + npm

## Quick Reference

```bash
# Markdown to PDF (default)
docverter convert README.md

# Markdown to DOCX (requires template)
docverter convert README.md --format docx --template ./templates

# Markdown to DOCX with single-page layout
docverter convert README.md -f docx --template ./templates --type single-page

# Excalidraw to SVG (default)
docverter convert architecture.excalidraw

# Excalidraw to PNG with custom scale
docverter convert architecture.excalidraw --format png --scale 3

# Batch convert
docverter batch "docs/**/*.md" --format pdf
docverter batch "**/*.excalidraw" --format png

# Preview (dry run)
docverter batch "docs/*.md" --format docx --template ./templates --dry-run
```

## DOCX Document Types

The `--type` flag controls the document structure:

| Type | Description |
|------|-------------|
| `multipage` | Cover page + Table of Contents + content body (default) |
| `single-page` | Content body only, no cover or TOC |
| `letterhead` | Letterhead-styled document |
| `rfp` | RFP/proposal template |

## Frontmatter

Title, subtitle, and date are extracted from YAML frontmatter:

```yaml
---
title: "Architecture Overview"
subtitle: "Q1 2026 Review"
date: "March 20, 2026"
---
```

If `title` is omitted, the first `# H1` heading is used. If no H1, the filename is title-cased.

## PDF Highlight Styles

```bash
docverter convert doc.md --highlight monokai
```

Available: `github` (default), `monokai`, `solarized-light`, `atom-one-dark`.

## Common Workflows

**Generate a report from markdown:**
```bash
docverter convert docs/report.md -f docx --template ./templates --type multipage
```

**Export all docs as PDFs:**
```bash
docverter batch "docs/**/*.md" --format pdf
```

**Export Excalidraw diagrams for documentation:**
```bash
docverter batch "**/*.excalidraw" --format png --scale 3
```

## Important Notes

- **Template required for DOCX** — The `--template` flag must point to a directory containing Word template files matching the `--type`.
- **Chrome required for PDF** — PDF generation uses headless Chrome via chromedp.
- **Node.js required for Excalidraw** — Uses `npx excalidraw-brute-export-cli` under the hood.
- **Format auto-detection** — `.md` defaults to PDF, `.excalidraw` defaults to SVG.
- **Shorthand** — `docverter README.md` is equivalent to `docverter convert README.md`.

For full command details and all flags, see `references/commands.md`.
