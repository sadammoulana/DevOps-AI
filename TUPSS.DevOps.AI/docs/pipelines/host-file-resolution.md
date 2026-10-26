# Host File Resolution

How the pipeline determines which hosts file to apply during deployment. Required only for projects with integration tests.

---

## Contents

- [Host File Resolution](#host-file-resolution)
  - [Contents](#contents)
  - [Overview](#overview)
  - [Path 1: Standard Single-Server Environments](#path-1-standard-single-server-environments)
    - [Configuration](#configuration)
    - [Behavior](#behavior)
  - [Path 2: Multi-Server Environments (Tag-Based)](#path-2-multi-server-environments-tag-based)
    - [Step 1 — Tags in `env.yml`](#step-1--tags-in-envyml)
    - [Step 2 — Tag → File Mapping (`HostFiles` variable group)](#step-2--tag--file-mapping-hostfiles-variable-group)
    - [Step 3 — YAML Iteration](#step-3--yaml-iteration)
    - [Step 4 — Two-Phase Variable Evaluation](#step-4--two-phase-variable-evaluation)
    - [Step 5 — File Application](#step-5--file-application)
  - [Resolution Flow Summary](#resolution-flow-summary)

---

## Overview

Hosts files are maintained centrally in `TUPSS.DevOps.CICD` under `/Hosts`. The pipeline copies the appropriate file to `C:\Windows\System32\drivers\etc\hosts` on the target server during deployment.

There are two resolution paths depending on whether a stage targets one server or many.

---

## Path 1: Standard Single-Server Environments

### Configuration

Each `local/env.yml` defines the hosts file directly:

```yaml
HostFile: 'name-of-file'
```

### Behavior

`HostFile` is passed into the deployment template as `HostsFileName`. The PowerShell step copies:

```
/Hosts/name-of-file → C:\Windows\System32\drivers\etc\hosts
```

---

## Path 2: Multi-Server Environments (Tag-Based)

Used for environments such as **staging** and **production**, where the same stage deploys to multiple servers in parallel.

### Step 1 — Tags in `env.yml`

```yaml
Tags: 'SERVER1,SERVER2'
```

Tags are logical identifiers, not file names.

### Step 2 — Tag → File Mapping (`HostFiles` variable group)

```text
SERVER1 = hosts-staging-app01
SERVER2 = hosts-staging-app02
```

Each tag resolves to a real file under `/Hosts`.

### Step 3 — YAML Iteration

The stage iterates over tags:

```yaml
each Tag in split(variables.Tags, ',')
```

For each tag, the template receives:

```yaml
HostsFileName: '$(${{ Tag }})'
```

### Step 4 — Two-Phase Variable Evaluation

This relies on ADO's compile-time and runtime evaluation order.

| Phase | Expression | Result |
|---|---|---|
| Compile time | `${{ Tag }}` | `SERVER1` |
| Runtime | `$(SERVER1)` | `hosts-staging-app01` (from variable group) |

### Step 5 — File Application

```
/Hosts/hosts-staging-app01 → C:\Windows\System32\drivers\etc\hosts
```

Repeats once per tag.

---

## Resolution Flow Summary

```
Tags (env.yml)
  ↓
SERVER1,SERVER2
  ↓ iteration
Tag = SERVER1
  ↓ compile-time expansion
$(SERVER1)
  ↓ variable group lookup
hosts-staging-app01
  ↓ filesystem copy
/Hosts/hosts-staging-app01
```
