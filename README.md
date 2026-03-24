# Autoresearch Skills for Business & Trading

Two autonomous research pipelines built as Claude Code skills — one for **business research** and one for **trading/investment research**. Both use the [autoresearch](https://github.com/karpathy/autoresearch) iterative improvement pattern, enhanced with ideas from [AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw), [Universal Autoresearch Skill](https://github.com/balukosuri/Andrej-Karpathy-s-Autoresearch-As-a-Universal-Skill), [ATLAS trading system](https://github.com/chrisworsey55/atlas-gic), [GOAL.md](https://github.com/jmilinovich/goal-md), [Autocontext](https://github.com/greyhaven-ai/autocontext), [Engram](https://github.com/tonitangpotato/autoresearch-engram), and [FARS](https://analemma.ai/blog/introducing-fars/).

## What it does

You give it a business topic. It runs an autonomous research pipeline — scoping hypotheses, searching the web for real data, filling in a structured document, scoring itself strictly, keeping improvements and discarding regressions, making PROCEED/REFINE/PIVOT decisions — then delivers a research document backed by specific facts, numbers, and named sources.

## How it works

**4-phase pipeline:**

```
Phase 1: SCOPING         Phase 2: RESEARCH LOOP       Phase 3: SYNTHESIS    Phase 4: DELIVERY
─────────────────         ──────────────────────       ─────────────────     ─────────────────
• Load past lessons       • Pick weakest area          • Cross-section       • Summary stats
• Generate hypotheses     • Choose mutation operator      coherence check    • Final document
• Create workspace        • Execute (web search,       • Source audit        • Extract lessons
• Set eval criteria         deepen, challenge...)      • Hypothesis          • Next steps
• Select validation       • Strict evaluation            reconciliation
  anchors                 • Red Team pass
• Baseline score          • KEEP or DISCARD
                          • Every 5 cycles:
                            PROCEED / REFINE / PIVOT
                          • Plateau detection
```

### What we took from each source

| Source | What we borrowed |
|--------|-----------------|
| **Karpathy's autoresearch** | Core loop: mutate → evaluate → keep/discard. Strict binary grading. One change per cycle. Simplicity wins ties. |
| **AutoResearchClaw** | Phased pipeline instead of flat loop. PROCEED/REFINE/PIVOT decisions. Cross-run learning. Quality gates. Multi-perspective critique. |
| **Universal Autoresearch Skill** | Confidence-based acceptance (beat best by >1). Validation anchors. Criteria health checks. Per-operator effectiveness tracking. |

## Usage

### Business research
```
/business-research
/business-research Is there a market for AI-powered inventory management for small restaurants?
```

### Trading research
```
/trading-research
/trading-research Is NVDA overvalued at current levels?
/trading-research Best semiconductor plays for 2026
```

## Research types

### Business (`/business-research`)

| Type | Best for |
|------|----------|
| `idea_validation` | "Should I build this?" — problem, solution, customer, market, risks, verdict |
| `market_analysis` | "How big is this market?" — size, growth, players, trends, barriers, opportunities |
| `business_plan` | "How would this work?" — model, unit economics, GTM, financials, risks |
| `competitive_analysis` | "Who else does this?" — profiles, features, pricing, gaps, positioning |
| `opportunity_scan` | "Where are the opportunities?" — trends, scored opportunity list, deep dives |

### Trading (`/trading-research`)

| Type | Best for |
|------|----------|
| `single_stock` | "Should I buy X?" — bull/bear case, valuation, catalysts, risk framework |
| `sector_analysis` | "Which sector to overweight?" — drivers, players, regime sensitivity, top picks |
| `macro_thesis` | "Will rates come down?" — thesis, evidence, contrary evidence, trade expression |
| `strategy_eval` | "Does this strategy work?" — edge, backtests, regime analysis, capacity |
| `comparative` | "ASML vs LRCX?" — side-by-side financials, valuation, moat, asymmetry |

## Depth settings

| Depth | Cycles | Use when |
|-------|--------|----------|
| `quick` | 3-5 | Fast sanity check, initial signal |
| `standard` | 8-12 | Solid research with real data (default) |
| `deep` | 15-25 | Thorough analysis for important decisions |

## Output

All artifacts in `.research/`:

| File | Purpose |
|------|---------|
| `best_document.md` | Final research document (the deliverable) |
| `document.md` | Working copy (may differ from best if last cycle was discarded) |
| `state.json` | Full state: hypotheses, scores, operator stats, decisions |
| `results.jsonl` | Every cycle logged: score, operator, description, keep/discard |
| `lessons.jsonl` | Cross-run learning — persists across research sessions |

## Key features

**Core engine (both skills)**:
- **Testable hypotheses**: 3-5 specific claims generated upfront, tracked as supported/refuted/inconclusive
- **8 mutation operators**: Web Research, Deepen, Add Evidence, Challenge, Restructure, Synthesize, Perspective Shift, Plateau Break
- **3-role critique**: Analyst → Challenger → Coach pipeline on weakest sections every cycle (from Autocontext)
- **Darwinian operator weights**: Operators that produce KEEPs gain weight (×1.05), DISCARDs lose weight (×0.95) — natural selection for mutations (from ATLAS)
- **Action catalog**: Prioritized high-impact research actions with estimated point values, executed before generic operators (from GOAL.md)
- **PROCEED/REFINE/PIVOT**: Autonomous strategic decisions every 5 cycles
- **Cross-run learning with decay**: Lessons persist across runs, weighted by recency — stale lessons fade (from Engram)
- **Negative results as first-class output**: "What We Ruled Out" section in every template (from FARS)
- **Confidence margin**: Must beat best by >1 point to KEEP (from Universal Autoresearch Skill)
- **Validation anchors**: Fixed sections in every eval to prevent score drift
- **Criteria health checks**: Flags too-easy or too-hard criteria at cycle 10
- **Quality gates** (optional): Pause mid-run for human review in `gated` mode

**Trading-specific**:
- **Both-sides rigor**: Bull AND bear cases required with equal evidence depth
- **Conviction calibration**: High/Medium/Low conviction tied to evidence strength
- **Regime sensitivity**: How does the thesis perform across different macro environments?
- **Risk framework**: Explicit stop-loss triggers, position sizing, maximum loss tolerance
- **Date everything**: All financial data must include dates — undated metrics are flagged
- **Investment disclaimer**: Always included, non-negotiable

## Installation

Two skill files: `skill/SKILL.md` (business) and `skill/trading/SKILL.md` (trading). Copy to your Claude Code skills directory, or use this repo as your research workspace.

No dependencies beyond Claude Code and its built-in WebSearch/WebFetch tools.

## Inspired by

- [karpathy/autoresearch](https://github.com/karpathy/autoresearch) — the original ML autoresearch pattern
- [aiming-lab/AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw) — 23-stage autonomous academic research pipeline with PROCEED/REFINE/PIVOT
- [balukosuri/Universal Autoresearch Skill](https://github.com/balukosuri/Andrej-Karpathy-s-Autoresearch-As-a-Universal-Skill) — confidence margin, validation anchors, criteria health
- [chrisworsey55/atlas-gic](https://github.com/chrisworsey55/atlas-gic) — Darwinian weight system for trading agents optimized against Sharpe ratio
- [jmilinovich/goal-md](https://github.com/jmilinovich/goal-md) — constructed fitness functions and ranked action catalogs
- [greyhaven-ai/autocontext](https://github.com/greyhaven-ai/autocontext) — multi-role critique pipeline (Analyst→Coach→Curator)
- [tonitangpotato/autoresearch-engram](https://github.com/tonitangpotato/autoresearch-engram) — persistent memory with Ebbinghaus forgetting decay
- [FARS by Analemma](https://analemma.ai/blog/introducing-fars/) — negative results as first-class research outputs
- [awesome-autoresearch](https://github.com/alvinunreal/awesome-autoresearch) — curated list that led us to many of these sources

## License

MIT
