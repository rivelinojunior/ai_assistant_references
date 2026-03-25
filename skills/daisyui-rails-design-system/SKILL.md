---
name: daisyui-rails-design-system
description: >
  Sets up a complete design system foundation using DaisyUI 5.5+ and Tailwind CSS 4
  in a Rails project. Use this skill whenever the user wants to establish a design system,
  style guide, or UI token system in a Rails app with DaisyUI — even if they just say
  "set up DaisyUI", "create a styleguide", "configure our design tokens", or "add theming
  to our Rails app". Covers environment discovery, aesthetic direction, design token
  extraction from screenshots, DaisyUI custom theme configuration, a dev-only living
  styleguide with accessibility and motion patterns, and a DESIGN.md decision record.
---

# DaisyUI 5.5+ Design System for Rails

Before writing a single line of code, explore the codebase and interview the user to resolve
every unknown. Work through each question one by one, providing a recommended answer for each.
If a question can be answered by reading the project files, read them instead of asking.

---

## Phase 1 — Explore & Interview

### 1a. Auto-discover from the codebase

Explore these files silently before asking anything:

| What to check | Files to read |
|---|---|
| Tailwind version & setup method | `package.json`, `Gemfile`, `Gemfile.lock` |
| DaisyUI already installed? | `package.json`, CSS entry files |
| CSS entry point | `app/assets/stylesheets/`, `app/javascript/`, `app/frontend/` |
| JS bundler | `Gemfile` (look for `cssbundling-rails`, `jsbundling-rails`, `vite_rails`), `package.json` scripts |
| Layout file | `app/views/layouts/application.html.erb` |
| Existing theme/brand hints | any existing CSS, `tailwind.config.js` if present |
| Routes file structure | `config/routes.rb` |

After exploring, resolve each of the following questions — if the codebase answers it, note
what you found; if it doesn't, ask the user with your recommended answer clearly marked.

### 1b. Questions to resolve (ask all in one message)

**Technical setup:**

1. **CSS pipeline** — How is Tailwind served?
   - `tailwindcss-rails` gem (standalone CLI, no Node build step)
   - `cssbundling-rails` + esbuild/bun
   - `vite_rails` (Vite + `@tailwindcss/vite`)
   - _Recommended: surface what you found in the Gemfile_

2. **Tailwind version** — v3 or v4?
   - DaisyUI 5 requires Tailwind v4. If v3 is detected, surface this blocker early.
   - _Recommended: upgrade to Tailwind v4 + DaisyUI v5 together_

3. **DaisyUI already installed?** — Check `package.json`.
   - If yes: which version? DaisyUI v5 has breaking changes from v4 — confirm upgrade is OK.
   - If no: confirm it's safe to install.

4. **Main CSS entry file** — Where should `@plugin "daisyui"` be added?
   - Common: `app/assets/stylesheets/application.tailwind.css`, `app/javascript/application.css`,
     `app/frontend/entrypoints/application.css`
   - _Recommended: surface the file you found_

5. **Theme name** — What should the custom theme be called?
   - _Recommended: use the Rails app name in snake_case_

6. **Design reference** — Is there a screenshot, Figma link, or existing brand color?
   - If yes → extract tokens and aesthetic direction from it in Phase 3
   - If no → ask for the primary brand color (hex is fine) and proceed to the aesthetic interview

7. **Dark mode** — Should the styleguide support a dark mode toggle?
   - _Recommended: yes, using DaisyUI's built-in `dark` theme_

**Aesthetic direction** (always ask these — don't infer):

8. **Aesthetic thesis** — Before picking any colors or fonts, commit to a direction.
   Ask the user to answer in one sentence: _"What should this UI feel like?"_
   If they're unsure, offer a set of poles to react to:
   - Calm / minimal vs. bold / expressive
   - Editorial / typographic vs. product / functional
   - Warm / organic vs. cool / technical
   - Refined / luxury vs. playful / approachable
   - _Recommended: pick a direction that contrasts with generic SaaS (avoid "clean and modern")_

9. **Typography** — What font pairing captures the aesthetic?
   - Two typefaces maximum: one for display/headings, one for body
   - _Recommended: pick a distinctive display font that matches the aesthetic thesis_
   - Avoid generic defaults: **Inter, Roboto, Arial, and system fonts are not acceptable choices** —
     they produce a generic AI aesthetic with no personality
   - Suggestions by tone:
     - Editorial/luxury → Playfair Display + DM Sans, or Fraunces + Outfit
     - Bold/expressive → Syne + Plus Jakarta Sans, or Cabinet Grotesk + Figtree
     - Technical/precise → Space Mono + IBM Plex Sans, or Fira Code + Source Sans 3
     - Warm/organic → Instrument Serif + Nunito, or Lora + Mulish

10. **Cards** — Does the UI actually need card grids as a primary layout pattern?
    - Cards should only be used when the content is genuinely interactive or selectable
    - _Recommended: default to clean list/table or prose layouts; add cards only where justified_

Do not proceed to Phase 2 until all 10 questions are answered.

---

## Phase 2 — Install & Configure

**Only run install steps for what is actually missing.** Show the user what you found first,
then confirm the action before running it.

### 2a. Verify / install Tailwind v4

```bash
# tailwindcss-rails (gem-based):
bundle add tailwindcss-rails   # if not present
bin/rails tailwindcss:install  # scaffolds the CSS entry file

# cssbundling-rails / vite_rails (npm-based):
npm install -D tailwindcss@latest   # only if not v4
```

### 2b. Verify / install DaisyUI 5.5+

```bash
# Check current version first:
cat package.json | grep daisyui

# Install or upgrade:
npm install -D daisyui@latest
```

Confirm `"daisyui": "^5.x.x"` appears in `package.json` before continuing.

### 2c. Wire up DaisyUI in the CSS entry file

Edit the CSS entry file identified in Phase 1. Add right after `@import "tailwindcss"`:

```css
@import "tailwindcss";
@plugin "daisyui" {
  themes: [theme-name] --default, dark --prefersdark;
}
```

### 2d. Rebuild the CSS

After any change to the CSS entry file, the compiled output in `app/assets/builds/` will be
stale until rebuilt. The browser will serve the old theme until this runs.

```bash
# tailwindcss-rails:
bin/rails tailwindcss:build

# cssbundling-rails / vite_rails:
yarn build:css   # or the equivalent script in package.json
```

If the dev server is already running with `bin/dev`, the watcher should rebuild automatically —
but if styles look wrong after editing the CSS file, run the build command manually.

---

## Phase 3 — Aesthetic Direction & Theme

### 3a. Translate the aesthetic thesis into design decisions

Before touching OKLCH values, resolve these from the Phase 1 interview:

| Aesthetic pole | Color implication | Radius implication | Depth implication |
|---|---|---|---|
| Calm / minimal | Low chroma, near-neutral palette | Sharp or barely-rounded | `--depth: 0`, flat |
| Bold / expressive | High chroma primary, sharp accent | Rounded or pill | `--depth: 1`, layered |
| Editorial | Monochrome + single ink accent | Sharp corners | Flat |
| Warm / organic | Warm neutrals (hue 30–60), earthy primary | Generous rounded | Subtle depth |
| Technical | Cool neutrals (hue 220–260), precise accent | Sharp or minimal | Flat or mono |
| Luxury | Deep base surfaces, muted primary, metallic accent | Refined rounded | Depth with texture (`--noise: 1`) |

**Color philosophy:** Pick one dominant color and one sharp accent. Distribute the palette
with intention — a timid, evenly-distributed palette reads as uncommitted. The accent should
surprise slightly against the dominant color.

### 3b. Extract tokens from the design reference

Identify from the screenshot/Figma/hex (and the aesthetic decisions above):

- **Primary**: the dominant brand color — full commitment, not muted
- **Accent**: the sharp surprise — should stand out against primary
- **Base surfaces**: derived from the aesthetic pole (warm/cool/neutral, matching the thesis hue)
- **Border radius**: matched to the aesthetic direction (see table above)
- **Depth**: flat or layered per the table

If only a hex was given, derive the full palette:
- Shift lightness along the same hue for base-100/200/300
- Reduce chroma toward 0 for neutrals (keep the hue, drop the saturation)
- Semantic colors can retain standard hues but adjust lightness to match the base tone

### 3c. Add the theme block to the CSS entry file

```css
@plugin "daisyui/theme" {
  name: "[theme-name]";
  default: true;

  /* ── PRIMARY (dominant color — commit fully) ── */
  --color-primary:         oklch([L]% [C] [H]);
  --color-primary-content: oklch([L]% [C] [H]);  /* ≥ 4.5:1 contrast */

  /* ── SECONDARY ── */
  --color-secondary:         oklch([L]% [C] [H]);
  --color-secondary-content: oklch([L]% [C] [H]);

  /* ── ACCENT (the sharp surprise) ── */
  --color-accent:         oklch([L]% [C] [H]);
  --color-accent-content: oklch([L]% [C] [H]);

  /* ── NEUTRAL ── */
  --color-neutral:         oklch([L]% [C] [H]);
  --color-neutral-content: oklch([L]% [C] [H]);

  /* ── BASE SURFACES (derived from aesthetic thesis hue) ── */
  --color-base-100:     oklch([L]% [C] [H]);  /* page background */
  --color-base-200:     oklch([L]% [C] [H]);  /* cards, sidebar   */
  --color-base-300:     oklch([L]% [C] [H]);  /* borders, dividers */
  --color-base-content: oklch([L]% [C] [H]);  /* body text — never pure black (oklch(0% 0 0));
                                                  use off-black e.g. oklch(15% 0.02 [hue]) */

  /* ── SEMANTIC ── */
  --color-info:            oklch(70% 0.18 220);
  --color-info-content:    oklch(98% 0.01 220);
  --color-success:         oklch(65% 0.22 145);
  --color-success-content: oklch(98% 0.01 145);
  --color-warning:         oklch(75% 0.22 60);
  --color-warning-content: oklch(20% 0.05 60);
  --color-error:           oklch(60% 0.26 25);
  --color-error-content:   oklch(98% 0.01 25);

  /* ── SHAPE & DEPTH (matched to aesthetic thesis) ── */
  --radius-box:   [e.g. 0.5rem];
  --radius-field: [e.g. 0.375rem];
  --radius-badge: [e.g. 1rem];
  --border:  1px;
  --depth:   [0 or 1];
  --noise:   [0 or 1];
}
```

**OKLCH quick reference:**
`oklch(L% C H)` — L = lightness 0–100%, C = chroma 0–0.4, H = hue 0–360.
White ≈ `oklch(100% 0 0)`, saturated blue ≈ `oklch(55% 0.28 250)`, warm green ≈ `oklch(65% 0.25 145)`.
Use [daisyui.com/theme-generator](https://daisyui.com/theme-generator/) for visual tuning.

### 3d. Typography — install the chosen font pair

Add to `app/views/layouts/application.html.erb` inside `<head>`:

```erb
<%# Display font (headings) + body font — replace with chosen pair %>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=[DisplayFont]:wght@600;700;800&family=[BodyFont]:wght@400;500;600&display=swap" rel="stylesheet">
```

Append to the CSS entry file:

```css
/* Typography — use the two-font pairing from the aesthetic thesis */
:root {
  --font-display: '[DisplayFont]', serif;
  --font-body:    '[BodyFont]', sans-serif;
}

body        { font-family: var(--font-body); }
h1, h2, h3  { font-family: var(--font-display); }
```

**Animation performance rules (apply to all motion, regardless of aesthetic):**
- Only animate `transform` and `opacity` — never layout-triggering properties (`top`, `left`, `width`, `height`, `margin`)
- Use `will-change: transform` sparingly — only on elements actively animating, remove it after
- Prefer `transition-*` utilities over `animate-*` for hover/interactive states; reserve `@keyframes` for entrance sequences

**Typography composition rules:**
- Scale gap between heading levels should be generous — each step should feel clearly larger
- Body line-height: `1.6–1.75` for readability
- Long-form text: max-width `65ch` to prevent overly long lines
- Two typefaces maximum — a third is almost never justified

### 3e. DaisyUI v5 rules

- Always use semantic names (`bg-primary`, `text-base-content`) — **never** hardcoded Tailwind colors for brand elements
- Inputs have borders by default — use `*-ghost` variant to remove (opposite of v4)
- `card-bordered` → `card-border` | `tabs-boxed` → `tabs-box` | `tabs-lifted` → `tabs-lift`
- Dark mode is automatic via `data-theme` — do **not** write a `.dark {}` block

### 3f. Motion — define 2–3 intentional interactions

Good interfaces have a few well-considered moments of motion, not scattered micro-animations.
Define at minimum:

1. **Entrance** — how does the page/content appear? (`animate-fade-in`, staggered Tailwind delays)
2. **Interaction feedback** — buttons, links, hover states (`transition-all duration-150`)
3. **State change** — loading indicators, alert appearances, form submission feedback

Use Tailwind's `transition-*` and `animate-*` utilities. For Turbo (Hotwire) page transitions,
consider a simple CSS `@keyframes` fade on `[data-turbo-body]` or use the Turbo `data-turbo-transition` attribute.

Add to the CSS entry file:

```css
/* Entrance animation for page content */
@keyframes fade-up {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

.animate-fade-up {
  animation: fade-up 0.3s ease-out both;
}

/* Stagger utility for list items — set --stagger-index via inline style on each child */
/* e.g. <div style="--stagger-index:0">, <div style="--stagger-index:1">, etc.        */
/* Works for any list length, no nth-child hardcoding needed.                          */
.stagger > * {
  animation: fade-up 0.25s ease-out both;
  animation-delay: calc(var(--stagger-index, 0) * 50ms);
}
```

---

## Phase 4 — Dev-Only Styleguide

The styleguide must not be reachable in production. Gate it at both the routing and
controller layers for defence in depth.

### 4a. Routes — development only

```ruby
# config/routes.rb
if Rails.env.development?
  namespace :styleguide do
    root "styleguide#index"
    # Component pages added by future prompts, e.g.:
    # get "components/button", to: "components#button"
  end
end
```

### 4b. Controller — secondary guard

```ruby
# app/controllers/styleguide/styleguide_controller.rb
module Styleguide
  class StyleguideController < ApplicationController
    layout "styleguide"
    before_action :development_only!

    def index
    end

    private

    def development_only!
      raise ActionController::RoutingError, "Not found" unless Rails.env.development?
    end
  end
end
```

### 4c. Navigation helper

```ruby
# app/helpers/styleguide_helper.rb
module StyleguideHelper
  NAVIGATION = [
    {
      title: "Foundation",
      items: [{ name: "Design Tokens", path: :styleguide_root_path }]
    },
    {
      title: "Components",
      items: []  # populated by future prompts
    }
  ].freeze
end
```

### 4d. Styleguide layout

The layout itself should demonstrate the aesthetic direction — the font pair, the base surface
colors, and the spacing rhythm all visible at a glance.

The sidebar is intentionally minimal: no background fill, just a `border-r`. The active nav item
uses `text-primary font-medium` rather than a filled pill — lighter visual weight that doesn't
compete with the content.

```erb
<%# app/views/layouts/styleguide.html.erb %>
<!DOCTYPE html>
<html lang="en" data-theme="[theme-name]">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>[AppName] Design System</title>

    <%# Font preconnect — match the pair chosen in Phase 3 %>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

    <%= stylesheet_link_tag "tailwind", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>
  <body class="bg-base-100 text-base-content">
    <div class="flex min-h-screen">

      <%# Minimal sidebar — no bg fill, border-r only %>
      <aside class="w-52 bg-base-100 border-r border-base-300 px-5 py-8 flex flex-col gap-8 fixed top-0 left-0 h-screen overflow-y-auto"
             role="navigation" aria-label="Design system sections">
        <div>
          <%= link_to styleguide_root_path,
                class: "block transition-opacity duration-150 hover:opacity-70" do %>
            <span class="block text-xs text-base-content/40 uppercase tracking-widest mb-1">[AppName]</span>
            <span class="block font-semibold text-base-content" style="font-family: var(--font-display); font-size: 1rem;">Design System</span>
          <% end %>
        </div>

        <nav class="flex flex-col gap-5">
          <% StyleguideHelper::NAVIGATION.each do |section| %>
            <% next if section[:items].empty? %>
            <div>
              <p class="text-xs text-base-content/40 uppercase tracking-widest mb-2">
                <%= section[:title] %>
              </p>
              <ul class="flex flex-col gap-0.5">
                <% section[:items].each do |item| %>
                  <li>
                    <%= link_to item[:name], send(item[:path]),
                          class: "block text-sm px-2 py-1.5 rounded-lg transition-colors duration-150 #{current_page?(send(item[:path])) ? 'text-primary font-medium' : 'text-base-content/60 hover:text-base-content hover:bg-base-200'}" %>
                  </li>
                <% end %>
              </ul>
            </div>
          <% end %>
        </nav>

        <div class="mt-auto">
          <label class="flex items-center gap-2 cursor-pointer" aria-label="Toggle dark mode">
            <input type="checkbox" class="toggle toggle-xs toggle-primary"
              onchange="document.documentElement.setAttribute('data-theme', this.checked ? 'dark' : '[theme-name]')" />
            <span class="text-xs text-base-content/50">Dark mode</span>
          </label>
        </div>
      </aside>

      <main class="flex-1 ml-52 overflow-auto animate-fade-up">
        <%= yield %>
      </main>

    </div>
  </body>
</html>
```

### 4e. Styleguide index page

The page should feel like it was designed, not generated. Key design principles in this template:

- **Section headings** use `text-xs uppercase tracking-widest text-base-content/40` — editorial,
  not structural h2s competing with the content being showcased
- **Color swatches** use `gap-px bg-base-300 rounded-2xl overflow-hidden` — the grid gap *becomes*
  the separator, no inner card borders needed
- **Component containers** use `border border-base-300 rounded-2xl` — lighter than `bg-base-200`,
  doesn't visually compete with the components inside
- **Typography specimen** uses a ghosted `Aa` backdrop (`text-base-content/8`) + a table-format
  type scale — shows the scale in context, not as isolated text blobs

Replace all `[...]` placeholders with the project's actual values from Phase 3.

```erb
<%# app/views/styleguide/styleguide/index.html.erb %>
<div class="max-w-5xl mx-auto px-12 py-16 space-y-24">

  <%# ── HEADER — eyebrow + display heading + brand voice subtitle ── %>
  <header class="max-w-lg">
    <p class="text-xs text-accent uppercase tracking-widest mb-3">Design System</p>
    <h1 class="text-6xl font-bold leading-none mb-4" style="font-family: var(--font-display)">
      [AppName]
    </h1>
    <p class="text-base-content/60 text-base leading-relaxed">
      Foundation tokens, type scale, and component patterns.
      [One sentence capturing the aesthetic thesis from Phase 1.]
    </p>
  </header>

  <%# ─────────────────────── COLOR PALETTE ─────────────────────────── %>
  <section aria-labelledby="colors-heading">
    <h2 id="colors-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Color Palette
    </h2>

    <%# Brand — gap-px grid: the gap IS the border between swatches %>
    <div class="grid grid-cols-4 gap-px bg-base-300 rounded-2xl overflow-hidden mb-3">
      <%# Replace with your theme's descriptive names, hex values, OKLCH, and role descriptions %>
      <% [
        ["primary",   "[Name]", "#[hex]", "oklch([L]% [C] [H])", "primary-content",   "[role]"],
        ["secondary", "[Name]", "#[hex]", "oklch([L]% [C] [H])", "secondary-content", "[role]"],
        ["accent",    "[Name]", "#[hex]", "oklch([L]% [C] [H])", "accent-content",    "[role]"],
        ["neutral",   "[Name]", "#[hex]", "oklch([L]% [C] [H])", "neutral-content",   "[role]"]
      ].each do |bg, name, hex, oklch, fg, desc| %>
        <div class="bg-base-100">
          <div class="h-36 bg-<%= bg %> flex items-end p-4">
            <span class="text-<%= fg %>/60 text-2xl font-bold" style="font-family: var(--font-display)">Aa</span>
          </div>
          <div class="px-4 py-3">
            <p class="text-sm font-semibold"><%= name %></p>
            <p class="text-xs text-base-content/40 mt-0.5 font-mono"><%= hex %></p>
            <p class="text-xs text-base-content/30 mt-0.5"><%= desc %></p>
          </div>
        </div>
      <% end %>
    </div>

    <%# Base Surfaces %>
    <div class="grid grid-cols-3 gap-px bg-base-300 rounded-2xl overflow-hidden mb-3">
      <% [
        ["base-100", "[Name]", "#[hex]", "Page background"],
        ["base-200", "[Name]", "#[hex]", "Cards, sidebar, panels"],
        ["base-300", "[Name]", "#[hex]", "Borders, dividers"]
      ].each do |bg, name, hex, desc| %>
        <div class="bg-base-100">
          <div class="h-20 bg-<%= bg %> border-b border-base-300"></div>
          <div class="px-4 py-3">
            <p class="text-sm font-semibold"><%= name %></p>
            <p class="text-xs text-base-content/40 mt-0.5 font-mono"><%= hex %></p>
            <p class="text-xs text-base-content/30 mt-0.5"><%= desc %></p>
          </div>
        </div>
      <% end %>
    </div>

    <%# Semantic %>
    <div class="grid grid-cols-4 gap-px bg-base-300 rounded-2xl overflow-hidden">
      <% [
        ["info",    "Info",    "Informational"],
        ["success", "Success", "Confirmation / on track"],
        ["warning", "Warning", "Caution / needs attention"],
        ["error",   "Error",   "Destructive / critical"]
      ].each do |bg, name, desc| %>
        <div class="bg-base-100">
          <div class="h-16 bg-<%= bg %>"></div>
          <div class="px-4 py-3">
            <p class="text-sm font-semibold"><%= name %></p>
            <p class="text-xs text-base-content/30 mt-0.5"><%= desc %></p>
          </div>
        </div>
      <% end %>
    </div>
  </section>

  <%# ─────────────────────── TYPOGRAPHY ────────────────────────────── %>
  <section aria-labelledby="type-heading">
    <h2 id="type-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Typography
    </h2>

    <%# Two-column specimen — ghosted Aa backdrop shows the font's character %>
    <div class="grid grid-cols-2 gap-12 mb-12">
      <div>
        <p class="text-xs text-base-content/40 mb-4">Display — [DisplayFont]</p>
        <div class="relative">
          <span class="block text-[9rem] leading-none font-bold text-base-content/8 select-none absolute -top-4 left-0"
                style="font-family: var(--font-display)">Aa</span>
          <p class="relative text-5xl font-bold leading-tight pt-12" style="font-family: var(--font-display)">[DisplayFont]</p>
          <p class="text-base text-base-content/60 mt-2 leading-relaxed" style="font-family: var(--font-display)">
            [One sentence describing the display font's character and when to use it.]
          </p>
        </div>
      </div>
      <div>
        <p class="text-xs text-base-content/40 mb-4">Body / UI — [BodyFont]</p>
        <div class="relative">
          <span class="block text-[9rem] leading-none font-bold text-base-content/8 select-none absolute -top-4 left-0">Aa</span>
          <p class="relative text-5xl font-bold leading-tight pt-12">[BodyFont]</p>
          <p class="text-base text-base-content/60 mt-2 leading-relaxed">
            [One sentence describing the body font's character and why it pairs with the display font.]
          </p>
        </div>
      </div>
    </div>

    <%# Type scale as a clean bordered table %>
    <div class="border border-base-300 rounded-2xl overflow-hidden">
      <% [
        ["text-5xl",  "font-bold",    "800", "3rem",    "[Heading example — use a real phrase from the app]",     true],
        ["text-3xl",  "font-semibold","700", "1.875rem","[Section heading example]",                              true],
        ["text-xl",   "font-medium",  "600", "1.25rem", "[Subheading example]",                                   true],
        ["text-base", "font-normal",  "400", "1rem",    "[Body text — a sentence from actual product copy.]",     false],
        ["text-sm",   "font-normal",  "400", "0.875rem","[Supporting text — labels, secondary info.]",            false],
        ["text-xs",   "font-normal",  "400", "0.75rem", "[Extra small — timestamps, captions, footnotes.]",       false],
      ].each_with_index do |(size, weight, w_num, px, example, is_display), i| %>
        <div class="flex items-baseline gap-8 px-6 py-4 <%= i < 5 ? 'border-b border-base-300' : '' %>">
          <div class="w-20 shrink-0">
            <p class="text-xs font-mono text-base-content/40"><%= px %></p>
            <p class="text-xs text-base-content/30"><%= w_num %></p>
          </div>
          <p class="<%= size %> <%= weight %> leading-tight"
             <%= is_display ? 'style="font-family: var(--font-display)"' : '' %>>
            <%= example %>
          </p>
        </div>
      <% end %>
    </div>
  </section>

  <%# ─────────────────────── SHAPE ─────────────────────────────────── %>
  <section aria-labelledby="radius-heading">
    <h2 id="radius-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Border Radius
    </h2>
    <%# Varying sizes show the scale progression more clearly than uniform boxes %>
    <div class="flex flex-wrap items-end gap-8">
      <% [
        ["rounded-none",  "none",  "w-12 h-12"],
        ["rounded-sm",    "sm",    "w-12 h-12"],
        ["rounded",       "base",  "w-12 h-12"],
        ["rounded-lg",    "lg",    "w-14 h-14"],
        ["rounded-xl",    "xl",    "w-14 h-14"],
        ["rounded-2xl",   "2xl",   "w-16 h-16"],
        ["rounded-3xl",   "3xl",   "w-16 h-16"],
        ["rounded-full",  "full",  "w-16 h-16"],
      ].each do |cls, label, size| %>
        <div class="text-center">
          <div class="<%= size %> bg-primary mx-auto <%= cls %>"></div>
          <p class="text-xs text-base-content/40 mt-2 font-mono"><%= label %></p>
        </div>
      <% end %>
    </div>
  </section>

  <%# ─────────────────────── MOTION ────────────────────────────────── %>
  <section aria-labelledby="motion-heading">
    <h2 id="motion-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Motion
    </h2>
    <div class="grid grid-cols-3 gap-4 mb-4">
      <div class="border border-base-300 rounded-2xl p-6 text-center">
        <p class="text-xs text-base-content/40 mb-4 uppercase tracking-wider">Entrance</p>
        <div class="animate-fade-up">
          <div class="h-10 bg-primary rounded-xl"></div>
        </div>
        <p class="text-xs text-base-content/40 mt-4 font-mono">animate-fade-up · 0.3s</p>
      </div>
      <div class="border border-base-300 rounded-2xl p-6 text-center">
        <p class="text-xs text-base-content/40 mb-4 uppercase tracking-wider">Interaction</p>
        <button class="btn btn-primary w-full transition-all duration-150 active:scale-95">
          Press me
        </button>
        <p class="text-xs text-base-content/40 mt-4 font-mono">active:scale-95 · 150ms</p>
      </div>
      <div class="border border-base-300 rounded-2xl p-6 text-center">
        <p class="text-xs text-base-content/40 mb-4 uppercase tracking-wider">State change</p>
        <div class="flex justify-center gap-4">
          <span class="loading loading-spinner loading-sm text-primary"></span>
          <span class="loading loading-dots loading-sm text-secondary"></span>
          <span class="loading loading-ring loading-sm text-accent"></span>
        </div>
        <p class="text-xs text-base-content/40 mt-4 font-mono">loading · async</p>
      </div>
    </div>
    <div class="border border-base-300 rounded-2xl p-6">
      <p class="text-xs text-base-content/40 mb-4 uppercase tracking-wider">Stagger entrance</p>
      <div class="stagger flex flex-wrap gap-2">
        <%# Replace with domain-relevant labels from the app %>
        <% ["Label A", "Label B", "Label C", "Label D", "Label E", "Label F"].each_with_index do |item, i| %>
          <span class="badge badge-primary" style="--stagger-index:<%= i %>"><%= item %></span>
        <% end %>
      </div>
      <p class="text-xs text-base-content/30 mt-3 font-mono">.stagger + --stagger-index · 50ms steps</p>
    </div>
  </section>

  <%# ─────────────────────── BUTTONS ───────────────────────────────── %>
  <section aria-labelledby="buttons-heading">
    <h2 id="buttons-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Buttons
    </h2>
    <div class="border border-base-300 rounded-2xl p-8 space-y-6">
      <div>
        <p class="text-xs text-base-content/40 mb-3">Variants</p>
        <div class="flex flex-wrap gap-3">
          <button class="btn btn-primary transition-all duration-150 active:scale-95">Primary</button>
          <button class="btn btn-secondary transition-all duration-150 active:scale-95">Secondary</button>
          <button class="btn btn-accent transition-all duration-150 active:scale-95">Accent</button>
          <button class="btn btn-neutral transition-all duration-150 active:scale-95">Neutral</button>
          <button class="btn btn-ghost transition-all duration-150">Ghost</button>
          <button class="btn btn-outline btn-primary transition-all duration-150">Outline</button>
          <button class="btn btn-error transition-all duration-150 active:scale-95">Destructive</button>
        </div>
      </div>
      <div class="border-t border-base-300 pt-6">
        <p class="text-xs text-base-content/40 mb-3">Sizes &amp; States</p>
        <div class="flex flex-wrap gap-3 items-center">
          <button class="btn btn-primary btn-sm active:scale-95">Small</button>
          <button class="btn btn-primary active:scale-95">Default</button>
          <button class="btn btn-primary btn-lg active:scale-95">Large</button>
          <button class="btn btn-primary" disabled aria-disabled="true">Disabled</button>
          <button class="btn btn-primary" aria-busy="true">
            <span class="loading loading-spinner loading-xs" aria-hidden="true"></span>
            Loading
          </button>
        </div>
      </div>
    </div>
  </section>

  <%# ─────────────────────── CARDS ─────────────────────────────────── %>
  <section aria-labelledby="cards-heading">
    <h2 id="cards-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Cards
    </h2>
    <%# Cards use bg-base-200 fill — the border wrapper is handled by section, not the card itself %>
    <div class="grid grid-cols-2 gap-4">
      <div class="card bg-base-200">
        <div class="card-body">
          <h3 class="card-title" style="font-family: var(--font-display)">[Card title]</h3>
          <p class="text-base-content/60 text-sm">Use when content is interactive or selectable — not as a default container.</p>
          <div class="card-actions justify-end mt-2">
            <button class="btn btn-primary btn-sm active:scale-95">Action</button>
          </div>
        </div>
      </div>
      <div class="card bg-base-200 card-border">
        <div class="card-body">
          <h3 class="card-title" style="font-family: var(--font-display)">[Confirm action?]</h3>
          <p class="text-base-content/60 text-sm">
            <code class="kbd kbd-sm">card-border</code> for explicit boundary when needed.
          </p>
          <div class="card-actions justify-end mt-2">
            <button class="btn btn-outline btn-sm">Cancel</button>
            <button class="btn btn-primary btn-sm active:scale-95">Confirm</button>
          </div>
        </div>
      </div>
    </div>
  </section>

  <%# ─────────────────────── BADGES ────────────────────────────────── %>
  <section aria-labelledby="badges-heading">
    <h2 id="badges-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Badges
    </h2>
    <div class="border border-base-300 rounded-2xl p-8 flex flex-wrap gap-2">
      <span class="badge">Default</span>
      <span class="badge badge-primary">Primary</span>
      <span class="badge badge-secondary">Secondary</span>
      <span class="badge badge-accent">Accent</span>
      <span class="badge badge-success">Success</span>
      <span class="badge badge-warning">Warning</span>
      <span class="badge badge-error">Error</span>
      <span class="badge badge-info">Info</span>
      <span class="badge badge-outline">Outline</span>
      <span class="badge badge-outline badge-secondary">Outline secondary</span>
      <span class="badge badge-primary badge-lg">Large</span>
      <span class="badge badge-primary badge-sm">Small</span>
    </div>
  </section>

  <%# ─────────────────────── ALERTS ────────────────────────────────── %>
  <section aria-labelledby="alerts-heading">
    <h2 id="alerts-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Alerts
    </h2>
    <div class="space-y-3">
      <%# aria-live: "status" for non-urgent updates, "alert" for important/error messages %>
      <% [
        ["alert-info",    "status", "This action has additional context worth noting."],
        ["alert-success", "status", "Changes saved — your work is up to date."],
        ["alert-warning", "alert",  "This will affect all records in the current scope."],
        ["alert-error",   "alert",  "Unable to complete the request. Please try again."]
      ].each do |cls, live, msg| %>
        <div role="alert" aria-live="<%= live %>" class="alert <%= cls %>">
          <span><%= msg %></span>
        </div>
      <% end %>
    </div>
  </section>

  <%# ─────────────────────── FORM INPUTS ───────────────────────────── %>
  <section aria-labelledby="forms-heading">
    <h2 id="forms-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Form Inputs
    </h2>
    <div class="border border-base-300 rounded-2xl p-8 grid grid-cols-2 gap-6">
      <fieldset class="fieldset">
        <legend class="fieldset-legend">Text input</legend>
        <input id="sg-text" type="text" class="input w-full" placeholder="Placeholder text"
               aria-describedby="sg-text-hint" />
        <p id="sg-text-hint" class="fieldset-label text-xs">
          Helper text lives here with <code>aria-describedby</code>.
        </p>
      </fieldset>

      <fieldset class="fieldset">
        <legend class="fieldset-legend">Select</legend>
        <select class="select w-full" aria-label="Select an option">
          <option disabled selected>Choose one…</option>
          <option>Option A</option>
          <option>Option B</option>
          <option>Option C</option>
        </select>
      </fieldset>

      <fieldset class="fieldset" role="radiogroup" aria-labelledby="sg-radio-legend">
        <legend id="sg-radio-legend" class="fieldset-legend">Radio group</legend>
        <div class="flex gap-4 mt-1">
          <% [["a", "Option A"], ["b", "Option B"], ["c", "Option C"]].each do |val, label| %>
            <label class="flex items-center gap-2 cursor-pointer">
              <input type="radio" name="sg-radio" class="radio radio-primary" value="<%= val %>"
                     <%= val == "a" ? "checked" : "" %> aria-label="<%= label %>" />
              <span class="text-sm"><%= label %></span>
            </label>
          <% end %>
        </div>
      </fieldset>

      <fieldset class="fieldset">
        <legend class="fieldset-legend">Toggles &amp; checkboxes</legend>
        <div class="flex flex-col gap-3 mt-1">
          <label class="flex items-center gap-3 cursor-pointer">
            <input type="checkbox" class="toggle toggle-primary toggle-sm" aria-label="Toggle option" />
            <span class="text-sm">Toggle option</span>
          </label>
          <label class="flex items-center gap-3 cursor-pointer">
            <input type="checkbox" class="checkbox checkbox-primary" checked aria-label="Checkbox option" />
            <span class="text-sm">Checkbox option</span>
          </label>
        </div>
      </fieldset>
    </div>
  </section>

  <%# ─────────────────────── FOCUS ─────────────────────────────────── %>
  <section aria-labelledby="focus-heading">
    <h2 id="focus-heading" class="text-xs text-base-content/40 uppercase tracking-widest mb-8">
      Focus &amp; Keyboard
    </h2>
    <div class="border border-base-300 rounded-2xl p-8 flex flex-wrap gap-3">
      <button class="btn btn-primary focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-primary">
        Tab to me
      </button>
      <a href="#focus-heading" class="btn btn-outline focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-primary">
        Link button
      </a>
      <input type="text" class="input" placeholder="Focus this input"
             aria-label="Focus example input" />
      <select class="select" aria-label="Focus example select">
        <option>Select an option</option>
      </select>
    </div>
  </section>

</div>
```

---

## Phase 5 — DESIGN.md Decision Record

Write `DESIGN.md` to the **project root**. This is the persistent written record of every
design decision made during the interview — the "why" that lives alongside the code.

It serves a different purpose from the live styleguide:

| | DESIGN.md | `/styleguide` |
|---|---|---|
| Format | Markdown in the repo | Browser page |
| Audience | Whole team, new contributors, AI tools | Developer coding UI |
| Purpose | Decisions + rationale | Visual reference |

All content comes from the interview answers already gathered in Phase 1. No extra questions needed.

### Color naming convention

Each color gets four pieces of information:
- **Descriptive name** — evocative, not generic (`"Warm Slate"`, not `"dark grey"`)
- **Hex** — for design tools and communication
- **OKLCH** — the actual CSS value in use
- **Role** — what it does in the UI

### DESIGN.md template

```markdown
# Design System: [App name]

> [One-sentence aesthetic thesis — the committed direction agreed in Phase 1]

---

## 1. Visual Theme & Atmosphere

[Two to three sentences. Describe the mood, density, and feel of the UI in plain language.
What does it remind you of? What does it deliberately avoid? What's the one thing
someone will remember after seeing it?]

---

## 2. Color Palette

### Brand

| Name | Hex | OKLCH | Role |
|---|---|---|---|
| [Descriptive name, e.g. "Deep Ocean"] | `#[hex]` | `oklch(L% C H)` | Primary actions, key interactive elements |
| [Descriptive name] | `#[hex]` | `oklch(L% C H)` | Secondary actions |
| [Descriptive name, e.g. "Ember"] | `#[hex]` | `oklch(L% C H)` | Accent — the sharp surprise against primary |
| [Descriptive name] | `#[hex]` | `oklch(L% C H)` | Neutral surfaces and borders |

### Base Surfaces

| Name | Hex | OKLCH | Role |
|---|---|---|---|
| [Descriptive name, e.g. "Warm Paper"] | `#[hex]` | `oklch(L% C H)` | Page background (base-100) |
| [Descriptive name] | `#[hex]` | `oklch(L% C H)` | Cards, sidebar (base-200) |
| [Descriptive name] | `#[hex]` | `oklch(L% C H)` | Borders, dividers (base-300) |
| [Descriptive name] | `#[hex]` | `oklch(L% C H)` | Body text (base-content) |

### Semantic

| Name | Hex | Role |
|---|---|---|
| [Descriptive name, e.g. "Sage"] | `#[hex]` | Success states |
| [Descriptive name] | `#[hex]` | Error and destructive actions |
| [Descriptive name] | `#[hex]` | Warnings and caution states |
| [Descriptive name] | `#[hex]` | Informational context |

---

## 3. Typography

**Display font:** [Name] — [Why it was chosen. What character it brings. One sentence.]
**Body font:** [Name] — [Why it pairs well. What it does for readability.]

**Usage rules:**
- Headings (`h1`–`h3`) use the display font
- All prose, labels, and UI text use the body font
- Two typefaces maximum — no exceptions without strong justification
- Body line-height: `1.6–1.75` | Long-form max-width: `65ch`
- Scale gap between heading levels is generous — each step reads clearly larger

---

## 4. Shape & Depth

**Border radius:** [Plain language description, e.g. "Gently rounded corners — approachable
without being playful. Applied consistently: boxes at `0.5rem`, fields at `0.375rem`, badges pill-shaped."]

**Depth:** [Plain language, e.g. "Flat. No shadows. Surfaces are differentiated by background
color alone (base-100 → base-200 → base-300). This keeps the UI calm and avoids visual noise."]

---

## 5. Motion

Three intentional moments — no more, no less:

1. **Entrance** — [Description, e.g. "Page content fades up 12px on load (`animate-fade-up`,
   0.3s ease-out). Staggered on list items with 50ms delays."]
2. **Interaction feedback** — [Description, e.g. "All interactive elements scale to 95% on
   press (`active:scale-95`, 150ms). Links and nav items fade on hover (`opacity-70`)."]
3. **State change** — [Description, e.g. "DaisyUI `loading-spinner` on async button actions.
   Alerts animate in using the same `fade-up` keyframe."]

---

## 6. Component Patterns

**Buttons:** [Plain language. E.g. "Primary actions use `btn-primary`. Secondary actions use
`btn-ghost` or `btn-outline`. Destructive actions use `btn-error` with a confirmation step.
All buttons include `active:scale-95` for press feedback."]

**Cards:** [Plain language. E.g. "Used only for interactive or selectable content — never as
a default container. `card-border` for explicit grouping. No shadow (flat depth)."]

**Forms:** [Plain language. E.g. "All inputs use DaisyUI defaults (bordered). Grouped with
`<fieldset>` + `<legend>`. Every input has a visible label. Error messages appear below the
field using `fieldset-label text-error`."]

---

## 7. What We Don't Do

- [Anti-pattern specific to this project, e.g. "No card mosaics as primary layout"]
- [Anti-pattern, e.g. "No purple gradients — the palette is warm, not cool-techy"]
- [Anti-pattern, e.g. "No Inter — we chose [font] specifically for its character"]
- [Anti-pattern, e.g. "No competing accent colors — ember is the only accent"]

---

## 8. Living Reference

Browse the full component and token showcase:
**`/styleguide`** — available in development only (`Rails.env.development?`).

The styleguide shows every color, type scale, radius, motion example, and component
variant rendered with the actual theme. DESIGN.md explains the decisions; the styleguide
shows the result.
```

---

## Directory Structure

```
DESIGN.md                              # decision record — aesthetic thesis, colors, type, patterns
app/
├── assets/stylesheets/
│   └── application.tailwind.css       # @plugin "daisyui" + theme + @keyframes + font vars
├── controllers/styleguide/
│   └── styleguide_controller.rb       # dev-only guard
├── helpers/
│   └── styleguide_helper.rb           # NAVIGATION constant
└── views/
    ├── layouts/
    │   └── styleguide.html.erb        # sidebar layout (dev only, animate-fade-up on main)
    └── styleguide/styleguide/
        └── index.html.erb             # design tokens + motion + accessibility showcase
config/
└── routes.rb                          # wrapped in `if Rails.env.development?`
```

---

## Output Checklist

**Setup**
- [ ] Explored `package.json`, `Gemfile`, CSS entry — no blind assumptions made
- [ ] Tailwind v4 confirmed (or upgrade path offered and accepted)
- [ ] `daisyui@^5.x` present in `package.json`
- [ ] `@plugin "daisyui"` wired in the correct CSS entry file
- [ ] `bin/rails tailwindcss:build` (or equivalent) run after CSS changes — compiled output is fresh

**Aesthetic & theme**
- [ ] Aesthetic thesis captured — mood/tone/differentiator agreed before any colors chosen
- [ ] Font pair chosen — display + body, neither is Inter/Roboto/Arial/system-ui
- [ ] `--font-display` and `--font-body` CSS variables set and used in headings/body
- [ ] All OKLCH placeholders replaced with real values (no `[L]`, `[C]`, `[H]` remain)
- [ ] Color palette — one dominant + one sharp accent, not timidly distributed
- [ ] All `*-content` colors pass ≥ 4.5:1 contrast against their background

**Motion**
- [ ] `@keyframes fade-up` and `.animate-fade-up` present in CSS
- [ ] Interaction feedback (`active:scale-95`) on all buttons
- [ ] Loading states use DaisyUI `loading` classes

**Styleguide**
- [ ] Routes wrapped in `if Rails.env.development?`
- [ ] Controller has `development_only!` before_action
- [ ] Styleguide reachable at `/styleguide` in development
- [ ] Dark mode toggle switches `data-theme` correctly
- [ ] Styleguide layout uses the actual display font
- [ ] All sections have `aria-labelledby`, inputs have `aria-label`, alerts have `role="alert"`
- [ ] Visually distinct focus rings on all interactive elements

**DESIGN.md**
- [ ] `DESIGN.md` written to project root
- [ ] All color table rows have descriptive name + hex + OKLCH + role (no placeholders)
- [ ] Atmosphere section reads as a committed paragraph, not a bullet list
- [ ] Component Patterns section written in plain prose (not CSS class names)
- [ ] "What We Don't Do" section lists anti-patterns specific to *this* project, not generic advice
- [ ] Section 8 references the correct `/styleguide` URL

---

## What to Avoid

- **Generic fonts**: Inter, Roboto, Arial, and system-ui produce a generic AI aesthetic — always choose something with personality
- **Card mosaics**: don't default to card grids as the primary layout pattern; cards are for interactive or selectable content only
- **Timid palettes**: evenly-distributed, low-contrast palettes read as uncommitted — pick a dominant color and commit to it
- **Scattered micro-animations**: define 2–3 intentional motion moments rather than animating everything weakly
- **Missing ARIA**: every section, alert, and interactive element needs semantic markup — accessibility is not optional
- **Competing accents**: one accent color unless an established system already exists
- **Mechanical extraction**: color choices should serve the aesthetic thesis, not just be copied from the screenshot pixel-by-pixel
