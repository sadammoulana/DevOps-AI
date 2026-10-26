---
description: "TUPSS DevOps team PR review standards for Azure DevOps YAML pipelines, PowerShell scripts, and Markdown documentation."
applyTo: "**/*.yml,**/*.yaml,**/*.ps1,**/*.md"
---

# TUPSS PR Review Standards — Azure DevOps

These standards apply to YAML pipelines, PowerShell scripts, and Markdown documentation. When reviewing, propose concrete edits — do not just identify problems.

> Background, rationale, and full procedures live in [docs/](../../docs/) (`pipelines/`, `source-control/`, `terraform/`) — these rules are the enforceable summary, not the whole standard.

---

## General (All File Types)

- **No hardcoded secrets or credentials.** Any secret, password, key, token, or connection string must reference a Key Vault, variable group, or environment variable — never inline.
- **Secrets must never be written to output or logs.** Variables holding secrets — including values retrieved from Key Vault, library variable groups, or pipeline secret variables — must not be passed to `Write-Host`, `Write-Output`, `Write-Verbose`, `Write-Debug`, `echo`, or any other logging or output mechanism. Flag any variable with a name matching common secret patterns (`*Key*`, `*Secret*`, `*Token*`, `*Password*`, `*Credential*`, `*ApiKey*`) that appears as an argument to an output command. This is a blocker regardless of file type.
- **No commented-out code in submitted PRs.** Dead code must be deleted before merge. If it's a deliberate placeholder, it must have a `# TODO:` comment explaining why and owned by a named person.
- **Descriptive naming.** Names must communicate intent. Single-letter variables, `temp`, `test123`, `foo`, or similar are blockers.
- **Scope is bounded.** A PR should do one thing. If unrelated files are modified, flag it — the author should split the PR or justify the coupling.

---

## YAML Pipelines (Azure DevOps)

### Structure
- Stages, jobs, and steps must have explicit `displayName` values. Unnamed steps are suggestions.
- Templates must be referenced over inline duplication. If the same sequence of steps appears in more than one place, it should be a template.
- Variable groups must be used for environment-specific values, and the correct group must be injected per stage/environment. Inline variable definitions for environment config are blockers; a mismatched or missing group per stage is a suggestion.
- All [required files](../../docs/pipelines/required-files.md) must be present for the pipeline type. Missing required files are a suggestion.
- If the solution file changed, it must reflect the same additions/removals as the PR's other file changes. A stale solution file is a suggestion.
- Agent pools must be explicitly set and appropriate for the job. Missing or default-fallback agent pools are a suggestion.

### Secrets and Security
- No plaintext secrets in pipeline YAML. All secrets must come from a linked Key Vault variable group or ADO secret variable.
- `$(System.AccessToken)` usage must be scoped and justified. Broad token usage without comment is a suggestion.

### Triggers and Conditions
- All pipelines must define an explicit `trigger` or `pr` block, or explicitly set `trigger: none`. Missing trigger definition is a suggestion.
- Branch triggers must match the pipeline type — `main` for the standard pipeline, `qa-lane2` for the hotfix pipeline. A mismatched branch trigger is a blocker.
- `condition` expressions must be readable. Deeply nested or opaque condition strings must be broken out into named variables or template expressions.
- Stage dependencies must match the intended deployment flow (see [stage-design.md](../../docs/pipelines/stage-design.md)). An incorrect `dependsOn` is a blocker.

### Solution File Contract
When a pipeline passes a solution-related parameter (`Solution` or `SolutionName`) to a shared build template (`DotNetRestore.yml`, `DotNetBuild.yml`, `NuGet.yml`, `Pre-Package.yml`, `VisualStudioBuild.yml`, `Database.yml`), the value must conform to the canonical input standard:

```yaml
# Correct — extensionless, variable-driven
variables:
  - name: SolutionName
    value: TUPSS.Repo.Name

parameters:
  Solution: ${{ variables.SolutionName }}
```

The following legacy patterns are non-conforming and must be flagged as `BLOCKER` with a proposed migration to the canonical form:

| Non-conforming pattern | Example |
|------------------------|---------|
| Hardcoded with extension | `Solution: 'TUPSS.Repo.Name.sln'` |
| Variable with extension appended | `Solution: '${{ variables.SolutionName }}.sln'` |
| Variable with extension appended (runtime) | `Solution: '$(SolutionName).sln'` |
| Glob with hardcoded name | `Solution: '**/TUPSS.Repo.Name.sln'` |
| Broad glob | `Solution: '**/*.sln'` |
| Variable holding full filename | `SolutionName: '${{ variables.SolutionFile }}'` where `SolutionFile` contains an extension |

When flagging, propose:
1. A `SolutionName` variable set to the extensionless base name
2. The parameter updated to `Solution: ${{ variables.SolutionName }}`

**Omitting `Solution` entirely is also non-conforming.** When a consumer pipeline calls a shared build template that accepts a `Solution` parameter (`Database.yml`, `DotNetBuild.yml`, `DotNetRestore.yml`, `NuGet.yml`, `Pre-Package.yml`, `VisualStudioBuild.yml`) without passing the parameter, the template falls through to its broad glob default. This is a `BLOCKER` — treat it the same as explicitly passing `**/*.sln`.

Note: the broad glob default (`**/*.sln`) is allowed in templates as a backward-compatibility fallback. Flag it only in consumer pipeline YAML, not in template definitions.

### Hotfix Pipeline
- A hotfix pipeline (`azure-pipeline-hotfix.yml`) is **not always required** — confirm it was explicitly requested before flagging its absence.
- Requested but missing: flag `SUGGESTION` with a proposed skeleton.

### Post-Merge Pipeline Setup
- These reminders apply only when the PR adds a **new** `azure-pipeline*.yml` file (a file that did not previously exist). Modifications to an existing pipeline file do not trigger them — assume the pipeline object and branch policy already exist.

> **[REMINDER]** — Confirm both pipeline objects exist in ADO  
> Manually create the standard pipeline (and hotfix pipeline, if applicable) in ADO: Pipelines → New Pipeline → point to the YAML file.

> **[REMINDER]** — Confirm build validation policy is set on `main`  
> Add the new pipeline as a build validation policy on the `main` branch.

### Environment Configuration
- `local/env.yml` files must be present and correct for every environment the pipeline deploys to (see [variable-groups.md](../../docs/pipelines/variable-groups.md)). Missing or incorrect env files are a suggestion.
- If integration tests are present, host files must be configured (see [host-file-resolution.md](../../docs/pipelines/host-file-resolution.md)). If host file changes are needed, a separate PR to `TUPSS.DevOps.CICD` is required — flag its absence as a suggestion.

### Hygiene
- Pipeline files must not contain large blocks of commented-out YAML from previous iterations.
- Steps that `echo` sensitive values (even masked ones) must be removed before merge.

---

## PowerShell

### Function and Script Structure
- All functions must use approved PowerShell verbs (run `Get-Verb` to verify). Unapproved verbs are blockers.
- Parameters must be typed. Untyped parameters (`$param` with no `[type]`) are suggestions.
- Functions with more than one purpose must be split. A function should do one thing.

### Error Handling
- Scripts that call external APIs, Azure cmdlets, or file system operations must use `try/catch` or `-ErrorAction Stop`. Silent failures are blockers.
- `-ErrorAction SilentlyContinue` is a blocker unless accompanied by an inline comment that specifically explains why suppression is intentional and appropriate. A vague comment (e.g., `# suppress errors`, `# ignore`) does not satisfy this rule. If the justification does not clearly describe the scenario being handled, treat the rule as unsatisfied and call it out.
- `Write-Error` is preferred over `throw` for non-terminating errors in pipeline scripts.

### Security
- No hardcoded credentials, connection strings, or tokens. Use Key Vault, environment variables, or parameter input.
- `ConvertTo-SecureString -AsPlainText -Force` used with a hardcoded string is a blocker.
- Avoid `Invoke-Expression` unless absolutely necessary and explicitly justified in a comment.

### Hygiene
- `Write-Host` is acceptable for user-facing scripts. In automation/pipeline scripts, use `Write-Output` or `Write-Verbose`.
- Scripts must not leave temp files or modified state without cleanup logic.
- Flag any non-ASCII characters in string literals, command arguments, or operator positions as a `SUGGESTION`. Common copy-paste artifacts — smart quotes (`"` `"` `'` `'`), em dashes (`—`), en dashes (`–`), and non-breaking spaces — are invisible to the eye but cause silent failures at runtime. Note: file-level encoding (UTF-8 vs ANSI) cannot be verified from a diff and is out of scope for this check.

---

## Markdown / Documentation

### Accuracy
- Code examples in README files must match the actual module interface. Field names, attribute keys, and types in code blocks must match the corresponding variable definitions in `.tf` files.
- Variable descriptions in examples must reference only fields that exist in the declared type — do not reference fields from a different usage pattern (e.g., do not use `subnet_key` in a basic example where that field is not in the type definition).

### Completeness
- CHANGELOG entries for breaking changes must include: what was removed, what replaces it, and a complete migration path. Dangling sentences or empty lists are blockers.
- No commented-out sections in final form unless they serve as deliberate rollback reference. In that case, a `<!-- TODO: remove after [date/milestone] -->` comment is required.

### Hygiene
- Migration steps must be actionable and sequenced. A consumer should be able to follow them without cross-referencing other files.

---

## Severity Levels

| Label | Meaning |
|---|---|
| **BLOCKER** | Must be fixed before this PR can be approved |
| **SUGGESTION** | Recommended change; author should address or explicitly decline |
| **NITPICK** | Minor style or consistency issue; author's discretion |
| **REMINDER** | A post-merge action item. Not a code problem — something that must be done in ADO or an external system that is easy to forget (e.g. creating the pipeline object in ADO after the YAML is merged). |
