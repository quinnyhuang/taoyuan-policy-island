# timeofday.js 使用說明（區塊 #2 時段系統）

抽自桃園政策島。一排按鈕切換清晨、白天、黃昏、夜晚，天空、霧、光線一起變，選擇會記住。需要 three.js 場景（搭配 `scene3d.js` 的燈光最方便）。

## 用法
```html
<div id="tod">
  <button data-t="dawn">清晨</button><button data-t="day">白天</button>
  <button data-t="dusk">黃昏</button><button data-t="night">夜晚</button>
</div>
```
```js
const SUNOFF = new THREE.Vector3(-30, 55, 28);
const PRESETS = {
  dawn:  { bg: 0xf1d8c6, sky: 0xffe2c8, gr: 0x7a8fa0, hi: .52, sun: 0xffc28e, si: .62, off: [48, 20, 16],  lamp: .45 },
  day:   { bg: 0xa7d0d8, sky: 0xfffaf0, gr: 0x6a9c98, hi: .5,  sun: 0xfff0d8, si: .74, off: [-30, 55, 28], lamp: 0 },
  dusk:  { bg: 0xeeb48c, sky: 0xffcfa2, gr: 0x6a6a8a, hi: .45, sun: 0xff9256, si: .62, off: [-50, 18, 10], lamp: .8 },
  night: { bg: 0x1f3358, sky: 0x9fb2ea, gr: 0x2c3858, hi: .56, sun: 0xb4c2ff, si: .3,  off: [-20, 50, 20], lamp: 1.8 }
};
const TOD = TimeOfDay.create({
  presets: PRESETS,
  ctx: { scene, hemi, sun, sunOffset: SUNOFF },
  buttons: document.querySelectorAll("#tod button"),
  storageKey: "myTod",          // 記住選擇；省略就不記
  fallback: "day",
  extra: (T, k) => {            // 場景專屬的變化
    const lamp = mat(0xfff4c8); lamp.emissive.setHex(0xffcf73); lamp.emissiveIntensity = T.lamp;
    GLOWS.forEach(g => g.visible = k === "night" || k === "dusk");
  }
});
TOD.set("night");   // 也可以用程式切換
TOD.current();      // 目前時段
```

## 預設欄位
| 欄位 | 意思 |
|---|---|
| `bg` | 天空底色，霧也會用同一色（遠方自然融進天空） |
| `sky` / `gr` / `hi` | 天光的天空色、地面反光色、強度 |
| `sun` / `si` | 太陽顏色、強度 |
| `off` | 太陽相對位置 `[x, y, z]`；清晨、黃昏放低，影子就會拉長 |

其他欄位（例如 `lamp`、`water`）隨你加，在 `extra` 裡使用。

## 政策島的夜晚效果（參考）
路燈材質發光、路燈下的光暈打開、水變深藍、環境光加強、窗戶微微發光。全部寫在 `extra` 裡，靠「同色共用材質」一次改完。

## 測試過的情境（政策島，2026-11-02）
四個時段切換後，整個場景（天空、霧、燈光、每個材質的發光）、按鈕狀態、存檔都和抽出前一致。
