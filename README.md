# Proof of Concept – Performance Review Module Infrastructure

We’ll build this module as a **new service**, starting from a blank slate. This will let us use modern patterns, ensure scalability, and avoid being tied down by the existing app’s structure.

## Two Possible Approaches

### Option 1 – Multiple Microservices

Independent microservices that talk to each other through REST APIs or a message queue.

### Option 2 – Modular Monolith

A single service with a “microservice-like” structure: each module is independent (controllers, models, logic separated), but everything lives in one codebase.

## Recommended Tech Stack (for both options)

- **Backend:** C# .NET Core
- **Real-time updates:** SignalR
- **Messaging/decoupling:** Azure Service Bus (Message Queue)
- **Database:** PostgreSQL or MongoDB (depending on reporting needs)
- **Hosting:** Azure App Service or Containers (scales with demand) (or on the vm that we already use I think)

## Suggested Path 

For the **proof of concept**, start with **Option 2 (Modular Monolith)**. It’s faster, cheaper, and easier to deliver. If adoption grows and requirements expand, we can gradually split modules into separate microservices (Option 1).


I’ll start developing my view based on **Option 2** from Marc’s services division.

In this approach, I would divide the solution into multiple independent projects that can still communicate with each other through REST APIs. Each project would have its own controllers and logic, without directly referencing the others. For example, everything related to **Delivery** would have its own implementation, while **Performance Management** would have its own models and features and so on. They would interact like microservices, but remain within the same solution.

The advantage of this approach is that it lowers the cost and complexity of managing multiple standalone microservices, while allowing faster delivery of a working project. With pure microservices, each service must be deployed, monitored, and maintained independently, which adds setup and development overhead.

With this modular monolith approach, all of Marc’s proposed logic is still fully implemented, but within a single codebase—giving us both speed and simplicity. And as mentioned earlier, if a particular module later becomes a bottleneck, we can always separate it into its own dedicated microservice.

I chose this path because we’re working under **time and cost constraints**, and I believe this option gives us the best balance of scalability and efficiency.




# -----------------
# IGNORER LE RESTE
# -----------------
# -----------------
# -----------------
# -----------------
# -----------------
# -----------------
# -----------------
# -----------------
# -----------------
# -----------------

# C# .NET Core Best Practices Guide

This guide provides best practices and recommendations for building maintainable, scalable, and clean C# .NET Core applications, including guidelines for naming conventions, file structure, project organization, business logic, and working with MongoDB.

---

## ✍️ Naming Conventions

### ✅ General Principles
- Use **PascalCase** for public members, class names, and method names.
- Use **camelCase** for private/internal fields, local variables, and method parameters.
- Prefix private fields with an underscore (`_`) only if it helps readability, otherwise prefer plain `camelCase`.
- Avoid overly short or abbreviated names unless they are widely understood (e.g., `id`, `url`).
- Use **PascalCase** for public members, class names, and method names.
- Use **camelCase** for private/internal fields and local variables.
- Use clear, meaningful, and descriptive names.

### ✅ Properties and Fields
```csharp
public string FirstName { get; set; }   // PascalCase for properties
private string firstName;               // camelCase for private fields
```

### ✅ Methods
```csharp
public void CalculateSalary() { }
private void loadConfiguration() { }   // Use PascalCase for all methods
```

### ✅ Variables
```csharp
string userName = "JohnDoe";          // Local variable (camelCase)
int maxAttempts = 5;                  // camelCase for method-scope variables
private string connectionString;      // Private field (camelCase)
private string _internalCache;        // Private field with underscore (optional style)
```

---

### 🔄 `var` vs. Explicit Types

In C#, **both `var` and explicit type declarations** are valid, and which one you should use depends on **readability, clarity, and context**.

Here are the best practices:

#### ✅ Use `var` when:
1. **The type is obvious from the right-hand side**:
   ```csharp
   var customer = new Customer(); // clear and concise
   ```

2. **You're working with LINQ**, especially with anonymous types:
   ```csharp
   var result = list.Where(x => x.IsActive).Select(x => new { x.Id, x.Name });
   ```

3. **You want to avoid long type names**:
   ```csharp
   var dictionary = new Dictionary<string, List<Tuple<int, string>>>();
   ```

#### ❌ Avoid `var` when:
1. **The type is not obvious**, and it hurts readability:
   ```csharp
   var data = GetSomething(); // not clear what data is
   ```
   Better:
   ```csharp
   DataResult data = GetSomething();
   ```

2. You're declaring **primitive types** and want to make the type explicit for readability:
   ```csharp
   int count = 5;
   string name = "John";
   ```

#### 🚀 Summary Guideline:
| Scenario                              | Recommendation       |
|---------------------------------------|----------------------|
| Type is obvious                       | ✅ Use `var`          |
| Type is not obvious                   | ❌ Use explicit type  |
| Complex or generic types              | ✅ Use `var`          |
| Primitive types                       | ✅ Use explicit type  |
| Anonymous types (e.g., in LINQ)       | ✅ Must use `var`     |

```csharp
string userName = "JohnDoe";          // Local variable (camelCase)
int maxAttempts = 5;                  // camelCase for method-scope variables
private string connectionString;      // Private field (camelCase)
private string _internalCache;        // Private field with underscore (optional style)
```

### ✅ Constants
```csharp
private const int MaxAttempts = 3;
```

### ❌ Avoid
- Abbreviations (e.g., `usrNm`, `qty`)
- Hungarian notation (e.g., `strName`)
- Underscores in variable names (`user_name`) except for private fields if preferred: `_userName`

---

## 📁 File and Project Structure Example

Given the structure below, each project has a clear responsibility and separation of concerns:

```
/ProjectName.sln
│
├── ProjectName.WebApi            // Startup project (Controllers, Program.cs, etc.)
│
├── ProjectName.Data              // Repositories, MongoDB queries, and entity models
│
├── ProjectName.Common            // Constants, helpers, extensions, static functions
│
├── ProjectName.ExternalServices  // Discord, SendGrid, other APIs integration
│
└── ProjectName.Tests             // xUnit test project for all modules
```

---

## 🧱 Recommended Layered Architecture

### 🧭 Layers
- **ProjectName.WebApi**: API layer, responsible for request handling and routing
- **ProjectName.Data**: Handles data access, MongoDB repositories, and entities
- **ProjectName.Common**: Shared reusable code (helpers, constants, utilities)
- **ProjectName.ExternalServices**: Integration with third-party services
- **ProjectName.Tests**: Automated tests

---

## 🧠 Business Logic Best Practices

### 🧩 Keep It in Services
- Create a `Services` folder or project if needed (e.g., `ProjectName.Application`) for all business rules.
- Avoid putting logic in controllers or repositories.

```csharp
public interface ITimeTrackingService
{
    Task TrackTimeAsync(TimeEntry entry);
    Task<IEnumerable<TimeEntry>> GetWeeklySummaryAsync(string userId);
}
```

```csharp
public class TimeTrackingService : ITimeTrackingService
{
    private readonly ITimeEntryRepository _repository;

    public TimeTrackingService(ITimeEntryRepository repository)
    {
        _repository = repository;
    }

    public async Task TrackTimeAsync(TimeEntry entry)
    {
        // Business logic here
        if (entry.Duration <= TimeSpan.Zero)
            throw new ArgumentException("Duration must be positive.");

        await _repository.InsertAsync(entry);
    }

    public async Task<IEnumerable<TimeEntry>> GetWeeklySummaryAsync(string userId)
    {
        // Combine/filter/sort domain data
        return await _repository.GetForWeekAsync(userId, DateTime.UtcNow);
    }
}
```

### ✅ Principles
- Keep logic testable and reusable
- Inject dependencies
- Avoid static classes for logic
- Prefer interfaces and dependency injection
- Group logic by domain feature (feature folders or modules)

---

## 🧩 MongoDB Integration Best Practices

### 🗂️ MongoDbSettings
```csharp
public class MongoDbSettings
{
    public string ConnectionString { get; set; }
    public string DatabaseName { get; set; }
}
```
Register in `Startup.cs`:
```csharp
services.Configure<MongoDbSettings>(Configuration.GetSection("MongoDb"));
```

### 📦 MongoDB Repository Pattern
```csharp
public interface IUserRepository
{
    Task<User> GetByIdAsync(string id);
    Task CreateAsync(User user);
    Task UpdateAsync(User user);
    Task DeleteAsync(string id);
}
```

### 🧼 Clean Query Examples

Using `async/await` with MongoDB queries is considered best practice for several important reasons:

#### ✅ Non-blocking I/O
MongoDB queries often involve I/O operations (disk access, network requests). Using `await` allows the thread to yield while the operation is in progress, freeing it to handle other tasks and improving scalability.

#### ✅ Better Performance in Web APIs
For ASP.NET Core Web APIs, asynchronous methods reduce thread exhaustion under heavy load. This results in better request throughput and responsiveness.

#### ✅ Built-in Support in MongoDB Driver
The official MongoDB C# driver provides asynchronous methods (e.g., `FindAsync`, `InsertOneAsync`) designed to work efficiently with `async/await`. These should be preferred over synchronous alternatives.

#### ✅ Improves Testability
Async code integrates better with unit testing frameworks and mocking libraries, making your codebase more testable and modern.

#### ❌ Avoid
```csharp
var user = _collection.Find(filter).FirstOrDefault(); // blocks the thread
```

✅ Prefer
```csharp
var user = await _collection.Find(filter).FirstOrDefaultAsync(); // async and scalable
```

```csharp
var filter = Builders<User>.Filter.Eq(u => u.Id, id);
var user = await _collection.Find(filter).FirstOrDefaultAsync();
```

---

## 🧪 Testing Guidelines
- Use **xUnit** or **NUnit** for unit tests.
- Structure test folders mirroring your main project.
- Use **Moq** or **FakeItEasy** for mocking dependencies.

```
/ProjectName.Tests
│
├── Services
│   └── TimeTrackingServiceTests.cs
│
└── Repositories
    └── UserRepositoryTests.cs
```

---

## 🌐 API Response Best Practices

### ✅ Use IActionResult or ActionResult<T>
- Return standardized responses using ASP.NET Core's built-in types like `Ok()`, `BadRequest()`, `NotFound()`, etc.
- This provides flexibility and improves API client experience.

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetUser(string id)
{
    var user = await _userService.GetByIdAsync(id);
    if (user == null)
        return NotFound("User not found");

    return Ok(user); // wraps the result in 200 OK
}
```

### ✅ Use ActionResult<T> for better type clarity
```csharp
[HttpPost]
public async Task<ActionResult<UserDto>> CreateUser(CreateUserRequest request)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    var user = await _userService.CreateAsync(request);
    return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
}
```

### ❌ Avoid returning raw objects or unwrapped types

#### ❌ Bad Example: Custom result wrapper that mimics HTTP status manually
```csharp
public Result DoSomething()
{
    var result = new Result();

    if (good)
    {
        result.Success = true;
        result.Data = new { /* ... */ };
        result.StatusCode = 200;
        return result;
    }

    result.Success = false;
    result.Message = "Unauthorized access.";
    result.StatusCode = 401;
    return result;
}
```
This pattern:
- Duplicates the purpose of HTTP status codes
- Makes it harder to integrate with API tooling (like Swagger)
- Is not idiomatic in ASP.NET Core

✅ Prefer using standard `IActionResult` patterns like:
```csharp
if (!authorized)
    return Unauthorized("Unauthorized access.");

return Ok(new { /* ... */ });
```


```csharp
public User GetUser(string id) { ... } // ❌ Bad practice
```

### ✅ Use meaningful error messages
- Always provide a helpful message or validation feedback.
- Use `ProblemDetails` or custom error models when needed.

---

## ✅ Code Style Checklist
- ✅ Use `async/await` for all I/O-bound operations
- ✅ Apply dependency injection throughout the app
- ✅ Group related code together (by feature if possible)
- ✅ Keep controllers thin, push logic to services
- ✅ Handle exceptions globally via middleware

---

## 📌 Other Good Practices
- Use `ILogger<T>` for logging
- Avoid magic strings and numbers — extract constants
- Validate inputs using FluentValidation or data annotations
- Use records for immutable DTOs
- Document public APIs using XML comments or Swagger

---

## 🔗 References
- [.NET Coding Conventions - Microsoft Docs](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [MongoDB C# Driver Docs](https://www.mongodb.com/docs/drivers/csharp/)
- [Clean Architecture Principles](https://github.com/ardalis/CleanArchitecture)

---

> ✨ Following these guidelines ensures your application is easier to maintain, test, and scale.

