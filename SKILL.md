---
name: siyuan-skill
description: 思源筆記（SiYuan）核心概念與最佳實踐。內容塊、引用、標籤、屬性、SQL 查詢、API 操作指南。
homepage: https://github.com/siyuan-note/siyuan
metadata:
  {
    "openclaw": {
      "emoji": "📔"
    }
  }
---

# 思源筆記（SiYuan）— 核心知識與最佳實踐

本 Skill 是思源筆記的知識庫，教 AI 怎麼「用好」思源的特色功能。
不包含個人歸檔規則（那是 note-flow 的職責）。

---

## 1) 核心概念：內容塊（Content Block）

思源的唯一核心概念是**內容塊**。所有內容都是塊，文件本身也是塊。

### 塊的類型

| 類型 | 說明 | type 值 |
|---|---|---|
| 文件塊 | 一份文件就是一個塊，有自己的 ID | `d` |
| 標題塊 | `#` ~ `######` | `h` |
| 段落塊 | 普通文字 | `p` |
| 列表塊 | 有序/無序/任務列表 | `l` |
| 列表項 | 列表中的每一項 | `i` |
| 引述塊 | `>` blockquote | `b` |
| 代碼塊 | ``` code ``` | `c` |
| 表格塊 | Markdown 表格 | `t` |
| 超級塊 | 橫向/縱向排版容器 | `s` |
| 嵌入塊 | `{{ SQL }}` 動態查詢嵌入 | `query_embed` |
| 數學公式 | LaTeX | `m` |
| 分隔線 | `---` | `tb` |

### 關鍵原則

- **文件也是塊** — 每個 `.sy` 文件有一個 block ID，就是文件 ID
- **塊可互轉** — 標題可轉段落、列表項可轉段落等
- **塊有屬性** — 每個塊都有 `id`、`updated`、`type`，可附加自訂屬性
- **塊級粒度** — 引用、嵌入、標籤都在塊級別操作

---

## 2) 引用與雙向連結

思源最強大的功能。建立引用後，自動產生正向連結和反向連結。

### 引用內容塊語法

```markdown
((<block_id> '動態錨文字'))    ← 單引號：錨文字隨原文變化（推薦）
((<block_id> "靜態錨文字"))    ← 雙引號：錨文字固定不變
```

### 連結方向

| 方向 | 說明 | 價值 |
|---|---|---|
| **正向連結** | 當前塊引用了哪些塊 | 直接可見 |
| **反向連結** | 當前塊被哪些塊引用 | 更有價值 — 發現意外關聯 |

### 關係圖

引用會在**關係圖**中可視化，可以看到塊之間的網絡關係。
- 全域關係圖：所有筆記的關聯網絡
- 局部關係圖：以某個塊為中心的關聯

### 最佳實踐

| 原則 | 說明 |
|---|---|
| **優先用引用** | 需要建立關聯時，用 `(())` 而不是超連結 |
| **超連結僅供外部** | `[文字](siyuan://blocks/ID)` 不建立關聯，僅供從外部跳轉 |
| **首次出現時引用** | 同一篇筆記中，同一個引用只嵌一次（首次出現時） |
| **多引用不怕** | 被多處引用的塊，反向連結面板自然形成知識索引 |
| **預設用單引號** | `'動態錨文字'` 隨原文變化，是思源原生慣例 |

### API 操作

建立引用不需要特殊 API — 直接在 markdown 內容中寫 `((<id> '文字'))` 即可，思源渲染時自動解析。

---

## 3) 標籤系統

標籤用於標記內容塊，所有打過標籤的塊會在標籤面板（`Alt+4` / `⌃4`）中列出。

### 語法

```markdown
#標籤名#                  ← 基本標籤（前後都要 #）
#分類/子分類#              ← 層級標籤（用 / 分隔）
#工具/LibreNMS#           ← 範例：工具類層級標籤
```

### 常見錯誤

```markdown
❌ `#Docker#`             → 行內代碼，不會變成標籤
❌ #Docker                → 缺少結尾 #
❌ # Docker#              → # 後有空格，變成 Markdown 標題
✅ #Docker#               → 正確
✅ 這是內容 #Task#        → 正確，標籤可以跟在文字後面
✅ #工具/Docker#          → 正確，層級標籤
```

### 最佳實踐

| 原則 | 說明 |
|---|---|
| **標籤 ≠ 分類** | 分類靠筆記本/資料夾，標籤是輔助索引 |
| **標籤回答「是什麼」** | 描述內容性質（事件、學習、SOP），不是位置 |
| **層級標籤更精準** | `#工具/LibreNMS#` 比 `#LibreNMS#` 更好，可按層級瀏覽 |
| **每則筆記至少一個** | 確保所有內容可透過標籤面板找到 |

### API 操作

```bash
# 查詢所有標籤
POST /api/search/getTags  {}

# SQL 查詢帶特定標籤的塊
SELECT * FROM blocks WHERE tag LIKE '%標籤名%'
```

---

## 4) 屬性系統

每個塊都可以附加屬性（key-value），用於分類、過濾、模板化。

### 內建屬性

| 屬性 | 說明 |
|---|---|
| `id` | 塊的唯一識別碼 |
| `updated` | 最後更新時間（YYYYMMDDHHmmss） |
| `type` | 塊類型（d/h/p/l/i/b/c/t/s...） |
| `name` | 塊命名（可搜尋） |
| `alias` | 塊別名（多個，逗號分隔） |
| `memo` | 塊備註 |
| `bookmark` | 書籤標記 |

### 自訂屬性

以 `custom-` 開頭的屬性可自由定義：

```markdown
{: custom-priority="high" custom-status="in-progress" }
```

### API 操作

```bash
# 設定屬性
POST /api/attr/setBlockAttrs
{ "id": "<block_id>", "attrs": { "custom-priority": "high" } }

# 讀取屬性
POST /api/attr/getBlockAttrs
{ "id": "<block_id>" }

# SQL 查詢有特定屬性的塊
SELECT * FROM attributes WHERE name = 'custom-priority' AND value = 'high'
```

---

## 5) 嵌入塊（Query Embed）

用 SQL 動態查詢並嵌入其他塊的內容，保持即時更新。

### 語法

在思源中用 `{{ }}` 包裹 SQL：

```sql
{{ SELECT * FROM blocks WHERE tag LIKE '%會議%' AND created >= '20260201' ORDER BY created DESC LIMIT 10 }}
```

### 常用場景

| 場景 | SQL 範例 |
|---|---|
| 列出所有待辦 | `SELECT * FROM blocks WHERE markdown LIKE '%- [ ]%' AND type='i'` |
| 某標籤的所有筆記 | `SELECT * FROM blocks WHERE tag LIKE '%工具/LibreNMS%' AND type='d'` |
| 最近更新的文件 | `SELECT * FROM blocks WHERE type='d' ORDER BY updated DESC LIMIT 20` |
| 某筆記本下的文件 | `SELECT * FROM blocks WHERE type='d' AND box='<notebook_id>'` |

---

## 6) SQL 查詢技巧

思源內建 SQLite 資料庫，可透過 API 查詢。

### 主要資料表

| 表 | 用途 |
|---|---|
| `blocks` | 所有內容塊（最常用） |
| `attributes` | 塊的自訂屬性 |
| `refs` | 引用關係（正向連結） |
| `spans` | 行內元素（標籤、引用等） |

### blocks 表重要欄位

| 欄位 | 說明 |
|---|---|
| `id` | 塊 ID |
| `root_id` | 所屬文件的 ID |
| `box` | 所屬筆記本 ID |
| `type` | 塊類型（d/h/p/l/i...） |
| `content` | 純文字內容（去除格式） |
| `markdown` | Markdown 原文 |
| `tag` | 標籤（含 #） |
| `hpath` | 人類可讀路徑（`/資料夾/文件名`） |
| `path` | 檔案路徑（含 ID） |
| `created` | 建立時間（YYYYMMDDHHmmss） |
| `updated` | 更新時間 |

### 常用查詢

```sql
-- 搜尋文件標題
SELECT id, content, hpath FROM blocks WHERE type='d' AND content LIKE '%關鍵字%'

-- 搜尋含特定標籤的塊
SELECT * FROM blocks WHERE tag LIKE '%工具/LibreNMS%'

-- 查某文件的所有子塊
SELECT * FROM blocks WHERE root_id = '<doc_id>' ORDER BY sort

-- 查引用某塊的所有來源
SELECT * FROM refs WHERE def_block_id = '<block_id>'

-- 查某筆記本下最近更新的文件
SELECT id, content, updated FROM blocks 
WHERE type='d' AND box='<notebook_id>' 
ORDER BY updated DESC LIMIT 10

-- 查某文件下的標題結構
SELECT id, content, type, subType FROM blocks 
WHERE root_id = '<doc_id>' AND type = 'h'
```

### API 呼叫

```bash
POST /api/query/sql
{ "stmt": "SELECT id, content FROM blocks WHERE type='d' AND content LIKE '%關鍵字%' LIMIT 10" }
```

⚠️ **注意**：SQL 查詢有索引延遲，文件操作後幾秒到幾十秒才更新。需要即時性的用 `listDocsByPath`。

---

## 7) REST API 常用端點

### 文件操作

| 端點 | 用途 | 注意 |
|---|---|---|
| `POST /api/filetree/createDocWithMd` | 建立文件 | `path` 最後一段自動成為標題（H1），markdown 不要再加 `# 標題` |
| `POST /api/filetree/removeDoc` | 刪除文件 | ⚠️ 會遞歸刪除所有子文件 |
| `POST /api/filetree/listDocsByPath` | 列出資料夾內容 | 即時性好，不受索引延遲影響 |
| `POST /api/filetree/renameDoc` | 重新命名 | |
| `POST /api/filetree/moveDoc` | 移動文件 | |

### 塊操作

| 端點 | 用途 | 注意 |
|---|---|---|
| `POST /api/block/appendBlock` | 追加塊到文件末尾 | 最常用的寫入方式 |
| `POST /api/block/insertBlock` | 在指定位置插入塊 | |
| `POST /api/block/updateBlock` | 更新塊內容 | |
| `POST /api/block/deleteBlock` | 刪除塊 | ⚠️ 對 doc 類型無效，刪文件用 `removeDoc` |
| `POST /api/block/getChildBlocks` | 取得子塊列表 | |
| `POST /api/block/getBlockKramdown` | 取得塊的 Kramdown 原文 | |

### 搜尋與查詢

| 端點 | 用途 |
|---|---|
| `POST /api/query/sql` | SQL 查詢 |
| `POST /api/search/fullTextSearchBlock` | 全文搜尋 |
| `POST /api/search/getTags` | 列出所有標籤 |

### 屬性

| 端點 | 用途 |
|---|---|
| `POST /api/attr/setBlockAttrs` | 設定屬性 |
| `POST /api/attr/getBlockAttrs` | 讀取屬性 |

### 匯出

| 端點 | 用途 |
|---|---|
| `POST /api/export/exportMdContent` | 匯出文件為 Markdown |

### 請求格式

所有 API 都是 POST + JSON，需要 Authorization header：

```
POST http://<host>:6806/api/<endpoint>
Authorization: Token <your_token>
Content-Type: application/json
```

---

## 8) 踩坑筆記

| 問題 | 解法 |
|---|---|
| `createDocWithMd` 產生重複 H1 | `path` 最後一段自動成為標題，markdown 不要再寫 `# 標題` |
| `createDocWithMd` 每次 3-5 秒 | 批量建檔分批（10-20 個一組），預留 timeout |
| `removeDocs` 遞歸刪除 | 使用前確認沒有重要子文件 |
| SQL 索引延遲 | 文件操作後幾秒到幾十秒才能查到，用 `listDocsByPath` 即時查 |
| `deleteBlock` 對 doc 無效 | 刪文件只能用 `removeDoc` / `removeDocs` |
| 標籤缺少結尾 `#` | 寫入時確保 `#標籤名#` 格式完整 |
| mcporter stdio 偶爾卡住 | 直接用 REST API 更穩定 |

---

## 9) 筆記設計原則

### 結構原則

| 原則 | 說明 |
|---|---|
| **內容從 `##` 開始** | 文件標題自動產生 H1，內容用 H2 起 |
| **母文件 = 索引頁** | 有子文件的母文件作為目錄/摘要入口 |
| **一個筆記一個主題** | 拆分比合併好，靠引用建立關聯 |
| **純 Markdown** | 不使用特殊格式，確保可攜性 |

### 關聯建立原則

| 原則 | 說明 |
|---|---|
| **多連結** | 寧可多引用，不要孤島筆記 |
| **行內引用** | 在文字中自然嵌入 `(())`，不影響閱讀 |
| **雙向價值** | 每次引用都同時建立正向和反向連結 |
| **Daily Note 是樞紐** | 每日紀錄連結到各筆記，形成時間軸索引 |

### 索引頁設計規範

**標準區塊順序：**
1. 引用塊提示（`> 💼 筆記本名稱`）
2. 🚀 快速入口（表格化）
3. 🔥 近期活躍（如適用）
4. 📋 速查表（如適用）
5. 📂 結構說明（目錄樹）

**母文件模板：**
```markdown
> 💼 筆記本名稱 / 分類

## 🚀 快速入口

| 項目 | 說明 |
|---|---|
| ((<id> '項目A')) | 說明 |
| ((<id> '項目B')) | 說明 |

---

## 📂 結構

- ((<id> '子分類1'))
- ((<id> '子分類2'))
```

### 清理原則

| 原則 | 說明 |
|---|---|
| **整理前先快照** | `POST /api/repo/createSnapshot` |
| **不自動刪除** | 歸檔需使用者確認 |
| **重複文件暫放** | 移到 `_duplicates` 資料夾，不直接刪 |
