# natram-skills

Personal Claude skills library — custom instructions that extend Claude's capabilities for specific workflows.

## Structure

```
cano-skills/
├── frontend/
│   ├── landing-page-builder/   # Full landing pages & multi-section sites
│   ├── ui-components/          # Reusable UI components (cards, modals, navbars…)
│   └── animation-effects/      # Animations, transitions & visual effects
└── backend/
    └── csharp/
        ├── aspnet-api-generator/  # Complete ASP.NET Core REST APIs
        ├── ef-core-crud/          # EF Core data layer + Repository pattern
        └── clean-architecture/    # Multi-layer Clean Architecture solutions
```

## Skills Index

### Frontend
| Skill | Trigger |
|---|---|
| `landing-page-builder` | landing page, home page, marketing site, hero section |
| `ui-components` | card, modal, navbar, toast, accordion, tabs, dropdown |
| `animation-effects` | animación, scroll effect, hover effect, micro-interaction |

### Backend — C# / .NET 8
| Skill | Trigger |
|---|---|
| `aspnet-api-generator` | create API, REST API, Web API, endpoint, controller |
| `ef-core-crud` | Entity Framework, DbContext, migrations, repository, model |
| `clean-architecture` | Clean Architecture, DDD, CQRS, onion architecture, project structure |

## How Skills Work

Each skill is a `SKILL.md` file with YAML frontmatter (name + description) and detailed instructions. Claude reads the description to decide when to use a skill, then reads the full file for implementation guidance.

---

Built by [@Estivencr](https://github.com/Estivencr)
