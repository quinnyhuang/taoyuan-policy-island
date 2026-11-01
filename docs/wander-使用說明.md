# wander.js 使用說明（區塊 #7 NPC 隨機走動）

抽自桃園政策島的市民。讓角色在一塊範圍內自己挑目的地、走過去、到了再挑下一個，不會穿過樹和房子。

只動 `position.x`、`position.z`、`rotation.y`，three.js 物件或任何有這些欄位的物件都能用。

## 三個工具

### 1. 挑目的地 `Wander.pickTarget`
```js
Wander.pickTarget({
  cx: 0, cz: 0,            // 範圍中心
  rMin: 1.5, rRange: 6,    // 距中心 1.5 ～ 7.5
  inside: (x, z) => 在可走範圍內嗎,
  from: npc.position,      // 從哪裡出發
  blocked: (x, z) => 這一點有障礙物嗎,   // 會沿直線檢查 8 個點
  tries: 14                // 試幾次，都失敗回傳 null
});
```

### 2. 每一格走一步 `Wander.step`
```js
// 每個 NPC 一個狀態物件，記錄目的地和暫停時間
const npc = { obj: mesh, tgt: null, pause: 0 };

// 主迴圈：
const moving = Wander.step(npc, npc.obj, dt, {
  pick: n => Wander.pickTarget({ …, from: n.obj.position }),
  speed: 1.05,   // 每秒幾單位
  turn: 6,       // 轉向速度
  arrive: .15,   // 到達距離
  retry: .3      // 挑不到目的地時先停幾秒
});
if (moving) 播走路動畫();
```
想讓 NPC 停下來（例如正在跟玩家說話），那一格就不要呼叫 `step`。

### 3. 轉向 `Wander.turnToward` / `Wander.turnAngle`
```js
Wander.turnToward(npc.obj, Math.atan2(玩家x - npc.x, 玩家z - npc.z), dt, 4);   // 慢慢轉頭看玩家
heading = Wander.turnAngle(heading, 目標角度, dt, 7);                         // 只算角度
```
會自動走最短方向，不會繞一大圈。

## 政策島的用法
市民在自己那座六角形島上走動；玩家靠近（5.5 單位內）或正在對話時就停下來轉向玩家。四姐、公車、市民的轉向都用 `turnToward`／`turnAngle`。

## 測試過的情境（政策島，2026-11-01）
固定亂數、手動推進約 50 秒的遊戲時間，17 位市民每個時間點的位置、面向、目的地、暫停時間都和抽出前一致。
