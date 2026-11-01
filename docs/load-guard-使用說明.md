# load-guard.js 使用說明（區塊 #14 載入保險）

抽自桃園政策島。適合所有從 CDN 載入外部函式庫的 App，特別是用到 WebGL（three.js 等）的。

## 它解決什麼問題
- 主要 CDN 載不到時，自動依序改從備援網址下載
- 建立 WebGL 失敗時，隔一下自動重試
- 顯示卡中途中斷繪圖時，通知你顯示「請重新整理」

## 放進 App
```html
<!-- 主要 CDN 照常寫 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="load-guard.js"></script>
```

## 三個功能

### 1. CDN 備援 `LoadGuard.loadScript`
```js
LoadGuard.loadScript({
  test: () => typeof THREE !== "undefined",      // 函式庫可以用了嗎
  urls: [                                         // 備援網址，依序試
    "https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js",
    "https://unpkg.com/three@0.128.0/build/three.min.js"
  ],
  onReady: startApp,                              // 可以用了
  onFail: () => showError("下載失敗，可能是網路不穩。")
});
```
主要 CDN 成功的話，不會多下載任何東西。

### 2. 建立＋重試 `LoadGuard.createRenderer`
```js
LoadGuard.createRenderer({
  create: () => new THREE.WebGLRenderer({ antialias: true }),
  retries: 2,      // 失敗後再試 2 次（共 3 次）
  delay: 800,      // 每次間隔 0.8 秒
  onReady: r => { renderer = r; run(); },
  onFail: () => showError("瀏覽器目前沒有提供 WebGL。")
});
```
`create` 丟出錯誤或回傳 `null` 都算失敗。不只 WebGL，任何「有時會失敗、再試一次可能就好」的東西都能用。

### 3. 顯示卡中斷 `LoadGuard.watchContextLost`
```js
LoadGuard.watchContextLost(renderer.domElement, () => showError("請重新整理。"));
```

## 錯誤畫面
錯誤畫面的樣式每個 App 不同，所以不放在這個檔裡。政策島的寫法可以參考 `index.html` 裡的 `noGL()`：顯示原因＋「重新整理」按鈕＋替代閱讀方式。

## 測試過的情境（政策島，2026-11-01）
正常載入、cdnjs 掛掉改用 jsDelivr、三個 CDN 全掛、瀏覽器沒有 WebGL、顯示卡中斷繪圖，畫面與文字都和抽出前一致。
