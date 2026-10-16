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

## Open questions
- [ ] Reference vessel size (~7–9k TEU secondary service vs. mainline ULCV)
- [x] FBX11 historical data: check Freightos free terminal (max 1 hour). Fallback: public WCI/SCFI anchor points
- [ ] Does FBX11 include terminal handling? If THC is billed separately to shippers, handling cost may largely net out
- [ ] Correlation between freight rate and fuel price (paired bootstrap if monthly FBX11 is available)
- [ ] Spot vs. contract share of cargo
- [ ] "Through the cycle" vs. "current market" framing of the result
