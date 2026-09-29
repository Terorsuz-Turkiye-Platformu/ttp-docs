# AGENTS.md — YuksiMain (Arc's DDD)

> **Read this before writing any domain, persistence, or API code.**  
> Full playbook: [`arc-ddd.md`](./arc-ddd.md).  
> Architecture: **Arc's DDD** (Australian DDD) by Arc (Arda Akgur). This is **not** classic onion/clean DDD.

---

## North star

**Arc's DDD = classic layered DDD + horizontal bounded-context folders in Core + per-context entity projections over shared tables so each domain only loads and ships the slice of the world it needs.**

Decisive rule:

> **Bounded contexts are folders. Shared database tables are allowed. Shared “god entities” are not.**

---


### Core

1. **No** `Microsoft.EntityFrameworkCore` (or any infra package) in Core.
2. **No** project references from Core to Domain or API.
3. **No** cross-folder imports between Core BCs.  
   - `Events` must not `using YuksiMain.Core.Users`.  
   - Define a **local projection** instead (`User2`, `UserForEvents`, etc.).
4. Entities: private parameterless ctor for EF, public ctor with invariants, behaviour methods for meaningful state changes.
5. Public get/set is OK (EF-friendly rich model). Prefer methods when there is a rule.
6. When two BCs need the same satellite shape, **duplicate the CLR type** in each namespace (same table, separate types). Prefer projections when fields/behaviour diverge.

### Domain

1. **One DbContext per horizontal BC.**
2. Map projections with `.ToTable("SamePhysicalTable")` + own key.
3. Navigations always point at the **local** projection type, never the owning BC’s full entity.
4. Repository interfaces return **domain-local** types only (`IQueryable<User2>`, not owning `User`).
5. Each repository owns / uses its own DbContext; `SaveChanges` is the UoW boundary for that context.
6. Prefer `IConfiguration` / `appsettings` for connections — not hardcoded secrets.
7. **Async rule (strict):** Methods named `*Async` or returning `Task`/`Task<T>` must use the `async` modifier and `await` on every async call (EF, HTTP, file I/O, etc.). **Never** return a `Task` from an EF/async API without `async`/`await` just to satisfy the signature — that looks async but is not. If a method must stay async but has no natural `await`, offload blocking work with `Task.Run(...)` (or an equivalent) and `await` that — do not leave a synchronous body on an `async Task` method.

### API

1. Controllers/services may inject **multiple** repositories when a use-case spans BCs.
2. DTOs live under `Models/` (mirror BC folders when helpful).
3. **DTO-only responses (hard rule):** HTTP responses must use API DTOs only. **Never** return Core entities, Domain/persistence types, EF entities, repository results, or any object that came from the database. **Never** expose indirect references (e.g. `FromUser(entity)` mappers on DTOs, returning entities wrapped in `ActionResult<TEntity>`, or serializing navigations). Map to DTOs (or auth tokens) inside services using explicit primitive fields only.
4. Never leak secret-bearing fields in DTOs (password hashes, refresh secrets, etc.).
5. Prefer DI for contexts/repos (Scoped). Do not `new` repositories in controllers in new code.
6. **Async rule (strict):** All controller endpoints, application services, and repository methods that touch I/O must be `async Task` / `async Task<T>`. Use `await` on DB, HTTP, and file calls. **Always** declare `async` on `*Async` methods — do not pass through `Task` without `await` (e.g. `return _context.Users.FirstOrDefaultAsync(...)` with no `async`/`await` is forbidden). Prefer real async APIs (`FirstOrDefaultAsync`, `SaveChangesAsync`, …). If there is genuinely nothing to `await`, wrap blocking work in `Task.Run(...)` and `await` it; never perform sync DB or I/O on the request thread inside an endpoint or service.
7. **Auth responses:** Login and register return **JWT only** (`token`, `expires_at`) — not user profile objects. Use `[Authorize]` on protected endpoints; read claims from the token, not by re-serializing the user row.

### Docs (mandatory — fail the PR if skipped)

Every code change **must** update [`docs/`](./docs/). A task is not done until the matching doc is written or revised. Chat summaries do not count.

1. New or changed capability (endpoint, service, entity, table, config, behaviour) → create or update the topic file under `docs/` (e.g. [`docs/email.md`](./docs/email.md)).
2. Architecture / layering / BC layout change → update [`docs/architecture.md`](./docs/architecture.md).
3. Schema, migration, or table ownership change → update [`docs/database.md`](./docs/database.md).
4. Shared constants / connection / JWT → update [`constants.md`](./constants.md).
5. Do **not** ship a feature with docs that still describe the previous behaviour.

---

## Projection pattern (the distinctive technique)

When Domain A needs an entity Domain B owns:

1. List **only** the fields + behaviours A needs.
2. Create a local CLR type in A’s Core folder.
3. Map it to the **same** table in A’s DbContext.
4. Point navigations at that local type.
5. Expose it from A’s repository.
6. **Never** pull B’s full aggregate into A’s context.

```csharp
// Ordering BC — only what Ordering needs
namespace YuksiMain.Core.Ordering
{
    public class Customer2
    {
        public Guid CustomerId { get; set; }
        public string DisplayName { get; set; }
        // ... only Ordering fields

        private Customer2() { } // EF

        public void SomeOrderingBehaviour() { /* ... */ }
    }
}

// OrderingDbContext
modelBuilder.Entity<Customer2>().ToTable("Customers").HasKey(c => c.CustomerId);
```

Naming: `User` / `User2` / `User3` is fine; clearer names (`CustomerForOrdering`) are preferred in new code. The idea is **projection identity**, not the suffix.

EF loads **only properties declared on the CLR type** → narrower `SELECT`s. No special `Ignore()` list required for this pattern.

---

## Decision table

| Situation | Do this |
|-----------|---------|
| New concept owned by one BC | Entity in that Core folder + map in that DbContext |
| Another BC needs a few fields | Projection in consuming BC + same table mapping |
| Another BC needs different behaviours on same row | Separate type + methods (`Event` vs `Event2`) |
| Same satellite in two BCs | Duplicate class in both namespaces |
| Use-case needs two BCs | Compose in API controller/service — never Core cross-`using` |
| Infra (SMS, email, PSP, bots) | API `Services/` (or a future infra project) — Core stays pure |
| Tempted to put full `User` in another BC’s context | **Stop.** Use a projection. |
| SQL / SSDT project | Optional; **not** required for Arc's DDD |

---

## Scaffold checklist for new BCs

- [ ] Core folder for the BC (no cross-BC usings)
- [ ] Owning entities in the owning BC only
- [ ] Projections for foreign tables the BC only peeks into
- [ ] Matching `*DbContext` in Domain with `.ToTable(...)` maps
- [ ] `Interfaces/I*Repository` + `Repositories/*Repository` returning local types
- [ ] API composition for multi-BC use-cases
- [ ] DTOs under API Models; no secret leakage
- [ ] `docs/` updated (topic file + architecture/database if those changed)

---

## Anti-patterns (do not introduce)

- One shared god `User` / aggregate referenced from every BC
- Cross-BC `using` inside Core
- EF packages in Core
- Single “god” DbContext for the whole app
- Classic Application-layer mediation as a **required** ring (not this architecture)
- Treating Arc's DDD as onion with renamed folders
- Returning Core/Domain/DB types from API controllers (including `FromEntity()` DTO helpers that accept live entities)
- Fake-async: `*Async` methods that omit the `async` modifier and only `return` an inner `Task` without `await`
- Shipping a code change without updating `docs/`

---

## Style notes for generated code

- Match existing naming: `YuksiMain.Core`, `YuksiMain.Domain`, `YuksiMain.API`.
- Prefer modern config/DI over Goldtag-era hardcoded connections and `new Repository()`.
- Copy **ideas** from [`arc-ddd.md`](./arc-ddd.md); do not invent Goldtag business logic here.
- When unsure, re-read sections 1, 3, 4, and 8 of `arc-ddd.md`.

---

## One-line reminder

**Folders are bounded contexts; tables may be shared; entities are not — project locally.**
