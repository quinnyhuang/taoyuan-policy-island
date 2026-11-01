# confetti3d.js 使用說明（區塊 #10 慶祝特效）

抽自桃園政策島。3D 場景裡的彩紙：從一點炸開、受重力落下、縮小消失。需要 three.js。

> 2D 網頁（例如西語 App）用不到這個；2D 彩紙要另外做一個 canvas 版本。

## 用法
```js
const CONFETTI = Confetti3D.create({
  THREE, scene,
  material: c => new THREE.MeshStandardMaterial({ color: c }),  // 可省略
  count: 36,                                        // 彩紙片數
  colors: [0xe9715d, 0x1f7a74, 0xe3aa45, 0x7b5cd6],
  reduced: matchMedia("(prefers-reduced-motion: reduce)").matches   // 使用者偏好減少動態就不播
});

// 主迴圈每一格：
CONFETTI.update(dt);

// 需要時：
CONFETTI.burst(x, 2, z);                            // 在某個位置炸開（完成任務、撿到東西）
CONFETTI.shower(() => player.position, { times: 6, interval: 700 });   // 連續放 6 次（大慶祝）
```
`shower` 的其他選項：`spread`（散布範圍，預設 8）、`yMin`／`yRange`（炸開高度，預設 6～10）。

## 運作方式
一開始就建好全部彩紙片（物件池），之後每次 `burst` 都是重複利用同一批，不會越放越多、拖慢速度。再次 `burst` 時，還在飛的彩紙會直接重新發射。

## 政策島的用法
聽完市民、撿到寶物、公車到站時 `burst`；集滿 6 件寶物時 `shower`，再加一架拖布條的飛機。飛機是場景專屬的 3D 模型，不在模組裡。

## 測試過的情境（政策島，2026-11-01）
固定亂數、手動推進時間：單次炸開後第 7 格和第 40 格、公車到站、撿寶物、集滿慶祝。每片彩紙的位置、旋轉、大小都和抽出前一致。
