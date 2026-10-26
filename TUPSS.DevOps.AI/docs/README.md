# DevOps Docs

Reference material and team standards for the DevOps platform — pipeline conventions, Terraform standards, PR practices, and how we work.

## Scope

Standards and conventions for the DevOps platform — written for people, structured for agents.

## Contents

| Folder | What's in it |
|---|---|
| [meta/](meta/) | Conventions for writing docs in this folder |
| [pipelines/](pipelines/) | Stage design, required files, variable groups, appsettings wiring, host file resolution, SonarQube/JFrog onboarding |
| [source-control/](source-control/) | PR standards, PR description template, branch strategy, branch policies |
| [terraform/](terraform/) | Module delivery pattern, local/pipeline testing, provider version management, changelog style, `.gitignore` standards |
| [_archive/](_archive/) | Retired docs, kept for historical reference only |

## How agents use this

- **PR Preflight** pulls the relevant doc section based on what it's reviewing (e.g., a Terraform PR pulls from `terraform/`, a pipeline PR pulls from `pipelines/`).
- Nothing here is force-loaded via `applyTo` — retrieval is deliberate and scoped, keeping context usage proportional to what's actually being reviewed.
