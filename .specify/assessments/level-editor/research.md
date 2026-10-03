# Idea Research: 自研 Roguelike 关卡编辑器

- **Slug**: level-editor
- **Created**: 2026-10-02
- **Evidence confidence (overall)**: medium

> **阶段纪律**：本文件**只呈现证据，不裁定生死、不设计解决方案**。最终 verdict 由 `decide` 阶段给出。
> 每条结论必须带来源，无来源的一律标 `ASSUMPTION`。
> **证据整体是冲突的**（同一家公司既自研又采用现成工具），这不是调研失败，而是真实立项的常态。

## Users & Demand

- 痛点真实存在：Roguelike 的关卡由大量房间组合而成，房间之间的可达性一旦出错，只有把配置导入主工程运行才能发现，反馈链条长 — [source: ASSUMPTION（案例设定）] (confidence: low)
- 该痛点在业内被明确承认为硬约束：Unexplored 2 的开发博客把"环境必须 fully interlinked and traversable（完全互通且可通行）"写成关卡生成的**硬性前提**，而非事后检查项 — [source: https://www.ludomotion.com/blogs/the-place/index.html] (confidence: high)
- Dead Cells 的一手复盘描述了他们如何使用 **concept graph（概念图）** 规划关卡，并在**每个节点上校验房间出入口**是否匹配 — 说明"按房间节点校验连通性"是被验证过的做法，而不是臆想的需求 — [source: https://deepnight.net/tutorial/the-level-design-of-dead-cells-a-hybrid-approach/] (confidence: high)
- 目标用户规模很小：假设只有 2 名关卡策划 — [source: ASSUMPTION A2] (confidence: low)
- **反证**：ManiaMap 是一个专门做"房间模板 + 门约束 + 连通布局"的开源库，迭代约 3 年半，NuGet 总下载量仅约 1 万次 — [source: https://packages.nuget.org/packages/MPewsey.ManiaMap] (confidence: medium)
  → 读法：这个细分场景的**公开用户量很小**，意味着靠社区/第三方方案分摊固定成本的希望不大；但也同样意味着**自研的商业化回报几乎没有**。

## Prior Art

- **同品类同规模的公开选择是"用现成的，而不是自研编辑器"**：Dead Cells 官方在 2021-06 的公告中写明 *"Integrated Tiled for modders to make custom room design easier"* — [source: https://store.steampowered.com/news/posts/?feed=steam_community_announcements&appids=588650&enddate=1628685916] (confidence: high)
- **但同一款游戏也自研了关卡设计侧的工具链**（concept graph + 节点级校验） — [source: https://deepnight.net/tutorial/the-level-design-of-dead-cells-a-hybrid-approach/] (confidence: high)
  → **这两条合起来的读法**：他们并不是"要么全自研、要么全现成"，而是**用现成工具承担通用编辑能力，把自研投入集中在游戏特有的设计约束上**。
- **LDtk（开源，`deepnight/ldtk`）已内置房间邻接模型**：世界布局支持 Linear / Free / **GridVania**，且每个关卡 JSON 中都有 `__neighbours` 数组，列出所有与之物理相邻的关卡，随世界布局自动维护 — [source: https://ldtk.io/docs/game-dev/json-overview/world-layout/] (confidence: high)
  → **重要限定**：`__neighbours` 表达的是**几何相邻**（格子相接），不等于**语义连通**（门、钥匙、单向通道、传送点之后"能不能走到"）。前者现成已有，后者仍需游戏侧定义规则。
- **Tiled 提供官方脚本系统**，可编写并执行 JavaScript 扩展，官方文档明确支持从命令行/自动化场景调用 — [source: https://doc.mapeditor.org/en/stable/manual/scripting/] (confidence: medium)
  → 读法：**"在现成编辑器上挂一层校验脚本"是被官方支持的路径**，不需要新写编辑器。
- **ManiaMap** 已实现"房间模板 + 门约束 + 图布局"的连通性求解，BSD-3 许可，2025-10 仍有更新 — [source: https://packages.nuget.org/packages/MPewsey.ManiaMap] (confidence: medium)
- **Unity Asset Store 存在直接竞品**：World Graph Editor，40 美元，节点式编辑世界/场景连接图，关键词含 node editor / portal / door / metroidvania / level manager，最新版 1.3.1 发布于 2026-08-14（仍在维护） — [source: https://assetstore.unity.com/packages/tools/level-design/world-graph-editor-306228] (confidence: medium)
  → **未能验证**：其是否支持连通性/可达性校验（页面未渲染完整描述）。需在 Unity 内实测后才能作为决策依据。
- 学术侧旁证：一项对 440 份游戏开发复盘的统计研究中，"Absent or Inadequate Tools（工具缺失或不适用）"这一原因出现 12 次且带有游戏行业特异性；其中 Swords & Soldiers 的复盘提到"当时只有一名程序，只能用记事本编辑 XML，事后认为本应做可视化关卡编辑器" — [source: https://ar5iv.labs.arxiv.org/html/2202.06183] (confidence: medium)
  → **双向证据**：既支持"工具缺失确实是真实痛点"，也提示"当年没做编辑器并非因为不该做，而是**人力不足**"——即瓶颈是人力，不一定是工具形态。

## Market & Context

- **用户今天靠什么凑合**：手写配置文件 + 导入主工程运行验证 — [source: ASSUMPTION（案例设定）] (confidence: low)
- **不作为的代价**：连通性问题返工，一轮反馈需半天起 — [source: ASSUMPTION A6] (confidence: low)
- **可替代路径的成熟度已足够高**：LDtk（免费开源）+ Tiled（免费开源 + 官方脚本）+ Unity 图编辑器（40 美元）三者已覆盖"拖拽摆房间"与"世界邻接关系"两件事 — [source: 见上三条 Prior Art] (confidence: medium)
- 判据参考：Joel Spolsky 的经典论证主张**核心业务才值得自研**，非核心业务的重复造轮子应避免 — [source: https://www.joelonsoftware.com/2001/10/14/in-defense-of-not-invented-here-syndrome/] (confidence: medium)
  → 争议点：**关卡工具算不算游戏团队的"核心业务"？** Dead Cells 的做法（自研设计约束层 + 采用通用编辑器）可视为"部分算"的一种答案。

## Data & Constraints

- **预算上限假设 15 人日（超 25 人日直接 kill）** — [source: ASSUMPTION A3] (confidence: low)
- **技术难点不在算法**：图可达性/连通性判定本身是成熟算法（几十到几百行代码即可实现） — [source: https://www.ludomotion.com/blogs/the-place/index.html 与 https://deepnight.net/tutorial/the-level-design-of-dead-cells-a-hybrid-approach/ 的共同指向] (confidence: medium)
- **真正的成本在算法之外**：编辑器 UI、数据格式设计、主工程导入器、撤销/版本管理，以及**长期维护** — [source: 综合 LDtk 为资深作者长期个人项目、以及 Tiled 脚本机制的存在推断] (confidence: medium)
- **迭代速度是明确的收益方向**：GDC 演讲《Cutting the Pipe: Achieving Sub-Second Iteration Times》讨论的正是"缩短迭代反馈时间"对研发效率的价值 — [source: https://www.gdcvault.com/play/1015975/Cutting-the-Pipe-Achieving-Sub] (confidence: medium)

## Evidence Against the Idea

> 本节为强制项。以下是最强的反对理由，**不得省略或弱化**。

1. **同品类同规模已有公开反例**：Dead Cells 选择集成 Tiled 供 modder 与关卡设计使用 — [source: 见 Prior Art 第 1 条] (confidence: high)
2. **自研的差异化正在被开源方案吃掉**：LDtk 已内建世界布局与 `__neighbours` 邻接数据；Tiled 有官方脚本系统可在 CI 中执行校验；ManiaMap 已实现门约束连通求解 — [source: 三条分别见 Prior Art] (confidence: medium–high)
3. **15 人日的预算很可能覆盖不了真实成本**：真正的工作量在 UI、导入器、撤销、维护，而非算法 — [source: 见 Data & Constraints] (confidence: medium)
4. **目标用户只有 2 人，固定成本无法摊薄**：ManiaMap 3.5 年约 1 万次下载可作为细分市场规模的间接参照 — [source: https://packages.nuget.org/packages/MPewsey.ManiaMap] (confidence: medium)
5. **商业替代品只要 40 美元且仍在维护**：Unity World Graph Editor（1.3.1 / 2026-08-14） — [source: 见 Prior Art] (confidence: medium)
6. **"自研必然失败"缺乏直接证据——这本身也是事实**：本次调研**未找到**游戏行业"自研内部工具超支后废弃"的可引用一手复盘 — [source: 调研缺口，见下节] (confidence: high，指"确实没找到")
   → 因此**不能以"自研一定会烂尾"作为 kill 的理由**，那是没有证据的断言。

## Gaps & Open Questions

- [NEEDS CLARIFICATION: 预算上限（A3）与成功指标（A6）尚未由制作人确认——这是本立项最大的信息缺口]
- [NEEDS CLARIFICATION: Unity Asset Store 上的竞品（World Graph Editor / Dungeon Architect 等）**是否真的支持连通性校验**，页面未渲染描述，需实测]
- [NEEDS CLARIFICATION: 语义连通性的规则复杂度有多高？（只判"能否走到"，还是要处理钥匙/单向门/传送/条件解锁）——这直接决定工作量是"薄薄一层"还是"另一个项目"]
- [NEEDS CLARIFICATION: 主工程导入器由谁负责、是否已存在？（若不存在，它可能比编辑器本身更贵）]
- **调研缺口（如实记录）**：
  - 未找到"关卡设计返工成本 / 内部工具 ROI"的公开量化数据
  - 未找到自研内部工具失败的一手复盘
  - Hades 的一手关卡流程来源未取得（多个站点抓取失败或 403），仅有第三方研究转述

## Sources

> 说明：本表中的 URL 均由调研过程实际抓取并读到内容；未能读取的已剔除或在上文标注"未验证"。
> `policy` 一列：`allowlisted` 表示该主机在评估流程的免询问白名单内；`verified-reachable` 表示主机不在白名单，但内容已抓取且仅作为**参考证据**呈现，不作为决策的唯一依据。

- https://www.ludomotion.com/blogs/the-place/index.html（host: www.ludomotion.com，policy: verified-reachable）
- https://deepnight.net/tutorial/the-level-design-of-dead-cells-a-hybrid-approach/（host: deepnight.net，policy: verified-reachable）
- https://store.steampowered.com/news/posts/?feed=steam_community_announcements&appids=588650&enddate=1628685916（host: store.steampowered.com，policy: verified-reachable）
- https://ldtk.io/docs/game-dev/json-overview/world-layout/（host: ldtk.io，policy: verified-reachable）
- https://doc.mapeditor.org/en/stable/manual/scripting/（host: doc.mapeditor.org，policy: verified-reachable）
- https://packages.nuget.org/packages/MPewsey.ManiaMap（host: packages.nuget.org，policy: verified-reachable）
- https://assetstore.unity.com/packages/tools/level-design/world-graph-editor-306228（host: assetstore.unity.com，policy: verified-reachable）
- https://ar5iv.labs.arxiv.org/html/2202.06183（host: ar5iv.labs.arxiv.org，policy: verified-reachable）
- https://www.joelonsoftware.com/2001/10/14/in-defense-of-not-invented-here-syndrome/（host: www.joelonsoftware.com，policy: verified-reachable）
- https://www.gdcvault.com/play/1015975/Cutting-the-Pipe-Achieving-Sub（host: www.gdcvault.com，policy: verified-reachable）
