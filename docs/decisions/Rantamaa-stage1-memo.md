# FX Hedging Decision Memo – EUR Receivable Exposure

**Created by:** Kira Rantamaa
**Updated by:** Kira Rantamaa
**Date Created:** 2026-04-03
**Date Updated:** 2026-04-03
**Version:** 1.0
**Used:** Claude 4.6

---


## Executive Summary 
OOur technology services firm expects to receive EUR 12,500,000 from a European client in 12 months. At today's spot rate of 1.1522, this receivable converts to approximately $14,402,500 in USD proceeds — but that figure depends entirely on EURUSD at maturity. A 5% depreciation of the euro would reduce USD proceeds by roughly $720,125, directly compressing margins. The 1-year forward rate of 1.0910 signals market expectations of euro weakness, placing $765,000 of spot value at risk if left unhedged. This memo outlines the exposure, explains the risk, and introduces three hedging strategies — a forward contract, a currency put option, and a money market hedge — for the CFO's consideration. Subsequent stages will build a quantitative model, document methodology, and deliver a final hedge recommendation.

---

## Background

Our firm has contracted to deliver technology services to a European client, with payment of EUR 12,500,000 due in 12 months. Because our cost base and financial reporting are USD-denominated, any weakening of the euro between now and the payment date directly erodes our realized revenue. The current EURUSD spot rate of 1.1522 values this receivable at $14,402,500 — but the 1-year forward rate of 1.0910 signals that currency markets already expect the euro to weaken significantly. Recent EURUSD volatility — driven by diverging Fed and ECB monetary policy, shifting U.S. trade policy, and eurozone growth uncertainty — makes this exposure material and time-sensitive.

**Primary objective:** Protect the USD value of the EUR 12.5M receivable against adverse euro depreciation.


---

## Method 

Three hedging strategies will be evaluated in Stages 2–4:

| Strategy | Mechanism | Pros | Cons |
|----------|-----------|------|------|
| **Forward Contract** | Lock in a sale of EUR 12.5M at the 1-year forward rate of 1.0910, guaranteeing $13,637,500 | Eliminates all FX uncertainty; no upfront cost; simple execution | Sacrifices all upside if EUR strengthens beyond 1.0910 at maturity |
| **EUR Put Option** | Purchase a EUR put option (premium = $0.017/EUR; total cost = $212,500) to set a floor on USD proceeds | Preserves upside if EUR appreciates; defines a worst-case floor | Upfront premium of $212,500 reduces net proceeds in every scenario |
| **Money Market Hedge** | Borrow EUR today at the EUR rate (€STR ≈ 1.93%), convert to USD at spot (1.1522), invest at the USD rate (SOFR ≈ 3.65%); repay EUR loan with the receivable at maturity | Replicates forward economics without a derivative; the +1.72% rate differential works in our favor | Consumes balance sheet borrowing capacity; execution depends on available credit facilities |

Each strategy will be modeled across a range of EURUSD outcomes (0.95–1.20) to compare net USD proceeds, breakeven rates, and cost-adjusted returns.

**Key inputs confirmed:**
- Spot rate: 1.1522 EURUSD (as of April 3, 2026)
- Forward rate (1-year): 1.0910
- USD interest rate (SOFR): 3.65%
- EUR interest rate (€STR): 1.93%
- EUR put premium: $0.017/EUR | EUR call premium: $0.022/EUR


---

## Next Steps


1. **Stage 2 – Excel Model Build:** Construct a working spreadsheet that computes and compares net USD proceeds for all three strategies across a full range of EURUSD outcomes at maturity.
2. **Stage 3 – Technical Specification:** Document the model's architecture, assumptions, and formulas — precise enough for an AI or analyst to reconstruct independently.
3. **Stage 4 – Final Analysis & Recommendation:** Select the optimal hedge strategy using model results, draft a structured AI prompt for sensitivity analysis, and present the final recommendation.

---

## References

- EURUSD spot rate (1.1522): Yahoo Finance / Reuters, April 3, 2026.
- SOFR (3.65%): Federal Reserve Bank of New York, April 3, 2026.
- €STR (1.93%): European Central Bank, April 3, 2026.
- FIN-321 Scenario 3 parameters (forward rate, option premiums).
- Hull, J.C. *Options, Futures, and Other Derivatives*, 11th ed. Pearson.
- Eun, C.S. & Resnick, B.G. *International Financial Management*, 9th ed. McGraw-Hill.
