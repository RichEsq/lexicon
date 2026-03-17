# Lexicon

**Write contracts in plain text. Get professionally formatted output.**

Lexicon is a plain-text legal contract format derived from Markdown. It extends standard Markdown with conventions for legal clause numbering, defined terms, cross-references, schedules, and signature blocks — then compiles to polished `.docx` output.

A Lexicon document is valid Markdown. It renders correctly in GitHub, Obsidian, VS Code, or any Markdown viewer. But when processed by Lexicon tooling, it becomes a production-ready legal contract.

[Website](https://lexicon.esq) | [Try the Playground](https://play.lexicon.esq) | [Specification](spec.md) | [Example Document](example.md)

<!-- TODO: Screenshot — hero image showing Lexicon source on the left, rendered .docx output on the right -->

## Why Markdown for Legal Documents?

Legal drafting has been locked inside opaque binary formats (`.docx`, `.pdf`) for decades. Contracts can't be meaningfully diffed, version control is limited to "Final_v3_FINAL(2).docx", and the formatting layer is tightly coupled to the content.

Markdown fixes the obvious problems — git-native version control, clean diffs, editor independence. But there's a more compelling reason in 2026:

**LLMs are exceptionally good at reading and writing Markdown.**

Large language models are trained overwhelmingly on plain text and Markdown. When a contract lives in `.docx`, working with an LLM means extracting text, losing structure, and hoping the model infers the clause hierarchy from indentation artefacts. When a contract lives in Lexicon Markdown:

- An LLM can **read the full contract** as-is — the structure, defined terms, cross-references, and metadata are all visible in the plain text
- An LLM can **draft new clauses** that slot directly into the document with correct syntax, anchors, and term references
- An LLM can **review and redline** by producing a diff against the original — reviewable in git, not tracked changes
- An LLM can **validate consistency** — checking that defined terms are used, cross-references resolve, and clause numbering holds
- The entire **negotiation history** lives in git commits, not email chains of annotated Word documents

Lexicon makes contracts a first-class input and output format for AI-assisted legal work, without sacrificing the `.docx` output that clients and counterparties expect.

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

A few things to notice:

- **YAML front-matter** at the top declares the parties, document type, and a schedule — the processor uses this to generate cover pages, signature blocks, and schedule sections
- **`{#definitions}`** is a cross-reference anchor — other clauses can link to it with `[clause 1](#definitions)`, and the processor auto-resolves the clause number
- **`**Claim** means ...`** defines a term — the processor tracks it and warns if it's never referenced
- **`**Payment** has the meaning given by the Schedule`** — the processor detects this phrase and auto-generates a schedule page with a blank entry for "Payment"
- Nested numbered lists (`1.`, indented `1.`, indented `1.`) become legal numbering: `1.`, `1.1`, `(a)`, `(i)`

<!-- TODO: Screenshot — the above example rendered as a formatted .docx -->

## What the Processor Does

Without a processor, you have a clean, readable Markdown document. With the processor, you get:

- **Legal clause numbering** — `1.`, `1.1`, `(a)`, `(i)`
- **Cross-reference resolution** — `[clause 1](#definitions)` auto-resolves to the correct clause number
- **Defined term validation** — warns if a defined term is never used
- **Cover pages, TOC, headers/footers** — generated from front-matter metadata
- **Schedule generation** — defined terms referencing a schedule are auto-collected into a schedule page
- **Signature pages** — jurisdiction-aware execution blocks
- **Draft watermarks** — automatic "DRAFT" watermark when `status: draft`

<!-- TODO: Screenshot — example of a generated cover page or signature block -->

## Try It

The fastest way to see Lexicon in action is the **[Playground](https://play.lexicon.esq)** — paste or edit Lexicon Markdown on the left, see formatted output on the right.

<!-- TODO: Screenshot — playground UI -->

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

## Specification

The full Lexicon Markdown specification is in [`spec.md`](spec.md). It covers:

- Document structure and clause hierarchy
- YAML front-matter fields and validation rules
- Defined terms and term validation
- Cross-reference anchors and resolution
- Schedule declaration and phrase-based item detection
- Addenda and exhibit declarations
- Processor capabilities and requirements

## Repository Structure

| Path | Description |
|------|-------------|
| [`spec.md`](spec.md) | The Lexicon Markdown specification (v1.0-draft) |
| [`example.md`](example.md) | A real-world Data Processing Agreement in Lexicon format |

## License

MIT
