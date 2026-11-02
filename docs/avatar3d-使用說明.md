# avatar3d.js 使用說明（區塊 #4 角色模型與動作）

抽自桃園政策島的小企鵝（玩家）、17 位市民和四姐的身體、6 種帽子、眨眼和走路動作。需要 three.js 和 `scene3d.js`（用它的積木工具蓋模型）。

## 建立
```js
const P3 = Scene3D.primitives(THREE, scene);
const AV = Avatar3D.create(THREE, P3);

const me = AV.penguin({ scarf: 0x3f8fd2 });   // 小企鵝，圍巾顏色
const npc = AV.human({ shirt: 0xe9715d, pants: 0x34495e, hair: 0x3b2a20 });   // grey:true 是白髮
scene.add(me, npc);
AV.setHat(npc, "cap");   // cap 棒球帽、dou 斗笠、helmet 安全帽、pilot 機長帽、driver 司機帽、umbrella 油紙傘；"none" 拿掉
```

## 動起來
模型的 `userData` 放著可以轉的關節：`L`（兩腿）、`A`（兩手）、`body`、`head`、`eyes`。
```js
const pose = Avatar3D.walkPose();       // 玩家：一個角色一個
// 主迴圈：
pose(me.userData, dt, elapsed, speed);  // speed 用 Controls.walker 回傳的速度；6 以上是全速
```
`walkPose` 會擺腿、擺手、身體左右晃；停下來時微微呼吸，每 5 秒拍一次翅膀；最後順便眨眼。

市民的動作比較簡單，政策島直接在迴圈裡轉 `L`、`A`，再呼叫 `Avatar3D.blink(userData, dt)` 眨眼。

## 換裝（政策島的做法）
企鵝的 `userData` 另外有：
- `scarf[0]` 圍巾環、`scarf[1]` 圍巾尾：換 `material` 就換顏色
- `tail2` 第二條飄帶、`tip`、`gl` 眼鏡：預設都隱藏，`visible = true` 打開
- `tailG` 圍巾尾的轉軸（`walkPose` 會讓它隨走路擺動）

換裝畫面（`openChar`）和「哪件寶物解鎖哪頂帽子」是政策島的內容，留在 `index.html`。

## 自訂帽子
```js
AV.hats.crown = h => { P3.cone(.5, .4, 0xf2c94c, 0, .1, 0, h, 5); };
AV.setHat(me, "crown");
```
`h` 已經放在頭頂（企鵝 .36、人 .37 的高度），在裡面蓋就好。

## 小提醒
- three.js 每建一個物件都會用掉亂數（產生 uuid）。模型的零件順序沒有改，所以政策島抽出後的隨機結果完全一樣；自己改模型時，後面用 `Math.random` 的東西位置可能會變，這是正常的。
- `human` 的眨眼起始時間用 `Math.random`，所以每個市民不會同時眨眼。

## 測試過的情境（政策島，2026-11-02）
固定亂數、手動推進時間、不畫圖，比對 17 個時間點（含換成機長帽＋飄帶＋眼鏡、換成油紙傘＋素色圍巾），場景每個物件都和抽出前一致；1280px、375px 真實截圖（司機帽＋飄帶＋眼鏡）逐位元相同。故意把機長帽高度改 .01 → 抓到。
