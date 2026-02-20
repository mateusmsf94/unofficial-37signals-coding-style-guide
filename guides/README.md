# Implementation Decision Guides

These guides help junior Rails developers navigate *when* to use each pattern from the 37signals style. Rather than explaining what the patterns are, each guide frames common decision points as flowcharts — so you can make the right call quickly.

All guides link back to the source documentation for deeper reading.

## Guides

| Guide | What it helps you decide |
|-------|--------------------------|
| [Core Rails](core-rails.md) | Where logic lives, boolean vs. record, concerns vs. POROs, authorization |
| [Frontend](frontend.md) | Turbo Frames vs. Streams, when to write Stimulus, which CSS layer |
| [Backend](backend.md) | Background jobs vs. inline, caching, multi-tenancy scoping, authentication |
| [Philosophy](philosophy.md) | When to add a gem, when to ship, service objects, new dependencies |

> **Note:** All patterns here are extracted from 37signals' Fizzy codebase. The source files in the root of this repo contain full explanations and code examples.
