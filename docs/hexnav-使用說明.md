# hexnav.js 使用說明（區塊 #3 六角格地圖＋尋路）

抽自桃園政策島的 14 座蜂巢島與橋。管三件事：格子位置怎麼排、哪裡能走、怎麼從 A 走到 B。島和橋的模型（顏色、欄杆、路燈）由 App 自己畫，這個模組只算數字。不需要 three.js。

## 1. 排格子
```js
// 軸座標 [q, r]：q 往右、r 往右下
const CELL = { a: [0, 0], b: [-1, 1], c: [0, 1], d: [1, 1] };
const GRID = HexNav.grid(CELL, 21.65);   // 第二個數字：相鄰兩格中心的距離
GRID.cellPos([0, 1]);   // → [x, z]
GRID.cx, GRID.cz;       // 全部格子的平均中心（拿來把整片地圖移到原點）
GRID.adj;               // [["a","b"], ["a","c"], ...] 相鄰的格子對＝要蓋橋的地方
```
政策島的距離是 `2 × 島的內切半徑 + 2.6`（島邊到島邊留 2.6 的海）。

## 2. 建立可走地圖
```js
const NAV = HexNav.create();          // 參數全部可省略，見下表
const { walkable, heightAt, hubAt, route } = NAV;

// 每座島：先畫你自己的模型，再登記
const hub = NAV.addHub(x, z, 11, myIslandGroup);   // 半徑 11，尖角朝 ±z
NAV.makeNav(hub);                                   // 中心＋6 個邊緣導航點

// 每座橋（兩座島都 makeNav 之後）：登記並拿到畫橋需要的數字
const b = NAV.addBridge(hubA, hubB);
// b.ux, b.uz 方向；b.s, b.e 橋頭橋尾（從 A 中心量）；b.len 橋長；b.H 拱高
```

| 參數 | 預設 | 意思 |
|---|---|---|
| `edgeInset` | 2.2 | 邊緣導航點離島邊多遠 |
| `walkMargin` | .6 | 角色離島邊至少多遠 |
| `bridgeWalk` | 1.05 | 橋面可走的半寬 |
| `archHalfWidth` | 1.6 | 拱起高度生效的半寬 |
| `archHeight` / `archRatio` | 1.3 / .2 | 拱高＝取「1.3」和「橋長 × .2」比較小的 |
| `bridgeOverlap` | .5 | 橋頭伸進島裡多少 |
| `entryInset` | .7 | 橋入口導航點離島邊多遠 |
| `directDist` | 4.5 | 同一座島上、距離比這個近就直接走直線 |

## 3. 用起來
```js
walkable(x, z);        // 能不能站：在島上（離邊 .6 以內不算）或在橋面上
heightAt(x, z);        // 橋上的拱起高度；不在橋上是 0。角色每格設 y = heightAt(...)
hubAt(x, z);           // 在哪座島（沒有就 undefined）
NAV.inHex(hub, x, z, 1.6);   // 在這座島的「內縮 1.6」範圍裡嗎（市民閒晃範圍用這個）

const wp = route(fromX, fromZ, toX, toZ);   // [{x,z}, ..., 目的地]
// 主迴圈：朝 wp[0] 走，距離 < .35 就 wp.shift()
```
需要額外的導航點（例如政策島的肥皂箱位置）時：
```js
const n = NAV.node(x, z, hub);
NAV.link(n, hub.c);   // 連到島中心
```

## 運作方式（給好奇的人）
- 每座島 7 個點（中心＋6 邊），橋兩端各 1 個入口點；入口點連到最近的邊緣點。
- `route` 先找起點、終點附近最近的 3 個點（在橋上時找最近的 2 個入口），再用 Dijkstra 找最短路。政策島 14 座島、24 座橋共 160 個點，算一次很快。
- 自動走路時不檢查 `walkable`（路線本身就在可走範圍內）；手動操控才檢查，撞到邊就只沿可走的方向滑動。

## 測試過的情境（政策島，2026-11-02）
固定亂數、手動推進時間、不畫圖，比對 5 個時間點（剛載入、走 3 秒、自動走到寵物島、再走到青年島、長時間）：整個場景每個物件、12,544 個取樣點的 `walkable`／`heightAt`／`hubAt`、50 條路線的每個轉折點、160 個導航點與連線、24 座橋的數字，全部和抽出前一致。故意把 `edgeInset` 改成 2.25 → 路線、導航點、角色位置都抓到差異。
