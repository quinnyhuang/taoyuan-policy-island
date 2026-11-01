# patrol.js 使用說明（區塊 #8 巡邏角色狀態機）

抽自桃園政策島的四姐和公車。適合「角色輪流到幾個定點、停下來做事、再走到下一點」的設計，以及載著玩家走一段路。

## 兩個工具

### 1. 狀態機 `Patrol.machine`
角色同一時間只處在一種狀態，每一格只跑目前狀態的程式。要換狀態就改 `state.mode`。
```js
const guide = { mode: "speak", t: 6, wp: null };

const SM = Patrol.machine(guide, {
  speak: c => { guide.t -= c.dt; if (guide.t <= 0) { guide.wp = route(…); guide.mode = "walk"; } },
  walk:  c => { const f = Patrol.follow(mesh.position, guide.wp, c.dt, { speed: 3.6, arrive: .25 });
                if (f.done) { guide.mode = "speak"; guide.t = 9; } }
});

// 主迴圈：
SM.step({ dt, t });      // 傳進去的物件會交給目前狀態
```
- 狀態之間要共用的東西（例如「這一格有沒有在走」）放在傳進去的物件裡，狀態程式設定 `c.walking = true`，外面再讀出來。
- `SM.go("walk")` 也可以從外面換狀態。

### 2. 沿路徑走 `Patrol.follow`
```js
const f = Patrol.follow(obj.position, waypoints, dt, { speed: 8, arrive: .3 });
// f.moved   這一格有移動
// f.reached 剛到達一個路徑點（已從陣列移除，這一格不移動）
// f.done    整條路徑走完
// f.dx, f.dz 移動方向，用來算面向：Math.atan2(f.dx, f.dz)
```
路徑點通常來自尋路（政策島用 `route()`，屬於 D 組的六角格尋路）。

## 政策島的兩個例子
- **四姐巡迴**：說話 → 跳下肥皂箱 → 沿路徑走到下一座島 → 跳上肥皂箱 → 說話。玩家靠近或正在對話時暫停倒數。四種狀態都用 `Patrol.machine`，走路用 `Patrol.follow`。
- **公車護送**：進站 → 等人 → 玩家上車 → 載著玩家沿路徑開 → 到站放人 → 離站。載客那段用 `Patrol.follow`；狀態切換的寫法和四姐相同。

## 測試過的情境（政策島，2026-11-01）
固定亂數、手動推進時間：四姐跑完兩輪以上「說話→跳下→走路→跳上」、公車進站／等候／載客／到站／離站，每個時間點的位置、面向、狀態都和抽出前一致。
