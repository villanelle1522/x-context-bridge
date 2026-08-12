# X Context Bridge GitHub Pages 發布檔

這個資料夾包含三種用途：

- `X_CONTEXT_BRIDGE.user.js`：電腦、Android 與 iPhone 的主要免費安裝檔。
- `CONSOLE_TEST.js`：可整份複製到 X 主控台的完整版。
- `index.html`：依裝置顯示安裝步驟，也提供一鍵複製主控台版。

API 分頁支援 Gemini 與 OpenAI 相容服務。OpenAI 相容模式預填官方 Base URL 與原本模型，既有使用者不用修改；自訂服務可另填 Base URL 與模型名稱。API Key 不會寫入發行檔或 GitHub。

「搜尋／備份」中的對話日曆採用本機捲動索引，不依賴 X 的內部 API。第一次按「掃描舊對話」時會自動回捲一次，只保存每天第一則訊息的定位資料，完成後回到最新訊息；日後畫面出現新日期時會自動加入索引。掃描期間會暫停背景翻譯，避免大量舊訊息同時翻譯造成卡頓或額外流量。

Notion 備份會用穩定的 `message-text-<UUID>` 合併舊版「文字＋畫面位置」記錄，舊版重複資料只會從下一次上傳中排除，不會自動刪除 Notion 頁面。確認新資料已正確合併後，再到 Notion 手動封存（Archive）舊頁面；下載到本機的 `.json` 備份則可在確認內容後自行刪除。

舊書籤檔仍保留供診斷與相容性研究，但不再是主要安裝方式。實測已確認 X 會以 CSP 阻擋 `GITHUB_BOOKMARKLET.txt` 從 GitHub Pages 載入外部程式。

GitHub Pages 主控台程式網址：

<https://villanelle1522.github.io/x-context-bridge/CONSOLE_TEST.js>

Userscript 安裝網址：

<https://villanelle1522.github.io/x-context-bridge/X_CONTEXT_BRIDGE.user.js>

## ChatGPT 網頁自動處理

完整自動流程必須使用 Violentmonkey 安裝版，因為同一支 userscript 需要同時在 X 與 `chatgpt.com` 執行。只把 `CONSOLE_TEST.js` 貼進 X 主控台無法跨網站注入 ChatGPT，也不能可靠地自動取回回覆。

從 v0.10.22 起，X 會先把任務寫入 Violentmonkey 的跨分頁共用儲存，再開啟 ChatGPT；ChatGPT 分頁會取出 prompt、填入、送出，等回覆停止生成後寫回結果，X 再自動套用譯文與語氣說明。v0.10.23 加入跨分頁變更監聽，避免 X 一直停在「正在自動送出」；v0.10.24 改以頁面狀態觸發送出確認，並把完成穩定等待由 1.8 秒縮短為 0.65 秒。更新舊版腳本時，必須接受新增的 `GM_setValue`、`GM_getValue`、`GM_deleteValue`、`GM_addValueChangeListener`、`GM_removeValueChangeListener` 權限，並確認 Violentmonkey 在 `x.com` 與 `chatgpt.com` 兩個網站都已啟用此腳本。

## v0.10.25 翻譯管線改進

參考 [Shinkansen](https://github.com/jimmysu0309/shinkansen) 對快取、混合語言批次、逾時與 SPA 效能的處理方式後，本專案以獨立實作加入以下改進：

- 韓文、英文與中文分開批次，避免 Google 對混合語言整批只做一次來源語言判斷而誤翻。
- 同批相同原文只送出一次；同一個仍在進行的請求也會共用，不再重複呼叫。
- 自動初譯使用最多 500 筆、90 天的可淘汰 LRU 快取；只保存 Google 草稿，不保存或取代人工修正。
- Google 請求加入 15 秒逾時、`Retry-After` 與帶隨機抖動的退避重試。
- 批次分隔符號遺失或回傳原文時，只補救受影響的項目，並限制同時請求數。
- 套用背景結果前再次核對原文與翻譯修訂版，晚到的結果不會蓋掉使用者剛做的更正。

Shinkansen 採用 Elastic License 2.0；本專案只參考公開設計與失敗案例，未直接複製其程式碼。

## v0.10.26 混合語言與 Clean 修正

- `Clean` 現在是可逆的總開關：立即還原 X 原生訊息與操作，保留本機資料，並留下低干擾的重新啟用按鈕。
- 同一則訊息混有韓文、英文與中文時，只翻譯使用者選定且不是目標語言的片段；既有目標語言、網址、`@帳號` 與 `#標籤` 保持原樣。
- 舊版產生的混合語言 Google 草稿不再沿用，會以新版分段方式重新產生；人工、AI 與匯入的修正仍會保留。

## v0.10.27 長訊息與引用備份修正

- 辨識 X 的 `data-testid="message-text-toggle"` 展開按鈕；掃描到長訊息時先展開，再更新完整原文快照。
- `message-text-toggle` 不再被誤認成訊息；展開後若原文由摘要變成全文，舊的自動翻譯會失效並依全文重譯，人工更正不會被清掉。
- 超過 Google GET 安全長度的單一訊息會沿換行、空白及標點切塊翻譯，再依原順序合併，不再把超長文字直接塞進同一個網址。
- 開始 Notion 備份時會先掃描當下掛載的所有訊息與引用框，包含不需要翻譯的中文引用。
- 所有引用備份及有引用關係的訊息會在每次增量同步重新執行 upsert，修復舊版曾經漏寫、但時間戳早於上次同步的紀錄。
- 若原訊息已不在 X，程式會以引用框仍保存的原文片段建立「引用備份」，並連回引用它的訊息；X 已從引用框本身省略的內容無法憑空還原，也不會被誤標成「確認刪除」。

不要上傳 Notion Token、同步密碼、任何 API Key、`.dev.vars` 或本機備份 JSON。

## 發布更新到 GitHub

以下流程在 PowerShell 執行。請把 `v0.10.12` 換成實際版本；發布前必須同步最新版 `main`，否則 Pull Request 可能無法合併。

```powershell
cd 'C:\Users\ASUS\Documents\Codex\2026-07-25\new-chat\outputs\x-context-bridge\github-pages-release'

git status -sb
node --check CONSOLE_TEST.js
node --check X_CONTEXT_BRIDGE.user.js

git fetch origin
git switch agent/publish-v0.10.12
git rebase origin/main
```

如果 rebase 沒有衝突，更新遠端分支：

```powershell
git push --force-with-lease -u origin agent/publish-v0.10.12
```

如果尚未建立 Pull Request：

```powershell
gh pr create --repo villanelle1522/x-context-bridge --base main --head agent/publish-v0.10.12 --title "Publish v0.10.12" --body "發布 v0.10.12-test。"
```

確認 PR 可以合併後發布：

```powershell
gh pr merge --repo villanelle1522/x-context-bridge agent/publish-v0.10.12 --squash --delete-branch
```

### PR 顯示無法乾淨合併時

1. 先執行 `git status`，確認是否正在 rebase，以及列出的衝突檔案。
2. 比較 `main` 與本次版本的內容後逐一解決衝突；不要直接把所有檔案全部選成同一側，以免覆蓋最新版功能。
3. 每個衝突檔案都處理完後執行：

```powershell
git add CONSOLE_TEST.js X_CONTEXT_BRIDGE.user.js index.html README.md
git rebase --continue
node --check CONSOLE_TEST.js
node --check X_CONTEXT_BRIDGE.user.js
git push --force-with-lease -u origin agent/publish-v0.10.12
```

4. 回到原本的 PR 再執行 `gh pr merge`，不需要重建另一個 PR。

若 Git 要求開啟編輯器而卡住，可先執行 `git commit --no-edit`，再執行 `git rebase --continue`。若 `gh` 顯示尚未登入，先執行 `gh auth login -h github.com`。發布完成後，GitHub Pages 通常需要短暫時間才會更新；重新整理首頁並確認顯示的新版本號。

### Codex 發布卡住時的備援流程

如果 Codex 的終端機無法連到 GitHub、`gh` 權限失效、GitHub connector 回傳 403，或 PR 一直顯示無法合併，不要反覆要求使用者登入，也不要叫使用者在錯誤目錄執行 `git push`。先確認目前資料夾必須是：

```powershell
C:\Users\ASUS\Documents\Codex\2026-07-25\new-chat\outputs\x-context-bridge\github-pages-release
```

如果仍然無法用 Git/PR 發布，使用已登入的 GitHub 網頁介面直接更新 `main`。這是 v0.10.12 實際成功的方法：

1. 用 Codex 內建瀏覽器或使用者已登入的瀏覽器打開：

```text
https://github.com/villanelle1522/x-context-bridge/upload/main
```

2. 上傳並覆蓋這四個發行檔：

```text
CONSOLE_TEST.js
X_CONTEXT_BRIDGE.user.js
index.html
README.md
```

3. Commit summary 使用類似 `Publish v0.10.12`，說明欄簡短寫清楚這次修正。
4. Commit 直接送到 `main`。
5. 如果先前已有卡住的 PR，確認 `main` 已更新後再關閉舊 PR，不需要繼續修同一個壞掉的 PR。
6. 最後打開以下網址驗證 GitHub Pages 是否已顯示新版本：

```text
https://villanelle1522.github.io/x-context-bridge/?commit=<commit前8碼>
```

注意：這個方法只適合發行檔已經在本機檢查過、且使用者明確要發布時使用。不要把 API Key、Notion Token、同步密碼、`.dev.vars` 或本機備份 JSON 上傳。
