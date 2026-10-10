[English](README.md) · **中文**

# Jayden

本科申请者，兴趣在信息系统，以及软件与硬件结合的计算机方向。
我做的软件，用的人都承受不起它出错：
一个帮视障跑者约志愿者陪跑的 App，在 Minecraft 里做规划的 LLM 智能体，
还有一个给费城步行者找更安全路线的工具。

我的大部分工作在私有仓库里（一个线上运行的服务，以及未发表的研究）。
这一页介绍里面做了什么；链接只指向公开代码。

---

## 助盲跑 AidRun —— 给视障跑者找陪跑员

<p>
  <img src="assets/aidrun-home.jpg" width="190" alt="跑者首页：下一次预约的陪跑和预约新陪跑的按钮">
  <img src="assets/aidrun-voice-booking.jpg" width="190" alt="语音预约页：一个开始说话的大按钮">
  <img src="assets/aidrun-running.jpg" width="190" alt="陪跑中：里程、时长、配速，以及告诉陪跑员放慢的按钮">
  <img src="assets/aidrun-safety.jpg" width="190" alt="求助与安全页：播报位置、拨打紧急联系人、120、110">
</p>

视障跑者用语音或表单约好时间和地点，系统匹配附近已实名认证的志愿者。
之后双方走同一条订单流程：（可选的）先通一次电话、出发、汇合、跑步、跑后记录。
跑步中跑者有一个求助按钮，按下会附带实时位置；需要二次确认或长按 3 秒，防止误触。

**我做了什么** —— 2026 年 4 月至 10 月，三个端共用一份接口契约：

| 部分 | 技术栈 | 我的提交 | 已合并 PR | 自动化测试 |
|---|---|---|---|---|
| iOS | Swift、SwiftUI、高德地图、AVSpeech / Speech、ActivityKit | 611 中 533 | 285 | 1,884 |
| 后端 | Java 17、Spring Boot 3.5、MySQL、Redis、WebSocket | 988 中 969 | 447 | 2,082 |
| Android | Kotlin、Jetpack Compose、Retrofit | 98 中 93 | 88 | 3,458 |

后端从第一行代码起就是我写的。iOS 端最早是队友的原型，2026 年 7 月起由我接手。
后端部署在阿里云。[隐私政策与用户协议](https://jayden23018.github.io/blindrun-legal/)已公开。
App 还没有上架 App Store。

**我觉得做得好的几个工程决定**

- **无障碍放在第一位，不是最后补。** 跑者端每一屏都按 VoiceOver 设计：触控区 64pt
  （Apple 下限是 44pt），支持动态字体，订单状态变化时有语音播报。
  UI 测试在真机 iPhone 上用 Apple 的无障碍审计检查主要页面。
- **语音下单用了大模型，但不盲信它。** 跑者说「明天早上 7 点公园南门」，
  后端用通义千问抽取时间地点，App 念回来让跑者确认，确认后才下单。
- **12 个状态的订单状态机**，每个状态下谁能取消都有明确规则；
  跑者第一次和新志愿者跑之前，双方先通一次电话。
- **求助不说假话。** 只有拿到新鲜的 GPS 定位才发送；也绝不对盲人说「已通知家属」——
  短信是异步发的，可能还会失败。
- **一份契约，由机器检查。** OpenAPI 规格放在后端仓库，iOS 的接口代码由它生成。
  App 调了规格里没有的接口，或后端改动会让旧客户端出错，CI 都会失败。

代码：[iOS（公开）](https://github.com/Jayden23018/blind-run-ios) · 后端与 Android 为私有。

---

## ImplicitEnhancer —— 帮 LLM 智能体在 Minecraft 里做规划

研究项目，我是**第一作者**。LLM 在 Minecraft 的长合成链上会失败，因为它缺少一步一步的操作知识。
我们加了一层不需要训练的检索（SAER，按状态检索示例），根据智能体当前的背包挑选示例。

- 在 **7 个模型 × 60 个留出任务 × 3 个随机种子** 上评估。不加检索时，7 个模型里有 6 个
  只能通过 0–20% 的任务。加了 SAER 后，7 个模型里有 5 个的提升统计显著
  （McNemar 精确检验，Bonferroni 校正）。
- 我亲手做的部分包括评估流程 —— 交叉验证脚本、McNemar 显著性表，以及在一个开源权重模型上的复现。
- 已被 NeurIPS 2026 的 NEmo workshop 接收为 poster。

技术栈：JavaScript（Node.js、Mineflayer）、Python。代码为私有。

---

## SafePath —— 宾大周边的更安全步行路线

宾夕法尼亚大学暑期项目的期末作品，一个人完成。它把约 107,000 条历史犯罪记录
（OpenDataPhilly）算成风险分，再叠上用户提交的人行道报告，放到校园周边 3.5 公里的步行路网上。
然后跑两次 Dijkstra：一次求最短路线，一次求最安全路线，两条一起显示。

技术栈：Python、Flask、Pandas、OpenStreetMap、Leaflet。
[代码](https://github.com/Jayden23018/safepath) · [在线演示](https://safepath-black.vercel.app)

---

## 小项目

- **物理演示** —— 双摆混沌的交互模拟，带一个 AI 答疑助手。React、TypeScript、Vite。
- **中文幻灯片生成器**（私有）—— 生成单文件、断网可用的 HTML 幻灯片，字体子集内嵌，可导出矢量 PDF。
  做它是因为通用工具排不好中文。
- **个人网站** —— Astro、MDX、GSAP；简历用 Typst 排版。

---

## 日常用的工具

Swift / SwiftUI · Java / Spring Boot · Kotlin / Compose · Python / PyTorch · TypeScript / React ·
MySQL · Redis · Git 与 GitHub Actions

## 联系方式

zw6963470@gmail.com
