# friend-links.js 使用說明（區塊 #17 友站互連）

抽自桃園政策島。適合想互相推薦的作品群：你自己的多個 App 互連、合作網站聯盟、作品集頁面。

## 它做什麼
- 友站資料集中寫成一份字典
- App 裡任何地方（按鈕、3D 物件、選單）都用 key 打開同一種介紹卡：小標、站名、一句說明、「前往 網址 ↗」按鈕、底部附註

## 放進 App
```html
<script src="friend-links.js"></script>
```
```js
const FRIENDS = {
  spanish: { t: "西語單字 App", d: "每天 10 分鐘的西班牙文單字練習。", u: "https://quinnyhuang.github.io/【repo 名稱】/" },
  party:   { t: "猜政黨", d: "看照片猜政黨的小遊戲。", u: "https://quinnyhuang.github.io/【repo 名稱】/" }
};
const FRIEND = FriendLinks.create(FRIENDS, {
  open: html => showSheet(html),     // 把卡片放進你的彈出面板
  color: "#d9645b",                  // 主色（預設）
  eyebrow: "友站",                    // 小標（預設）
  note: '<p class="fine">我做的其他小工具。</p>'   // 底部附註（可省略）
});

// 任何地方：
button.onclick = () => FRIEND.open("spanish");
```
- `FRIEND.html(key)`：只拿卡片 HTML，不打開
- `FRIEND.keys()`：列出所有 key，可以用來產生「所有友站」清單

## 卡片外觀
沿用 App 的 `.eyebrow`、`.btn.pri`、`.fine` 樣式。標題樣式可用 `titleStyle` 改。

## 政策島的 3D 入口（不在模組裡）
政策島在場景裡放飛機和布條，點了呼叫 `openFriend(k)`。3D 物件屬於 D 組，之後拆 3D 區塊時再一起處理；模組本身只管卡片。

## 測試過的情境（政策島，2026-11-01）
三個友站卡片的完整 HTML 在 375px 與 1280px 都和抽出前逐字相同。
