# Standards Excerpt — Provider Version Management

Source of truth: `terraform-provider-version-management.md`. This excerpt covers only the two rules this skill needs to enforce — see the full doc for the upgrade procedure and enforcement plans (tflint, pipeline gate), which are out of scope for this skill.

## The Rule

| Location | Mechanism | Constraint style |
|---|---|---|
| Consumer / root module (`main.tf`) | `required_providers` block | **Exact pin** — e.g. `version = "4.81.0"` |
| Reusable module (`providers.tf`) | `required_providers` block | **Floor only** — e.g. `version = ">= 4.0"` |

This skill only handles the **consumer repo** side (exact pin). Module-level floor constraints (`providers.tf` across `TUPSS.Terraform/modules/`) are a separate, one-time effort — not part of this skill.

Consumer repos pin exactly. No `~>`, no unbounded ranges. The root module is the only place that controls which resolved provider version actually gets used.

## Lock File Policy

Every consumer repo must commit a `.terraform.lock.hcl` at its root. `required_providers` alone only declares a constraint — it does not guarantee the same resolved version across machines or pipeline runs without a committed lock file.

Consumer repos do not keep a local `modules/` directory normally (it's injected as a pipeline artifact during the build stage), so generating the lock file requires simulating that artifact locally:

1. Copy `TUPSS.Terraform`'s `modules/` directory into the consumer repo root (temporary).
2. Run `terraform init -backend=false` — resolves providers and writes `.terraform.lock.hcl` without needing Azure credentials or a real backend.
3. Delete the temporary `modules/` copy before committing (it's gitignored, but remove it anyway — it doesn't belong on disk long-term).
4. Confirm `.gitignore` does **not** exclude `.terraform.lock.hcl`.

The lock file is the artifact this whole migration exists to produce — never skip committing it, and never let it get ignored.
