# stamp-book.js 使用說明（區塊 #9 集章護照，非 3D 部分）

抽自桃園政策島的「集章護照」。適合任何「收集進度＋集滿給獎勵」的設計：西語 App 每個主題學完蓋一個章、打卡滿 30 天、看完所有卡片。

> 政策島的「隱藏寶物」和「寶物解鎖造型」是 3D 場景裡的東西，不在這個模組裡（屬於 C2／D 組）。

## 兩個功能

### 1. 進度與集滿觸發（只觸發一次）
```js
const BOOK = StampBook.create({
  count: () => 已蓋幾個,
  total: () => 總共幾個,
  isClaimed: () => state.rewardGiven,              // 獎勵給過了嗎（存在你的存檔裡）
  claim: () => { state.rewardGiven = true; save(); },
  onComplete: () => showCertificate()               // 第一次集滿時
});

// 每次蓋完章後：
BOOK.check();
```
其他：`BOOK.count()`、`BOOK.total()`、`BOOK.complete()`。

### 2. 蓋章本畫面
```js
const html =
  StampBook.section("主題", TOPICS, t => [state.done[t.id], t.name, t.level]) +
  StampBook.section(`隱藏成就 ${n}/6`, SECRETS, s =>
    state.got[s.id] ? [1, s.name, "解鎖：" + s.reward] : [0, "？？？", "尚未找到"]);
```
`cell(item)` 回傳 `[是否已蓋, 名稱, 小字]`。還沒解鎖的可以回傳「？？？」把名稱藏起來，這就是政策島隱藏寶物的寫法。
單一格也可以直接用 `StampBook.stamp(on, text, sub)`。

## 樣式
沿用 App 的 CSS。政策島的寫法：
```css
.ph3{font-size:13px;color:var(--muted);margin:14px 0 6px}
.stps{display:grid;grid-template-columns:repeat(auto-fill,minmax(96px,1fr));gap:6px}
.stp{background:var(--paper);border-radius:12px;padding:8px 8px 8px 30px;position:relative;font-size:13.5px;line-height:1.3}
```
`.stp.on` 是已蓋的樣式，`.stp i` 是左邊的圓章（裡面放 ✓）。

## 測試過的情境（政策島，2026-11-01）
蓋章本在全空、部分、全滿三種狀態的 HTML，以及集滿時觸發、存檔、提示文字，都和抽出前一致。
