# 機制光束改純光球（移除拖尾）設計

日期：2026-07-14
狀態：已與企劃確認（方案 D：純光球無軌跡；移除路徑弧線與彗星尾，改呼吸脈動光球＋落點爆開加強）

## 背景

- 機制觸發（生成 WILD／轉換／清除）時，能量從怪物死亡位置飛向盤面，由 `drawEnergyBeam()` 繪製：一條半透明路徑弧線＋頭部光球後方拖一段彗星尾。企劃希望**換掉拖尾**。
- 以視覺展示頁（`.superpowers/brainstorm/*/content/trail-alternatives.html`）比較四案後，企劃選定 **D・純光球（無軌跡）**。
- 此方向與 2026-07-09 能量飛卡粒子設計「元素能量不要拖尾」的定案一致。
- 影響範圍：`drawEnergyBeam()` 僅兩處呼叫——`drawWildFlyEffect()`（WILD／巨型 WILD 飛入）與 `drawMechanismBeams()`（轉換／清除的目標格光束）。其他飛行物（`collect` 能量、`bossHit`、噴金幣）**不變**。

## 需求（已確認）

1. **移除拖尾**：刪除半透明路徑弧線與頭部後方的彗星尾線段。
2. **呼吸脈動光球**：飛行本體改為「外圈屬性色柔光暈＋內核亮球（flash 色）」，尺寸隨飛行輕微脈動（約 ±18%）；大小仍依 `style.beamWidth × widthScale` 縮放，保留等級差異（Lv1 小～Lv4 大）與巨型 WILD 放大（1.55×）。
3. **抵達爆開加強**：落點閃光在現有實心閃光外，追加一圈**往外擴散的光環**；光束落點閃光時長由 8 幀提高到 12 幀。
4. 節奏**完全不變**：`beamFly`、`beamStagger`、`wildFlyIn`、`bigWildFly` 等時間參數與落地時機、拋物線軌跡、多道光束錯落、落地震動全部照舊——不影響節奏表，不新增調校面板項目。

## 實作設計

### 1. `drawEnergyBeam()` 重寫（核心，函式介面不變）

- 刪除：`quadraticCurveTo` 路徑弧線、`tail = beamPoint(t - 0.18)` 彗星尾線段。
- 新繪製（沿用 `lighter` 加色混合）：
  - 脈動係數 `pulse = 1 + 0.18 × sin(t × π × 5)`，以飛行進度 `t` 驅動，不需新時間源。
  - 外圈柔光暈：`style.color` 低透明度實心圓，半徑約 `(10 + beamWidth × widthScale × 2) × pulse`。
  - 內核亮球：`style.flash` 實心圓＋`shadowBlur` 發光，半徑約 `(5 + beamWidth × widthScale) × pulse`（與原頭部光球同基準）。
- 兩個呼叫端不動，自動生效；WILD 圖示照舊畫在光球之上（發光脈動的 WILD 飛過去）。

### 2. 落點爆開加強

- `drawMechanismCellFlashes()`：在既有「實心閃光＋白圈」外，追加一圈擴散光環——半徑隨進度由約 `0.35×GEM_SIZE` 擴到約 `0.9×GEM_SIZE×size`，alpha 隨壽命淡出。所有 `spawnCellFlash` 落點（光束落點、WILD 落地、轉換換圖）統一獲得此加強，視覺語彙一致。
- `updateBeamLandings()`：`spawnCellFlash` 的 `frames` 由 8 → 12。

## 邊界情況

- 多道光束錯落（`beamStagger` > 0）：各光球獨立繪製，互不影響。
- 巨型 WILD（widthScale 1.55）：光暈按比例放大，圖示仍蓋在最上層。
- 跳過／快轉：落地判定與時序邏輯未動，無需特判。
- 效能：繪製呼叫數比原本（兩段 stroke＋圓）更少，無壓力。

## 驗證方式

- preview 實際觸發各等級機制（Lv1~4 生成／轉換／清除、巨型 WILD）：確認飛行中無任何拖尾、光球有呼吸脈動、落點有擴散光環；`collect` 能量與噴金幣不受影響；console 無錯誤；節奏（飛行時長、落地時機）與改前一致。
