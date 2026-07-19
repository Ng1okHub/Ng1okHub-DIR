# 🛡️ nProtect 检测篇：从事件回调反推 Engine 行为

[📖 繁體中文](readme.md)　|　[📖 简体中文](readme_cn.md)

前两篇 nProtect 相关内容主要停在 engine 修复与基础处理，这篇则往更里面看：它不是只记录「被检测到了」，而是把检测事件如何从 Native 层被包装、推送、再回到 Java 层回调的路径整理出来。

文章的价值在于，先把 `conditionCallback`、`DetectionInfo`、`DetectOptString` 与 runtime event queue 这几个观察点串起来，再用实测案例反推 root、虚拟化、Hook、Java stack 与服务端校验等检测面。对后续分析 nProtect / AppGuard 类保护来说，这能帮你先建立一个可验证的追踪框架。

## 🧭 这篇收敛出的检测面

| 检测面 | 文中处理到的内容 | 可看到的结果 |
| --- | --- | --- |
| 事件回调链路 | 从 `conditionCallback` 与 `DetectionInfo` 观察检测代号与上报内容 | 可把模糊提示对应回较明确的 detect code 与 detect string |
| root / Zygisk | 追到 `zygisk_3` 触发来源，并整理设备解锁、verified boot 与 attestation 之间的判断关系 | 能看出检测并不只是在找字串，也会结合设备状态判断 |
| 容器 / VA | 分析 mountinfo、`/proc/version` 与 `st_mode` 相关检查 | 可观察到多条虚拟化环境判断路径 |
| Hook 痕迹 | 针对 `libc.so`、Java stack、Xposed 类框架标记做定位 | 能把 Hook 检测拆成 Native 与 Java stack 两侧来看 |
| S2 Auth | 对比 engine 正常执行与 patch 后的服务端校验请求差异 | 提示单纯 patch engine 可能仍会留下断线或校验问题 |

## 🔎 文章重点

这篇的核心不是单点 bypass，而是先找出 nProtect 的检测事件如何被记录与分发。文中从 `startEngine()` 进入 `libengine.so` 的调度关系开始，逐步把 bootstrap、dispatcher、runtime event queue 与 Java 回调串起来，让后面的检测分析不再只能靠弹框或日志猜测。

比较有参考价值的部分，是把几个实际触发的检测点拆开：`zygisk_3`、VA / 容器环境、`libc.so` Hook、Java stack check，以及 S2 Auth 校验。每一块都不是只停在名称，而是往上追到它大致依赖什么状态、什么路径或什么事件上报。

## ⚠️ 绕过观察与限制

文中也整理了一条处理检测后闪退 / 卡死的路线：观察到被检测后会触发特定线程与退出相关逻辑，尝试 patch 相关线程函数并处理 runtime event 上报后，暂时能改善闪退或卡死现象。

不过这里的重点是「把现象收敛」，不是宣称已经把 nProtect 处理干净。S2 Auth 的对比也提醒了一点：即使本地 engine 路线被处理掉，服务端校验仍可能影响后续游戏流程，这部分还需要单独理解。

## 🎯 适合谁看

如果你已经能让 nProtect 样本跑起来，但还卡在不知道它到底检测了什么、为什么会延迟闪退、或怎么把 detect code 对应回实际检测点，这篇会比较适合先看。

## 💬 一句话说完

这篇把 nProtect 的检测事件、几类实测检测面，以及检测后的闪退处理观察，整理成一条更容易继续追下去的分析路线。
