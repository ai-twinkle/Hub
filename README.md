<p align="center">
  <img src="./assets/logo.png" alt="Twinkle Hub" width="96" height="96" />
</p>

<h1 align="center">Twinkle Hub</h1>

<p align="center">
  <strong>台灣第一個 MCP Hub — 串接 data.gov.tw 全量，讓 AI 直接查台灣資料</strong>
</p>

<p align="center">
  <a href="https://hub.twinkleai.tw">https://hub.twinkleai.tw</a>
</p>

<p align="center">
  <a href="https://discord.com/servers/twinkle-ai-1310544431983759450">
    <img src="https://img.shields.io/badge/Discord-Twinkle%20AI-5865F2?logo=discord&logoColor=white" alt="Discord" />
  </a>
  <a href="https://www.facebook.com/groups/twinkleai">
    <img src="https://img.shields.io/badge/Facebook-Twinkle%20AI-1877F2?logo=facebook&logoColor=white" alt="Facebook" />
  </a>
  <a href="https://www.linkedin.com/company/twinkle-ai/">
    <img src="https://img.shields.io/badge/LinkedIn-Twinkle%20AI-0A66C2?logo=linkedin&logoColor=white" alt="LinkedIn" />
  </a>
  <a href="https://twinkleai.tw">
    <img src="https://img.shields.io/badge/Website-twinkleai.tw-FFD500?logo=googlechrome&logoColor=black" alt="Website" />
  </a>
</p>

<p align="center">
  <strong>🇹🇼 繁體中文</strong> · <a href="#english">🇬🇧 English</a>
</p>

---

## 這是什麼

Twinkle Hub 是 **MCP-as-a-Service** — 一支 Bearer token 接進 Claude Desktop / Cursor / OpenClaw / Hermes Agent loop / 任何能講 MCP 的 AI client，然後 AI 就能查：

- 📊 **49,343 個台灣政府開放資料集**(data.gov.tw 96.6% 全量，即時可查)
- 🏛️ **13.5 萬筆政府採購** (PCC 政府電子採購網每日同步)
- 🛠️ **32 個台灣專用 utility tool**(身分證 / 統編 / 民國年 / 地址 / 機構查……)

**alpha 期完全免費**。試試：

> 「幫我規劃從臺北出發、3 天 2 夜的家庭親子行程，熱門景點 + 臺中合法民宿 + 沿路夜市」
>
> 「列出今年金額最大的政府採購 10 件、廠商分布」
>
> 「113 學年度大學招生錄取率最高的 10 個科系」

---

## 安裝(10 分鐘)

### 1. 取得 API key

到 [https://hub.twinkleai.tw/login](https://hub.twinkleai.tw/login)，用 Google 或 GitHub 一鍵登入。第一次登入會自動發給你一支 virtual API key (`sk-...`)。

### 2. Claude Desktop

**方式 A：一鍵安裝（推薦）**

下載 [twinkle-hub.mcpb](./twinkle-hub.mcpb) → 雙擊開啟 → Claude Desktop 會跳出安裝視窗 → 貼上你 dashboard 拿到的 sk- key → 完成。5 個 `opendata-*` tool + 32 個 `twtools-*` 立刻出現、不用碰 JSON。

**方式 B：手動編輯 config**

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

加入：

```json
{
  "mcpServers": {
    "twinkle-hub": {
      "type": "http",
      "url": "https://api.twinkleai.tw/mcp/",
      "headers": {
        "Authorization": "Bearer sk-..."
      }
    }
  }
}
```

把 `sk-...` 換成你 dashboard 看到的 key，重啟 Claude Desktop。

### 3. Cursor

Cursor 用 `~/.cursor/mcp.json`，格式跟 Claude Desktop 一樣。儲存後 → Settings → MCP → reload。

### 4. Claude Code (CLI)

```bash
claude mcp add --transport http twinkle-hub https://api.twinkleai.tw/mcp/ \
  --header "Authorization: Bearer sk-..."
```

### 5. OpenClaw / Hermes / 自己的 agent loop

我們是 **MCP endpoint(開放協定，不是 SDK)**。任何能講 MCP streamable-http 的 client / fastmcp Python loop / 你 fork 的 runtime 都能直接用。連 OAuth flow 都沒有，就一支可隨時 revoke 的 long-lived bearer。換個 endpoint 就走。

### 6. Codex app

編輯 config:

- **macOS / Linux**: `~/.codex/config.toml`
- **Windows**: `%USERPROFILE%\.codex\config.toml`

加入：

```toml
[mcp_servers.twinkle-hub]
url = "https://api.twinkleai.tw/mcp/"
bearer_token_env_var = "TWINKLE_HUB_TOKEN"
```
再把你的 API key 設成環境變數。

macOS / Linux
```
export TWINKLE_HUB_TOKEN="sk-..."
```
Windows PowerShell
```
[Environment]::SetEnvironmentVariable(
  "TWINKLE_HUB_TOKEN",
  "sk-...",
  "User"
)
```
把 sk-... 換成你 dashboard 看到的 key，重啟 Codex，Twinkle Hub 的 tools 就會出現。

安裝完成後可用下面指令確認：
```
codex mcp get twinkle-hub
```
如果設定成功，至少會看到 5 個 opendata-* tools。

完整教學：[https://hub.twinkleai.tw/docs](https://hub.twinkleai.tw/docs)

---

## 社群

| 想做什麼 | 去哪 |
|---|---|
| **聊天 / 問問題 / 想跟其他人討論** | [Discord — Twinkle AI](https://discord.com/servers/twinkle-ai-1310544431983759450) |
| **回 feature request / 報 bug** | [Issues](../../issues/new/choose) |
| **看 alpha 期動態 / 新功能 ship** | [Facebook 社團](https://www.facebook.com/groups/twinkleai) |
| **關注公司 / 國際版** | [LinkedIn](https://www.linkedin.com/company/twinkle-ai/) |
| **官網** | [https://twinkleai.tw](https://twinkleai.tw) |
| **支援 / 商業聯繫** | <support@twinkleai.tw> |

---

## 想要的功能 / 想接的資料？

**請到 [Issues](../../issues/new/choose) 開單**,我們有兩個模板：

- 🎯 **Feature request** — 想要新功能、新 dataset、新 tool
- 🐞 **Bug report** — 文件錯、tool 不能用、回應有問題

> 一般討論、想法腦力激盪、「這樣設計對不對」的問題，**請去 [Discord](https://discord.com/servers/twinkle-ai-1310544431983759450)** 不用開 issue。Discord 反應比較快。

---

<a id="english"></a>

## English

<p align="center">
  <a href="#twinkle-hub">🇹🇼 繁體中文</a> · <strong>🇬🇧 English</strong>
</p>

**Taiwan's first MCP Hub — the full data.gov.tw catalog wired straight into your AI.**

Twinkle Hub is **MCP-as-a-Service**. Drop one Bearer token into Claude Desktop / Cursor / OpenClaw / Hermes / any MCP client, and your AI can query:

- 📊 **49,343 Taiwan government open datasets** (96.6% of data.gov.tw, live)
- 🏛️ **135,000+ government procurement records** (PCC daily sync)
- 🛠️ **32 Taiwan-specific utility tools** (national ID / company ID / ROC year / address / agency lookups…)

**Free during alpha.** Try:

> "Plan a 3-day family trip from Taipei — popular spots + licensed B&Bs in Taichung + night markets along the way."
>
> "Top 10 highest-value government procurements this year, plus vendor distribution."
>
> "Top 10 university programs by admission rate for the 113 academic year."

### Install (10 minutes)

**1. Get an API key**

Sign in at [https://hub.twinkleai.tw/login](https://hub.twinkleai.tw/login) with Google or GitHub. You'll get a virtual API key (`sk-...`) automatically on first login.

**2. Claude Desktop**

**Option A — One-click install (recommended)**

Download [twinkle-hub.mcpb](./twinkle-hub.mcpb) → double-click → Claude Desktop opens the install prompt → paste your `sk-` key → done. All 5 `opendata-*` tools + 32 `twtools-*` show up instantly, no JSON editing.

**Option B — Manual config**

Edit config:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Add:

```json
{
  "mcpServers": {
    "twinkle-hub": {
      "type": "http",
      "url": "https://api.twinkleai.tw/mcp/",
      "headers": {
        "Authorization": "Bearer sk-..."
      }
    }
  }
}
```

Replace `sk-...` with your dashboard key, restart Claude Desktop.

**3. Cursor**

Cursor uses `~/.cursor/mcp.json` — same shape as Claude Desktop. Save → Settings → MCP → reload.

**4. Claude Code (CLI)**

```bash
claude mcp add --transport http twinkle-hub https://api.twinkleai.tw/mcp/ \
  --header "Authorization: Bearer sk-..."
```

**5. OpenClaw / Hermes / your own agent loop**

We're an **MCP endpoint (open protocol, not an SDK)**. Anything that speaks MCP streamable-http — fastmcp Python loops, forked runtimes, custom clients — works out of the box. No OAuth flow either: it's a long-lived Bearer token you can revoke any time. Swap the endpoint and go.

**6. Codex app**

Edit config:

- **macOS / Linux**: `~/.codex/config.toml`
- **Windows**: `%USERPROFILE%\.codex\config.toml`

Add:

```toml
[mcp_servers.twinkle-hub]
url = "https://api.twinkleai.tw/mcp/"
bearer_token_env_var = "TWINKLE_HUB_TOKEN"
```

Then export your API key as an env var.

macOS / Linux:
```
export TWINKLE_HUB_TOKEN="sk-..."
```

Windows PowerShell:
```
[Environment]::SetEnvironmentVariable(
  "TWINKLE_HUB_TOKEN",
  "sk-...",
  "User"
)
```

Restart Codex. Verify with:

```
codex mcp get twinkle-hub
```

At least 5 `opendata-*` tools should appear.

Full docs: [https://hub.twinkleai.tw/docs](https://hub.twinkleai.tw/docs)

### Community

| What you want | Where |
|---|---|
| **Chat / questions / discussion with others** | [Discord — Twinkle AI](https://discord.com/servers/twinkle-ai-1310544431983759450) |
| **Feature requests / bug reports** | [Issues](../../issues/new/choose) |
| **Alpha shipping updates / release notes** | [Facebook group](https://www.facebook.com/groups/twinkleai) |
| **Company / international news** | [LinkedIn](https://www.linkedin.com/company/twinkle-ai/) |
| **Website** | [https://twinkleai.tw](https://twinkleai.tw) |
| **Support / business** | <support@twinkleai.tw> |

### Requesting features or datasets

**Open an issue** — we have two templates:

- 🎯 **Feature request** — new capability, dataset, or tool
- 🐞 **Bug report** — broken docs, tool errors, unexpected responses

> For general discussion, brainstorming, or "does this design make sense?" — **go to [Discord](https://discord.com/servers/twinkle-ai-1310544431983759450)** instead of opening an issue. Discord is faster.

---

## License

這個 repo 是 community feedback hub，沒有 source code。Twinkle Hub 服務本身的條款看 [/terms](https://hub.twinkleai.tw/terms)，資料授權看 [/licenses](https://hub.twinkleai.tw/licenses)。

This repo is a community feedback hub; no source code lives here. Terms of service for Twinkle Hub itself are at [/terms](https://hub.twinkleai.tw/terms); data licenses at [/licenses](https://hub.twinkleai.tw/licenses).
