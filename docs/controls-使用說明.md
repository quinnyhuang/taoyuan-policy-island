# controls.js 使用說明（區塊 #4 角色操控＋雙鏡頭）

抽自桃園政策島的操作手感：電腦用 WASD／方向鍵、滑鼠拖曳旋轉、滾輪縮放、點地面走過去；手機用左下搖桿、單指拖曳、雙指縮放、點一下走過去；右上「沙盤」切成俯瞰模式。需要 three.js。

## 1. 輸入
```js
const keys = Controls.keyboard({ onDown: e => { if (e.key === "Escape") closeSheet(); } });
const JOY  = Controls.joystick(document.getElementById("joy"), { onMove: () => walker.stop() });
```
搖桿的 HTML／CSS（政策島的版本，`.hud` 是它的固定定位樣式）：
```html
<div class="hud" id="joy" aria-hidden="true"><i></i></div>
```
```css
#joy{left:max(18px,env(safe-area-inset-left,0px));bottom:calc(70px + env(safe-area-inset-bottom,0px));width:112px;height:112px;border-radius:50%;background:color-mix(in srgb,var(--card) 70%,transparent);touch-action:none}
#joy i{position:absolute;left:36px;top:36px;width:40px;height:40px;border-radius:50%;background:var(--teal);opacity:.9}
@media (hover:hover) and (pointer:fine){#joy{display:none}}   /* 有滑鼠的電腦就藏起來 */
```
`touch-action:none` 一定要有，不然手指一推，整個頁面會跟著捲動。

## 2. 角色移動
```js
const walker = Controls.walker(THREE, {
  obj: player,                 // 要動的 Object3D
  walkable: NAV.walkable,      // 能不能站（HexNav 的）
  colliders: COL,              // [[x,z,半徑], ...] 障礙物，會被推開
});
walker.go(NAV.route(player.position.x, player.position.z, x, z), () => openCard());   // 自動走過去，到了呼叫
```
主迴圈：
```js
const [wx, wz] = Controls.moveVector(keys, CAM.sandbox ? null : JOY.v, CAM.yaw);
const { speed } = walker.update(dt, CAM.sandbox ? 0 : wx, CAM.sandbox ? 0 : wz);
player.position.y = NAV.heightAt(player.position.x, player.position.z);   // 過橋會拱起
pose(player.userData, dt, t, speed);   // Avatar3D.walkPose
```
手動按鍵會取消自動走路。自動走路時不檢查障礙和可走範圍（路線本身就在可走範圍內）；手動時撞到邊會沿著能走的方向滑。

| 參數 | 預設 | 意思 |
|---|---|---|
| `walkSpeed` / `autoSpeed` | 6.5 / 9.5 | 手動／自動走路的速度 |
| `accel` / `decel` | 10 / 14 | 起步、停下的快慢（越大越靈敏） |
| `radius` | .55 | 角色和障礙物保持的距離 |
| `arrive` | .35 | 離路線點多近算到達 |
| `turnRate` / `turnMin` | 12 / .4 | 轉身速度、速度超過多少才轉身 |
| `state` | 自己建 | 放 `auto` 路線的物件（政策島傳整個 `st`，原本的 `st.auto` 寫法就不用改） |

## 3. 雙鏡頭
```js
const CAM = Controls.orbitCam(THREE, {
  cam, canvas: renderer.domElement,
  walkDist: innerWidth < 640 ? 34 : 27,   // 手機拉遠一點
  panLimit: 200,                          // 俯瞰時焦點最遠能拖到哪
  onTap: e => { /* 點一下：先試著點物件，沒點到就走到那個地面 */ },
});
// 主迴圈（render 之前）：
const [fx, fz] = CAM.update(dt, player.position.x, player.position.y, player.position.z, walker.vel.x, walker.vel.z);
Scene3D.followShadow(sun, fx, fz, 60, SUNOFF);
```
切換俯瞰：
```js
CAM.sandbox = true; CAM.target.set(16, 0, 10); CAM.sbDist = 185;
```

| 操作 | 走路模式 | 俯瞰模式 |
|---|---|---|
| 左鍵／單指拖曳 | 旋轉 | 旋轉（手指是平移） |
| 右鍵或 Shift＋拖曳 | 旋轉 | 平移 |
| 滾輪／雙指 | 縮放 10～70 | 縮放 26～300 |
| 點一下（移動不到 6px） | `onTap` | `onTap` |

鏡頭上下角限制在 .32～1.38（約 18°～79°），不會鑽到地底或翻過頭頂。走路模式的鏡頭會往角色前進方向多看一點（速度 × .3），走起來比較不暈。

## 測試過的情境（政策島，2026-11-02）
固定亂數、手動推進時間、不畫圖，比對抽出前後完全一致：
- 1280px：按住 W＋D 走、滑鼠拖曳旋轉、滾輪縮放三次、點地面走過去、切換俯瞰、右鍵平移、拖曳旋轉、滾輪、俯瞰時按 A 平移、俯瞰時跳到某座島、回到走路
- 375px（手機模式）：搖桿推動、雙指縮放、俯瞰時單指平移、俯瞰時點一下、走路時點地面走過去

故意把 `accel` 從 10 改成 10.1 → 從第一次自動走路開始就抓到差異。
