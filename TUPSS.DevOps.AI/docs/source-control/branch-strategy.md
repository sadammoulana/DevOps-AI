# Branch Strategy

Branch naming and purpose across pipeline repos, and how they map to pipeline triggers and deployment stages.

---

## Contents

- [Branch Strategy](#branch-strategy)
  - [Contents](#contents)
  - [Primary Branch](#primary-branch)
  - [DevOps Team Branch Practices](#devops-team-branch-practices)
  - [Terraform Module Branches](#terraform-module-branches)
  - [Working on a Pipeline-Only Branch](#working-on-a-pipeline-only-branch)


---

## Primary Branch

`main` is the primary branch for nearly every repo.

**Exception:** `TUPSS.DevOps.CICD` uses `devops` as its primary branch, not `main`. Confirm the primary branch before creating a working branch or opening a PR — don't assume `main`.

---

## DevOps Team Branch Practices

> TODO: define the DevOps team's own branching practices (working branch conventions, when to branch vs. work directly, merge/rebase expectations, retention/cleanup of old branches).

---

## Terraform Module Branches

Terraform module releases use **tagged releases with semver**, replacing the old `latest`/floating-tag model. `main` produces a final release tag (e.g. `2.0.0`); a `feature/<name>` branch produces a prerelease tag (e.g. `2.0.0-rc.1`). Full mechanics (version calculation, pipeline run instructions) live in `TUPSS.Terraform/README.md` — that is the source of truth, not this doc.

Same initiative referenced in [module-delivery.md](../terraform/module-delivery.md) and [module-testing.md](../terraform/module-testing.md).

---

## Working on a Pipeline-Only Branch

When building a new pipeline from scratch, no dev code exists yet on the working branch. Use this pattern to test before the PR:

1. Create your working branch off `main`
2. Build out `build/` and all required files (see [required-files.md](../pipelines/required-files.md))
3. When ready to test, create a temporary integration branch (e.g., `devops-sm-intg`)
4. Merge your working branch into it
5. Merge the relevant dev branch into it
6. Test against the integration branch; push fixes to your working branch and re-merge
7. When satisfied, raise the PR from your original working branch — verify all required files are present before submitting

> The integration branch is throwaway. The PR comes from the clean working branch.
