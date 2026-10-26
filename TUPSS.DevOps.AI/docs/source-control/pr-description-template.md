# PR Description Template

Structure for PR descriptions on the DevOps team. See [pr-standards.md](./pr-standards.md) for when this is required.

---

## Contents

- [PR Description Template](#pr-description-template)
  - [Contents](#contents)
  - [PR Title](#pr-title)
  - [Required Sections](#required-sections)
  - [Optional Sections](#optional-sections)
  - [Template](#template)

---

## PR Title

Short, imperative, no ticket-number prefix — e.g. `Add terraform-provider-pin skill`, `Fix branch trigger mismatch in hotfix pipeline`. Lead with the verb; the title alone should tell a reviewer what changed without opening the PR.

---

## Required Sections

A PR title (see [PR Title](#pr-title) above) plus these three sections, regardless of type:

- **Summary** — one or two sentences: what changed and why
- **Key Changes** — bullet list of the concrete changes (resources, files, stages, whatever's relevant)
- **Validation** — what was checked or run to confirm correctness (`terraform fmt`/`plan`, tests, PR Preflight review). Checks and tooling only — proof links live in the optional sections below.

---

## Optional Sections

Include only what applies to this PR. Don't pad with "N/A" — leave the section out entirely if it doesn't apply.

**Named evidence sections** — when a PR touches multiple environments, resources, or logical steps, break proof out into one section per unit of work, named after the work itself (not a generic "Proof" bucket). For example: `State Import`, `SmartQueue Resource Group Creation`, `Smart Notification & Notify API`. Each one is just a list of run links and what they validate.

**Recognized optional headers** — reach for these by name before inventing something new:

| Header | When to use |
|---|---|
| `Breaking Changes` | Anything a consumer must change to not break |
| `Resources Created` / `Resources Modified` / `Resources Destroyed` | Terraform — split by operation, destroys carry different risk than creates |
| `State Changes` | terraform state moves or imports involved |
| `Manual Post-Merge Steps` | Anything a human must do in ADO/Azure after merge (ties to the `REMINDER` severity in [azure-devops.instructions.md](../../.github/instructions/azure-devops.instructions.md)) |
| `New Variable Groups / Secrets` | Pipeline requires manual ADO setup before it can run |
| `Rollback Plan` | Non-obvious or risky rollback — how to revert if this goes wrong in prod |
| `Outstanding / Follow-up` | Intentionally incomplete work, with why |
| `Reviewer Focus` | Directing limited reviewer attention — "look closely at X, Y is boilerplate" |

---

## Template

```markdown
Title: Short, imperative summary of the change

## Summary

One or two sentences — what changed and why.

## Key Changes

- Bullet list of concrete changes

## Validation

- What was checked/run to confirm correctness

<!-- Optional sections go here — see above for named evidence sections and recognized headers. Include only what applies; omit entirely otherwise. -->
```
