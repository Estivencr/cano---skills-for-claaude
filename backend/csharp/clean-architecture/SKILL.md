---
name: clean-architecture
description: Scaffold and explain Clean Architecture (also called Onion Architecture or Hexagonal Architecture) projects in C# with .NET 8. Use this skill whenever the user asks about Clean Architecture, layered architecture, domain-driven design (DDD), separation of concerns, or asks to structure a .NET project with proper layer separation. Triggers on: "clean architecture", "arquitectura limpia", "onion architecture", "DDD", "domain layer", "application layer", "how to structure my project", "capas de arquitectura", or any request to organize a .NET solution beyond a single project. Always use this skill when the user wants a well-structured multi-project .NET solution.
---

# Clean Architecture (.NET 8 + C#)

Scaffolds a Clean Architecture solution: four layers, clearly separated, with dependency rules enforced by project references.

## The Core Rule

> Dependencies point INWARD only. Outer layers depend on inner layers. Inner layers know nothing about outer layers.

```
[ Presentation ] → [ Application ] → [ Domain ]
[ Infrastructure ] → [ Application ] → [ Domain ]
```

---

## Solution Structure

```
MyApp.sln
├── src/
│   ├── MyApp.Domain/           # Entities, Value Objects, Domain Events, Interfaces
│   ├── MyApp.Application/      # Use Cases, DTOs, Interfaces, Validators, Mappings
│   ├── MyApp.Infrastructure/   # EF Core, Repositories, External Services
│   └── MyApp.API/              # Controllers, Middleware, DI wiring
└── tests/
    ├── MyApp.Domain.Tests/
    ├── MyApp.Application.Tests/
    └── MyApp.API.Tests/
```

### Project References (csproj)
```
Domain      → (nothing)
Application → Domain
Infrastructure → Application (+ Domain transitively)
API         → Application + Infrastructure
```

---

## Layer 1: Domain

**Purpose**: Core business logic. No dependencies on any other layer or library.

```
MyApp.Domain/
├── Entities/
│   ├── BaseEntity.cs
│   └── {Entity}.cs
├── ValueObjects/
│   └── Money.cs
├── Enums/
│   └── OrderStatus.cs
├── Exceptions/
│   └── DomainException.cs
└── Interfaces/
    └── Repositories/
        └── I{Entity}Repository.cs
```

### Entity example with domain logic
```csharp
// Domain/Entities/Order.cs
public class Order : BaseEntity
{
    private readonly List<OrderItem> _items = new();

    public string CustomerId { get; private set; }
    public OrderStatus Status { get; private set; } = OrderStatus.Pending;
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();
    public decimal Total => _items.Sum(i => i.UnitPrice * i.Quantity);

    // Factory method — controls creation
    public static Order Create(string customerId)
    {
        if (string.IsNullOrWhiteSpace(customerId))
            throw new DomainException("CustomerId is required.");

        return new Order { CustomerId = customerId };
    }

    // Domain behavior — not in service, in entity
    public void AddItem(int productId, decimal unitPrice, int quantity)
    {
        if (Status != OrderStatus.Pending)
            throw new DomainException("Cannot add items to a non-pending order.");

        _items.Add(new OrderItem(productId, unitPrice, quantity));
    }

    public void Confirm()
    {
        if (!_items.Any())
            throw new DomainException("Cannot confirm an empty order.");

        Status = OrderStatus.Confirmed;
    }
}
```

### Repository interface (Domain defines, Infrastructure implements)
```csharp
// Domain/Interfaces/Repositories/IOrderRepository.cs
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<IEnumerable<Order>> GetByCustomerAsync(string customerId, CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
    Task UpdateAsync(Order order, CancellationToken ct = default);
}
```

---

## Layer 2: Application

**Purpose**: Orchestrates use cases. Depends on Domain only. Contains DTOs, Use Case handlers, validators, mapping interfaces.

```
MyApp.Application/
├── UseCases/
│   └── {Entity}/
│       ├── Commands/
│       │   ├── Create{Entity}/
│       │   │   ├── Create{Entity}Command.cs
│       │   │   └── Create{Entity}CommandHandler.cs
│       │   └── Update{Entity}/
│       │       └── ...
│       └── Queries/
│           ├── Get{Entity}ById/
│           │   ├── Get{Entity}ByIdQuery.cs
│           │   └── Get{Entity}ByIdQueryHandler.cs
│           └── GetAll{Entities}/
│               └── ...
├── DTOs/
│   └── {Entity}Dto.cs
├── Interfaces/
│   └── IUnitOfWork.cs
├── Validators/
│   └── Create{Entity}CommandValidator.cs
└── Mappings/
    └── {Entity}MappingProfile.cs
```

### Command + Handler (CQRS pattern)
```csharp
// Application/UseCases/Orders/Commands/CreateOrder/CreateOrderCommand.cs
public record CreateOrderCommand(string CustomerId) : IRequest<int>;

// Application/UseCases/Orders/Commands/CreateOrder/CreateOrderCommandHandler.cs
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, int>
{
    private readonly IOrderRepository _repository;

    public CreateOrderCommandHandler(IOrderRepository repository)
        => _repository = repository;

    public async Task<int> Handle(CreateOrderCommand request, CancellationToken ct)
    {
        var order = Order.Create(request.CustomerId);
        await _repository.AddAsync(order, ct);
        return order.Id;
    }
}
```

### Query + Handler
```csharp
public record GetOrderByIdQuery(int Id) : IRequest<OrderDto?>;

public class GetOrderByIdQueryHandler : IRequestHandler<GetOrderByIdQuery, OrderDto?>
{
    private readonly IOrderRepository _repository;
    private readonly IMapper _mapper;

    public GetOrderByIdQueryHandler(IOrderRepository repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }

    public async Task<OrderDto?> Handle(GetOrderByIdQuery request, CancellationToken ct)
    {
        var order = await _repository.GetByIdAsync(request.Id, ct);
        return order is null ? null : _mapper.Map<OrderDto>(order);
    }
}
```

### NuGet for Application layer
```bash
dotnet add MyApp.Application package MediatR
dotnet add MyApp.Application package AutoMapper
dotnet add MyApp.Application package FluentValidation
```

---

## Layer 3: Infrastructure

**Purpose**: Implements interfaces defined in Domain and Application. EF Core, external APIs, file system, email, etc.

```
MyApp.Infrastructure/
├── Persistence/
│   ├── AppDbContext.cs
│   ├── Configurations/
│   │   └── OrderConfiguration.cs
│   └── Repositories/
│       └── OrderRepository.cs
├── Services/
│   └── EmailService.cs
└── DependencyInjection.cs    ← registers all infrastructure services
```

### DependencyInjection.cs (Infrastructure extension)
```csharp
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));

        services.AddScoped<IOrderRepository, OrderRepository>();
        // Add other repos and services

        return services;
    }
}
```

---

## Layer 4: API (Presentation)

**Purpose**: HTTP concerns only. No business logic. Receives requests, sends Commands/Queries via MediatR, returns responses.

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly ISender _mediator;

    public OrdersController(ISender mediator) => _mediator = mediator;

    [HttpPost]
    public async Task<IActionResult> Create([FromBody] CreateOrderCommand command)
    {
        var id = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetById), new { id }, null);
    }

    [HttpGet("{id:int}")]
    public async Task<IActionResult> GetById(int id)
    {
        var result = await _mediator.Send(new GetOrderByIdQuery(id));
        return result is null ? NotFound() : Ok(result);
    }
}
```

### Program.cs (API layer wires everything)
```csharp
// Application layer registration
builder.Services.AddMediatR(cfg =>
    cfg.RegisterServicesFromAssembly(typeof(CreateOrderCommand).Assembly));
builder.Services.AddAutoMapper(typeof(CreateOrderCommand).Assembly);
builder.Services.AddValidatorsFromAssembly(typeof(CreateOrderCommand).Assembly);

// Infrastructure layer registration
builder.Services.AddInfrastructure(builder.Configuration);
```

---

## CLI Setup Commands

```bash
# Create solution
dotnet new sln -n MyApp

# Create projects
dotnet new classlib -n MyApp.Domain -o src/MyApp.Domain
dotnet new classlib -n MyApp.Application -o src/MyApp.Application
dotnet new classlib -n MyApp.Infrastructure -o src/MyApp.Infrastructure
dotnet new webapi -n MyApp.API -o src/MyApp.API

# Add to solution
dotnet sln add src/MyApp.Domain
dotnet sln add src/MyApp.Application
dotnet sln add src/MyApp.Infrastructure
dotnet sln add src/MyApp.API

# Set project references
dotnet add src/MyApp.Application reference src/MyApp.Domain
dotnet add src/MyApp.Infrastructure reference src/MyApp.Application
dotnet add src/MyApp.API reference src/MyApp.Application
dotnet add src/MyApp.API reference src/MyApp.Infrastructure
```

---

## Clean Architecture Checklist

- [ ] Domain has zero external NuGet dependencies
- [ ] Application only references Domain (no EF Core, no HTTP)
- [ ] Infrastructure implements interfaces from Domain/Application
- [ ] Controllers use MediatR — no direct service/repo injection
- [ ] No `new` for services in constructors — everything via DI
- [ ] Business rules live in Domain entities, not in services
- [ ] DTOs only cross layer boundaries, not entities

---

## Reference Files
- `references/domain-events.md` — Domain event pattern with MediatR notifications
- `references/unit-of-work.md` — Unit of Work pattern with transaction support
