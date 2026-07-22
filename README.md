# EWS Migration Tools

## Overview

Exchange Web Services (EWS) is a SOAP-based API that is used to access Exchange Online and Exchange Server. Microsoft has announced the deprecation of EWS for Exchange Online in favor of Microsoft Graph API, which provides access to many Microsoft 365 workloads including Exchange Online.

This repo contains tools to help you identify apps within your tenant that are using EWS and help you migrate them to Microsoft Graph API.

## EWS Usage Reporting

M365 provides [reports of EWS usage](https://admin.cloud.microsoft/?#/reportsUsage/EWSWeeklyUsage) in the Microsoft 365 admin center. This your first port of call to identify apps using EWS. The reports are available in the Microsoft 365 admin center under Reports > Usage > Exchange Web Services (EWS) Weekly Usage for tenants in the worldwide cloud.

If your tenant is in an isolated cloud (e.g. government or sovereign cloud), these reports are not available in the admin center at this time.

The tools in this repo can be used to identify EWS usage in all clouds with access to the Microsoft Graph AuditLog API.

The EWS Usage Reporting tools are located in folder `/src/Ews.AppUsage` and build on [Jim Martin](https://github.com/jmartinmsft)'s excellent and versatile scripts for detecting EWS usage in Exchange Online at [Exchange App Usage Reporting](https://github.com/jmartinmsft/Exchange-App-Usage-Reporting).

See the [EWS App Usage Reporting Readme](src/Ews.App.Usage/README.md) for more information on how to set up and run the tools.

## EWS Code Analyzer

Once you have identified applications in your tenant using EWS, you can use the EWS Code Analyzer to identify all EWS references in your code and get suggestions for migration to equivalent Microsoft Graph APIs from related documentation and GitHub Copilot.

The code analyzer is a Roslyn analyzer that can be used in Visual Studio and Visual Studio Code. At this time it supports any EWS application built using .NET SDKs for EWS.

The EWS Code Analyzer is located in folder `/src/Ews.CodeAnalyzer`.

See the [EWS Code Analyzer Readme](src/Ews.Code.Analyzer/README.md) for more information on how to set up and run the tools.

## Using the Plugins in Copilot CLI

This repository includes a Copilot plugin marketplace manifest at [`.copilot/plugin/marketplace.json`](.copilot/plugin/marketplace.json). It exposes the EWS migration skills and orchestrator from [Ews.Sample.Migration](src/Ews.Sample.Migration/README.md) so you can use them directly from Copilot CLI.

### Add the marketplace

Start a Copilot CLI session in the repository you want to migrate, then add this marketplace:

```text
/plugin marketplace add OfficeDev/ews-migration-analyzer
```

This registers the `ews-migration` plugin from this repository.

### Use the orchestrator

After the marketplace is installed, ask Copilot CLI to use the orchestrator skill to assess the repo and guide the migration end to end.

Example prompts:

```text
Use the EWS Migration Orchestrator to assess this repository and tell me which migration stage to start with.
```

```text
Run the EWS migration orchestrator and walk me through the migration with human approval gates.
```

### Use an individual skill

You can also invoke a single skill when you only need one part of the workflow.

Example prompts:

```text
Use the ews-discover skill to identify all EWS usage in this codebase and produce a discovery report.
```

```text
Use the ews-understand skill to derive the migration seams, requirements, and Copilot instructions from the current EWS implementation.
```

```text
Use the ews-refactor skill to plan the Graph migration based on the code-derived interfaces and models.
```

### What gets installed

The `ews-migration` plugin includes these skills:

- `ews-discover`
- `ews-understand`
- `ews-instrument`
- `ews-test`
- `ews-refactor`
- `ews-validate`
- `ews-migration-orchestrator`

If you want the full step-by-step walkthrough before using the plugin in Copilot CLI, see [src/Ews.Sample.Migration/README.md](src/Ews.Sample.Migration/README.md).

## Feedback

We welcome your feedback on the tools in this repo and also on your migration experience from EWS to Microsoft Graph API. You can provide feedback by creating an issue in this [repo](https://github.com/OfficeDev/ews-migration-analyzer/issues) or by posting questions on StackOverflow with the tag [exchangewebservices](https://stackoverflow.com/questions/tagged/exchangewebservices).

## Learn More

- [Deprecation of Exchange Web Services in Exchange Online](https://aka.ms/ews1pageGH)
- [Microsoft 365 Reports in the admin center – EWS usage](https://aka.ms/EwsAdminReports)
- [Exchange Web Services (EWS) to Microsoft Graph API mappings](https://aka.ms/ewsMapGH)

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit <https://cla.opensource.microsoft.com>.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow [Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
