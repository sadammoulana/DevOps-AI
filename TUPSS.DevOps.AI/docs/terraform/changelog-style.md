# Terraform Module Changelog Style Guide

> Establish a minimal, fact-based changelog format for TUPSS Terraform modules.

---

## Contents

- [Core Principles](#core-principles)
- [Format Template](#format-template)
- [Style Rules](#style-rules)
- [What NOT to Include](#what-not-to-include)
- [Approval Checklist for PR Review](#approval-checklist-for-pr-review)
- [Reference: FunctionApp Changelog](#reference-functionapp-changelog)
- [Questions to Ask When Writing](#questions-to-ask-when-writing)

---

Changelogs are **historical records**, not documentation or marketing. Every entry should be a verifiable fact about what changed in the codebase. Strip opinion, narrative, and explanation. If consumers need guidance on *how* to use the change, that goes in README.md or migration notes—not in the changelog entry itself.

This format follows [Keep a Changelog](https://keepachangelog.com), the industry standard, with two deliberate deviations: we use dates instead of semantic versioning (an alternative the standard explicitly allows), and we don't track a `Fixed` category—bugfixes and cosmetic changes don't get changelog entries at all (see Principle 5). We also skip `Deprecated`: we don't do soft deprecation, an input or resource is either present or removed.

Each module maintains its own `CHANGELOG.md` at the root of the module directory (e.g., `modules/FunctionApp/CHANGELOG.md`). Entries are listed newest first, with the most recent date at the top of the file.

---

## Core Principles

### 1. Date-Based Versioning (Not Semantic Versioning)

We use dates to organize changes, not semantic version numbers. Each PR is a release, and every change in that PR gets one date entry.

- **Format:** `## Month Day, Year` (e.g., `## July 29, 2026`)
- **Use the tag/merge date**, not individual commit timestamps.
- If commits span multiple days within the same PR, they still land under one date entry. If a colleague requests an additional change mid-PR, add it to the existing entry rather than creating a new one.

**RIGHT:**
```
## July 28, 2026
### Added
- `feature_a` input
- `feature_b` input
```

**WRONG:**
```
## July 24, 2026
### Added
- `feature_a` input

## July 28, 2026
### Added
- `feature_b` input
```

---

### 2. Minimal, Factual Descriptions

Each entry must state **what changed**, not **why** or **how to use it**.

**RIGHT:**
```
### Added
- `slot_dotnet_version_override` — optional `map(string)` that sets a different `dotnet_version` per named slot
```

Format:
- **Name:** backtick-quoted variable/attribute name
- **Type:** Terraform type signature (e.g., `string`, `map(string)`, `list(object({...}))`)
- **What it does:** One concise sentence describing the highest-level behavior
- Do not include implementation details, constraints, or usage instructions—those belong in README

**WRONG:**
```
### Added
- `slot_dotnet_version_override` — optional `map(string)` that sets a different `dotnet_version` per named slot. Keys must match values in `slot_name`. Slots not listed fall back to `var.dotnet_version`.
```
(Implementation details that belong in README, not changelog. Bloated.)

---

### 3. Standard Sections

Use these headings consistently:

- `### Added` — new inputs, outputs, or resources
- `### Changed` — modified behavior, logic, or defaults
- `### Removed` — deleted inputs, outputs, or resources

Only include sections that have content. A changelog entry can have just `Added`, just `Changed`, or a combination.

---

### 4. Breaking Changes

Flag breaking changes in the date line:

```
## June 9, 2026 - BREAKING CHANGE
```

For breaking changes:
- Provide a one-line summary of what broke
- Explain the old model (what was removed)
- Explain the new model (what replaces it)
- Include a `### Migration Notes` section with step-by-step instructions
- Use `### Removed` to list deprecated inputs/outputs

See the reference example below for a full breaking change writeup.

---

### 5. Only Functional Changes Get an Entry

The changelog records changes to module functionality—new inputs, changed behavior, removed resources. It is not a commit log.

- **Skip the changelog** for bugfixes with no functional/interface change, cosmetic changes (formatting, comments, variable renames with no behavior impact), and internal refactors that don't alter what the module does.
- **Add an entry** whenever an input, output, resource, or default behavior changes in a way a consumer could observe.

---

## Format Template

### For simple changes:

```markdown
## [Month Day, Year]

### Added
- `input_name` — description of what it does

### Changed
- `existing_input` — description of the behavior change

### Removed
- `old_input` — replaced by `new_input`
```

### For breaking changes:

```markdown
## [Month Day, Year] - BREAKING CHANGE

### [One-line summary of what broke]

[Explanation of old model and new model]

### Added
- [new inputs/outputs]

### Changed
- [modified behavior]

### Removed
- [deleted inputs]
- [deleted resources]

### Migration Notes
[Step-by-step migration instructions]
```

---

## Style Rules

### Backticks and Code Formatting

- Wrap Terraform identifiers in backticks: `` `input_name` ``, `` `azurerm_resource_type` ``
- Wrap code blocks in triple backticks with language tag (hcl, bash, etc.)
- Do not backtick concepts or product names (e.g., "Log Analytics Workspace" not `` `Log Analytics Workspace` ``)

### Descriptions

- **One sentence** per entry
- Start with the identifier and type
- Use em dash (`—`) to separate name from description
- Be specific: "sets a different `dotnet_version` per named slot" is better than "allows per-slot customization"

### Removed Entries

When removing inputs, specify what replaces them:

```markdown
### Removed
- `allowed_inbound_ips` — replaced by explicit `ip_restrictions` rule objects
```

### Migration Notes

Only include for breaking changes. Use:
- Ordered steps (`1.`, `2.`, etc.) or bulleted lists
- Code blocks with exact commands or configuration
- `<details>` tags to hide long sections if needed
- Cross-reference to README.md for additional examples

---

## What NOT to Include

❌ **Internal implementation details:**  
"Refactored the local block" — consumers don't care about your code structure.

❌ **Explanations of why:**  
"We removed this because nobody was using it" — keep it factual.

❌ **Usage instructions:**  
"To use the new feature, do X" — that belongs in README.md or examples.

❌ **Narrative or storytelling:**  
"We've been working on this for weeks..." — stick to what changed.

❌ **Vague language:**  
"Improved handling" — be specific. What behavior changed?

---

## Approval Checklist for PR Review

When reviewing a changelog entry:

- [ ] Is it grouped under the correct release date (or should it be moved to match other changes in this release)?
- [ ] Is the description factual and minimal?
- [ ] Are Terraform identifiers backtick-quoted?
- [ ] Type signature is provided for all inputs/outputs?
- [ ] Are there any usage instructions or explanations that belong in README instead?
- [ ] For breaking changes, are migration notes provided?
- [ ] Are removed items listed with their replacements?
- [ ] Is the entry verifiable by looking at the code diff?

---

## Reference: FunctionApp Changelog

`TUPSS.Terraform/modules/FunctionApp/CHANGELOG.md` contains:

- Simple added input (July 17, 2026): one-liner
- Simple changed entry (July 21, 2026): one-liner for behavior change
- Complex breaking change (June 9, 2026): full model refactor with migration notes

Use this as the gold standard for style and structure.

---

## Questions to Ask When Writing

1. **"What changed in the code?"** → That goes in the changelog.
2. **"Why did we change it?"** → That belongs in a commit message or PR description, not the changelog.
3. **"How do consumers use this?"** → That belongs in README.md or examples.
4. **"Could I verify this by reading the code diff?"** → If yes, you're ready to add it.

---

## Last Updated

July 29, 2026
