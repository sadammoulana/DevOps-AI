# Testing Terraform Modules Locally and In-Pipeline

> Module tagging/versioning mechanics live in `TUPSS.Terraform/README.md` — that is the source of truth. This doc only covers how to point a consumer pipeline at a module version under development.

## Context

Because modules are injected as a pipeline artifact at build time (see [module-delivery.md](./module-delivery.md)), there is no native local path to run or validate a consumer repo against modules under development. A pipeline run against a prerelease tag is the only way to actually test a module change end-to-end; the local validation option below is a lighter, faster supplementary check (syntax/schema only), not a substitute for it.

The local validation procedure is also the basis for generating/refreshing `.terraform.lock.hcl` — see [provider-version-management.md](./provider-version-management.md).

---

## Contents

- [Testing Terraform Modules Locally and In-Pipeline](#testing-terraform-modules-locally-and-in-pipeline)
  - [Context](#context)
  - [Contents](#contents)
  - [Full Test — Use a Prerelease Tag as the Modules Version (In-Pipeline)](#full-test--use-a-prerelease-tag-as-the-modules-version-in-pipeline)
  - [Local Validation — Copy `modules/` Directly Into the Consumer Repo](#local-validation--copy-modules-directly-into-the-consumer-repo)
  - [Summary](#summary)

---

## Full Test — Use a Prerelease Tag as the Modules Version (In-Pipeline)

`latest` (floating, no tag filter) is no longer used — module releases are tagged semver now (see `TUPSS.Terraform/README.md`). Instead, manually trigger the TUPSS.Terraform publish pipeline against your own `feature/*` branch to produce a scoped prerelease tag, then pin the consumer pipeline to it:

1. Go to the TUPSS.Terraform.Modules pipeline in ADO.
2. Run it manually against your `feature/<name>` branch with the appropriate `increment` value.
3. Wait for the run to complete — it creates and pushes a prerelease tag (e.g. `2.0.0-rc.1`) and publishes the `modules` artifact for that tag.
4. In the consumer repo's `azure-pipelines.yml`, set:

   ```yaml
   variables:
     - name: ModulesVersion
       value: '2.0.0-rc.1'
   ```

**Use this when:** you want a visible pipeline run against modules built from your specific branch.

**Risks:**

- Each run against the same branch/increment produces the next `rc.N` — make sure you're pinning to the tag your last run actually produced, not a stale one.
- Still a real published tag, not a local-only test — don't leave a consumer repo pinned to a prerelease tag on `main`.

**Revert after testing.** Restore the final release version tag before opening a PR.

---

## Local Validation — Copy `modules/` Directly Into the Consumer Repo

The simplest local option. Used for running `terraform validate` — syntax/schema only, does not exercise a real plan/apply. Use this as a quick pre-check before a pipeline run, not as a replacement for the Full Test above.

**Steps:**

1. Clone or pull `TUPSS.Terraform` locally.
2. Copy the entire `modules/` directory into the root of the consumer repo:
   ```
   TUPSS.YourApp.Terraform/
     main.tf
     variables.tf
     modules/        ← copied from TUPSS.Terraform
       ResourceGroup/
       FunctionApp/
       ...
   ```
3. From the consumer repo root, run:
   ```powershell
   terraform init -backend=false
   terraform validate
   ```
4. Clean up before committing:
   ```powershell
   Remove-Item -Recurse -Force .terraform
   Remove-Item -Recurse -Force modules
   ```

**`-backend=false`** skips remote state configuration, which requires Azure credentials and a real storage account. Do not run a full `init` (with backend).

`modules/` is a gitignored path per the `.gitignore` standard, so it won't get staged accidentally — but still remove it once you're done (see cleanup step above).

---

## Summary

| Approach | Scope | Requires pipeline run | Risk |
|--------|-------|----------------------|------|
| Full Test — prerelease tag (`rc.N`) | All modules, specific branch build, real plan/apply | Yes | Must remember to revert to the release tag before opening a PR |
| Local Validation — copy `modules/` locally | All modules, specific local version, syntax/schema only | No | Disk clutter if left in place (gitignored) |
