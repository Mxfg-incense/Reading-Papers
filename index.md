| 论文 | 时间/状态 | 它揭示的困难 |
|---|---:|---|
| [Where Did It Go Wrong? Process-Level Evaluation of Web Agents with Semantic State Tracking](https://arxiv.org/abs/2606.15673) | 2026-04 / arXiv，6 月被重新传播 | 只看最终成功率会遮蔽真正失败点；同样 31-33% 成功率的 agent，可能一个输在探索，一个输在执行。它用 semantic MDP 追踪过程，把失败定位到具体状态/技能/分叉点。 |
| [Signal-Driven Observation for Long-Horizon Web Agents](https://arxiv.org/abs/2606.06708) | 2026-06-04 / arXiv | DOM/AXTree 每步都太长，动辄数万 token，导致上下文退化。核心观点：观察频率不应和动作频率绑定，agent 不该每点一下就重读全网页。 |
| [WebSP-Eval: Evaluating Web Agents on Website Security and Privacy Tasks](https://arxiv.org/html/2604.06367v2) | 2026 / arXiv v2 | 隐私/安全设置页特别难。失败集中在导航、状态理解、幻觉成功、重复动作、部分完成；stateful UI 尤其糟糕，toggle 在许多模型上导致 45%+ 任务失败。 |
| [Odysseys: Benchmarking Web Agents on Realistic Long Horizon Tasks](https://arxiv.org/html/2604.24964v1) | 2026-04 / arXiv | 长程、多站点、高分叉任务仍很难。最强模型 100 步下 perfect success 约 44.5%；常见失败是一直收集信息但不产出、错误地转去脚本捷径、高 fanout 子任务覆盖不全。 |
| [Why Do LLM-based Web Agents Fail? A Hierarchical Planning Perspective](https://arxiv.org/html/2603.14248v2) | 2026-04 v2 / arXiv | 不是只会“不会规划”。即使给人类高层计划，低层执行仍是瓶颈：plan completion 38.5%、final success 36.4%；常见问题包括 hallucinated links、无状态变化的冗余动作、重复动作、跳出目标域。 |
| [WAAA! Web Adversaries Against Agentic Browsers](https://arxiv.org/html/2605.05509) | 2026-05 / arXiv | 浏览器 agent 面对真实 Web 攻击时是“confused deputy”：无法可靠区分用户目标、页面内容、攻击者诱导。论文归纳 5 类安全失败，包括跨站数据桥接、同站数据桥接、URL 幻觉、网页攻击 LLM、误用集成工具。 |
| [FocusAgent](https://openreview.net/forum?id=mINaJKSy7A) | 2026-02/05 / TMLR under review | 页面 observation 太大、太噪，完整读入增加成本和 prompt injection 风险；用轻量 LLM 从 AxTree 抽取相关行，可减少 50%+ observation，同时保持任务性能。 |
| [WebSuite: Systematically Evaluating Why Web Agents Fail](https://arxiv.org/abs/2406.01623) | 2024 / arXiv | 早期但很对题：把失败归因到具体 web action，而不是只报任务失败。说明不同 agent 的弱点会落在不同动作类型上。 |
| [An Illusion of Progress? Assessing the Current State of Web Agents](https://arxiv.org/abs/2504.01382) | 2025 / COLM 相关 | 指出很多 benchmark 过于乐观，live web 评测下能力被高估；真实网站、动态页面、多样任务会放大 sim-to-real gap。 |