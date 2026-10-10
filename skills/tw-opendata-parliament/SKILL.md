---
name: tw-opendata-parliament
description: '台灣立法院(開放國會):某位立委提過什麼案、怎麼投票、出席率、主提案修了哪些法;某一案院會表決誰贊成誰反對、各黨怎麼投;某部法律每一條的修法沿革與立法理由;某一屆有哪些立委;某個議題在立法院公報(官方逐字紀錄)裡怎麼被講的、某一件公報提案到底要求什麼(讀公報原文);某個法案的朝野協商 / 黨團協商有哪幾場、談了哪幾案、紀錄在公報哪幾頁。某位立委或某個議題在議場上**實際說了什麼**(議事影音逐字稿,可跳到影片的第幾秒;與公報併用)。用在使用者問到立法委員、院會表決、黨團投票、修法歷程、公報發言、公報提案內容、黨團協商紀錄的時候。資料來自立法院官方開放資料與國會圖書館法律系統。'
metadata:
  version: "3.0.0"
  surface: v3
  language: zh-tw
---

# 台灣立法院:委員、委員會名單、表決、法律沿革、公報、黨團協商

這一組工具,各管一件事:

| 使用者想知道 | 工具 | 給什麼 |
|---|---|---|
| **某位立委**提過什麼案、發言、怎麼投、出席率、主提案的修法;**某一屆**有哪些立委 | `tw_ly_legislator_profile` | 姓名(或只給屆別) |
| **某一案**院會表決誰贊成、誰反對、各黨怎麼投 | `tw_ly_vote_lookup` | 案由關鍵字 → `vote_id` |
| **某部法律**每一條歷次怎麼改、立法理由、最近一次修法的讀會進度 | `tw_ly_law_history` | 法律名稱(可加條號) |
| **某個議題**在公報(官方逐字紀錄)裡怎麼被講的、誰講的 | `tw_ly_gazette_search` | 關鍵詞(可加委員姓名、日期) |
| **那一案到底要求什麼**(公報原文、整案讀完) | `tw_ly_gazette_read` | `doc_id` + `at` 或 `item_seq`(都從 search 的結果拿) |
| **某個議題在議場上怎麼被講的**(跨委員找發言片段,AI 轉寫逐字稿) | `tw_ly_ivod_search` | 關鍵詞(可加委員姓名、日期、`role`) |
| **朝野協商 / 黨團協商怎麼談的**(哪一案在協商桌上定版) | `tw_ly_caucus_consultation` → 再用 `tw_ly_gazette_read` 讀全文 | 關鍵詞(議案名)、`term`、日期;都不給就回最近幾場 |
| **某一屆有誰** / **某委員會有哪些委員** / **誰是召集委員** / 某位委員**待過哪些委員會** | `tw_ly_roster` | `term`、或 `committee`(+ 選用 `term` / `session_period`)、或 `legislator` —— **三種互斥** |

依主題搜尋全部議案目前**沒有提供**;不要拿其他資料集去推測。

## 回答前一定要守的解讀規則

這些規則比任何一個範例都重要 —— 違反了,答案會「看起來正常但是錯的」:

1. **0 筆不代表沒做過。** 每個區塊都附 `coverage`(來源、資料範圍、說明),先讀再下結論。官方質詢資料很少(第 11 屆全部委員合計約 152 列),質詢 0 筆極常見;表決從第 10 屆第 7 會期起才有。
2. **表決以官方票數為準,名單人數要對得上。** `official` 是官方公布的贊成 / 反對 / 棄權票數,`named` 是名單上對得到委員的人數。兩者一致時直接講結果;不一致時(差額在 `unmatched`,對不到名錄的票在 `unresolved`)兩個都要講,不要把名單人數說成官方結果。各黨統計 `by_party` 依名單計。
3. **黨籍是委員名錄的最近一次登記**,不一定是表決當時的黨籍;有人中途退黨時要提醒。
4. **出席率只算院會**,而且官方名單只有「出席委員」,不分請假與公假 —— **未列名不代表曠職**。`summary` 是 `null` 時是資料不足或檢查未通過,不是 0%。
5. **議案日期是院會開會日**(`date_source = plenary_sitting`),不是提案日。
6. 同名或對到多位 → 回 `candidates`,**請使用者選**或改用 `legislator_id`,不要自己猜。
7. 回覆時附上回應裡的連結(`doc_url`、`gazette_url` 等),並提醒以立法院官方公報為準。
8. **不可以不呼叫工具就沿用範例數字。** 本文件裡的票數、筆數、出席率都是 2026-10-04 資料的示意;實際回答一律以工具**這次**回傳的值為準,資料每天更新。
9. **找「三讀通過」:先用 `vote_kind = "final_passage"`(案由含「全案表決」或「全案付表決」)查;查不到時,不加 `vote_kind`,改用日期把那一天的表決全部列出來再判斷**(案由寫法不一,分類可能漏)。「提議三讀」是程序表決(要不要進入三讀),「修正動議」「再修正動議」是條文表決,「復議」是對已通過的決議要求重新表決 —— 案由裡有「三讀」兩個字不代表它就是三讀。同一天同一法案常有好幾筆,票數不同:**列出來說明差別,不要挑一筆冒充三讀**。
10. **公報搜尋只回片段,要下結論一定要再讀原文。** `tw_ly_gazette_search` 的 `snippets` 是關鍵詞前後各 150 字(每份最多 2 段),`speakers` 是同時命中關鍵詞的發言段落。**片段不是結論的依據** —— 看到 `proposal` 有值(命中落在某一件提案裡)或要回答「那一案要求什麼」時,一定要再用 `tw_ly_gazette_read`(帶 `doc_id` + `item_seq` 或 `at`)把**整案讀完**,`truncated` 為 true 就用 `next_at` 續讀到底。公報的**涵蓋範圍一律看回應裡的 `coverage.date_range`**(不要記年份 —— 回補一進展就會變),而且關鍵詞是**子字串**比對(不斷詞)—— **0 筆不代表院會沒討論過**,要說「這個關鍵詞在收錄範圍內沒有命中」並建議換個講法。`url` 是官方公報檔(`.doc`),給使用者當出處,不要宣稱自己讀過它。
11. **查不到表決時,說「官方開放資料沒有收錄」,不要說「當天沒有表決」。** 上游並非每場院會都有釋出記名表決;收錄自第 10 屆第 7 會期(2023-02-21)起。
12. **IVOD 逐字稿是「我們轉的」,公報是「立法院寫的」—— 法律性質不同,不可混講。** `tw_ly_ivod_search` 回的 `text` 是**本服務用 AI 語音轉寫(ASR)**的結果,**不是立法院的官方紀錄**:會有錯字與同音字(人名、法條名、數字最容易錯)。引用時要說「這是議事影音的 AI 轉寫,正式發言內容以立法院公報為準」,要確認用詞就再用 `tw_ly_gazette_search` 查公報 —— **不可以把轉寫內容寫成「公報記載」或「官方紀錄」**。另外:片段可能是官員的答詢,而**答詢者姓名尚未回填**(`speaker` 只是說話人編號)→ **不要把答詢段落歸給特定官員**,也不要假設整支影片都是掛名那位委員在講。轉寫仍在進行中,涵蓋範圍看 `coverage.date_range`,**0 筆不代表該委員沒發言**(要說「目前已轉寫的範圍內沒有命中」)。
12. **委員會名單是「逐會期」的,而且 `leave_date` 為空不代表在職。** `tw_ly_roster` 的委員會名單來自上游 #14,鍵是(屆, **會期**, 委員會, 委員)—— 同一位委員不同會期可能在不同委員會,所以沒有「他這一屆的委員會」這種單一答案,引用一定要講是哪個會期。在職與否看 `left`(上游離職日有缺漏,`leave_date` 是空的不等於還在任)。名單裡 `legislator_id` 為 null 的列是**姓名對不上名冊**(補選 / 遞補可能還沒進名冊),姓名是上游原樣(兩字名中間可能有全形空白)—— **那一列是真的有這個人**,不要當成查無此人而拿掉。`truncated` 為 true 代表被 `limit` 截斷(預設 100、上限 300;一屆 113 位以上)—— 不要把截斷後的名單講成完整名單。
13. **黨團協商:`subject` 是議案清單,不是結論;`text_available` 為 false 不是「沒有紀錄」。** `tw_ly_caucus_consultation` 回的是**索引**(場次、該場要談哪幾案、在第幾本公報的第幾頁、官方檔案 `url`)。`subject` 只告訴你談了哪幾案 —— **要回答「協商談出什麼」一定要讀全文**:`text_available` 為 true 時照 `read_hint` 呼叫 `tw_ly_gazette_read`(帶 `doc_id`),`truncated` 為 true 就用 `next_at` 續讀到底。`text_available` 為 **false** 代表那份公報**我方尚未收錄全文**(我方的全文收錄範圍看 `coverage`,不是每一份都已入庫),此時 `read_hint` 是 null、只有索引資訊 —— 要說「這一場有紀錄,但我們還沒有收錄全文,可以從 `url` 的官方公報檔查證」,**不可以說成「立法院沒有留下紀錄」或「查不到這場協商」**。上游欄位有缺漏(`term` / `session_period` 可能是空字串、`meeting_name` 可能只有「黨團協商紀錄」、`meeting_date` 可能是空字串)—— 那些場次**真的存在**,而且用 `date_from` / `date_to` 篩選會把沒有日期的場次篩掉,查不到不代表那段期間沒有協商。

---

## 情境 1:某位立委的問政紀錄

使用者:「黃國昌在立法院做了什麼?」

第 1 步:摘要模式(每個區塊的總筆數 + 最新 3 筆):

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_legislator_profile", "arguments": {"name": "黃國昌"}}}
```

回 `legislator`(屆別、黨籍、選區、委員會、`onboard_date` / `leave_date`),以及 `bills`、`speeches`、`votes`、`attendance`、`law_proposals` 各區塊的 `total`。
第 11 屆的黃國昌:議案 18、發言 410、表決 1023、質詢 0(質詢 0 是資料涵蓋問題,見規則 1)。

第 2 步:要看某個區塊的完整清單,`sections` 只給那一個,用 `cursor` 翻頁:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_legislator_profile", "arguments": {"name": "黃國昌", "sections": ["votes"], "limit_per_section": 20}}}
```

下一頁:把回應的 `votes.next_cursor` 放進 `cursor`,其他參數不變。換委員、換日期就要從第一頁重來(舊 cursor 會被拒絕)。

限定期間:加 `date_from` / `date_to`(西元 `YYYY-MM-DD`)。民國日期先用 `tw-opendata-toolkit` 的方法換成西元。

---

## 情境 2:某一案院會表決,各黨怎麼投

使用者:「罷免法三讀那次,誰贊成誰反對?」

第 1 步:先找全案表決(規則 9):

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_vote_lookup", "arguments": {"query": "罷免法", "vote_kind": "final_passage"}}}
```

回 `votes[]`,每筆有 `date`、`issue`(案由)、`vote_kind`、`official` 票數與 `vote_id`。這裡會找到 2026-06-12「公職人員選舉罷免法第二十六條,條文修正草案 全案表決」。
找到好幾案時(不同年份、不同條文),**列出來請使用者選**,不要只挑第一筆。

**查不到、或想說明同一天的經過**:不加 `vote_kind`,用日期把那天的表決都列出來,再從案由判斷哪一筆是全案表決。不可以只因為 `final_passage` 查無,就說「沒收錄」:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_vote_lookup", "arguments": {"query": "罷免法", "date_from": "2026-06-12", "date_to": "2026-06-12"}}}
```

以下數字是 2026-10-04 資料的示意,實際回答一律以工具這次回傳的值為準。那天罷免法第二十六條有三筆:

| 案由(節錄) | vote_kind | 贊成 / 反對 | 意思 |
|---|---|---|---|
| 台灣民眾黨國民黨黨團修正動議 | `amendment` | 56 / 49 | 條文用哪個版本 |
| 台灣民眾黨國民黨黨團提議三讀 | `motion_third_reading` | 57 / 49 | 程序:要不要當天進入三讀 |
| 條文修正草案 全案表決 | `final_passage` | **57 / 48** | **這才是三讀通過** |

第 2 步:把全案表決那筆的 `vote_id` 丟回去看明細:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_vote_lookup", "arguments": {"vote_id": "v1-b64970440dffc3fd"}}}
```

以下數字是 2026-10-04 資料的示意,實際回答一律以工具這次回傳的值為準。回:
- `vote.vote_kind` = `final_passage`;`official` = 贊成 57 / 反對 48 / 棄權 0;`named` 也是 57 / 48 / 0,`unmatched` 全是 0 → 直接寫「三讀以 57 比 48 通過」。如果 `unmatched` 不是 0,就要加一句「其中 N 票在名單上對不到委員」;
- `by_party`:國民黨 47 贊成、民眾黨 8 贊成、無黨籍 2 贊成、民進黨 48 反對(依名單計);
- `legislators.agree` / `against` / `abstain`:各選項的委員名單;
- `no_record`:表決當天在任、但名單上沒有紀錄的委員(院長、副院長通常在這裡)。官方資料分不出是未投票、請假還是名單缺漏,**不要寫成「缺席」**;在任與否是依到職日、離職日推估的。

**連日期都列不出任何表決時**(例如 2024-12-20),才照規則 10 回答「官方開放資料沒有收錄那天的記名表決」。

搜尋最多翻到前 200 筆;要找更舊的表決,加關鍵字或日期範圍縮小,不要一頁一頁翻完。

只想看某一屆、某段期間的表決清單:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_vote_lookup", "arguments": {"term": 11, "date_from": "2026-05-01", "date_to": "2026-06-30", "limit": 20}}}
```

某位委員**自己**投過什麼,用情境 1 的 `votes` 區塊,不是這支工具。

---

## 情境 2.5:某個議題在公報裡怎麼被講的

使用者:「立法院公報裡有沒有討論過居住正義?誰講的?」

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_gazette_search", "arguments": {"query": "居住正義", "limit": 10}}}
```

回 `total`(命中幾份公報)、`items[]`(`doc_id`、`meeting_date`、`term`、`subject`、`title`、`url`、
`snippets`、`speakers`)、`coverage`(總份數與日期區間)與 `notice`。

限定某位委員有發言的公報(同時只列他自己命中的段落):

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_gazette_search", "arguments": {"query": "居住正義", "legislator": "王婉諭"}}}
```

限定日期區間:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_gazette_search", "arguments": {"query": "總預算案", "date_from": "2026-01-01", "date_to": "2026-06-30", "limit": 20}}}
```

怎麼回答:

- 先講命中幾份、分布在哪段期間,再逐份給 `meeting_date` + `title` + 片段,最後附 `url`。
- 片段是**前後各 150 字**的節錄,句子可能被切斷 —— 照實呈現,不要補字、不要當成完整發言。
- `speakers[].legislator` 是 `null` 時表示那段發言(例如主席、官員)對不到委員名錄,照實說「公報標示為 X」。
- 使用者要「全文」時給 `url`,不要反覆呼叫工具拼湊。
- 想知道某位委員**整體**問政情況,用情境 1;想知道**某一案怎麼表決**,用情境 2。

---

## 情境 2.6:那一案到底要求什麼(**只看片段會把答案講反**)

這是真實踩過的坑,務必照做。使用者問「蔣萬安那件提案到底要求什麼」,`tw_ly_gazette_search`
找到了正確的公報、也正確歸屬到提案人,但片段只有關鍵詞前後各 60 字(事後已加長到 150)—— 片段停在
「替代治療(美沙冬)門診據點逐年減少、偏鄉民眾取藥不便,顯示該服務量能不足……」,
**看起來像是在講美沙冬治療資源不足、要求強化該服務**。

真正的訴求在片段後面約 200 字、寫在**整案的最後**:

> 爰針對是項預算**凍結 100 萬元**,俟衛生福利部……提出專案報告並經同意後,始得動支。

也就是這一案是**凍結預算**,不是加強服務 —— 只看片段下的結論**完全相反**。
提案的訴求(凍結 / 刪減 / 要求專案報告)一律寫在案子結尾,所以:

**命中落在提案裡(`proposal` 不是 null)時,回答前一定要先把整案讀完。**

搜尋結果拿到 `doc_id` 與 `proposal.item_seq` 之後:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_gazette_read", "arguments": {"doc_id": "LCIDC01_1100901_00004", "item_seq": 1}}}
```

只有位置、沒有案次時(`at` 用搜尋結果給的字元位置),工具會自己找涵蓋該位置的提案並回**整案**;
位置不在任何提案裡才回以它為中心的視窗:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_gazette_read", "arguments": {"doc_id": "LCIDC01_1100901_00004", "at": 51234, "max_chars": 6000}}}
```

回 `passage`(原文照抄)、`start` / `end`、`truncated` / `next_at`、`proposal`(提案人 / 連署人 / 黨團)、
`doc_chars`(整份字數)。**`truncated` 為 true 代表還沒讀完**,把 `next_at` 放進 `at` 再呼叫一次,
一路讀到 `truncated` 為 false 為止(單份公報可達 28 萬字,一次最多給 8000 字)。

怎麼回答:

- 先講這一案的**訴求**(凍結 / 刪減多少、附什麼條件),再講理由,最後才是提案人 / 連署人。
- `passage` 是**原文**,照實引用,不要改寫成自己的話再當成公報內容;要節錄就說明是節錄。
- 沒讀到案子結尾(`truncated` 還是 true)就**不要**下結論,也不要用片段腦補。
- 一份公報裡常有好幾件提案,`item_seq` 不同就是不同案 —— 不要把別案的訴求安到這一案頭上。

---

## 情境 2.7:某個議題在議場上怎麼被講的(IVOD 逐字稿 · AI 轉寫)

使用者:「有沒有立委在質詢時提到長照人力?誰說的?」

公報是官方逐字紀錄但出版有時差;IVOD 是議事影音,我方做了 AI 語音轉寫。

⚠ **這兩支不是二選一。** 使用者問「誰談過 X、他們實際說了什麼」時,**兩支都要查**
(見下面的情境 2.8)。公報給正式用詞與書面提案,IVOD 給現場口語與**可點開驗證的影片時間點**;
少了任何一邊,答案都不完整。只有在使用者明確只要其中一種時才單查一支。

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_ivod_search", "arguments": {"query": "長照人力", "limit": 10}}}
```

回 `total`(**命中幾支影片**,不是幾段)、`items[]`(`ivod_id`、`meeting_date`、`legislator`、
`meeting`、`term`、`video_page`、`start_s`、`end_s`、`speaker`、`role`、`text`、`at`、
`video_at`、`context`、`context_start_s`、`context_end_s`、`hits[]`)、`coverage` 與 `notice`。
**一支影片一筆**(同一支影片的多個命中合併進 `hits`)。`text` 是**命中的那一句**原文照錄,
`context` 是**前後文**(命中段前後各 6 段、最多 600 字)——
⚠ ASR 按停頓切句,`text` 常常只有兩三個字(「長照30」),
**要講「委員說了什麼」一定要引 `context`,不要只引 `text`**;
`video_at`(`<video_page>#t=<秒>`)可以直接點開跳到那一秒。

限定委員、日期、只看委員發言(不看答詢):

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_ivod_search", "arguments": {"query": "長照人力", "legislator": "王婉諭", "date_from": "2026-01-01", "role": "legislator"}}}
```

怎麼回答:

- 一定要**先聲明這是 AI 語音轉寫、非官方紀錄**,再引用內容(規則 12)。
- 逐支影片給 `meeting_date` + `legislator` + **`context`**(前後文),`text` 只用來指出
  關鍵詞落在哪一句;附 `video_at` 讓使用者直接點開跳到那一秒。
  **只引 `text` 會得到「長照30」這種沒有參考價值的片段**(2026-10-08 產線實測的教訓)。
- `role` 是 `respondent` 的段落只能說「這段是答詢」—— **答詢者姓名尚未回填,不要指名官員**。
- 要「那位委員全部的逐字稿」而不是關鍵詞搜尋 → 改用
  `tw_ly_legislator_profile` 的 `sections=["ivod"]`(v3 一律用 `execute_tool`,不要直接用工具名呼叫)。
- 要「官方逐字紀錄怎麼寫」→ 改用 `tw_ly_gazette_search`,**兩者不要混著當同一個來源引用**。
- 查無結果:說「目前已轉寫的範圍內沒有命中」,並建議換個講法或改查公報 ——
  **不可以說「他沒講過」**(轉寫還在進行中)。

---

## 情境 2.8:「誰談過 X?他們實際說了什麼?」(★ 最常見,**兩支都要查**)

使用者:「有哪些立委在議場上談過長照?他們實際說了什麼?」

這是**最常被問的形狀**,也是最容易答得不完整的一題。2026-10-08 與 10-09 兩次產線實測,
模型都**只查了公報**,回答裡沒有任何影片時間點 —— 內容雖然紮實,但使用者拿不到
「可以自己點開聽」的證據,而那正是我們獨有的東西。

**兩支都查,不要只查一支:**

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_gazette_search", "arguments": {"query": "長照", "limit": 10}}}
```

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_ivod_search", "arguments": {"query": "長照", "limit": 10}}}
```

怎麼組織答案:

1. **先說清楚這是母體的一部分,不是完整名單。** 熱門議題動輒命中數百份,
   `total_capped` 為 true 時尤其要講 —— 不要讓使用者以為「只有這幾位談過」。
2. **依子題分組**(財源、人力、制度銜接…),比依委員分組好讀。
3. 每一位委員給:**日期 + 委員會/院會 + 他講的重點**。
   - 來自公報的 → 可以用引號引原文,標 `url`。
   - 來自 IVOD 的 → **引 `context`(前後文)不要只引 `text`**,附 `video_at` 讓使用者點開跳到那一秒。
4. **兩種來源要分別標示,不可以混成一句。** 公報是官方正式紀錄;
   IVOD 是**本服務的 AI 語音轉寫、非官方紀錄**(規則 12)。
5. 最後講**收錄範圍**:公報與逐字稿各自的 `coverage.date_range` ——
   「9 月以後的發言還查不到」這種話要講出來。

為什麼 IVOD 這一半不能省:同一句話在兩邊看到的東西不一樣。
2026-10-09 實測「長照」,IVOD 撈到**莊瑞雄談受刑人被排除在長照之外**
(`text` 只有「你說我們的長照」七個字,`context` 才看得出主題)——
這個角度在公報的摘要裡抓不到,而且附的影片連結讓使用者能自己確認。

反過來,公報這一半也不能省:書面提案(凍結預算、附帶決議)只存在於公報,
IVOD 只有口頭發言。

---

## 情境 3:某部法律怎麼一路修過來

使用者:「大學法第一條改過幾次?每次為什麼改?」

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_law_history", "arguments": {"name": "大學法", "article": "1"}}}
```

回 `article_history.versions[]`:每一版的 `version_date`、`action`(制定 / 修正 / 增訂)、`text`(條文)與 `reason`(立法理由),由舊到新。大學法第一條有 5 個版本。

條號寫法都接受:`1`、`第1條`、`第一條`、`10-1`、`第10條之1`。

不給 `article` 是摘要模式:法律基本資料、最近修正過的條文清單、最近一次修法的讀會進度(`bill_steps`,含提案人與公報連結)。

**分編的法典**(例如「民法」分成總則、債、物權…)會回 `candidates`,請使用者選一編,再用 `code` 查:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_law_history", "arguments": {"name": "民法"}}}
```

法案追蹤(`bill_steps`)只涵蓋**最近一次**修法;更早的修法只有沿革與立法理由。已廢止的法律不收錄。

---

## 情境 4:委員主提案修了哪些法律 → 接法律沿革

使用者:「羅廷瑋提過哪些修法?道交條例那次改了什麼?」

第 1 步:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_legislator_profile", "arguments": {"name": "羅廷瑋", "sections": ["law_proposals"], "limit_per_section": 10}}}
```

回 `law_proposals.items[]`:`law_code`、`law_name`、`meeting_date`(一讀日期)、`third_reading`、`promulgated`、公報連結。最新一筆是 `02017` 道路交通管理處罰條例(2026-07-03 一讀)。
這個區塊只收**主提案**、而且有進入國會圖書館法案追蹤的修法 —— 0 筆不代表沒提過法律案(全部議案看 `bills` 區塊)。

第 2 步:`law_code` 直接當 `code`:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_law_history", "arguments": {"code": "02017"}}}
```

---

## 情境 5:出席率

使用者:「柯建銘院會出席率多少?」

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_legislator_profile", "arguments": {"name": "柯建銘", "sections": ["attendance"]}}}
```

以下數字是 2026-10-04 資料的示意,實際回答一律以工具這次回傳的值為準。回 `attendance.summary` = `{attended: 195, expected: 212, rate_pct: 92.0, unit: "會議日"}`:任期內有出席名單的 212 個院會會議日,名單上有他的有 195 日。
`attendance.total`(228)比 `attended` 大,是因為清單另外列了全院委員會;出席率只算院會。回答時照規則 4 說明「未列名不代表曠職」。

---

## 情境 6:某一屆有哪些立委

使用者:「第 11 屆有幾位立委?」

只給 `term`、不給姓名 = 名單模式:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_legislator_profile", "arguments": {"term": 11, "limit_per_section": 50}}}
```

回 `total` = 123、`legislators[]`(`id`、`name`、`party`、`area`、`left`),用 `next_cursor` 翻頁。
123 位**含中途離職與遞補**(法定席次是 113);要算「現任」就排除 `left: true` 的。

---

## 情境 7:某委員會有哪些委員、誰是召集委員

使用者:「內政委員會這個會期有哪些委員?誰是召集委員?」

```json
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_roster", "arguments": {"committee": "內政委員會", "term": 11, "session_period": "1"}}}
```

每一列的 `co_chairman` 就是召集委員(另有 `co_chairmen` 彙總清單);`committee_info.duty` 是該委員會的職掌。
**委員會名稱不做模糊比對** —— 寫「內政」查不到,回應會附 `available_committees`,照上面的名稱再查一次。
不給 `session_period` 就是**所有會期**的列都回,那時同一個人會出現多次(不同會期),不要說成「他在這個委員會 N 次」。

使用者:「第 11 屆有哪些立委?」

```json
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_roster", "arguments": {"term": 11, "limit": 150}}}
```

一屆 113 位以上,`limit` 預設 100 會被截斷 → 先看 `total` 與 `truncated`,不夠就加大 `limit`(上限 300)。
每位的 `committees` 是**逐會期**的委員會與召委身分。

使用者:「黃捷待過哪些委員會?當過召委嗎?」

```json
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_roster", "arguments": {"legislator": "黃捷"}}}
```

回 `terms`(每一屆的黨籍、選區、到職 / 離職日與逐會期的委員會)與 `ever_co_chairman`。
`legislator` **不可**與 `term` / `committee` 一起給(三種查法互斥,給了會回 `invalid_params`)。
原住民族委員只給漢名也查得到(「伍麗華」對得到名冊的「伍麗華Saidhai‧Tahovecahe」)。

---

## 情境 7.5:這個法案朝野協商怎麼談的

使用者:「菸酒管理法的朝野協商有紀錄嗎?談了什麼?」

第 1 步:先找場次(`query` 是對**議案清單**的子字串比對):

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_caucus_consultation", "arguments": {"query": "菸酒管理法"}}}
```

回 `consultations[]`:`meeting_name`(場次)、`subject`(**該場要談的議案清單**)、`meeting_date`、
`page_start` / `page_end` / `n_pages`(在公報裡的頁碼)、`url`(官方公報檔)、
以及 **`text_available`** 與 **`read_hint`**。

第 2 步:**`subject` 不是結論** —— `text_available` 為 true 時照 `read_hint` 讀全文:

```v3
{"tool": "execute_tool", "arguments": {"tool_id": "tw_ly_gazette_read", "arguments": {"doc_id": "LCIDC01_1015001_00010", "max_chars": 8000}}}
```

`text_available` 為 **false** 時 `read_hint` 是 null:那一場**有紀錄**,只是我方還沒收錄全文
(我方的全文收錄範圍看 `coverage`,不是每一份都已入庫)。這時要給使用者 `url` 讓他查證,**不要說成「立法院沒有留下紀錄」**。

不給任何參數就是回**最近**幾場協商;也可以用 `term`(屆別)或 `date_from` / `date_to`(西元 `YYYY-MM-DD`)縮範圍 ——
但上游沒有會議日的場次會被日期篩選篩掉。`limit` 預設 20、上限 50。

---

## 不確定參數時

```v3
{"tool": "inspect_tool", "arguments": {"tool_id": "tw_ly_vote_lookup"}}
```

授權以回應 `coverage.license` 為準;同一個回應裡不同區塊的授權可能不同(例如主提案修法來自國會圖書館法律系統)。
