# cert-engine

> Monorepo for the **From intermediate to expert in Claude, AI, and agents** practical portfolio — 8 cumulative projects that build a **certification prep engine**.

This repo is my engineering journey, not a deployed product. The product lives separately: [passthecert.com](https://passthecert.com), the certification prep MVP (currently focused on CompTIA Security+). Here I build and **measure** the reusable pieces of the engine; when a piece is proven, it *graduates* to passthecert.

## Guiding design decision

I'm not building "for Security+". I'm building a **cert-agnostic prep engine** where Security+ is the first instance and a cloud certification will be the second. Everything I design (prompts, evals, RAG, skills) treats the certification as **input/configuration**, never as something hardcoded to a single exam.

**Legal/ethical caveat:** all material is **original** practice content, anchored to the official exam objectives. We never dump real questions or protected material from CompTIA or any other vendor.

## How it's organized

Monorepo, one folder per project. Each project reuses pieces from the previous one and everything converges in the capstone (an "Operator" that produces and manages cert content end-to-end).

| # | Project | Focus | Weeks | Feeds | Status |
|---|---------|-------|-------|-------|--------|
| 1 | [Prompt Lab & Evaluations](./01-prompt-lab/) | Rigorous prompt engineering with metrics | 1–2 | All | 🟡 In progress (v0.1) |
| 2 | Skills Library | Modular reuse with Claude Skills | 3–4 | 3, 6, 7 | ⬜ Pending |
| 3 | Assistant with persistent memory | State and context across sessions | 5–6 | 5, 6, 8 | ⬜ Pending |
| 4 | Custom MCP Server | Expose my own tools to Claude | 7–9 | 6, 7, 8 | ⬜ Pending |
| 5 | Enterprise RAG agent | Semantic search over documents | 10–12 | 6, 8 | ⬜ Pending |
| 6 | Multi-agent orchestrator | Coordinate specialized agents | 13–15 | 8 | ⬜ Pending |
| 7 | Distributable plugin | Package and share | 16–17 | 8 | ⬜ Pending (may move to its own public repo) |
| 8 | Capstone: AI Business Operator | End-to-end integration | 18–22 | — | ⬜ Pending |

### Stack per project

Language is chosen per project based on fit, not for repo uniformity:

- **Project 1 — Prompt Lab:** TypeScript (where I'm most fluent).
- **Project 5 — RAG agent:** **Python** is the right fit. The ecosystem for embeddings, vector stores (Chroma, LanceDB, FAISS), and ML tooling is significantly more mature and better documented in Python; forcing TS there would add friction without benefit.
- **Projects 2–4, 6–8:** to be decided when I get to each one.

## Model policy

Claude model names and prices change; **never assume them from memory**. When it's time to pick or compare models, fresh information is fetched and the decision is made **by metrics** (accuracy, cost, latency), not by intuition.

The **Prompt Lab (Project 1)** is also the comparison and migration harness: same prompt + dataset, multiple models, decision by the numbers. The **migration rule is explicit and written down** — see [`01-prompt-lab/README.md`](./01-prompt-lab/README.md).

## Working principles

- Build small, iterate fast: every project starts with a working v0.1.
- Evaluate everything: no project ships without its evaluation step.
- Reuse: skills, prompts, and modules from earlier projects integrate into the next ones.
- Document while you build: one README per project.
- Ship v0.1 before polishing; write the README first.

## Deliverables per project

Each project closes with: a README, before/after metrics, and a short thread/post that explains it.
