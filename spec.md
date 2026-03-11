# Lexicon Markdown Spec

**Version**: 1.0-draft
**Date**: 2026-03-09

## 1. Overview

Lexicon Markdown is a contract drafting format that extends standard Markdown with conventions for legal documents. It is designed to be:

1. **Valid Markdown** — any Lexicon Markdown document renders correctly in any Markdown renderer (GitHub, Obsidian, Pandoc, VS Code, etc.);
2. **Legally structured** — supporting the hierarchical clause structure, defined terms, cross-references, schedule items, and meta properties that legal contracts require; and
3. **Tooling-friendly** — a processor can extract structured data, auto-resolve cross-references, generate glossaries, validate defined terms, and render to `.docx` or `.pdf`.

Without a processor, a Lexicon Markdown document reads and renders as a well-structured Markdown document. With a processor, it gains auto-numbering, cross-reference resolution, defined term validation, and schedule generation.

## 2. Document Meta Properties

Every Lexicon Markdown document begins with a YAML front-matter block. This block contains structured metadata about the contract that does not form part of the substantive terms but is used by a processor for rendering cover pages, headers, footers, party blocks, and exhibit pages.

### 2.1. Syntax

The front-matter block is enclosed by `---` delimiters and must appear at the very beginning of the document, before any other content.

```yaml
---
title: Deed of Release
type: Deed
date: 2017-11-05
ref: "VL:RP:20161021"
author: Richard Prangell (Viridian Lawyers)
status: draft
version: 2
parties:
  - name: Elliot Anderson
    specifier: 123 Forth St, New York
    role: Employee
    entity_type: us-individual
  - name: ECorp Limited
    specifier: ACN 123 456 789
    role: Employer
    entity_type: au-company
exhibits:
  - title: Office Diagram
  - title: Employment Obligations
schedule:
  - title: Schedule
---
```

### 2.2. Fields

#### 2.2.1. `title` (required)

The plain-language title of the document. This will appear as the document title in any rendering, and as the content of the `# Heading` in the output.

```yaml
title: Contract for the Sale of Business
```

#### 2.2.2. `date` (required)

The effective date of the contract. Must be in `YYYY-MM-DD` format. A processor must validate this format.

```yaml
date: 2026-01-15
```

#### 2.2.3. `ref` (optional)

The drafter's reference number. Multiple references are separated by commas within the string.

```yaml
ref: "VL:RP:20161021, ASH/JN/123456"
```

#### 2.2.4. `author` (optional)

The drafter or authoring firm. Free text.

```yaml
author: Richard Prangell (Viridian Lawyers)
```

#### 2.2.5. `status` (optional)

The current status of the document. Must be one of the following values:

| Value      | Description                                                              |
| ---------- | ------------------------------------------------------------------------ |
| `draft`    | The document is a working draft, not yet agreed by the parties.          |
| `final`    | The document has been agreed but not yet executed.                       |
| `executed` | The document has been executed (signed) by the parties.                  |

If omitted, no status is assumed. A processor may use this field to render a watermark (e.g., "DRAFT") or status indicator on the output document.

```yaml
status: draft
```

#### 2.2.6. `type` (optional)

The type of legal document, used to refer to the document throughout the contract body (e.g., in a parties preamble or recitals). Defaults to `"Agreement"` if not specified.

The `type` is automatically treated as a defined term. A processor should include it alongside party roles in the list of automatic definitions.

Common values: `"Agreement"`, `"Deed"`, `"NDA"`, `"Addendum"`, `"Contract"`.

```yaml
type: Deed
```

#### 2.2.7. `version` (optional)

The version identifier of the document, expressed as a number or string. This tracks iterations during negotiation and review. A processor may display the version alongside the document metadata.

```yaml
version: 2
version: 1.4
version: "2.1.0"
```

#### 2.2.8. `parties` (required)

A list of parties to the contract. Each party has the following sub-fields:

| Sub-field     | Required | Description                                                        |
| ------------- | -------- | ------------------------------------------------------------------ |
| `name`        | Yes      | The legal name of the party.                                       |
| `specifier`   | No       | Identifying detail (address, ACN, ABN, registration number, etc.). |
| `role`        | Yes      | The drafting reference used throughout the contract (e.g., "Employer", "Contractor"). |
| `entity_type` | No       | A compound `{jurisdiction}-{type}` string identifying the kind of legal entity (e.g., `au-company`, `uk-individual`). Used by a processor to select the appropriate signature block template. |

```yaml
parties:
  - name: Elliot Anderson
    specifier: 123 Forth St, New York
    role: Employee
    entity_type: us-individual
  - name: ECorp Limited
    specifier: ACN 123 456 789
    role: Employer
    entity_type: au-company
```

Party roles are automatically treated as defined terms. A processor should include them in any generated glossary and may validate that the role appears in the contract body.

#### 2.2.9. `exhibits` (optional)

A list of external documents to be exhibited (attached for reference) to the contract. Each entry is an object with a `title` field and an optional `path` field. Exhibits are pre-existing documents included for reference that do not contain legal terms — for example, a property map exhibited to a lease, or a technical diagram.

| Sub-field | Required | Description |
|-----------|----------|-------------|
| `title`   | Yes      | The title of the exhibit, rendered as a heading for the exhibit section. |
| `path`    | No       | Path or URL to an image (PNG, JPG) or PDF file to import into the output. Relative paths are resolved against the input document's directory. HTTP and HTTPS URLs are fetched at processing time. |

```yaml
exhibits:
  - title: Plumbing Diagram
    path: ./plumbing-diagram.png
  - title: Site Plan
    path: ./plans/site-plan.pdf
  - title: Technical Specifications
    path: https://example.com/specs/tech-spec.pdf
  - title: Floor Plan
```

When `path` is provided, a processor imports the file and embeds it in the output:
- **Images** (PNG, JPG) are embedded in the exhibit section, scaled appropriately for the output format while preserving aspect ratio.
- **PDF** files are embedded directly or converted to images, depending on the output format.

When `path` is omitted, a processor generates a placeholder exhibit section with the exhibit number and title (e.g., "EXHIBIT 1 - Floor Plan").

#### 2.2.10. `schedule` (optional)

A list of schedules to be generated from defined terms in the contract body. Each entry is an object with a `title` field. See section 6 for full details on how schedule items are identified and rendered.

| Sub-field | Required | Description |
|-----------|----------|-------------|
| `title`   | Yes      | The title of the schedule, used as the section heading and matched in defined term phrases. |

```yaml
schedule:
  - title: Schedule
```

Multiple schedules:

```yaml
schedule:
  - title: Schedule of Particulars
  - title: Payment Schedule
```

If omitted, no schedules are generated.

### 2.3. Markdown Compatibility

YAML front-matter is widely supported. Renderers that do not support it will display the raw YAML at the top of the document, which is acceptable for plain-text readability. Renderers that do support it (Pandoc, Obsidian, Hugo, GitHub Pages) will parse it as metadata.

## 3. Document Structure

### 3.1. Document Title

The document title is specified in the `title` field of the front-matter (see section 2.2.1). It is not repeated as a `# Heading` in the document body. A processor will render the title from the front-matter.

If the document is being used without a processor, the author may optionally include a `# Title` heading at the top of the body for readability, but a processor should ignore it in favour of the front-matter `title`.

### 3.2. Top-Level Clauses

A top-level clause is represented by an ordered list item with a `##` heading:

```markdown
1. ## Definitions
```

This produces a numbered top-level clause with a heading that will appear in any table of contents.

### 3.3. Sub-clauses

Sub-clauses are represented by indented ordered list items. Each level of nesting uses 4 spaces of indentation:

```markdown
1. ## Termination

    1. A party may terminate this Agreement with immediate effect by giving written notice to the other if:

        1. the other party breaches any provision of this Agreement and fails to remedy the breach within 30 days after receiving notice requiring it to do so; or

        2. the other party undergoes an **Insolvency Event**.
```

The hierarchy is:

| Level          | Syntax                           | Rendered as (by processor) |
| -------------- | -------------------------------- | -------------------------- |
| Top-level      | `1. ## Heading`                  | `1. Heading`               |
| Clause         | `    1. Text`                    | `1.1` or `1.1.`           |
| Sub-clause     | `        1. Text`                | `(a)`                      |
| Sub-sub-clause | `            1. Text`            | `(i)`                      |
| Paragraph      | `                1. Text`        | `(A)`                      |
| Sub-paragraph  | `                    1. Text`    | `(I)`                      |

Without a processor, standard Markdown renderers will display these as nested numbered lists (`1.`, `1.`, `1.`, `1.`). With a processor (e.g., Pandoc with a custom filter, or a dedicated tool), the numbering is transformed to the legal convention shown in the table above.

### 3.4. Multiple Paragraphs Within a Clause

A clause may contain multiple paragraphs. Subsequent paragraphs are indented to the same level as their parent clause and separated by a blank line:

```markdown
1. ## Termination

    1. A party may terminate this Agreement with immediate effect.

        Nothing in this clause shall allow a Party to act unreasonably.

    2. At the end of this Agreement, each party must return all **Confidential Information**.
```

### 3.5. Blockquotes

Blockquotes are used for material that does not form a structural part of the clause hierarchy — formulae, examples, or quoted text:

```markdown
1. ## Termination Fee

    1. The Lessee will pay a termination fee calculated as follows:

        > (2 × A) − C
        > Where "A" has the value of 1 month's rent
        > Where "C" has the value of the Rental Bond
```

### 3.6. Sub-headings Within a Clause

Where a clause requires a sub-heading (e.g., for grouped terms within a miscellaneous clause), use a `###` heading inside the list item:

```markdown
14. ## General Terms

    1. ### Governing Law and Jurisdiction

        1. This Addendum is governed by the laws of New South Wales.

    2. ### Severance

        1. If any provision of this Addendum is invalid, it will be severed.
```

### 3.7. Superscript

Superscript text is marked with `^` delimiters:

```markdown
^2^
```

This renders as superscript in the output document. Useful for footnote markers, ordinals, or mathematical notation.

### 3.8. Prose Sections

Some parts of a contract are not structured as numbered clauses (e.g., addendum content, signature blocks). These are written as standard Markdown paragraphs, headings, lists, and tables outside of the numbered outline structure.

### 3.9. Recitals / Background

Many contracts include a "recitals" or "background" section before the operative clauses, providing context for why the agreement exists.

A recitals section is introduced with a level-1 heading of either `# Recitals` or `# Background` (case-insensitive). When a recitals section is present, the operative clauses must also be preceded by a level-1 heading (e.g. `# Operative Provisions`, `# Terms and Conditions`, or any other descriptive heading). This ensures the document remains readable in plain Markdown viewers without a processor.

```markdown
# Background

1. The parties have entered into a principal agreement
   for the provision of services (the **Principal Agreement**).

2. Party A wishes to engage Party B to process certain
   data on its behalf in connection with the **Principal Agreement**.

# Operative Provisions

1. ## Definitions

    1. **Term** means ...
```

#### 3.9.1. Content

The recitals section supports the same content types as the document body: ordered lists (which become lettered recital items), prose paragraphs, and tables.

#### 3.9.2. Numbering

Ordered list items in the recitals section use the same numbering hierarchy as body clauses: `1.`, `1.1`, `(a)`, `(i)`, `(A)`, `(I)`. Numbering restarts at 1 within the recitals section (independent of the body clause numbering).

#### 3.9.3. Cross-References and Defined Terms

Recitals support `{#id}` anchors and cross-references. A cross-reference to a recital resolves to "Recital 1", "Recital 1.1", etc. Bold terms in recitals are validated in the same way as the document body.

#### 3.9.4. Rules

- Only one recitals section per document.
- The recitals heading must appear before any operative clauses.
- When a recitals section is present, a body heading is required before the operative clauses. A processor should warn if this is missing.
- Both the recitals heading and the body heading appear in the table of contents.

## 4. Defined Terms

### 4.1. Overview

Defined terms are marked with **bold** formatting (`**Term**`) at the point of definition only. After a term is defined, all subsequent references to it appear as ordinary text. Bold must not be used for emphasis. For emphasis, use *italics*.

### 4.2. Formal Definitions

A formal definition is a bold term followed by the word "means" (or "has the meaning"):

```markdown
1. **Applicable Laws** means European Union or Member State laws with respect to any
   Merchant Personal Data in respect of which any Merchant Group Member is subject
   to EU Data Protection Laws.
```

The pattern is:

```
**Term** means ...
```

or:

```
**Term** has the meaning given to it in clause X.
```

A processor identifies formal definitions by matching this pattern. The bold text is the canonical defined term.

### 4.3. Inline Definitions

A term can be defined contextually within a clause, typically using a parenthetical:

```markdown
1. The Employer agrees to pay (the "**Payment**") to the Employee.
```

or:

```markdown
1. The parties to this agreement ("**Parties**") agree as follows.
```

The pattern is:

```
("**Term**")
```

or:

```
(the "**Term**")
```

A processor identifies inline definitions by matching bold text within parentheses and quotation marks.

### 4.4. References to Defined Terms

After a term is defined, all subsequent uses appear as ordinary (unformatted) text:

```markdown
1. The Employee must return all Confidential Information to the Employer.
```

A processor should:

1. collect all defined terms from bold text (formal definitions, inline definitions, and party roles);
2. scan the document text for occurrences of each defined term; and
3. warn if a defined term is never used in the document.

### 4.5. Party Roles as Defined Terms

Party roles declared in the front-matter `parties[].role` field are automatically defined terms. They do not need a separate formal definition in the body, though the author may include one. When used in the body, they appear as ordinary text:

```markdown
1. The Employer agrees to the following terms.
```

### 4.6. Generated Glossary

A processor may generate a definitions schedule or glossary by collecting all defined terms, their definitions, and their locations. This is optional and at the processor's discretion.

## 5. Cross-References

### 5.1. Overview

Cross-references allow one clause to refer to another by a stable identifier, rather than by a hard-coded clause number. This means clause numbers can change (due to insertions, deletions, or reordering) without breaking references.

### 5.2. Anchors

An anchor marks a location in the document that can be referenced from elsewhere.

#### 5.2.1. Heading-Level Anchors

Top-level clause headings receive anchors using Pandoc's attribute syntax, appended to the heading:

```markdown
1. ## Termination {#termination}
```

In standard Markdown renderers, `{#termination}` may render as literal text (harmless). In Pandoc and many other processors, it creates an HTML `id` attribute on the heading.

#### 5.2.2. Clause-Level Anchors

To anchor a specific sub-clause (not a heading), append the attribute syntax to the end of the clause text:

```markdown
    2. The Employer shall pay the Payment within the Timeframe. {#payment-timeframe}
```

In standard Markdown, this renders as literal text at the end of the clause. A processor strips the anchor from the rendered output and records it for cross-reference resolution.

### 5.3. References

A reference to an anchored clause uses standard Markdown link syntax:

```markdown
Notwithstanding the provisions of [clause 2.2](#payment-timeframe), payment is
subject to the following conditions.
```

The format is:

```
[display text](#anchor-id)
```

Where:

- `display text` is what appears in the rendered document (e.g., `clause 2.2`, `clause 5`, `this clause`).
- `anchor-id` is the identifier from the anchor.

#### 5.3.1. Without a Processor

In plain Markdown, these render as clickable hyperlinks (linking to `#payment-timeframe`). The display text is static and must be manually maintained. This is the same as the current system's behaviour.

#### 5.3.2. With a Processor

A processor:

1. resolves each anchor to its clause number (e.g., `#payment-timeframe` → `clause 1.2`);
2. replaces the display text with the resolved reference (e.g., `[clause 2.2](#payment-timeframe)` → `clause 1.2`);
3. warns if any reference points to a non-existent anchor.

The display text in the source serves as a fallback and a hint to the author. The processor overwrites it with the correct value.

### 5.4. Self-Referential Anchors

A clause may reference itself:

```markdown
    3. For the avoidance of doubt, nothing in [this clause](#avoidance-of-doubt) prevents
       the Consultant from working with Pearson Hardman. {#avoidance-of-doubt}
```

## 6. Schedules

### 6.1. Overview

A schedule is a section appended to the contract that lists variable terms to be completed at the time of execution. Schedule items are identified automatically from defined terms whose definitions reference the schedule by name.

This approach uses standard Markdown syntax (bold defined terms with ordinary prose), making schedules fully readable and cross-compilable without any special syntax.

### 6.2. Declaration

Schedules are declared in the front-matter `schedule` field. Each entry is an object with a `title` field:

```yaml
schedule:
  - title: Schedule
```

Multiple schedules are supported:

```yaml
schedule:
  - title: Schedule of Particulars
  - title: Payment Schedule
```

If `schedule` is omitted or empty, no schedule section is generated.

### 6.3. Schedule Items

A schedule item is a defined term whose definition text contains a phrase that references a schedule by its title. For example:

```markdown
1. **Objection Period** has the meaning given by the Schedule.

2. **Rent** is set out in the Schedule of Particulars.

3. **Payment Date** has the meaning specified in the Payment Schedule.
```

A processor identifies schedule items by matching the following phrases (case-insensitive) in the text following a bold defined term, where `{title}` is the title of a declared schedule:

| Phrase |
|--------|
| given by the {title} |
| set out in the {title} |
| specified in the {title} |
| described in the {title} |
| defined in the {title} |
| provided in the {title} |
| contained in the {title} |
| stated in the {title} |
| referred to in the {title} |
| as per the {title} |
| in accordance with the {title} |
| pursuant to the {title} |
| detailed in the {title} |

### 6.4. Behaviour

#### 6.4.1. Without a Processor

Schedule items are ordinary defined terms in standard Markdown. The text `**Objection Period** has the meaning given by the Schedule.` renders as bold text followed by plain prose — perfectly readable without any processor.

#### 6.4.2. With a Processor

A processor:

1. parses the `schedule` field from the front-matter to identify declared schedules;
2. scans all defined terms (bold text) in the document for phrases matching a declared schedule title;
3. collects matching terms as schedule items, associated with the referenced schedule;
4. generates a schedule section for each declared schedule, listing the collected terms with blank spaces for completion; and
5. warns if a declared schedule has no referencing terms, or if a term references a schedule title not declared in the front-matter.

### 6.5. Rendering

Each schedule renders as a separate section with:

- the schedule title as a heading; and
- a table listing each schedule item (the defined term name) with a blank space for completion at execution.

Items appear in document order by default. A processor may support alternative orderings (e.g., alphabetical) via configuration.

## 7. Tables

Standard Markdown tables are used where a contract requires tabular data:

```markdown
| Item        | Rate         | Frequency |
| ----------- | ------------ | --------- |
| Rent        | $1,000/month | Monthly   |
| Maintenance | $200/quarter | Quarterly |
```

A processor renders these as formatted tables in the output document.

> **TODO**: Define the semantics of tables within the clause hierarchy. Open questions:
> - Can tables appear at any nesting level within a clause?
> - Should table cells support inline Lexicon features (defined terms, cross-references)?
> - Are tables purely presentational, or can they carry structural meaning (e.g., a table of defined terms)?
> - How should tables interact with clause numbering (do they interrupt the flow, or sit inline)?

## 7A. Code Blocks

> **TODO**: Determine whether code blocks (fenced with `` ``` `` or indented by 4 spaces) have a role in Lexicon documents. Potential use cases:
> - Displaying formulae or algorithms in technical contracts
> - Embedding machine-readable identifiers or code snippets in technology agreements
> - Preserving whitespace-sensitive content (e.g., specific formatting requirements)
>
> Open questions:
> - Should code blocks be supported as a first-class construct, or simply passed through as standard Markdown?
> - How should a processor render code blocks in output formats (e.g., monospaced box in DOCX)?
> - Is there any interaction with clause numbering or defined terms within code blocks?

## 7B. Blockquotes (Extended)

> **TODO**: Clarify the role of blockquotes beyond the current definition in section 3.5. Open questions:
> - Section 3.5 defines blockquotes for formulae, examples, and quoted text. Should this be expanded?
> - Should blockquotes support nested Lexicon features (defined terms, cross-references)?
> - Are there distinct semantic types of blockquotes (formula vs. quotation vs. example) that a processor should distinguish?
> - How should blockquotes render in different output formats (indented? boxed? italicised?)

## 8. Addenda and Exhibits

Lexicon distinguishes between three types of attachments to a contract:

| Type | Purpose | Where declared | Content |
|------|---------|----------------|---------|
| **Schedule** | Variable terms for completion at execution (see section 6) | Front-matter `schedule` field + defined term phrases | Auto-generated schedule section with blanks |
| **Addendum** | Substantive attached sections that supplement the body | `# ADDENDUM` headings in the document body | Free-form Markdown |
| **Exhibit** | Pre-existing external documents included for reference | Front-matter `exhibits` field | Embedded files or placeholder sections |

### 8.1. Addenda

An addendum is an attached part of the contract that supplements but does not form part of the main body. Addenda may contain substantive terms, amend provisions in the body, or provide additional detail. If an addendum is removed, the contract body remains fully functional.

#### 8.1.1. Syntax

Addendum content appears after the main contract body under a top-level heading beginning with `ADDENDUM` (case-insensitive):

```markdown
# ADDENDUM - Details of Processing

This addendum includes details of processing...
```

The heading may optionally include a number (`# ADDENDUM 1 - Title`), but a processor auto-numbers addenda sequentially regardless of any number in the source heading. The title text is extracted from after the dash separator (any of `-`, `–`, `—`).

Any top-level heading (`#`) that does not begin with `ADDENDUM` will generate a warning. Only `ADDENDUM` headings are parsed as addendum content.

#### 8.1.2. Content

Addendum content is free-form Markdown (paragraphs, lists, tables, headings). It may include numbered clause lists, but does not follow the numbered clause structure of the main body unless appropriate.

### 8.2. Exhibits

An exhibit is a pre-existing document included for reference that does not contain legal terms — for example, a property map, a technical diagram, or an organisational chart.

#### 8.2.1. Declaration

Exhibits are declared in the front-matter `exhibits` field (see section 2.2.9). Each entry has a `title` field and an optional `path` field.

#### 8.2.2. Rendering

When an exhibit has a `path`, a processor imports the referenced file and embeds it in the output:
- **Images** (PNG, JPG) are embedded in the exhibit section, scaled appropriately for the output format while preserving the original aspect ratio.
- **PDF** files are embedded directly or converted to images, depending on the output format.

When `path` is omitted, a processor generates a placeholder exhibit section with the exhibit number and title (e.g., "EXHIBIT 1 - Property Map"). In paginated output, the physical document can then be inserted manually when printing or assembling the final contract.

Relative paths are resolved against the directory containing the input Markdown file. HTTP and HTTPS URLs are fetched at processing time. Supported formats are PNG, JPG/JPEG, and PDF.

## 9. Complete Example

```markdown
---
title: Deed of Release
type: Deed
date: 2017-11-05
ref: "VL:RP:20161021"
author: Richard Prangell (Viridian Lawyers)
status: final
version: 3
parties:
  - name: Elliot Anderson
    specifier: 123 Forth St, New York
    role: Employee
    entity_type: us-individual
  - name: ECorp Limited
    specifier: ACN 123 456 789
    role: Employer
    entity_type: au-company
exhibits: []
schedule:
  - title: Schedule
---

1. ## Definitions {#definitions}

    1. **Claim** means any and all claims, investigations, complaints, enquiries, demands, suits, causes of action, damages, debts, costs, proceedings, whether at law or in equity or under any statute (except workers compensation legislation).

    2. **Payment** has the meaning given by the Schedule.

    3. **Person** means a natural person, company, partnership, association, joint venture, or any other business or any other organisation, of any description.

    4. **Timeframe** has the meaning given by the Schedule.

2. ## Employer's Obligations {#employer-obligations}

    1. The Employer, without admission of any liability whatsoever, agrees to pay to the Employee the Payment.

    2. The Employer shall pay the Payment to the Employee, within the Timeframe of an executed counterpart of this Deed being received by the Employer, from the Employee. {#payment-timeframe}

3. ## Employee's Obligations {#employee-obligations}

    1. The Employee releases, discharges the Employer against all Claims which the Employee has, or which, but for this Deed, could, would or might at any time have or have had against the Employer.

    2. Notwithstanding the provisions of [clause 2.2](#payment-timeframe), payment of the Payment is subject to the Employee:

        1. completing a handover of the Employee's duties in good faith; and

        2. the Employee returning all Employer property, information, information technology access details and passwords and intellectual property in the Employee's possession, custody or control within 7 days.

4. ## Deed to Remain Confidential {#confidentiality}

    1. The parties to this Deed agree that they will not disclose the existence of this Deed or the terms of this Deed, to any person, directly or indirectly, without the other party's prior written approval, except:

        1. to their legal or financial advisers, in the course of obtaining advice;

        2. as may be required by law; or

        3. to treating medical practitioners, in the course of obtaining treatment.

    2. The parties acknowledge that their obligations under [this clause](#confidentiality) are on-going.

5. ## Warranties {#warranties}

    1. The Employee warrants and agrees that:

        1. the Employee remains under an ongoing duty not to use or disclose any confidential information belonging to the Employer for as long as that information is not available in the public domain (other than by breach of the terms of this Deed);

        2. prior to entering into this Deed, the Employee was given a reasonable opportunity to obtain any advice (legal, financial or otherwise) about the Deed and the obligations contained in it;

        3. the Employee has had sufficient time to consider the terms of this Deed, its implications and any advice given to the Employee in respect of it;

        4. the Employee has acted honestly and in good faith and has disclosed all matters to the Employer that may affect the terms of this Deed and the Employer's discretion to enter into this Deed;

        5. the Employee has read this Deed and agrees that its terms are fair and reasonable in the circumstances;

        6. the Employee has entered into this Deed voluntarily and of the Employee's own free will, without duress, coercion, undue influence or pressure from either the Employer or any other Person; and

        7. the Employer is relying upon these warranties in executing this Deed.

6. ## Miscellaneous {#miscellaneous}

    1. This Deed binds each of the parties and anyone who claims through a party.

    2. This Deed may be pleaded as a full and complete defence by the Employer, including as a bar, to any Claims commenced, continued or taken by or on behalf of the Employee in connection with any of the matters referred to in this Deed.

    3. The provisions of this Deed contain the entire understanding and agreement between the parties as to the subject matter of this Deed.

    4. All previous negotiations, understandings, representations, warranties, memoranda or commitments in relation to, or in any way affecting, the subject matter of this Deed are merged in and superseded by this Deed, and are of no force or effect whatever and no party will be liable to any other party in respect of those matters.

    5. If any provision of this Deed at any time is or becomes void, voidable or unenforceable, it will be severable and the remaining provisions of this Deed shall continue to be in full force and effect.

    6. This Deed is governed by and is to be construed in accordance with the laws in force in NSW.

    7. The parties will bear their own costs in connection with the preparation and execution of this Deed.

    8. This Deed may consist of a number of counterparts and if so, the counterparts taken together constitute one and the same Deed.

7. ## Non-Disparagement {#non-disparagement}

    1. The Employer undertakes not to make any statement or intimations derogatory of the Employee, and agrees not to discredit or disparage the Employee.

    2. The Employee undertakes not to make any statement or intimations derogatory of the Employer, and agrees not to discredit or disparage the Employer.
```

## 10. Processor Capabilities

A Lexicon Markdown processor should implement the following capabilities:

### 10.1. Parsing

1. Parse YAML front-matter and extract all meta properties.
2. Parse the document structure into a clause tree with depth levels.
3. Identify all anchors (heading-level and clause-level).
4. Identify all cross-references.
5. Identify all defined terms (formal, inline, and party-role).
6. Identify all schedule items by matching defined terms against declared schedule titles and built-in phrases.

### 10.2. Validation

1. Validate that `date` is in `YYYY-MM-DD` format.
2. Validate that all cross-references point to existing anchors.
3. Warn on defined terms that are never used in the document text.
4. Warn if a declared schedule has no referencing terms, or if a defined term references a schedule title not declared in the front-matter.

### 10.3. Transformation

1. Auto-resolve cross-references: replace display text with correct clause numbers.
2. Transform numbering to legal convention (`1.1`, `(a)`, `(i)`).
3. Strip anchor syntax from rendered output.
4. Generate a definitions glossary / schedule.
5. Generate schedule sections from defined terms that reference declared schedules.
6. Render to the target output format (e.g., `.docx`, `.pdf`, `.html`).

### 10.4. Output Formats

A processor may target any output format. Common targets include:

1. **Markdown** — a "resolved" Markdown file with cross-references updated and anchors stripped.
2. **DOCX** — a Word document with legal numbering, formatted parties block, cover page, addenda, and exhibits.
3. **PDF** — a paginated document equivalent to the DOCX output.
4. **HTML** — a web-renderable document with structured sections, embedded images, and interactive navigation.

## 11. Summary of Syntax

| Feature              | Syntax                                          | Markdown Compatible |
| -------------------- | ----------------------------------------------- | ------------------- |
| Meta properties      | YAML front-matter (`---`)                       | Yes (widely supported) |
| Top-level clause     | `1. ## Heading`                                 | Yes |
| Sub-clauses          | Indented ordered lists (4 spaces per level)     | Yes |
| Multiple paragraphs  | Blank line + indented continuation              | Yes |
| Blockquotes          | `>`                                             | Yes |
| Sub-headings         | `### Heading` inside list item                  | Yes |
| Superscript          | `^text^`                                        | Partial (some renderers support it) |
| Formal definition    | `**Term** means ...`                            | Yes (renders as bold) |
| Inline definition    | `("**Term**")`                                  | Yes (renders as bold) |
| Term reference       | Plain text (no markup)                          | Yes |
| Heading anchor       | `## Heading {#id}`                              | Partial (Pandoc yes, others show literal text) |
| Clause anchor        | `Text {#id}`                                    | Partial (shows literal text without processor) |
| Cross-reference      | `[clause X](#id)`                               | Yes (renders as link) |
| Schedule item        | `**Term** has the meaning given by the Schedule.` | Yes (renders as bold + prose) |
| Schedule declaration | Front-matter `schedule` field                   | Yes |
| Tables               | Standard Markdown tables                        | Yes |
| Exhibit declaration  | Front-matter `exhibits` field (with optional `path`) | Yes |
| Addendum content     | `# ADDENDUM` headings after main body           | Yes |
