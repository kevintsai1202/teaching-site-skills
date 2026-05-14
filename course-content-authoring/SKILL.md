---
name: course-content-authoring
description: Use this skill when an outline exists and you need to fill in the actual teaching content — lecture notes, sample materials (CSV / YAML / MD samples), quiz items, and any course-specific artifacts like prompt templates or worksheets. Triggers on phrases like "寫講義", "補素材", "出測驗題", "提示詞範本", "撰寫課程內容", "lecture notes", "course material", "quiz questions". Always invoke AFTER `course-outline-design` is stable, BEFORE `static-spa-conversion`. This skill produces only the raw `.md`/`.csv`/`.yaml` artifacts — turning them into a website is the next stage's job.
---

# Course Content Authoring

This skill produces the **teaching substance** of a course: the things a learner reads, copies, listens to, or works through. It anchors every artifact to the outline IDs from `course-outline-design`, so the next stage (SPA conversion) can wire them up mechanically.

## Deliverables (Standard Layout)

```
完整課程包/   (your project's content root)
├── Day{n}/
│   ├── 課程大綱.md           ← from outline stage (do NOT edit here)
│   └── 課程內容.md           ← THIS skill writes: lecture script, prompts, exercises, references
├── 共用案例設定.md            ← from outline stage
├── 課程輔助文件.md            ← THIS skill writes: FAQ, environment setup, pre-reading
└── 教學素材/                  ← THIS skill writes: standalone learner artifacts
    ├── README.md             ← index of materials
    └── *.md, *.csv, *.yaml   ← samples, templates, datasets
```

## Material Types (Pick What Fits Your Course)

The example workshop uses these — they're **examples**, not requirements. Decide per course what makes sense:

| Type | Format | When to use |
|---|---|---|
| Lecture script | `.md` | Per-day teaching narrative, embedded in `課程內容.md` |
| Sample document | `.md` | A "before" artifact learners will improve (e.g. an FAQ, a policy doc) |
| Tabular dataset | `.csv` (UTF-8 BOM!) | Practice data for AI processing exercises |
| Structured config | `.yaml` | When the exercise involves declarative configuration |
| PDF reference | `.pdf` | Official documents (laws, regulations) — usually external sources |
| Prompt template | `.md` block | Domain-specific reusable prompts (this course uses **RTFC** framework, others may not — adapt to your course's pedagogy) |
| Quiz items | inline in `課程內容.md` or separate | Pre-test, post-test, comprehension checks |
| Worksheets | `.md` | Fill-in templates learners complete |

> **About prompt templates**: This skill is **agnostic** to which prompt framework you use. RTFC (Role / Task / Format / Context) was used in the original workshop, but if your course teaches a different framework (e.g. CRISPE, COSTAR, plain examples), use that. Don't force RTFC into a course where it doesn't fit. The framework is *part of the course content*, not a property of this skill.

## CSV Trap (Easy to Miss)

When generating CSV files that will be opened in Excel by non-developer learners, **prepend a UTF-8 BOM** (`﻿`):

```js
const content = '﻿' + headerRow + '\n' + dataRows.join('\n');
```

Without it, Excel on Windows mis-detects encoding and shows mojibake. Learners will think the file is broken. This bug was found late in the example workshop — bake it in from the start.

## Anchor Everything to Outline IDs

Every artifact you produce must trace to a unit ID. Use this header pattern in lecture notes:

```markdown
## u-3: {Unit title from 課程大綱.md}

**對應任務**: d2-u3-t1, d2-u3-t2
**對應素材**: 客訴處理SOP.md, FAQ官方版.md

{lecture content here}
```

Why: the SPA conversion stage will read these markers to auto-link tasks, materials, and content. If you skip them, the SPA author has to re-read everything to figure out the wiring.

## Task IDs Are Forever

When writing task descriptions inside `課程內容.md`, assign each task a stable ID (e.g. `d2-u3-t1`). These IDs will become **localStorage keys** in the deployed website. Three rules:

1. **Never rename** a published task ID. Students' progress is keyed to it.
2. **Never reuse** a deleted task ID. If you remove a task, the ID retires permanently.
3. **Two formats coexist** in practice (`d{n}-u{m}-t{k}` and `day{n}-u{m}-t{k}`). Pick one for new content; don't mix.

## Quiz Item Discipline

If the course has a quiz, write items numbered sequentially (`q1`, `q2`, ...). Be aware these numbers will leak into multiple places once on the SPA:

- The quiz array itself
- A `qIndexToDay()`-style helper that maps each item to its source chapter (so wrong answers can link back)
- Hardcoded strings in the SPA: `"結訓測驗（N題）"`, score display `— / N`, passing threshold `s >= K`

**Renumber-safe authoring**: think of the quiz as **append-only**. If you delete `q3`, leave the slot empty in your draft and renumber only at the very last stable moment, with a checklist of all the places to update.

## Hidden Materials Pattern

Some materials are **for instructors only** (answer keys, pre/post-test scoring) or **conditional** (only shown if a feature flag is on). Mark them explicitly in `教學素材/README.md`:

```markdown
| 素材 | 對象 | 備註 |
|---|---|---|
| 員工差勤辦法.md | 學員 | Day 2 公開素材 |
| 結訓測驗解答.md | 講師 | 不要納入 SPA materials[]，僅講師端使用 |
```

The SPA conversion stage will read this column and exclude instructor-only items.

## Anti-Patterns

- **Writing content before outline is locked** — every back-edit cascades to multiple files.
- **Embedding prompt frameworks the course doesn't actually teach** — if your course doesn't use RTFC, don't force-fit it just because the example template has it.
- **Quiz items without source-chapter mapping** — when a learner gets one wrong, they need to know *where to review*. Tag every quiz item with the unit it came from.
- **Long flat material list with no instructor / learner distinction** — by Day 4 the instructor doesn't know which file is what.
- **Forgetting CSV BOM** — see above.

## Hand-off

When this stage finishes, you should have:
- Per-day `課程內容.md` filled in
- All material files in `教學素材/` with an index README
- Quiz draft (if applicable) numbered `q1..qN` with source-chapter tags
- Every artifact traceable to an outline unit ID

Tell the user: "content draft complete. Next stage (`static-spa-conversion`) will convert this into `course-data.js` + `index.html`. Don't change unit IDs or task IDs from this point — they'll become localStorage keys."
