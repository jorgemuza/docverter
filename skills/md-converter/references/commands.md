# Docverter Command Reference

## Synopsis

```
docverter [command] [flags]
docverter <file> [flags]
```

When called without a subcommand, `docverter` treats the first argument as a file and runs the `convert` command.

## Commands

### `convert`

Convert a single file to the specified format.

```
docverter convert <file> [flags]
```

**Arguments:**

| Argument | Description |
|----------|-------------|
| `<file>` | Path to the input file (.md, .excalidraw) |

**Examples:**

```bash
# Markdown to PDF (default format for .md)
docverter convert README.md

# Markdown to DOCX (requires --template)
docverter convert README.md --format docx --template ./templates/epsilon

# Markdown to DOCX with single-page layout
docverter convert README.md -f docx --template ./templates/epsilon --type single-page

# Excalidraw to SVG (default format for .excalidraw)
docverter convert arch.excalidraw

# Excalidraw to PNG with custom scale
docverter convert arch.excalidraw --format png --scale 3

# Shorthand (omit "convert")
docverter README.md -f docx --template ./templates/epsilon
```

### `batch`

Batch convert files matching a glob pattern. Supports `**` for recursive matching.

```
docverter batch <glob-pattern> [flags]
```

**Arguments:**

| Argument | Description |
|----------|-------------|
| `<glob-pattern>` | Glob pattern to match input files (quote to prevent shell expansion) |

**Examples:**

```bash
# Convert all markdown files to PDF
docverter batch "docs/**/*.md" --format pdf

# Convert all excalidraw files to PNG
docverter batch "**/*.excalidraw" --format png

# Preview what would be converted
docverter batch "docs/*.md" --format docx --template ./templates/epsilon --dry-run
```

### `version`

Print the version number.

```
docverter version
```

## Global Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--format` | `-f` | auto-detected | Output format: `pdf`, `docx`, `svg`, `png` |
| `--output` | `-o` | same dir as input | Output file path or directory |
| `--template` | | *(required for DOCX)* | Path to directory containing `.docx`/`.dotx` template files |
| `--type` | | `multipage` | DOCX document type: `multipage`, `single-page`, `letterhead`, `rfp` |
| `--highlight` | | `github` | PDF code highlight style: `github`, `monokai`, `solarized-light`, `atom-one-dark` |
| `--no-styles` | | `false` | Disable custom Word style mappings (headings, tables, etc.) |
| `--scale` | | `2` | Excalidraw export scale factor |
| `--dry-run` | | `false` | Preview conversions without writing files |
| `--verbose` | `-v` | `false` | Enable debug logging |

## Format Auto-Detection

When `--format` is omitted, the output format is inferred from the input file extension:

| Input Extension | Default Output Format |
|-----------------|----------------------|
| `.md`, `.markdown` | `pdf` |
| `.excalidraw` | `svg` |

## Document Types (DOCX)

The `--type` flag controls the DOCX document structure. Each type expects a corresponding template file in the `--template` directory:

| Type | Template File | Description |
|------|--------------|-------------|
| `multipage` | `multipage-template.docx` | Cover page + Table of Contents + content body |
| `single-page` | `single-page-template.docx` | Content body only, no cover or TOC |
| `letterhead` | `letterhead-template.docx` | Letterhead-styled document |
| `rfp` | `rfp-template.dotx` | RFP/proposal template |

### Multipage Layout

The `multipage` type automatically generates:

1. **Cover page** — title, subtitle, and date (extracted from YAML frontmatter or inferred from the document)
2. **Table of Contents** — native Word TOC field (right-click > "Update Field" in Word to populate)
3. **Content body** — the rendered markdown

### Frontmatter Fields

YAML frontmatter at the top of a markdown file controls document metadata:

```yaml
---
title: "My Document"
subtitle: "Optional subtitle"
date: "March 20, 2026"
author: "Name"
---
```

If `title` is omitted, the first `# H1` heading is used. If no H1 exists, the filename is title-cased.

## DOCX Style Mapping

Markdown elements are mapped to Word styles defined in the template:

| Markdown | Word Style | Notes |
|----------|-----------|-------|
| `## H2` | Heading 1 | Main sections |
| `### H3` | Heading 2 | Subsections |
| `#### H4` | H3Heading | |
| `##### H5` | H4Heading | |
| `- bullet` | Body Bullet | With numbering properties |
| `` ``` code ``` `` | Source Code | Consolas font |
| `> blockquote` | NotesTable | Single-cell callout table |
| `\| table \|` | List Table 3 | With header row |
| `---` (horizontal rule) | *(page break)* | Converted to page break |
| Single newline | *(line break)* | Preserved as `<w:br/>` |

Use `--no-styles` to disable this mapping and use default Word heading styles.

## Raw OpenXML Passthrough

For precise control over DOCX output, use fenced code blocks with the `{=openxml}` info string:

````markdown
```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```
````

The XML content is injected directly into the document body.

## Prerequisites

| Feature | Requirement |
|---------|-------------|
| DOCX conversion | Template directory with `.docx` files |
| PDF conversion | Google Chrome or Chromium (used headlessly via chromedp) |
| Excalidraw export | Node.js + npm (`npx excalidraw-brute-export-cli`) |
