# Writing Docs in This Folder

Rules for adding or updating content under `docs/`. The goal is always minimal and focused — faster for a human to read, fewer tokens for an agent to retrieve.

---

## Contents

- [Naming](#naming)
- [Structure](#structure)
- [File format](#file-format)
- [Content expectations](#content-expectations)
- [Adding, updating, deleting](#adding-updating-deleting)
- [Archiving](#archiving)

---

## Naming

- Kebab-case for all folders and files: `provider-version-management.md`, not `ProviderVersionManagement.md`
- Be descriptive over clever: `stage-design.md` over `stages.md`

## Structure

- One topic folder per area (`pipelines/`, `terraform/`, `source-control/`) — keep files flat within a topic folder, don't nest further
- If a new topic doesn't fit an existing folder, add one rather than forcing it into an unrelated folder

## File format

- Every doc starts with a single `# H1` title matching the filename
- Use fenced code blocks with a language tag for all code/config samples
- Use tables for comparisons, option lists, and matrices
- Use backtick inline code for filenames, flags, and values referenced in prose

## Content expectations

- Keep docs focused — one topic per file, nothing else bundled in
- Write for a technical audience — no need to define basic Azure/Git concepts
- Where a constraint isn't self-explanatory, a short *why* saves someone from "fixing" it later — but don't pad an otherwise simple fact with unnecessary explanation
- A stub is acceptable; an inaccurate doc is not — mark unknowns with `> TODO:`

## Adding, updating, deleting

- **Adding:** copy [`doc-template.md`](./doc-template.md), create the file in the right topic folder, PR it like any other change
- **Updating:** edit in place — don't create a new file for the same topic
- **Retiring:** if a doc is outdated but has historical value, archive it rather than deleting it; delete outright only if it has no historical value or was created in error

## Archiving

When a doc is no longer accurate or relevant but might be useful as historical reference, archive it — don't delete it and don't leave it in place going stale.

1. Move the file into [`_archive/`](../_archive/) at the root of `docs/`, keeping the original filename
2. Add this banner as the first thing in the file, above the existing `# H1` title:

   ```markdown
   > **ARCHIVED [Month Day, Year]** — retained for historical reference only. Do not treat as current guidance.
   ```
3. If the content was replaced by something else, link to the replacement in the banner
4. Update any docs that linked to it so they don't point at stale content

Archiving is a deliberate, PR-reviewed action — not a place to dump anything that might be useful someday. If it isn't worth a banner and a date, it's probably just worth deleting.
