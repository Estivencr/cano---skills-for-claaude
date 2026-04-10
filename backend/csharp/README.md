# Backend Skills — C# / .NET

Skills para construir backends en C# con .NET 8, ASP.NET Core, Entity Framework Core y SQL Server.

## Skills disponibles

### `aspnet-api-generator`
Genera Web APIs REST completas con ASP.NET Core: controllers, services, repositories, DTOs, validación con FluentValidation, logging con Serilog, y Swagger.

**Triggerear cuando:** el usuario pide crear una API, un endpoint, o un backend HTTP en C#.

**Archivos de referencia:**
- `references/jwt-auth.md` — Setup completo de JWT Authentication

---

### `ef-core-crud`
Genera la capa de datos con EF Core: entidades, DbContext, configuración Fluent API, Repository pattern genérico y específico, y guía de migraciones.

**Triggerear cuando:** el usuario pide modelos de base de datos, DbContext, migraciones, o acceso a datos en .NET.

**Archivos de referencia:**
- `references/seeding.md` — Seeding con HasData y seeders personalizados *(próximamente)*
- `references/pagination.md` — Paginación keyset y offset *(próximamente)*

---

### `clean-architecture`
Scaffoldea soluciones multi-proyecto con Clean Architecture: Domain, Application, Infrastructure, API — con CQRS via MediatR y AutoMapper.

**Triggerear cuando:** el usuario quiere estructurar un proyecto .NET en capas, o menciona Clean Architecture, DDD, CQRS, o separación de responsabilidades.

**Archivos de referencia:**
- `references/domain-events.md` — Domain Events con MediatR *(próximamente)*
- `references/unit-of-work.md` — Unit of Work con transacciones *(próximamente)*

---

## Stack Base

| Tecnología | Versión |
|---|---|
| .NET | 8.0 |
| ASP.NET Core | 8.0 |
| Entity Framework Core | 8.x |
| SQL Server | 2019+ |
| MediatR | 12.x |
| FluentValidation | 11.x |
| AutoMapper | 13.x |
| Serilog | 8.x |

## Principios Generales

- **Async/await** en toda la cadena
- **DTOs** en los límites de capa — nunca exponer entidades directamente
- **Dependency Injection** via constructor, sin `new` para servicios
- **Repository pattern** para aislar EF Core
- **Fluent API** sobre Data Annotations para configuración de EF
- **AsNoTracking()** en todas las consultas de solo lectura
