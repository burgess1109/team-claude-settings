> **這是範本檔案**，複製到你的團隊設定 repo 根目錄後再依實際情況修改。

# 團隊 AI 共享設定 — 維護指引

> 此檔案只在**維護這個 repo 時**生效，不會分發給團隊成員。
> 要讓所有人載入的內容，一律寫在 `settings/` 底下。

## 這個 repo 是什麼

集中管理團隊共用的 AI 輔助開發設定。成員 clone 後把工具指向 `settings/`，即套用全套團隊設定。

**它不是**：

- 不是放專案程式碼的地方
- 不是放個人偏好的地方（那是 `~/.claude/`）

---

## 我要加的東西該放哪？

先問一個問題：**這段內容什麼時候需要在 context 裡？**

| 何時需要 | 路徑 | 代價 |
|---|---|---|
| 幾乎每次 session | `settings/CLAUDE.md` | **最貴**，每人每次 session 都會讀取 |
| 只在碰到特定檔案時 | `settings/.claude/rules/<topic>.md`（加 `paths` frontmatter） | 中，只有相關的 session 會讀取 |
| 使用者或 AI 叫用時 | `settings/.claude/skills/<name>/SKILL.md` | 低，不叫用就不載入 |

判斷不出來時，**預設放 skill**，它代價最低，而且事後要往上搬（skill → 常駐）比往下搬容易得多。

> **不要新增 `.claude/commands/`。** custom command 已併入 skill，同名檔案兩邊行為相同，
> 而 skill 多了「可帶目錄放輔助檔案」與「叫用控制」兩項能力。舊 command 檔案仍可運作，
> 但新的一律寫成 skill。

### 幾條硬規則

- `settings/CLAUDE.md` 目標 **150 行以內**。接近上限時不要壓縮文字，要往下層搬
- 三層全部放在 `settings/` 底下，由**同一個 `--add-dir` 生效**。不要另外做連結腳本或 symlink——
  `--add-dir` 對 `.claude/skills/` 是明文的例外，會自動載入
- 這份設定**專門給 Claude Code**，所以路徑一律用 `.claude/`。
  不追求換工具可攜，換來的是零安裝步驟
- skill 主檔只留判斷邏輯與主流程，長內容（對照表、排錯清單）拆到 `references/*.md`，在主檔用一句話說明何時去查
- 每個 skill 都要想清楚叫用模式（見〈Skill 的叫用控制〉），不要用預設值了事

---

## 新增共用 skill 前

共用 skill 由成員自己評估後照這個 repo 的格式新增。**先評估值不值得共用**——上一節回答的是「放哪一層」，那已經預設了要共用。

| 情況 | 判斷 |
|---|---|
| 只有一兩個人的工作流程用得到 | 不共用，留在 `~/.claude/` |
| 綁定單一專案的架構或指令 | 不共用，放該專案的 `.claude/` |
| 跟既有 skill 重疊七成以上 | 不要另開一份，改既有那份並補上自己的觸發詞 |
| 跨專案通用、多數人都會用到 | 共用 |

**兩份高度相似的 skill 併存是最糟的結果**：Claude 會挑一個，挑哪個不穩定，使用者的體驗是「這功能有時候會有時候不會」，而且不會回報成 bug。

從自己在用的版本改寫成共用版本時：

- 拿掉絕對路徑（`/Users/<name>/...`）與個人 alias
- 拿掉只有自己機器上有的工具假設，或在 skill 裡寫明前置需求
- 寫死的環境值改成引用 `settings/CLAUDE.md` 的對照表
- **重新選一次叫用模式。** 自己用的時候作者就是唯一使用者，這個欄位通常沒認真設過
- 觸發詞從「自己會講的話」擴成「團隊會講的話」

個人 `~/.claude/` 底下那份要留、要刪還是改名，是各人自己的事，**不在這個 repo 的範疇**，不要主動提議去動它。

新增完更新 `README.md` 的 skill 清單，那是成員唯一的入口。

---

## Skill 的叫用控制

預設情況下，你和 Claude 都能叫用任何 skill。兩個 frontmatter 欄位可以改變這件事：

| Frontmatter | 你可叫用 | Claude 可叫用 | description 是否常駐 |
|---|---|---|---|
| （預設） | ✅ | ✅ | 是 |
| `disable-model-invocation: true` | ✅ | ❌ | **否**（完全不佔 context） |
| `user-invocable: false` | ❌ | ✅ | 是 |

### 怎麼選

**不要用「有沒有副作用」當判準**，那會誤傷。`disable-model-invocation` 擋掉的是**所有** AI 叫用，包含「使用者用講的」那一種——設了之後，使用者必須打 `/skill-name`，口語觸發詞也一併失效。

實際判準：

| 這個 skill 是⋯ | 設定 |
|---|---|
| 使用者本來就會主動發起、沒有口語觸發需求 | `disable-model-invocation: true` |
| 有副作用**但**需要口語觸發，且內部有確認閘門 | 預設（不設），靠閘門防守 |
| 沒有副作用 | 預設（不設） |
| AI 該在相關時主動套用的背景知識，但對使用者不是一個動作 | `user-invocable: false` |

先問「這個 skill 需不需要用講的觸發」，再問「內部有沒有確認閘門」。兩個答案都是否，才輪到 `disable-model-invocation`。

**真正在擋事故的通常是 skill 內部的確認閘門，不是叫用控制**——那道防線不管誰叫用都有效。

### 這些欄位是 Claude Code 的擴充

`disable-model-invocation`、`user-invocable`、`allowed-tools`、`context: fork` 都是 Claude Code 對 Agent Skills 標準的**擴充**，不在標準本身裡。

這份設定專門給 Claude Code，所以放心用。但如果你把某個 skill 複製到別的 agent 工具，要知道這些欄位會被**靜默忽略**——不會報錯，但叫用控制就消失了，你標成「只有人能叫」的部署流程會變回 AI 可以自己觸發。

---

## 撰寫慣例

### frontmatter 欄位參考

`SKILL.md` 開頭 `---` 之間的 YAML。全部選填，但 `description` 一定要寫。

| 組別 | 欄位 | 作用 |
|---|---|---|
| **識別** | `name` | 清單顯示名稱 |
| | `description` | Claude 判斷是否叫用的依據 ＋ `/` 選單說明 |
| | `when_to_use` | 補充觸發情境，接在 description 後面（共用 1536 字元上限） |
| **參數** | `argument-hint` | autocomplete 提示，純 UI，不驗證 |
| | `arguments` | 宣告具名參數，內文可用 `$issue` 取代 `$1` |
| **叫用控制** | `disable-model-invocation` | `true` = 只有人能叫 |
| | `user-invocable` | `false` = 只有 AI 能叫，從 `/` 選單隱藏 |
| **工具權限** | `allowed-tools` | 預先核准，免確認 |
| | `disallowed-tools` | 把工具移出可用池 |
| **執行環境** | `model` | 這個 skill 生效期間改用的模型 |
| | `effort` | 推理強度：`low` / `medium` / `high` / `xhigh` / `max` |
| | `context` | 設 `fork` 則丟到獨立 subagent 跑，不帶對話歷史 |
| | `agent` | 搭配 `context: fork`，指定 subagent 型別 |

**三個會咬人的地方：**

- **`name` 不決定指令名稱**，目錄名才決定。`skills/mr-create/` 就是 `/mr-create`，`name` 只是顯示標籤
- **`allowed-tools` 是預先核准，不是限制**。列進去的工具在該輪免確認，但沒列的照樣可用（走原本權限設定）。要真的擋要用 `disallowed-tools`。授權只有一輪，送出下一則訊息就失效
- **`agent` 沒搭配 `context: fork` 是無效設定**，不會報錯，只是什麼都不做

內文的字串替換：`$ARGUMENTS`、`$1`、`${CLAUDE_SKILL_DIR}`、`${CLAUDE_PROJECT_DIR}`。
另外 `` !`command` `` 會在 Claude 看到內容**之前**先執行，輸出直接嵌進 prompt——用來預先取得 git 狀態之類的資訊，可以省掉一輪 tool call。範例見 `mr-create` 與 `create-tag`。

### YAML frontmatter 的引號

`description` 含冒號、逗號、引號等特殊字元時，**必須用單引號整段包住**：

```yaml
# 錯誤：冒號會破壞 YAML 解析
description: 建立 tag 流程。Triggers: "打tag", "建tag"

# 正確
description: '建立 tag 流程。Triggers: "打tag", "建tag"'
```

壞掉的方式很陰險：**不是整個 skill 消失，而是 metadata 變空**——
`/skill-name` 照常能用，但 Claude 沒有 `description` 可比對，**自動觸發整個失效**。

自己測斜線指令會通過，所以不會發現；習慣用講的那些人則是從此觸發不了，而且多半不會回報。

`claude --debug` 看得到 parse error，但前提是你想到要去看。加單引號零成本，一律加。

### 內文怎麼組織

Skill 載入後，內容會**常駐 context 到 session 結束**，而且 Claude 不會重讀檔案。三個推論：

**（1）每一行都是重複成本。** 官方原則：

> State what to do rather than narrating how or why.

寫「必須詢問使用者要從哪個分支打 tag」，不要寫一段話解釋為什麼分支很重要。
只有「會改變模型判斷」的理由才留（例如「minor 還是 patch 由人決定」讓它知道不能自己選）。

**（2）貫穿全程的約束要獨立成段，不要埋進編號步驟。**
寫在「步驟 6」裡的規則，流程走到步驟 8 時讀起來像「已經做完的一步」。
把跨步驟或流程結束後仍有效的規則拉到最後，用「以下規則在整個流程中持續有效」開頭。

不是每個 skill 都需要這段——純參考型、單步驟、無副作用的 skill 加了只是儀式。

**（3）維護者說明放 `NOTES.md`，不要放 HTML 註解。**
`CLAUDE.md` 的 HTML 註解會被剝除，但**官方沒有保證 skill 檔案也一樣**，保守假設是不會。
做法是在 skill 目錄放一個 `NOTES.md` 並**不要從 `SKILL.md` 連結它**——
沒被連結的檔案不會被讀取，等於零成本的維護者文件。

```
my-skill/
├── SKILL.md      ← 必要，導覽與主流程（官方建議 500 行以內）
├── NOTES.md      ← 設計說明，不連結，不進 context
├── reference.md  ← 查表型內容，SKILL.md 指路後才載入
└── scripts/      ← 被「執行」不是被「載入」，原始碼不進 context
```

`reference.md` 這類附屬檔**沒有 frontmatter**，它們只是被 Read 的普通檔案。
要在 `SKILL.md` 裡寫清楚「裡面裝什麼、何時該讀」，否則 Claude 不會去開。

有確定演算法的工作（版號計算、格式轉換、解析）優先寫成 `scripts/` 底下的腳本，
搭配 `${CLAUDE_SKILL_DIR}` 定位。**不佔 context，而且確定性計算本來就不該交給模型推。**

### Skill 的 triggers

寫團隊真的會講的口語，不要只寫正式名稱。`create tag` 沒人會打，`"打tag" / "建tag" / "tag一下" / "建版"` 才會中。

**觸發不了的 skill 等於不存在。**

> 例外：設了 `disable-model-invocation: true` 的 skill **不需要寫 triggers**。
> 它的 description 根本不進 context，Claude 看不到那些關鍵字——只會顯示在 `/` 選單裡。
> 這種 skill 的 description 要寫給**人**看，不是寫給比對用的。

### 有副作用的流程

Skill 裡必須寫明哪些步驟需要人確認，不要留給 AI 自己判斷。AI 傾向把事情做完，不寫明它就會補一個合理的預設值，而合理的預設值有機率是錯的。

寫法範例：

```markdown
> **注意：** 下表為慣例參考，各 repo 可能不同。
> 執行時**必須由使用者明確確認**，不自行代入。
```

---

## 怎麼盤點架構事實

`settings/.claude/rules/infra-facts.md` 是空的填空框架，這節說明怎麼填。

它要收的**不是**「我們的架構長什麼樣」，而是「**AI 的預設答案在哪裡會出錯**」。
跟社群主流一致的東西一律不寫——寫了只是在付 context 的錢買 AI 本來就知道的事。

### 先問四個問題

去找負責基礎設施的人問：

1. 線上有哪些服務是「**相容版**」而不是原版？
2. 有哪些技術選型，**社群主流跟我們用的不一樣**？
3. 有哪些東西**本機跑得起來、線上不支援**？
4. 有哪些平台有**自己的工作流程或護欄**，會反過來限制我們的設計？

前三題找出「AI 會用錯的 API」，第四題找出「AI 會做錯的架構決定」——後者貴得多。

### 三種型態與常見例子

以下是用來**幫你想起自己有什麼**的提示，不是要照抄的內容。

**型態一：主流 ≠ 我們的主流**

同一個問題有兩套做法，社群風向偏 A，但我們全部建立在 B 上。AI 會規劃 A。

- Ingress：K8s Gateway API vs Istio Gateway CRD vs nginx-ingress
- Secrets：External Secrets vs Sealed Secrets vs 雲端原生方案
- 排程：K8s CronJob vs 工作流引擎

**型態二：相容 ≠ 等同（少了東西）**

宣稱相容，實際是子集。AI 訓練資料裡原版文件量遠大於衍生版，一定會回歸原版。

- MongoDB 相容的託管服務——部分 aggregation 行為與函式不支援
- ClickHouse 的商用衍生版——連線協定與可用引擎不同
- PostgreSQL wire-compatible 的分散式資料庫——鎖語意、序列、觸發器行為有差
- **Elasticsearch → OpenSearch**：官方 ES client 內建產品檢查，**會直接拒絕連線**。
  AI 預設會裝 `elasticsearch` 套件，必須指定改用 fork 的 client
- **Serverless / HTTP 介面的 Redis**：不支援 blocking 指令（`BLPOP`）與 pub/sub。
  AI 會寫出以 `BLPOP` 為基礎的佇列，語法完全正確但跑不動
- **S3 相容物件儲存**：storage class、bucket ACL、multipart 上限與原版有差

**型態三：託管平台有自己的世界觀**

不只是子集——它同時少了東西、多了東西、多了限制，而且影響的是**架構決定**而非某個 API。

- MySQL 相容的託管平台：不支援外鍵、migration 走資料庫分支與 deploy request、
  單次欄位異動有上限。這三項合起來會改變 schema 怎麼設計、參照完整性由誰保證
- **Serverless 執行環境（Lambda / Cloud Run 這類）**：回應送出後 CPU 被凍結或節流，
  `go func()` 之類的背景工作會被靜默中斷；本地磁碟不保證存在；縮到零時行程內排程不會觸發。
  AI 會寫出在常駐服務上完全正確的程式碼，而它**在這裡會安靜地掉資料**

### 寫的時候

1. **寫「不是真的 X」這句否定句。** 只寫「MySQL 相容」時，注意力會壓在「MySQL」上
2. **正面表述加反面禁止一起寫。** 「只支援 HTTP 連線」要配「不可使用 native TCP driver」
3. **有兩種標準時要寫為什麼選這個。** 邊界情況才有依據判斷，日後評估遷移也用得上
4. **測試環境與線上的能力落差要明講。** 「本機測試通過不代表線上部署得了」
5. **`paths` 要涵蓋產出物不只來源。** 規劃 migration 時可能先開 ORM model 檔案——
   **寧可多觸發幾次，也不要在該出現的時候沒出現**

## 改完之後

1. **本機驗證**：在一個真實專案裡啟動，用 `/context` 確認檔案有載入；
   新增 skill 的話用 `/` 選單確認看得到
2. **新增、改名或刪除 skill 時，同步更新 `README.md` 的 skill 清單**——
   清單跟實際不符時，新人會照著打一個不存在的指令
3. **commit**：Conventional Commits，`<type>(<scope>): <description>`
4. **push**
5. **通知成員 `git pull`**：pull 完就生效，沒有其他步驟

第 1 步不要跳過。設定類的錯誤不會 crash，只會靜默失效，而失效的東西沒人會回報。

---

## Review 這個 repo 的改動時

`settings/` 底下的任何改動都會**立刻影響所有人**，所以額外注意：

- **新增常駐內容**：真的每次 session 都需要嗎？能不能降到 rules 或 skill？
- **改動既有規則**：有沒有專案的慣例會因此衝突？團隊層應該讓路給專案層
- **新增 skill**：triggers 會不會跟既有 skill 打架？功能是不是跟現有的重疊？
- **刪除或改名 skill**：成員 `git pull` 後舊的就消失了，如果有人習慣打那個指令，記得在通知裡講
- **任何 skill 的增刪改名**：`README.md` 的 skill 清單有一起更新嗎？漏掉不會有任何錯誤訊息
