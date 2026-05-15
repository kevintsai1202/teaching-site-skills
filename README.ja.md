# Teaching Site Skills

**Languages:** [English](README.md) · [繁體中文](README.zh-TW.md) · [简体中文](README.zh-CN.md) · **日本語**

---

ゼロからインタラクティブな教材サイトを構築するワークフロー全体を捉えた、**11 個の agent skills**（[Anthropic Skills フォーマット](https://code.claude.com/docs/en/skills)）のセットです。パイプラインは、アウトライン設計 → コンテンツ執筆 → バニラ JS の SPA → インタラクション → ビジュアル素材 → 企業研修向け圧縮版 → 電子書籍納品 をカバーし、加えて 3 つの横断的 skill（動作検証、コンテンツ監査、ビジュアルデザインシステム）を含みます。

Claude Code、Codex、Antigravity（Gemini）、Cursor、OpenCode などの [55+ の AI コーディングエージェント](https://github.com/vercel-labs/skills) に対応（`skills` CLI でインストール）。主たる開発・テストプラットフォームは Claude Code で、他のエージェントは静的互換としてサポートされます。

4 日間のワークショップサイト（管理・財務領域の AI 自動化）の実装経験から抽出したものですが、各 skill は汎用レベルまで抽象化されており、「章立て / 複数ユニット / 素材付き」の教材サイトであればどんな案件にも適用できます。

---

## 構成：1 つの super-skill + 10 個のサブ skill

インストール後、各エージェントの skills ディレクトリに展開されます（Claude Code: `~/.claude/skills/`、Codex: `~/.codex/skills/`、Antigravity: `~/.gemini/antigravity/skills/`、その他は [skills CLI 対応表](https://github.com/vercel-labs/skills) を参照）：

```text
<agent-skills-dir>/
├── teaching-site/                          ← ⭐ super-skill エントリ（references/ 付き）
│   ├── SKILL.md                            ←   アーキテクチャ + dispatch ロジック
│   └── references/                         ←   必要に応じてロードされる補足資料
│       ├── consistency-checklists.md       ←   各 stage 後のファイル間整合性チェック
│       ├── scenarios.md                    ←   よくあるプロンプト → フローのマッピング
│       ├── troubleshooting.md              ←   故障モードの切り分け
│       └── design-rationale.md             ←   この skill 分割を採用した理由
│
├── course-outline-design/                  ← Stage 1  コースアウトライン
├── course-content-authoring/               ← Stage 2  コンテンツ執筆
├── static-spa-conversion/                  ← Stage 3  SPA 変換
├── static-spa-interactions/                ← Stage 4  インタラクション
├── web-visual-assets/                      ← Stage 5  ビジュアル素材
├── course-corporate-edition/               ← Stage 5b 企業研修版分岐
├── course-ebook-publishing/                ← Stage 6  電子書籍納品
│
├── web-visual-verification/                ← 横断：動作検証（Playwright）
├── web-content-audit/                      ← 横断：コンテンツ監査
└── teaching-site-design-system/            ← 横断：トークン、フォント、コンポーネントビジュアル
```

**2 つのトリガー経路が共存**：

- `teaching-site` に「教材サイトを作って」と話しかけると → super-skill が引き受け、どのサブ skill に dispatch するかを判断します。
- サブ skill に直接「電子書籍を作って」と話しかけると → `course-ebook-publishing` が直接トリガーされ、オーケストレーションは省略されます。

---

## インストール

前提：対応する AI コーディングエージェント（Claude Code、Codex、Antigravity、Cursor、OpenCode など）と Node.js（`npx` が同梱）がインストール済みであること。

[`skills` CLI](https://github.com/vercel-labs/skills) で 1 行インストール（Windows / macOS / Linux で同じコマンド）：

```bash
npx skills add kevintsai1202/teaching-site-skills --all
```

CLI はマシン上にインストールされているエージェントを**自動検出**し、対応する skills ディレクトリへ書き込みます。インストール後にエージェントを再起動（または Claude Code の `/reload` など、各エージェントのリロードコマンドを実行）すれば、11 個の skill が自動的にロードされます。

> **重要**：11 個の skill は相互参照しているため、必ず `--all` を付けてまとめてインストールしてください。個別に選んでインストールすると、dispatch 時に下流の skill が見つからなくなります。

**特定のエージェントを指定**（自動検出されない場合や、特定のエージェントだけにインストールしたい場合）：

```bash
# Claude Code のみ   →  ~/.claude/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent claude-code

# Codex のみ          →  ~/.codex/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent codex

# Antigravity のみ    →  ~/.gemini/antigravity/skills/
npx skills add kevintsai1202/teaching-site-skills --all --agent antigravity

# 対応する 55+ エージェント全てに一括インストール
npx skills add kevintsai1202/teaching-site-skills --all --agent '*'
```

**その他のよく使うコマンド**：

```bash
# インストールせずに、repo 内のインストール可能な skill を一覧表示
npx skills add kevintsai1202/teaching-site-skills --list

# 特定の skill のみインストール（非推奨：パイプラインが切れます）
npx skills add kevintsai1202/teaching-site-skills --skill teaching-site --skill course-outline-design
```

> **エージェント横断での注意**：本 skill セットは Claude Code 上で 4 日間ワークショップのフルパイプラインまでエンドツーエンドで検証済みです。他のエージェントは静的互換のサポートで、CLI でのインストールと frontmatter 形式は問題ありませんが、dispatch の体験、トリガーフレーズのヒット率、サブ skill の自動発見などはエージェント側の skill マッチング実装に依存します。新しいエージェントで最初に使うときは、「教材サイトを作って」の happy path を一度走らせて検証することを推奨します。

---

## パイプライン図

```text
Stage 1  course-outline-design          ← アウトライン（日数 / ユニット / 学習目標）
   ↓
Stage 2  course-content-authoring       ← 講義ノート / 素材 / クイズ
   ↓
Stage 3  static-spa-conversion          ← course-data.js + index.html
   ↓
Stage 4  static-spa-interactions        ← 進捗 / サイドバー / RWD / テーマ
   ↓
Stage 5  web-visual-assets              ← イラスト / スクリーンショット / QR / 地図
   ↓
   ├─→ Stage 5b  course-corporate-edition  ← 企業研修向け圧縮版（単一ファイル納品）
   └─→ Stage 6   course-ebook-publishing   ← PDF + DOCX 電子書籍
```

**横断（任意の stage から呼び出し可能）**：

| Skill | 答える問い | 失敗モード |
| --- | --- | --- |
| `web-visual-verification` | 動くか？見た目は正しいか？ | 非ゼロ exit、CI をブロック |
| `web-content-audit` | データ整合性は？何が欠けているか？ | 常に exit 0、レポートを出力 |
| `teaching-site-design-system` | どう見せるべきか、なぜか？ | 実行不可、デザインリファレンス |

**コーディネーター**：`teaching-site` がユーザーがどの stage にいるかを判断し、適切なサブ skill に dispatch します。

---

## トリガーフレーズ早見表

| Skill | 英語トリガーフレーズ | 中文觸發詞 |
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

> 日本語の場合は、「教材サイト作って」「ワークショップ用のページ」「電子書籍にして」「PDF にして」のような自然言語でも各 skill にディスパッチされます。

---

## よくあるシナリオ

**「4 日間のワークショップサイトをゼロから作って」**
→ `teaching-site` がリードを取り、outline → content → SPA → interactions → visuals の順に dispatch します。

**「すでに .md で講義ノートがあるので、サイトにして」**
→ `static-spa-conversion` に直接ジャンプ（stage 1–2 はスキップし、既存の markdown から構造を抽出します）。

**「サイトは完成した。受講者用に PDF を出力して」**
→ サイトが安定していることを確認してから、`course-ebook-publishing` を直接呼びます。

**「この公開講座を 6 時間の企業研修版に圧縮して」**
→ `course-corporate-edition` を直接呼び出します。出力は単一の `index.html`（COURSE を埋め込み）+ クライアントが zip にできるフォルダです。

**「サイトは動くが、モバイル表示がひどい」**
→ 直接 `static-spa-interactions` を呼び、完了後に `web-visual-verification` で複数ビューポートを検証します。

**「素材リンクが 404」「クイズの点数が間違って表示される」「進捗チェックがずれる」**
→ ほぼ確実に「三方同期」漏れです。`web-content-audit` を呼び出して、filesystem / course-data.js / `getMaterialUrl()` の 3 箇所を突き合わせます。

---

## 重要な設計判断（なぜこの分け方なのか）

1. **super-skill とサブ skill が共存**：広い要求は `teaching-site` のオーケストレーションを通り、具体的な要求はサブ skill を直接トリガーします。両方の経路が成立します。
2. **2 層アーキテクチャ — コアパイプライン + 横断**：コアの 7 つは順序付き（stage 1–6）で、横断の 3 つはどの stage からも呼び出せます。
3. **ファイル間の不変条件は、それを所有するサブ skill 内に置く**：例：「クイズに触ったら 5 箇所のハードコードを同期させる」は content-authoring に置く（触発する作業と同じ場所に共存）。
4. **検証と監査は意図的に 2 つの skill に分ける**：失敗モードが異なる（verify は CI をブロック / audit は常に成功）し、実行のタイミングも異なる（毎 PR / リリース時）から。
5. **デザインシステムは「何を」だけでなく「なぜ」も含む**：値は変わる（OKLCH トークンは調整される）が、その理由は通常変わりません。
6. **企業研修版は納品向けにリバース設計**：`window.COURSE` をインラインで埋め込み、素材の fallback チェーン（カスタム版 → 主版）、電子書籍ビルダーは `vm.runInContext` で COURSE を逆抽出。
7. **電子書籍の PDF は Chrome CLI ではなく Playwright の `page.pdf()`**：CLI が `footerTemplate` をサポートしていない（ページ番号が出ない）ため。
8. **localStorage の task-id キーは絶対にリネームしない**：公開後にリネームすると、全受講者の進捗チェックボックスがずれます。

詳しい理由は `teaching-site/references/design-rationale.md` を参照してください。

---

## 出典 & ライセンス

原実装：[ai-workshop](https://github.com/kevintsai1202/ai-workshop)（管理・財務領域の AI 自動化、4 日間ワークショップ）。

Skills は Claude Opus 4.7（1M コンテキスト版）が 2026-05-14 に、プロジェクトの履歴から抽出・抽象化・構造化したものです。執筆パスでは、30 個の git commits、33 個の verify/diagnose/audit スクリプト、`index.html`（3000 行以上）の CSS / JS セクション、および `企業包班/`（企業研修版）サブディレクトリを読み込みました。

各 skill は独立した markdown であり、使用・改変・再配布に制限はありません。改良したら、ぜひ原 repo へ PR を送ってください。
