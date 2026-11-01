# scene3d.js 使用說明（區塊 #1 3D 場景基底）

抽自桃園政策島。開一個新的 3D 作品時，燈光、陰影和堆積木的工具可以直接拿來用。需要 three.js（r128 測試過）。

## 1. 燈光 `Scene3D.lights`
```js
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xa7d0d8);
scene.fog = new THREE.Fog(0xa7d0d8, 170, 340);
const { hemi, amb, sun } = Scene3D.lights(THREE, scene, {
  sky: 0xfffaf0, ground: 0x6a9c98, hemi: .5,   // 天光：天空色、地面反光色、強度
  ambient: .06,                                // 環境光
  sun: 0xfff0d8, sunIntensity: .74, sunPos: [-30, 55, 28],
  shadowMap: 2048, shadowSize: 46              // 陰影貼圖解析度、陰影範圍半徑
});
renderer.shadowMap.enabled = true; renderer.shadowMap.type = THREE.PCFSoftShadowMap;
```
全部選項都有預設值，`Scene3D.lights(THREE, scene)` 就能用。

## 2. 陰影跟著鏡頭走 `Scene3D.followShadow`
大場景如果整片都要陰影，陰影貼圖會很模糊。做法是只在鏡頭附近算陰影，每一格把陰影範圍和太陽移到焦點：
```js
const SUNOFF = new THREE.Vector3(-30, 55, 28);   // 太陽相對焦點的位置
// 主迴圈，render 之前：
Scene3D.followShadow(sun, 焦點x, 焦點z, 範圍半徑, SUNOFF);
```
政策島的範圍半徑：走路模式約 46–80，俯瞰模式約 46–125（鏡頭越遠範圍越大）。

## 3. 積木工具 `Scene3D.primitives`
```js
const { mat, matF, mesh, rbox, box, cyl, sph, cone, roof, gable, sign, glowDisc } =
  Scene3D.primitives(THREE, scene, { roughness: .82 });
```
| 工具 | 用途 |
|---|---|
| `rbox(w,h,d,顏色,x,y,z,父層,圓角)` | 圓角方塊，低多邊形卡通感的主力 |
| `box(w,h,d,顏色,x,y,z,父層)` | 方塊 |
| `cyl(上半徑,下半徑,h,顏色,x,y,z,父層,邊數)` | 圓柱、樹幹、柱子 |
| `sph(r,顏色,x,y,z,父層,邊數)` | 球（y 是球心） |
| `cone(r,h,顏色,x,y,z,父層,邊數)` | 圓錐、針葉樹 |
| `roof(w,d,h,顏色,x,y,z,父層)` | 四角錐屋頂 |
| `gable(w,d,h,顏色,x,y,z,父層)` | 三角山牆屋頂 |
| `sign(文字,w,h,底色,字色,x,y,z,父層,{fs,ry,back})` | 文字招牌，字太長會自動縮小 |
| `glowDisc(x,z,r,透明度)` | 地上的圓形光暈，預設隱藏，夜晚再打開 |
| `mesh(幾何,顏色或材質,x,y,z,父層,陰影)` | 放任意 three.js 幾何 |
| `mat(顏色)` / `matF(顏色)` | 取材質（同色共用），`matF` 是平面著色 |

- **y 是底部高度**（`sph` 除外）：疊積木時直接寫「放在桌面上」的高度，不用自己加半個高度。
- **顏色寫數字**（`0xe9715d`）就會共用材質。想讓某種顏色在夜晚發光，只要改 `mat(0xfff4c8).emissive`，全部同色物件會一起亮。
- **父層**可以是 `THREE.Group`，整組一起移動、旋轉。省略就放進 scene。

範例，一棟小房子：
```js
const g = new THREE.Group(); g.position.set(5, 0, 3); scene.add(g);
rbox(2, 1.4, 1.6, 0xfbf7ee, 0, 0, 0, g, .12);    // 牆
gable(2.2, 1.8, .9, 0xd9645b, 0, 1.4, 0, g);      // 屋頂
sign("早餐店", 1.6, .4, "#2c6e80", "#fff", 0, 1.1, .82, g);
```

## 測試過的情境（政策島，2026-11-02）
固定亂數、只算不畫，比對整個場景 5,638 個物件的形狀、頂點、材質、顏色、位置、陰影設定，在剛載入、走動後、四個時段、移動玩家、切換俯瞰／走路都和抽出前一致；兩種螢幕寬度的實際畫面截圖逐位元相同。
