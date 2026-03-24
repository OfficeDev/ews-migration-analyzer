---
name: ews-test
description: "Generate comprehensive unit tests using xUnit and NSubstitute for .NET applications before EWS-to-Graph migration. Creates a safety net of automated tests that provide confidence during refactoring. Use when adding tests to an EWS application prior to API migration or major refactoring."
license: MIT
compatibility: "Requires .NET SDK 9.0+, xUnit, NSubstitute NuGet packages."
metadata:
  stage: "03"
  category: "ews-migration"
  prerequisites: "ews-instrument"
---

# Skill: Add Tests

## Purpose

You are an AI assistant specialized in generating comprehensive unit tests for .NET applications using xUnit and NSubstitute. Your goal is to create a safety net of automated tests that provide confidence during the EWS-to-Graph API migration. Tests ensure that refactoring in Skill 04 doesn't break existing functionality.

Testing is essential when replacing core functionality. The majority of tests can be AI-generated, but human review is critical to validate test quality and relevance.

## Context

This skill is Stage 03 of the EWS Migration Skills Marketplace. It depends on the documented and instrumented application from Skills 01–02. The tests produced here will be the primary regression detection mechanism during Skill 04 (Refactor & Migrate to Graph API).

## Prerequisites

- Completed Skill 02 (Add Instrumentation)
- Application documented with `requirements.md`
- `.github/copilot-instructions.md` exists
- Application running successfully under Aspire

---

## What You Do

### Step 1: Add Testing Best Practices to Copilot Instructions

Update `.github/copilot-instructions.md` to include testing guidance:

```markdown
## Testing Best Practices

### xUnit Conventions
- Use `[Fact]` for tests without parameters
- Use `[Theory]` with `[InlineData]` for parameterized tests
- Follow Arrange-Act-Assert (AAA) pattern
- Use descriptive test names: `MethodName_Scenario_ExpectedResult`
- One assertion per test when possible
- Use `Assert.Equal`, `Assert.True`, `Assert.Throws<T>`, etc.

### NSubstitute Conventions
- Use `Substitute.For<T>()` to create test doubles
- Use `.Returns()` to configure return values
- Use `.Received()` to verify method calls
- Use `Arg.Any<T>()` for flexible argument matching
- Use `Arg.Is<T>(predicate)` for specific argument matching
- Configure all dependencies before acting

### Test Organization
- Mirror source project structure in test project
- Group tests by class under test (e.g., `Controllers/MailControllerTests.cs`)
- Use helper methods for common test setup
- Separate test doubles into `TestDoubles/` folder if complex
```

**Copilot prompt**: `Add best practices for xUnit tests and NSubstitute to the .github/copilot-instructions.md file.`

### Step 2: Create Test Project

Create a new xUnit test project:

1. **Create project**: `dotnet new xunit -n [ProjectName].Tests`
2. **Add NuGet packages**:
   - `NSubstitute` — mocking framework
   - `NSubstitute.Analyzers.CSharp` — catches common NSubstitute mistakes
   - `Microsoft.NET.Test.Sdk` — test runner infrastructure
   - `xunit` — test framework
   - `xunit.runner.visualstudio` — Visual Studio test runner
   - `coverlet.collector` — code coverage collection
3. **Add project reference**: Reference the main application project
4. **Add to solution**: `dotnet sln add [ProjectName].Tests/[ProjectName].Tests.csproj`

### Step 3: Generate Unit Tests

For each controller and service class, generate comprehensive tests:

**Copilot prompt**: `Write unit tests for all methods in #[FileName].cs`

Expected test categories:

- **Positive path tests**: Normal operation with valid inputs
- **Validation tests**: Invalid inputs, null checks, empty strings
- **Exception handling tests**: Service failures, authentication errors
- **Edge case tests**: Boundary values, whitespace, special characters
- **Model state tests**: Invalid model state handling

### Step 4: Fix Compilation Issues

Common issues to watch for and fix:

1. **Task ambiguity**: `Microsoft.Exchange.WebServices.Data.Task` vs `System.Threading.Tasks.Task`
   - Fix: Add `using Task = System.Threading.Tasks.Task;` at top of test files

2. **NSubstitute mock configuration**: Mock setup may not match actual method signatures
   - Fix: Verify method signatures and adjust mock configuration

3. **Missing using statements**: Test generation may miss required namespace imports
   - Fix: Add missing usings

4. **Internal type access**: Tests may need access to internal types
   - Fix: Add `[assembly: InternalsVisibleTo("ProjectName.Tests")]` to main project

### Step 5: Run Tests and Iterate

1. Run all tests: `dotnet test`
2. For each failing test:
   - Analyze the failure message
   - Determine if the test is correct (code bug) or incorrect (test bug)
   - If test bug: fix the test
   - If code bug: fix the code (e.g., `string.IsNullOrEmpty` → `string.IsNullOrWhiteSpace`)
3. Remove irrelevant or flaky tests to prevent test technical debt
4. Re-run until all tests pass

**Copilot prompt for failures**: `This test is failing with error: [error message]. Analyze and fix the issue.`

### Step 6: Code Coverage Analysis

1. Run tests with coverage: `dotnet test --collect:"XPlat Code Coverage"`
2. Review coverage report
3. Document coverage gaps:
   - **Expected gaps**: Authentication flows, live EWS/Graph API calls (require integration tests)
   - **Actionable gaps**: Business logic paths not covered → generate additional tests
4. Establish baseline coverage percentage

---

## Expected Test Structure

```
[ProjectName].Tests/
├── Controllers/
│   └── MailControllerTests.cs
├── Services/
│   └── [ServiceName]Tests.cs (added in Skill 04)
├── Models/
│   └── [ModelName]Tests.cs
├── Helpers/
│   └── TestHelpers.cs
├── TestDoubles/
│   └── [Any complex test doubles]
├── [ProjectName].Tests.csproj
└── GlobalUsings.cs
```

---

## Common Test Patterns

### Controller Action Test

```csharp
[Fact]
public async Task Index_WithValidUser_ReturnsViewWithEmails()
{
    // Arrange
    var mockService = Substitute.For<IEmailService>();
    mockService.GetInboxEmailsAsync(Arg.Any<string>(), Arg.Any<int>())
        .Returns(new List<EmailMessage> { /* test data */ });
    var controller = new MailController(mockService, mockLogger);
    // Set up HttpContext with user claims

    // Act
    var result = await controller.Index();

    // Assert
    var viewResult = Assert.IsType<ViewResult>(result);
    var model = Assert.IsAssignableFrom<IList<EmailMessage>>(viewResult.Model);
    Assert.NotEmpty(model);
}
```

### Validation Test

```csharp
[Theory]
[InlineData(null)]
[InlineData("")]
[InlineData("   ")]
public async Task Reply_WithInvalidId_ReturnsBadRequest(string id)
{
    // Arrange
    var controller = new MailController(mockService, mockLogger);

    // Act
    var result = await controller.Reply(id);

    // Assert
    Assert.IsType<BadRequestObjectResult>(result);
}
```

---

## Reference Documentation

- .NET Unit Testing with xUnit: <https://learn.microsoft.com/en-us/dotnet/core/testing/>
- NSubstitute Documentation: <https://nsubstitute.github.io/>
- Code Coverage in .NET: <https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage>
- AI Assisted EWS Migration Tutorial — Testing: <https://aka.ms/ewsToolsAITutorial>

---

## Acceptance Criteria

- [ ] Test project created with xUnit and NSubstitute
- [ ] Test project references main application project
- [ ] Testing best practices added to `copilot-instructions.md`
- [ ] Unit tests generated for all controller/service methods
- [ ] All compilation issues resolved
- [ ] All tests pass after iterative fixing
- [ ] Irrelevant or flaky tests removed
- [ ] Code coverage baseline established and documented
- [ ] Coverage gaps identified and documented (expected vs actionable)

---

## Human Checkpoint

Before proceeding to Skill 04 (Refactor & Migrate), present test results to the developer:

1. **"Do all tests pass?"**
   - If no: investigate and fix remaining failures
2. **"Are the generated tests relevant and meaningful?"**
   - If no: remove irrelevant tests, request specific additional tests
3. **"Is the code coverage baseline acceptable?"**
   - If no: generate additional tests for uncovered paths
4. **"Did any tests reveal code bugs?"**
   - If yes: document the bugs found and fixes applied (e.g., `IsNullOrEmpty` → `IsNullOrWhiteSpace`)
5. **"Do you approve this test suite as the regression safety net for migration?"**
   - Options: [Approve and proceed] [Request more tests] [Remove specific tests]

Do NOT proceed to the next skill without explicit human approval.

---

## Next Skill

Upon approval → **Skill 04: Refactor & Migrate to Graph API** (`skill-04-refactor.md`)
