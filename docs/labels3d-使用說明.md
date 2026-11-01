# labels3d.js 使用說明（區塊 #6 3D 錨定標籤）

抽自桃園政策島的島名、市民名牌、四姐的對話泡泡。用一般 HTML 按鈕當 3D 物件的名牌，好處是字清楚、可以點、可以用 CSS 設計，也支援螢幕閱讀器。

## 放進 App
3D 畫布上面蓋一層放標籤的容器：
```html
<div id="stage"></div>          <!-- three.js 畫布 -->
<div id="labels"></div>         <!-- 標籤層 -->
```
```css
#labels{position:fixed;inset:0;pointer-events:none;z-index:3;overflow:hidden}
.zl{position:absolute;transform:translate(-50%,-100%);pointer-events:auto;transition:opacity .25s}
.zl.far{opacity:0;pointer-events:none}
```
```js
const LABELS = Labels3D.create({ container: document.getElementById("labels") });

// 新增：內容、額外 class、3D 位置、點擊、種類
const lb = LABELS.add("<b>01</b>交通", "isl", new THREE.Vector3(10, 4, -3), () => openCard("traffic"), "isl");

// 會動的物件：每一格更新錨點
lb.anchor.set(npc.position.x, 3.9, npc.position.z);

// 主迴圈，render 之後：
LABELS.update(cam, innerWidth, innerHeight, (l, dist, visible) => {
  if (!visible) return true;                         // 畫面外：藏
  if (l.kind === "npc" && dist > 44) return true;    // 市民名牌離 44 以上：藏
  return false;
});
```
不給規則時只藏畫面外的標籤。

## 政策島的隱藏規則（參考）
| 種類 | 走路模式 | 俯瞰模式 |
|---|---|---|
| 島名 `isl` | 72 以外藏 | 一直顯示 |
| 目標 `sub` | 34 以外藏 | 80 以外藏 |
| 市民 `npc` | 44 以外藏 | 90 以外藏（四姐例外） |
| 裝飾 `deco` | 110 以外藏 | 320 以外藏 |
| 泡泡 `bubble` | 沒在說話或 48 以外藏 | 90 以外藏 |
| 自己 `me` | 顯示 | 藏 |

規則的原則：遠看只留大標題，走近才出現細節，畫面才不會擠滿字。

## 小提醒
- 標籤很多時（政策島有 129 個），每一格都更新位置還是很順，因為只改 `left/top` 兩個值。
- `transform:translate(-50%,-100%)` 讓標籤的底部中間對準錨點，所以錨點要放在物件頭頂上方。

## 測試過的情境（政策島，2026-11-02）
129 個標籤的 class、螢幕位置，在剛載入、走動後、移動玩家、切換俯瞰／走路都和抽出前一致。
