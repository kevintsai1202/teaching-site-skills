# Teaching Site — Domain Primitives (Single Source of Truth)

This document is the **authoritative schema** for every data primitive used across the teaching-site skill chain. Every skill in this repo references this file — if a field name, type, or rule disagrees here vs. there, **this file wins**.

Extracted from the production reference implementation `ai-workshop/` (4 days × 6.5 h workshop, 4125-line `index.html`, 1728-line `course-data.js`).

---

## 0. Canonical Project Layout (English Filenames)

```
{project-root}/
├── index.html                     ← single SPA (CSS + JS inline, ~3000–4000 lines)
├── course-data.js                 ← window.COURSE = { ... }
├── instructor-data.js             ← optional: window.INSTRUCTOR (scraped)
├── tools-data.js                  ← optional: window.TOOLS (scraped)
├── viewer.html                    ← unified material viewer (md/csv/yaml in <iframe>)
├── package.json                   ← { "scripts": { "serve": "npx serve ." } }
├── assets/                        ← visual assets (see web-visual-assets)
│   ├── illustrations/             ← per-unit images (PNG + SVG fallback)
│   ├── scenarios/                 ← per-unit scene illustrations
│   ├── cases/                     ← shared case visuals
│   ├── characters/                ← persona portraits
│   ├── tools/                     ← scraped tool screenshots
│   ├── qr/                        ← generated QR codes
│   └── maps/                      ← classroom map screenshots/SVG
├── scripts/                       ← scraping / build / verification scripts
└── course-package/                ← markdown source of truth (read-only from SPA)
    ├── overview.md                ← top-level meta + day list (ex 課程總覽.md)
    ├── shared-scenario.md         ← persistent fictional case (ex 共用案例設定.md)
    ├── supporting-docs.md         ← FAQ / pre-reading / setup (ex 課程輔助文件.md)
    ├── day1/
    │   ├── outline.md             ← per-day outline (ex 課程大綱.md)
    │   └── content.md             ← per-day lecture script (ex 課程內容.md)
    ├── day2/, day3/, day4/        ← same structure
    └── materials/                 ← downloadable learner files (ex 教學素材/)
        ├── README.md              ← material index (audience: 學員 / 講師)
        ├── *.md / *.csv / *.yaml  ← raw material files
        ├── pdf/                   ← generated PDFs (build-pdf.ps1)
        └── _build/                ← pdf build script + style.css
```

> **English-first rule**: All filenames and directory names are English. Trigger phrases inside skill descriptions remain Chinese (because users speak Chinese), but **anything written to disk is English**. The previous Chinese-named structure (`完整課程包/`, `課程總覽.md`, etc.) is the legacy ai-workshop layout — new projects MUST use the English names above.

---

## 1. Top-Level: `window.COURSE`

```ts
window.COURSE = {
  meta:        Meta,                  // course-wide metadata
  sharedCase:  SharedCase,            // recurring fictional context
  day1: Day, day2: Day, day3: Day, day4: Day,    // one Day per teaching day
  materials:   Material[],            // cross-day material index
  quiz:        QuizItem[]             // optional end-of-course assessment
}
```

The `dayN` keys are **string-indexed**, not array. `meta.days[].id` must match these keys (`'day1'`, `'day2'`, ...). The renderer iterates `meta.days[]` and looks up `window.COURSE[day.id]`.

---

## 2. `Meta` — Course-Wide Metadata

```ts
type Meta = {
  title:        string                // e.g. 'AI Workflow Automation'
  subtitle?:    string
  program?:     string                // sponsoring program
  organizer?:   string
  dates:        string                // human-readable date list
  location:     string
  mapUrl?:      string                // Google Maps link
  format?:      string                // 'in-person' / 'online' / 'hybrid + N hours'
  instructor:   string
  completion?:  string[]              // certificate criteria
  objectives:   string[]              // 3–5 learning outcomes (action verbs)
  days: {
    id:    string                     // 'day1', 'day2', ... — MUST match window.COURSE[id]
    n:     number                     // 1, 2, 3, ...
    date:  string                     // 'M/D' or 'YYYY-MM-DD'
    title: string                     // day theme
    hours: number                     // contact hours that day
  }[]
}
```

---

## 3. `SharedCase` — Persistent Fictional Context

```ts
type SharedCase = {
  intro: string                       // 1-paragraph summary
  brands: Brand[]                     // 1–N fictional companies
  roles:  [name, brand, role, description][]
  variables: [token, brandA_value, brandB_value][]   // e.g. {店名}, {產品}
  tools?:    [name, description, usedIn][]
  pricing?:  Concept[]                // tool pricing comparison (rendered as concepts)
  deliverables?: [day_label, output_description][]
}

type Brand = {
  id:   string                        // 'A', 'B', ...
  name: string                        // brand display name
  type: string                        // industry / category
  rows: [field, value][]              // structured profile (業態, 地點, 員工 ...)
}
```

> If a course has no shared scenario, write `sharedCase: null` and include an explicit one-line note in `course-package/overview.md`: `本課無共用案例，每單元獨立舉例` — see `course-outline-design`.

---

## 4. `Day` — Per-Teaching-Day Container

```ts
type Day = {
  id:           string                // 'day1', 'day2', ... (matches meta.days[].id)
  title:        string
  date:         string
  hours:        string                // '6.5 hours' / '6 hours'
  learningGoal: string                // one-paragraph day-level outcome
  schedule:     [time_range, segment_title, focus][]   // hourly time table
  hero?:        { title, lead, illustration? }
  units:        Unit[]                // chapters within the day
  homework?:    string[]              // optional take-home items
}
```

---

## 5. `Unit` — Chapter Within a Day

```ts
type Unit = {
  id:           string                // 'u1', 'u2', ... (unique within the day)
  title:        string
  subtitle?:    string                // optional secondary line shown under title
  time:         string                // 'HH:MM ~ HH:MM'
  goals:        string[]              // 3–6 action-verb outcomes for the unit
  concepts:     Concept[]             // teaching blocks (the BODY of the unit)
  prompts:      Prompt[]              // RTFC prompt examples (may be empty)
  tasks:        Task[]                // checklist items (localStorage-persisted)
  materials:    Material[]            // downloadable files referenced in this unit
  illustrations: Illustration[]       // 1–3 supporting images (Stage 5 contract)
  faq?:         [question, answer][]  // optional Q&A
  __thinDemo?:  boolean               // ONLY set when Stage 2 was bypassed (renders badge)
}
```

**Display label (zh-TW UI)**: 「單元」 — used in sidebar headings and accordion summaries.
**Display label (en UI)**: "Unit"

### Render Order (fixed, in `renderUnit`)

```
1. <h4>學習目標 / Learning Goals</h4> + bullet list of unit.goals
2. (optional) renderScenarioVisual(unit.id)
3. <h4>核心觀念 / Core Concepts</h4> + unit.concepts.map(renderConcept)
4. <h4>RTFC 提示詞範例 / Prompt Examples</h4> + unit.prompts.map(renderPromptCard)
5. <h4>任務清單 / Tasks</h4> + renderTaskList(unit.tasks)
6. <h4>本單元教學素材 / Unit Materials</h4> + renderMaterialList(unit.materials)
7. <h4>常見學員疑問 / FAQ</h4> + faq table
```

Empty arrays skip the entire section. Render order is fixed — do not reorder.

---

## 6. `Concept` — Teaching Block (the unit's substance)

```ts
type Concept = {
  heading:      string                // section title (plain text, no markdown)
  body?:        string                // narrative paragraph (supports **bold** + [link])
  illustration?: string               // illustration name (PNG-first / SVG fallback)
  list?:        [key, value][]        // K-V row list (rendered as .kv rows)
  table?: {
    head: string[]                    // column headers (supports inline markdown)
    rows: string[][]                  // data rows (supports inline markdown per cell)
  }
  note?:        string                // callout / footnote (rendered as .note block)
}
```

**Display label**: 「核心觀念」/ "Core Concepts" — this is where teaching content lives. **The user's term 「範例 (example)」 maps here when the example is a comparison / table / before-after**; when the example is a copyable prompt, use `Prompt` instead.

**Renderer adds a copy-to-markdown button** to every concept (top-right corner). Don't manually wire this.

### Authoring Conventions

- `heading` is always plain text (no `**`, no `[]()`).
- `body` / `list` cells / `table` cells / `note` support **inline markdown only** (`**bold**` + `[text](url)` — no headers, no code blocks).
- For long form prose: prefer breaking into multiple concepts with their own headings, not one giant `body` paragraph.
- Tables max ~6 columns; wider tables wrap badly on mobile.
- `list` is for definition-style pairs (`['擅長', '改寫']`); `table` is for matrix comparisons (3+ columns).

---

## 7. `Prompt` — RTFC Prompt Example (copyable)

```ts
type Prompt = {
  id:    string                       // 'd1-p1', 'd1-p2', ... (unique within course)
  title: string                       // short label, e.g. 'A. 公告類（暖光咖啡）'
  note?: string                       // one-line context shown under title
  text:  string                       // full prompt body (preserved with whitespace in <pre>)
}
```

**Display label**: 「提示詞範例 / RTFC 提示詞」 / "Prompt Example"

**Render**: `.prompt-card` with header (title + note) + `<pre>` body + copy-to-clipboard button (with toast on success).

**id convention**: `d{N}-p{M}` (Day N, Prompt M within the course). IDs leak into anchor links — don't rename.

---

## 8. `Task` — Checklist Item (localStorage-persisted)

```ts
type Task = {
  id:    string                       // 'd1-u1-t1' or 'day1-u1-t1' — see lifecycle rules
  label: string                       // single-line description shown next to checkbox
}
```

**Display label**: 「任務 / 任務清單」 / "Tasks"

### Task ID Lifecycle (HARD RULES — break and you lose student progress)

1. **Never rename** a published task ID. The id is the localStorage key (`state.tasks[id] = true|false`).
2. **Never reuse** a deleted task ID for a new task. Retired IDs are permanently retired.
3. **Two formats coexist** in legacy projects: `d{n}-u{m}-t{k}` and `day{n}-u{m}-t{k}`. Pick **one** for new projects (recommended: `d{n}-u{m}-t{k}` — shorter).

### Storage Schema

```ts
localStorage['{project-prefix}-progress-v1'] = JSON.stringify({
  tasks: { 'd1-u1-t1': true, 'd1-u1-t2': true, ... },   // only checked items kept
  quiz:  { 'q1': 1, 'q2': 0, ... },                     // selected option index per question
  lastSection: 'day2-u3'                                // for scrollspy resume
})
```

The `-v1` version suffix is mandatory — schema migrations bump to `-v2` non-destructively.

---

## 9. `Material` — Downloadable File

```ts
type Material = {
  id:   string                        // 'd1-m1', 'd1-m2', ...
  name: string                        // human-readable display name
  type: 'PDF 文件' | 'TEXT' | 'CSV' | 'YAML' | 'MD' | '音檔'
  desc?: string                       // optional one-line context shown under name
}
```

**Display labels**: 「教學素材 / 下載檔案」 / "Materials" / "Downloads" — these are the **same primitive**, different UI labels depending on context (per-unit list = "教學素材", cross-day overview = "下載檔案總覽").

**Material `type` semantics**:
- `'PDF 文件'` → routes to `course-package/materials/pdf/{file}.pdf`, link gets `download` attribute (forces download)
- `'TEXT' | 'CSV' | 'YAML' | 'MD'` → routes to `course-package/materials/{file}.{ext}`, opens in `viewer.html?file=...` (unified viewer)
- `'音檔'` → routes to `assets/{file}.m4a`, browser-native HTML5 audio player

### Three-Place Sync Rule (HARD RULE — break and learner gets 404)

When adding a material, update **all three** places:

1. Drop the file into `course-package/materials/` (or `course-package/materials/pdf/` for PDFs).
2. Add an entry to `course-data.js:materials[]` (cross-day index) AND to the relevant `unit.materials[]`.
3. Add a `name.includes(...)` rule to `getMaterialUrl()` in `index.html`.

Missing any one → broken link. `web-content-audit` includes a check that diffs filesystem ↔ data ↔ router and reports orphans.

---

## 10. `QuizItem` — Assessment Question

```ts
type QuizItem = {
  id:       string                    // 'q1', 'q2', ... — sequential, no gaps
  type:     'single'                  // (only single-answer supported in current renderer)
  q:        string                    // question text
  options:  string[]                  // 2–5 answer choices
  answer:   number                    // 0-indexed correct option
}
```

**Display label**: 「結訓測驗 / 測驗」 / "Final Quiz" / "Quiz"

### Quiz Renumber Trap (HARD RULE — break and 5+ places diverge)

Changing `quiz[]` length forces synchronized updates in:

1. `course-data.js:quiz[]` — the array itself
2. `index.html:qIndexToDay()` — maps `q1..qN` → source day for "review chapter" links
3. `index.html` hardcoded strings (typically 5 places):
   - Section title: `"結訓測驗（N題）"`
   - Lead text mentioning total
   - Score display (two places): `— / N`
   - Toast message on submit
   - Passing threshold check: `if (score >= K)`

Treat quiz items as **append-only**: deleting `q3` leaves a hole in the visible numbering — only renumber at a deliberate "stable moment" with a checklist of all 5+ places to update.

---

## 11. `Illustration` — Visual Asset Slot (Stage 5 contract)

```ts
type Illustration =
  | { name: string, kind: 'hero' | 'diagram' | 'screenshot' | 'scene', alt: string, spec?: string }
  | { kind: 'waived', reason: string }                  // explicit "no image needed"
```

**Coverage Floor**: every `Unit` MUST have `illustrations.length >= 1` and `<= 3`. Empty arrays fail the gate (see `web-visual-assets` Coverage Floor).

**Legacy compatibility**: older projects may have `unit.illustration: 'foo.png'` (single string). Treat as `illustrations: [{ name: 'foo.png', kind: 'hero' }]` and migrate.

`name` resolves to `assets/illustrations/{name}.png` first, then `assets/illustrations/{name}.svg` as fallback (defensive against PNG generation failures).

---

## 12. `FAQ` — Per-Unit Q&A (optional)

```ts
type FAQ = [question: string, answer: string]
```

Stored as `unit.faq: FAQ[]`. Rendered as a 2-column table at the bottom of the unit body. Use sparingly — only for misconceptions you've observed students hit; not as a substitute for clear `concepts`.

---

## 13. Cross-Primitive Sync Rules (Summary Table)

| Action | Files / fields to update | Audit script |
|---|---|---|
| Add material | (1) `course-package/materials/{file}` (2) `course-data.js:materials[]` + `unit.materials[]` (3) `getMaterialUrl()` | `web-content-audit` |
| Rename unit ID | **Don't.** Renaming wipes student progress (id = localStorage key segment) | n/a |
| Rename task ID | **Don't.** Same as above | n/a |
| Add quiz item | (1) `quiz[]` (2) `qIndexToDay()` (3) all 5 hardcoded `(N題)` / `— / N` / threshold strings | manual diff |
| Add unit | (1) `course-package/day{n}/outline.md` (2) `course-package/day{n}/content.md` `## u-{id}` section (3) `course-data.js:day{n}.units[]` (4) `assets/illustrations/day{n}-{id}-*.png|svg` | `web-content-audit` |
| Switch theme | Only `[data-theme="dark|light"]` attribute on `<html>` (writes to localStorage) | n/a |

---

## 14. Required Templates (Where to Start a New Project)

Don't re-derive these from scratch — use the templates that ship with the relevant skill:

| File | Template location | Owner skill |
|---|---|---|
| `course-data.js` | `static-spa-conversion/templates/course-data.template.js` | `static-spa-conversion` |
| `index.html` | `static-spa-conversion/templates/index.html` | `static-spa-conversion` |
| Inline `<style>` block | `teaching-site-design-system/templates/tokens.css` (paste into `index.html`) | `teaching-site-design-system` |
| Per-day `outline.md` | `course-outline-design` (template embedded in SKILL.md) | `course-outline-design` |
| Per-day `content.md` | `course-content-authoring` (template embedded in SKILL.md) | `course-content-authoring` |
| `materials/README.md` | `course-content-authoring` (template embedded in SKILL.md) | `course-content-authoring` |

If a template diverges from this schema doc, **this doc wins** — fix the template and PR.

---

## 15. Production Reference

When in doubt, read the production implementation: **`d:/GitHub/ai-workshop/`** (4-day workshop, fully deployed). Specifically:

- `course-data.js` — full populated schema across 4 days
- `index.html:2080–4088` — render pipeline (`renderOverview`, `renderDay`, `renderUnit`, `renderConcept`, `renderPromptCard`, `renderTaskList`, `renderMaterialList`, `renderQuiz`)
- `index.html:2488` — `getMaterialUrl()` reference router
- `scripts/` — 30+ audit / build / capture / scrape scripts for production patterns

The reference is **read-only context** — copy patterns from it, but do not embed its course-specific content (brand names, prompts, materials) into your generated project.
