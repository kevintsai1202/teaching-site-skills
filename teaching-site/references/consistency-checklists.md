# Cross-Artifact Consistency Checklists

The actual hard part of teaching site maintenance: every change ripples to other files. Run the appropriate checklist after each stage finishes.

## After Stage 1 (outline change)

- [ ] Day count / hours / schedule consistent in: `課程總覽.md`, each `Day{n}/課程大綱.md`, and (if Stage 3+ exists) `course-data.js:meta.days[]`
- [ ] Shared scenario references match across all days

## After Stage 2 (content change)

- [ ] New material file dropped into `教學素材/` folder → added to `course-data.js:materials[]` AND in the SPA's material URL router (`getMaterialUrl()`)
- [ ] Quiz item count change → updates BOTH `quiz[]` array AND every hardcoded "(N題)" / "— / N" / "passing score" string in the SPA (typically 5 places)
- [ ] Per-unit `materials[]` references match `course-data.js:materials[]` item IDs

## After Stage 3 (SPA wiring)

- [ ] Task IDs in `course-data.js:tasks[]` are **stable** (they're localStorage keys; renaming wipes student progress)
- [ ] Material router has fallback for unknown types
- [ ] `window.COURSE.day1/day2/day3/day4` keys match `meta.days[].id`
- [ ] Each `unit.id` is unique within its day

## After Stage 4 (interactivity)

- [ ] Sidebar collapse logic does NOT merge desktop and mobile into one class — they have **opposite default states** (see `static-spa-interactions`)
- [ ] Zoom variable only on `.content`, never on `html` (cascades multiplicatively)
- [ ] `prefers-color-scheme` NOT used as theme source of truth — only `[data-theme]` attribute
- [ ] localStorage key has a version suffix (e.g. `-v1`) so future schema changes are non-destructive

## After Stage 5 (visuals)

- [ ] Every image referenced by `course-data.js:illustration` actually exists on disk
- [ ] PNG-first + SVG fallback path is intact for AI-generated images
- [ ] All AI illustrations use the **same style prompt** (consistency across the course)
- [ ] No image embeds Chinese text inside the PNG (AI usually fails this — use SVG instead)

## After Stage 5b (Corporate Edition)

- [ ] Corporate folder is **self-contained** (no `../` paths except documented fallback chain)
- [ ] Inlined `window.COURSE` is consistent with the source it was condensed from (run `web-content-audit`)
- [ ] Asset fallback chain (corporate `assets/` → public `assets/`) resolves every reference
- [ ] Brand customisation variables (client name, scenario) updated everywhere, including the corporate `README.md`
- [ ] `index.html` opens via double-click (no `fetch()` of local JSON, no features requiring HTTP)

## After Stage 6 (Ebook)

- [ ] PDF cover has no page number (`@page :first { margin: 0 }` works in Playwright `page.pdf()`)
- [ ] Quiz / pre-test / post-test content is **excluded** from ebook (avoid leaking assessment)
- [ ] Every image in composed markdown resolved to a real file (no broken-image squares)
- [ ] If both PDF and DOCX produced, content is identical (same composed `master.md`)
- [ ] DOCX inner structure inspected (`inspect-docx-images.mjs` confirms expected image count)

## Universal Checklist (Run After Any Change)

Regardless of which stage was touched:

- [ ] `web-visual-verification` passes (UI doesn't break, no `console.error`)
- [ ] `web-content-audit` produces an empty / clean report (no orphaned references)
- [ ] Manually visit the site once at desktop + mobile viewport before claiming "done"

The audit doesn't block release, but the human must read the report and make an explicit go/no-go call. The verification gates release — if it fails, block.
