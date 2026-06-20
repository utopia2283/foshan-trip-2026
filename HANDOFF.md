# HANDOFF · 佛山順德容桂 3 日遊投票站

> 接手用 · 由 [A6 IPP MET柴之旅](https://foshan-trip-2026.vercel.app/) 上一手 agent 寫低 · 新 session 第一件事讀呢份 doc

---

## TL;DR

- **個站**:單檔 HTML 投票站 · 9 個朋友 · 30 個地點 + 8 個伴手禮
- **Trip**:6/30 - 7/2 出發 · Match 搞手 · 其餘 8 個投票
- **Stack**:純 HTML / CSS / Vanilla JS · 單檔 · 冇 build step · 冇後端
- **資料**:全部 hardcode 入 `<script id="places-data">` 同 `<script id="users-data">`
- **State**:投票 / 留言 / 當下用戶 全部喺 `localStorage` · 每部機獨立
- **Deploy**:`git push origin main` 自動觸發 Vercel production
- **Repo**:github.com/utopia2283/foshan-trip-2026
- **Live**:https://foshan-trip-2026.vercel.app
- **Local dev**:`cd "/Users/match/Documents/TVC Director/foshan-trip-2026" && python3 -m http.server 8000` → http://localhost:8000

---

## File Layout

```
/Users/match/Documents/TVC Director/foshan-trip-2026/
├── index.html              ← 成個 app 喺度 (3,490 行)
├── README.md               ← 粵語 user guide · 用嚟教 9 個朋友點用
├── HANDOFF.md              ← 你而家睇緊呢份
├── .gitignore              ← node_modules / .vercel / .gstack
├── .vercel/                ← Vercel link 設定 · 唔好 commit
├── .gstack/                ← gstack 工作狀態 · 唔好 commit
└── .git/                   ← 8 個 commit · 全部已 push
```

**冇** `package.json` · **冇** node_modules · **冇** build step · 用 `python3 -m http.server` 就跑得。

---

## Code Map · index.html 結構

| 行數(大概) | 範圍 | 內容 |
|---|---|---|
| 1-11 | `<head>` | Google Fonts (Noto Serif TC + Noto Sans TC) · theme-color `#C8362A` |
| 13-1400 | `<style>` | 全部 CSS · 用 CSS variables (`--brand`, `--ink`, `--paper` 等) |
| 1330-1358 | `#login` | 9-button 登入頁 · H1 上有 `A6 IPP MET柴之旅` eyebrow pill |
| 1360-1450 | `#app` | Appbar + welcome band + tabs + filters + grid |
| 1452-1461 | `<dialog id="about-modal">` | 關於本站 modal |
| 1464-1508 | `<script id="users-data">` | 9 個用戶 + 酒店 + trip 元資料 |
| 1510-2095 | `<script id="places-data">` | 30 個地點 + 8 個伴手禮 + 全部 `dianping` URL |
| 2097-3450 | `<script>` (主 JS) | 渲染 · state · 投票 · 留言 · gallery · lightbox · 天氣 · export / import |

兩個 data `<script>` 嘅 JSON 用 `JSON.parse` 讀 · 出事即成頁空白 · 改嘢後**一定要** sanity check:

```bash
node -e "JSON.parse(require('fs').readFileSync('index.html','utf8').match(/id=\"places-data\">([\\s\\S]*?)<\/script>/)[1])"
```

---

## 資料結構速查

### Users (users-data)
```jsonc
{
  "version": 1,
  "tripName": "佛山順德容桂 3 日遊",
  "tripDates": "2026-06-30 至 2026-07-02",
  "tripStartTs": 1782844800000,
  "tripEndTs":   1783104000000,
  "organizer": "Match",
  "hotel": {
    "name": "佛山柏希酒店",
    "address": "順德區容桂東堤路16號",
    "url": "https://www.trip.com/w/DecTOJYQ5V2"
  },
  "users": [
    { "name": "Match",  "role": "organizer", "avatar": "🧲" },
    { "name": "Carrie",                  "avatar": "🍆" }
    /* Illich, Red, Tracy, Ann, Kenneth, Lanche, Jules ... */
  ]
}
```

### Places (places-data · 30 entries · restaurant / attraction / ktv / wellness)
```jsonc
{
  "id": "tangfeng",                    // URL-safe slug · 用嚟做 #place-{id} anchor
  "name": "糖楓甜品",
  "category": "restaurant",            // restaurant | attraction | ktv | wellness
  "subcategory": "糖水 · 甜品",
  "rating": 4.7,                       // 大眾點評星
  "reviewCount": 460,                  // 大眾點評 review 數
  "pricePerPerson": 30,                // ¥ · 0 = 免費
  "mustTry": ["抹茶布丁", "抹茶牛奶麻薯"],
  "hasAirCon": true,
  "indoor": "indoor",                  // indoor | outdoor | mixed
  "distanceFromHotel": 1.5,            // km · 顯示喺 welcome band
  "address": "順德區容桂居民區",
  "openHours": "13:00–23:00",
  "michelin": false,
  "imageIds": ["1752552750868-b0b57e545e35", "1757752462906-62c319c5082d"],
  "description": "容桂隱藏甜品店,抹茶控必去...",
  "dianping": "https://www.dianping.com/shop/1633685851"  // optional · 餐廳先有
}
```

### Souvenirs (places-data.souvenirs · 8 entries)
```jsonc
{
  "id": "manggong",
  "name": "盲公餅",
  "imageIds": ["1558961363-fa8fdf82db35", "1593508514355-3b3a9b1a8e95"],
  "price": "¥25-45 / 320g 袋",
  "category": "餅",                    // 餅 | 粉麵 | 甜品 | 肉食 | 調味
  "whereBuy": "合記餅家 · 京華店(禪城)",
  "shops": "馳名老字號,佛山手信第一位",
  "description": "盲公餅係佛山最有名嘅手信...",
  "rating": 4.5
}
```

### IDs 30 個 (要加新 ID 用呢個格式)

| Category | IDs |
|---|---|
| restaurant (17) | zhuroupo · meiweiju · fengcheng · rongbian · tangfeng · minxin · xunwei · baidazhoushen · zhoufunin · shaozhubiao · baizhangyuan · feichanghao · yihuchayi · niurougong · runjixiaoye · yexianyusheng · yiheyuan |
| attraction (5)  | wanxianghui · huanle · bingxue · haifang · yurenmatou |
| ktv (3)         | huoli · panda · chill |
| wellness (5)    | shuxinwan · hexianggong · shilitaohua · mumuhotspring · junlanmuzu |
| souvenir (8)    | manggong · chencunfen · shuangpi · jiujiangjiandui · dalangyejijuan · daliangchaonianiu · fengchengyupijiao · guangshiyou |

> 注意 `wanxianghui` 同時係 restaurant ID 之一(原本係商場) · 邏輯上應該入 attraction · 改動要小心別 double render

### Tabs 數量(全自動 render)
- 想食 = `restaurant`
- 想玩 = `attraction`
- 想唱 = `ktv`
- 想hea = `wellness`
- 想買 = `souvenir`(讀 `data.souvenirs` 唔讀 `data.places`)
- 全部 = 上面加埋

加新 tab:加新 category 唔使改 JS · 想買 tab 已經喺度 · 想搞第 6 個就改 `data.souvenirs` 嗰個 pattern · 改 `data.places` 同時加相應 category 即可。

---

## 重要外部服務

| 服務 | 用嚟做咩 | 限制 / 注意 |
|---|---|---|
| **Unsplash CDN** | 全部圖片(`https://images.unsplash.com/photo-{id}?w=...&q=...&auto=format&fit=crop`) | 圖片 ID 要對 · 錯咗會出錯主題相 · 必 verify |
| **open-meteo** | 歡迎頁 3 日天氣 widget | 免費 · 免 API key · Foshan = `lat=22.978, lon=113.222` · 命中率一般但有總好過冇 |
| **大眾點評** | 17 間餐廳有 `dianping` URL · 卡片大眾點評 tab 撳去佢哋 shop 頁 | 大眾點評 block 自動化 IP · `anysearch` skill 搵 ID · cookie-import 唔到內容 |

---

## Git Workflow · 一手用過嘅 commit 風格

```bash
cd "/Users/match/Documents/TVC Director/foshan-trip-2026"

# 1. 改嘢
# 2. 驗 JSON
node -e "JSON.parse(require('fs').readFileSync('index.html','utf8').match(/id=\"places-data\">([\\s\\S]*?)<\/script>/)[1])"
# 3. 開 local server 試
python3 -m http.server 8000
# 4. 用 browse skill 開 http://localhost:8000 睇
# 5. 睇咗 OK
git add -A
git -c user.name="Match" -c user.email="match@foshan-trip.local" commit -m "feat: ..."
git push origin main   # 10-20 秒 Vercel auto-deploy
```

Commit 風格:< 80 字 · 中文 / 英文 / 廣東話混用都得 · 用 `feat:` `fix:` `refactor:` `docs:` 前綴。

---

## Git History

```
b934a3f  feat: 加 A6 IPP MET柴之旅 title                                     ← 當前 HEAD
e8d5274  feat: 6th 想買 tab + 天氣 widget + 卡片 gallery/lightbox/tabs + 修返 5 張錯圖
098275c  refactor: 全部改成廣東話 + 解釋主揪
a9c328f  refactor: Elon-style UX pass — 9-button login, big vote, no fan hero
c802666  feat: 大眾點評連結 + 修 fan hero JSON
a7deeec  fix: 統一排版 · 全部 view 改用 category section
9a17bad  feat: surface 佛山柏希酒店 hotel info in welcome band
6b6f8f7  Initial commit: 順德 3 日遊投票站 (30 places, 9 users, 粵語 UI)
```

`main` 已經 = origin/main · 冇 uncommitted 嘢(改咗記得 commit)。

---

## 點 Deploy / 點 Rollback

**新嘢**:
```bash
git push origin main
# Vercel 自動 build · 通常 10-20 秒
curl -sI https://foshan-trip-2026.vercel.app/ | head -2  # 應該 200
```

**睇 deploy log**:
```bash
# Vercel dashboard → 揾 foshan-trip-2026 project
# 或者 CLI:
npx vercel ls foshan-trip-2026
```

**Rollback**:
```bash
git revert HEAD
git push origin main
# 或者 Vercel dashboard → Deployments → 揾舊 build → Promote to Production
```

---

## Known Issues · 用戶層面要知嘅嘢

### 已修(當前 commit 之後冇)
- [x] `百丈園` imageIds 重複 bug
- [x] 5 張 off-theme 圖(糖楓甜品 / 燒豬標 / 順德萬象匯 / 歡樂海岸 / 樂漫冰雪王國)
- [x] README 25 → 30
- [x] 整頁空白嘅 fan hero JSON bug(2026-06-20 之前已經修)

### 仲有嘅小問題(未做 · 唔影響主流程)

1. **`一壺茶藝館` 第 1 張圖係 兩公婆喺屋企煮嘢**(ID `1556909114-f6e7ad7d3136` · 同歡樂海岸之前用嘅一樣) · 應該換茶樓/點心場面 · 仲有 14 個 restaurant 嘅 2nd 圖未 audit · 可能都有 off-theme 問題
2. **「關於本站」modal 寫 "同 6 個朋友"** · 實際而家係 9 個人(1 搞手 + 8 投票) · 應該改 "同 8 個朋友"
3. **天氣 widget 雨量百分比 88-91% 係 open-meteo 實時 forecast** · 7/2 trip 仲有 10 日 · 到時可能變 · 唔使改 code · 個 widget 會自己 fetch 新數據
4. **9 個朋友入面有 1-2 個未 verify 過會唔會用 mobile 開** · 桌面版測試過 OK · mobile 要喺真機 check

### Architecture · 大嘅設計決定 · 唔好亂改

- 冇 build step · 冇 framework · 純 vanilla · 改任何嘢都直接改 `index.html`
- localStorage 設計係 single-device · 朋友 A 投咗 · 朋友 B 部機睇唔到 · Match 要手動 import 朋友 export 嘅 JSON
- 圖片用 Unsplash ID 而唔係 upload · 改 imageId 唔使 upload 任何嘢
- 大眾點評用 `window.open` 直接跳 shop 頁 · 唔 embed · 因為大眾點評 anti-bot
- 9 個用戶名 / 搞手角色 / 酒店資料都係寫死喺 JSON · 唔好擅改

---

## 任務範本 · 「我想做 X · 點做」

### 加新餐廳 / 景點
```bash
# 1. 開 index.html · grep "id: \"zhuroupo\"" 揾到入口
# 2. 對住 schema 加 entry(見上面)
# 3. 圖片:用 anysearch 搵 Unsplash ID
node /Users/match/.codex/skills/anysearch/scripts/anysearch_cli.js search "unsplash [主題]"
# 之後 extract URL 抽 photo-{id} 嗰段 · 預覽:
curl -sL "https://images.unsplash.com/photo-{id}?w=400&q=70&auto=format&fit=crop" -o /tmp/preview.jpg
# 4. JSON check + reload browser 睇效果
# 5. Commit + push
```

### 改文案(地點 / 描述 / mustTry)
直接改 `index.html` `places-data` JSON · reload 即時見到 · 改完 commit + push

### 搞手(Match)用緊嘅特殊功能
- 右上 `⋯` → 「搞手總覽」:30 個地點 + 票數 + 投票者頭像
- 右上 `⋯` → 「匯出 JSON」:copy 全部 votes 到 clipboard · send 畀朋友 import
- 右上 `⋯` → 「匯入 JSON」:貼朋友 export 嘅 JSON · 合併入自己個 state
- 右上 `⋯` → 「重設」:清晒 localStorage

### 想做 mobile-only 改動
- viewport `<meta>` 已經 set
- 卡片用 `grid-template-columns: repeat(auto-fit, minmax(420px, 1fr))` · < 720px 自動變 1 column
- 測:Chrome DevTools mobile view · 或者 iPhone 真機

---

## 點繼續 · OpenCode 起手

新 session 第一句:
> 「接手 `foshan-trip-2026` · 讀 HANDOFF.md · 然後 ping 我要做咩」

跟住 user 會直接講要做咩 · 你:

1. `cd "/Users/match/Documents/TVC Director/foshan-trip-2026"`
2. `git status` 確認 clean
3. `git log --oneline -5` 確認喺 `b934a3f`
4. `python3 -m http.server 8000` 起 server
5. 用 `browse` skill 開 `http://localhost:8000/index.html` · login Match 試
6. 做嘢 · commit · push · confirm live

**唔好做嘅事**:
- 唔好加 build step / package.json / framework
- 唔好拆 file 開多個 .js / .css
- 唔好改 theme color 唔好改字體
- 唔好擅自動 `users` 陣列 · Match / Carrie / Illich / Red / Tracy / Ann / Kenneth / Lanche / Jules 9 個名已經同朋友講過

---

## 環境備忘

- **OS**:macOS · shell zsh
- **Python**:3.14.5 at `/opt/homebrew/bin/python3`
- **Node**:v22.22.3 at `/Users/match/.nvm/versions/node/v22.22.3/bin/node`
- **browse skill**:`/Users/match/.claude/skills/gstack/browse/dist/browse` · headless Chromium · `browse --help` 睇指令
- **anysearch skill**:`/Users/match/.codex/skills/anysearch/scripts/anysearch_cli.js` · 網頁搜尋 + URL 提取 + Unsplash 搵 image · 攞 API key 可用 anonymous
- **冇 mcp** · 用 skill CLI 直接 invoke

---

## 緊急 Contact

呢個 project 嘅 owner:Match (TVC Director · Hong Kong) · 用粵語溝通 · 鍾意短答 · 唔鍾意長篇
Trip 出發:2026-06-30 (仲有 10 日)
