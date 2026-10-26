# Terraform Provider Version Management Standard

> Define how provider versions are declared, pinned, and upgraded across TUPSS Terraform consumer repos and reusable modules. This standard applies to every `TUPSS.*.Terraform` consumer repo and every module in `TUPSS.Terraform/modules/`.

---

## Contents

- [The Rule](#the-rule)
- [Lock File Policy](#lock-file-policy)
- [Upgrade Procedure](#upgrade-procedure)
- [Enforcement](#enforcement)

---

## The Rule

| Location | Mechanism | Constraint style |
|---|---|---|
| Consumer / root module (`main.tf`) | `required_providers` block | **Exact pin** — e.g. `version = "4.81.0"` |
| Reusable module (`providers.tf`) | `required_providers` block | **Floor only** — e.g. `version = ">= 4.0"` |

**Consumer repos pin exactly.** No `~>`, no unbounded ranges. The root module is the only place that controls which resolved version actually gets used.

**Reusable modules declare a floor only.** A module states the minimum provider version it was written against. It never sets a ceiling (`~>` or an exact pin) — a ceiling in a module can conflict with the consumer's exact pin and block `terraform init` for no functional reason. The root module's exact pin is what ultimately resolves.

### Consumer repo example

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "4.81.0"
    }
  }
  backend "azurerm" {}
}

provider "azurerm" {
  features {}
}
```

### Module example (`modules/<ModuleName>/providers.tf`)

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 4.0"
    }
  }
}
```

Every module in `TUPSS.Terraform/modules/` must have a `providers.tf` file declaring this floor constraint.

---

## Lock File Policy

Every consumer repo must commit a `.terraform.lock.hcl` at its root. The lock file freezes the exact resolved provider version and package hashes; `required_providers` alone only declares a constraint, it does not guarantee the same resolved version across machines or pipeline runs absent a committed lock file.

Consumer repos do not keep a local `modules/` directory (it is injected as a pipeline artifact during the build stage), so generating or refreshing the lock file requires simulating that artifact locally — use the same clone/copy setup as [module-testing.md — Local Validation](./module-testing.md#local-validation--copy-modules-directly-into-the-consumer-repo) (steps 1–2: clone `TUPSS.Terraform`, copy `modules/` into the consumer repo root), then:

1. From the consumer repo root, run:
   ```powershell
   terraform init -backend=false
   ```
   `-backend=false` skips remote state configuration (which requires Azure credentials and a real storage account) while still fully resolving providers and writing `.terraform.lock.hcl`. Unlike Local Validation, there's no need to run `terraform validate` here — generating the lock file is the goal.
2. Clean up before committing:
   ```powershell
   Remove-Item -Recurse -Force .terraform
   Remove-Item -Recurse -Force modules
   ```
   Leave `.terraform.lock.hcl` in place, same as [module-testing.md — Local Validation](./module-testing.md#local-validation--copy-modules-directly-into-the-consumer-repo) — that's the file you're committing.

`modules/` is a gitignored path (see the `.gitignore` standard), so a local copy won't get accidentally staged — but still remove it once you're done, it doesn't belong on disk long-term. Confirm your `.gitignore` does not exclude `.terraform.lock.hcl`.

---

## Upgrade Procedure

Bumping a provider version is a deliberate, reviewed action — not something that happens implicitly via a version range. This is a new process for the team; follow it in order.

1. **Change the pin** in `main.tf` to the new exact version.
2. **Re-copy `modules/` locally** from `TUPSS.Terraform` (see the steps under [Lock File Policy](#lock-file-policy) above) so `init` has something to resolve against.
3. Run:
   ```powershell
   terraform init -backend=false -upgrade
   ```
   The `-upgrade` flag forces Terraform to re-resolve providers against the new constraint instead of reusing the existing lock file entries.
4. **Review the lock file diff.** Confirm only the expected provider version/hash changed — nothing else should move.
5. **Run `terraform validate` locally** against the new version to catch syntax/schema issues early.
6. **Clean up the temporary `modules/` copy and the `.terraform/` directory.** Both are gitignored so they won't get staged accidentally, but remove them anyway once you're done — `.terraform/` caches the downloaded provider binary (tens/hundreds of MB per repo), and leaving it behind across dozens of cloned repos wastes real disk space over time.
7. **Commit and push** `main.tf` and the updated `.terraform.lock.hcl` together, in the same change.
8. **Trigger a pipeline run** against your working branch to get a `plan`.
9. **Review the plan output** for unexpected changes (attribute renames, removed fields, new required fields) introduced by the provider upgrade.

---

## Enforcement

- **tflint** — `terraform_required_providers` and `terraform_required_version` rules must remain **enabled** in `TUPSS.Terraform/.tflint.hcl` and any consumer-repo tflint config. These are the automated check that this standard isn't silently regressed.
- **Pipeline gate** — a future improvement is a pipeline step that fails the build if a consumer repo is missing `required_providers` or a committed lock file. Not yet implemented; tracked separately.
