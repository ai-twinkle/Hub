# Skills

這裡放 **Twinkle Hub 的 Agent Skills**：給 AI agent 看的用法指引。

Skill 不是程式，也不需要金鑰 —— 它是一份 Markdown，告訴 agent 「遇到這類問題，該用哪支工具、怎麼組合、答案要怎麼交代清楚」。沒有它工具也能用；有它比較不會踩坑。

## 怎麼用

**Claude Code / 支援 Agent Skills 的客戶端**

把資料夾放到你專案的 `.claude/skills/` 底下：

```bash
mkdir -p .claude/skills
curl -fsSL -o .claude/skills/tw-opendata-parliament/SKILL.md --create-dirs \
  https://raw.githubusercontent.com/ai-twinkle/hub/main/skills/tw-opendata-parliament/SKILL.md
```

**其他 agent**

直接把 `SKILL.md` 的內容當成 system prompt 的一部分，或丟給你的 agent 當參考文件。它就是純文字。

## 現有的 skill

| Skill | 用在什麼問題 |
|---|---|
| [`tw-opendata-parliament`](./tw-opendata-parliament/SKILL.md) | 台灣立法院：立委問政紀錄、院會表決、法律修法沿革與立法理由、委員會名單與召集委員、公報全文與提案訴求、黨團協商、議事影音逐字稿 |

## 這些指引在交代什麼

幾個反覆踩到、寫進 skill 的坑：

- **公報的片段只有前後各 150 字，提案的訴求寫在案子最後。** 只看片段下結論會把意思讀反 —— 例如看起來像「提案強化某項治療服務」，原文其實是「凍結該項預算、要求先提專案報告」。
- **「誰談過 X、實際說了什麼」要兩邊一起查。** 公報是官方正式紀錄（書面提案只存在這裡）；議事影音逐字稿是現場口語，而且可以直接跳到影片的那一秒自己確認。只查一邊都不完整。
- **議事影音逐字稿是 AI 語音轉寫，不是官方紀錄。** 引用時要標示，正式發言內容以立法院公報為準。
- **查無結果不等於沒發生。** 收錄範圍要看工具回傳的 `coverage`，不要替使用者下「他沒講過」這種結論。

## 授權

Skill 本身（這些 Markdown）：MIT。

工具回傳的**資料**另有授權，一律看回應裡標示的來源與授權條款 —— 立法院開放資料為「政府資料開放授權條款－第 1 版」。
