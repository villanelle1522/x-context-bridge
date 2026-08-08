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
