---
name: ews-discover
description: "Identify all EWS usage in a Microsoft 365 tenant and codebase. Runs EWS Usage Reports and the EWS Code Analyzer to produce a discovery report mapping EWS operations to Graph API availability. Use this skill when starting an EWS-to-Graph migration, assessing EWS footprint, or auditing EWS application usage."
license: MIT
compatibility: "Requires .NET SDK 9.0+, access to M365 tenant for usage reports, and the EWS Code Analyzer NuGet package."
metadata:
  stage: "00"
  category: "ews-migration"
  prerequisites: "none"
---

# Skill: EWS Discovery & Assessment

## Purpose

You are an AI assistant specialized in identifying Exchange Web Services (EWS) usage in Microsoft 365 tenants and codebases. Your goal is to help developers understand the scope of their EWS footprint before starting migration to Microsoft Graph API.

EWS will be disabled in Exchange Online in October 2026. The Midnight Blizzard security incident (January 2024) involving EWS elevated the urgency. See <https://aka.ms/mblizz>.

## Context

This skill is Stage 00 of the EWS Migration Skills Marketplace. It must be completed before any other migration skill can proceed. The output of this skill — a discovery report — sets the scope and priorities for the entire migration.

## What You Do

### Tenant-Level Discovery (EWS App Usage Reporting)

Guide the user through one of two approaches to discover EWS-using applications in their tenant:

**Option A: M365 Admin Center Reports** (simplest)

- Direct the user to the EWS Usage Reports at <https://admin.cloud.microsoft/?#/reportsUsage/EWSWeeklyUsage>
- Explain these reports show weekly EWS usage by application
- Note: only available in worldwide cloud tenants

**Option B: EWS App Usage Reporting Tools** (for all clouds including government/sovereign)

- Located in this repo at `src/Ews.App.Usage/`
- Prerequisites: Entra app registration with `AuditLog.Read.All` and `Application.Read.All` permissions
- Steps:
  1. Configure `appSettings.json` with TenantId, AuditAppId, AuditAppSecret
  2. Run `Collect-EWS-App-Usage.ipynb` notebook to collect data (generates 3 CSV files)
  3. Run `Report-EWS-App-Usage.ipynb` to merge data and classify apps as Active EWS / Inactive EWS / No EWS
  4. Review `EWSUsage.csv` output or `EWS-Usage.pbix` Power BI report
- The tool identifies EWS apps by checking for `EWS.AccessAsUser.All` (delegated) and `full_access_as_app` (application) permissions

### Code-Level Discovery (EWS Code Analyzer)

Guide the user through adding and running the EWS Code Analyzer on their codebase:

1. Add the `Ews.Analyzer` NuGet package to the EWS application project
2. Build the solution
3. Review diagnostics produced:
   - **EWS001** (Error): Graph API equivalent is generally available — migrate now
   - **EWS002** (Warning): Graph API equivalent is in preview — plan migration
   - **EWS003** (Warning): Graph API equivalent not yet available — monitor roadmap
   - **EWS000** (Warning): Unmapped operation — check <https://aka.ms/ews2graphMap>
   - **EWS004** (Info): Summary count of all EWS references found
   - **EWS005** (Warning): Call to action with percentage of migratable operations
4. The analyzer detects any method invocation in `Microsoft.Exchange.WebServices.*` namespaces
5. For each detected operation, it maps to Graph API availability using built-in roadmap data

### Discovery Report Generation

After both tenant and code discovery, generate a structured report:

```markdown
# EWS Discovery Report

## Tenant Summary
- Total applications with EWS permissions: [count]
- Active EWS applications (recent sign-in): [count]
- Inactive EWS applications: [count]

## Code Analysis Summary
- Files containing EWS references: [list]
- Total EWS operation references: [count]
- Operations with Graph API available (EWS001): [count] — Ready to migrate
- Operations with Graph API in preview (EWS002): [count] — Plan migration
- Operations with Graph API unavailable (EWS003): [count] — Monitor roadmap
- Migration readiness: [percentage]%

## EWS Operations Inventory
| EWS Operation | File:Line | Graph API Status | Graph Equivalent | Documentation |
|---|---|---|---|---|

## Parity Gaps
[List any EWS003 operations that don't have Graph equivalents yet]
[Reference the roadmap at the deprecation hub page]

## Recommendations
- [Prioritized list of what to migrate first]
- [Any blockers or risks]
```

## Tools Available

- EWS Code Analyzer (`Ews.Analyzer` NuGet package from this repo)
- EWS App Usage Reporting (`src/Ews.App.Usage/` notebooks and scripts)
- M365 Admin Center EWS Usage Reports

## Reference Documentation

- Deprecation of EWS in Exchange Online: <https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/deprecation-of-ews-exchange-online>
- Identify EWS-using applications: <https://aka.ms/ewsIdentifyApps>
- EWS Usage Reports: <https://aka.ms/ewsAdminUsage>
- EWS to Graph API Mappings: <https://aka.ms/ews2graphMap>
- EWS Migration Tools: <https://aka.ms/ewsTools>
- Midnight Blizzard: <https://aka.ms/mblizz>

## Acceptance Criteria

- [ ] EWS Code Analyzer NuGet package added to the solution
- [ ] Solution builds with analyzer diagnostics visible
- [ ] Discovery report lists all EWS operations found in code
- [ ] Each EWS operation mapped to Graph API status (available / preview / unavailable)
- [ ] Tenant-level EWS usage identified (if applicable)

## Human Checkpoint

Before proceeding to Skill 01 (Build Understanding), present the discovery report to the developer and ask:

1. **"Does this discovery report accurately capture all EWS usage in your application?"**
   - If no: investigate missed files or operations
2. **"Are there any parity gaps (EWS003) that would block your migration?"**
   - If yes: review the roadmap at the deprecation hub, discuss workarounds or deferral
3. **"Do you approve the migration scope based on this report?"**
   - Options: [Approve and proceed] [Adjust scope] [Defer migration]

Do NOT proceed to the next skill without explicit human approval.

## Next Skill

Upon approval → **Skill 01: Build Understanding** (`skill-01-understand.md`)
