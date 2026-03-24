# EWS Test Skill — References

## Microsoft Documentation

- **.NET Unit Testing with xUnit**: <https://learn.microsoft.com/en-us/dotnet/core/testing/>
- **Code Coverage in .NET**: <https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage>
- **AI Assisted EWS Migration Tutorial — Testing**: <https://aka.ms/ewsToolsAITutorial>

## Testing Frameworks

- **xUnit Documentation**: <https://xunit.net/>
- **NSubstitute Documentation**: <https://nsubstitute.github.io/>

## NuGet Packages

| Package | Purpose |
|---|---|
| `xunit` | Test framework |
| `xunit.runner.visualstudio` | Visual Studio / CLI test runner |
| `Microsoft.NET.Test.Sdk` | Test runner infrastructure |
| `NSubstitute` | Mocking framework |
| `NSubstitute.Analyzers.CSharp` | Static analysis for common NSubstitute mistakes |
| `coverlet.collector` | Code coverage collection |

## Common Test Patterns

### Controller Action Test (xUnit + NSubstitute)

```csharp
[Fact]
public async Task Index_WithValidUser_ReturnsViewWithEmails()
{
    // Arrange
    var mockService = Substitute.For<IEmailService>();
    mockService.GetInboxEmailsAsync(Arg.Any<string>(), Arg.Any<int>())
        .Returns(new List<EmailMessage> { /* test data */ });
    var controller = new MailController(mockService, mockLogger);

    // Act
    var result = await controller.Index();

    // Assert
    var viewResult = Assert.IsType<ViewResult>(result);
    var model = Assert.IsAssignableFrom<IList<EmailMessage>>(viewResult.Model);
    Assert.NotEmpty(model);
}
```

### Parameterized Validation Test

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

### xUnit Conventions

- `[Fact]` — tests without parameters
- `[Theory]` + `[InlineData]` — parameterized tests
- Arrange-Act-Assert (AAA) pattern
- Naming: `MethodName_Scenario_ExpectedResult`

### NSubstitute Conventions

- `Substitute.For<T>()` — create test doubles
- `.Returns()` — configure return values
- `.Received()` — verify method calls
- `Arg.Any<T>()` — flexible argument matching
- `Arg.Is<T>(predicate)` — specific argument matching

## Common Compilation Fixes

| Issue | Fix |
|---|---|
| `Task` type ambiguity (EWS vs System) | Add `using Task = System.Threading.Tasks.Task;` |
| Internal type access | Add `[assembly: InternalsVisibleTo("ProjectName.Tests")]` |
| Missing namespace imports | Add required `using` statements |
| Mock signature mismatch | Verify method signatures match mock configuration |
