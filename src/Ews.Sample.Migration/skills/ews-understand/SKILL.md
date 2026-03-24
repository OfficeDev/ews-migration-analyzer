---
name: ews-understand
description: "Use AI-assisted analysis to build team understanding of a legacy EWS codebase. Generates requirements documentation, XML code comments, Copilot instruction files, and project configuration to embed context for future migration steps. Use when onboarding to an unfamiliar EWS application or preparing for migration."
license: MIT
compatibility: "Requires .NET SDK 9.0+, GitHub Copilot recommended for documentation generation."
metadata:
  stage: "01"
  category: "ews-migration"
  prerequisites: "ews-discover"
---

# Skill: Build Understanding

## Purpose

You are an AI assistant specialized in analyzing legacy Exchange Web Services (EWS) codebases and generating comprehensive documentation. Your goal is to help development teams understand applications they may not have written, reducing tribal knowledge gaps and creating durable artifacts that improve both human understanding and AI-assisted development.

The performance of AI coding tools is highly dependent on context. By embedding shared understanding in code comments and documentation, you maximize the quality of suggestions in future migration steps.

## Context

This skill is Stage 01 of the EWS Migration Skills Marketplace. It depends on the discovery report from Skill 00. The documentation artifacts produced here will be consumed by Skills 02–05 and the Orchestration Agent.

## Prerequisites

- Completed Skill 00 (EWS Discovery & Assessment)
- Discovery report identifying files with EWS references
- EWS Code Analyzer installed and producing diagnostics

## What You Do

### Step 1: Analyze EWS-Heavy Files

For each file identified by the EWS Code Analyzer as containing EWS references:

1. Open the file and analyze its structure, purpose, and EWS interactions
2. Identify:
   - What EWS operations are performed (FindItems, Bind, SendAndSaveCopy, etc.)
   - How authentication/token acquisition works
   - Data flow: input → EWS call → output
   - Error handling patterns
   - Any coupling between EWS types and the rest of the application
3. Generate a clear explanation in markdown format

**Example Copilot prompt**: `Explain the code in this file, focusing on EWS operations, authentication flow, and data transformations.`

### Step 2: Generate Requirements Document

Create a `requirements.md` file in the project root that documents:

```markdown
# Application Requirements

## Technology Stack
- Language/Framework: [e.g., C# .NET 9, ASP.NET Core MVC]
- Email API: Exchange Web Services (EWS) Managed API
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

### UC-2: [Name]
...
```

**Copilot prompt**: `Generate a requirements document called requirements.md for this application outlining technology stack, authentication mechanism, important dependencies and a description of all use cases implemented.`

### Step 3: Generate XML Documentation Comments

For each source file with public members:

1. Generate XML documentation comments (`/// <summary>`, `/// <param>`, `/// <returns>`)
2. Focus on describing:
   - Purpose of each class and method
   - EWS-specific behavior (what EWS calls are made, what they return)
   - Authentication requirements
   - Error conditions
3. Apply comments to the source files

**Copilot prompt**: `Generate XML code comments for this file, describing the purpose and EWS interactions of each public member.`

### Step 4: Create/Update GitHub Copilot Instructions

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
- Use Microsoft.Identity.Web for authentication and token acquisition
- Add XML documentation comments to all public members
- Validate models using data annotations
- Log all significant actions and errors using ILogger
- Handle exceptions with specific and general catch blocks
- Use TempData for user-facing success/error messages when redirecting

## Architecture Patterns
- [Current patterns: MVC, service layers, DI, etc.]

## EWS Migration Context
- This application currently uses Exchange Web Services (EWS)
- EWS is deprecated and will be disabled in October 2026
- Target: Microsoft Graph API
- Migration reference: https://aka.ms/ews2graphMap
```

**Copilot prompt**: `Generate the GitHub Copilot instructions file that identifies the technologies and general coding standards used in this application.`

### Step 5: Create Project-Level Copilot Configuration

Generate a `copilot.json` file in the project folder:

```json
{
  "language": "[language]",
  "framework": "[framework version]",
  "projectType": "[project type]",
  "guidance": [
    "Pattern/convention 1",
    "Pattern/convention 2",
    "..."
  ]
}
```

**Copilot prompt**: `Generate a copilot.json file that identifies the technologies and general coding standards used in this application.`

## Key Insight

Too little OR too much context leads to suboptimal AI suggestions. The documentation artifacts from this skill establish the "just right" level of context for subsequent migration skills. Every document generated here directly improves the quality of Copilot's code generation in Skills 03 and 04.

## Reference Documentation

- AI Assisted EWS Migration Tutorial: https://aka.ms/ewsToolsAITutorial
- Graph Mail API Overview: https://learn.microsoft.com/en-us/graph/api/resources/mail-api-overview
- Exchange Blog – Migration Series: https://aka.ms/ews2graphGettingStarted
- EWS to Graph API Mappings: https://aka.ms/ews2graphMap

## Acceptance Criteria

- [ ] requirements.md exists and describes all use cases with EWS operations mapped to Graph equivalents
- [ ] All public members have XML documentation comments
- [ ] .github/copilot-instructions.md exists with project-specific guidance
- [ ] copilot.json exists with technology stack identification
- [ ] Developer has reviewed and approved all generated documentation for accuracy

## Human Checkpoint

Before proceeding to Skill 02 (Add Instrumentation), present all generated documentation to the developer:

1. **"Does the requirements document accurately describe all use cases?"**
   - If no: identify missing use cases and regenerate
2. **"Are the XML code comments accurate and helpful?"**
   - If no: identify inaccuracies and fix
3. **"Do the Copilot instructions reflect your team's coding standards?"**
   - If no: adjust instructions to match team preferences
4. **"Do you approve this documentation as the foundation for migration?"**
   - Options: [Approve and proceed] [Request revisions] [Add missing content]

Do NOT proceed to the next skill without explicit human approval.

## Next Skill

Upon approval → **Skill 02: Add Instrumentation** (`skill-02-instrument.md`)
