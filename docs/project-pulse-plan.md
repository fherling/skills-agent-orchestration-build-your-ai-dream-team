# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Build a small static Project Pulse dashboard for Mona's team as four files: `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`. The dashboard fetches the JSON data, renders one card per project (name, owner, status, recent activity, priority, and short summary), and applies polished styling via `.dashboard` and `.project-card` hooks. A VS Code launch configuration named **"Run Project Pulse Dashboard"** starts a local static server rooted in `app/` and opens `index.html` in a browser so learners see the dashboard — not a directory listing.

Work splits cleanly between two specialists:
- **Designer** owns visual + semantic structure (`app/styles.css`, plus the HTML skeleton/semantics of `app/index.html`).
- **Coder** owns runtime behavior (`app/index.html` fetch/render script), the data file (`app/project-data.json`), and the launch config (`.vscode/launch.json`).

Because both specialists touch `app/index.html`, that file is sequenced across two phases (Designer authors the skeleton first, then Coder wires the rendering script) to avoid write conflicts.

## 2. Ordered implementation steps

1. **Data contract lock-in.** Freeze the JSON schema: top-level `projects` array with objects containing `name`, `owner`, `status`, `recentActivity`, `priority`, plus one extra `summary` field to satisfy the brief's "short contributor-friendly summary." Freeze the enumerated values Designer will style: `status` ∈ {`Active`, `At Risk`, `Blocked`, `Complete`}; `priority` ∈ {`High`, `Medium`, `Low`}. (Prep for Designer + Coder.)
2. **Designer — HTML skeleton + semantics for `app/index.html`.** Author a semantic shell (`<header>`, `<main class="dashboard">`, a `<section>` or `<ul>` list container with a stable id like `#project-list`, and a single hidden `<template id="project-card-template">` describing the `.project-card` markup with named hook classes: `.project-card__title`, `.project-card__owner`, `.project-card__status` (badge), `.project-card__priority` (badge), `.project-card__activity`, `.project-card__summary`). Link `styles.css`. Leave a `<script>` placeholder tag (empty or with a `TODO` comment for Coder). Include ARIA labels, heading order, and a title.
3. **Designer — `app/styles.css`.** Implement polished layout: `.dashboard` grid/flex container, `.project-card` with `border-radius`, `box-shadow`, padding, readable typography; status/priority badges with color variants keyed by data attributes or modifier classes (e.g., `[data-status="At Risk"]`, `.priority--high`); responsive breakpoint; accessible contrast.
4. **Coder — `app/project-data.json`.** Write 4–6 realistic sample projects that exercise every enumerated `status` and `priority` value. Strict JSON (no comments, no trailing commas). Include the `summary` field on each project.
5. **Coder — `app/index.html` script wiring.** Inside the reserved `<script>` slot, `fetch('./project-data.json')`, parse, iterate `data.projects`, clone the template, populate fields, set `data-status`/`data-priority` attributes (or modifier classes) matching what Designer styled, append to `#project-list`. Handle fetch failure with a visible message. Must not alter Designer's structural markup or CSS class names.
6. **Coder — `.vscode/launch.json`.** Strict JSON, no comments. One configuration named exactly `Run Project Pulse Dashboard` that launches a static server rooted at `${workspaceFolder}/app` and opens `index.html` (not `/`). See §9 for validation criteria.
7. **Integration verification (Orchestrator).** Run the launch config, confirm dashboard renders cards from JSON, confirm required selectors and fields are present.

## 3. File assignments

| File | Owner(s) | Notes |
|---|---|---|
| `app/index.html` | **Designer** (Step 2: structure, semantics, CSS hooks, template), then **Coder** (Step 5: script wiring only) | Shared file — strictly sequenced, not parallel. Coder edits only inside the reserved `<script>` region. |
| `app/styles.css` | **Designer** (sole owner) | Must include `.dashboard` and `.project-card` selectors plus badge/priority styling. |
| `app/project-data.json` | **Coder** (sole owner) | Content follows schema locked in Step 1. |
| `.vscode/launch.json` | **Coder** (sole owner) | Strict JSON, `cwd = ${workspaceFolder}/app`, opens `index.html`. |

## 4. Designer responsibilities

- Define information hierarchy: dashboard header/title → grid of project cards → each card leads with project name + status badge, then owner, priority, recent activity, summary.
- Author the semantic HTML skeleton and the `<template id="project-card-template">` that Coder will clone. Establish every CSS hook Coder must populate (`.project-card__title`, `__owner`, `__status`, `__priority`, `__activity`, `__summary`) plus the `data-status` / `data-priority` attribute contract.
- Deliver `app/styles.css` with: `.dashboard` container, `.project-card` (border-radius, box-shadow, spacing, typography), status badge color variants, priority visual weighting, responsive layout (single column on narrow screens, multi-column on wider), accessible color contrast, focus styles.
- Do **not** write the fetch/render script. Leave a clearly reserved `<script>` slot for Coder.
- Report design tradeoffs, tokens/colors chosen, and any accessibility notes.

## 5. Coder responsibilities

- Author `app/project-data.json` with a top-level `projects` array containing 4–6 items; each item includes all six fields (`name`, `owner`, `status`, `recentActivity`, `priority`, `summary`). Strict JSON.
- Wire `app/index.html`'s reserved `<script>` slot to fetch and render the JSON into Designer's template. Populate the exact class/attribute hooks Designer defined; do not rename or restructure them. Handle fetch/parse errors with a visible fallback message.
- Author `.vscode/launch.json` with a single configuration:
  - `name`: `Run Project Pulse Dashboard`
  - Serves from `${workspaceFolder}/app` (either via a compound `preLaunchTask` starting a static server + a `chrome`/`msedge` launch to `http://localhost:<port>/index.html`, or via the built-in Simple Browser / `type: node` running a static server — pick one deterministic approach).
  - Explicitly opens `index.html` (not `/`) so no directory listing appears.
  - Strict JSON, no comments, no trailing commas.
- Do not modify `styles.css` or Designer-owned markup structure.

## 6. Dependencies

- Step 1 (schema lock) → blocks Steps 2, 3, 4, 5.
- Step 2 (Designer HTML skeleton + CSS hook contract) → blocks Step 3 (styles reference these hooks) and Step 5 (Coder script targets these hooks).
- Step 4 (JSON) → blocks Step 5 (script consumes it) and Step 7 (verification).
- Step 5 (script wiring) depends on Steps 2 and 4.
- Step 6 (`launch.json`) depends only on the existence of `app/index.html` in principle; can start as soon as Step 2 lands.
- Step 7 depends on all of 2–6.

## 7. Parallel work decisions

**Can run in parallel:**
- After Step 1: **Step 4 (Coder writes `project-data.json`)** and **Step 2 (Designer authors HTML skeleton)** — different files, no overlap.
- After Step 2: **Step 3 (Designer `styles.css`)** and **Step 6 (Coder `.vscode/launch.json`)** — different files, no overlap. Step 4 can also still be in flight.

**Must run sequentially:**
- Step 1 before everything else — the schema and enum values are a shared contract.
- Step 2 before Step 5 — both edit `app/index.html`; sequencing them into separate phases prevents write conflicts and guarantees Coder targets the real CSS hooks.
- Step 5 after Step 4 — the script needs the JSON to exist for a meaningful end-to-end test.
- Step 7 last — integration check requires all artifacts.

## 8. Edge cases to handle

- **`fetch()` blocked by `file://` protocol.** If a learner opens `index.html` directly without the launch config, `fetch` fails. Script must show a visible error message telling the user to use the launch configuration.
- **Missing/misspelled fields in JSON.** Render defensively: missing `summary` or `recentActivity` shouldn't break the card; use empty-string fallbacks.
- **Unknown enum values** for `status` or `priority` (typo in data). Fall back to a neutral badge style rather than unstyled text.
- **Empty `projects` array.** Show an empty-state message inside `.dashboard`.
- **JSON parse error / network error.** Catch and display an error message in the DOM, not just the console.
- **Directory listing risk.** If the launch config points to `http://localhost:<port>/` instead of `/index.html`, some static servers show a listing. URL must end in `index.html`.
- **Strict JSON pitfalls in `launch.json`.** No `//` comments (VS Code tolerates JSONC, but the brief requires strict JSON), no trailing commas.
- **Port conflicts.** Choose a stable, uncommon port (e.g., 5173 or 4173) or document the choice.
- **Accessibility.** Status colors must not be the only signal — include text labels inside badges. Ensure heading order and keyboard focus visibility.
- **Overlapping edits to `app/index.html`.** Enforced by sequencing (§7); Coder edits only the reserved `<script>` region.

## 9. Validation expectations

**`app/project-data.json`**
- Passes `JSON.parse` / `jq . app/project-data.json` with no errors.
- Top-level key `projects` is an array with ≥ 1 element.
- Every element contains all five required fields: `name`, `owner`, `status`, `recentActivity`, `priority` (plus `summary`).
- Enum values match what `styles.css` styles.

**`app/styles.css`**
- Contains a `.dashboard` selector and a `.project-card` selector (grep both).
- `.project-card` rule includes `border-radius` and `box-shadow` declarations.
- Includes visual styling for status badges and priority indicators.

**`app/index.html`**
- Valid HTML5 (`<!doctype html>`, `<html lang>`, `<title>`).
- Links `styles.css`.
- References the `project-card` class (either directly in markup or in a `<template>` used at runtime); after script execution, visible elements with class `project-card` exist in the DOM — one per project in the JSON.
- Renders the five required data fields per card visibly.
- Uses a container element with class `dashboard`.
- Handles fetch failure with a visible message.

**`.vscode/launch.json`**
- Strict JSON: parses cleanly with `jq . .vscode/launch.json`; contains no `//` or `/* */` comments and no trailing commas.
- Contains a configuration whose `name` is exactly `Run Project Pulse Dashboard`.
- `cwd` (or equivalent server root) resolves to `${workspaceFolder}/app`.
- The URL opened ends in `index.html` (not `/` or the folder root), so no directory listing appears.
- Launching it from VS Code's Run panel opens a browser showing the styled dashboard with cards populated from JSON.

**Integration check (Step 7)**
- Run the launch config → browser opens to the served `index.html` → dashboard renders N cards where N matches `projects.length` → all five fields visible per card → status/priority badges are visually distinct → responsive at narrow widths.

## 10. Open questions

1. **Static server choice for `launch.json`.** VS Code's built-in `chrome`/`msedge` debug types can point at a `file://` URL, but that breaks `fetch`. Preferred approach: a `preLaunchTask` that runs `npx serve app` (or Python's `http.server`) plus a browser launch to `http://localhost:<port>/index.html`. Does the exercise environment have Node/npx or Python guaranteed? If neither, we need a different strategy (e.g., VS Code Live Preview extension).
2. **Port number.** Any preference (5173, 4173, 8080)? Assume 5173 unless told otherwise.
3. **Number of sample projects.** Brief doesn't specify. Plan assumes 4–6 to exercise all enum variants.
4. **`summary` field.** Not in the brief's required field list but is called out as a required piece of information ("short contributor-friendly summary"). Plan adds it as a sixth field on each project. Confirm this is acceptable.
5. **Framework/library policy.** Plan assumes vanilla HTML/CSS/JS (no build step, no npm dependencies inside `app/`), consistent with "small static app." Confirm.
6. **Dark mode / theming.** Not requested; plan omits it. Confirm not needed.
7. **Persistence of `data-status` casing.** Plan uses the exact enum strings (e.g., `"At Risk"`) as `data-status` attribute values. Alternative is slugified values (`"at-risk"`). Either works; Designer and Coder must agree in Step 1.
