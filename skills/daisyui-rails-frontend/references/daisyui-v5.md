# daisyUI v5 Notes

Use this file when the task depends on current daisyUI behavior or theme syntax.

Source: official daisyUI v5 docs via Context7.

## Tailwind CSS 4 Setup

Use the CSS-based plugin flow:

```css
@import "tailwindcss";
@plugin "daisyui";
```

Do not assume the older `tailwind.config.*` plugin pattern for Tailwind 4 projects.

## Theme Tokens

daisyUI v5 uses readable semantic CSS variables such as:

- `--color-base-100`
- `--color-base-200`
- `--color-base-300`
- `--color-base-content`
- `--color-primary`
- `--color-primary-content`
- `--color-secondary`
- `--color-accent`
- `--color-neutral`
- `--color-info`
- `--color-success`
- `--color-warning`
- `--color-error`

Values are commonly written in `oklch(...)`. Keep design decisions semantic and traceable to these variables.

## Theme Customization

Override theme values with the daisyUI theme plugin block:

```css
@plugin "daisyui";

@plugin "daisyui/theme" {
  name: mytheme;
  --color-base-100: oklch(98% 0.01 95);
  --color-base-200: oklch(95% 0.015 95);
  --color-base-300: oklch(90% 0.02 95);
  --color-base-content: oklch(24% 0.03 250);
  --color-primary: oklch(64% 0.2 145);
  --color-primary-content: oklch(99% 0.01 95);
}
```

Use this for brand adaptation instead of scattering custom hex values through components.

## Font Customization

Set font identity at theme level when the default daisyUI typography weakens the concept:

```css
@plugin "daisyui/theme" {
  name: mytheme;
  font-family: "Assistant", "Segoe UI", sans-serif;
}
```

Prefer a deliberate display and body pairing when the project permits custom fonts.

## Component Output Control

Limit generated CSS when needed:

```css
@plugin "daisyui" {
  include: button, card, input, modal;
}
```

```css
@plugin "daisyui" {
  exclude: scrollbar;
}
```

Use this in focused apps or performance-sensitive builds.

## No-Build / Micro CSS

For no-build cases, daisyUI v5 supports component-level CSS files from CDN, for example:

```text
https://cdn.jsdelivr.net/npm/daisyui@5/components/toggle.css
```

Use this only when the project truly cannot run the normal Tailwind build.

## Practical Guidance

- Start from daisyUI semantics, then customize tokens aggressively enough that the UI no longer feels like a stock theme.
- Use `base-*` tokens to establish surface hierarchy across canvas, panels, and overlays.
- Keep raw Tailwind color utilities as exceptions, not the main system.
- Prefer theme-level adaptation over per-component overrides when multiple screens share the same visual language.
- Re-check Context7 if the task depends on exact component APIs, theme syntax, or installation details.
