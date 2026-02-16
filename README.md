# AI-Native System Builder: MyInvestPilot Architecture Series

> **[中文版本 (Simplified Chinese)](README.zh.md)**

Public architecture notes from building [MyInvestPilot](https://www.myinvestpilot.com/) — an AI-native investment system built through AI-driven development, constrained DSL design, hybrid agents, and a cost-aware solo cloud stack.

## Why This Repo Exists

This repository is the external technical narrative of MyInvestPilot: design choices, trade-offs, and failures from building a production system where AI writes most implementation code.

It is **not**:
- a product user manual (that belongs to product docs)
- an internal AI blueprint repo (used for private planning/execution)
- a generic prompt cookbook

The goal is straightforward: document what it actually takes to operate as an **AI-native system builder**, not just ship demos.

## The Series

### 01. Engine
**[From Rule-Based Scripts to AI-Native Engines: Why I Built a Constrained DSL](docs/01_ai_native_primitives_engine.md)**

How constrained IR (JSON DSL), schema-driven contracts, and deterministic execution work together for reliable decision systems.

### 02. Methodology
**[Why I Stopped Writing Code: The 60/40 Rule for AI-Native Engineering](docs/02_ai_driven_development.md)**

How AI-driven solo development evolved from 60/40 alignment/execution toward 90/10 in many cycles, with human quality gates still mandatory.

### 03. Agent Architecture
**[The Agent Paradox: Why We Built a "Boring" Hybrid Architecture](docs/03_hybrid_agent_architecture.md)**

Why we split Local Agent (flexible orchestration) and Remote Agent (deterministic async processing) instead of relying on a single autonomous agent.

### 04. Cloud Infrastructure
**[Running a Multi-Vendor, Event-Driven Cloud Architecture as a Solo Builder](docs/04_solo_company_infrastructure.md)**

How a multi-vendor, async-first, serverless-friendly stack supports real usage with low operational cost.

## Audience

This repo is for builders working on:
- AI-native application architecture
- agent systems with reliability boundaries
- cloud-native/serverless systems for solo teams

## About

I'm Dawei ([@madawei2699](https://twitter.com/madawei2699)), an independent builder.
I previously shipped [myGPTReader](https://github.com/myreader-io/myGPTReader) and [invest-alchemy](https://github.com/myinvestpilot/invest-alchemy), which later evolved into MyInvestPilot.

## License

MIT
