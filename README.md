# X Context Bridge GitHub Pages 發布檔

這個資料夾包含三種用途：

- `X_CONTEXT_BRIDGE.user.js`：電腦、Android 與 iPhone 的主要免費安裝檔。
- `CONSOLE_TEST.js`：可整份複製到 X 主控台的完整版。
- `index.html`：依裝置顯示安裝步驟，也提供一鍵複製主控台版。

API 分頁支援 Gemini 與 OpenAI 相容服務。OpenAI 相容模式預填官方 Base URL 與原本模型，既有使用者不用修改；自訂服務可另填 Base URL 與模型名稱。API Key 不會寫入發行檔或 GitHub。

舊書籤檔仍保留供診斷與相容性研究，但不再是主要安裝方式。實測已確認 X 會以 CSP 阻擋 `GITHUB_BOOKMARKLET.txt` 從 GitHub Pages 載入外部程式。

GitHub Pages 主控台程式網址：

<https://villanelle1522.github.io/x-context-bridge/CONSOLE_TEST.js>

Userscript 安裝網址：

<https://villanelle1522.github.io/x-context-bridge/X_CONTEXT_BRIDGE.user.js>

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
