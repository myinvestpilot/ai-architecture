# 为什么我越来越少写代码：AI Native 工程里的 60/40 法则

## 1. 忏悔（The Confession）

我以前很在意写代码速度。
现在更在意的是：我是否把系统边界定义清楚了。

在 MyInvestPilot 的长期迭代里，我逐步确认了一件事：如果把 AI 只当“更快的补全工具”，你只是更快地产生技术债。真正的转变是把自己从“实现者”切换为“架构约束定义者”。
这个系统目前已承载数千次试用交互，并形成持续增长的付费用户基础。

这也是为什么我常说：**先思考，后编码（Thinking First, Coding Later）在 AI 时代反而更重要。**

## 1.5 现实检验：Solo 的规模（Reality Check: The Scale of "Solo"）

这个方法不是口号，而是规模倒逼出来的。

- **代码规模**：约 28 个仓库，总计约 53.8 万行，约 11.6 万行可执行核心代码。
- **运行规模**：边缘侧约 12 个 Cloudflare Workers，核心侧约 7 个 Fly.io 服务。
- **文档规模（`plutus`）**：96 篇 Markdown/MDX，35,886 行，65 万+ 字符。

文档库同样是 AI 大量参与生成，我做质量闸门和最终裁决。
在这个体量下，手写和手工维护已经不是可持续方案。

## 2. 新的“源码”（The New "Source Code"）

在我的实践里，Python、Rust、TypeScript 更像“编译目标”，而不是系统真相本身。
真正的“源码”是这些可审阅的工件（artifacts）：

1. **Mermaid 图**：结构边界。
2. **OpenAPI / JSON Schema**：接口与类型契约。
3. **English Specs / Gherkin**：行为约束。
4. **规划快照（Snapshot）**：把 issue 讨论结果压缩成 AI 可执行上下文。

我通常先把零散想法记录在 issue，再与 Gemini / ChatGPT / Grok 做检索增强讨论，最后沉淀成“AI-facing planning docs”。
复杂阶段会配合一个私有 blueprint 仓库维护长期上下文，但对外叙事只暴露方法，不暴露内部资产。

## 3. 60/40 法则（Reverse Waterfall）

早期阶段，我大致是 **60% 对齐 + 40% 执行**：

- **60% Alignment（不写代码）**：定义边界、契约、计划与验收标准。
- **40% Execution（AI 执行）**：生成实现、运行测试、修复回归。

随着模型和工具演进，很多迭代已经接近 **90/10**，有些任务在对齐稳定后接近“零手写代码”。

但瓶颈没有消失，只是从“写代码”迁移到“对齐系统”：一旦对齐失败，后续会在跨服务 patch、回归和返工里成倍偿还。

### Chat Coding 的问题（The Problem with "Chat Coding"）

直接说一句“帮我写个行情服务”，通常只能得到 demo：

- 缺少异常和限流处理。
- 容易把实现绑定到单一数据提供方。
- 缺少跨服务契约检查，后续联调成本很高。

所以我更关注“先收敛生成空间，再放大执行速度”。

## 4. 实战：7 天交付一个分布式服务（War Story）

以 `investDataAPI` 为例，它要聚合 Alpha Vantage / FMP / AkShare，并且在并发高峰下稳定工作。
核心挑战是：同步直连会被限流击穿，必须转成异步任务队列。

### Days 1-4: The Blueprint Phase (60%)

这 4 天我几乎不写实现代码，主要做三件事：

- **上下文快照**：把已有代码和历史设计压缩成可讨论快照，让模型先发现隐藏依赖。
- **契约先行**：定义 `MarketAnalysisRequest` 及错误路径。
- **部署约束**：明确本地和线上一致的验证路径（例如通过 `fly proxy` 进行集成验证）。

这一步的价值不是“更聪明”，而是减少后续跨仓库返工。

### Days 5-7: The "Triangular Loop" (40%)

架构锁定后进入执行闭环。我采用一个三角回路：

1. **Creator（我 + OpenCode / Codex / Claude Code / Copilot）**：生成分支与补丁。
2. **Reviewer（CodeRabbit / Gemini Review Bot）**：PR 级别审查。
3. **Auditor（我）**：裁决反馈并回传修补指令，直到收敛。

其中一个典型案例是 `aiohttp` 会话在异常路径未关闭。
生成模型和人工都漏掉了，评审 Agent 抓到后回传修补，最终通过测试。

### The Result

- **交付时间**：7 天（solo）。
- **手写代码占比**：< 5%，主要是 glue code、测试和集成收口。
- **关键结论**：在复杂系统里，评审链路和生成链路同等重要。

## 5. 工具链演进（A Moving Target）

工具每几个月就会变，流程骨架基本不变。

### Phase 1: Chat Era (GPT-4)

- 角色：Pair Programmer。
- 特点：逻辑讨论有效，长链路工程能力有限。

### Phase 2: Context War (Gemini 1.5 Pro)

- 角色：依赖审计员 / 规划助手。
- 特点：大上下文适合做“改动前审计”，执行速度一般。

### Phase 3: Agentic Era (Today)

- 主要执行：**OpenCode / Codex / Claude Code / GitHub Copilot**（按额度和任务类型切换）。
- 对齐与研究：**Gemini / ChatGPT / Grok**（利用在线检索做事实和方案讨论）。
- 交互 IDE：**Cursor / Windsurf**（快速编辑与局部验证）。

我当前常用工具链可以概括为：**Claude、Claude Code、Cursor、Windsurf、Gemini、Codex、OpenCode、GitHub Copilot**。

## 6. 重新定义 Senior Engineer

AI 能生成大量代码，不代表复杂工程不需要资深工程师。
在我的经验里，人类的不可替代职责是三件事：

1. **一致性控制**：跨服务、跨仓库的契约一致性。
2. **边界控制**：明确什么能让模型做，什么必须人工兜底。
3. **质量控制**：PR 评审、回归裁决、发布门禁。

换句话说，今天的 Senior 不再以“手写多少代码”定义，而以“是否能持续把复杂系统收敛到可发布状态”定义。

### 一个可复用的执行模板（我当前的默认流程）

1. **Capture**：把零散想法先写进 issue，避免只停留在聊天窗口。
2. **Research + Align**：用 Gemini / ChatGPT / Grok 做检索增强讨论，反复收敛约束与验收标准。
3. **Plan Freeze**：把可执行计划固化到 issue 或 planning doc，作为本轮唯一输入契约。
4. **AI Execution**：用 OpenCode / Codex / Claude Code / Copilot 按计划实现，优先让它们自己完成 edit-run-fix 循环。
5. **PR Ping-Pong**：提交 PR 后交给 CodeRabbit / Gemini Review Bot 审查，我基于反馈继续驱动模型修复。
6. **Gate + Release**：无重大反馈、测试通过后发布；需要本地验证的链路，人工做最后一跳。

这套流程的重点不是“让 AI 自由发挥”，而是“让 AI 在清晰边界内高吞吐执行”。

### 核心设计原则

1. **Code is liability**：代码是负债，不是产出本身。
2. **Artifacts first**：先定义契约，再生成实现。
3. **Review is architecture**：评审链路是架构的一部分。
4. **Reproducibility over velocity**：可复现优先于一次性速度。

## 7. 加入讨论（Join the Discussion）

这套方法论会持续记录在我的公开架构仓库：
[myinvestpilot/ai-architecture](https://github.com/myinvestpilot/ai-architecture)（第一篇 DSL 引擎文章也在其中）。

如果你也在做 AI 驱动的复杂系统，欢迎交流你当前的“对齐/执行”比例，以及你最有效的质量门禁做法。X：[@madawei2699](https://twitter.com/madawei2699)。
