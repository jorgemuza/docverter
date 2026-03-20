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

### Build from source

```bash
git clone https://github.com/jorgemuza/docverter-cli.git
cd docverter-cli
go build -o docverter .
```

## Quick Start

```bash
# Markdown to PDF
docverter convert README.md

# Markdown to DOCX (template directory required)
docverter convert README.md --format docx --template ./templates/epsilon

# Shorthand — "convert" is the default command
docverter README.md -f docx --template ./templates/epsilon

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
templates/epsilon/
  multipage-template.docx
  single-page-template.docx
  letterhead-template.docx
  rfp-template.dotx
```

Select the document layout with `--type`:

```bash
# Multipage: cover page + TOC + content (default)
docverter convert doc.md -f docx --template ./templates/epsilon --type multipage

# Single-page: content only
docverter convert doc.md -f docx --template ./templates/epsilon --type single-page
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
docverter batch "docs/*.md" --format docx --template ./templates/epsilon --dry-run
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

## Project Structure

```
docverter/
├── main.go                        # Entry point
├── cmd/                           # Cobra CLI commands
│   ├── root.go                    # Root command + global flags
│   ├── convert.go                 # Single file conversion
│   ├── batch.go                   # Glob-based batch conversion
│   └── version.go                 # Version command
├── internal/
│   ├── config/config.go           # Configuration types
│   ├── detect/detect.go           # Input type + format detection
│   ├── markdown/
│   │   ├── parser.go              # Goldmark parser + frontmatter
│   │   └── rawxml.go              # {=openxml} block extension
│   ├── docx/
│   │   ├── writer.go              # Orchestrator: parse → render → write ZIP
│   │   ├── template.go            # Template loader (ZIP manipulation)
│   │   ├── renderer.go            # AST walker → OpenXML elements
│   │   ├── openxml.go             # Element types: Paragraph, Run, Table, etc.
│   │   ├── styles.go              # Heading level → Word style ID mapping
│   │   ├── cover.go               # Cover page generation
│   │   ├── toc.go                 # Native Word TOC field
│   │   ├── numbering.go           # List numbering definitions
│   │   └── images.go              # Image embedding + EMU sizing
│   ├── pdf/
│   │   ├── writer.go              # PDF conversion orchestrator
│   │   ├── html.go                # Goldmark → HTML with syntax highlighting
│   │   └── chrome.go              # chromedp HTML → PDF
│   ├── excalidraw/
│   │   └── export.go              # Subprocess wrapper
│   └── glob/
│       └── glob.go                # Glob pattern matching
├── templates/                     # Example template directory
│   └── epsilon/
└── testdata/                      # Test fixtures
```

## License

MIT
