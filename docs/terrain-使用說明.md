# terrain.js 使用說明（區塊 #5 地形產生器）

抽自桃園政策島周圍的大地：西邊的海岸、東南的山和石門水庫、大漢溪、北邊的埤塘和樹林。模組只提供「零件」，地圖長什麼樣子（海岸線在哪、山多高、河怎麼走）由 App 自己給資料、自己組合。

## 想法：高度＝一個函式
地形就是「給 (x, z)，回答高度」。先用零件做出幾個「距離欄位」，再在自己的高度函式裡組合：
```js
const coastX = Terrain.lineX([[60,-200],[30,0],[-10,200]]);   // 海岸線：z 由小到大
const inLake = Terrain.blobField([[80,40,6],[84,46,7],[88,52,5]]);  // 一串圓點＝湖
const inRiv  = Terrain.blobField(riverPoints);   // 河也一樣，之後再 push 點也會算進去

function height(x, z) {
  const d = x - coastX(z);                 // 離海岸多遠（負數＝海裡）
  if (d < 0) return -7 + d * .9;           // 海：越外面越深
  let h = -6.2 + Terrain.noise(x, z) * .5; // 平原，加一點起伏
  if (inLake(x, z) < 0) h = -8;            // 湖：挖下去
  h = Terrain.addBumps(h, x, z, [[[100, 60], 8, 9]]);   // 一座小山：位置、高度、寬度
  return h;
}
```
零件說明：
| 零件 | 回傳 | 用途 |
|---|---|---|
| `lineX(points)` | `fn(z) → x` | 海岸線、邊界線 |
| `blobField(list)` | `fn(x,z) → 距離` | 湖、河、港池；負數＝在裡面 |
| `noise(x,z)` | 約 -2～2 | 山坡起伏 |
| `addBumps(h,x,z,bumps)` | 新的 h | 疊小山包 |

## 產生地面
```js
const land = Terrain.heightMesh(THREE, scene, {
  size: 600, segments: 220, cx: 0, cz: 0, height,
  bands: [[-6.45, 0xe6d6ae], [-2, 0x9dae90], [6, 0x8e9d73], [16, 0x758b5f], [26, 0x607850]],   // 低於…用這個顏色
  top: 0x949a8a,   // 最高的山頂
});
```
`segments` 越大越細，但頂點數是平方成長：220 格 ≈ 4.9 萬個頂點，手機還順。

## 水面
```js
scene.add(Terrain.blobSurface(THREE, lakePoints, { y: 5, material: waterMat }));
```
每個圓點鋪一片圓形水面，相鄰兩點夠近（≤ 6）時中間再補一片，接成連續的湖。

## 散布（樹林、埤塘）
```js
// 矩形裡隨機試 3000 次，放滿 560 棵就停；place 回傳 false＝這裡不要
const trees = [];
Terrain.scatter({ tries: 3000, max: 560, x0: -300, z0: -300, w: 600, d: 600,
  place: (x, z) => { const h = height(x, z); if (h < -4 || inLake(x, z) < 2) return false; trees.push([x, h, z]); } });
scene.add(Terrain.instances(THREE, new THREE.ConeGeometry(1, 2.6, 7), treeMat, trees,
  (dm, t) => { dm.position.set(t[0], t[1] + 1.2, t[2]); }));   // 幾百棵樹只算一個物件，很省

// 分區散布＋彼此不重疊（埤塘）
const rnd = Terrain.seeded(7);       // 固定種子：每次打開地圖都一樣
const sp = Terrain.spacer();
Terrain.scatterZones({ zones: [[-70, -5, 62, 40, 26]], rnd, tries: 12, place: (x, z) => {
  const r = 1 + rnd() * 1.9;
  if (!sp.fits(x, z, r, 2.2)) return false;   // 跟已放的至少隔 2.2
  sp.add(x, z, r);
  const pts = Terrain.blobPolygon(rnd, r, r * .7);   // 不規則圓形輪廓
  const m = new THREE.Mesh(new THREE.ShapeGeometry(Terrain.shape(THREE, pts)), pondMat);
  m.rotation.x = -Math.PI / 2; m.position.set(x, -6.1, z); scene.add(m);
}});
```

## 什麼留在政策島（內容，不是機制）
海岸線座標、水庫與河的路線、`hgt()` 裡各段的組合方式（平原、山、水庫壩前、河道）、埤塘排除的地標與機場範圍、樹的顏色，以及機場、風機、燈塔、船、水壩等模型。

## 小提醒
- **three.js 每建立一個物件（Object3D、材質、幾何）都會用掉亂數**（產生 uuid）。如果地圖有用 `Math.random` 散布東西，多建或少建一個物件，後面的隨機結果就會跟著變。所以 `instances`、`blobSurface` 都可以傳入共用的 dummy（政策島就是這樣做，才能和抽出前完全一樣）。想要地圖固定不變，散布請用 `Terrain.seeded(種子)`。
- 政策島裡的 `PONDS`／`inPond` 是早期版本留下、目前沒有用到的程式，沒有抽出。

## 測試過的情境（政策島，2026-11-02）
固定亂數、手動推進時間、不畫圖，比對 5 個時間點：整個場景每個物件（含地面 4.9 萬個頂點的高度與顏色、樹林與水面的每個位置、每個埤塘形狀）、33,252 個取樣點的高度／離湖距離／離河距離、539 個海岸線取樣，全部和抽出前一致；真實畫面在 1280px、375px 截圖逐位元相同。故意把小山包的範圍從 3 倍寬改成 2.9 倍 → 抓到地形差異。
