# Core Rails Decision Guide

Flowcharts for the most common architecture decisions in Rails models, controllers, and views.

---

## Where should this logic live?

> Use this when you've written code in a controller action and it feels like it doesn't belong there.

```mermaid
flowchart TD
    A["I need to put some logic somewhere"] --> B{"Is it only about routing,\nparams, or rendering?"}
    B -->|Yes| C["Controller action\nkeep it thin"]
    B -->|No| D{"Does it need request-scoped\ndata: current user, session?"}
    D -->|Yes| E["Current context\nCurrent.user / Current.account"]
    D -->|No| F{"Is it about display\nor formatting?"}
    F -->|Yes| G["View helper or PORO decorator\ne.g. CardPresenter"]
    F -->|No| H{"Is it shared across\n2 or more models?"}
    H -->|Yes| I["Model concern\nmodule Archivable / Notifiable"]
    H -->|No| J{"Is it a complex\nmulti-step operation?"}
    J -->|Yes| K["PORO called from the model\nnot from the controller"]
    J -->|No| L["Model method\non the owning model"]
```

**Why?** Controllers should only route, authorize, and render — all domain logic belongs in the model layer. Moving logic down makes it testable in isolation and reusable without a request context.

→ See [`controllers.md`](../controllers.md) and [`models.md`](../models.md)

---

## Should this be a boolean column or a record?

> Use this when you're designing a feature that tracks whether something is active, archived, published, etc.

```mermaid
flowchart TD
    A["I need to track state on a model"] --> B{"Do you need to know\nwhen it changed?"}
    B -->|Yes| C["Record\ne.g. Activation, Publication, Archive"]
    B -->|No| D{"Could it ever have\nmetadata attached?"}
    D -->|Yes| C
    D -->|No| E{"Might you ever need\na history of changes?"}
    E -->|Yes| C
    E -->|No| F{"Is it truly just on/off\nwith no audit need?"}
    F -->|Yes| G["Boolean column\nbut think twice — records are flexible"]
    F -->|No| C
```

**Why?** Boolean columns lose history and can't hold metadata. A `Publication` record knows *when* something was published and *by whom*. 37signals defaults to records because requirements always expand.

→ See [`models.md`](../models.md) and [`database.md`](../database.md)

---

## Should I use a model concern or a PORO?

> Use this when a model is getting large and you want to extract logic.

```mermaid
flowchart TD
    A["I want to extract logic\nfrom a model"] --> B{"Is this behavior shared\nacross 2 or more models?"}
    B -->|Yes| C["Model concern\nmodule Archivable\n  included in Card, List\nend"]
    B -->|No| D{"Is it presentation or\ndisplay logic only?"}
    D -->|Yes| E["PORO decorator\nclass CardPresenter\n  def formatted_due_date\nend"]
    D -->|No| F{"Is it a multi-step\noperation with side effects?"}
    F -->|Yes| G["PORO with a call method\ncalled from the model\nnot the controller"]
    F -->|No| H["Model method\nkeep it on the model"]
```

**Why?** Concerns handle horizontal sharing; POROs handle complex vertical operations. Putting complex logic in a PORO and calling it *from the model* keeps controllers thin while avoiding the god-object problem.

→ See [`models.md`](../models.md)

---

## How should I authorize this action?

> Use this when you need to check whether a user can perform an operation.

```mermaid
flowchart TD
    A["I need to authorize an action"] --> B{"Thinking of adding\nPundit or CanCanCan?"}
    B -->|Yes| C["Stop — avoid authorization gems\n37signals does not use them"]
    C --> D
    B -->|No| D{"Is the policy simple?\ne.g. owner or admin only"}
    D -->|Yes| E["Model method\ncard.editable_by?(current_user)"]
    D -->|No| F{"Is the policy complex\nwith multiple conditions?"}
    F -->|Yes| G["PORO policy object\nclass CardPolicy\n  def can_edit?\nend\ncalled from model"]
    F -->|No| E
```

**Why?** Authorization logic lives closest to the data it protects — the model. Gems like Pundit add indirection without adding value. A simple `can_edit?(user)` method is readable, testable, and sufficient.

→ See [`controllers.md`](../controllers.md)
