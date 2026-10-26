# Branch Policies

> Branch protection rules enforced in ADO — quality gates, required reviewers, and build validation. Related but distinct from pipeline trigger `paths.exclude` filters (see [stage-design.md](../pipelines/stage-design.md#triggers)).

---

## Contents

- [Branch Policies](#branch-policies)
  - [Contents](#contents)
  - [Quality Gates](#quality-gates)
  - [Required Reviewers](#required-reviewers)
  - [Build Validation](#build-validation)
  - [Pipeline Trigger Exclusions](#pipeline-trigger-exclusions)

---

## Quality Gates

> TODO: document the quality gates enforced via ADO branch policy (e.g., SonarQube quality gate, minimum reviewer count, comment resolution requirements).

---

## Required Reviewers

> TODO: document required reviewer count/groups per branch (`main` / `devops`, etc.).

---

## Build Validation

> TODO: document which pipelines are wired as required build validation checks on protected branches.

---

## Pipeline Trigger Exclusions

Separate from ADO branch policy, but related: pipeline YAML uses `trigger.paths.exclude` to prevent a wasted CI run when only the pipeline file itself changes. See [stage-design.md — Triggers](../pipelines/stage-design.md#triggers) for the current pattern.

> TODO: confirm whether this needs its own section here or if the `stage-design.md` reference is sufficient once branch policy content is filled in.
