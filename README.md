# Business Research Autoresearch

An autonomous AI research agent that iteratively improves business research documents using the [autoresearch](https://github.com/karpathy/autoresearch) pattern. Instead of optimizing ML training code, it optimizes **business research** through cycles of web research, structured evaluation, and self-critique.

You describe a business idea or market question. The agent runs 8-25 research cycles autonomously -- searching the web, filling in specifics, evaluating against fixed criteria, keeping improvements and discarding regressions -- and delivers a research document backed by real data.

## How It Works

The autoresearch pattern, originally by Andrej Karpathy for ML experiments, follows a simple loop:

```
propose change -> execute -> evaluate -> keep or discard -> repeat
```

This skill adapts that loop from code optimization to research optimization:

| Karpathy's Autoresearch | Business Research Skill |
|-------------------------|------------------------|
| `program.md` (agent instructions) | `skill/SKILL.md` (the skill file) |
| `train.py` (code being optimized) | `.research/document.md` (research doc being improved) |
| `prepare.py` (fixed evaluation) | Binary eval criteria (specificity, source grounding, etc.) |
| `val_bpb` (single metric) | Score out of N (sections x criteria) |
| `results.tsv` (experiment log) | `.research/results.jsonl` (cycle log) |
| Git branch (keep/revert) | `best_document.md` (keep/revert) |

### The Research Loop

Each cycle, the agent:

1. **Picks a mutation operator** -- Web Research, Deepen Section, Add Evidence, Challenge & Strengthen, Restructure, or Synthesize
2. **Executes the action** -- searches the web for real data, rewrites a weak section, adds specifics
3. **Evaluates strictly** -- scores every section against every criterion (specificity, source grounding, internal consistency, plus type-specific criteria)
4. **Keeps or discards** -- if the score improved, keep; if not, revert to the best known version
5. **Logs everything** -- full cycle details appended to `results.jsonl`

The key insight from autoresearch: **strict evaluation + keep/discard gating** prevents the document from degrading. Every version is at least as good as the previous best.

## Research Types

| Type | Use When | Sections |
|------|----------|----------|
| `idea_validation` | "Is there a market for X?" | Problem, Solutions, Customer, Market Size, Revenue, Risks, Verdict |
| `market_analysis` | "How big is the X market?" | Definition, Size, Players, Segments, Trends, Barriers, Opportunities |
| `business_plan` | "Business plan for X" | Executive Summary, Problem, Market, Model, GTM, Competition, Financials |
| `competitive_analysis` | "Compare X vs Y vs Z" | Context, Profiles, Feature Matrix, Positioning, Sentiment, Gaps |
| `opportunity_scan` | "What businesses could work in X?" | Domain, Trends, Opportunity List, Scoring, Deep Dives, Next Steps |

## Installation

### Claude Code

```bash
# Copy the skill to your Claude skills directory
mkdir -p ~/.claude/skills/business-research
cp skill/SKILL.md ~/.claude/skills/business-research/SKILL.md
```

### Cursor

```bash
# Copy the skill to your Cursor skills directory
mkdir -p ~/.cursor/skills/business-research
cp skill/SKILL.md ~/.cursor/skills/business-research/SKILL.md
```

### Verify installation

In Claude Code, the skill should appear when you run `/business-research` or when you ask to "research a business idea."

## Usage

### Quick start

```
> /business-research

# Or just describe what you want:
> Research whether there's a market for AI-powered home energy management
> Write a business plan for a B2B SaaS that helps restaurants reduce food waste
> Compare Stripe vs Square vs Adyen for a marketplace startup
```

### Depth levels

- **quick** (3-5 cycles) -- fast sanity check, light web research
- **standard** (8-12 cycles) -- default, solid research with real data
- **deep** (15-25 cycles) -- thorough analysis, extensive web research

### What you get

After the run completes, you'll find in `.research/`:

| File | Contents |
|------|----------|
| `best_document.md` | The final research document (your deliverable) |
| `document.md` | Working copy (same as best after completion) |
| `state.json` | Run metadata, scores, cycle counts |
| `results.jsonl` | Full log of every cycle with scores and descriptions |

The agent also prints a summary with:
- Score progression (starting -> final)
- Which cycles contributed the most improvement
- Which mutation operators were most effective
- Weakest remaining sections
- Suggested next steps for further research

## How Evaluation Works

Every section is scored against binary criteria. A section either passes or fails each criterion -- no partial credit.

**Universal criteria** (always applied):
- **Specificity**: Contains specific facts, numbers, or named entities?
- **Source grounding**: Claims backed by identifiable evidence?
- **Internal consistency**: No contradictions with other sections?

**Type-specific criteria** (selected based on research type):
- Customer clarity, risk honesty, revenue plausibility (idea validation)
- Quantification, player specificity, trend evidence (market analysis)
- Unit economics, actionability, assumption transparency (business plan)
- Fair comparison, evidence-based strengths, actionable gaps (competitive)
- Breadth, scoring consistency, feasibility realism (opportunity scan)

The strict grading is intentional. From the autoresearch pattern: generous self-grading defeats the optimization loop. Vague claims always fail Specificity. Uncited assertions always fail Source Grounding.

## Design Principles

Borrowed from Karpathy's autoresearch and adapted:

1. **One mutation per cycle.** Small, testable changes. Don't rewrite everything at once.
2. **Always work from the best known version.** Never iterate on a failed attempt.
3. **Strict evaluation.** If in doubt, fail it. The loop recovers from false negatives but not false positives.
4. **Web research is the primary value-add.** The user can write vague text themselves. The agent's job is finding specific facts and evidence.
5. **Log everything.** The `.research/results.jsonl` file is the experiment history.
6. **Run autonomously.** No mid-loop questions. The human starts it and comes back to results.
7. **Simplicity wins ties.** If two versions score equally, keep the shorter one.

## Inspired By

- [karpathy/autoresearch](https://github.com/karpathy/autoresearch) -- the original ML autoresearch pattern
- [balukosuri/Andrej-Karpathy-s-Autoresearch-As-a-Universal-Skill](https://github.com/balukosuri/Andrej-Karpathy-s-Autoresearch-As-a-Universal-Skill) -- adapting autoresearch to a universal prompt optimization skill

## License

MIT
