# Why I Stopped Writing Code: The 60/40 Rule for AI-Native Engineering

## 1. The Confession

I used to pride myself on typing speed. I knew every shortcut in VS Code. I memorized the entire Python standard library.

Today, I pride myself on how little I type.

In building MyInvestPilot (a solo-operator investment system with thousands of users and a growing paid base), I discovered a counter-intuitive truth: **The strict engineering discipline I learned in big tech—"Thinking First, Coding Later"—is the only way to survive as a solo builder in the AI era.**

If you are treating Copilot as a "better autocomplete," you are missing the point. You are just writing legacy code faster.

The real shift is: **I stopped writing implementation code.** I only write **Architecture**.

## 1.5 Reality Check: The Scale of "Solo"
Before explaining *how*, let's look at *what* actually got built. This isn't a weekend demo. MyInvestPilot has evolved into a distributed, production-grade system evolved entirely by one person + AI:

*   **Codebase**: **28 repositories** total (21 currently active).
    *   **Scale**: **~538k LOC** raw, with **~116k lines** of executable code (excluding docs/config).
*   **Infrastructure**: A hybrid edge-core architecture.
    *   **Edge**: **12 Cloudflare Workers** (Gateway, Auth, Signal Processing).
    *   **Core**: **7 Fly.io Microservices** (Strategy Engine, LLM Agents, Data Proxy).
*   **Knowledge Base (`plutus`)**:
    *   **96 Markdown/MDX files** totaling **35,886 lines** (650k+ characters).
    *   *Note*: This entire documentation library is AI-maintained.

Managing this scale solo is impossible with traditional coding. I literally *cannot* write or maintain this line-count by hand. This forced the shift.

## 2. The New "Source Code"

In 2024, Python, Rust, and TypeScript are not source code. They are **intermediate assembly languages**. They are the compilation target of your intent.

So what is the *actual* source code?

1.  **Mermaid Diagrams**: The structural source of truth.
2.  **OpenAPI / JSON Schemas**: The contract source of truth.
3.  **Gherkin / English Specs**: The behavioral source of truth.

When I need to change a feature, I don't touch the Python file. I update the Schema. I update the Diagram. Then I tell the AI: *"Align the implementation with this new contract."*

This isn't "No-Code" (which is usually a toy). This is **"Architecture-First Code"**.

## 3. The 60/40 Rule (Reverse Waterfall)

My development cycle for any new service (like the `investDataAPI` in my repo) follows a strict ratio:

*   **60% Alignment (No Code)**: Defining intent, boundaries, and contracts.
*   **40% Execution (AI Coding)**: Generating, testing, and refining implementation.

Most developers do the opposite: 10% planning, 90% debugging a hallucinating AI. That is a death spiral.

### The Problem with "Chat Coding"
If you just open Cursor and say "Make me a market data service," you get a demo.
*   It lacks error handling.
*   It ignores API rate limits (Alpha Vantage has strict quotas).
*   It couples implementation to a specific provider (e.g., FMP).

To build a production system, you must **constrain the efficient compiler (AI)** with **explicit constraints (Artifacts)**.

## 4. War Story: The 1-Week Data Service

Let's look at a real example from my codebase: `investDataAPI`.
This is a mission-critical service that aggregates financial data from Alpha Vantage, FMP, and AkShare.

**The Challenge**: Users customize portfolios, triggering massive concurrent data requests. Direct API calls fail immediately due to rate limits.
**The Fix**: An async task queue architecture (Submit Task -> ID -> Poll Redis).

Here is the **Real 60/40 Breakdown**:

### Days 1-4: The Blueprint Phase (60%)
I wrote **zero** lines of Python. I spent 4 days arguing with **Gemini 1.5 Pro** and **Claude 3.5 Sonnet**.

*   **Artifact 1: The Context Snapshot**. I fed the *entire* legacy codebase (thousands of lines) into **Gemini 1.5 Pro** (using its 1M+ token window).
    *   *Prompt*: "Based on the existing `invest-ohlc-proxy`, design an async fallback strategy for the new service. What breaks?"
    *   *Result*: Gemini spotted a hidden dependency in the common library that I had forgotten.
*   **Artifact 2: The Contract**. I defined the `MarketAnalysisRequest` schema.
*   **Artifact 3: The Infrastructure**. We decided to use **Fly.io** for deployment and `fly proxy` for local Redis tunneling. This was a critical constraint for the testing strategy.

### Days 5-7: The "Triangular Loop" (40%)
Once the architecture was locked, I entered the execution phase. But I didn't just "vibe code." I used a **Triangular Feedback Loop**:
1.  **Creator (Me + Claude/Cline)**: Generate the feature branch.
2.  **Reviewer (CodeRabbit)**: An AI agent that reviews every PR.
3.  **Auditor (Me)**: I read CodeRabbit's feedback and feed it back to Claude.

*   **The Coding**: In one session, the AI generated the entire `clients/` directory.
*   **The Review**: **CodeRabbit** immediately flagged a subtle bug: *The `aiohttp` session wasn't being closed properly in the exception path.*
    *   *Insight*: Claude missed it. I missed it. The specialized Reviewer Agent caught it.
*   **The Integration**: I ran the integration tests using `fly proxy` to tunnel production Redis to my local machine.

### The Result
A robust, async, multi-provider data service with smart fallbacks.
**Time**: 7 Days (Solo).
**Code Written by Me**: < 5% (mostly config, tests, and integration glue).

## 5. The Evolution of My Stack (A Moving Target)

Tools matter, but they change every month. The key is understanding *why* you use them.

### Phase 1: The "Chat" Era (GPT-4)
*   **Role**: Senior Pair Programmer.
*   **Workflow**: Copy-paste code into web UI. Manual context management.
*   **Verdict**: Good for logic, bad for large-scale architecture.

### Phase 2: The "Context War" (Gemini 1.5 Pro)
*   **Role**: The Accountant / Architect.
*   **Workflow**: Feeding the *entire* repo (2M tokens) to check for hidden dependencies before starting.
*   **Verdict**: Essential for "Measure twice, cut once," but slow for execution.

### Phase 3: The "Agentic" Era (Today)
*   **The Executor**: **OpenCode / Claude Code (CLI)**.
    *   *Why*: This is my daily driver now. It lives in the terminal. It has direct access to the file system and tools.
    *   *Role*: I define the spec (schema/instruction), and **OpenCode** executes the entire loop (Edit -> Run -> Error -> Fix) autonomously. It's not just a "Typist"; it's a "Junior Engineer" with infinite stamina.
*   **The Reviewer**: **CodeRabbit**.
    *   *Why*: The "Auditor". It catches logic errors, security flaws, and style issues that the Executor misses. It's the critical "check" in the loop.

## 6. The "Senior Engineer" Redefined

In an AI-native world, "Senior" doesn't mean you write complex algorithms from scratch.
It means you can **define unambiguous constraints**.

If you can describe a system so clearly that a junior engineer (or an AI) cannot mess it up, you are a Senior AI-Native Builder.
If you need to tweak every line of code yourself, you are just a bottleneck.

### Core Design Principles

1.  **Code is a liability, not an asset.** (Focus on the 60% Blueprint).
2.  **Constraints liberate intelligence.** (Schemas and Mermaid are the rails).
3.  **Optimize for collaboration, not speed.** (Use the Triangular Feedback Loop).
4.  **Reproducibility is the test of seniority.** (Artifacts must consistently generate correct code).

## 7. Join the Discussion

This is just one piece of how I'm approaching AI-native systems. My GitHub ([github.com/madawei2699](https://github.com/madawei2699)) has related explorations in AI-driven development.

If you are a solo builder feeling the same shift, I’d love to hear your ratio today and what artifacts work best for you.

If you're also experimenting with "No-Code" architecture or AI-Native workflows, I'd love to hear your take—fork, critique, or just chat on X ([@madawei2699](https://twitter.com/madawei2699)). Open to collaborations or just geeking out over similar ideas—DMs open.
