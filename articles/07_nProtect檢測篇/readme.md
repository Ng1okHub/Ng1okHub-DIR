# 🛡️ nProtect 檢測篇：從事件回調反推 Engine 行為

[📖 繁體中文](readme.md)　|　[📖 简体中文](readme_cn.md)

前兩篇 nProtect 相關內容主要停在 engine 修復與基礎處理，這篇則往更裡面看：它不是只記錄「被檢測到了」，而是把檢測事件如何從 Native 層被包裝、推送、再回到 Java 層回調的路徑整理出來。

文章的價值在於，先把 `conditionCallback`、`DetectionInfo`、`DetectOptString` 與 runtime event queue 這幾個觀察點串起來，再用實測案例反推 root、虛擬化、Hook、Java stack 與服務端校驗等檢測面。對後續分析 nProtect / AppGuard 類保護來說，這能幫你先建立一個可驗證的追蹤框架。

## 🧭 這篇收斂出的檢測面

| 檢測面 | 文中處理到的內容 | 可看到的結果 |
| --- | --- | --- |
| 事件回調鏈路 | 從 `conditionCallback` 與 `DetectionInfo` 觀察檢測代號與上報內容 | 可把模糊提示對應回較明確的 detect code 與 detect string |
| root / Zygisk | 追到 `zygisk_3` 觸發來源，並整理設備解鎖、verified boot 與 attestation 之間的判斷關係 | 能看出檢測並不只是在找字串，也會結合設備狀態判斷 |
| 容器 / VA | 分析 mountinfo、`/proc/version` 與 `st_mode` 相關檢查 | 可觀察到多條虛擬化環境判斷路徑 |
| Hook 痕跡 | 針對 `libc.so`、Java stack、Xposed 類框架標記做定位 | 能把 Hook 檢測拆成 Native 與 Java stack 兩側來看 |
| S2 Auth | 對比 engine 正常執行與 patch 後的服務端校驗請求差異 | 提示單純 patch engine 可能仍會留下斷線或校驗問題 |

## 🔎 文章重點

這篇的核心不是單點 bypass，而是先找出 nProtect 的檢測事件如何被記錄與分發。文中從 `startEngine()` 進入 `libengine.so` 的調度關係開始，逐步把 bootstrap、dispatcher、runtime event queue 與 Java 回調串起來，讓後面的檢測分析不再只能靠彈框或日誌猜測。

比較有參考價值的部分，是把幾個實際觸發的檢測點拆開：`zygisk_3`、VA / 容器環境、`libc.so` Hook、Java stack check，以及 S2 Auth 校驗。每一塊都不是只停在名稱，而是往上追到它大致依賴什麼狀態、什麼路徑或什麼事件上報。

## ⚠️ 繞過觀察與限制

文中也整理了一條處理檢測後閃退 / 卡死的路線：觀察到被檢測後會觸發特定線程與退出相關邏輯，嘗試 patch 相關線程函數並處理 runtime event 上報後，暫時能改善閃退或卡死現象。

不過這裡的重點是「把現象收斂」，不是宣稱已經把 nProtect 處理乾淨。S2 Auth 的對比也提醒了一點：即使本地 engine 路線被處理掉，服務端校驗仍可能影響後續遊戲流程，這部分還需要單獨理解。

## 🎯 適合誰看

如果你已經能讓 nProtect 樣本跑起來，但還卡在不知道它到底檢測了什麼、為什麼會延遲閃退、或怎麼把 detect code 對應回實際檢測點，這篇會比較適合先看。

## 💬 一句話說完

這篇把 nProtect 的檢測事件、幾類實測檢測面，以及檢測後的閃退處理觀察，整理成一條更容易繼續追下去的分析路線。
