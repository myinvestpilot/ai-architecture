# AI Native 系统构建实录：MyInvestPilot 架构系列

> **[English Version](README.md)**

这是 MyInvestPilot 的对外技术叙事仓库，聚焦架构设计、工程取舍和踩坑复盘。

它关注的不是“怎么写一个炫技 Prompt”，而是：
- 如何把 LLM 约束进可验证的系统边界
- 如何在 AI 写大部分实现代码的前提下保持质量
- 如何用一人团队可负担的成本运行复杂系统

## 这个仓库是什么 / 不是什么

这是：
- 面向外部的架构与方法论展示
- 以 MyInvestPilot 为中心的 AI Native 系统实践复盘
- 用于沉淀 AI native system builder 能力的公开内容

这不是：
- 产品使用文档（在独立文档站）
- 内部 AI blueprint 执行仓库
- 通用 Prompt 模板库

## 系列文章

### 01. 引擎篇
**[从规则脚本到 AI Native 引擎：我为什么做了受约束 DSL](docs/zh/01_ai_native_primitives_engine.md)**

讲清楚受约束 IR（JSON DSL）、Schema 契约和确定性执行，如何组合成一个可复现、可审计的决策引擎。

### 02. 方法篇
**[为什么我越来越少写代码：AI Native 工程里的 60/40 法则](docs/zh/02_ai_driven_development.md)**

讲清楚从 60/40（对齐/执行）到很多迭代接近 90/10 的演进，以及为什么资深工程师仍然是质量闸门的核心。

### 03. 架构篇
**[Agent 悖论：为什么我们做的是“无聊但可靠”的混合架构](docs/zh/03_hybrid_agent_architecture.md)**

讲清楚 Local Agent（灵活编排）与 Remote Agent（确定性异步处理）的分层边界，避免“全能 Agent”带来的不稳定性。

### 04. 基建篇
**[一人公司的云架构：多供应商、事件驱动与低成本运行](docs/zh/04_solo_company_infrastructure.md)**

讲清楚多 vendor、EDA、队列驱动扩缩容和 serverless 友好设计如何共同支撑低成本运行。

## 适合谁看

如果你正在做这些方向，这个仓库会更有价值：
- AI Native 应用/系统设计
- Agent 应用中的可靠性边界设计
- 一人公司场景下的 cloud native / serverless 工程实践

## 关于

我是 Dawei ([@madawei2699](https://twitter.com/madawei2699))，独立开发者。
此前发布过 [myGPTReader](https://github.com/myreader-io/myGPTReader) 和 [invest-alchemy](https://github.com/myinvestpilot/invest-alchemy)；后者是 MyInvestPilot 的早期开源前身。

## License

MIT
