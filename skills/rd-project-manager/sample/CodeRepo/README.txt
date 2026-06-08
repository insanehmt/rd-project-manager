CodeRepo — 專案程式碼儲存庫
=====================================

此資料夾用來存放專案的 Git 程式碼儲存庫。
Place the project's Git repository (or clone it) here.

使用方式 / Usage:
  1. 在此資料夾下 clone 或 init 專案 Git repo：
       git clone <repo_url> .
       git init

  2. 或將現有 code 複製至此資料夾後初始化：
       git init
       git add .
       git commit -m "initial commit"

建議結構 / Suggested structure:
  CodeRepo\
  ├── .git\          ← Git repository data
  ├── src\           ← Source code
  ├── tests\         ← Test files
  ├── docs\          ← Code-level documentation
  └── README.md      ← Code README

注意 / Notes:
  - 此資料夾內容由開發者自行管理，AI skill 在讀取 code 時會從此資料夾尋找原始碼。
  - When the skill needs to read or modify source code, it will look here first.
  - SPEC.txt 中 [FILES] 欄位應填寫相對於 CodeRepo\ 的路徑。
    Example: CodeRepo\src\main.py
