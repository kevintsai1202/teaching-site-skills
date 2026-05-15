# Teaching Site Skills

**Languages:** **English** · [繁體中文](README.zh-TW.md) · [简体中文](README.zh-CN.md) · [日本語](README.ja.md)

---

A set of **11 agent skills** ([Anthropic Skills format](https://code.claude.com/docs/en/skills)) that capture the full workflow of building an interactive teaching site from scratch. The pipeline covers: outline design → content authoring → vanilla-JS SPA → interactions → visual assets → corporate-edition compression → ebook delivery, plus three cross-cutting skills: behavioral verification, content audit, and a visual design system.

Supports Claude Code, Codex, Antigravity (Gemini), Cursor, OpenCode, and [55+ AI coding agents](https://github.com/vercel-labs/skills) — installed via the `skills` CLI. Claude Code is the primary development and test platform; other agents are statically compatible.

Distilled from a four-day workshop site (admin & finance AI automation), but every skill has been abstracted to a generic level — applicable to any "chaptered / multi-unit / with-materials" teaching site.

---

## Structure: 1 super-skill + 10 sub-skills

After installation, the skills are unpacked into each agent's skills directory (Claude Code: `~/.claude/skills/`; Codex: `~/.codex/skills/`; Antigravity: `~/.gemini/antigravity/skills/`; others — see the [skills CLI matrix](https://github.com/vercel-labs/skills)):

```text
<agent-skills-dir>/
├── teaching-site/                          ← ⭐ super-skill entry (with references/)
│   ├── SKILL.md                            ←   architecture + dispatch logic
│   └── references/                         ←   load-on-demand supplements
│       ├── consistency-checklists.md       ←   post-stage cross-file checks
│       ├── scenarios.md                    ←   common prompt → flow mapping
│       ├── troubleshooting.md              ←   failure-mode triage
│       └── design-rationale.md             ←   why the skills are split this way
│
├── course-outline-design/                  ← Stage 1  course outline
├── course-content-authoring/               ← Stage 2  content authoring
├── static-spa-conversion/                  ← Stage 3  SPA conversion
├── static-spa-interactions/                ← Stage 4  interactions
├── web-visual-assets/                      ← Stage 5  visual assets
├── course-corporate-edition/               ← Stage 5b corporate edition branch
├── course-ebook-publishing/                ← Stage 6  ebook delivery
│
├── web-visual-verification/                ← cross-cutting: behavioral verification (Playwright)
├── web-content-audit/                      ← cross-cutting: content audit
└── teaching-site-design-system/            ← cross-cutting: tokens, fonts, component visuals
```

**Two trigger paths coexist:**

- Address `teaching-site` with "build me a course site" → super-skill takes over, decides which sub-skill to dispatch.
- Address a sub-skill directly with "make an ebook" → triggers `course-ebook-publishing`, skipping orchestration.

---

## Installation

Prerequisites: any supported AI coding agent (Claude Code, Codex, Antigravity, Cursor, OpenCode, …) and Node.js (`npx` ships with it).

One-line install via the [`skills` CLI](https://github.com/vercel-labs/skills) (same command on Windows / macOS / Linux):

```bash
npx skills add kevintsai1202/teaching-site-skills --all
```

The CLI **auto-detects** which agents are installed on your machine and writes to the corresponding skills directories. Restart the agent (or run its reload command — e.g. Claude Code's `/reload`) and all 11 skills load automatically.

> **Important:** the 11 skills cross-reference each other. Always pass `--all` to install them together; cherry-picking will break dispatch at runtime when a downstream skill is missing.

**Targeting a specific agent** (when auto-detection misses it, or you want to install only to one):

```bash
# Claude Code only   →  ~/.claude/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent claude-code

# Codex only         →  ~/.codex/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent codex

# Antigravity only   →  ~/.gemini/antigravity/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent antigravity

# Install to every supported agent (55+) at once
npx skills add kevintsai1202/teaching-site-skills --all --agent '*'
```

**Other useful commands:**

```bash
# List installable skills in the repo without installing
npx skills add kevintsai1202/teaching-site-skills --list

# Install only selected skills (not recommended — breaks the pipeline)
npx skills add kevintsai1202/teaching-site-skills --skill teaching-site --skill course-outline-design
```

> **Cross-agent note:** this skill set has been end-to-end tested on Claude Code with a full 4-day workshop pipeline. Other agents are statically compatible — the CLI installs correctly and the frontmatter format is portable — but dispatch UX, trigger-phrase hit rate, and sub-skill auto-discovery vary across each agent's skill-matching implementation. On first use with a new agent, run a "build me a course site" happy-path to verify.

---

## Pipeline diagram

```text
Stage 1  course-outline-design          ← outline (days / units / learning goals)
   ↓
Stage 2  course-content-authoring       ← lecture notes / materials / quizzes
   ↓
Stage 3  static-spa-conversion          ← course-data.js + index.html
   ↓
Stage 4  static-spa-interactions        ← progress / sidebar / RWD / theming
   ↓
Stage 5  web-visual-assets              ← illustrations / screenshots / QR / maps
   ↓
   ├─→ Stage 5b  course-corporate-edition  ← compressed corporate edition (single file)
   └─→ Stage 6   course-ebook-publishing   ← PDF + DOCX ebook
```

**Cross-cutting (callable from any stage):**

| Skill | The question it answers | Failure mode |
| --- | --- | --- |
| `web-visual-verification` | Does it work? Does it look right? | Non-zero exit, blocks CI |
| `web-content-audit` | Is the data consistent? What's missing? | Always exit 0, emits a report |
| `teaching-site-design-system` | How should it look, and why? | Not executable — a design reference |

**Coordinator:** `teaching-site` figures out which stage the user is at and dispatches the right sub-skill.

---

## Trigger-phrase quick reference

| Skill | English trigger phrases | 中文觸發詞 |
| --- | --- | --- |
| ⭐ `teaching-site` | course microsite, workshop site, build a teaching site | 做課程網站、教學網頁、工作坊網站、做一套課程 |
| `course-outline-design` | syllabus design, course outline, learning objectives | 規劃課程、課程大綱、學習目標 |
| `course-content-authoring` | lecture notes, course material, quiz authoring, prompt templates | 寫講義、補素材、出測驗題、提示詞範本 |
| `static-spa-conversion` | static site from markdown, vanilla JS site, course-data.js | 做成網頁、轉成 SPA、course-data.js |
| `static-spa-interactions` | RWD, dark mode, progress tracking, scrollspy | 加進度勾選、響應式、暗色模式、scrollspy |
| `web-visual-assets` | illustrations, screenshots, QR codes, instructor cards | 插圖、工具截圖、QR、講師卡、地圖 |
| `course-corporate-edition` | corporate edition, intensive version, single-file deliverable | 企業包班、濃縮版、客製化、壓縮成一天 |
| `course-ebook-publishing` | course handbook, PDF ebook, DOCX deliverable | 做電子書、產 PDF、印給學員 |
| `web-visual-verification` | Playwright tests, visual regression, RWD verification | 驗證網頁、RWD 驗證、截圖比對 |
| `web-content-audit` | content audit, asset coverage, three-way sync check | 盤點內容、稽核資產、找缺圖、三處同步檢查 |
| `teaching-site-design-system` | design tokens, color system, glass cards, typography | 視覺風格、設計系統、色票、字體、玻璃卡片 |

---

## Common scenarios

**"Build me a 4-day workshop site from scratch."**
→ `teaching-site` takes the lead and dispatches outline → content → SPA → interactions → visuals in order.

**"I already have lecture notes in .md — turn them into a site."**
→ Jump straight to `static-spa-conversion` (skipping stages 1–2; structure is extracted from the existing markdown).

**"The site is done — print me a PDF for the attendees."**
→ Confirm the site is stable, then call `course-ebook-publishing` directly.

**"Compress this open-class into a 6-hour corporate edition."**
→ Call `course-corporate-edition` directly. Output is a single `index.html` (with embedded COURSE) plus a folder the client can zip.

**"The site works, but the mobile view is terrible."**
→ Go directly to `static-spa-interactions`, then run `web-visual-verification` across viewports.

**"Material link 404s," "quiz score shows wrong," "progress check stops syncing."**
→ Almost always a missed "three-way sync." Call `web-content-audit` to diff filesystem / course-data.js / `getMaterialUrl()`.

---

## Key design decisions (why the skills are split this way)

1. **Super-skill + sub-skills coexist:** broad requests go through `teaching-site` orchestration; specific requests trigger sub-skills directly. Both trigger paths work.
2. **Two-layer architecture — core pipeline + cross-cutting:** the 7 core skills are sequential (stages 1–6); the 3 cross-cutting skills are callable from any stage.
3. **Cross-file invariants live inside the sub-skill that owns them:** e.g. "touching a quiz means syncing 5 hardcoded places" lives in content-authoring — co-located with the work that triggers it.
4. **Verification and audit are deliberately split into two skills:** different failure modes (verify blocks CI / audit always passes) and different cadences (every PR / release milestones).
5. **The design system encodes the "why," not just the "what":** values change (OKLCH tokens get tuned), but the reasoning usually doesn't.
6. **The corporate edition is reverse-engineered for delivery:** `window.COURSE` embedded inline, an asset fallback chain (custom edition → main edition), and the ebook builder uses `vm.runInContext` to extract COURSE back out.
7. **Ebook PDFs use Playwright `page.pdf()`, not Chrome CLI:** because the CLI doesn't support `footerTemplate` (no page numbers).
8. **localStorage task-id keys must never be renamed:** renaming after publish desyncs every attendee's progress checkboxes.

Full reasoning lives in `teaching-site/references/design-rationale.md`.

---

## Origin & license

Original implementation: [ai-workshop](https://github.com/kevintsai1202/ai-workshop) — a 4-day workshop on admin & finance AI automation.

Skills were distilled, abstracted, and structured by Claude Opus 4.7 (with 1M context) on 2026-05-14 from the project's history. The authoring pass read 30 git commits, 33 verify/diagnose/audit scripts, the CSS/JS sections of `index.html` (3000+ lines), and the `企業包班/` (corporate-edition) sub-directory.

Each skill is standalone markdown — no restrictions on use, modification, or redistribution. If you improve them, PRs back to the original repo are welcome.
