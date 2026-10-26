# Required Pipeline Files

The files expected in every **standard .NET application** pipeline PR (API, Function App, App Service). These live in the repo root or under `build/` and must be present before a pipeline is considered complete.

> **Scope:** This structure applies to standard .NET application pipelines only. Other pipeline types (Database, Terraform, Angular, mobile/Android, NuGet-package-only) have their own shape and are not yet documented — see [TODO: pipeline types](#pipeline-types-todo).

---

## Contents

- [Directory Structure](#directory-structure)
- [Root Files](#root-files)
- [build/ Files](#build-files)
- [build/local/ Files](#buildlocal-files)
- [Pipeline Types (TODO)](#pipeline-types-todo)

---

## Directory Structure

```
├── build/
│   ├── local/
│   │   ├── dev.yml
│   │   ├── dev-lane2.yml
│   │   ├── test.yml
│   │   ├── staging.yml
│   │   ├── prod.yml
│   │   └── qa-lane2.yml
│   ├── azure-pipeline.yml
│   ├── azure-pipeline-hotfix.yml
│   └── sources.props
├── codeCoverage.runsettings
├── global.json
├── NuGet.config
├── <REPO.NAME>.sln
└── version.props
```

---

## Root Files

### `version.props`

Controls versioning for the build. The `VersionPrefix` is the only value that changes per project. Everything else is standard — do not modify the condition logic.

```xml
<Project>
  <PropertyGroup>
    <VersionPrefix>1.0.0</VersionPrefix>
    <VersionSuffix>rtm</VersionSuffix>
    <BuildNumber Condition="'$(BUILD_BUILDNUMBER)' == ''">0</BuildNumber>
    <BuildNumber Condition="'$(BUILD_BUILDNUMBER)' != ''">$(BUILD_BUILDNUMBER)</BuildNumber>
    <BuildId Condition="'$(BUILD_BUILDID)' == ''">0</BuildId>
    <BuildId Condition="'$(BUILD_BUILDID)' != ''">$(BUILD_BUILDID)</BuildId>
    <PackageVersion Condition="'$(IsFinalBuild)' == 'true' AND '$(VersionSuffix)' == 'rtm' ">$(VersionPrefix)</PackageVersion>
    <PackageVersion Condition="'$(IsFinalBuild)' == 'true' AND '$(VersionSuffix)' != 'rtm' ">$(VersionPrefix)-$(VersionSuffix)-final</PackageVersion>
    <PackageVersion Condition="'$(IsFinalBuild)' != 'true' AND '$(VersionSuffix)' != '' ">$(VersionPrefix)-$(VersionSuffix)-$(BuildNumber)</PackageVersion>
    <PackageVersion Condition="'$(IsFinalBuild)' != 'true' AND '$(VersionSuffix)' == '' ">$(VersionPrefix)-$(BuildNumber)</PackageVersion>
    <VersionSuffix Condition="'$(IsFinalBuild)' != 'true' ">$(VersionSuffix)-$(BuildNumber)</VersionSuffix>
    <VersionSuffix Condition="'$(VersionSuffix)' == 'rtm' ">$(BuildNumber)</VersionSuffix>
    <Version>$(VersionPrefix)-$(VersionSuffix)</Version>
    <FileVersion>$(VersionPrefix).$(BuildId)</FileVersion>
    <FileVersionRC>$(FileVersion.Replace('.', ','))</FileVersionRC>
  </PropertyGroup>
</Project>
```

### `global.json`

Pins the .NET SDK version. Confirm the version with the dev team — do not guess.

```json
{
  "sdk": {
    "version": "8.0.101",
    "rollForward": "latestFeature"
  }
}
```

### `NuGet.config`

Standard configuration for internal ADO feed + JFrog. Order matters — `tupss-feed` first.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <clear />
        <add key="tupss-feed" value="https://pkgs.dev.azure.com/theupsstore/_packaging/tupss-feed/nuget/v3/index.json" />
        <add key="UPSjfrog" value="https://jfrog.inside.ups.com/artifactory/api/nuget/v3/nuget-remote/index.json" />
    </packageSources>
</configuration>
```

### `codeCoverage.runsettings`

Configures SonarQube code coverage. The `<ModulePath>` values must match the project's `.dll` output names — verify against `/bin` output after a local build.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<RunSettings>
  <DataCollectionRunSettings>
    <DataCollectors>
      <DataCollector friendlyName="Code Coverage" uri="datacollector://Microsoft/CodeCoverage/2.0"
        assemblyQualifiedName="Microsoft.VisualStudio.Coverage.DynamicCoverageDataCollector, Microsoft.VisualStudio.TraceCollector, Version=11.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a">
        <Configuration>
          <CodeCoverage>
            <ModulePaths>
              <Include>
                <ModulePath>TUPSS.Example.Api.dll</ModulePath>
                <ModulePath>TUPSS.Example.Core.dll</ModulePath>
              </Include>
            </ModulePaths>
            <UseVerifiableInstrumentation>True</UseVerifiableInstrumentation>
            <AllowLowIntegrityProcesses>True</AllowLowIntegrityProcesses>
            <CollectFromChildProcesses>True</CollectFromChildProcesses>
            <CollectAspDotNet>False</CollectAspDotNet>
            <EnableStaticNativeInstrumentation>True</EnableStaticNativeInstrumentation>
            <EnableDynamicNativeInstrumentation>True</EnableDynamicNativeInstrumentation>
            <EnableStaticNativeInstrumentationRestore>True</EnableStaticNativeInstrumentationRestore>
          </CodeCoverage>
        </Configuration>
      </DataCollector>
    </DataCollectors>
  </DataCollectionRunSettings>
</RunSettings>
```

### `<REPO.NAME>.sln`

The solution file must include the following. Folders are added as same-named solution folders; loose files go into the root "Solution Items" folder.

**Solution folders** (added as folders of the same name):
- `build/` (all contents)
- `build/local/` (all contents)

**Solution Items** (root folder):
- `codeCoverage.runsettings`
- `global.json`
- `NuGet.config`
- `version.props`

If files are added or removed, the solution file must be updated. When starting fresh, get the dev team's latest `.sln` and repack it with the DevOps additions.

#### Visual Studio version

The `# Visual Studio Version` header in the `.sln` file reflects which VS version last saved it:

| Value | Visual Studio version |
|---|---|
| `17` | Visual Studio 2022 |
| `18` | Visual Studio 2026 |

New **.NET 10** projects should be opened and saved with **VS 2026** to write version `18`. For existing projects, devs handle this — no DevOps action required.

#### `.slnx` format

Microsoft introduced a new XML-based `.slnx` format (VS 2022 17.14+, .NET SDK 9.0.200+ required to build). It is significantly cleaner and produces fewer merge conflicts. Some dev teams are beginning to adopt it. Solution file format is the dev team's decision — this is noted for awareness only.

### `TUPSS.snk`

A common signing key file shared across all projects. Copy it from any existing repo — do not generate a new one.

---

## build/ Files

### `sources.props`

Standard NuGet restore sources. Must include both internal ADO feed and JFrog.

```xml
<Project>
  <PropertyGroup>
    <RestoreSources>
      https://pkgs.dev.azure.com/theupsstore/_packaging/tupss-feed/nuget/v3/index.json;
      https://jfrog.inside.ups.com/artifactory/api/nuget/v3/nuget-remote/index.json;
    </RestoreSources>
  </PropertyGroup>
</Project>
```

### `azure-pipeline.yml`

Main CI/CD pipeline. Covers `main` and `dev-lane2` branches. See [stage-design.md](./stage-design.md) for trigger and condition standards.

### `azure-pipeline-hotfix.yml`

Hotfix pipeline. Covers the `qa-lane2` branch only. See [stage-design.md](./stage-design.md).

---

## build/local/ Files

One file per environment. Each defines the environment name, server tags, and test pool.

```yaml
# Example: dev.yml
variables:
  Environment: 'TUPSS-CM-<PROJECT>-APP-DEV'
  Tags: 'APP71,APP72'
  TestPool: 'TUPSS-TestPool-Agent'
```

| File | Branch trigger | Notes |
|---|---|---|
| `dev.yml` | `main` (non-PR) | |
| `dev-lane2.yml` | `dev-lane2` | |
| `test.yml` | `main` (non-PR) | |
| `staging.yml` | `main` | Uses staging tags and test pool |
| `prod.yml` | `main` (`IsFinalBuild = true`) | |
| `qa-lane2.yml` | `qa-lane2` (hotfix pipeline) | |

Host files are only required when the project has integration tests. See [host-file-resolution.md](./host-file-resolution.md).

---

## Pipeline Types (TODO)

This doc currently only covers standard .NET application pipelines (API/Function App/App Service — Framework or Core). We don't yet have a formal, agreed-upon list of pipeline types across the team; other project shapes (Database, Terraform, Angular, mobile/Android, NuGet-package-only) need their own required-files reference.

> **TODO:** Define the full list of pipeline types and give each its own required-files section (or its own doc). Candidates raised so far, not yet confirmed: Database, Android/Mobile, .NET (Framework/Core, API/FunctionApp/AppService variants), NuGet-package-only, Terraform, Angular.
