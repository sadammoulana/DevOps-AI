# TUPSS.DevOps.AI

The TUPSS DevOps team's shared AI tooling — agents, instructions, and skills for Copilot in VS Code and Visual Studio.

## What This Repo Is

This is the centralized home for AI-assisted workflows across the TUPSS DevOps platform. It is not an application — it has no build, no deployable artifact. It makes the team faster and more consistent by packaging shared knowledge into reusable Copilot tooling.

It grows over time. New agents, instruction sets, and skills are added as the team identifies workflows worth automating or standardizing.

## How It Works

Add this repo to a multi-root workspace alongside any consumer repos you are working in. The agents, instructions, and skills defined here activate automatically across all repos in that workspace.

```
MyWorkspace.code-workspace
  ├── TUPSS.DevOps.AI          ← this repo
  ├── TUPSS.Example.Repo1      ← work target
  └── TUPSS.Example.Repo2      ← work target
```

## What's Inside

### Agents
Custom Copilot personas. Switch into an agent for a specific workflow — it brings the right context, tools, and behavior for that task.

| Agent | Purpose |
|---|---|
| `preflight.agent.md` | PR Preflight — reviews staged changes against team standards before a PR is opened |

### Instructions
Team standards automatically applied when editing matching file types.

| File | Applies To |
|---|---|
| `azure-devops.instructions.md` | YAML pipelines, PowerShell, Markdown |
| `terraform.instructions.md` | Terraform / IaC |

### Skills
Packaged, repeatable workflows for a specific task. Invoked on demand rather than always-on.

| Skill | Purpose |
|---|---|
| `terraform-provider-pin` | One-time migration: pins a `TUPSS.*.Terraform` consumer repo to an exact `azurerm` version and generates a committed `.terraform.lock.hcl` |

### Global Context
`copilot-instructions.md` is always loaded — it carries team identity, conventions, and cross-cutting context into every Copilot interaction.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).