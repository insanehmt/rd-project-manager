# GitHub Copilot Instructions — Project_Sample

> 本檔案為 AI 輔助開發的行為準則。所有 Copilot / AI Agent 在協助此專案時，必須遵守以下規範。

---

## 1. 前提條件

- **回覆語言**：所有回覆、說明、註解一律使用**繁體中文**。
- **大規模變更防護**：單次改動超過 **200 行**時，必須先提出變更計畫（列出修改範圍、影響模組、預期結果），經確認後再執行。
- **逐步確認**：進行跨檔案重構或刪除既有邏輯前，先說明意圖並列出受影響的檔案清單，等待使用者確認。
- **不擅自安裝套件**：在建議新增依賴前，需明確說明用途並等待確認，不自動修改 `package.json` / `requirements.txt` 等依賴檔案。
- **保留既有風格**：維持專案現有的命名慣例、縮排風格與程式碼結構，不自行格式化整份檔案。

---

## 2. 應用程式概述

- **專案名稱**：Project_Sample
- **目的**：（描述此專案解決的問題或提供的服務）
- **主要功能**：
  - （功能一）
  - （功能二）
  - （功能三）
- **目標使用者**：（描述主要用戶群）
- **平台**：（Web / Desktop / Mobile / CLI 等）

---

## 3. 技術堆疊

| 類別 | 技術 | 版本 |
|------|------|------|
| 語言 | （例：TypeScript） | （例：5.x） |
| 框架 | （例：React / Vue / FastAPI） | （例：18.x） |
| 狀態管理 | （例：Zustand / Pinia） | （例：4.x） |
| 樣式 | （例：Tailwind CSS） | （例：3.x） |
| 資料庫 | （例：PostgreSQL / SQLite） | （例：15.x） |
| 測試 | （例：Vitest / Jest） | （例：1.x） |
| 建置工具 | （例：Vite / Webpack） | （例：5.x） |

> **注意**：不得 import 上表以外的函式庫，需新增依賴時請先提出需求。

---

## 4. 目錄結構

```
project-root/
├── src/
│   ├── features/        # 按功能模組分類（每個功能獨立資料夾）
│   │   └── {feature}/
│   │       ├── components/   # 該功能的 UI 元件
│   │       ├── hooks/        # 該功能的自訂 Hook
│   │       └── api/          # 該功能的 API 呼叫
│   ├── shared/          # 跨功能共用元件、工具函式、型別定義
│   │   ├── components/
│   │   ├── utils/
│   │   └── types/
│   ├── pages/           # 頁面層級元件（路由對應）
│   └── main.ts          # 應用程式進入點
├── tests/               # 測試檔案（鏡像 src 結構）
├── docs/                # 文件
└── .github/             # CI/CD 設定、Copilot 指示
```

> 新增檔案時，請依照上述結構放置於對應目錄，不得直接放在 `src/` 根目錄。

---

## 5. 架構與設計指引

- **整體架構**：（例：Feature-Sliced Design / Clean Architecture / MVVM）
- **元件設計**：（例：Atomic Design — atoms → molecules → organisms → pages）
- **資料流**：（例：單向資料流，禁止元件直接修改 store 以外的全域狀態）
- **API 呼叫**：所有外部 API 呼叫集中於 `src/features/{feature}/api/`，禁止在 UI 元件中直接呼叫 fetch / axios。
- **錯誤處理**：非同步操作必須包含 try/catch，錯誤訊息統一透過全域 toast 通知機制呈現。
- **型別安全**：函式參數與回傳值必須明確標註型別，禁止使用隱式 `any`。

---

## 6. 測試方針

- **測試框架**：（例：Vitest + Testing Library）
- **測試檔案位置**：放置於 `tests/` 目錄下，鏡像 `src/` 的資料夾結構。
  - 例：`src/features/auth/hooks/useLogin.ts` → `tests/features/auth/hooks/useLogin.test.ts`
- **覆蓋率目標**：整體覆蓋率 ≥ 80%，核心業務邏輯 ≥ 90%。
- **測試範圍**：
  - Unit Test：純函式、工具函式、自訂 Hook
  - Integration Test：跨元件互動、API Mock 整合
  - 禁止為純 UI 樣式撰寫 snapshot test（維護成本過高）
- **命名規則**：測試描述使用中文，格式為 `describe('模組名稱') + it('應該…')`。

```typescript
// 範例
describe('useLogin', () => {
  it('應該在帳號密碼正確時回傳使用者資訊', async () => { ... })
  it('應該在帳號密碼錯誤時拋出錯誤', async () => { ... })
})
```

---

## 7. 反模式（禁止事項）

以下行為在本專案中**明確禁止**：

| 禁止項目 | 說明 |
|---------|------|
| `default export` | 所有模組一律使用 named export，方便 IDE 追蹤引用 |
| 隱式 `any` 型別 | 明確標註型別，若確實需要請使用 `unknown` 並做型別守衛 |
| 直接操作 DOM | 使用框架提供的響應式機制，禁止 `document.querySelector` |
| Magic Number | 數字常數必須定義為具名常數（`const MAX_RETRY = 3`） |
| 巢狀三元運算子 | 超過一層的條件判斷改用 `if/else` 或 early return |
| 在元件內呼叫 API | API 邏輯須抽離至 `api/` 層，元件只負責呈現 |
| 忽略 Promise 錯誤 | 所有 `async/await` 必須有 try/catch 或 `.catch()` 處理 |
| 提交 `console.log` | 除 debug 用途外，不得在正式程式碼中留下 `console.log` |
| 修改 `node_modules` | 任何情況下不得直接修改 node_modules 內的檔案 |

---

*最後更新：2026-06-04 ｜ 維護者：Steven Huang*
