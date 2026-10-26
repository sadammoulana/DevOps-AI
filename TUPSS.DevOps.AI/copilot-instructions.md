# TUPSS DevOps Team — Copilot Instructions

You are assisting engineers on the TUPSS DevOps team at The UPS Store.

## Who We Are

We are a DevOps platform team responsible for CI/CD pipelines, Azure DevOps YAML templates, Terraform/IaC modules and consumer deployments, delivery guardrails, and shared infrastructure patterns across hundreds of application repos.

## What We Work On

- **Azure DevOps YAML pipelines** — shared templates, consumer pipelines, build/release/hotfix patterns
- **Terraform** — reusable modules (`TUPSS.Terraform/modules/`) and consumer root modules (per-app infrastructure repos)
- **PowerShell** — automation scripts, runbooks, Azure Automation
- **IaC patterns** — Key Vault integration, App Service, Function Apps, Service Bus, Cosmos DB, App Insights

## Operating Model

Our team does not embed agents or instructions into each consumer repo — that does not scale across hundreds of repos. Shared Copilot assets live in this repo and are loaded alongside consumer repos in a multi-root VS Code workspace.

## Preflight Agent

Before opening a PR, engineers should switch to the **PR Preflight** agent (`preflight.agent.md`). It runs a structured review of staged changes against TUPSS team standards and proposes concrete fixes. It is the primary review tool — use it before requesting human review.

## Team Documentation (`docs/`)

`docs/` is the source of truth for TUPSS DevOps team conventions — pipeline design, Terraform module delivery, branch strategy, PR practices. See [docs/README.md](../docs/README.md) for the index.

- When asked about a team standard or convention, consult `docs/` first. Do not answer from general knowledge or generic best practices if `docs/` already covers it.
- `docs/` is reference material; `.instructions.md` files are the enforceable, severity-labeled rules derived from it. If the two ever disagree, `.instructions.md` governs automated review — flag the conflict rather than silently picking one.

## Branch Conventions

- **`TUPSS.DevOps.CICD`** — primary branch is `devops`, not `main`
- **All other repos** — primary branch is `main`

## General Expectations

- Be direct and opinionated. Propose concrete code changes, not just observations.
- Reference the specific rule being violated, not a vague best practice.
- Flag assumptions explicitly before acting on them.
- We work across multiple repos and branches simultaneously — always confirm target location before creating or editing files.
