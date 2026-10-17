# Decision log

One line per decision: what was decided and why. Newest at the bottom.

## Scope and model
- **Voyage scope:** Westbound Asia–Northern Europe leg (Shanghai → Yantian → Tanjung Pelepas → Rotterdam), not a nonstop voyage. Matches how liner services operate.
- **Perspective:** Time charter. Charter hire covers crew, maintenance and insurance, so crew is not modeled separately (avoids double counting).
- **Output metric:** "Voyage result", not "profit". Overhead/SG&A is excluded, so calling it profit would overstate what the model covers.
- **Utilization:** Defined as average westbound utilization across the leg. Avoids modeling container flows port by port.
- **FEU conversion:** Loaded FEU = loaded TEU / 2. Simplification because FBX11 is quoted per FEU.
- **Sea days vs. port days:** Split so main-engine fuel only applies to sea days. Charter applies to all days.
- **Added cost items:** Cargo handling, equipment and EU ETS added because the first baseline had a ~59% margin, far above what carriers report.

## Data
- **Free data only.** Anything paywalled is replaced by a proxy, a documented synthetic assumption, or a hardcoded value with source and date.
- **Fuel price:** Brent (FRED MCOILBRENTEU) used as a proxy for VLSFO *movements*, not levels. Assumes 1:1 % pass-through. Free historical VLSFO data is not available.
- **COVID outliers kept:** April 2020 (−42.6%) and May 2020 (+59.8%) are real market moves. Kept, and bootstrap is used instead of a normal distribution.
- **Fuel consumption:** EU MRV 2025, container ships, zero sea time removed, 5th percentile low-exposure screen. Cleaned but not yet calibrated to vessel size.
- **Raw vs. processed:** Raw files are never edited. Analysis notebooks write processed files, the Monte Carlo notebook only reads processed files.
- **Which variables are stochastic (V1):** Freight rate, fuel price, utilization, delay. Everything else is hardcoded in `assumptions.csv`. Rule: only variables that are both uncertain and material are stochastic.
- **Freight rate source:** Historical FBX11 data (Freightos) is not freely available. Replaced by the Drewry World Container Index (WCI) Shanghai → Rotterdam spot rate (USD/FEU), which is published weekly and republished in free shipping press. One observation per month (first weekly assessment of each month), Jan 2019 – Aug 2026, to match the Brent sample and allow paired sampling of freight rate and fuel price. Each value is stored with its source URL.
- - **Freight sample period:** Jul 2021 – Aug 2026 (62 months). Free monthly WCI data before mid-2021 is too sparse to collect reliably. The sample still covers five market regimes (COVID peak, crash, 2023 trough, Red Sea crisis, normalisation). Bootstrap draws months at random, so continuity is not required.
- **Freight data quality:** 49 values read directly, 7 derived exactly from the following week's report, 6 approximations from rounded percentages (all flagged in the note column, deviation <1% where verified).
- **Paired bootstrap (freight + fuel):** Each simulation draws one historical month and uses both the WCI freight rate and the Brent level from that month. Fuel price = VLSFO baseline × (Brent that month / latest Brent). P(loss): 19.51% (fuel as returns, independent) → 17.91% (fuel as levels, independent) → 17.17% (levels, paired). Pairing effect: −0.74 pp.
- **VLSFO baseline must match Brent date:** vlsfo_baseline must be the VLSFO price observed in the same month as the latest Brent observation, otherwise all simulated fuel prices are biased.
- **VLSFO baseline:** $822/mt, Singapore monthly average for Aug 2026 (Ship & Bunker), matching the month of the latest Brent observation. Singapore chosen because the service calls Tanjung Pelepas, adjacent to the Singapore bunkering hub. G20 average for the same month was $810/mt, so the choice has limited impact.
 Effect: P(loss) 17.17% → 20.20% (+3.0 pp). Larger than expected because many 2025–26 freight months lie just above scenario-specific break-even levels ($1,800–2,400).
- **EU ETS:** Included for the Tanjung Pelepas → Rotterdam leg. 50% coverage (voyage into EU from non-EU port), 100% phase-in (2026), CO2 factor 3.114, EUA $96/t (Aug 2026 avg €82.3). Leg share 0.8 is an estimate from distances. Rotterdam at-berth emissions not included in V1. Cost ≈ $347k per voyage. P(loss) 20.20% → 23.34%.
- **Pairing effect depends on cost level:** At current costs, paired vs independent differs by only ~0.1 pp (vs −0.74 pp earlier), because the months near break-even (early 2026) combine low freight with high oil.
- **Equipment cost:** $200/FEU (synthetic). Container lease ~$50/FEU (lessor rates $0.35–0.65/TEU/day, ~50 days per leg) + empty repositioning ~$150/FEU (between top-down estimate of $100–160 from BCG's 5–8% of operating costs and bottom-up ~$250–300). Weakest input in the model, range $100–350 to be tested in sensitivity analysis. P(loss) 23.34% → 27.63%.
- **Reference vessel: NYK Venus** (IMO 9312793). 9,012 TEU, GT 97,825, built 2007. Deployed on ONE FE1 Asia–North Europe. No scrubber fitted (ShipNext, single source), so burns VLSFO. Fuel consumption 75.1 t/day from own MRV 2025 record. Replaces CMA CGM EXEMPLARITY, which operates on a Mediterranean–Caribbean service and was not representative of the route.

## Open questions
- [ ] Reference vessel size (~7–9k TEU secondary service vs. mainline ULCV)
- [x] FBX11 historical data: check Freightos free terminal (max 1 hour). Fallback: public WCI/SCFI anchor points
- [ ] Does FBX11 include terminal handling? If THC is billed separately to shippers, handling cost may largely net out
- [ ] Correlation between freight rate and fuel price (paired bootstrap if monthly FBX11 is available)
- [ ] Spot vs. contract share of cargo
- [ ] "Through the cycle" vs. "current market" framing of the result
