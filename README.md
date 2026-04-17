# AI Reliability Architecture

A framework for building AI automation systems that stay reliable in production. The core problem: LLMs make good decisions but produce inconsistent outputs. This architecture separates decision-making from execution so each layer does only what it's good at.

The `CLAUDE.md` in this repo is the actual agent configuration used across production systems. Drop it in any repo and Claude Code picks it up automatically.

---

## The Problem with Naive AI Automation

When an AI agent handles everything itself, errors compound:

- 90% accuracy per step = 59% success over 5 steps
- Errors at step 2 corrupt everything downstream
- When something fails, there's no clean place to fix it
- The agent can't learn from failures - next session starts from scratch

---

## The Solution: 3-Layer Architecture

```
Layer 1: Directives  (directives/*.md)
-----------------------------------------
Natural-language SOPs. What to accomplish,
what tools to use, what edge cases to handle.
Written like instructions to a capable employee.

          reads
            |
            v

Layer 2: Orchestration  (Claude / AI agent)
-----------------------------------------
Reads directives. Decides which scripts to call
and in what order. Handles errors. Asks for
clarification when inputs are ambiguous.
Updates directives with new learnings.

          calls
            |
            v

Layer 3: Execution  (execution/*.py)
-----------------------------------------
Deterministic Python scripts. API calls, data
transforms, file writes, database operations.
No judgment calls. Just inputs and outputs.
```

---

## Why It Works

**Separation of concerns**: The AI touches only routing and judgment. Deterministic work lives in deterministic code.

**Self-annealing**: When a script breaks, the agent reads the error, fixes the script, tests it, then updates the directive. Each failure makes the system more robust.

**Living directives**: Directives accumulate real-world knowledge over time - API quirks, rate limits, edge cases. They're not static documentation; they're an improving instruction set.

**Testable execution**: Scripts can be run and tested independently. You can verify behavior without involving the AI at all.

---

## File Layout

```
├── CLAUDE.md               # Agent instructions (loaded automatically by Claude Code)
├── directives/             # SOPs - what to do and how
│   ├── example_data_pipeline.md
│   └── example_investigation.md
└── execution/              # Deterministic Python scripts (not in this repo - see implementations)
```

---

## The CLAUDE.md

`CLAUDE.md` is the agent instruction file. It loads automatically in Claude Code when you open the repo. It defines:

- The 3-layer architecture and the agent's role within it
- Operating principles (check for tools first, self-anneal, update directives)
- File organization conventions
- Style rules (no em dashes, no trailing summaries, etc.)

Copy it into any project and the agent inherits the same behavior.

---

## Example Directives

This repo includes two example directives that demonstrate the pattern:

- [directives/example_data_pipeline.md](directives/example_data_pipeline.md) - fetch, transform, write to destination
- [directives/example_investigation.md](directives/example_investigation.md) - given an ID, pull all relevant data and produce a report

---

## Production Implementations

- **[enterprise-ai-automation-platform](https://github.com/Justin2259/enterprise-ai-automation-platform)** - Full platform showcase: architecture overview, example directives, example execution scripts
- **[genesys-cloud-rag](https://github.com/Justin2259/genesys-cloud-rag)** - Semantic search over a Genesys Cloud contact center org
- **[ai-tools-platform](https://github.com/Justin2259/ai-tools-platform)** - Catalog-driven tool platform for 200+ non-technical users
- **[ivr-testing-suite](https://github.com/Justin2259/ivr-testing-suite)** - Automated IVR testing via parallel AI call agents
- **[call-forensics-engine](https://github.com/Justin2259/call-forensics-engine)** - Full forensic reconstruction of any phone call

---

## Getting Started

1. Copy `CLAUDE.md` into your project root
2. Create a `directives/` folder with at least one SOP
3. Create an `execution/` folder with the scripts your directive references
4. Open the project in Claude Code - it reads `CLAUDE.md` automatically

The agent will follow the architecture without further configuration.
