# appsettings Environment Wiring

> How environment-specific appsettings files are selected at deploy time and loaded at runtime for .NET API/webapp pipelines.

---

## Contents

- [appsettings Environment Wiring](#appsettings-environment-wiring)
  - [Contents](#contents)
  - [Overview](#overview)
  - [Deploy Time: Which File Survives](#deploy-time-which-file-survives)
  - [Runtime: Which File ASP.NET Core Loads](#runtime-which-file-aspnet-core-loads)
  - [The Staging Gotcha](#the-staging-gotcha)

---

## Overview

Two separate mechanisms control which appsettings file an app uses. They must align or the wrong config loads.

1. **Deploy time** — the pipeline deletes all env-specific files except one
2. **Runtime** — ASP.NET Core picks which remaining file to load based on `ASPNETCORE_ENVIRONMENT`

---

## Deploy Time: Which File Survives

The `PowerShell-ReplaceConfigAndJson.yml` shared template handles this. It removes all `appsettings.*.json` files except the one matching `$(Environment)`.

```powershell
# Keeps: appsettings.json (base) + appsettings.$(Environment).json
# Deletes: everything else
Get-ChildItem -Filter "appsettings.*.json" `
    -Exclude "appsettings.json","appsettings.$(Environment).json" | Remove-Item
```

`$(Environment)` comes from the environment-specific variable group (e.g., `StagingEnvironment`). See [variable-groups.md](./variable-groups.md).

> **Note:** `RenameJson: true` is used for integration test runs only. It renames `appsettings.$(Environment).json` → `appsettings.json` rather than leaving both files in place.

---

## Runtime: Which File ASP.NET Core Loads

ASP.NET Core merges:
1. `appsettings.json` (base, always loaded)
2. `appsettings.{ASPNETCORE_ENVIRONMENT}.json` (overlay, if it exists)

`ASPNETCORE_ENVIRONMENT` is usually a Windows system environment variable set directly on the VM — not in the pipeline YAML, web.config, or Terraform. A handful of projects break this pattern and set it directly in pipeline YAML instead (e.g. `TUPSS.CET.Api`, `TUPSS.FRS.Services.API`, `TUPSS.CAMS.Validation.Api`, `TUPSS.FRAMIS.Api`) — confirm per-project which mechanism is in play before assuming the VM-level default.

For the deployed file to actually be loaded, `$(Environment)` (pipeline) and `ASPNETCORE_ENVIRONMENT` (VM) must match.

---

## The Staging Gotcha

Whether Staging runs on its own settings or collapses onto Production is a **per-team/ADO-project decision**, set via that team's `StagingEnvironment` variable group. There is no single default — check the team before assuming either way.

> The table below is a snapshot, not the source of truth. Always verify the actual `StagingEnvironment` variable group in ADO before making a decision.

| Team / ADO Project | `StagingEnvironment.Environment` |
|---|---|
| CAMS | `Staging` |
| Center Evaluations (CET) | `Production` |
| Center Management (MPS) | `Staging` |
| Enterprise | `Production` |
| FRAMIS | `Production` |
| Franchise Reporting (FRS) | `Production` |
| Package Management | `Staging` |

When `Environment = Production`:
- The pipeline **keeps `appsettings.Production.json`** and deletes `appsettings.Staging.json`
- Staging runs on Production app settings

This is an established pattern for those teams. Consequences:
- Any config that differs between Staging and Production must go in `appsettings.Production.json`
- A separate `appsettings.Staging.json` file will never be deployed — do not add one expecting it to be used
- If segregation is needed (e.g., separate telemetry connection strings), `Environment` in `StagingEnvironment` must be changed to `Staging` **and** `ASPNETCORE_ENVIRONMENT` on the Staging VMs must be updated to match — coordinate with the team before making this change
