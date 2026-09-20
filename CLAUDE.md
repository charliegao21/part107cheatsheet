# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, zero-dependency interactive study guide for the FAA Part 107 (commercial sUAS/drone pilot) exam. Everything — markup, CSS, and JavaScript — lives in [index.html](index.html). There is no build step, no framework, no package manager, and no tests.

To view: open `index.html` in a browser (or serve the directory, e.g. `python -m http.server`). The only network dependency is the IBM Plex font from Google Fonts; it degrades to system fonts offline.

## Content model

The study material is organized into **nine "sheets"**, each a `<section class="sheet" id="p1">` … `id="p9"`. Sheet titles: The Test, Regulations, Airspace, Sectional Charts, Weather, Loading & Performance, Operations, Maintenance & Emergencies, and the "Night-Before Sheet" (numbers/mnemonics/traps). Only one sheet is visible at a time (`.sheet.is-on`); the rest are `display:none`.

The left nav (`.navlink` buttons) and sheets are paired **by DOM order, by index** — `links[x]` drives `sheets[x]`. If you add, remove, or reorder a sheet, keep the nav list and the `<section>` order in exact lockstep, or the `go(n)` navigation will point at the wrong content.

## The one script (bottom of index.html, ~line 1443)

A single IIFE wires up all interactivity against DOM that already exists in the HTML. Key pieces:

- **`go(n)`** — sheet switching, driven by index. Called by nav clicks, the generated prev/next footers, and ArrowLeft/ArrowRight.
- **Search** — the `#q` box does live full-text search across all nine sheets. `markSheet()` walks text nodes with a `TreeWalker` and wraps matches in `<mark class="hl">`; `clearHits()` unwraps them and re-normalizes. Enter / Shift+Enter (or the ▲▼ buttons) step through hits, jumping to the right sheet; `/` focuses the box; Escape clears it. **Search deliberately skips `<svg>` and `.footnav`** (see the `acceptNode` filter) — injecting an HTML `<mark>` into SVG breaks labels. Preserve that exclusion.
- **Airspace diagram (Sheet 3)** — the `ASP` object is the single source of truth for the Class A/B/C/D/E/G explainer content. Clicking (or Enter/Space on) an SVG group with a `data-cls` attribute calls `pick()`, which renders that class's rows into `#aspRows`. To edit airspace copy, edit the `ASP` object, not the DOM.
- **Print** — "Print all" calls `window.print()`. The `@media print` block (~line 263) force-shows every sheet, hides nav/masthead/footers, and sets page breaks so the whole guide prints as a packet.

## Conventions to match

- **Colors are semantic and map to real sectional-chart conventions** — CSS custom properties in `:root`: `--mag` (magenta) = uncontrolled/advisory, `--blue` = controlled/regulatory, `--grn` = allowed, `--red` = prohibited/limit. Reuse these variables rather than hardcoding hex; the color choices carry meaning for the reader.
- Content is hand-authored HTML tables and cards. Special characters use HTML entities (`&mdash;`, `’` inside JS strings, etc.).
- Accessibility is intentional: `aria-live`, `aria-current`, keyboard handlers on interactive SVG, `prefers-reduced-motion`, and visible focus rings. Keep new interactive elements keyboard-reachable and labeled.
