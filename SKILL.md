---
name: target-scoring
description: Score PE/VC acquisition targets using the "Moat in the Machine" framework. Runs 4 parallel research agents, scores verticals (Napkin Q1/Q2/Q3), company subscores (Wind/Sail/Fit), software moats (SC_Depth, Data Moat, Platform), and generates a 9-slide investor memo PPTX. Use when evaluating software or services companies for acquisition or investment.
---

# Target Scoring — Moat in the Machine

## Overview

Score PE/VC acquisition targets using Dutchess Management's "Moat in the Machine" framework (Framework v2). The pipeline:

1. **4 parallel research agents** gather financials, competitive landscape, GenAI risk, and tech moat data
2. **Scoring rubric** classifies targets into quadrants: AI Goldmine, Contrarian Bet, Sand Castle, Rising Tide
3. **JSON data file** captures all scores, evidence, and analysis
4. **PPTX generator** produces a polished 9-slide investor memo (Title, Snapshot, Napkin, Company Scores, Software Moat, Data Moat, Competitive, GenAI, Sources)

## When to Use This Skill

Activate when users request:
- Score or evaluate a company for acquisition
- Run target analysis or due diligence
- Create an acquisition/investment memo
- Assess a company's AI defensibility or moat
- Generate a target scoring PPTX

## Workflow

### Step 1: Gather Target Info

Ask the user for:
- **Company name** (required)
- **Vertical/industry** (required — e.g., "debt collection software", "contact center")
- **Known financials** (optional — revenue, growth, margins, funding)
- **Specific concerns** (optional — e.g., "worried about AI agent displacement")

### Step 2: Launch Research (Hybrid: Parallel CLI + Claude Code Agents)

Use a two-layer research approach for maximum coverage and depth:

#### Layer 1: Parallel CLI Deep Research (fire-and-forget)

Launch 4 parallel research runs using `parallel research run` with `--processor ultra-fast --no-wait --json`. These run in the background while Claude Code agents work simultaneously.

```bash
# Launch all 4 in rapid succession
parallel research run "[COMPANY] financial and operational profile: revenue, revenue growth rate, EBITDA or operating margin, profitability status, employee count, geographic footprint, customer metrics, retention rate. Cross-reference PitchBook, CBInsights, Crunchbase, Tracxn, SEC filings. Include all funding rounds with dates, amounts, lead investors." --processor ultra-fast --no-wait --json -o research/<company>/<company>_financials

parallel research run "[COMPANY] competitive landscape and market positioning for M&A analysis: (1) direct competitors with revenue, funding, differentiation, (2) traditional incumbents and market share, (3) TAM with authoritative market sizing from IBISWorld, Grand View Research, GlobeNewswire, (4) recent M&A activity in the segment. Prioritize industry reports over blog posts." --processor ultra-fast --no-wait --json -o research/<company>/<company>_competitive

parallel research run "[COMPANY] GenAI resilience and acquisition attractiveness: (1) AI substitution susceptibility for physical vs digital layers, (2) operational AI uplift ceiling with quantified benchmarks (McKinsey, BCG, UPS ORION), (3) moat durability -- network effects, data flywheel, switching costs, (4) Physical AI optionality (AV, drones, robotics), (5) roll-up economics. Use PE/M&A analyst sources." --processor ultra-fast --no-wait --json -o research/<company>/<company>_genai

parallel research run "[COMPANY] technology platform and AI capabilities for M&A due diligence: (1) proprietary systems, (2) ML/AI capabilities and what they optimize, (3) patents or IP filings, (4) tech leadership, (5) tech stack details, (6) partnership and pricing model analysis -- channel mix, pricing structure, pricing trends, customer concentration by channel. Use authoritative sources: company tech pages, engineering blogs, patent databases, LinkedIn, trade press." --processor ultra-fast --no-wait --json -o research/<company>/<company>_techmoat
```

#### Layer 2: Claude Code Research Agent Team (parallel with Layer 1)

While Parallel CLI runs are in flight, launch 4 Claude Code `general-purpose` agents using the Agent tool. Each agent uses web search, URL extraction, and analysis to build a complementary research file. Run all 4 in the background simultaneously.

**Agent 1 — Financials & Operations:**
> Research [COMPANY] for PE/M&A due diligence. Find: (1) revenue and revenue growth, (2) EBITDA/operating margins, (3) employee count and trajectory, (4) funding history with amounts/investors, (5) customer count and key logos, (6) retention/churn metrics (GRR, NRR if available), (7) geographic footprint. Use web search, Crunchbase, PitchBook extracts, SEC filings, press releases. Save findings as structured markdown to research/<company>/<company>_agent_financials.md.

**Agent 2 — Competitive & Market:**
> Research [COMPANY] competitive landscape for M&A analysis. Find: (1) top 5 direct competitors with revenue/funding/differentiation, (2) market share estimates, (3) TAM/SAM/SOM with sources, (4) recent M&A in the vertical (last 24 months), (5) pricing comparison across competitors. Save to research/<company>/<company>_agent_competitive.md.

**Agent 3 — GenAI Risk & Disruption:**
> Assess [COMPANY] GenAI disruption risk for PE acquisition. Analyze: (1) which core functions are automatable by AI agents, (2) specific AI products/startups threatening this vertical, (3) customer AI adoption signals, (4) pricing model vulnerability to AI automation, (5) partnership dependency risks. Save to research/<company>/<company>_agent_genai.md.

**Agent 4 — Tech Moat & Switching Costs:**
> Assess [COMPANY] software moat for PE acquisition using the SC_Depth framework. Score: (1) workflow embedding depth (system-of-record status, compliance certs), (2) data gravity (proprietary data, retention requirements), (3) integration complexity (API count, SI ecosystem), (4) contractual lock-in (contract terms, ETFs), (5) brand/relational moat (G2 reviews, NPS, community). Also assess data moat type (Travis May taxonomy) and platform ecosystem. Save to research/<company>/<company>_agent_techmoat.md.

#### Layer 3: Collect & Synthesize

Poll Parallel CLI results:
```bash
parallel research poll <run_id_1> -o research/<company>/<company>_financials --timeout 600
parallel research poll <run_id_2> -o research/<company>/<company>_competitive --timeout 600
parallel research poll <run_id_3> -o research/<company>/<company>_genai --timeout 600
parallel research poll <run_id_4> -o research/<company>/<company>_techmoat --timeout 600
```

Then synthesize all 8 research files (4 Parallel CLI + 4 Claude Code agent) into the scoring in Step 3. Cross-reference findings between layers — where they agree, confidence is high; where they diverge, flag for deeper investigation.

### Step 3: Score the Target

#### 3a. Vertical Scoring — The Napkin

Score the target's vertical using three questions (1-5 Likert, weighted 60/60/40):

| Question | Weight | What It Measures |
|---|---|---|
| Q1: Can AI do the work? | 60 | GenAI task automatability % of core revenue-generating work |
| Q2: Can you keep the gains? | 60 | Defensibility Index |
| Q3: Can a robot show up? | 40 | Physical AI / TSR margin lift |

**Defensibility Index:**
```
D_v = (L_v + (1 - C_v / C_max) + P_v) / 3
```
- L_v = licensing proportion
- C_v = annual churn; C_max = 0.35
- P_v = physical presence proportion

For software targets, replace `(1 - C_v/C_max)` with `SC_Depth_normalized = (SC_Depth - 1) / 4`.

**Composite:** `S_v = 0.6 * Q1 + 0.6 * Q2 + 0.4 * Q3`

#### 3b. Quadrant Classification

| Quadrant | Criteria | Margin Retention | Target MOIC |
|---|---|---|---|
| AI Goldmine | High auto (A_v >= 40%), high moat (D_v >= 0.70) | 80-90% | ~2.1x |
| Contrarian Bet | Low auto, high moat | 85-95% | 1.9-2.3x |
| Sand Castle | High auto, low moat | 25-40% | ~0.48x (avoid) |
| Rising Tide | Low auto, low moat | 50-65% | Varies |

#### 3c. Company-Level Subscores

| Subscore | Definition | Backtest Signal |
|---|---|---|
| Wind | Vertical composite S_v | r = +0.201, p = 0.008 — **only significant predictor** |
| Sail | normalized(R&D intensity + AI patent count + mgmt AI commentary) | NOT significant (p = 0.478) |
| Fit | normalized(inverse EV/Revenue, growth-adjusted) | Significant but size proxy |
| AI Signal | normalized(R&D intensity x AI keyword frequency) | **Negative** (p = 0.008) — hype discount |

**Interpretation:** Vertical selection dominates. Don't overpay for "AI-ready." High AI Signal = worse returns.

#### 3d. Software Moat Overlay (if software target)

**SC_Depth** — Five-factor weighted rubric (each 1-5):

| Dimension | Weight | What It Captures |
|---|---|---|
| D1: Workflow & Compliance Embedding | 30% | System-of-record, process dependency, regulatory re-cert |
| D2: Data Gravity & Format Lock-In | 25% | Data volume, proprietary schema, retention requirements |
| D3: Technical Integration Complexity | 20% | Integration count, API dependencies, SI ecosystem |
| D4: Financial & Contractual Lock-In | 15% | Contract length, ETFs, implementation amortization |
| D5: Relationship & Brand Ownership | 10% | Direct end-user relationship, brand equity, NPS, CS depth |

```
SC_Depth = 0.30 * D1 + 0.25 * D2 + 0.20 * D3 + 0.15 * D4 + 0.10 * D5
```

**Data Moat Classification** — Travis May's six-type taxonomy (Data Currency > Long-Tail Aggregation > Give-to-Get > Exclusive Source > Proprietary Creation > Exhaust Data). Three-criteria test: (1) substantial competitive value? (2) genuinely prevents competitor access? (3) no functional substitutes incl. synthetic data?

**Platform/Ecosystem Moat** — Score across: ecosystem depth, integration moat depth, network effect strength, switching cost friction, ecosystem governance. NRR >120% is the single best proxy.

**Composite Software Moat:**
```
Software_Moat = 0.30 * SC_Depth + 0.25 * DataNet + 0.20 * Expansion + 0.15 * Displacement + 0.10 * AI_Trajectory
```

| Score | Tier | CAP | PE Implication |
|---|---|---|---|
| 4.5-5.0 | Wide Moat | 15+ yr | Premium valuation. Underwrite price increases. |
| 3.5-4.4 | Narrow Moat | 7-15 yr | Standard PE. Moat deepening is the thesis. |
| 2.5-3.4 | Nascent Moat | 3-7 yr | Discount for moat risk. Execution-dependent. |
| <2.5 | No Moat | <3 yr | Avoid as platform investment. Growth story only. |

#### 3e. Partnership Dependency (if applicable)

Score 1-5 (inverted — 5 = highest risk) when single partnership >15% of revenue. Feeds genai_risk table. Score >= 4 = gating diligence item.

#### 3f. Pricing Model Vulnerability

Score 1-5 (5 = most durable). If score <= 2: cap AI_Trajectory at 3, require pricing pivot scenario in 100-Day.

| Score | Model | AI Resilience |
|---|---|---|
| 5 | Outcome/value-based or regulatory-mandated | Highest |
| 4 | Platform subscription + consumption upside | High |
| 3 | Hybrid (seat + consumption) | Medium |
| 2 | Pure per-seat/per-agent | Low |
| 1 | Per-minute/per-interaction in AI-automatable domain | Lowest |

### Step 4: Create JSON Data File

Create a JSON file with all scores and evidence. Save to `research/<company>/<company>_memo_data.json`.

**Required fields:**
```json
{
  "company_name": "string",
  "subtitle": "Target Acquisition Memo — [Vertical]",
  "quadrant": "AI Goldmine | Contrarian Bet | Sand Castle | Rising Tide",
  "date": "Month YYYY",
  "kpis": [["Metric", "Value"], ["Revenue", "$XXM"], ...],
  "thesis": "paragraph",
  "scores_summary": {"Wind": 0.0, "Sail": 0.0, "Fit": 0.0, "SC_Depth": 0.0},
  "recommendation": "paragraph",
  "napkin": {
    "vertical": "string",
    "composite": "X.X / 8.0",
    "quadrant": "string",
    "questions": [
      {"label": "Q1: Can AI do the work?", "score": 0, "weight": 60, "rationale": "..."},
      {"label": "Q2: Can you keep the gains?", "score": 0, "weight": 60, "rationale": "..."},
      {"label": "Q3: Can a robot show up?", "score": 0, "weight": 40, "rationale": "..."}
    ],
    "evidence": ["bullet1", "bullet2"]
  },
  "company_subscores": [
    {"name": "Wind", "score": 0.0, "definition": "...", "evidence": "..."},
    {"name": "Sail", "score": 0.0, "definition": "...", "evidence": "..."},
    {"name": "Fit", "score": 0.0, "definition": "...", "evidence": "..."},
    {"name": "AI Signal", "score": 0.0, "definition": "...", "evidence": "..."}
  ],
  "software_moat": {
    "sc_depth": [
      {"dimension": "Workflow & Compliance", "weight": "30%", "score": 0, "rationale": "..."}
    ],
    "composite_scores": {"SC_Depth": 0.0, "Data Moat": 0.0, "Platform": 0.0, "Software_Moat": 0.0},
    "tier": "Wide Moat | Narrow Moat | Nascent Moat | No Moat"
  },
  "data_moat": {
    "classification": ["bullet1"],
    "platform": ["bullet1"],
    "ai_trajectory": ["bullet1"]
  },
  "competitive": {
    "table": [["Competitor", "Revenue", "Funding", "Differentiation"], ...],
    "notes": ["bullet1"]
  },
  "genai_risk": {
    "table": [["Risk Vector", "Severity", "Timeline", "Mitigation"], ...],
    "analysis": ["bullet1"]
  },
  "sources": ["source1", "source2"],
  "playbook_100day": ["flat bullet array (backward compat)"],
  "playbook_100day_structured": {
    "phases": [
      {"phase": 1, "name": "Financial Foundation", "days": "1-30", "items": [], "gate": "..."}
    ]
  }
}
```

**Optional fields:**
```json
{
  "partnership_dependency": {"partner": "", "score": 0, "revenue_dependency": "", "exclusivity": "", "termination_risk": "", "integration_depth": "", "mitigation": ""},
  "pricing_model_risk": {"score": 0, "model": "", "trend": "", "ai_impact": ""},
  "v2_delta": {"trigger": "", "scores_v1": {}, "scores_v2": {}, "quadrant_changed": false, "material_changes": [], "recommendation_changed": false, "rationale": ""}
}
```

### Step 5: Generate PPTX

Run the PPTX generator. The script resolves the template path relative to the skill directory automatically.

```bash
# Using skill-bundled template (default):
python3 ~/.claude/skills/target-scoring/scripts/gen_target_memo_pptx.py \
  research/<company>/<company>_memo_data.json \
  research/<company>/<company>_Target_Memo.pptx

# Or with explicit template:
python3 ~/.claude/skills/target-scoring/scripts/gen_target_memo_pptx.py \
  research/<company>/<company>_memo_data.json \
  research/<company>/<company>_Target_Memo.pptx \
  ~/.claude/skills/target-scoring/templates/moat_in_the_machine_v2.pptx
```

### Step 6: Review & Present

1. Confirm PPTX was generated successfully (check slide count = 9)
2. Present a summary to the user:
   - Quadrant classification + recommendation
   - Key scores: Wind, Sail, Fit, SC_Depth, Software_Moat
   - Top 3 risks from genai_risk
   - Investment thesis (1-2 sentences)
3. Ask if they want to adjust any scores or run a v2 rescore

## V2 Rescore Process

Trigger a v2 when:
- Data delta >15% from v1 estimates
- Composite score within 5 points of quadrant boundary
- Material new information (funding, acquisition, product launch)
- IC request for deeper analysis

V2 refinement principles:
1. Name specific competitors from v1
2. Target specific data gaps
3. Include vertical-specific AI threats
4. Request pricing model data
5. Cross-validate with bull/bear framing

All v2 artifacts use `_v2_` infix. Produce a `v2_delta` JSON object documenting changes.

## 100-Day Playbook Template

| Phase | Days | Focus | Gate |
|---|---|---|---|
| 1. Financial Foundation | 1-30 | ARR validation, cohort retention, margin structure, cash flow | Financial Summary Memo |
| 2. Customer & SC Validation | 15-45 | 10-15 reference calls, G2 analysis, integration audit, SC_Depth validation | SC Validation Report |
| 3. Technology & AI | 30-60 | CTO interview, IP audit, AI capability, platform/ecosystem, security | Technology Assessment Memo |
| 4. Competitive & Market | 45-75 | Independent market sizing, competitor deep-dive, win/loss, pricing vulnerability | Market & Competitive Memo |
| 5. Financial Model | 60-90 | 3-scenario model, comps, sensitivity on SC_Depth erosion | IC Slide Deck |
| 6. IC Memo | 75-100 | Full IC memo, updated PPTX, risk register, deal structure, Go/No-Go | Go/No-Go Decision |

## Design System

- **Font:** Open Sauce One
- **Dark slides:** bg dark navy, headers #E8563A (accent red), body #C5CCD6, muted #8B95A5
- **Light slides:** headers #E8563A, dark text #0F1B2D, body #5A6577
- **Quadrant colors:** Green #2E7D32, Blue #1565C0, Red #C62828, Amber #F9A825
- **Slide size:** 10" x 5.625" (9144000 x 5143500 EMU)
- **Footer:** "Target Memo: {company} | Dutchess Management | {date} | Confidential"

### Layout Engine

The PPTX generator uses a `LayoutCursor` system that **guarantees no element overlaps the footer**:

- `HEADER_TOP = 274320` — consistent section header y-position across all slides
- `CONTENT_TOP = 620000` — content starts below header
- `CONTENT_BOTTOM = FOOTER_Y - 50000` — hard ceiling, nothing placed below
- Tables use `col_ratios` for proportional column widths (text-heavy columns get 44-67%)
- All table cells use `MSO_ANCHOR.TOP` vertical alignment with word wrap
- Minimum font size: 8pt for tables, 9pt for bullets

### Slide Structure (9 slides)

| # | Slide | Layout | Key Elements |
|---|---|---|---|
| 1 | Title | DARK | Company name, subtitle, quadrant badge, date |
| 2 | Snapshot | LIGHT | KPI table (left), thesis (right), score badges, recommendation |
| 3 | Napkin | LIGHT | Q1/Q2/Q3 scoring table, evidence bullets |
| 4 | Company Scores | LIGHT | Wind/Sail/Fit/AI Signal table, score badges |
| 5 | Software Moat | LIGHT | SC_Depth table, composite score badges |
| 6 | Data Moat | LIGHT | Classification (left), platform (right), AI trajectory |
| 7 | Competitive | LIGHT | Competitor table, analysis notes |
| 8 | GenAI | LIGHT | Risk table, analysis bullets |
| 9 | Sources | LIGHT | Citation list |

## References

Full scoring methodology: `references/scoring_playbook.md`
Software moat rubrics: `references/software_moat_framework.md`
