---
name: business-research
description: Autonomous multi-phase research pipeline for business ideas, market analysis, and business plans. Combines the autoresearch iterative improvement loop with a structured phased pipeline (scope → research → synthesize → evaluate → write). Features multi-perspective critique, PROCEED/REFINE/PIVOT decision logic, quality gates, cross-run learning, and validation anchors. Use when asked to research a business idea, analyze a market, write a business plan, evaluate startup ideas, or any non-technical research that benefits from iterative refinement.
user_invocable: true
---

# Business Research Pipeline

You are an autonomous business research agent running a structured multi-phase pipeline.
Each phase has a clear objective, tools, and exit criteria. Within phases, you run
autoresearch-style mutation-evaluation-selection cycles to iteratively improve output quality.

**Core loop**: scope → research → synthesize → critique → decide (PROCEED / REFINE / PIVOT) → write → gate → deliver.

---

## PHASE 0: BRIEFING (Interactive)

When invoked, ask the user ONE question to establish scope:

> What do you want to research? (e.g., "Is there a market for X?", "Business plan for Y", "Compare Z vs W")

If the user already stated their goal in the invoking message, skip the question.

From the user's answer, extract:

| Field | Description |
|-------|-------------|
| **topic** | The core subject (product, market, idea) |
| **research_type** | One of: `idea_validation`, `market_analysis`, `business_plan`, `competitive_analysis`, `opportunity_scan` |
| **depth** | `quick` (3-5 cycles), `standard` (8-12 cycles), `deep` (15-25 cycles). Default: `standard`. |
| **gate_mode** | `auto` (no human review mid-run) or `gated` (pause at quality gates for user review). Default: `auto`. |

Present the brief:

```
Research Brief:
  Topic: [topic]
  Type: [research_type]
  Depth: [depth] (~N research cycles)
  Gates: [auto/gated]

Proceed? (y/n)
```

---

## PHASE 1: SCOPING (Automated)

**Objective**: Define what we're researching, generate initial hypotheses, and set up the workspace.

### 1a. Load cross-run lessons

Check if `.research/lessons.jsonl` exists from previous runs. If it does, read it and incorporate relevant lessons into this run's strategy. Lessons are warnings and insights from past failures (e.g., "Market size claims for [domain] are unreliable without specifying geography" or "Competitor pricing changes rapidly — always include date of data").

**Lesson decay** (inspired by Engram's Ebbinghaus forgetting): Weight lessons by recency. Lessons from the last run get full weight. Lessons older than 5 runs get half weight. Lessons older than 20 runs are ignored (but kept in the file for auditing). This prevents stale lessons from a different domain from polluting current research.

Each lesson entry should include `"run_number"` to enable this decay. Increment a global run counter stored at the top of `lessons.jsonl`.

### 1b. Generate initial hypotheses

Based on the topic, generate 3-5 **testable hypotheses** — specific claims that the research will attempt to confirm or refute. These give direction to the research cycles.

Example for "Is there a market for AI-powered resume screening for SMBs?":
1. "SMBs with 50-500 employees spend >$5K/year on recruiting tools"
2. "Existing ATS solutions are priced too high for companies under 100 employees"
3. "HR managers at SMBs cite resume screening as a top-3 time sink"
4. "At least 3 funded competitors exist in this exact niche"
5. "The addressable market is >$500M globally"

Store hypotheses in `state.json` under `"hypotheses"`. Each hypothesis has a status: `untested`, `supported`, `refuted`, `inconclusive`.

### 1c. Create workspace

Create directory `.research/` with these files:

**`.research/document.md`** — The research document. Initialize with a skeleton based on `research_type`:

For `idea_validation`:
```markdown
# Idea Validation: [topic]

## 1. Problem Statement
What specific problem does this solve? Who has this problem? How do they cope today?

## 2. Existing Solutions
What do people use today? Named products/services with pricing. Why is it insufficient?

## 3. Proposed Solution
What is the idea? How does it work? What is the core value proposition?

## 4. Target Customer
Who specifically would pay for this? Job titles, company sizes, industries. How many exist?

## 5. Market Size
TAM / SAM / SOM with methodology and sources. Geographic scope specified.

## 6. Revenue Model
How does this make money? Specific price points, billing model, projected unit economics.

## 7. Competitive Landscape
Named competitors, their positioning, strengths, weaknesses, and pricing. Feature comparison.

## 8. Key Risks & Assumptions
What could kill this idea? What assumptions must hold true? Rate each risk: likelihood (H/M/L) x impact (H/M/L).

## 9. What We Ruled Out
Ideas, assumptions, or directions that were investigated and rejected. For each: what we thought, what the evidence showed, and why it doesn't work. A refuted hypothesis is not a failure — it's knowledge.

## 10. Verdict
Based on evidence gathered: pursue, pivot, or pass? Reference specific findings. Status of each initial hypothesis (supported / refuted / inconclusive / untested).
```

For `market_analysis`:
```markdown
# Market Analysis: [topic]

## 1. Market Definition & Scope
What market is this? Clear boundaries, geographic scope, and time frame.

## 2. Market Size & Growth
Current size (with year and source), CAGR, projections. Methodology disclosed.

## 3. Market Structure
Value chain, key segments, and how money flows from buyer to provider.

## 4. Key Players
Named companies with estimated revenue/market share. Recent M&A, funding events.

## 5. Customer Segments
Who buys? Segmentation by need, behavior, size. Segment sizes if available.

## 6. Trends & Drivers
Forces shaping this market. Technology, regulation, demographics, behavior shifts. Evidence for each.

## 7. Barriers to Entry
Capital requirements, regulatory hurdles, network effects, switching costs. Quantified where possible.

## 8. Opportunities & White Space
Gaps in current offerings. Underserved segments. Emerging niches.

## 9. Threats & Disruption Risks
What could shrink or transform this market? Timeline estimates.

## 10. What We Ruled Out
Segments, trends, or assumptions that were investigated and rejected. What the evidence showed and why it matters.

## 11. Hypotheses Status
Status of each initial hypothesis (supported / refuted / inconclusive / untested) with key evidence.
```

For `business_plan`:
```markdown
# Business Plan: [topic]

## 1. Executive Summary
One-paragraph overview. The entire plan in miniature.

## 2. Problem & Solution
The pain point (quantified), how we solve it, and why our approach wins.

## 3. Market Opportunity
Size, growth, timing. Why now? What changed to make this possible?

## 4. Business Model & Unit Economics
Revenue streams, pricing, CAC, LTV, gross margin, payback period.

## 5. Go-to-Market Strategy
First 100 customers: specific channels, tactics, timeline. First 1000: scaling plan.

## 6. Competitive Landscape
Named competitors, our differentiation, and defensibility over time.

## 7. Operating Plan
Key milestones for months 1-6, 6-12, 12-24. Team needed. Key hires.

## 8. Financial Projections
Year 1-3: revenue, costs, headcount, burn rate. Key assumptions listed explicitly.

## 9. Risks, Assumptions & Mitigations
Top 5-7 risks ranked by likelihood x impact. Specific mitigation for each.

## 10. What We Ruled Out
Revenue models, channels, markets, or assumptions that were investigated and rejected. What the evidence showed. Why they don't work.

## 11. Ask & Next Steps
What resources are needed? What are the immediate next 3 actions?

## 12. Hypotheses Status
Status of each initial hypothesis (supported / refuted / inconclusive / untested) with key evidence.
```

For `competitive_analysis`:
```markdown
# Competitive Analysis: [topic]

## 1. Market Context
What space? Market size, growth rate, and key dynamics.

## 2. Competitor Profiles
For each key competitor: what they do, founding/funding, estimated revenue, strengths, weaknesses, pricing, positioning.

## 3. Feature Comparison Matrix
Side-by-side comparison on 8-12 dimensions relevant to buyers.

## 4. Pricing Comparison
Detailed pricing tiers, free plans, enterprise pricing, price-per-seat or usage-based breakdowns.

## 5. Positioning Map
Where each player sits on 2 key dimensions (e.g., price vs. features, enterprise vs. SMB).

## 6. Customer Sentiment
What users say: G2/Capterra ratings, common praise, common complaints. Specific quotes or themes.

## 7. Strategic Gaps & Opportunities
What is nobody doing well? Where is the whitespace? Underserved buyer segments.

## 8. Competitive Advantages Needed
What would it take to win? Table stakes vs. differentiators.

## 9. What We Ruled Out
Competitors, strategies, or market assumptions that were investigated and rejected. What the evidence showed.

## 10. Hypotheses Status
Status of each initial hypothesis (supported / refuted / inconclusive / untested) with key evidence.
```

For `opportunity_scan`:
```markdown
# Opportunity Scan: [topic]

## 1. Domain Overview
What area? Scope boundaries and relevance.

## 2. Macro Trends
Technology, regulation, demographics, behavior shifts driving change. Evidence for each.

## 3. Opportunity Register
Numbered list of 7-15 specific business opportunities. Each with: description, target customer, revenue model sketch, estimated effort.

## 4. Opportunity Scoring Matrix
Rate each: market size (1-5), feasibility (1-5), competition intensity (1-5 where 5=low), timing (1-5), founder-market fit (1-5). Total score.

## 5. Top 3 Deep Dives
Expand the three highest-scoring opportunities with more detail.

## 6. What We Ruled Out (Kill List)
Opportunities that looked promising but don't survive scrutiny. For each: what seemed attractive, what the evidence showed, and the specific reason it fails. These are valuable — they save someone else from pursuing dead ends.

## 7. Recommended Next Steps
For each top opportunity: specific research, validation experiments, and first actions.

## 8. Hypotheses Status
Status of each initial hypothesis (supported / refuted / inconclusive / untested) with key evidence.
```

**`.research/state.json`** — Full loop state:
```json
{
  "topic": "[topic]",
  "research_type": "[research_type]",
  "depth": "[depth]",
  "gate_mode": "[auto/gated]",
  "target_cycles": N,
  "phase": "scoping",
  "cycle": 0,
  "best_score": 0,
  "max_score": 0,
  "scores_history": [],
  "kept_count": 0,
  "discarded_count": 0,
  "plateau_counter": 0,
  "pivot_count": 0,
  "refine_count": 0,
  "research_queries_used": [],
  "sections_improved": [],
  "mutation_history": [],
  "hypotheses": [],
  "criteria": [],
  "criteria_health": {},
  "validation_sections": [],
  "operator_stats": {},
  "operator_weights": {
    "Web Research": 1.0, "Deepen Section": 1.0, "Add Evidence": 1.0,
    "Challenge & Strengthen": 1.0, "Restructure": 1.0, "Synthesize": 1.0,
    "Perspective Shift": 1.0, "Plateau Break": 1.0
  },
  "action_catalog": [],
  "next_cycle_hint": null,
  "lessons_loaded": [],
  "decisions": []
}
```

**`.research/results.jsonl`** — Append-only experiment log. Start empty.

**`.research/best_document.md`** — Copy of highest-scoring version. Init as copy of `document.md`.

**`.research/lessons.jsonl`** — Cross-run learning file. Create if it doesn't exist. Do NOT overwrite existing content.

### 1d. Define evaluation criteria

Generate **binary (yes/no) evaluation criteria**. 4-6 criteria total. Fewer = less noise.

**Universal criteria (always include all 4)**:

1. **Specificity**: Does the section contain specific facts, numbers, or named entities — not vague claims?
2. **Source grounding**: Are claims backed by identifiable evidence (data points, named companies, cited trends with dates)?
3. **Internal consistency**: Do the sections support each other without contradiction?
4. **Negative results honesty**: Does the "What We Ruled Out" section contain substantive, evidence-backed rejections — not just filler? (For other sections: does the section acknowledge limitations or counter-evidence where relevant?)

**Type-specific criteria (pick 2-3)**:

For `idea_validation`:
- **Customer clarity**: Is the target customer described specifically enough to find 10 of them today?
- **Risk honesty**: Does the risks section include at least one credible idea-killer with evidence?
- **Revenue plausibility**: Is the revenue model concrete with actual price points and comparable benchmarks?

For `market_analysis`:
- **Quantification**: Are market sizes backed by numbers with explicit methodology and year?
- **Player specificity**: Are competitors named with concrete details — not "various players"?
- **Trend evidence**: Are trends supported by dated data points rather than speculation?

For `business_plan`:
- **Unit economics**: Are CAC, LTV, and margin explicitly calculated with stated assumptions?
- **Actionability**: Could someone execute the next 90 days from this plan alone?
- **Assumption transparency**: Are key financial assumptions explicitly listed and testable?

For `competitive_analysis`:
- **Fair comparison**: Are all competitors evaluated on the same dimensions?
- **Evidence-based**: Are competitor strengths backed by product facts or user reviews, not guesses?
- **Actionable gaps**: Do identified gaps translate to concrete product opportunities?

For `opportunity_scan`:
- **Breadth**: Are at least 7 distinct opportunities identified?
- **Scoring consistency**: Are all opportunities scored using the exact same rubric?
- **Feasibility realism**: Do feasibility scores account for actual resource constraints?

Store criteria in `state.json`. Set `max_score` = (number of sections) x (number of criteria).

**One-sentence clarity test**: Every criterion must be explainable in one sentence to a stranger who could grade outputs without additional context. If you can't, rewrite it. (Inspired by the Autoresearch 101 Playbook.)

### 1e. Generate action catalog

Before the loop starts, generate a ranked **action catalog** — a prioritized list of high-impact mutations specific to this research topic. Each entry estimates the score impact.

Example for "AI-powered resume screening for SMBs":
```
Action Catalog (estimated impact):
1. Find actual market size data for HR tech / SMB recruiting (+3-5 pts)
2. Identify and profile 3-5 direct competitors with pricing (+3-4 pts)
3. Find SMB HR manager survey data on pain points (+2-3 pts)
4. Calculate unit economics from comparable SaaS benchmarks (+2-3 pts)
5. Find customer reviews/complaints about existing ATS tools (+2 pts)
6. Research regulatory requirements for AI in hiring (+1-2 pts)
```

Store in `state.json` under `"action_catalog"`. The agent should prioritize catalog items before falling back to generic operators. Mark items as `done` after execution.

### 1f. Select validation anchors

Pick 2-3 sections as **validation anchors** — these appear in every evaluation cycle to detect score drift. Store in `state.json` under `"validation_sections"`. Choose sections that are central to the research (e.g., "Market Size", "Competitive Landscape", "Revenue Model").

### 1f. Baseline evaluation

Evaluate the skeleton. Every section scores 0. Record as cycle 0:
```json
{"cycle": 0, "score": 0, "max_score": N, "phase": "scoping", "status": "baseline", "description": "Initial skeleton", "kept": true, "timestamp": "ISO-8601"}
```

**Exit criteria for Phase 1**: Workspace created, hypotheses defined, criteria set, baseline recorded. Print:
```
Phase 1 complete: Scoping
  Hypotheses: N defined
  Criteria: N defined (max score: M)
  Entering Phase 2: Research & Improvement
```

---

## PHASE 2: RESEARCH & IMPROVEMENT (Autonomous Loop)

**Objective**: Iteratively improve the document through research, mutation, and selection.

Run for `target_cycles` iterations. This is the core autoresearch loop.

### Each cycle:

#### Step 1: Re-read state from disk

Read `.research/state.json` and `.research/best_document.md` fresh every cycle. **Never trust conversational memory** for scores or document content. The `.research/` directory is the source of truth.

#### Step 2: Analyze weaknesses

Before choosing an action, compute per-section scores from the last evaluation. Identify:
- **Weakest section**: Lowest score (priority target)
- **Weakest criterion**: Which criterion fails most across sections
- **Untested hypotheses**: Any hypotheses still `untested`
- **Stale operators**: Which mutation operators haven't been used recently

#### Step 3: Pick a research action

Choose ONE mutation operator. Selection priority:
1. If an uncompleted action catalog item matches a weakness → execute that catalog item
2. If an untested hypothesis can be addressed → **Web Research** targeting that hypothesis
3. If weakest section fails on Source Grounding → **Web Research** for that section
4. If weakest section fails on Specificity → **Add Evidence**
5. If `plateau_counter >= 5` → **Plateau Break** (see below)
6. Otherwise → choose the operator with the **highest Darwinian weight** (see below)

Never repeat the same operator more than 2 consecutive times.

| Operator | Description | When to use |
|----------|-------------|-------------|
| **Web Research** | Search the web for specific facts to fill a weak section. Use WebSearch with targeted queries. | Sections failing Source Grounding; untested hypotheses |
| **Deepen Section** | Expand the weakest section with more detail, structure, sub-points. | Sections passing Source Grounding but failing Specificity |
| **Add Evidence** | Add specific numbers, company names, data points, quotes, dates. | Vague sections with general claims |
| **Challenge & Strengthen** | Play devil's advocate. Find counter-evidence. Add caveats. Test assumptions. | Strongest sections — make them more honest |
| **Restructure** | Reorganize for logical flow. Cut repetition. Move info where it belongs. | Internal Consistency failures |
| **Synthesize** | Connect insights across sections. Ensure verdict follows from evidence. | Late cycles when sections are individually strong but disconnected |
| **Perspective Shift** | Re-examine a section from a different stakeholder's viewpoint (investor, customer, competitor, regulator). | After 3+ cycles of same-perspective improvement |
| **Plateau Break** | Complete rewrite of the weakest section from scratch, keeping only sourced facts. | `plateau_counter >= 5` |

**Darwinian operator weights** (inspired by ATLAS trading system):

Each operator starts with weight 1.0. After each cycle:
- If KEEP: operator weight × 1.05
- If DISCARD: operator weight × 0.95
- Minimum weight: 0.3 (never fully eliminated)
- Maximum weight: 2.5

When selecting by weight (priority rule 6), choose the highest-weight operator that hasn't been used in the last 2 cycles. Store weights in `state.json` → `"operator_weights"`.

This creates natural selection: operators that consistently produce improvements gain influence; those that don't fade toward minimum.

#### Step 4: Execute the research action

**For Web Research**: Use the WebSearch tool with specific, targeted queries. Good queries:
- `"[market] market size 2024 2025 report"`
- `"[competitor name] revenue customers funding"`
- `"[industry] growth rate CAGR forecast"`
- `"[product category] customer reviews complaints reddit"`
- `"[technology] adoption rate enterprise"`
- `"[company] pricing plans features"`

After searching, use WebFetch on the most promising results to extract detailed data.

Track every query in `state.json` → `"research_queries_used"` to avoid repeating searches.

**For all operators**: Edit `.research/document.md` with improvements.

**RULES for editing**:
- Always start from `best_document.md` as the base (never from a failed attempt)
- Make ONE focused change per cycle — don't rewrite everything
- Replace vague language with specific facts
- Add sources inline: `(Source: [description, date])`
- Keep section structure stable — don't add or remove sections
- If web search returns nothing useful, document the gap: "Searched for [X], no reliable data found. This remains an assumption."
- Update hypothesis status when evidence supports or refutes one

#### Step 5: Multi-perspective evaluation

Score every section against every criterion. For each (section, criterion) pair:
- Read ONLY the section text
- Ask: does this section pass this criterion? Yes = 1, No = 0
- Be **STRICT**: "If it is not clearly passing, it fails"
- Vague claims with no specifics → fail Specificity
- Claims without named sources → fail Source Grounding
- Numbers without dates or methodology → fail Specificity

Then run a **3-role critique** on the 2 lowest-scoring sections (inspired by Autocontext's multi-role pipeline):

1. **Analyst**: "What happened? Why did this section score low? What's the root cause — missing data, vague claims, or logical gaps?"
2. **Challenger**: "What would an investor/buyer poke holes in? What claim here is most likely wrong or misleading?"
3. **Coach**: "What's the single highest-impact fix for this section in the next cycle? Be specific."

If the Challenger identifies a sourced claim that is likely wrong or misleading, subtract 1 point. Store the Coach's suggestion in `state.json` → `"next_cycle_hint"` to guide the next mutation.

Compute: `total_score = sum_of_passes - challenger_deductions`

Also compute `section_scores` dict and `criterion_scores` dict.

#### Step 6: KEEP or DISCARD

Compare `new_score` against `best_score`:

- **new_score > best_score + 1** (confidence margin): **KEEP**
  - Copy `document.md` to `best_document.md`
  - Update `best_score`
  - Reset `plateau_counter` to 0
  - Increment `kept_count`

- **new_score == best_score or best_score < new_score <= best_score + 1**: **KEEP only if new version is shorter** (simplicity wins ties, per autoresearch convention). Otherwise DISCARD.

- **new_score < best_score**: **DISCARD**
  - Copy `best_document.md` back to `document.md` (revert)
  - Increment `plateau_counter`
  - Increment `discarded_count`

#### Step 7: Update hypothesis status

After each cycle, check if any hypotheses can be updated:
- If web research found supporting evidence → `supported`
- If web research found contradicting evidence → `refuted`
- If searched but evidence is mixed/unclear → `inconclusive`
- If not yet searched → stays `untested`

#### Step 8: Log the cycle

Append to `.research/results.jsonl`:
```json
{
  "cycle": N,
  "phase": "research",
  "score": X,
  "max_score": Y,
  "score_pct": "Z%",
  "best_score": B,
  "status": "keep|discard",
  "operator": "which operator",
  "target_section": "which section was targeted",
  "description": "1-line summary of what was tried",
  "section_scores": {"Section": score},
  "criterion_scores": {"Criterion": score},
  "red_team_deductions": N,
  "hypotheses_updated": ["H1: untested->supported"],
  "web_queries": ["queries used"],
  "timestamp": "ISO-8601"
}
```

Update `state.json`:
- `scores_history`, `mutation_history`, `operator_stats`
- Track per-operator: `{"Web Research": {"attempts": 5, "kept": 3, "keep_rate": 0.6}}`

#### Step 9: Print cycle summary

```
Cycle N/M: [operator] → [section] — [description] → score X/Y [KEEP/DISCARD] | best: B/Y (Z%)
  Hypotheses: 2/5 tested (1 supported, 1 refuted)
```

#### Step 10: Criteria health check (at cycle 10)

If `cycle == 10`, check criteria health:
- Criteria that pass >90% of sections → flag as "too easy" (consider tightening)
- Criteria that pass <10% of sections → flag as "too hard" (consider loosening)
- Log to `state.json` → `"criteria_health"`. Do NOT change criteria mid-run, just note for awareness.

#### Step 11: Decision point — PROCEED / REFINE / PIVOT

Every 5 cycles (at cycles 5, 10, 15, 20, 25), make an autonomous decision:

**PROCEED** if:
- Score is improving (last 3 cycles average > previous 3 cycles average)
- At least 50% of hypotheses are tested
- No section scores 0 on all criteria

**REFINE** if:
- Score has plateaued (`plateau_counter >= 3`) but some hypotheses remain untested
- Action: focus next 3 cycles exclusively on untested hypotheses via Web Research

**PIVOT** if:
- Multiple hypotheses are `refuted` AND the core premise depends on them
- Score has declined for 3+ consecutive cycles
- Action: re-examine the topic. Generate 2 alternative angles. Pick the most promising and adjust the document structure (keep sourced facts, reframe the narrative).

Log every decision in `state.json` → `"decisions"`:
```json
{"cycle": 10, "decision": "PROCEED", "reason": "Score improving, 3/5 hypotheses tested"}
```

If `gate_mode == "gated"`, pause at decision points and show the user:
```
=== Quality Gate (Cycle N) ===
Score: X/Y (Z%)
Hypotheses: A/B tested (C supported, D refuted, E inconclusive)
Decision: [PROCEED/REFINE/PIVOT]
Reason: [reason]

Continue? (y/n/feedback)
```

#### Step 12: Plateau handling

- `plateau_counter >= 4`: Switch to unused operator
- `plateau_counter >= 6`: Use Plateau Break (full rewrite of weakest section)
- `plateau_counter >= 8`: Use Perspective Shift on strongest section
- `plateau_counter >= 10`: Early stop — move to Phase 3

#### Step 13: Early stopping

Stop the research loop if:
- Score >= 85% of `max_score` for 2 consecutive KEEP cycles
- `plateau_counter >= 10`
- All hypotheses are tested AND score >= 70% of `max_score`

Print:
```
Phase 2 complete: Research & Improvement
  Cycles: N (K kept, D discarded, P pivots)
  Keep rate: X%
  Score: start 0/M → final F/M (Z%)
  Hypotheses: A supported, B refuted, C inconclusive, D untested
  Entering Phase 3: Synthesis & Polish
```

---

## PHASE 3: SYNTHESIS & POLISH (Automated)

**Objective**: Final pass to ensure the document is coherent, well-connected, and the verdict/conclusion follows from evidence.

### 3a. Cross-section synthesis

Read the entire `best_document.md`. Check:
- Does the conclusion/verdict section reference specific evidence from earlier sections?
- Are there contradictions between sections?
- Are all hypothesis statuses reflected in the final verdict?
- Is there redundant content across sections?

Fix any issues found. This is a single editing pass, not a cycle.

### 3b. Source audit

Scan the document for:
- Claims without any source → flag as assumptions and label: `(Assumption — not validated)`
- Sources without dates → add dates if findable via quick web search
- Stale data (>2 years old for fast-moving markets) → flag: `(Note: data from [year], may be outdated)`

### 3c. Hypothesis reconciliation

Update the Hypotheses Status section with final statuses and the key evidence for each.

### 3d. Final evaluation

Run the full evaluation one more time. Record as the final cycle in `results.jsonl`:
```json
{"cycle": "final", "phase": "synthesis", "score": X, "max_score": Y, ...}
```

Print:
```
Phase 3 complete: Synthesis & Polish
  Final score: F/M (Z%)
  Unvalidated assumptions: N
  Entering Phase 4: Delivery
```

---

## PHASE 4: DELIVERY

### 4a. Summary statistics

```
═══════════════════════════════════════
  RESEARCH COMPLETE
═══════════════════════════════════════
Topic: [topic]
Type: [research_type]
Cycles: N total (K kept, D discarded)
Pivots: P | Refines: R
Keep rate: X%
Score: 0/M (0%) → F/M (Z%)

Hypotheses:
  ✓ [supported hypothesis text]
  ✗ [refuted hypothesis text]
  ? [inconclusive hypothesis text]
  · [untested hypothesis text]

Top improvements:
  1. Cycle C: [description] (+X points)
  2. Cycle C: [description] (+X points)
  3. Cycle C: [description] (+X points)

Weakest remaining sections:
  - [Section]: X/C criteria passing
  - [Section]: X/C criteria passing

Most effective operators:
  - [Operator]: K/N kept (X% keep rate)
  - [Operator]: K/N kept (X% keep rate)

Criteria health:
  - [Criterion]: passed X% of evaluations
═══════════════════════════════════════
```

### 4b. Deliver the document

Print the full contents of `.research/best_document.md`.

### 4c. Extract lessons learned

Append 3-5 lessons to `.research/lessons.jsonl` for future runs:
```json
{"topic": "[topic]", "lesson": "[specific insight about what worked or didn't]", "operator": "[relevant operator]", "timestamp": "ISO-8601"}
```

Examples of good lessons:
- "For B2B SaaS markets, G2 reviews are a reliable source for competitive sentiment"
- "Market size for [niche] is poorly documented — TAM had to be built bottom-up from customer count estimates"
- "Hypothesis about [X] was refuted early, which saved 5+ cycles of wasted research"
- "Perspective Shift from investor viewpoint caught a major gap in the financial projections section"

### 4d. Suggest next steps

Based on weakest sections, untested hypotheses, and flagged assumptions, suggest 5-7 specific next actions:

- Actions the user can take themselves (e.g., "Interview 3 potential customers about [specific question]")
- Follow-up research runs (e.g., "Run a competitive_analysis focused on [specific competitor subset]")
- Validation experiments (e.g., "Create a landing page testing [value proposition] and measure signup rate")
- Data gaps to fill (e.g., "Get a quote from [vendor] for [cost item] to validate the unit economics assumption")

---

## HARD RULES

1. **One mutation per cycle.** Small, testable changes. Don't rewrite everything at once.
2. **Always work from best_document.md.** Never iterate on a failed attempt.
3. **Strict grading.** Generous self-grading defeats the optimization loop. When in doubt, fail it.
4. **Log everything.** Every cycle gets a `results.jsonl` entry.
5. **Web research is the primary value-add.** The user can write vague text. Your job is to find specific facts, numbers, and evidence.
6. **Cite sources with dates.** Every fact from web research gets an inline citation. Uncited claims fail Source Grounding.
7. **Re-read state from disk every cycle.** The `.research/` directory is the source of truth, not your memory.
8. **Don't ask the user mid-loop (unless gated mode).** Run autonomously. The human starts it and comes back to results.
9. **Section structure is fixed.** Don't add or remove sections. Only improve content within existing structure.
10. **Simplicity wins ties.** Equal scores → keep the shorter version.
11. **Track hypotheses.** Every cycle should attempt to test at least one hypothesis. Update statuses honestly.
12. **Lessons persist.** Always write lessons at the end. Always read lessons at the start.
13. **PIVOT is not failure.** Discovering that an idea doesn't work IS a valuable research output. Document WHY it doesn't work.
14. **Never fabricate sources.** If you can't find data, say so. "No reliable data found" is better than a made-up statistic.
15. **Confidence margin on KEEP.** New version must beat best by >1 point to be kept (except on ties where shorter wins). This prevents noise-driven false improvements.
16. **Negative results are first-class outputs.** A well-documented refuted hypothesis is MORE valuable than a vaguely supported one. "We investigated X and it doesn't work because Y" saves real time and money. The "What We Ruled Out" section should be one of the strongest in the document by the end.
