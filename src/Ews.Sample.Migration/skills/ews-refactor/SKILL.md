---
name: ews-refactor
description: "Core EWS-to-Graph migration skill. Extracts a service layer with interface-based design, implements Microsoft Graph API as the email service, validates with feature toggles, and removes all EWS dependencies. Has three sub-phases (extract service layer, implement Graph, remove EWS) each with human approval gates. Use when migrating a .NET EWS application to Microsoft Graph API."
license: MIT
compatibility: "Requires .NET SDK 9.0+, Microsoft.Graph NuGet package, Microsoft.Identity.Web, Entra ID app registration with Graph API permissions."
metadata:
  stage: "04"
  category: "ews-migration"
  prerequisites: "ews-test"
---

# Skill: Refactor & Migrate to Graph API

## Purpose

You are an AI assistant specialized in migrating .NET applications from Exchange Web Services (EWS) to Microsoft Graph API. This is the core migration skill — the most complex and impactful step in the entire journey. You will guide the developer through three distinct sub-phases, each with its own human approval gate:

1. **Phase 4a**: Extract Service Layer — separate concerns through interfaces and dependency injection
2. **Phase 4b**: Implement Graph API — create a Graph-based implementation of the email service
3. **Phase 4c**: Validate & Remove EWS — confirm Graph works, then remove all EWS dependencies

The most important step before replacing a core dependency is separating concerns through service layers and dependency injection. This allows alternative implementations to be created without disrupting the rest of the application.

## Context

This skill is Stage 04 of the EWS Migration Skills Marketplace. It depends on the tested application from Skill 03. The unit tests from Skill 03 are the primary safety net — they must continue to pass after every change. The Aspire dashboard from Skill 02 is the primary debugging tool for runtime issues.

## Prerequisites

- Completed Skills 00-03
- All unit tests passing
- Application running under Aspire with visible telemetry
- requirements.md documenting all use cases
- .github/copilot-instructions.md with testing best practices

## Security Motivation

The Midnight Blizzard security incident (January 2024) involved EWS and elevated the urgency of migrating away from EWS. See <https://aka.ms/mblizz>. Every EWS application in your tenant represents a potential attack surface that will be eliminated by migrating to Graph API.

---

## Phase 4a: Extract Service Layer

### Goal

Separate the email functionality from the controller into an interface-based service layer, making the email implementation swappable.

### Steps

#### 1. Define the IEmailService Interface

Create `Services/IEmailService.cs` with methods that abstract all email operations:

```csharp
public interface IEmailService
{
    /// <summary>Get inbox emails for a user</summary>
    Task<IList<EmailMessage>> GetInboxEmailsAsync(string userEmail, int count = 10);

    /// <summary>Get a specific email by ID</summary>
    Task<EmailMessage?> GetEmailByIdAsync(string emailId, string userEmail);

    /// <summary>Create a reply model for a specific email</summary>
    Task<EmailReplyModel?> CreateReplyModelAsync(string emailId, string userEmail);

    /// <summary>Send a reply to an email</summary>
    Task<bool> SendReplyAsync(EmailReplyModel replyModel, string userEmail);
}
```

**Copilot prompt**: `Create a service layer that separates the business logic and email handling into their own service classes. Use interfaces and dependency injection to make the implementations swappable.`

#### 2. Create Domain Model

Create `Models/EmailMessage.cs` as a domain model that abstracts away EWS-specific types:

```csharp
public class EmailMessage
{
    public string Id { get; set; }
    public string Subject { get; set; }
    public string From { get; set; }
    public string FromName { get; set; }
    public DateTime? DateTimeReceived { get; set; }
    public DateTime? DateTimeSent { get; set; }
    public string Body { get; set; }
    public string BodyPreview { get; set; }
    public bool HasAttachments { get; set; }
    public bool IsRead { get; set; }
    public string Importance { get; set; }
    public List<string> ToRecipients { get; set; }
    public List<string> CcRecipients { get; set; }
}
```

#### 3. Extract EwsEmailService

Create `Services/EwsEmailService.cs` implementing `IEmailService`:

- Move all EWS logic from the controller into this service
- Convert EWS types to domain `EmailMessage` types
- Preserve all authentication and token acquisition logic
- Keep error handling intact

#### 4. Refactor Controller

Update the controller to depend only on `IEmailService`:

- Inject `IEmailService` via constructor
- Replace all direct EWS calls with service method calls
- Remove EWS-specific imports from the controller
- Simplify action methods to delegation pattern

#### 5. Configure Dependency Injection

In `Program.cs`, register the service:

```csharp
builder.Services.AddScoped<IEmailService, EwsEmailService>();
```

#### 6. Validate

- Build the solution — should compile without errors
- Run all existing tests — they should still pass (may need minor updates for new constructor signature)
- Run the application — verify it still works with EWS under the hood

### Human Checkpoint (Phase 4a)

**"The service layer has been extracted. The controller now depends on IEmailService, and EwsEmailService implements it. All existing tests pass. Do you approve this refactoring?"**

- Options: [Approve Phase 4a] [Request changes] [Review code diff]

---

## Phase 4b: Implement Graph API

### Goal

Create a Graph-based implementation of IEmailService and enable switching between EWS and Graph via configuration.

### Steps

#### 1. Add Graph API Best Practices to Copilot Context

Update `.github/copilot-instructions.md`:

```markdown
## Microsoft Graph API Best Practices
- Use the Microsoft Graph SDK (`Microsoft.Graph` NuGet package) for type-safe access
- Handle throttling (HTTP 429) with exponential backoff and retry
- Use `$select` to request only needed properties
- Use `$top` to limit result sets
- Use `$orderby` for server-side sorting
- Batch multiple requests when possible using `$batch` endpoint
- Use delegated permissions for user-context operations
- Required scopes for mail: Mail.Read, Mail.ReadWrite, Mail.Send
- Handle `ServiceException` from Graph SDK
- Use `Microsoft.Identity.Web.GraphServiceClient` for integrated token management
```

**Copilot prompt**: `Add Microsoft Graph API best practices to copilot-instructions.md`

#### 2. Add Graph NuGet Packages

Add to the web application project:

- `Microsoft.Graph` (v5.x or later)
- `Microsoft.Identity.Web.GraphServiceClient` (latest)

```shell
dotnet add package Microsoft.Graph
dotnet add package Microsoft.Identity.Web.GraphServiceClient
```

#### 3. Configure Graph Scopes

Update `appsettings.json`:

```json
{
  "MicrosoftGraph": {
    "BaseUrl": "https://graph.microsoft.com/v1.0",
    "Scopes": "Mail.Read Mail.ReadWrite Mail.Send"
  }
}
```

Also update the Entra ID app registration with Graph API permissions:

- `Mail.Read` (delegated)
- `Mail.ReadWrite` (delegated)
- `Mail.Send` (delegated)

#### 4. Implement GraphEmailService

Create `Services/GraphEmailService.cs` implementing `IEmailService`:

Key implementations:

**GetInboxEmailsAsync**:

```csharp
var messages = await _graphServiceClient.Me.Messages
    .GetAsync(config => {
        config.QueryParameters.Select = new[] { "id", "subject", "from", "receivedDateTime", "bodyPreview", "hasAttachments", "isRead", "importance" };
        config.QueryParameters.Top = count;
        config.QueryParameters.Orderby = new[] { "receivedDateTime desc" };
    });
// Convert to domain EmailMessage
```

**GetEmailByIdAsync**:

```csharp
var message = await _graphServiceClient.Me.Messages[emailId]
    .GetAsync(config => {
        config.QueryParameters.Select = new[] { "id", "subject", "from", "body", "receivedDateTime", "sentDateTime", "toRecipients", "ccRecipients" };
    });
```

**SendReplyAsync**:

```csharp
await _graphServiceClient.Me.Messages[emailId].Reply
    .PostAsync(new ReplyPostRequestBody { Message = replyMessage });
```

**Copilot prompt**: `Implement IEmailService using Graph API`

#### 5. Add Feature Toggle

Update `Program.cs` for conditional DI registration:

```csharp
var useGraphApi = builder.Configuration.GetValue<bool>("UseGraphApi");
if (useGraphApi)
{
    builder.Services.AddMicrosoftGraph(builder.Configuration.GetSection("MicrosoftGraph"));
    builder.Services.AddScoped<IEmailService, GraphEmailService>();
}
else
{
    builder.Services.AddScoped<IEmailService, EwsEmailService>();
}
```

Add to `appsettings.json`:

```json
{
  "UseGraphApi": true
}
```

#### 6. Generate Unit Tests for Graph Implementation

Generate tests for GraphEmailService:

**Copilot prompt**: `Write unit tests for the GraphEmailService class`

Fix any test issues iteratively. Common issues:

- GraphServiceClient mocking can be complex — use wrapper patterns if needed
- Model conversion tests (Graph `Message` → domain `EmailMessage`)
- Error scenario tests (404 for missing emails, auth failures)

#### 7. Validate

- Run all tests (both EWS and Graph tests should pass)
- Set `UseGraphApi: true` in appsettings.json
- Run the application under Aspire
- Debug any token/scope issues using the Aspire dashboard
- Common issue: GraphServiceClient not configured for correct scopes → update `AddMicrosoftGraph` configuration
- Verify: emails load via Graph API, replies send via Graph API

### Human Checkpoint (Phase 4b)

**"The Graph API implementation is complete. The application works with both EWS and Graph via feature toggle. All tests pass (EWS + Graph). Do you approve the Graph implementation?"**

- Options: [Approve Phase 4b] [Request changes] [Test more scenarios]

---

## Phase 4c: Validate & Remove EWS

### Goal

Confirm Graph implementation is complete, then remove all EWS dependencies from the codebase.

### Steps

#### 1. End-to-End Validation with Graph

With `UseGraphApi: true`:

- [ ] Login/authentication works
- [ ] Inbox emails load correctly
- [ ] Email details display properly
- [ ] Reply form pre-fills correctly
- [ ] Sending a reply works
- [ ] Error scenarios handled (no network, invalid token, etc.)

#### 2. Remove EWS Implementation

**Copilot prompt**: `Remove all references to EWS including logic, tests and NuGet packages specific to EWS`

Specifically remove:

- `EwsEmailService.cs` — delete file
- `ExchangeServiceFactory.cs` — delete file (if exists)
- EWS NuGet packages — remove from `.csproj`:
  - `Microsoft.Exchange.WebServices.NETStandard`
  - Any other `Microsoft.Exchange.*` packages
- EWS-specific test classes — delete files
- `UseGraphApi` feature toggle — no longer needed (Graph is the only implementation)
- EWS configuration in `appsettings.json` — remove EWS-specific settings
- EWS references in `.github/copilot-instructions.md`
- EWS references in `copilot.json`
- Unused `using Microsoft.Exchange.*` statements throughout

#### 3. Simplify DI Registration

Update `Program.cs` — remove feature toggle, register Graph directly:

```csharp
builder.Services.AddMicrosoftGraph(builder.Configuration.GetSection("MicrosoftGraph"));
builder.Services.AddScoped<IEmailService, GraphEmailService>();
```

#### 4. Clean Up

- Remove unused `using` statements
- Remove catch blocks for EWS-specific exceptions (e.g., `ServiceResponseException`)
- Update all documentation to reflect Graph-only architecture

#### 5. Final Validation

1. **Build**: `dotnet build` — zero errors, zero warnings
2. **Test**: `dotnet test` — all tests pass (count reduced after removing EWS tests)
3. **Analyzer**: Add `Ews.Analyzer` NuGet package back temporarily and build — confirm **zero EWS references** (EWS004 should report 0 references)
4. **Runtime**: Start application under Aspire, verify all functionality
5. **Remove analyzer** (optional): Remove Ews.Analyzer if no longer needed

### Human Checkpoint (Phase 4c)

**"All EWS code has been removed. The application runs entirely on Microsoft Graph API. Final validation results:"**

- Build: [pass/fail]
- Tests: [X] tests passing (down from [Y] after removing EWS-specific tests)
- EWS Analyzer: Zero EWS references
- Runtime: All use cases working

**"Do you approve the completed migration?"**

- Options: [Approve — migration complete] [Request changes] [Rollback to Phase 4b]

---

## EWS to Graph API Operation Mapping

See [EWS to Graph API Mapping](references/ews-graph-mapping.md) for the complete operation mapping table and parity gap guidance.

## Acceptance Criteria (Complete Skill)

- [ ] IEmailService interface defined with all email operations
- [ ] Domain model EmailMessage created (decoupled from EWS/Graph types)
- [ ] Controller refactored to depend only on IEmailService
- [ ] GraphEmailService implements all IEmailService methods
- [ ] Graph scopes configured (Mail.Read, Mail.ReadWrite, Mail.Send)
- [ ] Application retrieves and sends emails via Graph API
- [ ] All unit tests pass (including new Graph-specific tests)
- [ ] EWS Code Analyzer reports zero EWS references
- [ ] All EWS NuGet packages removed
- [ ] EwsEmailService and related EWS files deleted
- [ ] Documentation updated to reflect Graph-only architecture

## Next Skill

Upon approval → **Skill 05: Final Validation & Documentation** (`skill-05-validate.md`)
