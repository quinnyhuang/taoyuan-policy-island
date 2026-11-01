# pickups3d.js 使用說明（區塊 #9 隱藏寶物，3D 部分）

抽自桃園政策島的隱藏寶物。場景裡的物品會旋轉、上下浮動、底下光圈閃爍，玩家走到旁邊就自動撿起。

## 用法
```js
// 每個物品：g 是物品的 three.js 群組，glow 是地上的光圈（可省略），x/z 是位置
const ITEMS = [{ id: "book", g, glow, x: 3, z: -2 }, …];

// 主迴圈每一格：
Pickups3D.update(ITEMS, t, player.position.x, player.position.z, {
  taken: it => state.items[it.id],          // 撿過了嗎
  onPick: it => {                           // 走到旁邊時
    state.items[it.id] = 1;
    it.g.visible = it.glow.visible = false;
    save(); CONFETTI.burst(it.x, 1, it.z);
  },
  enabled: !俯瞰模式,                        // 不能撿的時候傳 false（動畫照播）
  radius: 1.4                               // 撿取距離
});
```
外觀選項：`baseY`（高度，預設 .7）、`bob`（浮動幅度，預設 .12）、`spin`（旋轉速度，預設 1.2）。

## 獎勵怎麼給
由 `onPick` 決定。政策島的設計是「獎勵是外觀，不是關卡」：每件寶物解鎖一種造型，6 件全收集解鎖金色圍巾和金色完走證，再放一次大慶祝。蓋章本畫面上，沒撿到的寶物顯示「？？？」（見 `stamp-book.js`）。

## 換裝沒有抽出
換裝畫面（`openChar`）和套用外觀（`applyLook`）是直接改小企鵝 3D 模型的零件，和角色模型綁在一起，屬於 D 組角色的一部分。

## 測試過的情境（政策島，2026-11-01）
固定亂數、手動推進時間：物品動畫、靠近撿取、撿到第 1 件與第 6 件的提示和慶祝，都和抽出前一致。
