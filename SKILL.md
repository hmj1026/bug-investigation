---
name: bug-investigation
description: "Systematic 5-phase bug investigation workflow for unexpected behavior, data inconsistencies, and root cause tracing. Use when users ask to investigate/trace bugs or data flow (e.g., bug investigation, 調查 Bug, 追蹤資料流, root cause analysis)."
---

# Bug Investigation Skill

## 概述

一套系統化方法，用於調查複雜程式碼中的錯誤與異常行為。此技能包含五個階段：
1. **問題釐清** - 理解回報的問題與影響範圍
2. **證據蒐集** - 從資料庫與日誌收集可驗證的證據
3. **根因分析** - 追蹤資料流找出來源與分歧點
4. **修正方案設計** - 提出與評估解決方案並形成決策依據
5. **知識文件化** - 留存可重用的知識與調查結果

**強制要求**：
- 所有五個階段必須完成，不可跳過。若受阻，必須記錄原因、缺口與下一步，並在調查文件中標示未完成狀態。
- 所有輸出文件與報告以正體中文為主；保留原始 log、程式碼與欄位名稱。

## 知識庫

調查過程中獲得的程式功能邏輯文件應同步存放在**專案內部**的知識庫資料夾：

```
docs/knowledge/
├── [feature-name]/
│   ├── investigation.md      # 調查總表與進度
│   ├── data-flow.md           # 資料流圖解
│   ├── key-functions.md       # 關鍵函數說明
│   ├── related-tables.md      # 相關資料表結構
│   └── solution-proposal.md   # 修正方案與決策依據
```

**好處**：
- 知識庫與專案程式碼一同版本控制
- 團隊成員可共享調查結果
- 日後調查類似問題時可先查閱
- 減少重複的 code tracing

**範例參考**：`references/examples.md` 與 `examples/state-inconsistency-example/`。

## 參考

- `references/scripts.md`：工具安裝與腳本使用說明
- `references/examples.md`：調查案例與寫作模板

---

## Phase 1: 問題釐清

> 提示：首次使用先執行 `./scripts/check-tools.sh`（詳見 `references/scripts.md`）。

### 1.1 收集初始資訊

向使用者詢問以下資訊：
- [ ] **問題描述**：預期行為與實際行為的差異為何？
- [ ] **樣本資料**：具體的 ID、時間戳記或交易編號
- [ ] **可重現性**：問題是否能穩定重現？
- [ ] **環境資訊**：受影響的環境、系統或資料庫

### 1.2 建立調查文件

在 `docs/knowledge/[feature-name]/investigation.md` 建立調查文件：

```bash
mkdir -p docs/knowledge/[feature-name]
```

```markdown
# [問題標題] 調查紀錄

## 問題描述
- **預期行為**：
- **實際行為**：
- **樣本資料**：

## 調查進度
- [ ] Phase 1: 問題釐清
- [ ] Phase 2: 證據蒐集
- [ ] Phase 3: 根因分析
- [ ] Phase 4: 修正方案設計
- [ ] Phase 5: 知識文件化

## 阻礙與缺口
- [ ] [描述阻礙、缺口與下一步]
```

---

## Phase 2: 證據蒐集

### 2.1 資料庫驗證

產生 SQL 查詢以驗證問題：

```sql
-- 範本：查詢主要交易
SELECT * FROM [main_table] WHERE [id] = '[sample_id]';

-- 範本：查詢關聯紀錄
SELECT * FROM [related_table] WHERE [foreign_key] = '[sample_id]';

-- 範本：查詢日誌
SELECT * FROM [log_table] WHERE [reference] = '[sample_id]';
```

### 2.2 記錄發現

在 `docs/knowledge/[feature-name]/investigation.md` 中記錄資料庫證據：

```markdown
## 資料庫證據

| 資料表 | 欄位 | 預期 | 實際 | 備註 |
|--------|------|------|------|------|
| [table] | [field] | [expected] | [actual] | [note] |
```

### 2.3 識別矛盾點

尋找資料不一致的地方：
- [ ] 相關資料表的資料是否匹配？
- [ ] Log 記錄是否與交易資料一致？
- [ ] 資料中是否有時序問題？

---

## Phase 3: 根因分析

### 3.1 追蹤資料流向

描繪資料從輸入到資料庫的完整路徑：

```
1. 使用者動作 → [函式/API]
           ↓
2. 前端處理 → [JS 函式]
           ↓
3. 後端 API → [Controller/Action]
           ↓
4. 資料庫寫入 → [資料表]
```

### 3.2 程式碼調查

對資料流中的每個步驟：

1. **搜尋關鍵變數** (使用專業工具):
   ```bash
   # 使用 ripgrep (推薦)
   rg "<variable_name>" --type php --type js
   
   # 或使用技能提供的腳本
   ./scripts/trace-data-flow.sh <variable_name>
   
   # 搜尋資料表操作
   ./scripts/search-database-queries.sh <table_name>
   ```

2. **追蹤資料來源**：
   - 哪個函式計算或提供此值？
   - 資料如何從前端傳遞到後端？
   - 使用 `analyze-function-calls.sh` 分析函式呼叫關係

3. **識別分歧點**：
   - 預期與實際行為在哪裡分歧？
   - 什麼條件導致進入錯誤的路徑？
   - 使用 `generate-flow-diagram.sh` 生成流程圖輔助分析

### 3.3 記錄根本原因

更新 `docs/knowledge/[feature-name]/investigation.md`：

```markdown
## 根因分析

### 資料流
[流程圖或逐步說明]

### 問題位置
- **檔案**： [檔案路徑]
- **行號**： [行號]
- **問題**： [問題描述]

### 發生原因
[觸發問題的條件與原因說明]
```

---

## Phase 4: 修正方案設計

### 4.1 建立修正方案文件（必做）

在 `docs/knowledge/[feature-name]/solution-proposal.md` 記錄修正方案與判斷依據：

```markdown
# [問題標題] 修正方案

## 方案選項
| 方案 | 描述 | 優點 | 風險/缺點 | 影響範圍 | 測試需求 |
|------|------|------|-----------|----------|----------|
| A | [前端修正] | [...] | [...] | [...] | [...] |
| B | [後端修正] | [...] | [...] | [...] | [...] |
| C | [前後端整合] | [...] | [...] | [...] | [...] |

## 判斷依據
- [引用 Phase 2/3 的證據與限制]
- [風險、成本、效益、時間與可回滾性]

## 推薦方案
- [建議方案與理由]

## 後續行動
- [實作步驟 / 測試 / 佈署與回滾]
```

### 4.2 設計解決方案選項

提出 2-3 個解決方案，並回填到 `solution-proposal.md`。

### 4.3 推薦解決方案

向使用者呈現建議：
- 推薦哪個選項？為什麼？
- 有什麼風險？
- 需要什麼測試？

### 4.4 建立 OpenSpec Proposal（如適用）

如果修復需要正式文件化，先參考 `openspec/AGENTS.md` 的格式與流程：

```bash
# 建立 OpenSpec proposal
mkdir -p openspec/changes/[YYYY-MM-DD]-[fix-description]
```

包含：
- `proposal.md` - 問題分析與解決方案
- `tasks.md` - 實作檢查清單
- `specs/[capability]/spec.md` - 規格變更

---

## Phase 5: 知識文件化

### 5.1 檢查現有知識庫

在深入研究程式碼之前，檢查是否已有相關文件：

```bash
# 搜尋知識庫是否已有相關文件
ls docs/knowledge/
```

### 5.2 建立功能知識文件

調查完成後，記錄功能邏輯供未來參考：

```bash
mkdir -p docs/knowledge/[feature-name]
```

建立以下文件：

#### `data-flow.md`
```markdown
# [功能名稱] - 資料流

## 概述
[功能簡述]

## 資料流圖
使用者動作 → [前端函式] → [後端 API] → [資料表]

## 關鍵變數
| 變數 | 位置 | 用途 |
|------|------|------|
| `[var]` | [file:line] | [用途說明] |
```

#### `key-functions.md`
```markdown
# [功能名稱] - 關鍵函數

## 前端 (JavaScript)
| 函數 | 檔案 | 說明 |
|------|------|------|
| `[func]()` | [file:line] | [功能說明] |

## 後端 (PHP)
| 函數 | 檔案 | 說明 |
|------|------|------|
| `[func]()` | [file:line] | [功能說明] |
```

#### `related-tables.md`
```markdown
# [功能名稱] - 資料表

## 主要資料表
| 資料表 | 主鍵欄位 | 用途 |
|--------|----------|------|
| `[table]` | `[pk]` | [用途說明] |

## 紀錄表
| 資料表 | 主鍵欄位 | 用途 |
|--------|----------|------|
| `[table]` | `[pk]` | [用途說明] |
```

### 5.3 更新檢查清單

將 Phase 5 加入調查檢查清單：

```markdown
### Phase 5: 知識文件化
- [ ] 已檢查現有知識庫
- [ ] 已建立/更新功能知識文件
- [ ] 已記錄資料流向
- [ ] 已列出關鍵函數與檔案位置
- [ ] 已記錄相關資料表
- [ ] 已確認 solution-proposal.md 完整
```

---

## 關鍵原則

### 調查方法論
- **追隨資料** - 從來源追蹤數值到目的地
- **信任證據** - 資料庫記錄不會說謊
- **一次一個假設** - 先測試和驗證再前進
- **記錄一切** - 保留調查軌跡

### 溝通方式
- **全程正體中文** - 所有輸出與文件維持正體中文
- **游進式報告** - 不要等到最後才報告
- **提出澄清問題** - 與使用者驗證假設
- **解釋推理** - 幫助使用者理解分析

### 解決方案設計
- **最小變更原則** - 只修復損壞的部分
- **預防未來問題** - 考慮如何避免類似的 bug
- **完整測試** - 驗證修復不會引入新問題

---

## 檢查清單總結

```markdown
## 調查檢查清單

### Phase 1: 問題釐清
- [ ] 理解預期與實際行為
- [ ] 獲取樣本資料 (ID、時間戳記)
- [ ] 建立調查文件
- [ ] 記錄阻礙與缺口（如有）

### Phase 2: 證據蒐集
- [ ] 執行資料庫驗證查詢
- [ ] 記錄資料表/欄位的差異
- [ ] 識別資料矛盾

### Phase 3: 根因分析
- [ ] 描繪完整資料流向
- [ ] 搜尋關鍵變數 (使用 ripgrep/scripts)
- [ ] 識別分歧點/問題程式碼
- [ ] 記錄根本原因

### Phase 4: 修正方案設計
- [ ] 建立 solution-proposal.md
- [ ] 提出 2-3 個解決方案選項
- [ ] 向使用者呈現建議
- [ ] 實作同意的解決方案
- [ ] 建立 OpenSpec proposal (如適用)
- [ ] 透過測試驗證修復

### Phase 5: 知識文件化
- [ ] 檢查現有知識庫
- [ ] 建立/更新功能知識文件
- [ ] 記錄資料流向 (data-flow.md)
- [ ] 列出關鍵 function 及檔案位置 (key-functions.md)
- [ ] 記錄相關資料表 (related-tables.md)
- [ ] 確認 solution-proposal.md 完整
```
