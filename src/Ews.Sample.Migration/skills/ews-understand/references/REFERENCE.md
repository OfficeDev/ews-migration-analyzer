# Reference: EWS Understanding & Documentation

## Microsoft Documentation

- **AI Assisted EWS Migration Tutorial**: <https://aka.ms/ewsToolsAITutorial>
- **Graph Mail API Overview**: <https://learn.microsoft.com/en-us/graph/api/resources/mail-api-overview>
- **Exchange Blog - Migration Series**: <https://aka.ms/ews2graphGettingStarted>
- **EWS to Graph API Mappings**: <https://aka.ms/ews2graphMap>

## Copilot Prompt Templates

### Analyze EWS-Heavy Files

```text
Explain the code in this file, focusing on EWS operations, authentication flow, and data transformations.
```

### Derive Migration Seams

```text
Analyze the current EWS implementation and derive the exact service interfaces, domain models, and Graph implementation mappings this application needs. Create migration-seams.md using names, method shapes, and fields justified by the code rather than generic examples.
```

### Generate Requirements Document

```text
Generate a requirements document called requirements.md for this application outlining technology stack, authentication mechanism, important dependencies, all implemented use cases, and the code-derived migration seams each use case depends on.
```

### Generate XML Documentation Comments

```text
Generate XML code comments for this file, describing the purpose and EWS interactions of each public member.
```

### Generate GitHub Copilot Instructions

```text
Generate the GitHub Copilot instructions file that identifies the technologies, coding standards, and code-derived migration seams used in this application.
```

### Generate Project-Level Copilot Configuration

```text
Generate a copilot.json file that identifies the technologies, coding standards, and code-derived migration seams used in this application.
```
