# SonarQube and JFrog

Configuration steps for onboarding a pipeline to SonarQube (code quality) and JFrog (artifact proxy). These are standard for all new `.NET` pipelines.

Reference: [TUPSS Sonar & JFrog Onboarding](https://dev.azure.com/theupsstore/Release%20Management/_wiki/wikis/Release-Management.wiki/4934/TUPSS-Sonar-Jfrog-Onboarding)

---

## Contents

- [SonarQube and JFrog](#sonarqube-and-jfrog)
  - [Contents](#contents)
  - [SonarQube](#sonarqube)
    - [Enabling the Quality Gate](#enabling-the-quality-gate)
  - [JFrog](#jfrog)
    - [Pipeline Change](#pipeline-change)
    - [NuGet Source Updates](#nuget-source-updates)

---

## SonarQube

> **Scope:** Standard and hotfix pipelines only. Database pipelines do not run SonarQube.

Add the prepare and publish steps around the build and test steps. The quality gate is optional at onboarding but is intended to be enabled once the project has baseline coverage.

```yaml
# Without quality gate (onboarding)
- template: Templates/Build/SonarQubePrepare.yml@templates

# With quality gate enforced
- template: Templates/Build/SonarQubePrepare.yml@templates
  parameters:
    QualityGate: "true"

# After DotNetTest — always required
- template: Templates/Build/SonarQubeRunPublish.yml@templates
```

Coverage is configured via `codeCoverage.runsettings`. See [required-files.md](./required-files.md) for the standard template and module path instructions.

### Enabling the Quality Gate

Enabling the quality gate is a three-part change. All three steps are required — the parameter alone is not sufficient.

**1. Set the parameter**

Update `SonarQubePrepare` as shown above.

**2. Verify stage conditions are at the standard pattern**

Enabling the quality gate is a checkpoint to audit and correct any condition drift. Prod must be locked to `refs/heads/main` — this ensures only code merged through a validated PR can reach that environment. Staging is typically locked to `main` as well, but exceptions exist.

See [stage-design.md](./stage-design.md) for the standard conditions. Pay particular attention to:
- `Staging` — typically uses `eq(variables['Build.SourceBranch'], 'refs/heads/main')` (see exception below)
- `Prod` — must depend on `succeeded('Staging')` and also be locked to `main`

If the project's conditions differ from standard and aren't the documented MPS/Center Management exception, that's drift — align it to standard as part of this upgrade.

**3. Add a build validation policy in Azure DevOps**

Without this, the quality gate has no enforcement on PRs — a developer can complete a PR even if the build failed.

- Navigate to **Repos → Branches → `main` → Branch Policies**
- Under **Build Validation**, add the pipeline
- Set **Path filter** to `/*; !/build/*azure-pipeline*` (excludes pipeline YAML changes, matching the trigger exclusion on the pipeline itself)
- Set to **Required**
- Set trigger to **Automatic** and expiration to **After the latest sourcecode is pushed** (prevents stale passing results from satisfying the policy)

---

## JFrog

### Pipeline Change

Add `useJfrogSettings: true` to `DotNetRestore`:

```yaml
- template: Templates/Build/DotNetRestore.yml@templates
  parameters:
    Solution: '<REPO_SOLUTION_FILE>'
    useJfrogSettings: true
```

### NuGet Source Updates

Update both `sources.props` and `NuGet.config` to replace any `api.nuget.org` references with the JFrog proxy.

```xml
<!-- sources.props — add to RestoreSources -->
https://jfrog.inside.ups.com/artifactory/api/nuget/v3/nuget-remote/index.json;

<!-- NuGet.config — add to packageSources -->
<add key="UPSjfrog" value="https://jfrog.inside.ups.com/artifactory/api/nuget/v3/nuget-remote/index.json" />
```

See [required-files.md](./required-files.md) for the complete standard `sources.props` and `NuGet.config` templates.
