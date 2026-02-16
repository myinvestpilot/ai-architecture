# From Open-Source Quant Libs to AI-Native Engines: Why I Built a DSL for Investment

## 1. The Builder's Journey

Hi, I'm Dawei ([@madawei2699](https://github.com/madawei2699)), an independent builder who's shipped open-source tools like [myGPTReader](https://github.com/myreader-io/myGPTReader) (4.4k stars) and [invest-alchemy](https://github.com/myinvestpilot/invest-alchemy) (750+ stars).

Now, through MyInvestPilot, I'm exploring how to make LLMs truly reliable for composing complex, verifiable decision logic. Whether you're in quant trading, AI agents, or building constrained decision systems, I hope this resonates.

## 2. The Secret Sauce: Schema-Driven, Auto-Generated Prompt Engineering

The real magic isn't just the DSL; it's how we teach the AI to use it.

I treat the System Prompt as **software**, not text. The [Quickstart Guide](https://www.myinvestpilot.com/docs/primitives/_llm/llm-quickstart.txt) (publicly viewable – feel free to study how it enforces DAGs, strict typing, and PIT rules) is **auto-generated from the JSON Schema** during the build process.

*   **Versioned & Synced**: When I add a new primitive (e.g., `CryptoSentiment`), the schema updates, and the prompt automatically includes it. No drift between the engine and the AI's instruction manual.
*   **Mental Model Enforcement**: It explicitly forces the AI to think in **DAGs (Directed Acyclic Graphs)**, not procedural code. It warns against common hallucinations (like inline definitions) with "❌ WRONG / ✅ CORRECT" examples.
*   **Domain Constraints**: It enforces strictly point-in-time correctness for fundamental data, preventing the AI from accidentally creating look-ahead bias.

This turns the LLM from a "creative writer" into a **reliable compiler frontend**.

## 3. The Builder's Journey

My journey didn't start with a business plan; it started with code. I spent years building `invest-alchemy` for developers, but I hit a wall: **The barrier to sophisticated decision-making was still too high.**

When ChatGPT arrived, I saw an opportunity. But I also saw a problem.

## 4. The Bottleneck: The "Python Trap"

In my current project (a systematic investment platform called MyInvestPilot), I needed a way for users—and AI—to define complex strategies.

The standard playbook was: **Python scripts**.
*   **Safety**: How do I stop a user's code from crashing my server or stealing data?
*   **Complexity**: Most users aren't engineers. They understand "Golden Cross," not `pandas.DataFrame`.
*   **AI Mutation**: I tried letting GPT-3.5 write these Python strategies. It was a disaster. It would hallucinate libraries, write infinite loops, or use future data (look-ahead bias).

I realized that **Code is not the right interface for AI-driven decisions.** Code is too broad. It allows *anything* to happen.

To build a truly AI-Native system, I needed a **General Decision Engine**.

## 5. The Exploration: Why Not Code or APIs?

I didn't start with a DSL. In fact, DSLs are hard to build. I wanted the simplest solution that would allow AI (Agents) to build strategies for users.

I evaluated three paths:

### Path A: Code Generation + Sandbox (The "Natural" Choice)
The most obvious idea: *Just let the AI write Python code and run it in a secure Sandbox (Docker/Wasm).*
*   **Pros**: Infinite flexibility. AI knows Python well.
*   **Cons**:
    *   **Hallucination**: The AI imports libraries that don't exist. It writes `while True` loops that hang the sandbox.
    *   **Safety**: Even with a sandbox, malicious logic (like infinite resource consumption) is hard to contain.
    *   **Maintenance**: When the AI writes a bug, the user can't fix it because they don't know Python.

### Path B: The "Super API" (REST/GraphQL)
The standard SaaS approach: *Build a massive API endpoint where users config params.*
*   `POST /create-strategy { "ma_period": 20, "rsi_threshold": 30 ... }`
*   **Pros**: Safe. Easy to validate.
*   **Cons**:
    *   **Parameter Explosion**: To support "RSI > 30 AND (MA50 > MA200 OR Volume > 1M)", the API parameters became a nightmare of nested objects.
    *   **Logic Limits**: You can't easily express *logic* (If-This-Then-That) in a standard REST payload without inventing a clumsy JSON structure anyway.

### Path C: The Decision (DSL)
I realized I needed the **Safety of an API** but the **Expressiveness of Code**.
I needed a **Domain Specific Language**.

*   **Constraint**: The AI can only use valid "Primitives" (LEGO blocks). It cannot import `os` or `sys`.
*   **Expressiveness**: It can combine these blocks orthogonally (Trend + Logic + Risk).
*   **Visual**: Because it's structured data (JSON), I can build a [visual editor](https://www.myinvestpilot.com/primitives-editor) (a quick visual demo I built for instant verification) to verify AI output instantly.

## 6. The Solution: Orthogonal Primitives (Investment IR)

This engine treats investment not as a script, but as a configuration of **Orthogonal Truths**:
*   **Trend**: Where is the market going? (SMA, EMA)
*   **Value**: Is it cheap? (RSI, PE_TTM)
*   **Quality**: Is the asset healthy? (ROE, Cashflow)

By combining these atomic units, we can express infinite complexity without writing code.

### Validation: A Multi-Factor Risk Strategy

To validate this engine, I implemented a complex multi-factor risk model. This isn't a toy example; it demonstrates how the DSL handles technical trend following, fundamental health checks, and dynamic position sizing in a single declarative graph.

**The Logic**:
1.  **Trend Filter**: Market must be in an uptrend (EMA50 > EMA200).
2.  **Quality Gate**: Company must have ROE > 10% and Positive Cashflow.
3.  **Safety Gate**: Debt Ratio must be under 80%.
4.  **Dynamic Sizing**: If Debt is high, reduce position size (Linear Weighting).

**The AI-Generated Configuration (JSON)**:

```json
{
  "fundamental_inputs": {
    "metrics": ["roe", "pe_ttm", "revenue_yoy", "operating_cashflow", "debt_ratio"],
    "frequency": "daily",
    "point_in_time": true, 
    "max_staleness_days": 120
  },
  "trade_strategy": {
    "indicators": [
      { "id": "ema_fast", "type": "EMA", "params": { "period": 50, "column": "Close" } },
      { "id": "ema_slow", "type": "EMA", "params": { "period": 200, "column": "Close" } },
      { "id": "roe_metric", "type": "ROE" },
      { "id": "debt_metric", "type": "DebtRatio" },
      { "id": "cashflow_metric", "type": "OperatingCashflow" },
      { "id": "roe_min", "type": "Constant", "params": { "value": 10.0 } },
      { "id": "debt_max", "type": "Constant", "params": { "value": 80.0 } },
      { "id": "cashflow_min", "type": "Constant", "params": { "value": 0.0 } }
    ],
    "signals": [
      // 1. Technical Trend
      {
        "id": "trend_up",
        "type": "GreaterThan",
        "inputs": [{ "ref": "ema_fast" }, { "ref": "ema_slow" }]
      },
      // 2. Fundamental Gates
      {
        "id": "roe_ok",
        "type": "GreaterThan",
        "inputs": [{ "ref": "roe_metric" }, { "ref": "roe_min" }]
      },
      {
        "id": "debt_ok",
        "type": "LessThan",
        "inputs": [{ "ref": "debt_metric" }, { "ref": "debt_max" }]
      },
      {
        "id": "cashflow_ok",
        "type": "GreaterThan",
        "inputs": [{ "ref": "cashflow_metric" }, { "ref": "cashflow_min" }]
      },
      // 3. Combine Logic (AND)
      {
        "id": "risk_gate",
        "type": "And",
        "inputs": [
          { "ref": "trend_up" },
          { "ref": "roe_ok" },
          { "ref": "debt_ok" },
          { "ref": "cashflow_ok" }
        ]
      },
      // 4. Dynamic Position Sizing based on Debt
      {
        "id": "debt_scaled_weight",
        "type": "LinearScaleWeight",
        "inputs": [{ "ref": "debt_metric" }],
        "params": {
          "min_indicator": 20.0, "max_indicator": 80.0,
          "min_weight": 1.0, "max_weight": 0.45,
          "clip": true
        }
      },
      // 5. Final Execution Signal
      {
        "id": "target_weight_signal",
        "type": "Multiply",
        "inputs": [{ "ref": "debt_scaled_weight" }, { "ref": "risk_gate" }]
      }
    ],
    "outputs": {
      "buy_signal": "risk_gate",
      "target_weight": "target_weight_signal"
    }
  }
}
```

This configuration is **Stateless** and **Declarative**.
The engine doesn't know *how* to calculate EMA; it just knows *that* it needs an EMA. This separation allows us to optimize the engine (using Rust, columnar data, etc.) without breaking user strategies.

## 7. Why This Matters for AI

The true power of this DSL is that it makes AI a **Strategy Architect**.

If we used Python, the AI would be a "Coder"—and a bad one at that. It would make syntax errors.
But with DSL, the AI is constrained to a **Schema**. It can only combine valid blocks.

1.  **User Prompt**: "Help me build a conservative strategy."
2.  **System Prompt**: It is fed the [AI Quickstart Guide](https://www.myinvestpilot.com/docs/primitives/_llm/llm-quickstart.txt) (a prompt template I use to guide the LLM) as context.
3.  **AI Output**: Generates valid JSON (like above).
4.  **Verification**: The [visual editor](https://www.myinvestpilot.com/primitives-editor) (a quick visual demo I built for instant verification) renders the JSON as immediate feedback.

This loop—Input -> JSON -> Visual Verification -> Execution—is the **Holy Grail of AI UX**. It transforms the AI from a chatbot into a reliable tool for building complex systems.

## 8. Join the Discussion

This is just one piece of how I'm approaching AI-native systems. My GitHub ([github.com/madawei2699](https://github.com/madawei2699)) has related explorations in LLM agents and workflows.

If you're building similar constrained AI decision engines, I'd love to hear your take—fork, critique, or just chat on X ([@madawei2699](https://twitter.com/madawei2699)). Open to collaborations or just geeking out over similar ideas—DMs open.
