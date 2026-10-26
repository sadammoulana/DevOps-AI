# Pull Request Standards

How to structure and submit PRs on the DevOps team. The goal is to give reviewers exactly the context they need — no more.

---

## Contents

- [Pull Request Standards](#pull-request-standards)
  - [Contents](#contents)
  - [Review Expectations](#review-expectations)
  - [PR Description](#pr-description)
  - [Submitter Responsibilities](#submitter-responsibilities)

---

## Review Expectations

1. Submitter runs the PR Preflight agent before requesting review
2. First human review by one team member
3. Final human review by team lead before merge

Reviewer guidance: check the full PR end-to-end (not just selected files).

**Exception — trivial changes:** for a one- or two-line change, skip PR Preflight and the description template entirely. A single reviewer is sufficient.

---

## PR Description

Every PR must use the [PR description template](./pr-description-template.md) — see that doc for required/optional sections. PR Preflight generates this for you; you shouldn't need to write it by hand. Exception: trivial changes (see above) don't need a description at all.

---

## Submitter Responsibilities

- Confirm branch policies are enabled on the target branch post-merge, if this PR set them up for the first time (new repo, new branch)
- Follow up with whoever requested the change once it's merged and deployed — don't assume they're watching the PR

> TODO: expand this list as more submitter-owned follow-through items come up.

