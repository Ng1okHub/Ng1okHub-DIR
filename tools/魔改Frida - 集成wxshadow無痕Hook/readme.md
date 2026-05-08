# 魔改Frida：集成 wxshadow 的無痕 Hook 版本

這套魔改 Frida 的重點很直接，就是把常見的主動 hook 能力接到 wxshadow 路徑上，盡量把直接 patch 帶來的可見痕跡壓低。它不是一個已經收斂到很穩的通用方案，現有記錄也明確提到手機仍有卡死風險，而且測試範圍主要集中在較固定的內核與設備環境。

## 🧭 它做了什麼

這個版本補了三個與原生用法對應的入口：`Interceptor.wxshadow_attach`、`Interceptor.wxshadow_replace` 與 `Memory.wxshadowPatchCode`。從使用角度來看，重點不是多了一套新語法，而是把原本常見的 hook 動作換到 wxshadow 這條路上，方便直接拿來測試主動 hook 的可見性。

## 🪟 看到的結果是什麼

在文中給出的 Appdome 樣本裡，這套改版可以直接拿來 hook `libloader.so` 和遊戲邏輯所在的 so，當前記錄下沒有觸發檢測。這說明它至少已經能在部份樣本上，把常見的主動 hook 動作壓到比較可用的狀態。
![Appdome 樣本下的 hook 結果](images/appdome-result.png)

hunter attach情況下:
![Hunter 樣本下的邊界表現](images/hunter-boundary.png)

## 🎯 適用場景

厭倦了各種APP無窮無盡的花樣檢測? 或許可以試試這個工具。

## 💬 一句話說完

這是一個把 Frida 主動 hook 能力接到 wxshadow 路徑上的改版工具，已在部份樣本看到效果，但穩定性和暴露邊界仍需要一起評估。
