# Philosophy Decision Guide

Flowcharts for the meta-decisions: when to add gems, when to ship, how to structure code, and when to build vs. depend.

---

## Should I add this gem?

> Use this before running `bundle add` on anything.

```mermaid
flowchart TD
    A["I want to add a gem"] --> B{"Does Rails already\nprovide this?"}
    B -->|Yes| C["Use the Rails built-in\nno gem needed"]
    B -->|No| D{"Can I build it in\nunder 200 lines?"}
    D -->|Yes| E["Build it yourself first\nadd the gem later if requirements grow"]
    D -->|No| F{"Is it a well-defined\nprotocol or standard?\ne.g. OAuth, S3, SMTP"}
    F -->|Yes| G["Gem is appropriate\nuse the canonical one"]
    F -->|No| H{"Does it replace\na Rails built-in?\ne.g. auth, queues, caching"}
    H -->|Yes| I["Don't gem it\nRails built-ins are the right tool"]
    H -->|No| J["Evaluate carefully\ncheck what-they-avoid.md first"]
```

**Why?** Every gem is a dependency you must maintain, upgrade, and debug through. 37signals has built authentication (~150 lines), background jobs (Solid Queue), and caching systems from scratch because owning the code means understanding it.

→ See [`what-they-avoid.md`](../what-they-avoid.md) and [`development-philosophy.md`](../development-philosophy.md)

---

## Should I ship this now or keep polishing?

> Use this when you have working code but you're unsure if it's ready.

```mermaid
flowchart TD
    A["My feature works\nbut the code could be better"] --> B{"Are there known\nroot cause issues?"}
    B -->|Yes| C["Fix the root cause first\nnot the symptom\nthen ship"]
    B -->|No| D{"Do tests pass?"}
    D -->|No| E["Fix the tests\nthen reconsider"]
    D -->|Yes| F{"Is this for internal\nuse or user-facing?"}
    F -->|Internal| G["Ship it\nShip / Validate / Refine\nprototype quality is fine"]
    F -->|User-facing| H{"Is it a prototype\nor a final implementation?"}
    H -->|Prototype| G
    H -->|Final| I["Ship it\nrefine based on real feedback\nnot hypothetical polish"]
```

**Why?** The Ship/Validate/Refine cycle is 37signals' core rhythm. Shipping prototype-quality code internally is encouraged — you learn more from real use than from polishing in isolation. Root cause fixes are the only non-negotiable before shipping.

→ See [`development-philosophy.md`](../development-philosophy.md)

---

## Service object or something else?

> Use this when you're tempted to create a `app/services/` directory.

```mermaid
flowchart TD
    A["I need to extract\nbusiness logic from somewhere"] --> B{"Thinking of a\nservice object?"}
    B -->|Yes| C["Stop — 37signals avoids service objects\nthey scatter domain logic"]
    C --> D
    B -->|No| D{"Is the logic shared\nacross 2 or more models?"}
    D -->|Yes| E["Model concern\nmodule Archivable, Notifiable, etc."]
    D -->|No| F{"Is it a complex\noperation with multiple steps?"}
    F -->|Yes| G["PORO called from the model\nnot from the controller\ne.g. card.archive! calls Archiver.new(card).call"]
    F -->|No| H["Model method\nkeep it on the model — simplest is best"]
```

**Why?** Service objects push domain logic out of models into disconnected classes, making it harder to find and test. A PORO called *from* the model keeps the model as the public API while still isolating complex logic.

→ See [`models.md`](../models.md) and [`development-philosophy.md`](../development-philosophy.md)

---

## Should I add a new pattern or dependency?

> Use this when you're considering introducing something the codebase doesn't already use.

```mermaid
flowchart TD
    A["I need a new capability\nor pattern"] --> B{"Does vanilla Rails cover it?"}
    B -->|Yes| C["Use Rails\nno addition needed"]
    B -->|No| D{"Does Turbo cover it?"}
    D -->|Yes| E["Use Turbo"]
    D -->|No| F{"Does Stimulus cover it?"}
    F -->|Yes| G["Use Stimulus"]
    F -->|No| H{"Is it a core infrastructure\nconcern?\ne.g. auth, queues, email"}
    H -->|Yes| I["Build it yourself\n37signals owns its infrastructure"]
    H -->|No| J{"Is there a focused gem\nthat does exactly this one thing?"}
    J -->|Yes| K["Consider the gem\nbut read what-they-avoid.md first"]
    J -->|No| I
```

**Why?** The hierarchy is: Rails → Turbo → Stimulus → build it → gem it. Working up the stack in that order keeps the codebase lean and ensures you understand each layer before adding the next.

→ See [`what-they-avoid.md`](../what-they-avoid.md)
