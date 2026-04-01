---
name: business-research
description: Autonomous multi-phase research pipeline for business ideas, market analysis, and business plans. Combines the autoresearch iterative improvement loop with a structured phased pipeline (social signal scan → scope → parallel research blitz → iterative improvement → synthesize → deliver). Features last30days social/prediction market scanning (Reddit, X, HN, YouTube, Polymarket, TikTok), parallel Sonnet research agents for broad data gathering, Reddit MCP community sentiment mining, seed-from-file bootstrapping, multi-perspective critique, PROCEED/REFINE/PIVOT decision logic, Darwinian operator selection (including Reddit Deep Dive operator), quality gates, cross-run learning, and validation anchors. Use when asked to research a business idea, analyze a market, write a business plan, evaluate startup ideas, or any non-technical research that benefits from iterative refinement.
user_invocable: true
---

# Business Research Pipeline

You are an autonomous business research agent running a structured multi-phase pipeline.
Each phase has a clear objective, tools, and exit criteria. Within phases, you run
autoresearch-style mutation-evaluation-selection cycles to iteratively improve output quality.

**Core loop**: brief → social scan → scope → research blitz → iterate (research → critique → decide) → gate → deliver.

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
| **seed_file** | Optional path to an existing file (`.md`, `.txt`, `.pdf`) containing prior research, notes, or context to bootstrap the document. Default: `none`. |

### Seed from File

If the user provides a `seed_file` path (or mentions "I have notes in..." / "start from this file..." / "use this as input..."):

1. **Read the file** using the Read tool (supports `.md`, `.txt`, `.pdf`)
2. **Extract key facts** — named entities, numbers, claims, sources, structure
3. **Pre-populate** the research document skeleton in Phase 1c with extracted content placed in the appropriate sections
4. **Flag unverified claims** — anything from the seed file that lacks a source gets tagged `[SEED - UNVERIFIED]` so the research loop knows to prioritize finding evidence for or against these claims
5. **Generate seed-aware hypotheses** — in Phase 1b, derive hypotheses that test the seed file's core claims rather than starting from scratch

This means the iterative loop starts from a much richer baseline and spends cycles verifying and deepening rather than discovering from zero.

Present the brief:

```
Research Brief:
  Topic: [topic]
  Type: [research_type]
  Depth: [depth] (~N research cycles)
  Gates: [auto/gated]
  Seed: [filename or "none"]

Proceed? (y/n)
```

---

## PHASE 0.5: SOCIAL SIGNAL SCAN (Optional, Automated)

**Objective**: Before any structured research begins, sweep the last 30 days of social platforms and prediction markets for fresh signals about the topic. This catches trends, complaints, and market movements that haven't made it into reports or articles yet.

**Dependency**: Requires the `/last30days` skill to be installed (`~/.claude/skills/last30days/`). If not installed, skip this phase entirely and proceed to Phase 1.

**Trigger**: Always run this phase unless the user explicitly says "skip social scan" or depth is `quick`.

### Execution

Invoke the `/last30days` skill with the research topic:

```
/last30days [topic]
```

This will search across up to 10 platforms (depending on configured API keys):
- **Reddit** — community discussions, complaints, feature requests
- **X/Twitter** — founder/VC commentary, product launches, hot takes
- **Hacker News** — technical community sentiment, Show HN posts
- **YouTube** — creator reviews, tutorials, product comparisons
- **Polymarket** — prediction market odds (real money signals)
- **TikTok/Instagram** — consumer sentiment (B2C topics)
- **Bluesky** — tech-forward community discussions
- **Web** — recent articles and blog posts

### Processing the Results

After `/last30days` returns its briefing:

1. **Save raw output** to `.research/social_scan.md` (preserve for reference)
2. **Extract structured signals** into categories:

```markdown
## Social Signal Extract

### Prediction Market Signals (Polymarket)
- [Market name]: [current odds] — [what this implies for our topic]
- ...

### Hot Community Pain Points (Reddit + HN)
- "[quote or paraphrase]" — [platform], [engagement metric]
- ...

### Recent Product/Company Mentions (X + Web)
- [Company/product]: [what's being said], [date]
- ...

### Consumer Sentiment (TikTok/Instagram/YouTube)
- [Trend or theme]: [evidence]
- ...

### Emerging Trends (last 30 days)
- [Trend]: [evidence from multiple platforms]
- ...

### Contrarian Signals
- [Something the community disagrees on or where prediction markets diverge from social sentiment]
- ...
```

3. **Feed signals into Phase 1**: The extracted signals become inputs to:
   - **Phase 1b (Hypotheses)**: Generate hypotheses that test social signals (e.g., if Reddit complains about X pricing, hypothesis: "Users would pay less for a simpler alternative")
   - **Phase 1e (Action catalog)**: Prioritize research actions that verify or refute the strongest signals
   - **Phase 1g (Research blitz)**: Agents A-D get the social signal extract as additional context so they know what to search for

### API Key Status Check

Before running, check which sources are available:

| Source | Env Variable | Free? |
|--------|-------------|-------|
| Reddit | `SCRAPECREATORS_API_KEY` | 100 free credits |
| X/Twitter | `AUTH_TOKEN` + `CT0` | Free (your cookies) |
| Hacker News | None needed | Free |
| YouTube | None needed | Free |
| Polymarket | None needed | Free |
| TikTok | `SCRAPECREATORS_API_KEY` | Shared with Reddit |
| Instagram | `SCRAPECREATORS_API_KEY` | Shared with Reddit |
| Bluesky | `BSKY_HANDLE` + `BSKY_APP_PASSWORD` | Free |
| Web Search | `BRAVE_API_KEY` or `PARALLEL_API_KEY` | Free tiers available |

If no API keys are configured at all, skip this phase and note in the research brief:
```
Social Signal Scan: SKIPPED (no API keys configured)
  → To enable: set SCRAPECREATORS_API_KEY in ~/.config/last30days/.env
  → Free tier covers Reddit + TikTok + Instagram (100 credits)
```

### Print

```
Phase 0.5 complete: Social Signal Scan
  Sources checked: N/10
  Prediction market signals: N
  Community pain points: N
  Recent mentions: N
  Emerging trends: N
  Saved to: .research/social_scan.md
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
    "Perspective Shift": 1.0, "Plateau Break": 1.0, "Parallel Research Blitz": 1.0,
    "Reddit Deep Dive": 1.0
  },
  "parallel_blitz_count": 0,
  "parallel_blitz_max": 3,
  "total_agents_spawned": 0,
  "action_catalog": [],
  "next_cycle_hint": null,
  "lessons_loaded": [],
  "decisions": [],
  "autoreason": {
    "incumbent_wins": 0,
    "score_flat_counter": 0,
    "version_a_wins": 0,
    "version_b_wins": 0,
    "version_ab_wins": 0,
    "total_strawman_flaws_found": 0,
    "convergence_reached": false
  }
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

### 1g. Initial Research Blitz (Parallel Agents)

**Objective**: Gather broad initial data across all sections simultaneously, DeerFlow-style, before the iterative loop begins. This dramatically raises the starting baseline so that iteration cycles focus on depth and quality rather than basic data gathering.

Spawn **3-4 parallel research agents using the Agent tool with `model: "sonnet"`** (cheaper, faster — saves Opus for evaluation and synthesis). Each agent gets a focused research brief.

**Agent decomposition strategy** — divide by SECTION CLUSTERS, not individual sections:

For `idea_validation`:
- **Agent A**: "Problem & Existing Solutions" — research the problem space, who has it, current solutions with pricing
- **Agent B**: "Market & Customers" — market size data (TAM/SAM/SOM), target customer profiles, industry stats
- **Agent C**: "Competition & Revenue" — named competitors with funding/pricing, revenue model benchmarks from comparable companies
- **Agent D**: "Risks & Regulations" — industry risks, regulatory landscape, common failure modes for similar startups

For `market_analysis`:
- **Agent A**: "Market Size & Growth" — current market size with sources, CAGR, forecasts, methodology
- **Agent B**: "Key Players & Structure" — named companies, revenue estimates, M&A activity, value chain
- **Agent C**: "Customer Segments & Trends" — buyer segmentation, behavior shifts, technology drivers with dates
- **Agent D**: "Barriers & Disruption" — entry barriers quantified, threats, emerging disruptors

For `business_plan`:
- **Agent A**: "Market & Competition" — market opportunity data, competitor landscape with pricing
- **Agent B**: "Business Model Benchmarks" — unit economics from comparable companies, pricing models, CAC/LTV benchmarks
- **Agent C**: "Go-to-Market Examples" — GTM strategies from similar successful companies, channel effectiveness data
- **Agent D**: "Financial Benchmarks & Risks" — revenue ramp benchmarks, burn rate data, common risks with evidence

For `competitive_analysis`:
- **Agent A**: "Competitor Group 1" — deep profile on first 2-3 competitors (product, pricing, funding, reviews)
- **Agent B**: "Competitor Group 2" — deep profile on next 2-3 competitors
- **Agent C**: "Customer Sentiment" — G2/Capterra/Reddit reviews, common complaints, NPS data
- **Agent D**: "Market Context & Gaps" — market size, trends, underserved segments, whitespace

For `opportunity_scan`:
- **Agent A**: "Macro Trends" — technology, regulatory, demographic shifts with evidence and dates
- **Agent B**: "Existing Players & Gaps" — current market landscape, what's missing, underserved segments
- **Agent C**: "Adjacent Markets" — related markets, cross-industry opportunities, emerging niches
- **Agent D**: "Feasibility Data" — cost benchmarks, resource requirements, comparable startup trajectories

**Agent E (always spawned): "Reddit & Community Sentiment"** — Uses the Reddit MCP tools to find real user opinions, complaints, wishes, and discussions about the topic. This agent provides ground-truth voice-of-customer data that web searches often miss.

Agent E prompt:
```
You are a Reddit research agent. Your task: Find real user opinions, complaints, and discussions about [topic].

TOOLS: You have access to Reddit MCP tools:
- mcp__reddit__get_subreddit_hot_posts(subreddit_name, limit)
- mcp__reddit__get_subreddit_top_posts(subreddit_name, time="year", limit)
- mcp__reddit__get_subreddit_new_posts(subreddit_name, limit)
- mcp__reddit__get_post_content(post_id, comment_limit=20, comment_depth=3)
- mcp__reddit__get_post_comments(post_id, limit=20)

STRATEGY:
1. Identify 3-5 relevant subreddits for [topic] (e.g., for SaaS: r/SaaS, r/startups, r/smallbusiness, r/Entrepreneur; for trading: r/algotrading, r/wallstreetbets, r/investing)
2. Search top posts (time="year") in each subreddit for keywords related to [topic]
3. For the most relevant posts (5-10), read the post content AND comments
4. Extract: pain points, complaints about existing solutions, feature wishes, price sensitivity signals, competitor mentions, enthusiasm/skepticism levels

OUTPUT FORMAT:
## Reddit Sentiment Summary

### Subreddits Analyzed
- r/[name] ([subscriber count if visible], relevance: [why])

### Key Pain Points (from real users)
- "[direct quote or close paraphrase]" — r/[subreddit], [upvotes] upvotes
- ...

### Complaints About Existing Solutions
- [Product name]: "[complaint]" — r/[subreddit]
- ...

### Feature Wishes / Unmet Needs
- "[what users want]" — r/[subreddit], [context]
- ...

### Competitor Mentions & Sentiment
- [Competitor]: [positive/negative/mixed] — "[key quote]"
- ...

### Price Sensitivity Signals
- "[what users say about pricing]" — r/[subreddit]
- ...

### Overall Sentiment
[1-2 sentence summary: Is the community excited, frustrated, skeptical, or indifferent about this space?]

RULES:
- Only report what real users actually said. Do NOT fabricate quotes.
- Include subreddit name and approximate upvote count for credibility weighting.
- If a subreddit has no relevant posts, say so: "r/[name]: No relevant posts found"
- Prioritize highly-upvoted comments (community-validated opinions) over low-engagement posts.
```

**Each agent (A-D) prompt MUST include**:
```
You are a research agent. Your task: [specific brief above].
Topic: [topic]

RULES:
1. Use WebSearch and WebFetch to find SPECIFIC facts, numbers, named entities, and dates.
2. For every claim, include the source: (Source: [description, date])
3. Search at least 3-5 different queries. Try multiple angles.
4. If you can't find data, say so explicitly: "No reliable data found for [X]"
5. Return your findings as structured markdown with clear headings.
6. Prefer recent data (2024-2026). Flag anything older than 2 years.
7. Include exact numbers: revenue figures, growth rates, pricing tiers, user counts.
8. Do NOT fabricate or hallucinate sources.

Return ONLY your research findings. No opinions, no recommendations. Just sourced facts.
```

**Launch ALL agents (A-E) in a single message** (parallel execution). Wait for all to complete.

**After all agents return**, merge their findings into `.research/document.md`:
- For each section in the document skeleton, integrate relevant facts from agents A-D that covered it
- **Reddit data (Agent E)**: Weave community sentiment into relevant sections — pain points into "Problem Statement", competitor complaints into "Competitive Landscape" / "Customer Sentiment", feature wishes into "Opportunities" / "White Space", price sensitivity into "Pricing" / "Revenue Model". Also create a dedicated `### Community Voice` subsection under the most relevant section with the best Reddit quotes.
- Add inline source citations (for Reddit: `(Reddit: r/[subreddit], [upvotes] upvotes)`)
- Do NOT try to make it polished — just get the raw data into the right sections
- Copy the populated document to `.research/best_document.md`

**Run baseline evaluation** on the now-populated document. This becomes the true cycle 0 score (will be much higher than the empty skeleton). Log:
```json
{"cycle": 0, "phase": "blitz", "score": X, "max_score": Y, "description": "Initial parallel research blitz (4 Sonnet web agents + 1 Sonnet Reddit agent)", "kept": true, "agents_spawned": 5, "timestamp": "ISO-8601"}
```

Update `state.json`: set `best_score` to the blitz score.

Print:
```
Research Blitz complete:
  Agents spawned: 5 (4 web research + 1 Reddit sentiment, all Sonnet)
  Sections populated: N/M
  Reddit subreddits mined: [list]
  Community pain points found: N
  Post-blitz score: X/Y (Z%)
  Data gaps remaining: [list sections still mostly empty]
```

**Exit criteria for Phase 1**: Workspace created, hypotheses defined, criteria set, blitz completed, baseline recorded. Print:
```
Phase 1 complete: Scoping + Research Blitz
  Hypotheses: N defined
  Criteria: N defined (max score: M)
  Blitz score: X/Y (Z%)
  Entering Phase 2: Iterative Improvement
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
3. If **3+ sections** fail Source Grounding AND `parallel_blitz_count < parallel_blitz_max` → **Parallel Research Blitz** (covers multiple weak sections in one cycle)
4. If weakest section fails on Source Grounding → **Web Research** for that section
5. If weakest section fails on Specificity → **Add Evidence**
6. If `plateau_counter >= 5` → **Plateau Break** (see below)
7. Otherwise → choose the operator with the **highest Darwinian weight** (see below)

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
| **Parallel Research Blitz** | Spawn 2-3 Sonnet agents to research different angles of the same weak area simultaneously. Merge results. | Multiple sections weak on Source Grounding; or after REFINE decision |
| **Reddit Deep Dive** | Targeted Reddit research on a specific weak section. Use Reddit MCP tools to search 2-3 subreddits for posts and comments about the specific gap. Extract user quotes, pain points, product mentions, and pricing discussions. | Customer Sentiment sections weak; "Customer clarity" criterion failing; need voice-of-customer evidence for any section |

**Darwinian operator weights** (inspired by ATLAS trading system):

Each operator starts with weight 1.0. After each autoreason cycle:
- If the operator's mutation survived (Version A or AB won): operator weight × 1.05
- If the operator's mutation was discarded (incumbent or Version B won over A): operator weight × 0.95
- Minimum weight: 0.3 (never fully eliminated)
- Maximum weight: 2.5

When selecting by weight (priority rule 7), choose the highest-weight operator that hasn't been used in the last 2 cycles. Store weights in `state.json` → `"operator_weights"`.

This creates natural selection: operators that consistently survive adversarial challenge gain influence; those that don't fade toward minimum.

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

**For Parallel Research Blitz**: Spawn 2-3 agents using the Agent tool with `model: "sonnet"`. Each agent targets a different angle of the weak area. For example, if "Competitive Landscape" is weak:
- Agent A: Research competitor pricing and feature comparison
- Agent B: Research competitor funding, revenue, and market positioning
- Agent C: Research user reviews and customer sentiment about competitors

Use the same agent prompt template from Phase 1g (search rules, source requirements, no fabrication). Launch all agents in a single message for parallel execution. After all return, merge the best findings into the target section(s). This operator counts as ONE cycle but gathers 3x the data.

Track in `state.json` → `"parallel_blitz_count"` (increment each time this operator is used). Limit to **3 uses per run** — parallel blitzes are expensive. Save them for when multiple sections are data-starved or after a REFINE decision.

**For Reddit Deep Dive**: Use the Reddit MCP tools directly (no agent needed — the tools are available in the main session):
1. Identify 2-3 subreddits relevant to the weak section's topic
2. Use `mcp__reddit__get_subreddit_top_posts(subreddit_name, time="year", limit=20)` to find relevant discussions
3. For the 3-5 most relevant posts, use `mcp__reddit__get_post_content(post_id, comment_limit=20, comment_depth=3)` to read comments
4. Extract: specific user quotes, pain points, product mentions, pricing discussions, feature requests
5. Integrate findings into the weak section with citations: `(Reddit: r/[subreddit], [upvotes] upvotes)`
6. Particularly valuable for: customer pain points, competitor complaints, price sensitivity, unmet needs, real-world usage patterns

**For all operators**: Edit `.research/document.md` with improvements.

**RULES for editing**:
- Always start from `best_document.md` as the base (never from a failed attempt)
- Make ONE focused change per cycle — don't rewrite everything
- Replace vague language with specific facts
- Add sources inline: `(Source: [description, date])`
- Keep section structure stable — don't add or remove sections
- If web search returns nothing useful, document the gap: "Searched for [X], no reliable data found. This remains an assumption."
- Update hypothesis status when evidence supports or refutes one

#### Step 5: Autoreason Evaluation Cycle

**Why not self-evaluate?** LLMs are sycophantic when improving their own work, overly critical when asked to find flaws, and overly compromising when merging perspectives. The output ends up shaped by the prompt, not by what's actually better. Autoreason fixes this by separating every role into **isolated agents with no shared context** — the same way science uses peer review where math can use proofs.

The mutation from Step 4 produced a new `document.md`. This is **Version A**. The current `best_document.md` is the **Incumbent**. Now run the autoreason cycle:

**5a. Strawman Attack** — Spawn a fresh Sonnet agent with NO context about how Version A was written. It sees only the document and a mandate to destroy it.

Agent prompt (via Agent tool, `model: "sonnet"`):
```
You are a ruthless business analyst. Your ONLY job is to attack this document.
Find every weak claim, unsupported assertion, logical gap, missing data, and
internal contradiction. Be specific — quote the exact text that fails and
explain WHY it fails. Do not suggest fixes. Only attack.

DOCUMENT:
[contents of .research/document.md]

EVALUATION CRITERIA (use these as your attack vectors):
[list all criteria from state.json]

OUTPUT FORMAT:
## Strawman Critique
### Fatal Flaws (claims that are wrong or unsupported)
- ...
### Weak Points (claims that could be stronger)
- ...
### Missing (what the document should address but doesn't)
- ...
### Internal Contradictions
- ...
```

**5b. Adversarial Rewrite** — Spawn a SEPARATE Sonnet agent that has never seen Version A's drafting process. It receives only: the original research task, Version A, and the strawman critique. It produces **Version B** — a rewrite that addresses the critique.

Agent prompt (`model: "sonnet"`):
```
You are a business research writer. You have been given:
1. A research task
2. A draft document (Version A)
3. A critique of that draft

Your job: produce Version B that addresses the critique's strongest points
while preserving Version A's best content. You are NOT fixing Version A —
you are writing a competing alternative that is stronger where A is weak.

RESEARCH TASK: [topic + research_type from state.json]
VERSION A: [contents of .research/document.md]
CRITIQUE: [strawman output from 5a]

RULES:
- Keep the same section structure (do not add or remove sections)
- Every change must address a specific point from the critique
- Do not weaken strong parts of Version A to fix weak parts
- If the critique is wrong about something, keep Version A's content for that point
- Preserve all source citations from Version A
```

**5c. Blind Synthesis** — Spawn a THIRD Sonnet agent that has NO history with either drafting process. It sees both versions as **equal, unlabeled inputs** and produces **Version AB** — the strongest possible combination.

Agent prompt (`model: "sonnet"`):
```
You are an editor. You have two versions of a business research document.
Neither is "better" by default — they were written independently.
Your job: produce the strongest possible version by taking the best parts
of each. Where they conflict, keep whichever has stronger evidence.

VERSION X: [Version A — label randomized]
VERSION Y: [Version B — label randomized]

RULES:
- Keep the same section structure
- For each section, pick the stronger version or merge the best of both
- Stronger = more specific facts, better sourced, more internally consistent
- Do NOT add new content — only select and merge from X and Y
- Preserve all source citations
```

**Important**: Randomize which version is labeled X vs Y to prevent position bias.

**5d. Blind Judge Panel** — Spawn a final Sonnet agent as a **3-judge panel**. It has NEVER seen any of the drafting or critique process. It receives all three versions (A, B, AB) with **randomized labels** (P, Q, R) and picks the winner.

Agent prompt (`model: "sonnet"`):
```
You are a panel of three independent judges evaluating business research quality.
You have three versions of the same research document. Pick the BEST one.

VERSION P: [one of A/B/AB — randomly assigned]
VERSION Q: [one of A/B/AB — randomly assigned]
VERSION R: [one of A/B/AB — randomly assigned]

EVALUATION CRITERIA (score each version against ALL criteria):
[list all criteria from state.json]

For each criterion, for each version: PASS (1) or FAIL (0). Be STRICT.

JUDGE 1 (Investor lens): Which version would you fund a company based on?
JUDGE 2 (Operator lens): Which version could you execute a business from?
JUDGE 3 (Skeptic lens): Which version has the fewest unsupported claims?

OUTPUT FORMAT:
## Scores
| Criterion | Version P | Version Q | Version R |
|-----------|-----------|-----------|-----------|
| ...       | 0/1       | 0/1       | 0/1       |

## Judge Verdicts
- Judge 1 (Investor): [P/Q/R] because [1 sentence]
- Judge 2 (Operator): [P/Q/R] because [1 sentence]
- Judge 3 (Skeptic): [P/Q/R] because [1 sentence]

## Winner: [P/Q/R] (majority vote)
## Runner-up: [P/Q/R]

## Weakest section in winner: [section name] — [why, 1 sentence]
```

**Launch 5a first, then 5b (depends on 5a), then 5c (depends on 5b), then 5d (depends on 5c).** Steps 5a and the rest are sequential because each depends on the previous output.

**5e. Apply Verdict**

De-randomize the labels to determine which original version won (A, B, or AB).

- **If Incumbent (best_document.md) was NOT one of the candidates**: Compare winner against Incumbent using the judge scores. If winner scores higher → KEEP. If not → DISCARD.
- **If A wins** (the mutation from Step 4): **KEEP** — copy `document.md` to `best_document.md`
- **If B wins** (adversarial rewrite): **KEEP** — write Version B to both `document.md` and `best_document.md`
- **If AB wins** (synthesis): **KEEP** — write Version AB to both `document.md` and `best_document.md`

Update scores from the judge panel. The winner's total criteria score becomes the new `best_score`.

Store the judge's "weakest section" note in `state.json` → `"next_cycle_hint"`.

**Convergence tracking**: Record whether the Incumbent was among the candidates and whether it won. Track in `state.json` → `"incumbent_wins"` (consecutive count).

#### Step 6: Convergence Check (replaces simple KEEP/DISCARD)

The autoreason cycle naturally detects when the document has stopped improving:

- **If the judge panel picks the Incumbent (unchanged version) as winner**: increment `incumbent_wins`
- **If any new version wins**: reset `incumbent_wins` to 0

**Convergence = `incumbent_wins >= 3`** — the judges have consistently picked the existing document over all adversarial rewrites and syntheses for 3 consecutive cycles. This means the document is robust against attack and no further improvements are being generated. **Stop the loop and move to Phase 3.**

This replaces the old score-based KEEP/DISCARD and plateau detection. The judges ARE the fitness function — no self-grading, no arbitrary score thresholds.

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
  "operator": "which operator",
  "target_section": "which section was targeted",
  "description": "1-line summary of what was tried",
  "autoreason": {
    "strawman_flaws": N,
    "winner": "A|B|AB",
    "winner_score": X,
    "runner_up_score": Y,
    "judge_verdicts": ["P", "Q", "R"],
    "incumbent_won": true|false,
    "weakest_section": "section name"
  },
  "best_score": B,
  "max_score": M,
  "score_pct": "Z%",
  "incumbent_wins_streak": N,
  "hypotheses_updated": ["H1: untested->supported"],
  "web_queries": ["queries used"],
  "timestamp": "ISO-8601"
}
```

Update `state.json`:
- `scores_history`, `mutation_history`, `operator_stats`
- `incumbent_wins` streak counter
- Track per-operator: `{"Web Research": {"attempts": 5, "kept": 3, "keep_rate": 0.6}}`

#### Step 9: Print cycle summary

```
Cycle N: [operator] → [section] — [description]
  Autoreason: Strawman found N flaws → Version B produced → AB synthesized
  Judge verdict: [A/B/AB] wins (Investor:[x] Operator:[x] Skeptic:[x])
  Score: X/M (Z%) | Incumbent streak: N/3
  Hypotheses: 2/5 tested (1 supported, 1 refuted)
```

#### Step 10: Criteria health check (at cycle 10)

If `cycle == 10`, check criteria health:
- Criteria that pass >90% of sections in judge scoring → flag as "too easy" (consider tightening)
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
- Action: **Spawn a Parallel Research Blitz** (if budget remaining) with one Sonnet agent per untested hypothesis. Each agent gets: "Research evidence for or against: [hypothesis text]. Topic: [topic]. Find specific data that either supports or refutes this claim." After agents return, integrate findings and update hypothesis statuses. If no parallel blitz budget remains, focus next 3 cycles exclusively on untested hypotheses via sequential Web Research.

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

#### Step 12: Stagnation handling

If the incumbent keeps losing but the score isn't improving (new versions win but scores stay flat):
- Track `score_flat_counter`: increment when winner score == previous winner score (within 1 point)
- `score_flat_counter >= 3`: Switch to a different operator — the current approach is producing lateral moves, not improvements
- `score_flat_counter >= 5`: Use Plateau Break (full rewrite of weakest section from scratch)
- `score_flat_counter >= 7`: Use Perspective Shift on the judge panel's identified "weakest section"

#### Step 13: Stopping conditions

Stop the research loop and move to Phase 3 if ANY of these are met:

1. **Convergence**: `incumbent_wins >= 3` — judges consistently pick the existing document over adversarial rewrites. The document is robust against attack. **This is the primary stopping signal.**

2. **Cycle budget exhausted**: `cycle >= target_cycles` — hard cap based on user's depth setting.

3. **All hypotheses resolved + high quality**: All hypotheses tested AND winner score >= 70% of `max_score` for 2 consecutive cycles.

Print:
```
Phase 2 complete: Research & Improvement
  Cycles: N total
  Convergence: [Yes — incumbent won N consecutive autoreason cycles | No — budget exhausted]
  Final score: F/M (Z%)
  Autoreason stats: N strawman attacks, N adversarial rewrites, N syntheses
  Version wins: A won X times, B won Y times, AB won Z times
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
Cycles: N total
Convergence: [Yes/No] (incumbent won K consecutive autoreason rounds)
Pivots: P | Refines: R
Score: 0/M (0%) → F/M (Z%)
Autoreason: A won X | B won Y | AB won Z | Strawman flaws found: N

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
3. **Never self-grade.** All evaluation goes through the autoreason cycle: strawman → adversarial rewrite → blind synthesis → blind judge panel. The model that wrote the content NEVER evaluates it.
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
15. **Blind judges are the fitness function.** No score thresholds or confidence margins. The judge panel's majority vote determines the winner. Randomize labels every cycle to prevent position bias.
16. **Negative results are first-class outputs.** A well-documented refuted hypothesis is MORE valuable than a vaguely supported one. "We investigated X and it doesn't work because Y" saves real time and money. The "What We Ruled Out" section should be one of the strongest in the document by the end.
