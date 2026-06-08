# rd-project-manager

> **R&D Project Manager Skill** — An AI agent skill that keeps your code and all project documents always in sync.

[![skill-flow compatible](https://img.shields.io/badge/skill--flow-compatible-blue)](https://github.com/insanehmt/rd-project-manager)

---

## 簡介 / Introduction

**中文：** 這是一個 AI Agent Skill，讓 GitHub Copilot、Claude Code、Codex、Trae、openclaw 等 AI 助理扮演「R&D 專案文件管理員」的角色。每次修程式碼、新增功能、修 Bug，它會自動同步更新 SPEC、IssueList、ReleaseNote、雙語 UserGuide，讓文件永遠不落後於程式碼。

**English:** An AI agent skill that turns your AI assistant (GitHub Copilot, Claude Code, Codex, Trae, openclaw, etc.) into an R&D project document manager. Every code change, new feature, or bug fix is automatically reflected across SPEC, IssueList, ReleaseNote, and bilingual UserGuides — documents always stay in sync with code.

---

## 安裝 / Installation

### via skill-flow (Recommended)

```powershell
skill-flow add https://github.com/insanehmt/rd-project-manager
```

Select `rd-project-manager` and your target agents in the interactive TUI.

### Manual

Copy `skills/rd-project-manager/` into your agent's skills folder:

| Agent | Skills folder |
|-------|--------------|
| GitHub Copilot | `~/.copilot/skills/` |
| Claude Code | `~/.claude/skills/` |
| Cursor | `~/.cursor/skills/` |
| Codex | `~/.codex/skills/` |
| Trae | `~/.trae/skills/` |
| openclaw | `~/.openclaw/skills/` |

---

## 功能 / Features

### 文件架構 / File Layout

每個專案建立以下 5 個文件：

```
<PROJECT_DIR>\
├── <ProjectName>_SPEC.txt          ← Developer source (primary)
├── <ProjectName>_SPEC.docx         ← Auto-generated polished Word output
├── <ProjectName>_IssueList.txt     ← Issue tracking (plain text)
├── <ProjectName>_ReleaseNote.txt   ← Release notes (plain text)
├── UserGuide_En.md                 ← English user guide
├── UserGuide_Zh.md                 ← 中文使用手冊
├── ReferData\                      ← Reference materials (docs, photos, specs)
└── CodeRepo\                       ← Project source code (git repo)
```

### 觸發指令 / Trigger Commands

| Command | Action |
|---------|--------|
| `fix issue ISS-XXX` / `fix it` | Bug Fix Flow |
| `update spec` / `add feature F-XX` | Spec Update Flow |
| `new release` / `release v1.2.3` | Release Flow |
| `update docs` / `sync docs` | Doc Sync Flow |
| `/rdpm init <path> <ProjectName>` | Initialize new project |
| `/rdpm status` | Show project status report |
| `export spec` / `generate word` | Generate SPEC.docx from TXT |

### SPEC.docx 自動生成 / Auto Word Generation

The included `make_spec_docx.py` generates a professionally styled Word document from your `SPEC.txt`:

```powershell
python skills/rd-project-manager/make_spec_docx.py \
  --project-dir "C:\MyProject" \
  --project-name "MyProject"
```

**Features:**
- Cover page with version & status badge
- Color-coded feature status tables
- Styled section headings with brand colors
- Header/footer with page numbers
- Alternating-row tables

---

## 快速開始 / Quick Start

### 1. 建立新專案 / Initialize a new project

```
/rdpm init C:\MyWork MyProject
```

This creates the folder `C:\MyWork\Project_MyProject\` and populates it with all files:

```
C:\MyWork\Project_MyProject\
├── MyProject_SPEC.txt
├── MyProject_IssueList.txt
├── MyProject_ReleaseNote.txt
├── UserGuide_En.md
├── UserGuide_Zh.md
├── ReferData\             ← place reference docs/photos here
└── CodeRepo\              ← git clone or init your code here
```

### 2. 修 Bug / Fix a bug

Mark an issue as `FIX IT` in the IssueList, then tell the AI:

```
fix it
```

The AI will fix the code and update all 5 documents automatically.

### 3. 新增功能 / Add a feature

```
add feature F-03: dark mode support
```

### 4. 發布新版本 / New release

```
new release v1.2.0
```

---

## Sample 範例專案 / Sample Project

See the [`sample/`](./sample/) folder for a complete template project with all 5 files pre-filled.

---

## 相依套件 / Dependencies

The `make_spec_docx.py` script requires:

```bash
pip install -r skills/rd-project-manager/requirements.txt
```

Or manually:

```bash
pip install "python-docx>=1.1.0,<2.0.0"
```

---

## 版本 / Version

`1.0.1` — 2026-06-08

---

## 授權 / License

MIT License — see [LICENSE](./LICENSE)
