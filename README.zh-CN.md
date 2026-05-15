# Teaching Site Skills

**Languages:** [English](README.md) · [繁體中文](README.zh-TW.md) · **简体中文** · [日本語](README.ja.md)

---

一套 **11 个 agent skills**（[Anthropic Skills 格式](https://code.claude.com/docs/en/skills)），捕捉「从零到一打造交互式教学网站」的完整工作流程。涵盖：大纲设计 → 内容撰写 → 纯前端 SPA → 交互效果 → 视觉资产 → 企业内训精简 → 电子书产出，外加三个横切 skill：行为验证、内容稽核、视觉设计系统。

支持 Claude Code、Codex、Antigravity（Gemini）、Cursor、OpenCode 等 [55+ AI coding agent](https://github.com/vercel-labs/skills)（通过 `skills` CLI 安装）。原始开发与主测平台为 Claude Code，其他 agent 为兼容支持。

源自一个四天工作坊网站（行政与财务 AI 自动化）的实战经验，但每个 skill 都已抽象到通用层级——可套用到任何「分章节 / 多单元 / 含素材」的教学网站。

---

## 结构：1 个总入口 + 10 个子 skill

安装后展开于各 agent 的 skills 目录（Claude Code 为 `~/.claude/skills/`、Codex 为 `~/.codex/skills/`、Antigravity 为 `~/.gemini/antigravity/skills/`，其余请参考 [skills CLI 对照表](https://github.com/vercel-labs/skills)）：

```text
<agent-skills-dir>/
├── teaching-site/                          ← ⭐ 总入口（super-skill with references/）
│   ├── SKILL.md                            ←   架构 + dispatch 逻辑
│   └── references/                         ←   按需加载的补充
│       ├── consistency-checklists.md       ←   各 stage 后的联动检查
│       ├── scenarios.md                    ←   常见 prompt → 流程
│       ├── troubleshooting.md              ←   故障排查
│       └── design-rationale.md             ←   为什么这样拆 skill
│
├── course-outline-design/                  ← Stage 1 课程大纲
├── course-content-authoring/               ← Stage 2 内容撰写
├── static-spa-conversion/                  ← Stage 3 SPA 转换
├── static-spa-interactions/                ← Stage 4 交互效果
├── web-visual-assets/                      ← Stage 5 视觉资产
├── course-corporate-edition/               ← Stage 5b 企业内训分支
├── course-ebook-publishing/                ← Stage 6 电子书产出
│
├── web-visual-verification/                ← 横切：行为验证（Playwright）
├── web-content-audit/                      ← 横切：内容稽核
└── teaching-site-design-system/            ← 横切：色板、字体、组件视觉
```

**两种触发方式并存**：

- 对 `teaching-site` 说「做课程网站」→ 走总入口，由它判断该 dispatch 哪个子 skill
- 对子 skill 说「做电子书」→ 直接触发 `course-ebook-publishing`，跳过 orchestration

---

## 安装

前提：已安装任一支持的 AI coding agent（Claude Code、Codex、Antigravity、Cursor、OpenCode 等）与 Node.js（`npx` 随附）。

通过 [`skills` CLI](https://github.com/vercel-labs/skills) 一键安装（Windows / macOS / Linux 共用同一条命令）：

```bash
npx skills add kevintsai1202/teaching-site-skills --all
```

CLI 会**自动检测**机器上已安装的 agent 并装到对应的 skills 目录。安装完重启该 agent（或执行其 reload 命令，如 Claude Code 的 `/reload`），11 个 skill 自动加载。

> **重要**：11 个 skill 互相引用，请务必加 `--all` 一次装齐；单独抽装会在 dispatch 时找不到下游 skill。

**指定特定 agent**（没被自动检测到、或想装到不同 agent 时）：

```bash
# 只装到 Claude Code  →  ~/.claude/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent claude-code

# 只装到 Codex         →  ~/.codex/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent codex

# 只装到 Antigravity   →  ~/.gemini/antigravity/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent antigravity

# 一次装到全部 55+ 支持的 agent
npx skills add kevintsai1202/teaching-site-skills --all --agent '*'
```

**其他常用命令**：

```bash
# 先列出 repo 内可装的 skill（不安装）
npx skills add kevintsai1202/teaching-site-skills --list

# 只装特定几个 skill（不推荐，会打断 pipeline）
npx skills add kevintsai1202/teaching-site-skills --skill teaching-site --skill course-outline-design
```

> **跨 agent 注意**：本套 skill 在 Claude Code 上实测过完整 4-day workshop pipeline。其他 agent 为静态兼容支持——CLI 会正确安装、frontmatter 格式兼容，但 dispatch 体验、trigger phrase 命中率、子 skill 自动发现等行为依各 agent 的 skill 匹配算法而异，建议首次使用时跑一次「做课程网站」happy path 验证。

---

## Pipeline 图

```text
Stage 1  course-outline-design          ← 课程大纲（天数 / 单元 / 学习目标）
   ↓
Stage 2  course-content-authoring       ← 讲义 / 素材 / 测验题
   ↓
Stage 3  static-spa-conversion          ← course-data.js + index.html
   ↓
Stage 4  static-spa-interactions        ← 进度 / sidebar / 响应式 / 主题
   ↓
Stage 5  web-visual-assets              ← 插图 / 截图 / 二维码 / 地图
   ↓
   ├─→ Stage 5b  course-corporate-edition  ← 企业内训精简版（单文件交付）
   └─→ Stage 6   course-ebook-publishing   ← PDF + DOCX 电子书
```

**横切（任何阶段都可调用）**：

| Skill | 问题 | 失败模式 |
| --- | --- | --- |
| `web-visual-verification` | 能动吗？看起来对吗？ | exit 非零，CI 阻挡 |
| `web-content-audit` | 对得上吗？该补什么？ | 永远 exit 0，出报告 |
| `teaching-site-design-system` | 该长什么样？为什么？ | 不执行，是设计参考 |

**协调者**：`teaching-site` 判断用户在哪阶段、dispatch 子 skill。

---

## 触发词速查表

| Skill | 中文触发词 | 英文触发词 |
| --- | --- | --- |
| ⭐ `teaching-site` | 做课程网站、教学网页、工作坊网站、做一套课程 | course microsite, workshop site |
| `course-outline-design` | 规划课程、课程大纲、学习目标 | syllabus design, course outline |
| `course-content-authoring` | 写讲义、补素材、出测验题、提示词模板 | lecture notes, course material |
| `static-spa-conversion` | 做成网页、转成 SPA、course-data.js | static site from markdown, vanilla JS site |
| `static-spa-interactions` | 加进度勾选、响应式、暗色模式、scrollspy | RWD, dark mode, progress tracking |
| `web-visual-assets` | 插图、工具截图、二维码、讲师卡、地图 | illustrations, screenshots, QR codes |
| `course-corporate-edition` | 企业内训、精简版、定制化、压缩成一天 | corporate edition, intensive version |
| `course-ebook-publishing` | 做电子书、生成 PDF、印给学员 | course handbook, PDF ebook, DOCX deliverable |
| `web-visual-verification` | 验证网页、RWD 验证、截图对比 | Playwright tests, visual regression |
| `web-content-audit` | 盘点内容、稽核资产、找缺图、三处同步检查 | content audit, asset coverage |
| `teaching-site-design-system` | 视觉风格、设计系统、色板、字体、玻璃卡片 | design tokens, color system |

---

## 常见使用场景

**「帮我从零做一个 4 天工作坊网站」**
→ `teaching-site` 总入口接手，依次 dispatch outline → content → SPA → interactions → visuals。

**「我已经有讲义 .md 了，帮我做成网页」**
→ 直接调用 `static-spa-conversion`（跳过 stage 1–2，从既有 markdown 提取结构）。

**「网站做完了，帮我打印一份 PDF 给学员」**
→ 确认网站稳定后直接调用 `course-ebook-publishing`。

**「帮我把公开班压缩成企业内训 6 小时版」**
→ 直接调用 `course-corporate-edition`，输出单文件 `index.html`（内嵌 COURSE）+ 客户可 zip 的文件夹。

**「网站能动了，但手机端很糟」**
→ 直接调用 `static-spa-interactions`，完成后调用 `web-visual-verification` 多 viewport 验证。

**「素材链接 404」「测验分数显示错」「进度勾选跑偏」**
→ 几乎都是「三处同步」没做完。调用 `web-content-audit` 对照 filesystem / course-data.js / `getMaterialUrl()` 三处。

---

## 关键设计决策（为什么这么拆）

1. **总入口 + 子 skill 并存**：宽泛请求走 `teaching-site` orchestration；具体请求直接触发子 skill。两种触发路径都通。
2. **核心 pipeline + 横切两层架构**：核心 7 个有先后关系（stage 1–6），横切 3 个任何阶段都可调用。
3. **联动规则放在子 skill 内**：例如「动到测验题就同步 5 处硬编码」放在 content-authoring，符合内聚。
4. **验证与稽核必须分开两个 skill**：失败模式不同（verify 阻挡 / audit 不阻挡）、跑的时机不同（每次 PR / 发布里程碑）。
5. **设计系统的「为什么」也写进 skill**：值会变（OKLCH 色板会调），但理由通常不会。
6. **企业内训用反向设计**：`window.COURSE` 内嵌 HTML、资产 fallback chain（定制版 → 主版）、电子书 builder 用 `vm.runInContext` 反向抽出 COURSE。
7. **电子书 PDF 用 Playwright `page.pdf()` 而非 Chrome CLI**：因为 CLI 不支持 `footerTemplate`（没页码）。
8. **localStorage 键的 task-id 永远不能改名**：published 后重命名会让所有学员的勾选进度错位。

详细理由见 `teaching-site/references/design-rationale.md`。

---

## 来源与许可

原始实现：[ai-workshop](https://github.com/kevintsai1202/ai-workshop)（行政与财务 AI 自动化 4 天工作坊）。

Skills 由 Claude Opus 4.7（with 1M context）在 2026-05-14 从项目历史中提取、抽象、结构化而成。撰写过程读取了 30 个 git commits、33 个 verify/diagnose/audit 脚本、`index.html`（3000+ 行）的 CSS / JS 区块，以及 `企业包班/` 子文件夹。

各 skill 为独立 markdown，使用、修改、分发皆无限制。如果改写得更好，欢迎 PR 回原 repo。
