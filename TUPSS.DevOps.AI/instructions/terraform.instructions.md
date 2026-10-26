---
description: "TUPSS DevOps team PR review standards for Terraform/IaC."
applyTo: "**/*.tf,**/*.tfvars"
---

# TUPSS PR Review Standards — Terraform / IaC

These standards apply to all Terraform files. When reviewing, propose concrete edits — do not just identify problems.

> Background, rationale, and full procedures live in [docs/terraform/](../../docs/terraform/) — these rules are the enforceable summary, not the whole standard.

---

## Scope Definitions

Two distinct types of Terraform code exist in this codebase. Rules are scoped accordingly.

| Type | Description | Examples |
|---|---|---|
| **Consumer root module** | Deploys real infrastructure. Has a backend, provider declarations, `.tfvars`, and calls reusable modules. | `CAMS.Extensions.AzureFunctions.SMB.Terraform`, `CAMS.Validation.Api.Terraform` |
| **Reusable module** | A composable building block. No backend, no provider declaration, no `.tfvars`. Exposes variables and outputs. | `TUPSS.Terraform/modules/FunctionApp`, `/modules/KeyVault` |

Where a rule applies only to one type, it is marked **[consumer]** or **[module]**. Unmarked rules apply to both.

---

## General (All File Types)

- **No hardcoded secrets or credentials.** Any secret, password, key, token, or connection string must reference a Key Vault, variable group, or environment variable — never inline.
- **No commented-out code in submitted PRs.** Dead code must be deleted before merge. If it's a deliberate placeholder, it must have a `# TODO:` comment explaining why and owned by a named person.
- **Descriptive naming.** Names must communicate intent. Single-letter variables, `temp`, `test123`, `foo`, or similar are blockers.
- **Scope is bounded.** A PR should do one thing. If unrelated files are modified, flag it — the author should split the PR or justify the coupling.

---

## Terraform / IaC

### Formatting and Validation

- All `.tf` files must be formatted with `terraform fmt`. Unformatted code (misaligned `=` signs, inconsistent indentation) is a **BLOCKER**. Run `terraform fmt -recursive` to fix.
- `terraform validate` must pass before a PR is opened. If the diff introduces a resource or module call, the reviewer should confirm validation is implied by the pipeline run or ask the author to confirm.

### Version and Provider Constraints

> Full rationale and procedures: [docs/terraform/provider-version-management.md](../../docs/terraform/provider-version-management.md).

- **[consumer]** Every consumer root module must declare `required_providers` with an **exact pin** on `azurerm` (e.g., `version = "4.81.0"`) — no `~>` and no unbounded ranges. Missing the constraint, or using a range/pessimistic operator instead of an exact pin, is a **BLOCKER**. Example:

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
  ```

- **[consumer]** A consumer root module must have a committed `.terraform.lock.hcl`. A missing lock file is a **BLOCKER** — it means a local `init` was never done and the resolved provider version/hashes are not guaranteed to match across machines or pipeline runs.
- **[module]** Every reusable module must have a `providers.tf` declaring a **floor-only** constraint on `azurerm` (e.g., `version = ">= 4.0"`) — never a ceiling (`~>`) or an exact pin. Missing `providers.tf`, or a module declaring a ceiling/exact pin, is a **BLOCKER**. A ceiling in a module can conflict with the consumer's exact pin and block `terraform init` for no functional reason. Example:

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

- We do not enforce `required_version` (Terraform CLI version) anywhere, consumer or module. The pipeline is the single source of truth for the CLI version; there are no external teams or separate pipelines consuming these repos. Do not flag its absence.

### State and Backend *(consumer root modules only)*

- Consumer root modules use an **empty backend block** (`backend "azurerm" {}`). This is intentional — the backend config is injected at pipeline runtime via `-backend-config`. Do not flag this as incomplete.
- Do not flag `prevent_deletion_if_contains_resources = false` in provider features. This is the team standard to allow resource group teardown in lower environments.
- If a `state-moves.tf` file is present, it must contain comments explaining what state addresses are being moved and why. An uncommented `moved` block with no context is a **SUGGESTION** to document.
- Hardcoded backend storage account names or container names in a non-sandbox root module are a **BLOCKER**. These must be externalized.

### Naming and Structure

- All variable names must use `snake_case`. CamelCase or mixed conventions (e.g., `ResourceGroup_name`) are a **BLOCKER** in new code; flag as a **SUGGESTION** in existing untouched code (pre-existing issue).
- Every variable must have a `description` field. Empty or placeholder descriptions (`"TODO"`, `"placeholder"`) are **BLOCKERS**.
- Variable attributes must follow this order: `description` → `type` → `default` (when present). Any other ordering is a **NITPICK**.
- **[module]** Every variable with a constrained set of valid values must have a `validation` block. Example: a variable named `os_type` that only accepts `"Windows"` or `"Linux"` must validate that. Flag missing validation as a **SUGGESTION**. Validation blocks are not expected in consumer root modules — the module interface is the right place to enforce contracts.
- Every resource block must pass `common_tags` or equivalent tagging variable. Untagged resources are **BLOCKERS**.
- **[module]** Module `outputs` must include a `description`. Missing output descriptions are **BLOCKERS**.
- **[module]** Every module must expose at minimum an `id` output. Missing `id` on a module that creates a named resource is a **SUGGESTION**.

### Values

- No hardcoded resource names, locations, or IDs in `.tf` files. These must come from variables or data sources.
- Default values in variables must be safe/empty where possible. Defaulting to a real resource ID or a non-null sensitive value is a **BLOCKER**.
- `default = null` is acceptable for optional values, but the consuming module or resource block must handle the null case (conditional or `dynamic` block).
- **[module]** The `location` variable must default to `"westus3"`. This is the team standard region. Any other default (e.g. `"westus2"`) is a **SUGGESTION** to correct. The default acts as a safe fallback — consumers are still expected to set location explicitly in `.tfvars`.
- **[consumer]** The `location` variable in `variables.tf` must default to `"westus3"`. If the default is missing or set to a different region, flag as a **SUGGESTION**. Do not flag `.tfvars` files for omitting `location` — the default in `variables.tf` is the single source of truth for region.

### Sensitive Variables

- Any variable whose name contains `_key`, `_secret`, `_password`, or `_token` must be marked `sensitive = true`. Without this, the value is printed in `terraform plan` output in CI logs. This is a **BLOCKER**.

  ```hcl
  # Correct
  variable "storage_account_access_key" {
    description = "Access key for the storage account. Source from Key Vault."
    type        = string
    sensitive   = true
  }
  ```

- Storage account access keys passed as variables must be marked `sensitive = true` even if the value is sourced from a Key Vault data source upstream. The sensitivity declaration travels with the variable, not the source.
- `sensitive = true` on output values is required if the output contains or derives from a sensitive input. Flag missing `sensitive = true` on outputs that reference sensitive variables as a **BLOCKER**.

### Security

- Storage account access keys must not be passed as plain strings in `.tfvars` or inline. Reference Key Vault via data source.
- `ip_restriction_default_action` should default to `"Deny"` unless there is an explicit justification.
- `scm_ip_restriction_default_action` should default to `"Deny"` unless there is an explicit justification.
- User-assigned identities must be preferred over system-assigned where the identity needs to be shared, referenced by other resources, or pre-provisioned. System-assigned identities on shared resources are a **SUGGESTION** to convert.
- **[consumer]** Cross-subscription provider aliases (e.g., `provider "azurerm" { alias = "network" }`) must have an inline comment explaining the purpose and the target subscription context. An unexplained alias is a **SUGGESTION**. Reusable modules do not declare providers and should never contain provider alias blocks.

  ```hcl
  # Cross-subscription alias: reads VNet/Subnet resources from the shared networking subscription
  provider "azurerm" {
    alias           = "network"
    subscription_id = var.subscription_id_vnet_common
    features {}
  }
  ```

### Data Sources *(consumer root modules only)*

- Data sources must be used to reference **existing** resources not managed by this root module. Creating a resource and also reading it with a data source in the same root module is a **BLOCKER** — use resource attributes directly.
- Data sources that are conditionally needed (e.g., prod-only networking reads) must use `count = var.is_prod ? 1 : 0` or equivalent. An unconditional data source that will fail in environments where the resource does not exist is a **BLOCKER**.
- Prefer data sources over hardcoded IDs for existing shared infrastructure (VNets, subnets, service plans, identities). Hardcoded resource IDs in `.tf` files are a **BLOCKER**; hardcoded IDs in `.tfvars` are a **SUGGESTION** to move to data source lookup.

### Locals

- `locals.tf` should contain computed values, transformations, and tag maps — not simple variable pass-throughs. A `locals.tf` that only does `local.foo = var.foo` is a **NITPICK** to simplify.
- Complex IP restriction transformations (e.g., `for` expressions mapping subnet keys to IDs) belong in `locals.tf`, not inline in `main.tf`. If this logic appears inline in a resource argument, flag as a **SUGGESTION** to extract.
- Every `locals` block with non-obvious logic must have an inline comment explaining the transformation. Uncommented nested `for` expressions are a **SUGGESTION**.

### Lifecycle Blocks

- `lifecycle { ignore_changes = [...] }` must have an inline comment explaining why those attributes are excluded. An uncommented `ignore_changes` block is a **SUGGESTION**.

  ```hcl
  lifecycle {
    # app_settings and connection_string are managed post-deploy via pipeline variable substitution
    # tags are managed by Azure Policy; Terraform changes would be overwritten
    ignore_changes = [app_settings, connection_string, tags]
  }
  ```

- **[module]** `lifecycle { prevent_destroy = true }` is **recommended** on stateful resources (storage accounts, Key Vaults, databases, Service Bus namespaces). Modules own the resource definition and are the right place to enforce this guard. Its absence on those resource types is a **SUGGESTION**.
- `lifecycle { create_before_destroy = true }` should be used when replacing a resource would cause downtime. If a resource name is computed (and thus immutable), flag the absence of this as a **SUGGESTION**.

### Module Calls *(consumer root modules only)*

- Required variables must not be omitted. A missing required variable with no default is a **BLOCKER**.
- Unused optional variables passed to a module are noise — flag as a **SUGGESTION** to clean up.
- Module `source` paths must be either:
  - Local relative paths (`./modules/FunctionApp`) — the team standard for consumer repos
  - Do not flag local paths as needing registry or git conversion — local paths are the established convention.
- `for_each` is preferred over `count` when creating multiple instances of a resource where each instance has a meaningful identity (e.g., slot names, environment names). Using `count` for named resources makes state addressing fragile. Flag as a **SUGGESTION**.
- In `module` blocks, argument order must follow this sequence: `source` first (it is the block's identity), then `for_each` or `count` if present, then a blank line, then all module-specific arguments, then `depends_on` or `providers` last (with a blank line before them) if needed. Incorrect ordering is a **NITPICK**.

  ```hcl
  # Correct
  module "service_bus_subscription" {
    source   = "./modules/ServiceBusSubscription"
    for_each = toset(var.environment)

    subscription_name = each.key
    namespace_id      = data.azurerm_servicebus_namespace.ns[each.key].id
    common_tags       = var.common_tags
  }
  ```

- In `resource` blocks, meta-arguments (`for_each`, `count`) must appear first in the block body, separated from the resource-specific arguments by one blank line. `lifecycle` and `depends_on` go last, also separated by a blank line. Incorrect ordering is a **NITPICK**.

  ```hcl
  # Correct
  resource "azurerm_windows_function_app_slot" "example" {
    for_each = toset(var.slot_names)

    name            = each.key
    function_app_id = azurerm_windows_function_app.example.id

    lifecycle {
      ignore_changes = [app_settings]
    }
  }
  ```

---

## Severity Levels

| Label | Meaning |
|---|---|
| **BLOCKER** | Must be fixed before this PR can be approved |
| **SUGGESTION** | Recommended change; author should address or explicitly decline |
| **NITPICK** | Minor style or consistency issue; author's discretion |
| **REMINDER** | No code change required — flags a required manual follow-up action outside the PR (e.g., a step in ADO, Azure Portal, or a downstream system that must be completed after merge) |
