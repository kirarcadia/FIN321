University of Hawaiʻi at Mānoa · Shidler College of Business

FIN-321 International Finance & Securities

FX Transaction Hedging Project — Technical Specification

# FX Transaction Hedge Model · Technical Specification

> Post-build specification documenting the Stage 2 Excel hedge model, validating it against the scenario data, and articulating the refinements required for a production-grade version. Drives the Stage 4 AI prompt and final analysis.

| Field | Value |
| --- | --- |
| **Created by** | Kira Rantamaa |
| **Updated by** | Kira Rantamaa |
| **Date Created** | 2026-04-03 |
| **Date Updated** | 2026-04-29 |
| **Version** | 0.1 |
| **LLM Used** (optional) | Claude Sonnet 4.6 — used to assist drafting Stage 3 spec from Stage 2 model |
| **Role** | Treasury Analyst / FP&A Analyst |
| **Audience** | CFO / Director of Treasury |
| **Companion Workbook** | `Rantamaa-Kira-stage2-model.xlsx` |

---

## 1. Problem Statement

A U.S.-based technology services firm expects a EUR 12,500,000 receivable from a European client, settling in 12 months. A depreciation of the euro against the dollar over that horizon would reduce realized USD proceeds and compress operating cash flow. At the spot rate of 1.1522 USD/EUR prevailing on April 3, 2026, the unhedged position is worth approximately USD 14,402,500; a 5% EUR depreciation would reduce proceeds by roughly USD 720,000. This specification documents the analytical framework used to quantify and compare four strategies — **no hedge**, **forward hedge**, **money-market hedge**, and **option (put) hedge** — and to produce the sensitivity evidence that supports the Stage 4 hedging recommendation.

---

## 2. Inputs (Known Variables)

All inputs are exposed as workbook named ranges so the Calculation Flow (§4) is portable across Excel, Python, or an AI prompt. Market inputs (spot, forward, rates, premia) are the only cells an analyst should adjust for scenario work. Rates and prices sourced from Bloomberg as of April 3, 2026; interest rates reflect SOFR (USD) and €STR (EUR).

### 2.1 Core Inputs

| Standardized Name | Description | Unit | Value | Source |
| --- | --- | --- | --- | --- |
| `FC_AMT` | Foreign-currency receivable | EUR | 12,500,000 | Contract |
| `S0_in` | Spot exchange rate at inception (EUR/USD) | USD per EUR | 1.1522 | Bloomberg |
| `F0_in` | 1-year forward rate (EUR/USD) | USD per EUR | 1.0910 | Bloomberg |
| `R_USD` | U.S. 1-year interest rate (SOFR) | Annual % | 3.65% | Bloomberg |
| `R_FC` | EUR 1-year interest rate (€STR) | Annual % | 1.93% | Bloomberg |
| `T_DAYS` | Days to settlement | Days | 360 | Assumed |
| `BASIS` | Day-count denominator (simplified single value) | Days | 360 | ACT/360 convention |
| `BASIS_USD` *(rigorous variant)* | USD-leg day-count denominator | Days | 360 | ACT/360 |
| `BASIS_FC` *(rigorous variant)* | EUR-leg day-count denominator | Days | 360 | ACT/360 (EUR money market) |
| `K_PUT` | Put option strike price (EUR/USD) | USD per EUR | 1.0910 | At-the-money-forward |
| `PREM_PUT` | Put premium per unit of EUR | USD per EUR | 0.0170 | Bloomberg indicative mid-market |

*No call option is modeled in Stage 2. `K_CALL` and `PREM_CALL` are reserved for the Stage 4 collar extension.*

### 2.2 Derived / Intermediate Values

| Name | Description | Formula |
| --- | --- | --- |
| `DF_USD` | USD accumulation factor | `1 + R_USD × T_DAYS / BASIS` |
| `DF_FC` | EUR accumulation factor | `1 + R_FC × T_DAYS / BASIS` |
| `MM_BORROW` | EUR amount borrowed today (PV of receivable) | `FC_AMT / DF_FC` |
| `MM_CONVERT` | USD proceeds from spot conversion | `MM_BORROW × S0_in` |
| `MM_INVEST` | USD proceeds at maturity (money market result) | `MM_CONVERT × DF_USD` |
| `FV_PREM_PUT` | Future value of put premium at settlement | `−PREM_PUT × FC_AMT × DF_USD` |
| `S_T_grid` | Sensitivity spot grid at settlement | `S0_in ± 5%` in 1% steps (13 rows) |
| `USD_NO_HEDGE` | USD proceeds under no hedge | `S_T × FC_AMT` |

---

## 3. Assumptions & Constraints

- **Quote convention:** All rates expressed as USD per unit of EUR. A higher quote means EUR appreciation.
- **Horizon:** Single-maturity model; `T_DAYS = 360`. A 1-year tenor is assumed throughout.
- **Day-count basis:** Simple interest is used throughout. The day-count convention is ACT/360 for both USD and EUR legs (`BASIS = 360`), consistent with money market convention. A rigorous build should split into `BASIS_USD = 360` and `BASIS_FC = 360` to allow per-leg adjustment.
- **Parity:** The money-market hedge is expected to replicate the forward hedge under covered interest rate parity (CIP). The Stage 2 model shows a significant divergence ($14,645,532 vs. $13,637,500) because the quoted forward of 1.0910 reflects market-priced EUR weakness well beyond what CIP alone predicts from the given rates. The CIP-implied forward using the quoted rates is approximately 1.1717 USD/EUR. This divergence is flagged for resolution in §6.2 and Stage 4.
- **Option premium:** Paid upfront in USD, quoted per 1 EUR (no contract multiplier). Treated as a negative cash flow at t₀ and carried forward at `R_USD` to place it on the same settlement-date footing as USD proceeds.
- **Put strike:** Set at-the-money-forward (`K_PUT = F0_in = 1.0910`), consistent with standard hedging practice.
- **Transaction costs:** All brokerage fees, credit charges, and bid-ask spreads excluded for simplicity.
- **Counterparty / credit risk:** Excluded. All derivatives assumed frictionless and creditworthy.
- **Tax / accounting treatment:** Excluded. Model reports pre-tax cash outcomes only.
- **Scenario construction:** Future spot `S_T` is varied deterministically across a grid; no probability weights or implied-volatility distribution are applied.

---

## 4. Calculation Flow

Described in named-range pseudocode so the logic is portable across Excel, Python, and AI prompts. Written for a **receivable** exposure.

### Step 1 — Derived Inputs

1. `DF_USD` = `1 + R_USD × T_DAYS / BASIS` = `1 + 0.0365 × 360/360` = **1.0365**
2. `DF_FC` = `1 + R_FC × T_DAYS / BASIS` = `1 + 0.0193 × 360/360` = **1.0193**
3. `FV_PREM_PUT` = `−PREM_PUT × FC_AMT × DF_USD` = `−0.017 × 12,500,000 × 1.0365` = **−$220,256**

### Step 2 — Forward Hedge (certainty benchmark)

- `USD_FWD` = `FC_AMT × F0_in` = `12,500,000 × 1.0910` = **$13,637,500**
- Locked in at t₀; invariant across the `S_T` grid.

### Step 3 — Money-Market Hedge (parity check)

1. **Borrow** the present value of the EUR receivable today:
   `MM_BORROW` = `FC_AMT / DF_FC` = `12,500,000 / 1.0193` = **EUR 12,263,318**
2. **Convert** to USD at spot:
   `MM_CONVERT` = `MM_BORROW × S0_in` = `12,263,318 × 1.1522` = **$14,129,795**
3. **Invest** USD to maturity:
   `MM_INVEST` = `MM_CONVERT × DF_USD` = `14,129,795 × 1.0365` = **$14,645,532**
4. At settlement, the EUR receivable repays the EUR borrowing exactly; the USD deposit matures to the locked-in amount.

> **Parity check:** `USD_MM ≈ USD_FWD` under CIP. The $1,008,032 gap in the Stage 2 model indicates the quoted forward of 1.0910 diverges from the CIP-implied forward (~1.1717). This is the most significant finding from Stage 2 and must be reconciled in Stage 4.

### Step 4 — Option Hedge (floor with upside)

Put-and-hold strategy on the receivable:

- At settlement, for each `S_T` on the grid:
  - If `S_T < K_PUT` (put in-the-money): `USD_PUT(S_T)` = `K_PUT × FC_AMT + FV_PREM_PUT` = **$13,417,244** (floor)
  - If `S_T ≥ K_PUT` (put expires): `USD_PUT(S_T)` = `S_T × FC_AMT + FV_PREM_PUT`
  - General form: `USD_PUT(S_T)` = `MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT`

### Step 5 — Sensitivity Table (rows of the grid)

For each `S_T` in `S_T_grid`:

| Column | Output | Formula |
| --- | --- | --- |
| No hedge | `USD_NO_HEDGE(S_T)` | `S_T × FC_AMT` |
| Forward | `USD_FWD` | constant across rows |
| Money market | `USD_MM` | constant across rows |
| Option (put) | `USD_PUT(S_T)` | `MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT` |
| Hedge profit | `USD_k − USD_NO_HEDGE` | one sub-column per strategy |
| Overall winner (incl. no hedge) | label | `ARGMAX(USD_NO_HEDGE, USD_FWD, USD_MM, USD_PUT)` |
| Best active hedge (excl. no hedge) | label | `ARGMAX(USD_FWD, USD_MM, USD_PUT)` |

### Step 6 — Summary Metrics (scalar outputs)

- `USD_FLOOR_PUT` = `MIN(USD_PUT)` across `S_T_grid` = **$13,417,244**
- `USD_BASE_k` = `USD_k` evaluated at `S_T = S0_in = 1.1522` for each strategy (the "baseline" row)

---

## 5. Outputs

| Output | Description | Format | Purpose |
| --- | --- | --- | --- |
| Input panel | All named-range inputs with units, sources, and values | Top of worksheet | Single source of truth |
| Strategy summary | `USD_FWD`, `USD_MM`, `USD_BASE_k`, `USD_FLOOR_PUT` | Table above sensitivity grid | Executive at-a-glance |
| Sensitivity table | USD proceeds for each strategy across `S_T_grid` ± 5% | 13-row table | Core analytical evidence |
| Hedge-profit columns | `USD_k − USD_NO_HEDGE` per row | Sub-table | Isolates hedge value-add |
| Winner / best-hedge labels | `ARGMAX` labels per row | Two label columns | Quick-read decision cue |
| Sensitivity chart | Line chart of USD outcome vs. `S_T` for all four strategies | Embedded chart *(to be added in Stage 4)* | Visual comparison |
| Executive summary | Narrative recommendation | Stage 4 memo | Downstream deliverable |

### 5.1 Base-Case Output Values (at S_T = S0_in = 1.1522)

| Strategy | USD Proceeds | Hedge Profit vs. No Hedge |
| --- | --- | --- |
| No hedge | $14,402,500 | — |
| Forward hedge | $13,637,500 | −$765,000 |
| Money market hedge | $14,645,532 | +$243,032 |
| Option (put) hedge | $14,182,244 | −$220,256 |

---

## 6. Model Review — What Worked & What to Improve

### 6.1 What Worked

- **Four-strategy comparison on one canvas.** No hedge, forward, money market, and option are all priced against the same `S_T` grid, making trade-off inspection immediate.
- **Winner / best-hedge labels per row.** Two columns identify the dominant strategy at each scenario — a useful read for a non-quant stakeholder.
- **Put payoff vectorized across the grid.** The option column applies `MAX(0, (K_PUT − S_T) × FC_AMT)` for every scenario, so the kinked payoff curve is directly visible.
- **Baseline marker at `S_T = S0_in`.** The annotation anchors the scenario range to the current market reference point.
- **Clean input/output separation.** Editable inputs are grouped at the top, clearly separated from formula cells.

### 6.2 What to Improve

- **Forward/money-market parity gap must be reconciled.** The $1,008,032 gap between `USD_FWD` ($13,637,500) and `USD_MM` ($14,645,532) is the most important issue in the Stage 2 model. The two should tie within rounding under CIP. Either the quoted forward of 1.0910 or the quoted interest rates are inconsistent; Stage 4 should flag this explicitly and use the CIP-implied forward for a clean comparison.
- **Named-range coverage is incomplete.** Intermediate steps (`MM_BORROW`, `MM_CONVERT`, `MM_INVEST`) lack standardized named ranges. Add the full §2.2 set to make every formula auditable.
- **Sensitivity step size is hard-coded.** Introduce a `STEP_FRAC` input so the grid is `S0_in × (1 + n × STEP_FRAC)` for `n = −5…+5`, giving 11 symmetric rows driven by a single toggle.
- **No sensitivity chart.** A line chart (USD proceeds vs. `S_T`, one series per strategy) is required for executive presentation and for Stage 4.
- **Collar strategy is absent.** Adding a collar (buy put + sell call) would reduce net premium cost and is the most common real-world extension. Priority improvement for Stage 4.
- **EUR rate sensitivity is informal.** The model flags a "what if 8%?" EUR rate scenario in a comment; this should be formalized as a second sensitivity table in Stage 4.
- **Day-count is implicit.** Introducing `T_DAYS` and a `BASIS` toggle makes the model reusable at non-annual tenors without formula edits.

### 6.3 Auditability Checklist

- Every input has a standardized named range from §2.1
- Every formula in §4 uses named ranges — no bare cell references
- Money-market hedge ties to forward hedge within 0.05% under CIP (parity check)
- Put payoff at `S_T = K_PUT` equals `K_PUT × FC_AMT + FV_PREM_PUT` (kink verification)
- Sensitivity grid is symmetric around `S_T = S0_in` and driven by `STEP_FRAC`
- Notes tab records spot / forward / rate sources with access dates
- Cell colors match the legend: yellow = inputs, blue = assumptions, black = formulas, green = cross-tab links

---

## 7. Sensitivity Plan

- **Grid:** `S_T_grid` spans `S0_in × (1 ± 5%)` in 1% increments → 11 rows (including the baseline at `S_T = 1.1522`). Stage 2 uses 0.01 absolute steps producing 13 rows; Stage 4 will normalize to 11 symmetric percentage-based rows via `STEP_FRAC`.
- **Strategies plotted:** no hedge, forward, money market, put option.
- **Primary chart:** line chart with `S_T` on the x-axis and USD proceeds on the y-axis. Forward and money-market series are horizontal by construction; no-hedge is a straight line; option is piecewise-linear with a kink at `K_PUT = 1.0910`.
- **Secondary table:** hedge profit vs. no hedge for each strategy, making the visual intuition numeric.
- **What the chart communicates:** the trade-off between certainty (forward / money-market, flat lines), optionality (put, kinked payoff), and naked exposure (no hedge, unbounded on both sides).
- **Chart series formatting (per brand standards):** No hedge — Black `#000000`, solid, 1.5 pt · Forward — UH Green `#024731`, solid, 2.0 pt · Money market — UH Green 700 `#013D26`, dashed, 1.5 pt · Option — Neutral-600 `#525252`, dotted, 2.0 pt · Gridlines: Silver `#B2B2B2`, 0.5 pt, horizontal only.

---

## 8. Limitations & Next Steps

**Excluded from this model:**
- Partial / layered / dynamic hedging (static, full-notional hedge at t₀ only)
- Credit, counterparty, and settlement risk
- Implied-volatility-based option pricing (premia are scenario inputs, not Black-Scholes outputs)
- Accounting treatment (ASC 815 / IFRS 9 hedge accounting designation)
- Transaction costs, bid-ask spreads, and margin requirements
- Collar strategy and other multi-leg option structures
- Multi-currency or multi-horizon portfolio effects

**Next steps — Stage 4 will:** (a) translate the sensitivity evidence into a structured CFO recommendation memo, (b) formalize the AI prompt using §4 as the instruction block and §6.2 as the improvement brief, (c) implement the parity reconciliation and standardized named ranges, and (d) add the sensitivity line chart and collar strategy.

---

## Appendix A — Change Log

| Version | Date | Author | Change |
| --- | --- | --- | --- |
| 0.1 | 2026-04-29 | Kira Rantamaa | Initial post-build draft from Stage 2 model |

---

## Appendix B — Brand & Formatting Standards

All FIN-321 deliverables conform to the University of Hawaiʻi at Mānoa Brand Style Guide as codified in `docs/_branding/design.json` (v1.0.0).

**Primary colors:** UH Green `#024731` (headings, accents) · Black `#000000` (body text) · Silver `#B2B2B2` (borders, rules) · White `#FFFFFF` (backgrounds)

**Workbook cell color coding:** Yellow fill = editable inputs · Blue text = analyst scenario assumptions · Black text = formula cells · UH Green text = cross-tab links

**Typography:** Open Sans (web) / Avenir (print); body minimum 11–12 pt for printed copies; flush left, ragged right alignment; no centered or fully justified body text.

**Accessibility:** All text/background combinations must clear ADA AA contrast (4.5:1 for body). UH Green on white ✓ (11.5:1). Never use red type for body content. Never layer body copy on dark backgrounds.

---

*Prepared per UH Mānoa brand standards (`docs/_branding/design.json` v1.0.0). Primary green `#024731` · Black `#000000` · Silver `#B2B2B2` · Body type Open Sans Regular, 11–12 pt for printed copies · ADA-compliant contrast · Flush-left, ragged-right alignment.*
