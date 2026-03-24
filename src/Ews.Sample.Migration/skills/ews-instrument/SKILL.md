---
name: ews-instrument
description: "Add observability infrastructure using .NET Aspire to an EWS application before migration. Configures structured logging, distributed tracing, and metrics so that changes during migration can be monitored and debugged at runtime. Use when preparing an application for EWS-to-Graph migration."
license: MIT
compatibility: "Requires .NET SDK 9.0+, .NET Aspire workload installed."
metadata:
  stage: "02"
  category: "ews-migration"
  prerequisites: "ews-understand"
---

# Skill: Add Instrumentation

## Purpose

You are an AI assistant specialized in adding observability infrastructure to .NET applications using .NET Aspire. Your goal is to instrument the application so that changes during the EWS-to-Graph migration can be monitored, debugged, and validated at runtime through structured logs, traces, and metrics.

Even for relatively simple applications, Aspire adds significant value through improved inner-loop development, structured telemetry, and a unified dashboard for investigating issues that arise during migration.

## Context

This skill is Stage 02 of the EWS Migration Skills Marketplace. It depends on the documentation from Skill 01. The observability infrastructure added here will be critical during Skill 04 (Refactor & Migrate) for debugging token acquisition issues, Graph API errors, and runtime regressions.

## Prerequisites

- Completed Skill 01 (Build Understanding)
- .NET SDK 9.0 or later installed
- Application documented with requirements.md and copilot instructions

## What You Do

### Step 1: Add Aspire ServiceDefaults Project

Create a new `ServiceDefaults` project in the solution:

1. Add a new project using the **Aspire Service Defaults** template
2. This project contains shared service defaults and extension methods
3. It enables consistent telemetry configuration across all projects in the solution
4. Key file: `Extensions.cs` — provides `AddServiceDefaults()` extension method

**Command**: `dotnet new aspire-servicedefaults -n ServiceDefaults`

### Step 2: Add Aspire AppHost Project

Create a new `AppHost` project in the solution:

1. Add a new project using the **Aspire App Host** template
2. This project serves as the orchestration layer for development
3. It will be the startup project when running the application

**Command**: `dotnet new aspire-apphost -n AppHost`

### Step 3: Wire Up Project References

1. **AppHost → Web Application**: Add a project reference from AppHost to the main web application project
   ```
   dotnet add AppHost reference ../Contoso.Mail.Web/Contoso.Mail.Web.csproj
   ```

2. **Web Application → ServiceDefaults**: Add a project reference from the web application to ServiceDefaults
   ```
   dotnet add Contoso.Mail.Web reference ../ServiceDefaults/ServiceDefaults.csproj
   ```

3. **Update solution file**: Add both new projects to the solution
   ```
   dotnet sln add AppHost/AppHost.csproj
   dotnet sln add ServiceDefaults/ServiceDefaults.csproj
   ```

### Step 4: Configure ServiceDefaults in the Web Application

In the web application's `Program.cs`, add the service defaults call after creating the builder:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.AddServiceDefaults(); // Add this line
// ... rest of configuration
```

### Step 5: Configure AppHost Orchestration

In `AppHost/Program.cs` (or `AppHost.cs`), register the web application:

```csharp
var builder = DistributedApplication.CreateBuilder(args);
var web = builder.AddProject<Projects.Contoso_Mail_Web>("mail-web");
builder.Build().Run();
```

### Step 6: Set AppHost as Startup Project

Configure the solution to use AppHost as the startup project for development.

### Step 7: Verify Aspire Dashboard

1. Start the application (F5 or `dotnet run` in AppHost)
2. A command window opens with log output
3. Find the Aspire dashboard URL in the logs
4. Verify the dashboard shows:
   - The `mail-web` application listed and running
   - **Logs**: Structured log entries from the application
   - **Traces**: Request traces (drill into trace details to see individual spans)
   - **Metrics**: Application metrics (request rates, response times, etc.)
5. If available (Aspire v0.9.2+), test the **GitHub Copilot integration** in the dashboard for explaining error messages

## Resulting Project Structure

```
Solution/
├── AppHost/
│   ├── Program.cs (or AppHost.cs) — orchestration
│   ├── AppHost.csproj
│   ├── appsettings.json
│   └── appsettings.Development.json
├── ServiceDefaults/
│   ├── Extensions.cs — AddServiceDefaults() method
│   └── ServiceDefaults.csproj
├── Contoso.Mail.Web/
│   ├── Program.cs — updated with builder.AddServiceDefaults()
│   └── ... (rest of application)
└── Solution.sln — updated with new projects
```

## Why This Matters for Migration

The Aspire dashboard will be invaluable during Skill 04 (Refactor & Migrate):
- **Token acquisition errors**: When switching from EWS tokens (`outlook.office365.com/.default`) to Graph tokens (`Mail.Read`, `Mail.ReadWrite`, `Mail.Send`), MSAL errors will be visible in traces
- **Graph API errors**: HTTP 401/403/404 responses from Graph will appear in traces with full request/response details
- **Performance comparison**: Metrics help compare EWS vs Graph API performance
- **Regression detection**: Trace comparison helps identify behavioral changes

## Reference Documentation

- .NET Aspire Overview: https://learn.microsoft.com/en-us/dotnet/aspire/
- .NET Aspire Service Defaults: https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/service-defaults
- .NET Aspire App Host: https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/app-host-overview
- AI Assisted EWS Migration Tutorial: https://aka.ms/ewsToolsAITutorial

## Acceptance Criteria

- [ ] ServiceDefaults project added to solution
- [ ] AppHost project added to solution
- [ ] Web application references ServiceDefaults
- [ ] AppHost references web application
- [ ] `builder.AddServiceDefaults()` called in Program.cs
- [ ] AppHost orchestrates the web application
- [ ] Application starts successfully under Aspire
- [ ] Aspire dashboard is accessible
- [ ] Logs visible in dashboard
- [ ] Traces visible in dashboard
- [ ] Metrics visible in dashboard

## Human Checkpoint

Before proceeding to Skill 03 (Add Tests), demonstrate the Aspire dashboard to the developer:

1. **"Can you see the application running in the Aspire dashboard?"**
   - If no: debug startup issues, check project references
2. **"Are logs, traces, and metrics flowing?"**
   - If no: verify `AddServiceDefaults()` is called, check ServiceDefaults configuration
3. **"Do you approve the instrumentation setup?"**
   - Options: [Approve and proceed] [Debug issues] [Skip (already instrumented)]

Do NOT proceed to the next skill without explicit human approval.

## Next Skill

Upon approval → **Skill 03: Add Tests** (`skill-03-test.md`)
