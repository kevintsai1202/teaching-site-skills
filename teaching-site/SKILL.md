---
name: teaching-site
description: Use this skill as the main entry point whenever the user wants to build, plan, or evolve an interactive teaching website / course microsite / workshop landing page — from a blank slate, from existing course materials, or any state in between. Triggers on broad phrases like "做課程網站", "做教學網頁", "做工作坊網站", "把講義變網頁", "course microsite", "workshop site", "interactive lesson page", "multi-day curriculum website", "做一套課程". This skill is the router for the whole teaching-site pipeline (outline → content → SPA → interactions → visuals → corporate / ebook) and dispatches to the 10 specialised sub-skills as needed. Prefer this when the user's request is broad or unclear about which stage they're at — sub-skills (e.g. `course-ebook-publishing`) are still triggerable directly for stage-specific requests.
---

# Teaching Site — Main Entry Point

This is the **top-level skill** for producing an interactive teaching website. It coordinates 10 specialised sub-skills covering every stage from blank-slate outline to delivered PDF ebook.

## When This Skill vs. a Sub-Skill

- **Use this skill** (`teaching-site`) when:
  - The user's request is broad ("做一套課程網站", "我想做工作坊").
  - You're starting from scratch and don't know where to begin.
  - The user is jumping between stages and needs orchestration.
  - You need to check cross-artifact consistency after a change.

- **Use a sub-skill directly** when:
  - The user names the artifact ("做電子書" → `course-ebook-publishing`).
  - You're already deep into one stage and don't need pipeline overview.
  - Another skill is dispatching here for a focused subtask.

Both paths converge — sub-skills can be reached either by direct trigger or via this skill's dispatch.

## The Production Pipeline

A teaching site is built in five **core** layers, with two optional **derivative** layers at the end:

```
[Stage 1] outline (.md)
    ↓ produces: meta, day list, unit skeleton, learning goals, shared scenario
[Stage 2] content authoring (.md / .csv / .yaml)
    ↓ produces: lecture notes, sample materials, quiz items, optional prompt templates
[Stage 3] SPA conversion (.js + .html)
    ↓ produces: course-data.js (data) + index.html (renderers), local serve setup
[Stage 4] interactions (.js + .css inside index.html)
    ↓ produces: progress persistence, sidebar/scrollspy, theme, RWD, quiz UX
[Stage 5] visual assets (.png / .svg / scraping scripts)
    ↓ produces: illustrations, screenshots, QR codes, maps
    ▼
─── site is now feature-complete ─── (optional derivatives below)
    ▼
[Stage 5b] Corporate Edition branch   ← optional, parallel to ebook
    ↓ produces: 企業包班/ folder, condensed units, inlined COURSE, asset fallback chain
[Stage 6]  Ebook Publishing            ← optional, ALWAYS after site is stable
    ↓ produces: dist/{name}.pdf + .docx via single composed master.md
```

Stages 1–5 are usually traversed in order, but users often jump back (add a unit → must update outline AND course-data.js AND any related visual). Catching these back-references is this skill's main value-add.

**Stages 5b and 6 are downstream consumers** of the finished site. They read but never modify the canonical site. Do not invoke them while Stages 1–5 are still in flux — the rework cost is high.

## Sub-Skill Dispatch Table

Invoke the matching sub-skill via your agent's skill activation mechanism (Claude Code: `Skill` tool, Codex: `skill` tool, Antigravity / Gemini CLI: `activate_skill`) when the matching stage is active:

| Stage | Sub-skill | Trigger phrases |
|---|---|---|
| 1 | `course-outline-design` | "規劃課程", "課程大綱", "幾天怎麼排", "學習目標" |
| 2 | `course-content-authoring` | "寫講義", "補素材", "出測驗題", "提示詞範本" |
| 3 | `static-spa-conversion` | "做成網頁", "轉成 SPA", "course-data.js", "render 函式" |
| 4 | `static-spa-interactions` | "加進度勾選", "響應式", "暗色模式", "縮放", "scrollspy" |
| 5 | `web-visual-assets` | "插圖", "工具截圖", "QR", "講師卡", "地圖" |
| 5b | `course-corporate-edition` | "企業包班", "濃縮版", "客製化", "壓縮成一天" |
| 6 | `course-ebook-publishing` | "做電子書", "產 PDF", "印給學員", "DOCX 交付" |
| ╳ | `web-visual-verification` | "驗證網頁", "RWD 驗證", "Playwright 測試", "截圖比對" |
| ╳ | `web-content-audit` | "盤點內容", "稽核資產", "找缺圖", "三處同步檢查" |
| ╳ | `teaching-site-design-system` | "視覺風格", "設計系統", "色票", "字體", "玻璃卡片" |

The last three (marked ╳) are **cross-cutting** — not tied to a stage. The first two (verification / audit) are the safety net for runtime behaviour and cross-file references. The third (design-system) is the visual authority every stage reads from.

## Stage 1 Gate (Hard Rule — read before any dispatch)

Before dispatching to ANY stage (including 2–6, 5b, and the cross-cutting skills), confirm Stage 1 deliverables exist on disk:

- [ ] An overview file (e.g. `課程總覽.md`) with populated `對象` / `總時數` / `每日主題` fields — not just a heading.
- [ ] At least one per-day outline file (e.g. `Day1/課程大綱.md`) listing unit IDs and learning goals.

> Shared scenario (`共用案例設定.md`) is **optional** — it's a downstream decision handled by `course-outline-design`'s Completion Gate, not an entry requirement here.

**If either of the two is missing or only a stub:** do NOT dispatch downstream, even if the user explicitly named a later stage ("做電子書", "幫我寫 quiz"). Dispatch to `course-outline-design` first and tell the user:

> 「我看到還沒有完整的課程綱要 — 直接跳到 Stage N 會讓後面每改一次 outline 就連動多個檔案重做。先用 `course-outline-design` 把 outline 鎖定，大概 10 分鐘決策，省下後續數小時 rework。」

### Override Policy (lenient with audit trail)

If the user explicitly insists on skipping the gate ("我知道，先做就好", "skip the outline, just do X"), proceed under these conditions:

1. Ask them to paste **3 bullets in chat** before any downstream action:
   - **對象**：一句話描述學員是誰 + 先備知識
   - **總時數 + 每日時段**：例如「2 天 × 6 小時」
   - **每日主題**：Day 1 / Day 2 / ... 各一行
2. Save those 3 bullets **verbatim** into a stub `課程總覽.md` (mark it `<!-- stub created via Stage 1 Gate override on YYYY-MM-DD -->`) before dispatching to the requested stage.
3. If the user refuses even the 3-bullet stub, fall back to the strict path — refuse the override and dispatch to `course-outline-design`.

The stub is the audit trail: future sessions reading this site can immediately see Stage 1 was bypassed and recover context.

## How to Detect the Current Stage

After the Stage 1 Gate passes, look for these signals to pick the right downstream stage:

- **Outline `.md` exists, but no `course-data.js`** → Stage 2 (content), or skip to 3 if user only wants a thin demo.
- **`course-data.js` exists, but `index.html` has no renderers / no local serve** → Stage 3.
- **SPA renders, but progress isn't persisted / no responsive / no theme** → Stage 4.
- **Site works but has placeholder `<div>` for images / 404 thumbnails** → Stage 5.
- **Site is feature-complete, user mentions corporate / in-house / shorter version** → Stage 5b.
- **Site is feature-complete, user wants a printed / archived / shareable file** → Stage 6.

Ambiguous? Ask one short question, then dispatch. Don't try to do all stages at once — the cross-stage rework cost is high.

## Operating Principles

1. **Never skip stages forward.** Don't generate Stage 3 SPA from a half-baked Stage 1 outline — the structural mismatches multiply. If the user wants to skip, say so explicitly.

2. **Always trace back-references.** When the user modifies a downstream artifact (e.g. "add unit u-6 to day 2"), update upstream (`課程大綱.md`) AND downstream (`course-data.js`, related visuals) in the same change. Half-updated sites accumulate technical debt very fast.

3. **Course-specific knowledge belongs to the user's content, not to a skill.** The sub-skills are intentionally agnostic to the *topic* of the course (AI workshop / cooking class / accounting training — same pipeline). If a sub-skill seems to be hardcoding domain examples, that's a smell.

4. **Verification commands are part of the deliverable.** When dispatching to Stage 3+, remind the user how to verify (`npm run serve`, then visit `http://localhost:3000`). Static sites often fail silently when opened with `file://` (localStorage blocked).

## Reference Files (Read When Needed)

When you need deeper detail than this entry point provides, read the appropriate reference:

- **`references/consistency-checklists.md`** — Read after every stage transition. Contains the full per-stage cross-artifact checklist (what to verify after Stage 1, 2, 3, ... 6, 5b). Catches "I changed X but forgot to update Y" bugs.

- **`references/scenarios.md`** — Read when the user's intent is ambiguous. Contains a dozen typical user prompts ("我有講義 .md 了", "客戶要濃縮版 + PDF") mapped to the exact sub-skill dispatch flow.

- **`references/troubleshooting.md`** — Read when something is reported broken. Maps symptoms ("素材連結 404", "進度勾選跑掉", "PDF 沒頁碼") to the most likely sub-skill / audit / verification path.

- **`references/design-rationale.md`** — Read when explaining the skill architecture itself, or when deciding whether a new feature deserves its own sub-skill vs. fitting into an existing one. Captures *why* the pipeline is shaped this way.

## What This Skill Does NOT Do

- Does not write any code itself — always dispatches to a sub-skill.
- Does not pick a CSS framework or build tool — assumes vanilla HTML/JS by default (see `static-spa-conversion` for why).
- Does not decide the course topic, audience, or pedagogy — that's the user's domain.
