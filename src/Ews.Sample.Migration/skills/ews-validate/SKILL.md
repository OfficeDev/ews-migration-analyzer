---
name: ews-validate
description: "Post-migration validation and documentation for applications migrated from EWS to Microsoft Graph API. Confirms zero EWS references, validates all tests pass, generates updated documentation, and produces a migration completion report. Use after completing EWS-to-Graph migration to verify and document the result."
license: MIT
compatibility: "Requires .NET SDK 9.0+, EWS Code Analyzer NuGet package for final verification."
metadata:
  stage: "05"
  category: "ews-migration"
  prerequisites: "ews-refactor"
---

# Skill: Final Validation & Documentation

## Purpose

You are an AI assistant specialized in post-migration validation and documentation for applications that have migrated from Exchange Web Services (EWS) to Microsoft Graph API. Your goal is to verify the migration is complete, generate updated documentation reflecting the new architecture, and produce a migration completion report.

This skill closes the migration loop — it confirms that the application is fully migrated, documents the new state, and optionally guides the admin to disable EWS access.

## Context

This skill is **Stage 05** of the EWS Migration Skills Marketplace. It is the final skill before the orchestrator declares migration complete. It depends on a fully migrated application from Skill 04 with all tests passing and zero EWS references.

## Prerequisites

- Completed Skill 04 (Refactor & Migrate to Graph API)
- All unit tests passing
- EWS Code Analyzer confirms zero EWS references
- Application running successfully with Graph API

---

## What You Do

### Step 1: Final EWS Verification

Run the EWS Code Analyzer one final time to confirm zero EWS references:

1. Ensure `Ews.Analyzer` NuGet package is referenced (add temporarily if removed)
2. Build the solution: `dotnet build`
3. Review diagnostics:
   - **EWS004** should report: "Found 0 references"
   - **EWS005** should NOT appear (no call to action needed)
   - No EWS001, EWS002, EWS003, or EWS000 diagnostics
4. If any EWS references remain: go back to Skill 04 Phase 4c

### Step 2: Full Test Suite Validation

Run all tests and confirm 100% pass rate:

```shell
dotnet test --verbosity normal
```

- Document the final test count
- Confirm no test failures
- Note: test count should be lower than Skill 03 baseline (EWS-specific tests were removed in Skill 04)

### Step 3: Runtime Validation

Start the application under Aspire and manually verify all use cases from requirements.md:

For each use case documented in requirements.md:

- [ ] **UC-1**: [Use case name] — Verify via Graph API
- [ ] **UC-2**: [Use case name] — Verify via Graph API
- [ ] ... (enumerate all)

Check Aspire dashboard:

- [ ] No error traces
- [ ] All requests returning 200 OK
- [ ] Graph API calls visible in traces
- [ ] No EWS-related log entries

### Step 4: Update Requirements Document

Regenerate or update `requirements.md` to reflect the new architecture:

**Changes to make**:

- **Technology Stack**: Replace "EWS Managed API" with "Microsoft Graph SDK"
- **Authentication**: Update token scopes from `https://outlook.office365.com/.default` to `Mail.Read Mail.ReadWrite Mail.Send`
- **Dependencies**: Remove EWS packages, add Graph packages
- **Use Cases**: Update process descriptions to reference Graph API operations instead of EWS
- **Architecture**: Add service layer description (IEmailService → GraphEmailService)

**Copilot prompt**: `Update requirements.md to reflect the migration from EWS to Microsoft Graph API. Replace all EWS references with Graph API equivalents.`

### Step 5: Update README

Update the project README.md with:

- New architecture description
- Updated prerequisites (Graph API permissions instead of EWS)
- Updated setup instructions (Graph scopes, Entra app registration)
- Migration completion note

### Step 6: Update Copilot Instructions for Ongoing Development

Update `.github/copilot-instructions.md` and `copilot.json`:

- Remove all EWS-related guidance
- Add Graph API as the primary email integration
- Update coding standards for Graph SDK patterns
- Add guidance for handling Graph API throttling
- Add guidance for Graph API error handling (ServiceException)

**Copilot prompt**: `Update copilot-instructions.md to remove all EWS references and focus on Microsoft Graph API best practices for ongoing development.`

### Step 7: Generate Migration Completion Report

Create a `migration-report.md` documenting the entire migration.

Use the [Migration Report Template](references/migration-report-template.md) to generate the completion report.

### Step 8: Optional — Disable EWS Access

Guide the admin to disable EWS for the migrated application:

1. Review instructions at https://aka.ms/EWSEnabledChange
2. Options:
   - **Organization level**: Disable EWS for entire tenant (if all apps migrated)
   - **User level**: Disable EWS for specific users/service principals
3. **Warning**: Only disable after confirming ALL applications in the tenant have migrated
4. This step requires admin privileges and should be done carefully

---

## Reference Documentation

- How to Disable EWS at Org/User Level: https://aka.ms/EWSEnabledChange
- EWS Deprecation Timeline: https://techcommunity.microsoft.com/blog/exchange/exchange-online-ews-your-time-is-almost-up/4492361
- EWS Migration Tools: https://aka.ms/ewsTools
- Graph Mail API Overview: https://learn.microsoft.com/en-us/graph/api/resources/mail-api-overview

---

## Acceptance Criteria

- [ ] EWS Code Analyzer reports zero diagnostics
- [ ] Full test suite passes (all tests green)
- [ ] All use cases verified working via Graph API
- [ ] requirements.md updated to reflect Graph API
- [ ] README.md updated with new architecture
- [ ] copilot-instructions.md updated for Graph-based development
- [ ] copilot.json updated to remove EWS guidance
- [ ] Migration completion report generated (migration-report.md)
- [ ] Application meets all original use cases via Graph API

---

## Human Checkpoint

Present the migration completion report to the developer and stakeholders:

1. **"Does the migration report accurately capture all changes made?"**
   - If no: update the report with missing information
2. **"Have all original use cases been verified working via Graph API?"**
   - If no: identify failing use cases and investigate
3. **"Is the updated documentation accurate?"**
   - If no: revise documentation
4. **"Are you ready to disable EWS access for this application?"**
   - Options: [Yes — disable EWS] [Not yet — other apps still need EWS] [Skip]
5. **"Do you approve this migration as complete?"**
   - Options: [Approve and close migration] [Request updates] [Identify remaining issues]

Do NOT mark the migration as complete without explicit human approval.

---

## Completion

Upon approval → Migration is **COMPLETE**. The orchestration agent records the final status and generates a summary.
