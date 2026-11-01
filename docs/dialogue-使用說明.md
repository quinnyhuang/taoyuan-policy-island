# dialogue.js 使用說明（區塊 #7 多段對話，非 3D 部分）

抽自桃園政策島的市民對話和四姐對話。適合角色一句一句說話的介面：西語 App 的情境對話、教學導覽、故事模式。

> NPC 在 3D 場景裡走動、避障、頭上的完成標記不在這個模組裡（屬於 C2）。

## 畫面
角色區（小標、名字、說明）＋對話泡泡＋「繼續（1/3）」按鈕。最後一句時，按鈕換成你指定的結尾按鈕。

## 用法
```js
Dialogue.show({
  open: html => showSheet(html),          // 你的彈出面板
  head: { eyebrow: "咖啡店", name: "María", note: "店員（虛構角色）" },
  count: LINES.length,
  text: i => LINES[i],                     // 第 i 句（可以是 HTML）
  nextLabel: "繼續聽",                      // 會顯示成「繼續聽（1/3）」
  end: `<button class="btn pri" id="done">完成</button>`,      // 最後一句的按鈕
  always: `<button class="btn" id="skip">跳過</button>`,       // 每句都有的按鈕（可省略）
  after: (i, isLast) => {                  // 每畫一句後呼叫
    document.getElementById("skip").onclick = finish;
    if (isLast) markLearned();             // 聽完了
  }
});
```
- 「繼續」按鈕模組會自己處理，不用綁
- 從中間開始：加 `i: 2`
- 小標要換顏色：`head.eyebrowStyle: "color:var(--gold)"`

## 政策島的兩種用法（參考）
- 市民：結尾按鈕「看政見卡」，另有一顆一直都在的「直接看政見卡」；聽到最後一句就算完成。
- 四姐：結尾換成四顆功能按鈕（學經歷、政見總覽、叫公車、彩蛋）；最後一句會依收集進度加一段提示。

## 樣式
沿用 App 的 `.npc-head`、`.bubble`、`.row`、`.btn`。

## 測試過的情境（政策島，2026-11-01）
17 位市民每一句（實際按「繼續聽」翻頁）、直接跳過、四姐全部步驟與彩蛋來回、全部聽完後的結尾，HTML 與「聽完」紀錄都和抽出前一致。
