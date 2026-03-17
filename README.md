# Lexicon

Lexicon is a plain-text legal contract format that is derived from Markdown.

**Write contracts in plain text. Get professionally formatted output.**

## What is Lexicon Markdown?

Lexicon Markdown extends standard Markdown with conventions for legal documents. A Lexicon document is valid Markdown that renders correctly in any Markdown viewer (GitHub, Obsidian, VS Code, etc.), but when processed by Lexicon tooling, gains:

- **Legal clause numbering** — `1.`, `1.1`, `(a)`, `(i)`
- **Cross-reference resolution** — anchors like `{#termination}` and links like `[clause 3](#termination)` are auto-resolved to correct clause numbers
- **Defined term validation** — `**Confidential Information** means ...` defines a term; the processor warns if it's never used
- **Cover pages, TOC, headers/footers** — generated from YAML front-matter metadata
- **Schedule generation** — defined terms referencing a schedule are auto-collected into a schedule page
- **Signature pages** — jurisdiction-aware execution blocks with configurable templates
- **Draft watermarks** — automatic "DRAFT" watermark when `status: draft`

Without a processor, Lexicon Markdown reads as a clean, well-structured document. With a processor, it becomes a production-ready legal contract.

## Why Markdown for Legal Documents?

Legal drafting has been locked inside opaque binary formats (`.docx`, `.pdf`) for decades. This creates real problems: contracts can't be meaningfully diffed, version control is limited to "Final_v3_FINAL(2).docx", and the formatting layer is tightly coupled to the content.

Markdown changes this. But beyond the usual benefits of plain text — git-native version control, clean diffs, editor independence — there's a more compelling reason in 2026:

**LLMs are exceptionally good at reading and writing Markdown.**

Large language models are trained overwhelmingly on plain text and Markdown. When a contract lives in `.docx`, working with an LLM means extracting text, losing structure, and hoping the model infers the clause hierarchy from indentation artefacts. When a contract lives in Lexicon Markdown:

- An LLM can **read the full contract** as-is — the structure, defined terms, cross-references, and metadata are all visible in the plain text
- An LLM can **draft new clauses** that slot directly into the document with correct syntax, anchors, and term references
- An LLM can **review and redline** by producing a diff against the original — reviewable in git, not tracked changes
- An LLM can **validate consistency** — checking that defined terms are used, cross-references resolve, and clause numbering holds
- The entire **negotiation history** lives in git commits, not email chains of annotated Word documents

Lexicon Markdown makes contracts a first-class input and output format for AI-assisted legal work, without sacrificing the formatted `.docx` output that clients and counterparties expect.

## Quick Example

```markdown
---
title: Deed of Release
type: Deed
date: 2026-01-15
status: draft
parties:
  - name: Alice Smith
    role: Employee
  - name: Acme Corp
    specifier: ACN 123 456 789
    role: Employer
schedule:
  - title: Schedule
---

1. ## Definitions {#definitions}

    1. **Claim** means any and all claims, demands, or causes of action.

    2. **Confidential Information** means all information disclosed
       by one party to the other.

    3. **Payment** has the meaning given by the Schedule.

2. ## Release {#release}

    1. The Employee releases the Employer from all Claims.

    2. The obligations in [clause 1](#definitions) survive termination.

        1. This includes any Confidential Information held by the Employee.
```

## Repository Structure

| Path | Description |
|------|-------------|
| [`spec.md`](spec.md) | The Lexicon Markdown specification (v1.0-draft) |
| [`example.md`](example.md) | A real-world Data Processing Addendum in Lexicon format |
| [`lexicon-docx/`](lexicon-docx/) | Rust CLI processor — converts Lexicon Markdown to `.docx` |

## Tooling

### lexicon-docx

The [`lexicon-docx`](lexicon-docx/) CLI converts Lexicon Markdown files into formatted Word documents with legal numbering, cover pages, tables of contents, signature blocks, and more. See the [lexicon-docx README](lexicon-docx/README.md) for installation and usage.

```bash
cd lexicon-docx
cargo build --release
cargo run -- build ../example.md -o output.docx
```

## Specification

The full Lexicon Markdown specification is in [`spec.md`](spec.md). It covers:

- Document structure and clause hierarchy
- YAML front-matter fields and validation rules
- Defined terms and term validation
- Cross-reference anchors and resolution
- Schedule declaration and phrase-based item detection
- Addenda and exhibit declarations
- Processor capabilities and requirements

## Front-Matter Fields

```yaml
---
title: Contract Title          # required
type: Agreement                # optional, defaults to "Agreement"
date: 2026-01-15               # required, YYYY-MM-DD
ref: "ABC:123"                 # optional, drafter's reference
author: Jane Doe (Law Firm)    # optional
status: draft                  # optional: draft | final | executed
version: 2                     # optional
parties:                       # required
  - name: Party Name
    specifier: ACN 123 456 789 # optional
    role: Buyer                # used as a defined term
    entity_type: au-company    # optional, for signature block templates
exhibits:                      # optional
  - title: Exhibit Title
    path: ./diagram.png        # optional: local path to PNG/JPG/PDF
schedule:                      # optional
  - title: Schedule            # generates a schedule page from defined terms
---
```

## License

MIT
