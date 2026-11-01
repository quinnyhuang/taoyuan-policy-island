# 社群分享預覽範本（區塊 #16）

抽自桃園政策島。網址貼到 LINE、Facebook、Threads、X 時，顯示標題、說明和縮圖的那張卡片。

> 這個區塊**不是 `.js` 模組**。社群平台抓預覽時不會執行 JavaScript，標籤一定要直接寫在 HTML 的 `<head>` 裡。

## 範本（複製到 `<head>`，換掉【】的部分）
```html
<meta name="description" content="【一句話說明，給搜尋引擎】">
<meta name="author" content="Quinny">
<meta property="og:type" content="website">
<meta property="og:title" content="【App 名稱｜副標】">
<meta property="og:description" content="【分享卡上的說明，40–60 字內】">
<meta property="og:url" content="https://quinnyhuang.github.io/【repo 名稱】/">
<meta property="og:image" content="https://quinnyhuang.github.io/【repo 名稱】/og.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:locale" content="zh_TW">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="【同 og:title】">
<meta name="twitter:description" content="【同 og:description】">
<meta name="twitter:image" content="https://quinnyhuang.github.io/【repo 名稱】/og.png">
<title>【同 og:title】</title>
```

## 縮圖 og.png
- 尺寸 **1200×630**，放 repo 根目錄
- 重要文字放中間，四周留白：LINE 和部分平台會裁邊
- 檔案盡量小於 1MB（政策島的 og.png 約 520KB）
- 網址一定要寫**完整網址**（https:// 開頭），寫相對路徑平台抓不到

## 上線檢查清單
1. `og:url`、`og:image` 都是完整網址，而且打得開
2. 標題、說明三組（og／twitter／title）內容一致
3. Facebook 用 [分享偵錯工具](https://developers.facebook.com/tools/debug/) 貼網址檢查，順便按「重新抓取」
4. 貼到 LINE 給自己看預覽

## 改了預覽但 LINE 還顯示舊的
LINE 會把預覽快取一段時間。分享時在網址後面加 `?v=` 和任意數字，LINE 會當成新網址重新抓：
```
https://quinnyhuang.github.io/taoyuan-policy-island/?v=2
```
每改一次就換個數字（`?v=3`、`?v=4`…）。這個參數不影響網頁內容。

## 適合加在哪些 App
所有會被分享出去的網頁都該加。西語 App、猜政黨如果還沒有 og.png，可以照這份補上。
