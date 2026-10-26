# Contributing to TUPSS.DevOps.AI

Every engineer on the team can and should contribute — especially when an agent gets something wrong or a standard is missing.

---

## How to Contribute

### Option 1 — Agent-assisted (recommended)

The PR Preflight agent has a built-in feedback loop. At the end of any review session where the agent got something wrong or missed something, ask it:

> "Propose updates to the instructions based on what we discussed."

The agent will:
1. Summarize the issues from the review and ask you to confirm
2. Open a feedback branch in this repo
3. Make the proposed edits to the relevant instruction file(s)
4. Hand off — it will not commit or push

You then review the diff, adjust as needed, commit, push, and open a PR.

This is the fastest path. A review session where you push back on the agent can turn into a merged standards update in under 10 minutes.

### Option 2 — Direct edit

Clone the repo, create a feedback branch, edit the relevant instruction file, open a PR. No ceremony required.

### Option 3 — Raise it verbally / in standup

If you caught an issue but don't have time to fix it, flag it to the team so someone can follow up. This is the slowest path — prefer Option 1 or 2 if you're already in the repo.

---

## Branch Naming

All contribution branches use the `feedback/` prefix:

```
feedback/YYYYMMDD-<2-4-word-kebab-slug>
```

Examples:
- `feedback/20260626-terraform-sensitive-vars`
- `feedback/20260701-yaml-trigger-rule`
- `feedback/20260710-hotfix-pipeline-reminder`

If the agent creates the branch for you (Option 1), it follows this convention automatically and handles name collisions.

---

## Writing Good Rules

When adding or modifying a rule in an instruction file, follow these conventions:

- **Be specific, not vague.** "Variables must have descriptions" is a rule. "Write good code" is not.
- **Assign a severity.** Every rule should indicate whether a violation is a `BLOCKER`, `SUGGESTION`, or `NITPICK`.
- **Scope module rules.** In the Terraform instruction file, mark rules that only apply to reusable modules as `[module]` and consumer-only rules as `[consumer]`. Unmarked rules apply to both.
- **Include an example.** Rules without examples are harder to apply consistently. Show the correct pattern in a code block.
- **Match existing tone.** Direct, imperative, no hedging. The instruction files are standards documents, not guidelines.

---

## PR Expectations

- One concern per PR. If you found two unrelated issues, open two branches.
- PRs against this repo follow the same process as any other: run the PR Preflight agent on your own changes before requesting review. Yes, you can use the agent to review changes to the agent.
- The `main` branch is the primary branch for this repo. Target all PRs there.
