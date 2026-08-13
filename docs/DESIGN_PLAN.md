# Signal & Paper Redesign Plan

## Goal

Apply the design system in [`.interface-design/system.md`](../.interface-design/system.md) to the public site without changing its factual content or publishing hidden example material. The redesign should help research supervisors, collaborators, and technical recruiters identify the research direction quickly and inspect project evidence efficiently.

This document is an implementation plan only. Saving it does not authorize the visual changes described below.

## Scope

The redesign covers the global shell, homepage, project index, project detail pages, repositories page, footer, light mode, dark mode, and responsive states. The CV remains a PDF link from the homepage. Example posts and other unpublished template content remain excluded.

## Implementation Boundaries

- Extend `assets/css/main.scss` for site-level tokens and styling.
- Reuse existing page content and front matter wherever possible.
- Avoid creating `_layouts/`, `_includes/`, `_sass/`, or another starter-local build pipeline.
- Use CSS pseudo-elements for the signal-path motif when existing markup is sufficient.
- If a required layout change cannot be made without overriding gem-owned markup, stop and review that boundary before implementation.
- Keep code, mathematical notation, and icon fonts unchanged.

## Phase 1 — Baseline and Tokens

1. Capture desktop (`1440 × 1000`) and mobile (`390 × 844`) baselines for `/`, `/projects/`, one project detail page, and `/repositories/` in light and dark mode.
2. Add named CSS custom properties for the approved light and dark palettes, typography scale, spacing unit, border, radius, and transition values.
3. Map existing al-folio global theme variables to the new palette without introducing one-off colors.
4. Confirm EB Garamond weights 400–700, italic, bold, code, icons, and fallback behavior.

Acceptance: all public pages build; text, icons, code, and both themes remain readable; no layout changes have occurred yet.

## Phase 2 — Global Shell

1. Refine the navigation height, typography, divider, active state, hover state, and keyboard focus.
2. Increase the main content width only where necessary while preserving a readable text measure.
3. Replace the heavy footer treatment with the signal-path terminus pattern.
4. Apply consistent page-title and section-heading hierarchy.

Acceptance: navigation works at desktop and mobile widths; menus do not overflow; focus is visible; footer content remains complete.

## Phase 3 — Homepage

1. Retain the existing introduction and portrait content while strengthening the text-first hierarchy.
2. Balance the desktop split and mobile stacking order without duplicating contact or education details.
3. Use signal-path section treatment only where the existing content provides a real section boundary.
4. Reduce the visual dominance of the CV and email icons while keeping both easy to discover.

Acceptance: a first-time visitor can identify name, field, current research, representative systems, and CV access within the first screen or one short scroll.

## Phase 4 — Projects

1. Restyle the project index as numbered editorial entries instead of generic floating cards.
2. Preserve category and importance ordering; do not alter project facts or URLs.
3. Apply the article reading width, larger body type, section rhythm, expanded technical figures, and same-size blockquotes to every project detail page.
4. Standardize captions and spacing around system-overview images.

Acceptance: all four projects have consistent hierarchy; images do not overflow; captions and blockquotes remain legible; source links are obvious.

## Phase 5 — Repositories and Supporting States

1. Make repository entries visually consistent with project evidence while remaining secondary to the authored project pages.
2. Verify loading, empty, and upstream-image-failure behavior already provided by the theme.
3. Check search overlay, dropdowns, external-link indicators, hover, focus, and visited-link behavior against the palette.

Acceptance: repository names, descriptions, language metadata, and links remain readable even when an external statistics image fails.

## Phase 6 — Responsive, Dark Mode, and Accessibility

1. Review widths at `390`, `768`, `1024`, and `1440` pixels.
2. Check light mode, dark mode, zoom at 200%, long links, and reduced-motion preference.
3. Verify contrast, focus order, landmark structure, alternative text, and target sizes.
4. Confirm the signal-path motif appears in the five documented locations and remains restrained.

Acceptance: no horizontal overflow, clipped controls, unreadable text, missing focus indicators, or contrast regressions.

## Phase 7 — Verification and Release

Run:

```bash
npm run lint:prettier
npm run lint:style-contract
bundle exec jekyll build
```

Then run the relevant integration and visual tests, inspect desktop and mobile screenshots, check internal public links, and review the final diff for accidental content or generated artifacts.

Release only after the user approves the rendered light and dark previews. Keep visual redesign changes in a dedicated commit so they can be reviewed or reverted independently.

## Explicit Non-goals

- No new biography, education, research claim, project metric, or social account
- No animated hero, background particles, decorative circuit pattern, or custom illustration
- No CV page restoration
- No new content section that repeats existing information
- No modification to gem-owned runtime behavior
