# share-card.js 使用說明（區塊 #11 分享圖產生器）

抽自桃園政策島的「完走證」。適合打卡成果、成就證書、學習紀錄等要讓使用者分享到 IG／LINE 的圖。

## 分工
- **你負責**：`draw` 函式，在畫布上畫內容（每個 App 不同）
- **這個檔負責**：開畫布、等字型、轉 PNG、預覽圖、分享按鈕、下載按鈕、不支援分享時的提示

## 放進 App
```html
<script src="share-card.js"></script>
```
HTML 裡準備一個放結果的空位：
```html
<div id="shareOut"></div>
```

## 基本用法
```js
ShareCard.render({
  format: "story",                 // "story" 9:16（1080×1920）或 "post" 4:5（1080×1350）
  out: "shareOut",                 // 元素 id 或元素本身
  fileName: "我的打卡成果.png",
  title: "我的打卡成果",            // 分享時的標題，也是預覽圖的替代文字
  text: "我連續學了 30 天！",       // 分享時附的文字
  onUnsupported: msg => toast(msg),// 瀏覽器不支援傳檔時；不給就用 alert
  draw: (c, { W, H, story }) => {  // c 是 canvas 2D context
    c.fillStyle = "#f4ede0"; c.fillRect(0, 0, W, H);
    c.fillStyle = "#1d3433"; c.font = "96px sans-serif"; c.textAlign = "center";
    c.fillText("連續 30 天", W / 2, story ? 400 : 250);
  }
});
```
`story` 是 true／false，方便兩種比例用不同版面。政策島的作法是準備兩組座標表 `L = story ? {...} : {...}`。

## 比例切換按鈕
```html
<div class="cvtab" id="fmt">
  <button data-f="story" class="on">限時動態 9:16</button>
  <button data-f="post">貼文 4:5</button>
</div>
```
```js
ShareCard.bindFormatTabs(document.getElementById("fmt"), f => { state.fmt = f; drawMine(); });
```
按下時自動切換 `on` class。

## 自訂按鈕外觀與文字
```js
ShareCard.render({
  ...,
  classes: { hint: "fine", row: "row", primary: "btn pri", button: "btn" },  // 預設值
  texts:   { hint: "手機請長按圖片存檔；電腦可以按「下載」。", share: "分享", download: "下載",
             unsupported: "這個瀏覽器不支援直接分享，請長按圖片存檔或按下載。" }
});
```
`onDone({ blob, url, file })` 可以拿到產生的圖，例如上傳到雲端。

## 注意
- 自訂字型（例如 Google Fonts）要在 HTML 先載入；這個檔會等 `document.fonts.ready` 才畫。
- iPhone Safari 和 Android Chrome 支援直接分享檔案；電腦版多半不支援，會顯示提示。
- 畫面上的預覽圖支援長按存圖。

## 測試過的情境（政策島，2026-11-01）
9:16 與 4:5 兩種完走證 PNG 和抽出前逐位元相同；比例切換、分享按鈕、不支援時的提示都一致。
