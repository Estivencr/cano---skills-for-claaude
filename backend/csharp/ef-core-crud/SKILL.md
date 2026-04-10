---
name: ef-core-crud
description: Generate Entity Framework Core data layer code for .NET 8 and SQL Server. Use this skill whenever the user asks to set up EF Core, create a DbContext, write entity models, generate migrations, or build a data access layer in C#. Triggers on: "Entity Framework", "EF Core", "DbContext", "migrations", "create a model", "database first", "code first", "repository pattern", "IQueryable", or any request involving database access in .NET. Always use this skill for data-layer requests in C# projects.
---

# EF Core CRUD (.NET 8 + SQL Server)

Generates complete Entity Framework Core data access layers: entities, DbContext, Repository pattern, and migrations guide.

## Stack

| Package | Version |
|---|---|
| Microsoft.EntityFrameworkCore | 8.x |
| Microsoft.EntityFrameworkCore.SqlServer | 8.x |
| Microsoft.EntityFrameworkCore.Tools | 8.x |
| Microsoft.EntityFrameworkCore.Design | 8.x |

Install command:
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 8.*
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 8.*
dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.*
```

---

## Phase 1: Entity Model

### Base Entity (always include)
```csharp
public abstract class BaseEntity
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
}
```

### Entity Example
```csharp
public class Product : BaseEntity
{
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public bool IsActive { get; set; } = true;

    // Navigation property (FK example)
    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;
}
```

### Fluent API Configuration (preferred over Data Annotations)
```csharp
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("Products");

        builder.HasKey(x => x.Id);

        builder.Property(x => x.Name)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(x => x.Price)
            .HasPrecision(18, 2);

        builder.Property(x => x.Description)
            .HasMaxLength(500);

        builder.HasOne(x => x.Category)
            .WithMany(c => c.Products)
            .HasForeignKey(x => x.CategoryId)
            .OnDelete(DeleteBehavior.Restrict);

        // Index
        builder.HasIndex(x => x.Name).IsUnique();
    }
}
```

---

## Phase 2: DbContext

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();
    public DbSet<Category> Categories => Set<Category>();
    // Add DbSets here

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Apply all IEntityTypeConfiguration in the assembly
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);

        base.OnModelCreating(modelBuilder);
    }

    public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // Auto-set UpdatedAt on modified entities
        foreach (var entry in ChangeTracker.Entries<BaseEntity>())
        {
            if (entry.State == EntityState.Modified)
                entry.Entity.UpdatedAt = DateTime.UtcNow;
        }

        return base.SaveChangesAsync(cancellationToken);
    }
}
```

---

## Phase 3: Generic Repository Pattern

### Interface
```csharp
public interface IRepository<T> where T : BaseEntity
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T?> GetByIdAsync(int id);
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
    Task<bool> ExistsAsync(int id);
    IQueryable<T> Query(); // for custom queries with Include/Where
}
```

### Implementation
```csharp
public class Repository<T> : IRepository<T> where T : BaseEntity
{
    protected readonly AppDbContext _context;
    protected readonly DbSet<T> _dbSet;

    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<IEnumerable<T>> GetAllAsync()
        => await _dbSet.AsNoTracking().ToListAsync();

    public async Task<T?> GetByIdAsync(int id)
        => await _dbSet.AsNoTracking().FirstOrDefaultAsync(x => x.Id == id);

    public async Task<T> AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }

    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(T entity)
    {
        _dbSet.Remove(entity);
        await _context.SaveChangesAsync();
    }

    public async Task<bool> ExistsAsync(int id)
        => await _dbSet.AnyAsync(x => x.Id == id);

    public IQueryable<T> Query()
        => _dbSet.AsQueryable();
}
```

### Specific Repository (when you need custom queries)
```csharp
public interface IProductRepository : IRepository<Product>
{
    Task<IEnumerable<Product>> GetByCategoryAsync(int categoryId);
    Task<IEnumerable<Product>> GetActiveAsync();
}

public class ProductRepository : Repository<Product>, IProductRepository
{
    public ProductRepository(AppDbContext context) : base(context) { }

    public async Task<IEnumerable<Product>> GetByCategoryAsync(int categoryId)
        => await Query()
            .Include(p => p.Category)
            .Where(p => p.CategoryId == categoryId && p.IsActive)
            .AsNoTracking()
            .ToListAsync();

    public async Task<IEnumerable<Product>> GetActiveAsync()
        => await Query()
            .Where(p => p.IsActive)
            .AsNoTracking()
            .ToListAsync();
}
```

---

## Phase 4: Migrations Workflow

```bash
# Add first migration
dotnet ef migrations add InitialCreate --project MyApi

# Apply to database
dotnet ef database update

# Add subsequent migration
dotnet ef migrations add Add{FeatureName}

# Rollback to a specific migration
dotnet ef database update {MigrationName}

# Remove last migration (if not applied)
dotnet ef migrations remove

# Generate SQL script (for production deployments)
dotnet ef migrations script --output migration.sql
```

---

## EF Core Best Practices

| Practice | Why |
|---|---|
| `AsNoTracking()` on read-only queries | Faster, less memory |
| Fluent API over Data Annotations | Separation of concerns |
| `IQueryable` for composable queries | Avoids N+1 |
| `Include()` for navigation properties | Explicit eager loading |
| `HasPrecision(18,2)` for `decimal` | Prevents SQL type mismatch |
| Never expose `DbContext` outside data layer | Encapsulation |
| Seed data in `OnModelCreating` with `HasData()` | Reproducible dev environments |

---

## Soft Delete Pattern (optional)
```csharp
// Add to BaseEntity:
public bool IsDeleted { get; set; } = false;

// Add to DbContext OnModelCreating:
modelBuilder.Entity<Product>().HasQueryFilter(p => !p.IsDeleted);

// Delete becomes:
entity.IsDeleted = true;
await _context.SaveChangesAsync();
```

---

## Reference Files
- `references/seeding.md` — Data seeding patterns with HasData and custom seeders
- `references/pagination.md` — Keyset and offset pagination with IQueryable
