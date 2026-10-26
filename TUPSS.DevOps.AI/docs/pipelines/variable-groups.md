# Variable Groups

Variable groups injected into pipeline stages. Each stage pulls a combination of groups depending on environment and project needs.

---

## Contents

- [Variable Groups](#variable-groups)
  - [Contents](#contents)
  - [Service Account Groups](#service-account-groups)
  - [Environment Secret Groups](#environment-secret-groups)
  - [Infrastructure Groups](#infrastructure-groups)
  - [Per-Stage Reference](#per-stage-reference)

---

## Service Account Groups

Connected to Azure Key Vault. Injected per stage to provide service account credentials for app pool identity.

| Group | Environments |
|---|---|
| `KeyVaultDevQA` | Dev, Dev-Lane2, Test, QA-Lane2 |
| `KeyVaultStaging` | Staging |
| `KeyVaultProd` | Prod |

---

## Environment Secret Groups

Contain values consumed by the application at runtime (connection strings, feature flags, etc.). Names are consistent across projects.

| Group | Environment |
|---|---|
| `DevEnvironment` | Dev |
| `Dev-Lane2Environment` | Dev-Lane2 |
| `TestEnvironment` | Test |
| `StagingEnvironment` | Staging |
| `ProdEnvironment` | Prod |
| `Test-Lane2Environment` | QA-Lane2 (hotfix) |

> **Note:** `StagingEnvironment` doesn't always set `Environment = Staging`. See [appsettings-environment.md](./appsettings-environment.md#the-staging-gotcha) for the breakdown.

---

## Infrastructure Groups

| Group | Purpose |
|---|---|
| `DevOpsPlatform` | Common DevOps values injected by the pipeline team (e.g., service connection names). Rollout in progress across projects |
| `HostsFiles` | Tag-to-hosts-file mapping for multi-server environments. See [host-file-resolution.md](./host-file-resolution.md) |
| `KeyVaultDev` | Supplies `TUPSS-ENT-AUTHORIZATION-API-SIGN-KEY` for non-prod stages. Not every project needs it — check for `JwtAuthorizationToken:IssuerSigningKey` in `appsettings.*.json` to confirm. |
| `KeyVaultNonAzure` | Non-Azure secrets (e.g., on-prem credentials) |
| `KeyVaultPlatform` | Platform-level secrets |
| `KeyVaultThirdParty` | Third-party service credentials |

---

## Per-Stage Reference

Typical variable group combination per stage:

```yaml
# Dev-Lane2
variables:
  - template: local/dev-lane2.yml
  - group: Dev-Lane2Environment
  - group: KeyVaultDevQA

# Dev / Test (pattern)
variables:
  - template: local/dev.yml         # or test.yml
  - group: DevEnvironment           # or TestEnvironment
  - group: KeyVaultDevQA

# Staging
variables:
  - template: local/staging.yml
  - group: StagingEnvironment
  - group: KeyVaultStaging

# Prod
variables:
  - template: local/prod.yml
  - group: ProdEnvironment
  - group: KeyVaultProd

# QA-Lane2 (hotfix)
variables:
  - template: local/qa-lane2.yml
  - group: Test-Lane2Environment
  - group: KeyVaultDevQA

```
