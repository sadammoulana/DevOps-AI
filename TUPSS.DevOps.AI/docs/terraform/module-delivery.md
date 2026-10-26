# Terraform Module Delivery Pattern

> Module tagging/versioning mechanics (how the version is calculated, branch behavior, manual pipeline run instructions) are fully documented in `TUPSS.Terraform/README.md` — that is the source of truth. This doc only covers how a consumer repo pins to and consumes a published version.

---

## Contents

- [Problem Statement](#problem-statement)
- [Key Repositories](#key-repositories)
- [Why `source = "./modules/XYZ"` Works](#why-source--modulesxyz-works)
- [The Full Pipeline Flow](#the-full-pipeline-flow)
- [Version Pinning](#version-pinning)
- [How TUPSS.Terraform Publishes the Artifact](#how-tupssterraform-publishes-the-artifact)

---

## Problem Statement

Consumer Terraform repos reference modules with local paths like `source = "./modules/ResourceGroup"`. There is no `modules/` directory in those repos. This document explains why that path works and how the modules get there.

---

## Key Repositories

| Repo | Role |
|------|------|
| `TUPSS.Terraform` | **Source of truth** for all reusable modules. Contains `modules/` directory. Has its own ADO pipeline (`TUPSS.Terraform.Modules`, definition `775`, Enterprise project) that publishes the modules as a versioned pipeline artifact. The old classic pipeline (definition `194`) remains enabled but is deprecated, slated for decommission once all consumers migrate to v2 modules. |
| `TUPSS.*.Terraform` | Consumer repos. Contain only their own `.tf` files. Reference modules via `./modules/XYZ` local paths. |
| `TUPSS.DevOps.CICD` | Hosts shared pipeline templates used by all consumer repos. |

---

## Why `source = "./modules/XYZ"` Works

The `./modules/` directory does **not** exist in the consumer repo on disk. It is **injected as a pipeline artifact** during the Build stage and bundled alongside the consumer's `.tf` files before any Terraform command runs.

---

## The Full Pipeline Flow

### Stage 1 — Build (`Templates/Build/Terraform.yml@templates`)

Three tasks run in sequence:

**1. Download modules artifact**

```yaml
task: DownloadPipelineArtifact@2
  project:    3e8e71b6-0be4-468a-97bd-6d66940a709a  # Enterprise ADO project
  definition: 775                                    # TUPSS.Terraform.Modules pipeline
  tags:       $(ModulesVersion)                      # e.g. '2.0.0' or a prerelease tag like '2.0.0-rc.1'
  artifactName: modules
  targetPath: $(System.ArtifactsDirectory)\modules   # resolves to the staging dir
```

This downloads the entire `modules/` tree from TUPSS.Terraform and places it at `$(Build.ArtifactStagingDirectory)\modules`.

**2. Copy consumer TF files into the same staging directory**

```yaml
task: CopyFiles@2
  SourceFolder: $(Build.SourcesDirectory)   # the consumer repo clone
  TargetFolder: $(Build.ArtifactStagingDirectory)
```

After this step, `$(Build.ArtifactStagingDirectory)` contains:

```
main.tf
variables.tf
env-vars/
  dev.tfvars
  prod.tfvars
build/
  azure-pipelines.yml
  local/
modules/                   ← from TUPSS.Terraform (step 1 — already here)
  ResourceGroup/
  FunctionApp/
  AppInsights/
  ...
```

**3. Publish the combined directory as `dropTerraform`**

```yaml
task: PublishPipelineArtifact@1
  targetPath: $(Build.ArtifactStagingDirectory)
  artifact: dropTerraform
```

The artifact now contains both the consumer TF files and the full `modules/` tree.

---

### Stage 2 — Deploy (`Templates/Release/Terraform.yml@templates`)

Each environment stage (Dev, Prod) is a deployment job that:

1. Downloads `dropTerraform` to `$(Pipeline.Workspace)\dropTerraform`
2. Runs token replacement on `.tfvars` files
3. Installs the pinned Terraform version (currently `1.11.4`)
4. Runs `terraform init` in `$(Pipeline.Workspace)\dropTerraform`
5. Runs `terraform plan` (output saved as `.tfplan` artifact)
6. Gates on a **manual validation** step (skipped if no changes)
7. Runs `terraform apply` using the saved plan file

At plan/apply time, `./modules/ResourceGroup` resolves correctly because the modules were embedded in the artifact at build time.

---

## Version Pinning

Consumer repos pin to a specific modules version via a pipeline variable, using the semver tag produced by the `TUPSS.Terraform` publish pipeline:

```yaml
# azure-pipelines.yml (consumer repo)
variables:
  - name: ModulesVersion
    value: '2.0.0'
```

A prerelease tag (e.g. `2.0.0-rc.1`) can be used the same way while testing against an in-progress module change on a `feature/*` branch — see [module-testing.md](./module-testing.md).

This value is passed as the `Version` parameter to the Build template, which uses it as a Git tag filter to locate the exact TUPSS.Terraform build to download from.

**To upgrade modules:** change `ModulesVersion` to the new tag value and trigger a new build. See `TUPSS.Terraform/README.md` for how new tags are created.

---

## How TUPSS.Terraform Publishes the Artifact

TUPSS.Terraform has its own ADO pipeline (`TUPSS.Terraform.Modules`, definition `775` in the Enterprise project) that publishes the `modules/` directory as a pipeline artifact named `modules` and tags the build with a semver tag (`main` → release, `feature/*` → prerelease). Full mechanics: `TUPSS.Terraform/README.md`.

> The old classic pipeline (definition `194`) remains enabled but is deprecated, slated for decommission once all consumers have migrated to v2 modules. Do not point new consumer setups at it.

Consumer repos reference that specific tagged run by version number.
