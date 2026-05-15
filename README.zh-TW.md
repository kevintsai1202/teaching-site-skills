# Teaching Site Skills

**Languages:** [English](README.md) · **繁體中文** · [简体中文](README.zh-CN.md) · [日本語](README.ja.md)

---

一套 **11 個 agent skills**（[Anthropic Skills 格式](https://code.claude.com/docs/en/skills)），捕捉「從零到一打造互動式教學網站」的完整工作流程。涵蓋：綱要設計 → 內容撰寫 → 純前端 SPA → 互動效果 → 視覺資產 → 企業包班濃縮 → 電子書產出，加上三個橫切 skill：行為驗證、資料稽核、視覺設計系統。

支援 Claude Code、Codex、Antigravity（Gemini）、Cursor、OpenCode 等 [55+ AI coding agent](https://github.com/vercel-labs/skills)（透過 `skills` CLI 安裝）。原始開發與主測平台為 Claude Code，其他 agent 為相容支援。

源自一個四天工作坊網站（行政與財務 AI 自動化）的實作經驗，但每個 skill 都已抽象到通用層級——可套用到任何「分章節 / 多單元 / 含素材」的教學網站。

---

## 結構：1 個總入口 + 10 個子 skill

安裝後展開於各 agent 的 skills 目錄（Claude Code 為 `~/.claude/skills/`、Codex 為 `~/.codex/skills/`、Antigravity 為 `~/.gemini/antigravity/skills/`，其餘參見 [skills CLI 對照表](https://github.com/vercel-labs/skills)）：

```text
<agent-skills-dir>/
├── teaching-site/                          ← ⭐ 總入口（super-skill with references/）
│   ├── SKILL.md                            ←   架構 + dispatch 邏輯
│   └── references/                         ←   依需要載入的補充
│       ├── consistency-checklists.md       ←   各 stage 後的連動檢查
│       ├── scenarios.md                    ←   常見 prompt → 流程
│       ├── troubleshooting.md              ←   故障排查
│       └── design-rationale.md             ←   為什麼這樣拆 skill
│
├── course-outline-design/                  ← Stage 1 課程綱要
├── course-content-authoring/               ← Stage 2 內容撰寫
├── static-spa-conversion/                  ← Stage 3 SPA 轉換
├── static-spa-interactions/                ← Stage 4 互動效果
├── web-visual-assets/                      ← Stage 5 視覺資產
├── course-corporate-edition/               ← Stage 5b 企業包班分支
├── course-ebook-publishing/                ← Stage 6 電子書產出
│
├── web-visual-verification/                ← 橫切：行為驗證（Playwright）
├── web-content-audit/                      ← 橫切：資料稽核
└── teaching-site-design-system/            ← 橫切：色票、字體、元件視覺
```

**兩種觸發方式並存**：

- 對 `teaching-site` 喊「做課程網站」→ 走總入口、由它判斷該 dispatch 哪個子 skill
- 對子 skill 喊「做電子書」→ 直接觸發 `course-ebook-publishing`、跳過 orchestration

---

## 安裝

前提：已安裝任一支援的 AI coding agent（Claude Code、Codex、Antigravity、Cursor、OpenCode 等）與 Node.js（`npx` 隨附）。

透過 [`skills` CLI](https://github.com/vercel-labs/skills) 一鍵安裝（Windows / macOS / Linux 共用同一條指令）：

```bash
npx skills add kevintsai1202/teaching-site-skills --all
```

CLI 會**自動偵測**機器上已安裝的 agent 並裝到對應的 skills 目錄。安裝完重新啟動該 agent（或執行其 reload 指令，如 Claude Code 的 `/reload`），11 個 skill 自動載入。

> **重要**：11 個 skill 互相引用，請務必加 `--all` 一次裝齊；單獨抽裝會在 dispatch 時找不到下游 skill。

**指定特定 agent**（沒被自動偵測到、或想裝到不同 agent 時）：

```bash
# 只裝到 Claude Code  →  ~/.claude/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent claude-code

# 只裝到 Codex         →  ~/.codex/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent codex

# 只裝到 Antigravity   →  ~/.gemini/antigravity/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent antigravity

# 一次裝到全部 55+ 支援的 agent
npx skills add kevintsai1202/teaching-site-skills --all --agent '*'
```

**其他常用指令**：

```bash
# 先列出 repo 內可裝的 skill（不安裝）
npx skills add kevintsai1202/teaching-site-skills --list

# 只裝特定幾個 skill（不建議，會打斷 pipeline）
npx skills add kevintsai1202/teaching-site-skills --skill teaching-site --skill course-outline-design
```

> **跨 agent 注意**：本套 skill 在 Claude Code 上實測過完整 4-day workshop pipeline。其他 agent 為靜態相容支援——CLI 會正確安裝、frontmatter 格式相容，但 dispatch 體驗、trigger phrase 命中率、子 skill 自動發現等行為依各 agent 的 skill 匹配演算法而異，建議首次使用時跑一次 `做課程網站` happy path 驗證。

---

## Pipeline 圖

```text
Stage 1  course-outline-design          ← 課程綱要（天數 / 單元 / 學習目標）
   ↓
Stage 2  course-content-authoring       ← 講義 / 素材 / 測驗題
   ↓
Stage 3  static-spa-conversion          ← course-data.js + index.html
   ↓
Stage 4  static-spa-interactions        ← 進度 / sidebar / 響應式 / 主題
   ↓
Stage 5  web-visual-assets              ← 插圖 / 截圖 / QR / 地圖
   ↓
   ├─→ Stage 5b  course-corporate-edition  ← 企業包班濃縮版（單檔交付）
   └─→ Stage 6   course-ebook-publishing   ← PDF + DOCX 電子書
```

**橫切（任何階段都可呼叫）**：

| Skill | 問題 | 失敗模式 |
| --- | --- | --- |
| `web-visual-verification` | 會動嗎？看起來對嗎？ | exit 非零，CI 阻擋 |
| `web-content-audit` | 對得起來嗎？該補什麼？ | 永遠 exit 0，產報告 |
| `teaching-site-design-system` | 該長怎樣？為什麼？ | 不執行，是設計參考 |

**協調者**：`teaching-site` 判斷使用者在哪階段、dispatch 子 skill。

---

## 觸發詞速查表

| Skill | 中文觸發詞 | 英文觸發詞 |
| --- | --- | --- |
| ⭐ `teaching-site` | 做課程網站、教學網頁、工作坊網站、做一套課程 | course microsite, workshop site |
| `course-outline-design` | 規劃課程、課程大綱、學習目標 | syllabus design, course outline |
| `course-content-authoring` | 寫講義、補素材、出測驗題、提示詞範本 | lecture notes, course material |
| `static-spa-conversion` | 做成網頁、轉成 SPA、course-data.js | static site from markdown, vanilla JS site |
| `static-spa-interactions` | 加進度勾選、響應式、暗色模式、scrollspy | RWD, dark mode, progress tracking |
| `web-visual-assets` | 插圖、工具截圖、QR、講師卡、地圖 | illustrations, screenshots, QR codes |
| `course-corporate-edition` | 企業包班、濃縮版、客製化、壓縮成一天 | corporate edition, intensive version |
| `course-ebook-publishing` | 做電子書、產 PDF、印給學員 | course handbook, PDF ebook, DOCX deliverable |
| `web-visual-verification` | 驗證網頁、RWD 驗證、截圖比對 | Playwright tests, visual regression |
| `web-content-audit` | 盤點內容、稽核資產、找缺圖、三處同步檢查 | content audit, asset coverage |
| `teaching-site-design-system` | 視覺風格、設計系統、色票、字體、玻璃卡片 | design tokens, color system |

---

## 常見使用情境

**「幫我從零做一個 4 天工作坊網站」**
→ `teaching-site` 總入口接手，依序 dispatch outline → content → SPA → interactions → visuals。

**「我有講義 .md 了，幫我做成網頁」**
→ 直接 `static-spa-conversion`（跳過 stage 1–2，從既有 markdown 萃取結構）。

**「網站做完了，幫我印一份 PDF 給學員」**
→ 確認網站穩定後直接呼叫 `course-ebook-publishing`。

**「幫我把公開班濃縮成企業包班 6 小時版」**
→ 直接呼叫 `course-corporate-edition`，輸出單檔 `index.html`（內嵌 COURSE）+ 客戶可 zip 的資料夾。

**「網站可以動了，但手機版很爛」**
→ 直接 `static-spa-interactions`，完成後呼叫 `web-visual-verification` 多 viewport 驗證。

**「素材連結 404」「測驗分數顯示錯」「進度勾選跑掉」**
→ 幾乎都是「三處同步」沒做完。呼叫 `web-content-audit` 對照 filesystem / course-data.js / getMaterialUrl() 三處。

---

## 關鍵設計決策（為什麼這麼拆）

1. **總入口 + 子 skill 並存**：寬廣請求走 `teaching-site` orchestration；具體請求直接觸發子 skill。兩種觸發路徑都通。
2. **核心 pipeline + 橫切兩層架構**：核心 7 個有先後關係（stage 1–6），橫切 3 個任何階段都可呼叫。
3. **連動規則放在子 skill 內**：例如「動到測驗題就同步 5 處硬編碼」放在 content-authoring，符合內聚。
4. **驗證與稽核必須分開兩個 skill**：失敗模式不同（verify 阻擋 / audit 不阻擋）、跑的時機不同（每次 PR / 釋出里程碑）。
5. **設計系統的「為什麼」也寫進 skill**：值會變（OKLCH 色票會調），但理由通常不會。
6. **企業包班用反向設計**：`window.COURSE` 內嵌 HTML、資產 fallback chain（客製版 → 主版）、電子書 builder 用 `vm.runInContext` 反向抽出 COURSE。
7. **電子書 PDF 用 Playwright `page.pdf()` 而非 Chrome CLI**：因為 CLI 不支援 `footerTemplate`（沒頁碼）。
8. **localStorage 鍵的 task-id 永遠不能改名**：published 後重新命名會讓所有學員勾選進度錯位。

詳細理由見 `teaching-site/references/design-rationale.md`。

---

## 來源與授權

原始實作：[ai-workshop](https://github.com/kevintsai1202/ai-workshop)（行政與財務 AI 自動化 4 天工作坊）。

Skills 由 Claude Opus 4.7（with 1M context）在 2026-05-14 從專案歷史中萃取、抽象、結構化而成。寫作過程讀取了 30 個 git commits、33 個 verify/diagnose/audit 腳本、`index.html`（3000+ 行）的 CSS / JS 區塊，以及 `企業包班/` 子資料夾。

各 skill 為獨立 markdown，使用、修改、分發皆無限制。如果改寫得更好，歡迎 PR 回原 repo。
