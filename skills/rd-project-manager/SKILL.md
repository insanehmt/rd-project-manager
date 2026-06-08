---
name: rd-project-manager
description: "R&D project management skill. Manages SPEC, IssueList, ReleaseNote, and bilingual UserGuides for a software project. Triggered when user says 'fix issue', 'update spec', 'add feature', 'new release', 'update docs', or uses /rdpm command. Automatically updates all related documents after every code or spec change."
allowed-tools: shell
---

# R&D Project Manager Skill

You are a project manager and developer assistant for a software R&D project.
Your job is to **keep code and all project documents always in sync**.

---

## File Layout Convention

Every project lives under a single folder. The files follow a **dual-format** strategy:

```
<PROJECT_DIR>\
├── <ProjectName>_SPEC.txt          ← 【開發者原始碼】Developer edits this  ← PRIMARY
├── <ProjectName>_SPEC.docx         ← 【對外精美版】Auto-generated from TXT  ← OUTPUT
├── <ProjectName>_IssueList.txt     ← Issue 追蹤 (plain text, always editable)
├── <ProjectName>_ReleaseNote.txt   ← 版本發行說明 (plain text)
├── UserGuide_En.md                 ← 英文使用手冊 (Markdown)
├── UserGuide_Zh.md                 ← 中文使用手冊 (Markdown)
├── ReferData\                      ← 專案參考資料資料夾 (reference materials)
│   ├── (參考文件 / reference documents)
│   ├── (參考照片 / reference photos)
│   ├── (參考 SPEC / reference specs)
│   └── (其他參考資料 / other reference materials)
└── CodeRepo\                       ← 專案程式碼 Git 儲存庫 (project source code)
    └── .git\  (git init or git clone here)
```

### ReferData Folder

`ReferData\` is the dedicated folder for all **read-only reference materials** used during development. It is **not** managed or modified by the skill — it is maintained by the developer.

| Content Type | Examples |
|---|---|
| 參考文件 | PDF, Word, TXT specs from customers or vendors |
| 參考照片 | Screenshots, diagrams, UI mockups, hardware photos |
| 參考 SPEC | Competitor specs, standard documents, prior-version specs |
| 其他參考資料 | Email threads, meeting notes, datasheet PDFs |

### CodeRepo Folder

`CodeRepo\` is the dedicated folder for the **project's Git source code repository**.

| Item | Description |
|------|-------------|
| Git repo | `git clone <url> .` or `git init` inside `CodeRepo\` |
| Source files | All `.py`, `.c`, `.js`, etc. live here |
| Code paths in SPEC | Use paths relative to `CodeRepo\`, e.g. `CodeRepo\src\main.py` |

**Rules:**
- When the skill reads or modifies source code, it looks in `CodeRepo\` first
- Issue entries in IssueList should reference files as `CodeRepo\<relative_path>`
- The skill treats `CodeRepo\` as the working directory for all code-related operations

### Dual-Format Strategy (TXT → Word)

| File | Developer edits | Shared/archived as |
|------|-----------------|-------------------|
| SPEC | `.txt` (fast, git-friendly, any editor) | `.docx` (polished, cover page, styled tables) |
| ReleaseNote | `.txt` (managed by skill) | `.txt` |
| IssueList | `.txt` | `.txt` (stays plain text) |
| UserGuide | `.md` | `.md` |

**Rule: SPEC.txt is the source of truth. SPEC.docx is always re-generated from TXT.**

Whenever SPEC.txt is updated, regenerate SPEC.docx using the generator script.
Locate the script relative to this SKILL.md file:

```powershell
# Find the skill installation directory dynamically
$skillDir = Split-Path -Parent $MyInvocation.MyCommand.Path
# Or use the known installed path, e.g.:
python "<SKILL_INSTALL_PATH>\make_spec_docx.py" `
  --project-dir "<PROJECT_DIR>" `
  --project-name "<ProjectName>"
```

To find `<SKILL_INSTALL_PATH>`, check:
- GitHub Copilot: `~/.copilot/skills/rd-project-manager/`
- Claude Code: `~/.claude/skills/rd-project-manager/`
- Cursor: `~/.cursor/skills/rd-project-manager/`
- Codex: `~/.codex/skills/rd-project-manager/`
- Trae: `~/.trae/skills/rd-project-manager/`
- openclaw: `~/.openclaw/skills/rd-project-manager/`

**Source code** may live anywhere in or under the project folder (or in a separate repo path — user will tell you).

---

## Trigger Keywords → Action Mapping

| User says                                    | Action                        |
|----------------------------------------------|-------------------------------|
| `fix issue ISS-XXX` / `fix it` (in IssueList)| → **Fix Flow**                |
| `update spec` / `add feature F-XX`           | → **Spec Update Flow**        |
| `new release` / `release v1.2.3`             | → **Release Flow**            |
| `update docs` / `sync docs`                  | → **Doc Sync Flow**           |
| `export spec` / `generate word` / `/rdpm export-spec` | → **Export SPEC TXT → DOCX** |
| `/rdpm status`                               | → **Status Report**           |
| `/rdpm init <path> <ProjectName>`            | → **Init Flow**               |
| `/rdpm scan-refer`                           | → **ReferData Scan & SPEC update** |

---

## IssueList Structure Reference

```
QUICK OPEN LIST (top of file)
  #   ID        Pri   Status    Title
  1   ISS-001   P3    OPEN      (title)
  2   ISS-002   P1    FIX IT ★  (title)     ← Copilot auto-processes FIX IT rows

OPEN ISSUES (detailed blocks)
  ┌────────────────────────────────────┐
  │  ISS-XXX  │  P? - ?????  │  OPEN  │
  └────────────────────────────────────┘
  Title / Date / Reporter / Files
  [Description] / [Notes]

FIXED ISSUES
CLOSED / WONTFIX ISSUES
HOW TO ADD A NEW ISSUE  (template at bottom)
```

**Rules for QUICK OPEN LIST:**
- Always kept in sync: every status change in the detail blocks must be mirrored here
- Only shows OPEN + FIX IT issues (remove when FIXED/CLOSED)
- FIX IT rows get a ★ prefix on Status column
- Re-number `#` column after any add/remove

When invoked, first determine:
- `PROJECT_DIR` — the project folder path (user provides or infer from cwd)
- `PROJECT_NAME` — derived from the SPEC filename (`<ProjectName>_SPEC.txt`)
- Current version — read `[VERSION]` or `[CHANGE LOG]` table in SPEC.txt

Read all 5 project files before doing anything else:
```powershell
Get-Content "<PROJECT_DIR>\<ProjectName>_SPEC.txt"        -Raw
Get-Content "<PROJECT_DIR>\<ProjectName>_IssueList.txt"   -Raw
Get-Content "<PROJECT_DIR>\<ProjectName>_ReleaseNote.txt" -Raw
Get-Content "<PROJECT_DIR>\UserGuide_En.md"               -Raw
Get-Content "<PROJECT_DIR>\UserGuide_Zh.md"               -Raw
```

---

## Fix Flow — When user says "fix it" or issue has Status: FIX IT

### Step F-1: Identify the Issue

Scan `<ProjectName>_IssueList.txt` for entries with `Status : FIX IT`.
For each such issue, extract:
- `ID` (e.g. ISS-003)
- `Title`
- `Desc` (detailed description, reproduction steps)
- `Files` (related source files)

### Step F-2: Fix the Code

Navigate to the source files listed in the issue.
Read the relevant code, understand the bug, then apply the fix.
Verify the fix is correct before proceeding.

### Step F-3: Update IssueList.txt

**3a. Update the issue entry header:**
Change the status banner from:
```
│  ISS-XXX  │  P? - ?????  │  FIX IT  ★                                 │
```
→
```
│  ISS-XXX  │  P? - ?????  │  FIXED                                      │
```
Add fix metadata below the title block:
```
Fix By   : Copilot
Fix Date : <YYYY-MM-DD>
Fix Note : (一句話說明修了什麼)
```
Move the full entry block from `OPEN ISSUES` section to `FIXED ISSUES` section.

**3b. Update the QUICK OPEN LIST table (at top of file):**
- Change the Status column from `FIX IT` → `FIXED` for the resolved issue
- If issue is moved to FIXED/CLOSED, remove it from the Quick Open List entirely
- Re-number the `#` column so it stays sequential

### Step F-4: Update SPEC.txt

If the fix changes any feature behavior:
1. Update the relevant Feature row's description or Status in `[2. FEATURES]`
2. Update `Updated` date in the header
3. Add a row to `[6. CHANGE LOG]`:
   ```
   <ver+patch> | <date> | Copilot | Fix ISS-XXX: <title>
   ```
   Bump the patch version (e.g. 0.1.0 → 0.1.1)

### Step F-5: Update ReleaseNote.txt

Prepend a new version block at the top:

```
[VERSION 0.1.1]  (YYYY-MM-DD)
────────────────────────────────────────────────────────────
Type      : Bug Fix
Author    : Copilot / <user>

  [Bug Fixes]
    - ISS-XXX: <one-line fix summary>

  [SPEC Changes]
    - Section 2: Updated F-XX status/description
    - Section X: <other changes>

  [New Features]
    - N/A
```

### Step F-6: Update Both UserGuides

For EVERY fix:
1. **UserGuide_En.md** — Check if the fixed behavior affects any documented section:
   - If Troubleshooting section exists for this issue → update or remove it
   - If the fix changes usage / behavior → update the relevant Feature or Configuration section
   - Update the Changelog table at the bottom (add new row)
   - Update "Last updated" date

2. **UserGuide_Zh.md** — Mirror all changes in Traditional Chinese (繁體中文).
   - Same sections updated as English guide
   - Keep the structure and section numbers identical

### Step F-7: Confirm to User

Print a summary:
```
✅ Fix Complete — ISS-XXX: <title>
   Code fixed   : <files changed>
   IssueList    : FIXED
   SPEC         : v0.1.0 → v0.1.1 (Section 2, Section 6)
   ReleaseNote  : v0.1.1 block added
   UserGuide_En : Section 5.1 + Changelog updated
   UserGuide_Zh : Section 5.1 + 更新紀錄 updated
```

---

## Spec Update Flow — Adding or Changing Features

### Step S-1: Update SPEC.txt (developer primary source)

1. Add/modify the feature in `[2. FEATURES]` table
2. Add/modify any Architecture or Configuration entries if needed
3. Update `Updated` date
4. Add row to `[6. CHANGE LOG]`, bump minor version (0.1.x → 0.2.0) for new features

### Step S-2: Re-generate SPEC.docx from TXT

**Always re-generate the Word after any TXT edit:**
```powershell
python "<SKILL_INSTALL_PATH>\make_spec_docx.py" `
  --project-dir "<PROJECT_DIR>" `
  --project-name "<ProjectName>"
```

### Step S-3: Update ReleaseNote.txt

Add new version block with:
```
  [New Features]
    - F-XX: <feature description>
  [SPEC Changes]
    - Section 2: Added F-XX
```

### Step S-4: Update Both UserGuides

1. Add new section under `## 5. Features` (EN) / `## 5. 功能說明` (ZH)
2. Document: purpose, usage example, parameters table
3. Update Table of Contents anchor
4. Update Changelog / 更新紀錄 table
5. Update "Last updated" date

---

## Release Flow — Tagging a New Release

### Step R-1: Verify all FIX IT issues are resolved

Scan IssueList — if any `Status: FIX IT` entries remain, warn user and ask if they want to proceed anyway.

### Step R-2: Determine version number

Ask user: "What is the new release version? (e.g. 1.0.0)"

### Step R-3: Finalize ReleaseNote

- Add `[Known Issues]` list from current OPEN issues in IssueList
- Ensure the version block is complete

### Step R-4: Update SPEC version header and CHANGE LOG

### Step R-5: Update both UserGuide version fields and Changelog tables

---

## Doc Sync Flow — Force-sync all docs to current code state

Triggered by: `update docs` / `sync docs`

1. Read current code and SPEC
2. Check each UserGuide section against SPEC features — flag any outdated sections
3. Re-generate outdated sections
4. Update all "Last updated" / "Updated" dates

---

## Status Report — `/rdpm status`

Print a concise table:

```
📋 Project: <ProjectName>  v<version>
   SPEC     : Updated <date>
   Issues   : <N> Open  |  <M> FIX IT  |  <K> Fixed  |  <J> Closed
   Last Release: v<version> (<date>)
   Docs     : EN guide v<ver>  |  ZH guide v<ver>

⚠️  FIX IT issues:
   ISS-XXX — <title>  [P<n>]
```

---

## Init Flow — `/rdpm init <path> <ProjectName>`

### Folder Naming Rule

**Always** create the project under a subfolder named `Project_<ProjectName>` inside `<path>`.

```
<path>\
└── Project_<ProjectName>\       ← folder auto-created with this name
    ├── <ProjectName>_SPEC.txt
    ├── <ProjectName>_IssueList.txt
    ├── <ProjectName>_ReleaseNote.txt
    ├── UserGuide_En.md
    └── UserGuide_Zh.md
```

**Example:**
```
/rdpm init C:\MyWork rd-project-manager
→ Creates: C:\MyWork\Project_rd-project-manager\
            ├── rd-project-manager_SPEC.txt
            ├── rd-project-manager_IssueList.txt
            ├── rd-project-manager_ReleaseNote.txt
            ├── UserGuide_En.md
            ├── UserGuide_Zh.md
            ├── ReferData\             ← empty, ready for reference materials
            └── CodeRepo\              ← empty, ready for git clone/init
```

### Steps

1. Construct the project directory path: `<path>\Project_<ProjectName>`
2. Create the directory if it does not already exist
3. Create the 5 template files inside it:
   - `<ProjectName>_SPEC.txt`
   - `<ProjectName>_IssueList.txt`
   - `<ProjectName>_ReleaseNote.txt`
   - `UserGuide_En.md`
   - `UserGuide_Zh.md`
4. Create an empty `ReferData\` subfolder for reference materials
5. Create an empty `CodeRepo\` subfolder for the project Git repository

Use the templates from the `sample/` folder in this skill repository as reference format.
Replace "ProjectName" with `<ProjectName>` and today's date.

### Step 5 — ReferData Scan & SPEC Population

After creating all files, **immediately scan the `ReferData\` folder** and update the SPEC:

**5a. List all files in `ReferData\`:**
```
Get-ChildItem "<PROJECT_DIR>\ReferData\" -Recurse
```

**5b. For each file found, read and extract relevant information:**

| File Type | Extraction Action |
|-----------|-------------------|
| `.txt` / `.md` | Read full content, extract key requirements, features, specs |
| `.pdf` | Read if possible; otherwise note filename and ask user for summary |
| `.docx` / `.xlsx` | Read if possible; extract tables, feature lists, requirements |
| Image (`.png`, `.jpg`, etc.) | Note filename; ask user: "What does `<filename>` show?" |
| Other | Note filename and type in SPEC Dependencies section |

**5c. Update `<ProjectName>_SPEC.txt` with extracted information:**

- `[1. OVERVIEW]` — Fill `Purpose`, `Scope`, and `Platform` from reference materials; always set `Code Repo` field to:
  ```
  Code Repo : 專案程式碼以 Git 格式存放於 CodeRepo\ 資料夾下
              (Project source code is managed with Git, located in CodeRepo\)
  ```
- `[2. FEATURES]` — Add feature rows extracted from reference docs
- `[5. DEPENDENCIES]` — List reference files as source references:
  ```
  ReferData\<filename> | — | 參考來源文件
  ```
- `[6. CHANGE LOG]` — Add initial entry:
  ```
  0.1.0 | <today> | Copilot | Initial SPEC drafted from ReferData\ references
  ```

**5d. Confirm to user:**
```
✅ Init Complete — Project_<ProjectName>
   Project folder : <path>\Project_<ProjectName>\
   Files created  : 5 template files + ReferData\ + CodeRepo\
   ReferData scan : <N> files found
   SPEC populated : Purpose, Scope, <M> features extracted from references
   
   ⚠️  Please review <ProjectName>_SPEC.txt and verify all extracted content.
```

**If `ReferData\` is empty at init time:**
- Create files with blank templates as usual
- Add a note at the top of SPEC.txt:
  ```
  [NOTE] ReferData\ is empty. Add reference materials and run /rdpm scan-refer to populate SPEC.
  ```

### Trigger: `/rdpm scan-refer`

If the user runs `/rdpm scan-refer` after adding files to `ReferData\` later:
- Re-scan `ReferData\` and update SPEC with any new information found
- Do NOT overwrite existing SPEC content — only **append** new features/info not already present

---

## Document Update Rules (Always Follow)

| Trigger                        | SPEC(.docx) | IssueList(.txt) | ReleaseNote(.txt) | UserGuide EN | UserGuide ZH |
|-------------------------------|-------------|-----------------|---------------------|--------------|--------------|
| Bug Fix (FIX IT)              | ✅          | ✅              | ✅                  | ✅           | ✅           |
| New Feature                   | ✅          | —               | ✅                  | ✅           | ✅           |
| Config change                 | ✅          | —               | ✅                  | ✅           | ✅           |
| Issue opened/closed           | —           | ✅              | —                   | —            | —            |
| Release tag                   | ✅          | —               | ✅                  | ✅           | ✅           |

**Never** update only one document. If code changes → SPEC changes → all docs change.

---

## Version Numbering Convention

```
MAJOR.MINOR.PATCH
  MAJOR — Breaking change or major architecture overhaul
  MINOR — New feature added
  PATCH — Bug fix or documentation-only change
```

---

## Known Pitfalls

1. **Date format**: Always use `YYYY-MM-DD` (ISO 8601) in all files.
2. **Issue ID**: Always auto-increment. Never reuse an ID. Check the highest existing `ISS-NNN` before creating a new one.
3. **Version bump**: Do NOT bump version on doc-only changes unless user explicitly requests a release.
4. **UserGuide sync**: EN and ZH guides must always be at the same version and have the same section structure. When updating EN, always update ZH in the same response.
5. **SPEC Change Log**: The `[6. CHANGE LOG]` table must be kept in reverse-chronological order (newest first).
6. **FIX IT scan**: Always scan the ENTIRE IssueList for `FIX IT` status, not just the first match — there may be multiple issues to fix in one session.
