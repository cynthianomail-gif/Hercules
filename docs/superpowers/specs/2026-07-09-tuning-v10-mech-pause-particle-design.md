# 調校 v10：套用節奏 JSON＋掉落分拆滑條＋機制後停頓＋粒子調整 設計

日期：2026-07-09
狀態：已與企劃確認（四項一次做，TUNE_VERSION 升 10，機制後停頓預設 0.3s）

## 需求（已確認）

1. **套用企劃匯出的調校 JSON（v9）為程式預設值**：比對後 `timeline`／`layout` 與現行內建值完全相同，僅 `timing` 有 32 個數值差異 → 更新 `DEFAULT_TIMING`；`TUNE_VERSION` 9→10，舊 localStorage 存檔自動作廢改吃新預設。
2. **舊圖示清除／新圖示掉入 分拆滑條**：現行兩者共用 `spinDropIn`，`spinClear` 為閒置舊欄位 → 復用 `spinClear`＝舊圖示掉出畫面時長（預設 0.25）、`spinDropIn`＝新圖示掉入時長（預設 0.30），兩者皆加入面板「掉落（停輪）」群組（spinClear 新增、spinDropIn 改標籤「新圖示掉入」）。
3. **機制後→連線演繹間隔**：新增 `mechCheckPause`（預設 0.3s，範圍 0~5s），只在擊殺機制（WILD／生成／轉換／清除／FG/BG 收集等，皆經 `finishKillEffect`）結束後的那一次 CHECKING 生效；一般連鎖仍用 `interBoardPause`。面板放「WILD／機制」群組。
4. **能量粒子加大＋改飛攻擊力數字**：粒子整體加大約五成（光核 4.5→7、環繞光點與濺射/爆開粒子同步放大、環繞半徑 10→13）；能量目標點由「卡牌頂端中央 (baseY+24)」改為 `drawCardAtk` 的實際數字座標（NG：卡寬 0.855 / 卡高 0.112；FG：置中 / +58；皆含 `LAYOUT.atkNum` 偏移）。

## 實作要點

- **①**：只改 `DEFAULT_TIMING` 數值與 `TUNE_VERSION`；`GANTT_START_PRESET`、`DEFAULT_LAYOUT` 不動（已一致）。
- **②**：SPINNING 狀態中舊盤面離場的 `clearF` 由 `TF(TIMING.spinDropIn)` 改 `TF(TIMING.spinClear)`；新圖示掉入路徑維持 `spinDropIn`。甘特起點沿用（spinClear 0.00、spinDropIn 0.25）。
- **③**：新增旗標 `afterMechPause`；`finishKillEffect()` 設 true；GRAVITY 落定後進 CHECKING 時 `checkPauseTimer = TF(afterMechPause ? mechCheckPause : interBoardPause)` 並清旗標；`startSpin()` 重置旗標保險。
- **④**：`ENERGY_FX` 尺寸相關數值放大；能量 push 目標 `(cardCX, cardTY)` 改以 `getCardDrawMetrics` ＋ `LAYOUT.atkNum` 計算的攻擊力數字座標，NG/FG 分別對應。

## 邊界情況

- 舊存檔（v9 以前）載入即作廢，玩家端等同重置為新預設，無資料遺失疑慮（新預設＝企劃匯出值）。
- `mechCheckPause`：連續多個機制排隊（`pendingKillEffects`）時，停頓只在最後一個機制結束、真正回到連線判定前生效一次。
- 佈局面板調整 `atkNum` 偏移後，能量落點自動跟著移動（每次觸發時即時讀取）。

## 驗證方式

- preview 確認：新預設值生效（TIMING 抽查）、舊圖示清除與新圖示掉入分開變速有感、機制後停頓可調有感、粒子變大且爆開位置落在攻擊力數字上、console 無錯誤。
