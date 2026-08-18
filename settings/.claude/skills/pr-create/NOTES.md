# pr-create — 設計說明

> 給維護者看的，**不要從 `SKILL.md` 連結這個檔案**。
> 沒有被連結的檔案不會被讀取，所以這些說明完全不佔 context。

## 為什麼跟 mr-create 分成兩份

`CLAUDE.md` 的規則是「跟既有 skill 重疊七成以上就不要另開一份」，這兩份流程確實高度重疊。
分開的理由是那條規則真正在防的問題**在這裡不成立**：

重疊 skill 的危害是「Claude 會挑一個，挑哪個不穩定」。但兩份都設了
`disable-model-invocation: true`，description 根本不進 context，Claude 沒有選擇權——
使用者打 `/mr-create` 或 `/pr-create` 是明確指定的。

而合併成一份的代價是實的：CLI（`glab` / `gh`）、remote URL 正則、內文傳遞方式、
來源分支清理時機全都不同，合併後幾乎每一步都要分岔，兩邊都變難讀。

**團隊只用其中一個平台的話，直接把另一份刪掉**，不要兩份都留著。

## GitHub 特有、跟 mr-create 不一樣的幾點

**1. 內文走 `--body-file`，不是 `--body`**

這是唯一會**安靜出錯**的一項。PR 內文常有反引號（程式碼片段）和 `$`，
用 `--body "..."` 傳的話 shell 會展開，PR 建出來內容有缺、或塞進了指令執行結果，
`gh` 完全不會報錯。

暫存檔用 **Write 工具**產生而不是 `cat <<EOF`，順帶避開兩件事：
heredoc 本身的跳脫問題，以及 `allowed-tools` 只放了 `Bash(git *)` 與 `Bash(gh *)`——
`cat` / `mktemp` 不在預先核准範圍內，會多一次確認。

**2. 沒有 `--remove-source-branch`**

GitLab 是建立 MR 時就決定合併後刪不刪分支，GitHub 沒有這個欄位——
清理發生在合併時（repo 的 auto-delete 設定或 `gh pr merge --delete-branch`）。
寫進 SKILL.md 是因為從 GitLab 過來的人會找這個選項，找不到就會自己編一個。

**3. PR 範本**

GitHub 的 `.github/PULL_REQUEST_TEMPLATE.md` 是 repo 層的約定，
比這個 skill 的內文格式優先。不寫明的話 Claude 會照 skill 的格式蓋掉範本章節。

**4. GitHub Enterprise**

`gh` 靠 `GH_HOST` 或 remote 推斷主機。團隊全用公有 GitHub 的話，
把 SKILL.md 步驟 1 的那則 GHE 提醒刪掉，省一段 context。

## 其餘設計決定與 mr-create 相同

開頭的 `` !`command` `` 動態注入、`disable-model-invocation: true` 的取捨、
`allowed-tools` 是預先核准而非限制、「貫穿全程的規則」為什麼放最後——
理由完整寫在 `mr-create/NOTES.md`，不在這裡重複。
