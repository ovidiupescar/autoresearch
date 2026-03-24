# Business Research Pipeline — Claude Code Skill

An autonomous multi-phase research pipeline for business ideas, market analysis, and business plans. Built as a Claude Code skill using the [autoresearch](https://github.com/karpathy/autoresearch) iterative improvement pattern, enhanced with ideas from [AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw) and [Universal Autoresearch Skill](https://github.com/balukosuri/Andrej-Karpathy-s-Autoresearch-As-a-Universal-Skill).

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

```
/business-research
```

Or with a topic:

```
/business-research Is there a market for AI-powered inventory management for small restaurants?
```

## Research types

| Type | Best for |
|------|----------|
| `idea_validation` | "Should I build this?" — problem, solution, customer, market, risks, verdict |
| `market_analysis` | "How big is this market?" — size, growth, players, trends, barriers, opportunities |
| `business_plan` | "How would this work?" — model, unit economics, GTM, financials, risks |
| `competitive_analysis` | "Who else does this?" — profiles, features, pricing, gaps, positioning |
| `opportunity_scan` | "Where are the opportunities?" — trends, scored opportunity list, deep dives |

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

- **Testable hypotheses**: Generates 3-5 specific claims upfront, tracks them as supported/refuted/inconclusive
- **8 mutation operators**: Web Research, Deepen, Add Evidence, Challenge, Restructure, Synthesize, Perspective Shift, Plateau Break
- **Red Team pass**: Adversarial re-evaluation of weakest sections every cycle
- **PROCEED/REFINE/PIVOT**: Autonomous strategic decisions every 5 cycles
- **Quality gates** (optional): Pause mid-run for human review in `gated` mode
- **Cross-run learning**: Lessons from past runs loaded at start, new lessons saved at end
- **Validation anchors**: Fixed sections in every eval to prevent score drift
- **Criteria health**: Flags too-easy or too-hard criteria at cycle 10
- **Confidence margin**: Must beat best by >1 point to KEEP (reduces noise)
- **Operator tracking**: Per-operator keep rate drives smarter operator selection

## Installation

The skill is a single file: `skill/SKILL.md`. Copy it to your Claude Code skills directory, or use this repo directly as your research workspace.

No dependencies beyond Claude Code and its built-in WebSearch/WebFetch tools.

## Inspired by

- [karpathy/autoresearch](https://github.com/karpathy/autoresearch) — the original ML autoresearch pattern
- [aiming-lab/AutoResearchClaw](https://github.com/aiming-lab/AutoResearchClaw) — 23-stage autonomous academic research pipeline
- [balukosuri/Universal Autoresearch Skill](https://github.com/balukosuri/Andrej-Karpathy-s-Autoresearch-As-a-Universal-Skill) — adapting autoresearch to universal prompt optimization

## License

MIT
