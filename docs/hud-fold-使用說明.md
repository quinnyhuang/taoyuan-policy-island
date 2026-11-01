# hud-fold.js 使用說明（區塊 #15 窄螢幕 HUD）

抽自桃園政策島。適合畫面上有浮動標題卡、說明卡，在手機上會擋住主畫面的 App。

## 它做什麼
- 標題卡右上角一個「−／＋」按鈕，手動收合或展開
- 收合時點標題也能展開
- 窄螢幕（預設小於 700px）在你指定的時機自動收合，例如介紹看完之後
- 自動維護 `aria-expanded`、`aria-label`，螢幕閱讀器也讀得懂

## 放進 App
```html
<div class="hud brand" id="brand">
  <button class="fold" id="fold" aria-label="收合標題" aria-expanded="true">−</button>
  <h1>App 名稱</h1>
  <p>副標、說明文字…</p>
</div>
<script src="hud-fold.js"></script>
```
```js
const fold = HudFold.create({
  panel: document.getElementById("brand"),
  button: document.getElementById("fold"),
  title: document.querySelector("#brand h1"),  // 可省略
  autoBelow: 700,     // 螢幕寬度小於多少才自動收合
  autoDelay: 2500     // 自動收合前等幾毫秒
});

// 介紹看完時：
fold.auto();
```
其他方法：`fold.set(true)` 收合、`fold.set(false)` 展開、`fold.toggle()`、`fold.isMin()`。

按鈕文字和說明可以改：
```js
HudFold.create({ ..., icons: { fold: "▲", unfold: "▼" }, labels: { fold: "收起", unfold: "打開" } });
```

## 收合後的樣子由 CSS 決定
模組只負責加上或拿掉 `.min` class。政策島的 CSS（收合後只留標題、縮小字級）：
```css
.brand .fold{position:absolute;top:8px;right:8px;width:30px;height:30px;border:0;border-radius:50%;cursor:pointer}
.brand h1{padding-right:28px;white-space:nowrap}
.brand.min{padding:8px 46px 8px 14px;border-radius:14px}
.brand.min>*:not(h1):not(.fold){display:none}
```

## 375px 不互相遮擋的 CSS 原則（政策島的做法）
- 所有浮動元素共用 `.hud`：`position:fixed`，外框 `pointer-events:none`、子元素 `pointer-events:auto`，透明區域不會擋住點擊
- 位置都加上 `env(safe-area-inset-*)`，避開 iPhone 瀏海和底部橫條
- 用 `@media (max-width:560px)` 把次要文字標籤藏起來，只留圖示和數字
- 側欄用 `@media (min-width:1000px)` 才常駐顯示，窄螢幕改成按鈕打開

## 測試過的情境（政策島，2026-11-01）
375px 與 1280px：初始狀態、自動收合（只在窄螢幕觸發）、按鈕切換、點標題展開，狀態與畫面截圖和抽出前一致。
