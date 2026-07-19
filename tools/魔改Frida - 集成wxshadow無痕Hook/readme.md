# 魔改Frida：集成 wxshadow 的無痕 Hook 版本

[📖 繁體中文](readme.md)　|　[📖 简体中文](readme_cn.md)

這套魔改 Frida 的重點很直接，就是把常見的主動 hook 能力接到 wxshadow 路徑上，盡量把直接 patch 帶來的可見痕跡壓低。它不是一個已經收斂到很穩的通用方案，現有記錄也明確提到手機仍有卡死風險，而且測試範圍主要集中在較固定的內核與設備環境。

## 🧭 做了什麼

- 集成wxshadow的無痕hook能力
- 去掉frida各種特徵

## 🪟 測試樣本

注: 只在P6, Android13, Kernel5.10上進行過測試, 其他機型/版本大概率用不了
1. 8ball (Appdome)  
![Appdome 樣本下的 hook 結果](images/appdome-result.png)

2. hunter  
![Hunter](images/hunter-boundary.png)


## 🎯 適用場景

厭倦了各種APP無窮無盡的花樣檢測? 或許可以試試這個工具。
