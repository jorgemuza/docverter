# Docverter

A single Go binary that converts Markdown to DOCX and PDF, and Excalidraw diagrams to SVG/PNG.

## Installation

### Homebrew (macOS / Linux)

```bash
brew install jorgemuza/tap/docverter
```

### Scoop (Windows)

```powershell
scoop bucket add jorgemuza https://github.com/jorgemuza/scoop-bucket.git
scoop install docverter
```

### Shell script

```bash
curl -sSfL https://raw.githubusercontent.com/jorgemuza/docverter/main/install.sh | sh
```

### Go install

```bash
go install github.com/jorgemuza/docverter@latest
```

## Quick Start

```bash
# Markdown to PDF
docverter convert README.md

# Markdown to DOCX (template directory required)
docverter convert README.md --format docx --template ./path/to/templates

# Shorthand — "convert" is the default command
docverter README.md -f docx --template ./path/to/templates

# Excalidraw to SVG
docverter convert architecture.excalidraw

# Batch convert
docverter batch "docs/**/*.md" --format pdf
```

## Supported Conversions

| Input | Output | Engine |
|-------|--------|--------|
| `.md` | `.pdf` | Goldmark + chromedp (headless Chrome) |
| `.md` | `.docx` | Goldmark + direct OpenXML generation |
| `.excalidraw` | `.svg`, `.png` | excalidraw-brute-export-cli (Node.js subprocess) |

## DOCX Templates

DOCX conversion requires a `--template` flag pointing to a directory with Word template files. The template provides styles, headers, footers, numbering, and themes — docverter replaces only the document body.

```
my-templates/
  multipage-template.docx
  single-page-template.docx
  letterhead-template.docx
  rfp-template.dotx
```

Select the document layout with `--type`:

```bash
# Multipage: cover page + TOC + content (default)
docverter convert doc.md -f docx --template ./path/to/templates --type multipage

# Single-page: content only
docverter convert doc.md -f docx --template ./path/to/templates --type single-page
```

### Frontmatter

Title, subtitle, and date are extracted from YAML frontmatter:

```yaml
---
title: "Architecture Overview"
subtitle: "Q1 2026 Review"
date: "March 20, 2026"
---
```

If `title` is omitted, the first `# H1` heading is used. If no H1, the filename is title-cased.

## PDF Generation

PDF conversion uses headless Chrome (via chromedp). Chrome or Chromium must be installed.

```bash
docverter convert doc.md                              # default: github highlight theme
docverter convert doc.md --highlight monokai          # alternative themes
```

Available highlight styles: `github`, `monokai`, `solarized-light`, `atom-one-dark`.

## Batch Mode

Convert multiple files with glob patterns:

```bash
docverter batch "docs/**/*.md" --format pdf
docverter batch "**/*.excalidraw" --format png --scale 3
docverter batch "docs/*.md" --format docx --template ./path/to/templates --dry-run
```

Use `--dry-run` to preview what would be converted without writing files.

## Prerequisites

| Feature | Requirement |
|---------|-------------|
| Core (DOCX) | Go 1.21+, template directory |
| PDF | Google Chrome or Chromium |
| Excalidraw | Node.js + npm |

## Documentation

- [Command Reference](docs/command-reference.md) — all commands, flags, and options

## License

MIT
