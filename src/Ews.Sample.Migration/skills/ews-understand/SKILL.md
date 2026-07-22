---
name: ews-understand
description: "Use AI-assisted analysis to build team understanding of a legacy EWS codebase. Generates requirements documentation, a code-derived migration seam specification, XML code comments, Copilot instruction files, and project configuration to embed context for future migration steps. Use when onboarding to an unfamiliar EWS application or preparing for migration."
license: MIT
compatibility: "Requires .NET SDK 9.0+, GitHub Copilot recommended for documentation generation."
metadata:
  stage: "01"
  category: "ews-migration"
  prerequisites: "ews-discover"
---

# Skill: Build Understanding

## Purpose

You are an AI assistant specialized in analyzing legacy Exchange Web Services (EWS) codebases and generating comprehensive documentation. Your goal is to help development teams understand applications they may not have written, reduce tribal knowledge gaps, and derive the exact migration seams already implied by the code so later stages can refactor against the application's real abstractions instead of generic examples.

The performance of AI coding tools is highly dependent on context. By embedding shared understanding in code comments and documentation, you maximize the quality of suggestions in future migration steps.

## Context

This skill is Stage 01 of the EWS Migration Skills Marketplace. It depends on the discovery report from Skill 00. The documentation artifacts produced here will be consumed by Skills 02-05 and the Orchestration Agent.

## Prerequisites

- Completed Skill 00 (EWS Discovery & Assessment)
- Discovery report identifying files with EWS references
- EWS Code Analyzer installed and producing diagnostics

## Generalization Rule

Do not assume the application is a simple mail client and do not hardcode example abstractions such as `IEmailService`, `EmailMessage`, or a fixed set of Graph operations.

Instead, derive the migration seam from the actual EWS implementation in the codebase:

- Infer the business capabilities implemented on top of EWS
- Infer the service boundaries already present or most naturally extractable from the code
- Infer the domain models needed by the rest of the application after EWS types are removed
- Infer the matching Microsoft Graph implementation strategy for each capability

Every artifact produced by this skill must reflect names, method shapes, models, and flows that come from the application's code, not from this skill document.

## What You Do

### Step 1: Analyze EWS-Heavy Files and Trace Real Boundaries

For each file identified by the EWS Code Analyzer as containing EWS references:

1. Open the file and analyze its structure, purpose, and EWS interactions
2. Identify:
   - What EWS operations are performed (FindItems, Bind, SendAndSaveCopy, etc.)
   - How authentication/token acquisition works
   - Data flow: input -> EWS call -> output
   - Error handling patterns
   - Any coupling between EWS types and the rest of the application
   - Which public actions, handlers, services, jobs, or workflows depend on those EWS calls
   - Which non-EWS types are already acting as stable boundaries
   - Which EWS types or DTOs are leaking into controllers, UI models, background jobs, or external contracts
3. Generate a clear explanation in markdown format

**Example Copilot prompt**: `Explain the code in this file, focusing on EWS operations, authentication flow, and data transformations.`

### Step 2: Derive the Migration Seam from Code

Create a `migration-seams.md` file in the project root that documents the exact abstractions the later migration should implement.

The file must be derived from the application's current EWS implementation and should answer:

- What business capabilities currently depend on EWS?
- What service interfaces should exist after extraction, based on those capabilities?
- What methods should each interface expose, with exact names and signatures inferred from the calling code when possible?
- What domain models should replace EWS types, and which fields are actually required by the rest of the application?
- What Graph endpoints, SDK calls, batching strategy, pagination behavior, and error handling should back each method?
- What transformation rules are required to map EWS objects to application-owned models and then to Graph objects?

Use a structure like this:

```markdown
# Migration Seams

## Capability Inventory
### Capability: [Derived capability name]
- Calling code: [files/classes/actions/jobs]
- Current EWS operations: [EWS methods/types used]
- Authentication context: [delegated/app-only, token source]
- Side effects: [writes, sends, deletes, attachments, notifications]

## Candidate Service Interfaces
### Interface: [Derived interface name]
- Why this boundary exists in the current code
- Consumers: [callers]
- Methods:
  - `[ExactMethodNameAsync](input...) -> output`
  - Notes on pagination, filtering, concurrency, cancellation, retries

## Domain Models
### Model: [Derived model name]
- Replaces: [EWS type or mixed DTO]
- Required properties actually used by the app
- Optional properties that can remain implementation-specific
- Validation and invariants implied by existing code

## Graph Implementation Mapping
### Method: [Interface].[Method]
- Current EWS implementation: [file/class/method]
- Graph equivalent: [endpoint/SDK path]
- Request shape: [select/filter/top/orderby/body]
- Response mapping: [Graph fields -> domain model]
- Gaps or risks: [parity, throttling, preview APIs, missing features]

## Extraction Plan
- Minimal refactoring order to introduce the interface and models safely
- Which existing classes should become the first EWS-backed implementation
- Which tests should be added or updated before replacing EWS with Graph
```

Rules for this artifact:

- Prefer names already present in the codebase over inventing generic names
- If multiple EWS workflows exist, derive multiple interfaces and models as needed
- If a single interface would be too broad, split it by capability based on actual callers
- Do not include properties or methods that are not justified by current usage
- Record uncertainty explicitly when the code does not prove a detail

**Copilot prompt**: `Analyze the current EWS implementation and derive the exact service interfaces, domain models, and Graph implementation mappings this application needs. Create migration-seams.md using names, method shapes, and fields justified by the code rather than generic examples.`

### Step 3: Generate Requirements Document

Create a `requirements.md` file in the project root that documents:

```markdown
# Application Requirements

## Technology Stack
- Language/Framework: [e.g., C# .NET 9+, ASP.NET Core MVC]
- Email API: Exchange Web Services (EWS) Managed API or other EWS integration surface identified in code
- Authentication: [e.g., Microsoft.Identity.Web, OAuth2, OpenID Connect]
- Additional frameworks: [e.g., .NET Aspire, Razor Pages]

## Authentication Mechanism
- Provider: [e.g., Azure AD / Microsoft Entra ID]
- Flow: [e.g., OAuth2 Authorization Code with PKCE]
- Token scope: [e.g., https://outlook.office365.com/.default]
- Libraries: [e.g., Microsoft.Identity.Web v2.19.0]

## Important Dependencies
| Package | Version | Purpose |
|---------|---------|---------|
| Microsoft.Exchange.WebServices | x.x.x | EWS API access |
| Microsoft.Identity.Web | x.x.x | Authentication |
| ... | ... | ... |

## Use Cases
### UC-1: [Name]
- **Description**: [What the user can do]
- **Input**: [What triggers this use case]
- **Process**: [Step-by-step flow including EWS operations]
- **Output**: [What the user sees/gets]
- **EWS Operations**: [Specific EWS methods called]
- **Graph API Equivalent**: [Corresponding Graph API calls, from discovery report]
- **Migration Seam**: [Interface method and domain model from migration-seams.md that should own this use case]

### UC-2: [Name]
...

## Derived Migration Summary
- Primary service interfaces to extract: [from migration-seams.md]
- Domain models to own in the application layer: [from migration-seams.md]
- Graph parity risks and special handling: [from migration-seams.md]
```

**Copilot prompt**: `Generate a requirements document called requirements.md for this application outlining technology stack, authentication mechanism, important dependencies, all implemented use cases, and the code-derived migration seams each use case depends on.`

### Step 4: Generate XML Documentation Comments

For each source file with public members:

1. Generate XML documentation comments (`/// <summary>`, `/// <param>`, `/// <returns>`)
2. Focus on describing:
   - Purpose of each class and method
   - EWS-specific behavior (what EWS calls are made, what they return)
   - Authentication requirements
   - Error conditions
3. Apply comments to the source files

**Copilot prompt**: `Generate XML code comments for this file, describing the purpose and EWS interactions of each public member.`

### Step 5: Create/Update GitHub Copilot Instructions

Generate a `.github/copilot-instructions.md` file in the solution root:

```markdown
# GitHub Copilot Instructions

## Project Overview
[Brief description of the application]

## Technology Stack
- [Language, framework, versions]

## Coding Standards
- Use async/await for all I/O and service calls
- Use dependency injection for configuration, logging, and authentication
- Use Microsoft.Identity.Web for authentication and token acquisition when present in this solution
- Add XML documentation comments to all public members
- Validate models using data annotations when the application already uses them
- Log all significant actions and errors using ILogger or the project's logging abstraction
- Handle exceptions with specific and general catch blocks
- Preserve the application's existing presentation and flow conventions

## Architecture Patterns
- [Current patterns: MVC, service layers, DI, etc.]

## Derived Migration Seams
- Prefer the service interfaces documented in `migration-seams.md`
- Prefer application-owned domain models documented in `migration-seams.md`
- Do not reintroduce EWS SDK types outside the EWS-backed implementation
- When generating Graph code, match the method contracts and model fields justified by existing callers

## EWS Migration Context
- This application currently uses Exchange Web Services (EWS)
- EWS is deprecated and will be disabled in October 2026
- Target: Microsoft Graph API
- Migration reference: https://aka.ms/ews2graphMap
```

**Copilot prompt**: `Generate the GitHub Copilot instructions file that identifies the technologies, coding standards, and code-derived migration seams used in this application.`

### Step 6: Create Project-Level Copilot Configuration

Generate a `copilot.json` file in the project folder:

```json
{
  "language": "[language]",
  "framework": "[framework version]",
  "projectType": "[project type]",
  "guidance": [
    "Pattern/convention 1",
    "Pattern/convention 2",
    "Use the interfaces and models derived in migration-seams.md instead of generic examples",
    "..."
  ]
}
```

**Copilot prompt**: `Generate a copilot.json file that identifies the technologies, coding standards, and code-derived migration seams used in this application.`

## Key Insight

Too little OR too much context leads to suboptimal AI suggestions. The documentation artifacts from this skill establish the "just right" level of context for subsequent migration skills. The critical output is not just generic documentation, but a code-derived seam specification that tells later skills exactly which interfaces, models, and Graph implementations to create for this application.

## Reference Documentation

- AI Assisted EWS Migration Tutorial: https://aka.ms/ewsToolsAITutorial
- Graph Mail API Overview: https://learn.microsoft.com/en-us/graph/api/resources/mail-api-overview
- Exchange Blog - Migration Series: https://aka.ms/ews2graphGettingStarted
- EWS to Graph API Mappings: https://aka.ms/ews2graphMap

## Acceptance Criteria

- [ ] migration-seams.md exists and derives service interfaces, domain models, and Graph implementation mappings from the current code
- [ ] requirements.md exists and describes all use cases with EWS operations mapped to Graph equivalents and linked migration seams
- [ ] All public members have XML documentation comments
- [ ] .github/copilot-instructions.md exists with project-specific guidance
- [ ] copilot.json exists with technology stack identification
- [ ] Developer has reviewed and approved all generated documentation for accuracy

## Human Checkpoint

Before proceeding to Skill 02 (Add Instrumentation), present all generated documentation to the developer:

1. **"Does migration-seams.md accurately capture the real service boundaries, models, and Graph mappings implied by your code?"**
   - If no: identify the mismatched interface, model, or Graph mapping and regenerate from the relevant code paths
2. **"Does the requirements document accurately describe all use cases?"**
   - If no: identify missing use cases and regenerate
3. **"Are the XML code comments accurate and helpful?"**
   - If no: identify inaccuracies and fix
4. **"Do the Copilot instructions reflect your team's coding standards?"**
   - If no: adjust instructions to match team preferences
5. **"Do you approve this documentation as the foundation for migration?"**
   - Options: [Approve and proceed] [Request revisions] [Add missing content]

Do NOT proceed to the next skill without explicit human approval.

## Next Skill

Upon approval -> **Skill 02: Add Instrumentation** (`skill-02-instrument.md`)
