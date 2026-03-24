# Tutorial - Migrating from EWS to Microsoft Graph API with GitHub Copilot

## Overview

EWS has been identified as a security vulnerability and will be disabled in October 2026 for Exchange Online (EWS will continue to work for Exchange Server on-premises). This means all applications using EWS on Exchange Online must either be sunset or migrated to a supported API, such as Microsoft Graph API.

The many applications using EWS have been built a over the past 20 years reflect a variety of design patterns and coding practices. It is highly likely that the applications weren't written by the teams now responsible for modernizing them. Documentation and automated tests may be lacking or out of date. All of these factors can make getting started a challenge and upgrading the applications a chore.

Fortunately, there are tools and techniques that can help build the understanding of legacy applications in your team's portfolio and accelerate the migration of those applications to a supported platform.

This folder contains a simple mail application built with ASP.NET MVC that uses Exchange Web Services (EWS) to view and reply to recent emails for an authenticated M365 user. It serves as the baseline for the migration.

While all the migration steps described in this sample can be performed manually, we'll make heavy use of GitHub Copilot to accelerate the process and improve the code quality as we go.

We hope that learning about using AI tools like GitHub Copilot on what would otherwise be a tedious process will benefit your teams twofold by eliminating a dangerous security vulnerability and set you up for success with AI tools on future projects.

## Table of Contents

- [Getting Started](#getting-started)
- [Prerequisites](#prerequisites)
- [Setting the Baseline](./00-Baseline/README.md)
- [Build Understanding with GitHub Copilot](./01-Build_Understanding/README.md)
- [Adding Instrumentation with .NET Aspire](./02-Add_Instrumentation/README.md)
- [Adding Unit Tests with xUnit and NSubstitute](./03-Add_Tests/README.md)
- [Refactor for Modularity with and remove EWS](./04-Refactor/README.md)
- [Copilot Skills Marketplace](#copilot-skills-marketplace)
- [Orchestration Agent](./skills/ews-migration-orchestrator/SKILL.md)

## Copilot Skills Marketplace

In addition to the step-by-step tutorial, this folder includes an **EWS Migration Skills Marketplace** built to the [Agent Skills](https://agentskills.io) open standard — a set of Copilot skills and an orchestration agent that can automate and guide the migration end-to-end with human-in-the-loop checkpoints.

### Available Skills

Each skill follows the [agentskills.io specification](https://agentskills.io/specification): a directory containing a `SKILL.md` with YAML frontmatter and markdown instructions, plus optional `references/` for supplementary documentation.

| Skill | Directory | Stage | Description | Complexity |
|-------|-----------|-------|-------------|------------|
| [EWS Discovery & Assessment](./skills/ews-discover/SKILL.md) | `ews-discover/` | 00 | Identify all EWS usage in your tenant and codebase using EWS Usage Reports and the Code Analyzer | Low |
| [Build Understanding](./skills/ews-understand/SKILL.md) | `ews-understand/` | 01 | AI-assisted analysis to generate requirements docs, code comments, and Copilot instructions | Low |
| [Add Instrumentation](./skills/ews-instrument/SKILL.md) | `ews-instrument/` | 02 | Add .NET Aspire observability for runtime monitoring during migration | Medium |
| [Add Tests](./skills/ews-test/SKILL.md) | `ews-test/` | 03 | Generate xUnit/NSubstitute unit tests as a safety net for refactoring | Medium |
| [Refactor & Migrate to Graph API](./skills/ews-refactor/SKILL.md) | `ews-refactor/` | 04 | Extract service layer, implement Graph API, validate, and remove EWS (3 sub-phases) | High |
| [Final Validation & Documentation](./skills/ews-validate/SKILL.md) | `ews-validate/` | 05 | Post-migration validation, documentation update, and completion report | Low |

### Orchestration Agent

The [Migration Orchestrator Agent](./skills/ews-migration-orchestrator/SKILL.md) guides you through each skill in sequence. It:

- Assesses current migration state and supports resuming from any point
- Presents relevant Microsoft documentation at each stage
- Enforces human approval gates before and after every skill
- Handles parity gaps where Graph API equivalents are not yet available
- Generates a final migration summary when all skills complete

### Installing as a Plugin

The skills are registered as a plugin marketplace in [`.claude-plugin/marketplace.json`](../../.claude-plugin/marketplace.json). To install in a compatible agent:

```
/plugin marketplace add OfficeDev/ews-migration-analyzer
```

### Using the Skills

Each skill is a self-contained directory following the [Agent Skills](https://agentskills.io) format. Skills can be:

1. **Orchestrated**: Use the orchestrator agent for end-to-end guided migration
2. **Invoked individually**: Use any skill independently for a specific migration phase
3. **Customized**: Adapt skill prompts to your application's specific needs

### Skill Directory Structure

```
skills/
├── ews-discover/                     # Stage 00
│   ├── SKILL.md                      # Frontmatter + instructions
│   └── references/REFERENCE.md       # Microsoft documentation links
├── ews-understand/                   # Stage 01
│   ├── SKILL.md
│   └── references/REFERENCE.md
├── ews-instrument/                   # Stage 02
│   ├── SKILL.md
│   └── references/REFERENCE.md
├── ews-test/                         # Stage 03
│   ├── SKILL.md
│   └── references/REFERENCE.md
├── ews-refactor/                     # Stage 04
│   ├── SKILL.md
│   └── references/
│       ├── REFERENCE.md
│       └── ews-graph-mapping.md      # EWS→Graph operation mapping
├── ews-validate/                     # Stage 05
│   ├── SKILL.md
│   └── references/
│       ├── REFERENCE.md
│       └── migration-report-template.md
└── ews-migration-orchestrator/       # Orchestrator
    ├── SKILL.md
    └── references/REFERENCE.md
```

### Key Resources

- [Deprecation of EWS in Exchange Online](https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/deprecation-of-ews-exchange-online)
- [EWS to Graph API Mappings](https://aka.ms/ews2graphMap)
- [Midnight Blizzard Security Incident](https://aka.ms/mblizz)
- [EWS Migration Tools](https://aka.ms/ewsTools)
- [Agent Skills Specification](https://agentskills.io/specification)

## Getting Started

### Prerequisites

1. Ensure you have the prerequisites installed:
   - .NET SDK (version 9.0 or later)
   - Visual Studio 2022 or later
   - Microsoft Exchange Online account for testing
   - Access to Microsoft Entra ID (Azure AD) for app registration
1. Create an app registration in Entra ID with the following settings:
    - Delegated Graph API permissions (grant admin consent if you can):
       - `EWS.AccessAsUser.All`
       - `User.Read`
    - Redirect URIs for platform Web:
       - `http://localhost:5024/signin-oidc` (port may vary based on your setup)
       - `https://localhost:7020/signin-oidc` (port may vary based on your setup)
1. Take note of the following values from the app registration:
   - Application (client) ID
   - Directory (tenant) ID

We'll revisit this app registration later to add the Microsoft Graph API permissions needed for migration.
