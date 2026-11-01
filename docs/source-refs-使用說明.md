# source-refs.js 使用說明（區塊 #12 出處資料模型）

抽自桃園政策島。適合任何需要附出處的內容：法案說明、知識卡、新聞整理。

## 概念
出處只寫一次，集中在一份字典；內容裡只寫 key。好處：
- 同一篇報導被很多張卡片引用時，網址改一處就全部更新
- 上線前可以自動檢查有沒有打錯 key、有沒有寫了沒用到的出處

## 放進 App
```html
<script src="source-refs.js"></script>
```

## 寫字典
格式：`key: ["媒體｜標題", "網址", "日期"]`。媒體和標題中間用全形「｜」。
```js
const SRC = {
  c0727: ["中央社｜某某新聞標題", "https://www.cna.com.tw/...", "2026-07-27"],
  l0727: ["自由時報｜另一篇標題", "https://news.ltn.com.tw/...", "2026-07-27"]
};
const REF = SourceRefs.create(SRC);
```
key 的命名建議「媒體縮寫＋日期」（c＝中央社、l＝自由、u＝聯合），同一天同媒體第二篇加 b。

## 用法
```js
REF.list(["c0727", "l0727"])   // → 出處列表 HTML（<div class="srcs"><a>…</a></div>）
REF.get("c0727")               // → { title, url, date, outlet: "中央社", key }
REF.missing(["c0727", "x99"])  // → ["x99"]：字典裡沒有的 key
REF.unused(所有用到的key)      // → 字典裡從沒被引用的 key
```
卡片資料裡放 `src: ["c0727","l0727"]`，畫卡片時呼叫 `REF.list(c.src)` 就好。

引用語的寫法（政策島範例）：
```js
`<cite>${REF.get(key).date}，見${REF.get(key).outlet}</cite>`
```

## 選項
```js
SourceRefs.create(SRC, { className: "srcs", arrow: " ↗" })  // 這兩個是預設值
```

## 樣式
CSS 由 App 自己寫。政策島的寫法：
```css
.srcs{display:flex;flex-direction:column;gap:6px}
.srcs a{font-size:13px;color:var(--ink);text-decoration:none;background:var(--paper);border-radius:10px;padding:7px 10px}
.srcs a span{color:var(--muted);font-size:12px;margin-left:6px}
```

## 和原本寫法的差別
找不到的 key 以前會讓整張卡片出錯，現在只會略過那一筆並在 console 警告。

## 測試過的情境（政策島，2026-11-01）
全部出處列表、14 張政見卡的完整 HTML，和抽出前逐字相同。
