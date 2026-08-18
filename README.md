# Claude Code 團隊共用設定範本

一套讓整個團隊共用同一份 Claude Code 設定的骨架。成員 clone 後把 Claude Code 指向 `settings/`，指引、規則與 skill 一次到位。

> **適用環境：Claude Code + macOS。**
> 內容為可直接套用的範本，實際使用時請依團隊情況修改。
> 此 repo 是文章〈從一個專案到整個團隊：AI 協助開發流程的實戰心得〉的配套範例。

## 目錄結構

```
team-claude-settings/
├── CLAUDE.md                          ← 維護此 repo 時的指示（不分發）
├── README.md                          ← 本檔案
├── .claude/skills/onboard/SKILL.md    ← 改造範本用的引導流程（不分發）
└── settings/                          ← 真正要分發的團隊設定
    ├── CLAUDE.md                      ← 常駐：每次 session 都載入
    └── .claude/
        ├── rules/                     ← 條件載入：依 paths 觸發
        │   ├── testing.md
        │   └── infra-facts.md         ← 線上環境的架構事實
        └── skills/                    ← 叫用才載入
            ├── mr-create/SKILL.md     ← 只有人能叫（disable-model-invocation）
            ├── pr-create/SKILL.md     ← 同上，GitHub 版
            └── create-tag/SKILL.md    ← 開放 AI 叫用，靠內部閘門防守
```

`settings/` 底下就是三層**載入時機**不同的設定，詳見 [CLAUDE.md](CLAUDE.md)。

三層**都由同一個 `--add-dir` 生效**——不需要連結腳本，也沒有任何額外安裝步驟。

沒有 `commands/` 這一層是刻意的——custom command 已併入 skill，兩者行為相同，新的一律寫成 skill。

`mr-create` / `pr-create` 與 `create-tag` 刻意採取相反的叫用策略，對照著看：

| | `mr-create`／`pr-create` | `create-tag` |
|---|---|---|
| `disable-model-invocation` | `true` | 不設（預設） |
| `Triggers:` 關鍵字 | 不寫（Claude 讀不到） | 寫 |
| description 是否常駐 context | 否 | 是（約一行） |
| 主要防線 | 叫用控制 | 內部確認閘門 |

副作用更大的 `create-tag` 反而開放 AI 叫用，是因為「打個 tag」「建版」是團隊真的會講的話——關掉等於把最常用的入口拆掉。判準不是「有沒有副作用」，是**「需不需要用講的觸發」＋「內部有沒有閘門」**。

---

## 有哪些 skill 可用

從這裡開始，**這張表是唯一需要先看過的東西**。

| 指令 | 做什麼 | 怎麼觸發 |
|---|---|---|
| `/mr-create` | 建立 GitLab Merge Request：確認分支、查工作單、預覽草稿、建立 MR，可選擇發審核通知到團隊頻道 | **只能打指令** |
| `/pr-create` | 建立 GitHub Pull Request：流程同上，走 `gh` CLI | **只能打指令** |
| `/create-tag` | 建立並推送 git tag：算下一個版號、列出將被包含的 commit、確認後才建立 | 打指令，或直接說「打個 tag」「建版」 |

「只能打指令」的那些設了 `disable-model-invocation`，用講的叫不動——這是為了不佔 context 的取捨，代價就是**你得先知道它存在**。所以這張表要維護。

> 新增、改名或刪除 skill 時一併更新這張表。**過期的清單比沒有清單更糟**：
> 照著打一個已經不存在的指令，只會得到「查無此指令」，然後不會回報。

---

## 開始使用

### 1. Clone

```bash
git clone https://github.com/burgess1109/team-claude-settings.git
```

### 2. 改造成你們團隊的版本

```bash
cd team-claude-settings
claude
```

然後執行：

```
/onboard
```

它會逐區塊引導你填寫或**刪除**範本內容——語言、commit 慣例、對照表、環境、架構事實、測試慣例，以及範例 skill 的去留，最後驗收沒有殘留的 `<...>` 佔位符。

> **刪除比填寫重要。** `settings/CLAUDE.md` 每次 session 都會載入，留著用不到的內容是全團隊每天在付的錢。`/onboard` 會主動建議你刪。

注意事項：

- 改完記得把 git remote 指向團隊自己的 repo
- onboard 會改寫整個 `settings/` 底下的內容（包含 `settings/CLAUDE.md`），**請務必逐項確認產出是否正確**——這份設定之後每個成員每天都會載入，錯的規則會被全團隊當成權威
- 特別檢查 `infra-facts.md`：如果對某條規則沒把握，寧可先留白也不要寫錯。**錯的架構事實比沒有更危險**，它會讓人和 AI 都停止查證
- `/onboard` 不會被 AI 自動觸發，要自己打。中途離開也沒關係，可以分次做完
- 改造完成後，`.claude/skills/onboard/` 可以刪掉，它是一次性的引導流程
- **但根目錄的 `CLAUDE.md` 要留著。** 撰寫慣例與「什麼樣的 skill 值得共用」的判準都在裡面，日後成員新增共用 skill 時都靠它。它不在 `settings/` 底下，不會分發給成員，留著不佔任何人的 context

### 3. 讓成員套用

在 `~/.zshrc` 加入：

```bash
function claude() {
    CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 \
    command claude --add-dir "<你的實際路徑>/settings" "$@"
}
```

套用：`source ~/.zshrc`

**這一行就完成全部三層的載入**——`CLAUDE.md`、`rules/` 與 `skills/` 都會生效。

> `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` 是必要的——沒設的話 `--add-dir` 指到的 `CLAUDE.md` 不會載入（`skills/` 則不受影響，一律會載入）。
>
> 包成 function 而不是 alias，是為了讓 `claude --resume`、`claude -p "..."` 這些帶參數的用法都能正常運作。

### 4. 驗證

在任一專案內啟動 `claude`：

- `/context` → **Memory files** 底下應該看得到團隊的 `CLAUDE.md`
- `/` 選單 → 應該看得到團隊的 skill

看不到就是第 3 步沒生效。

---

## 日後更新

```bash
git -C <你的實際路徑> pull
```

**pull 完就生效**，三層都是。沒有腳本要重跑，也沒有 symlink 會失效。

---

## 貢獻你的 skill

好用的 skill 本來就散在各人的 `~/.claude/skills/` 底下。**要不要拿出來共用由你自己評估**（判準見下），決定要共用就照這個 repo 的格式新增進來：

1. 在 `settings/.claude/skills/<name>/` 開目錄，**`<name>` 就是未來的指令名稱**（`mr-create/` → `/mr-create`，`name` 欄位不決定這件事）
2. **去掉個人化的部分**：絕對路徑、你自己的 alias、只有你機器上有的工具與目錄
3. `description` **一律用單引號整段包住**，並寫上團隊真的會講的口語觸發詞
4. 確認觸發詞沒跟既有 skill 撞
5. 在真實專案裡跑一次：`/` 選單看得到，用講的也叫得動
6. 更新上面那張 skill 清單
7. 開 MR

第 2 步和第 5 步最容易漏。**自己在用的 skill 只有自己是使用者**，所以往往帶著只有本機成立的假設，而且叫用模式通常沒認真設過。

判準與撰寫慣例見 [CLAUDE.md](CLAUDE.md) 的「新增共用 skill 前」——什麼樣的 skill 值得共用、跟既有的重疊時怎麼辦、改寫成共用版本要拿掉哪些東西。frontmatter 各欄位也在同一份檔案。

你個人 `~/.claude/` 底下那份要留、要刪還是改名，自己決定就好，這個 repo 不管。

---

## 常見問題

- 跟各專案自己的 `CLAUDE.md` 會不會衝突？ 
不會衝突。優先權：專案層 > 團隊層（此 repo）> 個人層 `~/.claude/`。團隊設定應該讓路給專案設定。

- `/context` 看不到團隊 `CLAUDE.md`？ 
確認 `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` 有設，且 `--add-dir` 指到的是 `settings/` 而不是 repo 根目錄。

- 改了團隊設定，其他人什麼時候會拿到？
他們 `git pull` 之後。這是刻意的——分發機制越笨越好，因為它壞掉時是全團隊一起壞。

- 可以只用其中一部分嗎？
可以。三層彼此獨立，`/onboard` 會逐區塊問你要不要保留，用不到的整份刪掉即可。刪掉某一層不影響其他層。

- 一定要用 `/onboard` 嗎？
不用，它只是引導流程。你也可以直接編輯 `settings/` 底下的檔案，把 `<...>` 佔位符換掉、用不到的刪掉。`/onboard` 的價值在於它會提醒你哪些該刪、哪些細節容易寫錯。
