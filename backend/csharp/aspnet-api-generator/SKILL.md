---
name: aspnet-api-generator
description: Generate complete, production-ready ASP.NET Core Web API projects targeting .NET 8. Use this skill whenever the user asks to create, scaffold, or build a REST API, Web API, or HTTP service in C#. Triggers on: "create an API", "build a REST API", "crea una API", "hazme un endpoint", "necesito un Web API", "scaffold a controller", or any request to build an HTTP backend in C# or .NET. Also use this skill when the user describes a domain/entity and wants an API around it. Always use this skill for API-level requests, not just for individual classes.
---

# ASP.NET Core API Generator (.NET 8)

Generates complete, production-ready ASP.NET Core Web API projects or individual API layers.

## Stack & Versions

| Package | Version |
|---|---|
| .NET | 8.0 |
| ASP.NET Core | 8.0 |
| Swashbuckle (Swagger) | 6.x |
| FluentValidation.AspNetCore | 11.x |
| Serilog.AspNetCore | 8.x |

---

## Phase 1: Understand the Domain

Before generating code, identify:
- **Entity/Resource**: What is the main resource? (e.g., `Product`, `Order`, `User`)
- **Operations**: Which HTTP methods? (GET all, GET by id, POST, PUT, DELETE, PATCH)
- **Auth**: Public API, JWT-protected, or API key?
- **Relationships**: Any navigation properties / foreign keys?

If the user provides enough context, proceed directly.

---

## Phase 2: Project Structure

For a new API project, generate this structure:

```
MyApi/
├── MyApi.csproj
├── Program.cs
├── appsettings.json
├── appsettings.Development.json
├── Controllers/
│   └── {Entity}Controller.cs
├── Models/
│   ├── {Entity}.cs
│   └── DTOs/
│       ├── {Entity}CreateDto.cs
│       ├── {Entity}UpdateDto.cs
│       └── {Entity}ResponseDto.cs
├── Services/
│   ├── I{Entity}Service.cs
│   └── {Entity}Service.cs
├── Repositories/
│   ├── I{Entity}Repository.cs
│   └── {Entity}Repository.cs
├── Data/
│   └── AppDbContext.cs
├── Validators/
│   └── {Entity}CreateDtoValidator.cs
└── Middleware/
    └── ExceptionHandlingMiddleware.cs
```

For individual endpoint requests (not full projects), generate only the relevant files.

---

## Phase 3: Code Templates

### Program.cs (.NET 8 minimal hosting)
```csharp
using Microsoft.EntityFrameworkCore;
using Serilog;
using FluentValidation;
using MyApi.Data;
using MyApi.Services;
using MyApi.Repositories;
using MyApi.Middleware;

var builder = WebApplication.CreateBuilder(args);

// Logging
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .CreateLogger();
builder.Host.UseSerilog();

// Services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// DI - Repositories & Services
builder.Services.AddScoped<I{Entity}Repository, {Entity}Repository>();
builder.Services.AddScoped<I{Entity}Service, {Entity}Service>();

// Validation
builder.Services.AddValidatorsFromAssemblyContaining<Program>();

// AutoMapper (optional)
builder.Services.AddAutoMapper(typeof(Program));

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseMiddleware<ExceptionHandlingMiddleware>();
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Controller
```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class {Entity}Controller : ControllerBase
{
    private readonly I{Entity}Service _service;
    private readonly ILogger<{Entity}Controller> _logger;

    public {Entity}Controller(I{Entity}Service service, ILogger<{Entity}Controller> logger)
    {
        _service = service;
        _logger = logger;
    }

    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<{Entity}ResponseDto>), StatusCodes.Status200OK)]
    public async Task<IActionResult> GetAll()
    {
        var items = await _service.GetAllAsync();
        return Ok(items);
    }

    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof({Entity}ResponseDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetById(int id)
    {
        var item = await _service.GetByIdAsync(id);
        if (item is null) return NotFound();
        return Ok(item);
    }

    [HttpPost]
    [ProducesResponseType(typeof({Entity}ResponseDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Create([FromBody] {Entity}CreateDto dto)
    {
        var created = await _service.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpPut("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Update(int id, [FromBody] {Entity}UpdateDto dto)
    {
        var success = await _service.UpdateAsync(id, dto);
        if (!success) return NotFound();
        return NoContent();
    }

    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Delete(int id)
    {
        var success = await _service.DeleteAsync(id);
        if (!success) return NotFound();
        return NoContent();
    }
}
```

### Exception Handling Middleware
```csharp
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsJsonAsync(new
            {
                error = "An unexpected error occurred.",
                detail = ex.Message
            });
        }
    }
}
```

### FluentValidation Validator
```csharp
public class {Entity}CreateDtoValidator : AbstractValidator<{Entity}CreateDto>
{
    public {Entity}CreateDtoValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required.")
            .MaximumLength(100).WithMessage("Name must not exceed 100 characters.");

        // Add rules per property
    }
}
```

---

## Phase 4: appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database={DbName};Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Serilog": {
    "MinimumLevel": "Information",
    "WriteTo": [
      { "Name": "Console" },
      { "Name": "File", "Args": { "path": "logs/log-.txt", "rollingInterval": "Day" } }
    ]
  },
  "AllowedHosts": "*"
}
```

---

## Code Standards

- **Async all the way**: All service/repository methods return `Task<T>`
- **DTOs**: Never expose domain entities directly from controllers
- **Dependency Injection**: Always inject via constructor, never `new`
- **Response types**: Annotate with `[ProducesResponseType]` for Swagger accuracy
- **Naming**: PascalCase classes, camelCase variables, `I` prefix on interfaces
- **No magic strings**: Use `nameof()`, constants, or enums

---

## Reference Files
- `references/jwt-auth.md` — JWT authentication setup pattern
- `references/response-wrapper.md` — Standard `ApiResponse<T>` wrapper pattern
