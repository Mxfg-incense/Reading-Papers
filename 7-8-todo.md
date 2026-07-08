下面是截至 **2026-07-09** 我查到的脉络。结论先说：**Microsoft UI Automation/UIA 更像 Windows 桌面自动化的“结构化感知层”，而现在的 Computer Use 论文主线更偏“视觉截图 + 坐标动作 + 长任务规划”。最值得做的方向，是把 UIA tree 和视觉模型结合起来。**

**1. 基础层：Microsoft UI Automation / UIA**

Microsoft UI Automation 是 Windows 的无障碍与自动化框架，能暴露桌面应用 UI 元素、属性、Control Pattern、事件等，既用于屏幕阅读器，也用于自动化测试。[Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-uiautomationoverview) 明确说它可让脚本与 UI 交互。它是 MSAA 的后继，微软也建议新应用优先支持 UIA 而不是旧的 Microsoft Active Accessibility。[对比文档](https://learn.microsoft.com/en-us/windows/win32/winauto/microsoft-active-accessibility-and-ui-automation-compared)

UIA 的核心价值：比纯截图更稳定，因为可以拿到按钮、文本框、菜单、窗口层级、可调用动作等结构化信息。缺点是覆盖不完美：Electron、Canvas、自绘控件、游戏、远程桌面、某些浏览器区域和老旧软件可能暴露得很差。

**2. UIA 相关开源项目**

| 项目 | 作用 | 备注 |
|---|---|---|
| [pywinauto](https://github.com/pywinauto/pywinauto) | Python Windows GUI 自动化 | 支持 `backend="win32"` 和 `backend="uia"`，适合脚本/RPA/测试 |
| [Python-UIAutomation-for-Windows](https://github.com/yinkaisheng/Python-UIAutomation-for-Windows) | Python UIA wrapper | 直接面向 Microsoft UIAutomation，支持 WPF/WinForms/Qt/Chrome 等 |
| [FlaUI](https://github.com/FlaUI/FlaUI/blob/main/README.md) | .NET UIA wrapper | C# 生态里很常用，封装 UIA2/UIA3，适合测试工程 |
| [FlaUInspect](https://github.com/FlaUI/FlaUInspect) | UIA inspect 工具 | 查看控件树、AutomationId、Pattern，调试必备 |
| [WinAppDriver](https://github.com/microsoft/winappdriver) | 微软的 Selenium-like Windows UI 测试服务 | 支持 UWP/WinForms/WPF/Win32，但社区普遍认为维护活跃度偏低 |
| [Appium Windows Driver](https://github.com/appium/appium-windows-driver) | Appium 对 WinAppDriver 的代理 | 适合已有 Appium/Selenium 测试栈 |
| [FlaUI-MCP](https://github.com/shanselman/FlaUI-MCP) | 用 MCP 暴露 Windows 桌面自动化能力 | 很适合把 UIA 接给 LLM agent，当“桌面版 Playwright”来用 |
| [Windows-Use](https://github.com/CursorTouch/Windows-Use) | LLM + Windows UIA 的 Computer Use agent | 直接读取 UIA tree，让 LLM 决定点击/输入/滚动，不依赖纯视觉模型 |

**3. Computer Use / GUI Agent 论文主线**

| 论文/系统 | 重点 | 价值 |
|---|---|---|
| [OSWorld](https://proceedings.neurips.cc/paper_files/paper/2024/file/5d413e48f84dc61244b6be550f1cd8f5-Paper-Datasets_and_Benchmarks_Track.pdf) | 真实 OS 任务 benchmark | Computer Use 领域事实标准之一 |
| [OSWorld 2.0](https://arxiv.org/abs/2606.29537) | 长任务、真实工作流 | 2026 新论文，108 个长流程任务，显示当前 agent 距专业级使用仍很远 |
| [Windows Agent Arena](https://arxiv.org/abs/2409.08264) / [项目页](https://microsoft.github.io/WindowsAgentArena/) | Windows 桌面 agent benchmark | 154 个 Windows 任务，覆盖 Edge/Chrome/VS Code/Office 类应用等 |
| [WorldGUI](https://arxiv.org/html/2502.08047v4) | 动态初始状态桌面 GUI benchmark | 强调 agent 对非默认状态、恢复、适应能力 |
| [OmniParser](https://github.com/microsoft/omniparser) / [V2 博文](https://www.microsoft.com/en-us/research/articles/omniparser-v2-turning-any-llm-into-a-computer-use-agent/) | 把截图解析为结构化 UI 元素 | 微软路线，偏视觉解析，不依赖 UIA |
| [Agent S](https://arxiv.org/abs/2410.08164) / [GitHub](https://github.com/simular-ai/agent-s) | 经验增强的层级规划 GUI agent | OSWorld 上强 baseline，强调 ACI、经验检索、长程规划 |
| [Agent S2](https://arxiv.org/abs/2504.00906) | generalist + specialist 组合 | 用 Mixture-of-Grounding 改善定位，用分工模型改善规划 |
| [OpenCUA](https://arxiv.org/abs/2508.09123) / [GitHub](https://github.com/xlang-ai/OpenCUA) | 开放数据、模型、训练框架 | AgentNet 覆盖 3 个 OS、200+ 应用/网站，是开源 CUA 的重要基础设施 |
| [Fara-7B](https://github.com/microsoft/fara) / [微软博客](https://www.microsoft.com/en-us/research/blog/fara-7b-an-efficient-agentic-model-for-computer-use/) | 微软 7B Computer Use 小模型 | 开权重，专注网页任务，纯视觉坐标动作，不依赖 accessibility tree |
| [PC Agent-E](https://arxiv.org/html/2505.13909v1) | 高效训练 CUA | 关注用少量人工轨迹 + 合成数据训练 |
| [A Comprehensive Survey of Agents for Computer Use](https://arxiv.org/html/2501.16150v3) | 综述 | 适合快速建立全局地图 |

**4. 大厂产品/接口方向**

OpenAI 的 [Computer-Using Agent](https://openai.com/index/computer-using-agent/) 走截图感知 + 动作执行路线，官方称 CUA 在 OSWorld 上达到 38.1%。OpenAI API 文档也建议长流程可结合 Playwright/PyAutoGUI 这类运行时工具。[OpenAI Computer Use API](https://developers.openai.com/api/docs/guides/tools-computer-use)

Anthropic 的 [Claude computer use](https://www.anthropic.com/news/developing-computer-use) 是较早公开 API 的桌面控制能力，通过截图、鼠标、键盘与虚拟桌面交互。[平台文档](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool) 也把它定义为 beta 桌面自动化工具。

微软这边有两条线：一条是 Copilot Studio 的 enterprise “computer-using agents”，偏治理、Cloud PC、凭据和监控；另一条是研究/开源线，如 Windows Agent Arena、OmniParser、Fara-7B。[Copilot Studio 博文](https://www.microsoft.com/en-us/microsoft-copilot/blog/copilot-studio/computer-using-agents-now-deliver-more-secure-ui-automation-at-scale/)

**5. 我的判断**

UIA 不是当前论文主线的中心，但它在 Windows 上非常实用。论文喜欢用视觉，是因为跨平台、对自绘 UI 更通用、和 VLM 训练范式一致；工程落地喜欢 UIA，是因为便宜、稳定、可解释、token 少。

最值得关注的研究/工程结合点：

1. **UIA tree + screenshot 双模态 grounding**：UIA 提供候选元素和语义，截图补 UIA 缺失区域。
2. **LLM tool interface 类似 Playwright for Desktop**：FlaUI-MCP 这个方向很有潜力。
3. **从 UIA 录制人类轨迹**：用于 OpenCUA/AgentNet 类数据采集，比纯坐标日志更有语义。
4. **可靠性评估**：用 Windows Agent Arena / WorldGUI / OSWorld 做 benchmark，再区分“UIA agent”和“vision agent”的失败模式。
5. **企业自动化**：UIA 更适合受控 Windows 桌面、内部软件、legacy app；纯视觉更适合任意网页/远程环境。

如果你要快速上手，我会按这个顺序读/试：**FlaUI-MCP 或 Windows-Use → Windows Agent Arena → OmniParser → Agent S/S2 → OpenCUA → OSWorld 2.0**。这条线能同时覆盖“能马上跑的工程”和“最新研究问题”。



























