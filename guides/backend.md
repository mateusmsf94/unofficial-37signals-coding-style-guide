# Backend Decision Guide

Flowcharts for background jobs, caching, multi-tenancy scoping, and authentication.

---

## Should this run inline or as a background job?

> Use this when writing code that might be slow or touch external services.

```mermaid
flowchart TD
    A["I need to run some code\nafter a user action"] --> B{"Does it touch an\nexternal service?\ne.g. email, webhook, API"}
    B -->|Yes| C["Background Job\nalways async for external calls"]
    B -->|No| D{"Does it take\nmore than ~500ms?"}
    D -->|Yes| C
    D -->|No| E{"Must it complete\nbefore sending the response?"}
    E -->|Yes| F["Inline\nin the model or controller"]
    E -->|No| G{"Must it run after\nthe DB transaction commits?"}
    G -->|Yes| H["Job with enqueue_after_transaction_commit\navoids race conditions"]
    G -->|No| F
```

**Why?** Background jobs keep response times fast and make failures retryable. `enqueue_after_transaction_commit` is critical — enqueueing inside a transaction can cause the job to run before the data it needs is actually committed.

→ See [`background-jobs.md`](../background-jobs.md)

---

## Should I cache this?

> Use this before adding any caching layer.

```mermaid
flowchart TD
    A["Should I cache this?"] --> B{"Is it expensive to compute?\ne.g. complex query, aggregation"}
    B -->|No| C["Don't cache\nRails is fast enough for simple queries"]
    B -->|Yes| D{"Is it real-time sensitive?\ne.g. live count, presence"}
    D -->|Yes| E["Don't cache — use Turbo instead\nbroadcast updates via ActionCable"]
    D -->|No| F{"Is it per-user?"}
    F -->|Yes| G["Fragment cache\nkey on user + record + updated_at"]
    F -->|No| H{"Is it an HTTP response\ne.g. a show action?"}
    H -->|Yes| I["HTTP caching\nETag / Last-Modified headers"]
    H -->|No| J["Fragment cache\nkey on record updated_at"]
```

**Why?** Over-caching creates stale data bugs. Start with no cache, measure, and add caching only where profiling shows a real bottleneck. For real-time data, Turbo broadcasts are a better answer than short TTLs.

→ See [`caching.md`](../caching.md) and [`performance.md`](../performance.md)

---

## How should I scope this for multi-tenancy?

> Use this every time you write a query or send data on behalf of a user.

```mermaid
flowchart TD
    A["I'm writing code that reads\nor writes data"] --> B{"Where is this code?"}
    B -->|"Controller or Model"| C["Scope to Current.account\ncurrent_account.boards.find(id)"]
    B -->|"Background Job"| D["Inherit from AccountJob\ntenant is preserved automatically"]
    B -->|"Mailer"| E["Inherit from tenant-scoped\nmailer base class"]
    B -->|"ActionCable channel"| F["stream_for current_account\nscope broadcasts to account"]
    C --> G{"Did you forget\nCurrent.account?"}
    G -->|"Yes — unscoped query"| H["Stop — this is a security risk\ncross-tenant data leak"]
    G -->|No| I["You're scoped correctly"]
```

**Why?** Every database query must be scoped to the current account. An unscoped `Board.find(id)` can return data from another tenant. 37signals uses path-based tenancy with `Current.account` as the ubiquitous scope.

→ See [`multi-tenancy.md`](../multi-tenancy.md) and [`background-jobs.md`](../background-jobs.md)

---

## Authentication: gem or custom?

> Use this when starting a new Rails app or adding authentication to an existing one.

```mermaid
flowchart TD
    A["I need user authentication"] --> B{"Thinking of adding Devise?"}
    B -->|Yes| C["Stop — 37signals does not use Devise\nbuild passwordless instead"]
    B -->|No| D{"Do users need\nOAuth or SSO?"}
    D -->|Yes| E["Add only the gem for that provider\ne.g. omniauth-google-oauth2"]
    D -->|No| F{"Do users need\nemail + password?"}
    F -->|Yes| G["Use has_secure_password\nno gem needed\nRails provides this built-in"]
    F -->|No| H["Build magic link auth\n~150 lines, no gem\nTokens table + mailer + session controller"]
    C --> H
```

**Why?** Devise solves problems most apps don't have and obscures Rails conventions behind DSLs. A passwordless magic link implementation is ~150 lines of plain Rails — readable, debuggable, and ownable.

→ See [`authentication.md`](../authentication.md)
