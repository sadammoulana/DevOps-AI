---
name: terraform-provider-pin
description: One-time migration skill for the azurerm provider pinning rollout. Pins a single TUPSS.*.Terraform consumer repo to an exact azurerm version and generates a committed .terraform.lock.hcl, following the team's provider version management standard. Use when the user asks to "pin" a consumer repo's provider version, run the provider pin migration, or generate a lock file for a Terraform consumer repo.
---

# Terraform Provider Pin

Applies the TUPSS provider version pinning standard to a single Terraform consumer repo (`TUPSS.*.Terraform`). This is a one-time migration skill — it exists to roll out `required_providers` pinning + committed `.terraform.lock.hcl` files across every consumer repo. It is not intended for routine/ongoing provider bumps (a version bump afterward is a one-line diff — see the standard's "Upgrade Procedure" in [docs/terraform/provider-version-management.md](../../../docs/terraform/provider-version-management.md)).

**Scope note:** this skill only ever produces a **staged, uncommitted** change set. It never commits or pushes. The user reviews the diff, writes the commit message, and pushes themselves — the real validation (does `init` resolve to the target version against the real backend, does `plan` show unexpected drift) only happens after a push triggers the pipeline.

**A note on model selection:** this skill has been tested and is known to perform safely with Claude Sonnet 5. This workflow interacts with the shared `TUPSS.Terraform` module source and modifies `.gitignore` and provider pinning across many repos — mistakes here are more consequential than typical Copilot edits. Exercise caution running this skill under `Auto` model selection: it has been observed producing dangerous hallucinations when `gpt-5.3-codex` was selected. If you have discretion over model selection, prefer a model this skill has been validated against.

## Prerequisites — check before doing anything else

1. **`TUPSS.Terraform` must be a sibling folder in the current workspace, and end up on an up-to-date `main`.**
   - Check `git status` first. If the working tree has uncommitted/edited changes, **stop** and tell the user — do not stash. Uncommitted local edits are the only reason to stop here.
   - If the working tree is clean but checked out on a branch other than `main`, just switch to `main` and pull latest — no need to stop or ask, this repo isn't being modified, only read from for the `modules/` copy.
   - Run `git fetch origin` and confirm local `main` is up to date with `origin/main` (pull if behind).
2. **The target consumer repo must exist as a sibling folder in the workspace.** The user names the repo when invoking this skill. If it isn't present, stop and ask them to add it to the workspace.
   - Same rule as `TUPSS.Terraform` above: check `git status` only to see if the working tree is dirty. Don't read or evaluate file contents on whatever branch happens to be checked out first — that branch is irrelevant and about to be switched away from, so there's nothing useful to learn from it.
   - If clean, switch to `main` and pull latest, then proceed — no need to pause or ask, this is routine setup, not a decision point. The Branch step below handles creating the actual pin branch from here.

Do not proceed past this point if either repo has uncommitted/edited changes.

## Branch

The branch name is always exactly **`provider-pin-4.81.0`** — that's the target version for this migration, so there's nothing to fill in. No ticket/ID number is used — this runs once per repo, so collisions aren't a practical concern. If the branch already exists (locally or on `origin`), check it out and resume rather than recreating it — this makes retries after a failed run safe.

Checkout the target consumer repo's `main`, pull latest, then create/checkout `provider-pin-4.81.0`.

## Steps

1. **Prereqs** — see above. Stop if either check fails.
2. **Branch** — checkout/create `provider-pin-4.81.0` off the consumer repo's up-to-date `main`.
3. **Add the pin.** Add or update the `required_providers` block inside the `terraform { }` block in the consumer repo's root `main.tf` with an **exact** version pin (never `~>`, never unbounded). See `references/provider-pin-block.tf` for the exact shape. Only touch the `terraform { }` block — leave any existing `provider "azurerm" { }` block (and its `features { }` contents) untouched.
4. **Copy `modules/` locally (temporary).** Copy the `modules/` directory from the sibling `TUPSS.Terraform` repo into the root of the consumer repo, alongside `main.tf`.
5. **Init without backend:**
   ```powershell
   terraform init -backend=false
   ```
   This resolves providers and writes `.terraform.lock.hcl` without needing Azure credentials or a real backend. If `init` fails (e.g. provider resolution error, module incompatibility with the pinned version), **stop** — do not proceed to validate/cleanup/staging. Leave the temporary `modules/` copy and the `main.tf` pin change in place (nothing is staged yet, so nothing needs undoing) and report the failure output to the user for triage.
6. **Validate now, while the modules copy is still present and accurate:**
   ```powershell
   terraform validate
   ```
   If validation fails, stop and report the failure — do not proceed to cleanup/staging.
7. **Clean up: delete both the temporary `modules/` copy and the `.terraform/` directory** that `init` created (it caches the downloaded provider binary — tens/hundreds of MB per repo — and isn't needed once the lock file is written). Both are gitignored, but must not be left on disk long-term; across dozens of cloned repos that adds up to real, avoidable disk usage. See `references/standards-excerpt.md` (Lock File Policy):
   ```powershell
   Remove-Item -Recurse -Force .terraform
   Remove-Item -Recurse -Force modules
   ```
8. **Update `.gitignore`.** The result must always be the exact block in `references/gitignore-additions.md`. Remove any existing Terraform-related lines first (a `# Terraform` section, or scattered/overlapping individual entries), then insert the block as a single contiguous unit — do not merge line-by-line or leave old Terraform entries alongside the new block. Leave all non-Terraform lines untouched. Never add `.terraform.lock.hcl` or `*.tfvars` to `.gitignore` — see `references/standards-excerpt.md`.
9. **Stage the changes.** Stage exactly: `main.tf` (or wherever the pin was added), `.terraform.lock.hcl`, and `.gitignore` (only if it was modified). Do **not** commit. Do **not** push.
10. **Report back** to the user: confirm the resolved provider version in the lock file matches the target pin, list the staged files, and remind them that the real test (init against the real backend + `plan` review) happens after they push and the pipeline runs.

## References

- [`references/provider-pin-block.tf`](./references/provider-pin-block.tf) — exact `required_providers` block shape for consumer repos
- [`references/gitignore-additions.md`](./references/gitignore-additions.md) — canonical Terraform `.gitignore` lines for consumer repos, with dedup guidance
- [`references/standards-excerpt.md`](./references/standards-excerpt.md) — trimmed excerpt of the team's provider version management standard (pin rule + lock file policy only)
