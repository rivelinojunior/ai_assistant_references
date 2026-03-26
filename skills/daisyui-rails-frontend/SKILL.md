---
name: daisyui-rails-frontend
description: Build production-grade Rails frontend pages and components using DaisyUI 5+, Tailwind CSS 4, Hotwire (Turbo + Stimulus), and ERB. Use this skill whenever creating a new page, adding a UI section, building a component, wiring up Stimulus interactivity, or refining the visual quality of any screen in a Rails app. Requires DESIGN.md and a living /styleguide — both are pre-conditions, not optional. Always forces context7 doc lookups before making DaisyUI, Tailwind, Hotwire, or Stimulus API decisions. Activate for: new page, new view, new partial, new Stimulus controller, new form, dashboard section, navigation, modal, flash message, or any screen that must match the project's design system.
---

# Rails Frontend Design

Build memorable, production-ready Rails interfaces that express the project's design identity — not generic DaisyUI defaults.

**Stack:** DaisyUI 5+ · Tailwind CSS 4 · Hotwire (Turbo Frames + Turbo Streams) · Stimulus · ERB · Rails view helpers

---

## Pre-Requirements (MANDATORY — do not skip)

Before writing a single line of HTML:

1. **Read `DESIGN.md`** — the project's design decisions document. It defines palette, typography, shape, depth, motion rules, and what this product explicitly rejects. All implementation must align with it.

2. **Check the living styleguide** — in development, `/styleguide` renders every token, color swatch, type scale, radius variant, and component state with the actual theme applied. Know what already exists before inventing anything new.

3. **Run context7 lookups** — use context7 (`mcp__context7__resolve-library-id` then `mcp__context7__query-docs`) for the libraries you will actually use before making any API decision. Do not rely on memory for current syntax.

   Priority lookups:
   - `daisyui` — component classes, theme tokens, v5 plugin syntax
   - `tailwindcss` — when using features beyond standard utilities
   - `hotwire-turbo` or `turbo-rails` — Turbo Frame/Stream attributes and helpers
   - `stimulus` / `stimulus-rails` — controller lifecycle, targets, values, outlets

   **Why:** DaisyUI 5, Tailwind 4, and Hotwire APIs change. One wrong class name, attribute, or helper signature ships broken code. Context7 gives you the current source of truth.

---

## Design Intent

Before coding a new page or significant component, resolve:

```text
Intent:    who uses this, what must they do, how should it feel
Palette:   which design system tokens are primary here (primary, accent, base-*)
Surfaces:  base-100 → base-200 → base-300 hierarchy for this screen
Typography: which type scale levels appear (display, heading, body, label, data)
Depth:     this project's single depth strategy (borders / surface shifts / shadows)
Motion:    which motion moments apply (entrance, interaction, state change)
```

If DESIGN.md answers any of these, follow it. Do not reinvent project-level decisions.

---

## Workflow

### 1. Analyze the Design

If given a screenshot, Figma URL, or description, identify:

**Layout structure:**
- Top-level sections (header, sidebar, main, aside, footer)
- Column layout (1-col, 2-col + sidebar, 3-col)
- Fixed vs. scrolling regions
- Container widths and max-width constraints

**Content hierarchy:**
- Primary action — what the user came here to do (must be visually dominant)
- Supporting content — enables or contextualizes the primary action
- Metadata / secondary information — muted, smaller

**Section breakdown:**
```
Page: [NAME]
├── [Layout region]
│   ├── [Section: purpose]
│   └── [Section: purpose]
└── [Layout region]
    └── [Section: purpose]
```

### 2. Map to DaisyUI + Rails Components

For each UI element, decide:

| Visual Element | DaisyUI / Rails Pattern | Notes |
|---|---|---|
| Navigation sidebar | Custom ERB partial | `w-52 bg-base-100 border-r border-base-300` |
| Page header | ERB partial | eyebrow + display heading pattern from styleguide |
| Cards (interactive/selectable) | `card card-border` | only for interactive content |
| Data displays | prose, table, list | never card-wrap data-dense content |
| Primary action | `btn btn-primary` | `active:scale-95 transition-all duration-150` |
| Secondary action | `btn btn-ghost` or `btn btn-outline` | |
| Destructive action | `btn btn-error` | always confirm before execute |
| Form inputs | DaisyUI `input`, `select`, `textarea` | wrapped in `fieldset` + `legend` |
| Flash / alerts | project `.fatfit-flash` or equivalent | left-border colored pattern |
| Loading states | `loading loading-spinner loading-xs` | inside btn or inline |
| Dynamic content | `<turbo-frame>` or Turbo Stream target | |
| JS behavior | Stimulus controller | `data-controller` on root element |

**Always prefer existing project patterns over inventing new ones.** Check the styleguide and existing partials first.

### 3. Define ERB Structure

Plan your partial / layout hierarchy before writing markup:

```
app/views/
├── layouts/
│   └── [layout].html.erb          # page shell, yields
├── [resource]/
│   ├── _header.html.erb            # page header partial
│   ├── _sidebar.html.erb           # if needed
│   ├── index.html.erb              # or show / new / edit
│   └── _[component].html.erb       # isolated reusable chunks
└── shared/
    └── _flash.html.erb             # global patterns
```

Use `render` with locals over instance variables bleeding into partials.

### 4. Build the ERB Markup

Structural skeleton:

```erb
<%# Main page wrapper — mirror the layout's ml-[sidebar-width] if sidebar is fixed %>
<div class="max-w-5xl mx-auto px-12 py-16 space-y-24 animate-fade-up">

  <%# Page header %>
  <header>
    <p class="text-xs text-accent uppercase tracking-widest mb-2">Section label</p>
    <h1 class="text-6xl font-bold text-base-content" style="font-family: var(--font-display)">
      Page Title
    </h1>
    <p class="text-base-content/50 mt-3 max-w-lg">Supporting description.</p>
  </header>

  <%# Content sections with space-y-24 rhythm %>
  <section>
    <h2 class="text-xs text-base-content/40 uppercase tracking-widest mb-8">Section</h2>
    <%# section content %>
  </section>

</div>
```

Section heading pattern (from DESIGN.md): always `text-xs text-base-content/40 uppercase tracking-widest` — labels should not compete with showcased content.

### 5. Wire Hotwire Interactivity

**Turbo Frames** — replace a discrete region without a full page reload:

```erb
<%# In the page %>
<turbo-frame id="meal-options">
  <%= render "meals/options", meals: @meals %>
</turbo-frame>

<%# In the partial — must have matching id %>
<turbo-frame id="meal-options">
  <% meals.each do |meal| %>
    <%= link_to meal.name, meal_path(meal) %>  <%# navigates inside the frame %>
  <% end %>
</turbo-frame>
```

**Turbo Streams** — surgically update the DOM from a controller response:

```ruby
# controller action
respond_to do |format|
  format.turbo_stream do
    render turbo_stream: [
      turbo_stream.replace("meal-summary", partial: "meals/summary", locals: { meal: @meal }),
      turbo_stream.append("flash-container", partial: "shared/flash", locals: { message: "Saved" })
    ]
  end
end
```

```erb
<%# Target element needs matching id %>
<div id="meal-summary">...</div>
```

Check context7 for current Turbo helper signatures before using.

### 6. Add Stimulus Controllers

Stimulus handles progressive enhancement — anything that needs JS behavior but isn't a full page update.

```javascript
// app/javascript/controllers/toggle_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["content"]
  static values = { open: { type: Boolean, default: false } }

  toggle() {
    this.openValue = !this.openValue
  }

  openValueChanged() {
    this.contentTarget.hidden = !this.openValue
  }
}
```

```erb
<div data-controller="toggle">
  <button data-action="click->toggle#toggle" class="btn btn-ghost btn-sm">
    Show details
  </button>
  <div data-toggle-target="content" hidden>
    <%# content %>
  </div>
</div>
```

Rules:
- One controller per behavior concern. Do not pile multiple behaviors into one controller.
- Use `values` for state, `targets` for DOM references, `outlets` for cross-controller communication.
- Prefer Turbo for server-driven interactions. Use Stimulus only for client-side behavior that doesn't need a server round-trip.
- Always check context7 for current Stimulus API (`static targets`, `static values`, lifecycle callbacks).

### 7. Apply Motion

Follow DESIGN.md motion rules — three intentional moments only:

```erb
<%# 1. Entrance — page/section fade up %>
<div class="animate-fade-up">
  ...
</div>

<%# 2. Stagger on lists %>
<% items.each_with_index do |item, i| %>
  <div class="stagger" style="--stagger-index: <%= i %>">
    ...
  </div>
<% end %>
```

```css
/* Interaction feedback — on all interactive elements */
.btn { @apply active:scale-95 transition-all duration-150; }
```

Only animate `transform` and `opacity` — never layout-triggering properties.

### 8. Critique Before Finishing

Run these tests before declaring done:

- **Swap test** — replace every DaisyUI class with a plain description. If the result could be any app, the design isn't specific enough.
- **Squint test** — blur your eyes. Can you still read the hierarchy? Can you find the primary action? Borders, text levels, and surface shifts must hold up blurred.
- **Signature test** — identify one element that could only belong to this product. If you can't name it, add one.
- **Token test** — scan for raw hex values or arbitrary colors. Every color must trace back to a design system token.
- **State test** — hover, focus, active, disabled, loading, empty, error. All interactive elements must respond.
- **Content test** — read every visible string. Does this screen tell one coherent story for a real user?

---

## DaisyUI Rules

- Use semantic theme tokens (`primary`, `secondary`, `accent`, `neutral`, `base-*`, status colors) over raw Tailwind shades.
- `base-100` → `base-200` → `base-300` is the surface elevation hierarchy. Never skip levels.
- Customize theme variables at the CSS plugin level, not per-component.
- Cards are for interactive/selectable content only. Data, plans, lists, and nutritional info use prose, tables, and lists.
- Never ship a page that looks like the DaisyUI docs. If it still resembles stock theme output, keep customizing.
- Re-check context7 whenever a component API, modifier, or configuration option is uncertain.

---

## Design Constraints (from DESIGN.md philosophy)

- No raw Inter/Roboto/system-ui — the project chose specific typefaces for a reason.
- No purple-on-white gradients or electric-color accents — unless DESIGN.md explicitly includes them.
- No card mosaics as default layout — reserve cards for interactive content.
- No shadows unless DESIGN.md permits them — default is flat surfaces differentiated by base-* levels.
- One accent color. Do not introduce a second accent.
- No competing cool-toned UI against a warm palette (or vice versa).
- Every interactive state must be styled: hover, focus (visible ring), active (scale-95), disabled, loading.

---

## Rails-Specific Patterns

**Forms:**
```erb
<%= form_with model: @resource, class: "space-y-6" do |f| %>
  <fieldset>
    <legend class="text-sm font-medium text-base-content mb-4">Section</legend>

    <div class="flex flex-col gap-1">
      <%= f.label :name, class: "text-sm text-base-content/70" %>
      <%= f.text_field :name, class: "input input-bordered w-full" %>
      <% if @resource.errors[:name].any? %>
        <p class="fieldset-label text-error text-xs"><%= @resource.errors[:name].first %></p>
      <% end %>
    </div>
  </fieldset>

  <%= f.submit "Save", class: "btn btn-primary active:scale-95 transition-all duration-150" %>
<% end %>
```

**View helpers** — use Rails helpers semantically:
- `link_to` with block for complex anchor content
- `button_to` for non-GET actions (generates a form)
- `content_tag` / `tag` for dynamic element generation
- `render` with `locals:` always — avoid relying on instance variable leakage

**Partials** — keep them focused:
- One concern per partial
- Accept explicit locals, document them with a comment at the top
- Avoid database queries inside partials (pass data from the controller/presenter)

---

## Reference Map

- Read [references/principles.md](references/principles.md) — token architecture, spacing, depth, typography hierarchy, craft floor.
- Read [references/daisyui-v5.md](references/daisyui-v5.md) — DaisyUI 5+ theme token names, CSS plugin syntax, component output control.
- Read [references/critique.md](references/critique.md) — before finalizing any design-heavy build.
- Read [references/example.md](references/example.md) — subtle layering decisions and composition thinking.
- Read [references/validation.md](references/validation.md) — when deciding whether to record patterns in the design system.

---

## Output Standard

Every significant UI build must resolve these before implementation:

```text
Intent:     who + task + feeling
Palette:    primary tokens used on this screen
Surfaces:   which base-* levels and where
Typography: type scale levels present
Depth:      single strategy name (border / surface-shift / shadow)
Motion:     which moments apply (entrance / stagger / state-change)
Hotwire:    which frames/streams are needed + why
Stimulus:   which controllers, what behavior they handle
```

If any field is "I'm not sure," stop, read DESIGN.md, check the styleguide, run context7. Then fill it in.
