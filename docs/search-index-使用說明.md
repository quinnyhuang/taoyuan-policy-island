# search-index.js 使用說明（區塊 #13 搜尋索引側欄）

抽自桃園政策島的「政見索引」。適合任何「一份清單＋搜尋框，點了跳過去」的介面：單字表、法條目錄、文章列表。

## 它做什麼
- 搜尋框打字時即時篩選，不分大小寫、自動去掉前後空白
- 一筆資料可以同時比對很多欄位（標題、摘要、細項…）
- 點清單上的項目，呼叫你指定的函式
- 找不到時顯示提示

## 放進 App
```html
<aside id="side">
  <input id="q" type="search" placeholder="搜尋…">
  <div id="list"></div>
</aside>
<script src="search-index.js"></script>
```
```js
const idx = SearchIndex.create({
  input: document.getElementById("q"),
  list:  document.getElementById("list"),
  items: () => WORDS,                                  // 資料陣列（用函式）
  text:  w => [w.es, w.zh, w.example],                 // 要被搜尋的欄位
  row:   w => `<button class="row" data-id="${w.id}">${w.es}<small>${w.zh}</small></button>`,
  rowSelector: ".row",                                 // 可點元素（預設 [data-id]）
  empty: q => `<p class="empty">找不到「${q}」。</p>`,
  onPick: id => openWord(id)                           // 點了哪一筆
});
idx.render();   // 第一次畫出全部
```
- `idx.render("關鍵字")`：用指定的字篩選
- `idx.refresh()`：用搜尋框目前的字重畫。資料或完成狀態改變時呼叫（例如打勾後要更新 ✓）

## 政策島的寫法（參考）
每座島比對名稱、卡片標題、摘要、重點、細項；每列顯示編號、名稱、副標、項目數和完成記號；點了就讓角色走到那座島：
```js
var IDX = SearchIndex.create({ input: $("pq"), list: $("plist"), rowSelector: ".pi", items: () => ISL,
  text: i => [i.name, i.card.t, i.card.sum, ...i.card.pts, ...i.subs.flat()],
  row: i => `<button class="pi" data-id="${i.id}">…</button>`,
  onPick: id => travelIsl(islOf(id), true) });
```

## 側欄的版面
樣式由 App 自己寫。政策島的做法：寬螢幕（1000px 以上）側欄常駐在右邊；窄螢幕藏起來，改用「政見索引」按鈕打開。

## 測試過的情境（政策島，2026-11-01）
375px 與 1280px：空白、中文關鍵字、前後有空白、英文大小寫、查無結果、實際在搜尋框打字、點擊連動，結果和抽出前一致。
