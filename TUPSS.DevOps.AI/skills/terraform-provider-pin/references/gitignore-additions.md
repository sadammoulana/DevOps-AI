# `.gitignore` Additions — Consumer Repos

Source of truth: `terraform-gitignore-standards.md`. This is a trimmed, skill-facing excerpt — check the full doc if anything here is ambiguous.

## Required lines (consumer repos: `TUPSS.*.Terraform`)

```gitignore
# Terraform
**/.terraform/*
*.tfstate
*.tfstate.*
.terraform.tfstate.lock.info
crash.log
crash.*.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json
.terraformrc
terraform.rc

modules/
modules-test/
```

## Dedup check before dropping in the block

Existing `.gitignore` files across TUPSS repos vary — most are the default Visual Studio template with little to no Terraform-specific content, but don't assume that. The end result must always be **exactly** the block above, present once. Before writing it in:

1. Read the existing `.gitignore` in full.
2. Identify any existing Terraform-related lines — a `# Terraform` section, or scattered individual lines matching/overlapping any entry in the block above (exact matches or broader patterns that cover part of it, e.g. an existing `*.tfstate*`).
3. **Remove all of those existing Terraform-related lines**, then insert the block above as a single, exact, contiguous unit. Don't merge/reconcile line-by-line and don't leave the old lines alongside the new block — that produces duplicates or near-duplicate variants. The block is authoritative; anything Terraform-related that predates it gets replaced, not preserved.
4. Leave everything else in the `.gitignore` (non-Terraform entries) untouched.

## Do NOT add

- **`.terraform.lock.hcl`** — must be committed, never ignored. This is the entire point of the migration; if you see it already ignored, that's a bug in the existing `.gitignore` and must be fixed (remove the ignore entry), not preserved.
- **`*.tfvars`** — TUPSS consumer repos track `.tfvars` per environment (`dev.tfvars`, `prod.tfvars`, etc.) normally. They contain no secrets (secrets are injected via pipeline token replacement). This is a deliberate deviation from the generic Terraform `.gitignore` template, not an oversight — do not "fix" it by adding `*.tfvars`.
