# Dissertation Plan: UK LDI Pension Crisis — Agent-Based Stress-Testing Framework
# Quantitative Research Plan: Modelling LDI Liquidity Cascades & Collateral Dynamics

- **Degree:** BSc Computer Science, UCL
- **Supervisor:** Prof. Fabio Caccioli
- **Hard Deadline:** 2nd Week of April 2026
- **Target Grade:** First-Class (70%+)
- **Approach:** Industry-grade systemic risk framework (Buyside/Quant Perspective) bridged with Academic Rigour

---

## 1. Problem Statement

In September 2022, a rapid spike in UK gilt yields exposed a fundamental liquidity mismatch in Liability-Driven Investment (LDI) strategies. The prevailing narrative simplifies this to "leverage caused fire sales," but from an institutional buyside perspective, the crisis was explicitly a **repo and collateral squeeze**. 

LDI funds used repo markets and interest rate swaps to lever their matching assets. When gilt yields blew out, they faced simultaneous Variation Margin (VM) calls on swaps and Initial Margin (IM) increases (haircut widening) on repo lines. While funds were often technically solvent, they lacked immediate *liquid* collateral (cash/unencumbered gilts). The inability to raise cash quickly enough from pension sponsors (capital call delay) forced liquidation in a distressed, dealer-constrained market, creating a transient convexity in price impact. 

**The industry gap:** Most academic models use simplistic single-asset structures and Amihud-style linear liquidity. An industry-grade framework must model the distinction between solvency and liquidity by explicitly tracking collateral buffer depletion, recapitalisation lags, and non-linear market depth under stress.

---

## 2. Research Question

> *For a stylised population of heterogeneous LDI funds operating in a single gilt market with realistic price-impact dynamics, what combinations of initial leverage, yield-shock size, and central-bank intervention threshold determine the phase boundary between contained portfolio adjustment and systemic fire-sale cascades?*

Quant mode:
> *In a network of heterogeneous LDI portfolios constrained by repo repo haircuts and swap margin requirements, how do capital-call delays (recapitalisation speed) and non-linear liquidity regimes interact with leverage to drive the boundary between contained collateral rebalancing and systemic liquidity cascades?*
---

## 3. Objectives & Desired Outcomes

| # | Objective | Desired Outcome | Measured By |
|---|-----------|----------------|-------------|
| O1 | Assemble a calibration dataset from public sources | A reproducible data pipeline yielding gilt yield curves, estimated LDI sector leverage distributions, and gilt market depth parameters | Data loaded, documented, and version-controlled |
| O2 | Build a heterogeneous agent-based model of LDI funds and a gilt market | A simulation that reproduces the qualitative dynamics of the Sep 2022 event (yield spike → margin calls → forced sales → price drop → feedback) | Single-run time-series matching stylised facts |
| O3 | Construct a stress-test framework over a 2D parameter grid (shock size × leverage) | Phase diagrams (heatmaps) showing the cascade/no-cascade boundary | Clear, annotated visualisations with identified critical thresholds |
| O4 | Evaluate the effect of central-bank intervention on the stability boundary | Comparative phase diagrams with/without intervention at varying thresholds | Quantified shift in the critical boundary |
| O5 | Compare pre-crisis vs post-reform (BoE 250 bps resilience standard) parameter regimes | Assessment of whether the post-crisis regulatory recommendations move the system firmly inside the stable region | Side-by-side phase diagrams with policy interpretation |
| O6 | Deliver a well-structured dissertation with clear critical analysis | A submission meeting UCL CS first-class criteria across all rubric dimensions | Self-assessment against rubric before submission |

Quant mode:
| # | Objective | Desired Outcome |
|---|-----------|----------------|
| O1 | Market & Repo Calibration | Reconstruct the Gilt/SONIA swap spread and proxy repo haircut dynamics using BoE and ICMA data. |
| O2 | Institutional-Grade Agent Model | distinct accounting for cash, encumbered gilts (repo), unencumbered gilts, and swap VM obligations, rigorously tested via unit tests. |
| O3 | Liquidity Mismatch Testing | Heatmaps of cascade triggers mapped across leverage vs. sponsor recapitalisation delay. |
| O4 | Non-linear Market Depth | Replace linear impact with a regime-switching or Bouchaud propagator model to reflect dealer balance sheet constraints. |
| O5 | Policy Efficacy Evaulation | Quantify if the BoE's 250 bps buffer survives severe haircut expansions, providing actionable macroprudential alpha. |
| O6 | Academic Deliverables & Rigour | Deliver a formal BSc project report matching the UCL CS Rubric: critical literature review, robust hypothesis, clear analytical narrative, and critical consideration of weaknesses. |

---

## 4. Data Plan (Priority 1)

If you have a weak universe, you have a weak model. We need accurate spread and collateral parameters.

Fallback order for calibration (to avoid data blocking):
- First, use public BoE/DMO/FRED sources programmatically where possible.
- If public calibration is slow, use a small calibrated synthetic generator to produce conservative test inputs.
- Only attempt richer/private calibration if time permits after the MVP is working.

### 4.1 Data Sources

| Data Need | Source | Notes |
|-----------|--------|-------|
| **Yield curves (Gilts vs SONIA)** | BoE Database / FRED | Swap spreads drive basis risk. Need 10y-30y tenors. |
| **Repo Haircuts** | ICMA ERC / BoE Working Papers | Need baseline repo haircuts for long-gilts and stress-regime spikes. |
| **Gilt Market Depth** | DMO Turnover / BoE | Calibrate transient impact. Focus on the bid-ask spread blowout in late Sep. |
| **LDI Sector Specs** | BoE FSR (Dec 22, Mar 23) | Initial buffer levels (£ cash), target leverage, AUM distributions. |

## MVP Acceptance Criteria (minimal)
- Runnable CLI that executes a single scenario and outputs time-series (JSON) and a single reproducible heatmap (PNG).
- One small reproducible notebook or script that runs the MVP and regenerates the main figure.
- At least one unit test (pytest) verifying margin math (swap VM and repo haircut handling).
- Use the existing `simulation/pyproject.toml` for lint/type settings and ensure `ruff`/`mypy` pass for MVP code.

---

## 5. Model Design (Priority 2)

### 5.1 Agents: `LDIPortfolio`

Accounting needs to treat liquidity as king.
Each fund $i$ tracks:
- $Cash_i$ — Liquid cash buffer
- $G_i^{free}$ — Unencumbered gilts
- $G_i^{repo}$ — Gilts pledged as repo collateral
- $Swap_i$ — IRS receive-fixed position (hedging liabilities)
- $\delta_i$ — Capital call delay (days to receive sponsor cash)

**Margin Mechanisms:**
1.  **Swap VM:** $\Delta y_{swap} \rightarrow$ immediate cash drain.
2.  **Repo IM/Haircut ($h_t$):** Higher haircuts require posting more *Cash* or $G^{free}$ to maintain existing leverage.

**Distress Logic:** If $Cash_i < ReqMargin_i$, fund triggers capital call. If it cannot meet margin intra-day, it must fire-sell $G_i^{free}$, facing forced-liquidation spreads.

### 5.2 Market: `GiltRepoMarket`

- **Yield & Price:** Explicit linkage of repo rates, gilt prices, and swap rates.
- **Liquidity/Impact Model:** Switch from simple linear to an order-flow propagator model or regime-switching depth: 
  $\Delta P_t / P_{t-1} = - \kappa (V_t^{sell})^\beta$ 
  where $\beta < 1$ (e.g. 0.5 for square-root impact) and $\kappa$ spikes in stress regimes (dealer constraint).

### 5.3 Simulation Loop Mechanics
Steps must hit intraday liquidity timing:
1. Yield / Swap rate shock applied.
2. Haircuts $h_t$ updated based on volatility regime.
3. Compute VM on swaps and Margin on repo.
4. If $Cash$ depleted $\rightarrow$ initiate capital call (arrives at $t+\delta_i$) AND fire-sell unencumbered assets to cover immediate shortfall.
5. Price impact applies to entire market. Next step yields update.

---

## 6. Stress-Test Framework (Priority 3)

We care about multidimensional risk mapping.

**Experiments:**
1.  **E1: The Liquidity Squeeze:** Heatmap of Yield Shock vs. Recapitalisation Delay ($\delta_i$). This is completely novel and highly relevant to GP/LP structures.
2.  **E2: Haircut Contagion:** Phase diagram of Baseline Leverage vs. Repo Haircut Shock magnitude. Shows when secured funding freezes.
3.  **E3: Policy Buffer Efficacy:** Overlay BoE's 250 bps cash-buffer rule over E1. Does the 250 bps buffer survive if recapitalisation takes 5 days and haircuts triple?
4.  **E4: Convex Price Impact:** Compare cascades under linear vs square-root price impact.

---

## 7. Execution Timeline (Ultra-Aggressive 4-Week Schedule)

Given the extreme 4-week window, there is *no* room for scope creep. The model must be functional by Week 2, leaving Week 3 for automated data generation, and Week 4 exclusively for the LaTeX write-up.

Scope freeze (MVP): heterogeneous population is optional for the MVP and should be postponed until the core engine, single‑fund dynamics, and the initial grid search are working and validated.

| Phase | Details | Academic Milestones |
|------|---------|---------------------|
| **W1: Core Engine (Mar 10 - 17)** | Synthesize yield/repo data placeholders. Build the `LDIPortfolio` accounting engine (Repo + Swap math). Implement linear price impact to secure MVP. | Ensure data usage strictly complies with UCL Ethics (only public/synthetic data; no ethics approval needed). |
| **W2: Dynamics & Tests (Mar 18 - 24)** | Implement non-linear price impact (Bouchaud). Close the simulation loop to recreate the margin spiral. Write automated unit tests for math functions. | Software testing (unit tests) finalized early to fulfill the "Testing" rubric requirement. |
| **W3: Grid Search (Mar 25 - 31)** | Freeze codebase. Run E1-E4 heatmaps via automated scripts. Generate core visualizations (Seaborn/Matplotlib) and document observations. | Draft "Methodology" and "Results" sections while generating the plots. |
| **W4: Reporting (Apr 1 - Deadline)**| Stop coding. Write abstract, literature review (keep tightly bound), and critically analyze limitations. Format to strict UCL guidelines. | Final Report drafting: Ensure deep alignment with the "Critical Analysis" and "Clarity" rubric criteria. |

---

## 8. Risk Register & Mitigations (4-Week Reality Check)

| Risk | Mitigation |
|------|-----------|
| Running out of time for the Write-up | The write-up accounts for >60% of the grade. Stop all coding on Day 21 (end of W3), regardless of what is finished. |
| Overcomplicating agent logic | Stick to mechanistic margin/repo valuation rules; do not attempt "intelligent"/learning agents. MVP uses single representative fund before heterogeneous arrays. |
| Unobservable private haircuts | Parameterize $h_t$ as a static range and treat it as a shock variable in the heatmaps rather than claiming precision. |

---

## 9. Success Criteria (UCL CS First-Class Benchmark)

The project will be considered a First-Class tier (70%+) achievement if it hits the following dimensions from the UCL CS marking rubric:

1. **Background, Aims and Organisation:** A deeply researched literature review grounding the project in existing systemic risk literature (e.g. Caccioli 2014) and industry context (BoE FSRs). Aims are unambiguously formulated as quantifiable hypotheses.
2. **Difficulty Level & Achievement:** The simulation moves far beyond generic agent-based models. It fully replicates valid collateral transfer mechanics (cash vs gilts) and repo margin math, producing entirely novel "recapitalisation lag" phase boundaries.
3. **Clarity:** Codebase is structurally advanced and readable. The dissertation report presents logical flow, is cleanly formatted to university standard, with high-quality, explicitly annotated heatmaps and data visualizations. 
4. **Analysis / Testing:** Robust software engineering practices (unit testing for margin math). Crucially, an extensive "Critical Analysis" section dissecting the model's limitations, discussing assumptions (e.g., paramatised private haircuts), and contrasting simulation results with real-world complexities.
