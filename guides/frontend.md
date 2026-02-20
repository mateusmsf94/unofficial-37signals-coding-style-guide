# Frontend Decision Guide

Flowcharts for choosing between Turbo primitives, Stimulus, and CSS patterns.

---

## Does this UI update need JavaScript at all?

> Use this before writing any frontend code. Turbo handles more than you think.

```mermaid
flowchart TD
    A["I need to update the UI"] --> B{"Is it a standard\nform submission or link click?"}
    B -->|Yes| C["No JS needed\nTurbo intercepts automatically"]
    B -->|No| D{"Is it a single region update\ntriggered by the current user?"}
    D -->|Yes| E["Turbo Frame\nwrap the region in turbo-frame"]
    D -->|No| F{"Does it update multiple\nDOM regions at once?"}
    F -->|Yes| G["Turbo Stream\nappend / prepend / replace / remove"]
    F -->|No| H{"Is it custom browser behavior\ne.g. copy text, drag, animate?"}
    H -->|Yes| I["Stimulus controller"]
    H -->|No| J["Turbo Frame is probably enough\ncheck the Turbo Frame guide first"]
```

**Why?** Turbo eliminates the need for JavaScript in the vast majority of UI interactions. Reach for Stimulus only when Turbo genuinely cannot do it.

→ See [`hotwire.md`](../hotwire.md)

---

## Turbo Frame or Turbo Stream?

> Use this once you've determined you need a Hotwire update.

```mermaid
flowchart TD
    A["I need a Hotwire-powered update"] --> B{"Should other users\nsee this update in real time?"}
    B -->|Yes| C["Turbo Stream via ActionCable\nbroadcast_replace_to / broadcast_append_to"]
    B -->|No| D{"Does the update touch\nmore than one DOM region?"}
    D -->|Yes| E["Turbo Stream in the controller response\nformat.turbo_stream { render ... }"]
    D -->|No| F{"Is it triggered by\nnavigation or lazy loading?"}
    F -->|Yes| G["Turbo Frame with src= attribute\nloads content on visit or intersection"]
    F -->|No| H["Turbo Frame\nstandard in-place replace on form submit"]
```

**Why?** Frames scope updates to one region and are driven by user action. Streams can target multiple regions and can be broadcast server-side. When in doubt, start with a Frame — upgrade to Stream if you need multi-target or broadcast.

→ See [`hotwire.md`](../hotwire.md) and [`actioncable.md`](../actioncable.md)

---

## Should I create a new Stimulus controller?

> Use this when you think you need to write JavaScript for a UI behavior.

```mermaid
flowchart TD
    A["I need JavaScript behavior"] --> B{"Does a catalog controller\nalready handle this?\ne.g. clipboard, reveal, auto-submit"}
    B -->|Yes| C["Reuse the existing controller\ndata-controller='clipboard'"]
    B -->|No| D{"Can Turbo handle it\nwithout any JavaScript?"}
    D -->|Yes| E["Use Turbo\nskip the Stimulus controller"]
    D -->|No| F{"Will this be reused\nacross multiple views?"}
    F -->|Yes| G["Create a generic controller\nname it by behavior not by screen\ne.g. 'toggle' not 'sidebar'"]
    F -->|No| H["Focused controller for this screen\nor inline data- attributes only"]
```

**Why?** The Stimulus catalog in `stimulus.md` has many reusable controllers ready to drop in. New controllers should be named by their generic behavior — `toggle`, `clipboard`, `countdown` — so they can be reused without modification.

→ See [`stimulus.md`](../stimulus.md)

---

## Which CSS layer should I write this in?

> Use this when adding new styles. 37signals uses native CSS cascade layers instead of preprocessors or utility frameworks.

```mermaid
flowchart TD
    A["I need to write CSS"] --> B{"Is it a global reset\nor HTML element default?"}
    B -->|Yes| C["@layer base\nhtml, body, a, h1 defaults"]
    B -->|No| D{"Is it a reusable\nUI component?"}
    D -->|Yes| E["@layer components\n.card, .button, .badge"]
    D -->|No| F{"Is it a one-off\noverride or utility?"}
    F -->|Yes| G["@layer utilities\n.sr-only, .truncate"]
    F -->|No| H{"Is it a color value?"}
    H -->|Yes| I["Use OKLCH\ncolor: oklch(70% 0.2 220)\nnever hex or rgb"]
    H -->|No| J["Scope to a component\nin @layer components"]
```

**Why?** Cascade layers make specificity explicit and predictable. `@layer utilities` always wins over `@layer components`, which always wins over `@layer base` — no specificity hacks needed. OKLCH gives perceptually uniform color manipulation.

→ See [`css.md`](../css.md)
