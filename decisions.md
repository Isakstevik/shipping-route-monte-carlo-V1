# Decision log

What was decided and why. Organised by topic, reflecting the final state of the model. All results refer to the base case (Suez, full sample, 100,000 simulations, seed 42) unless stated otherwise.

---

## 1. Scope and model structure

- **Voyage scope:** Westbound Asia–Northern Europe leg (Shanghai → Yantian → Tanjung Pelepas → Rotterdam), not a nonstop voyage. Matches how liner services operate.
- **Perspective:** Time charter. Charter hire covers crew, maintenance and hull insurance, so these are not modelled separately (avoids double counting).
- **Output metric:** "Voyage result" for the westbound leg, not "profit". Overhead and the eastbound return leg are excluded, so calling it profit would overstate what the model covers.
- **Utilization:** Defined as average westbound utilization across the leg. Avoids modelling container flows port by port.
- **FEU conversion:** Loaded FEU = loaded TEU / 2. Simplification because the freight index is quoted per FEU.
- **Sea days vs. port days:** Split so main-engine fuel applies only to sea days. Charter applies to all days. Port delay adds port days (charter + auxiliary fuel only).
- **Single source of truth:** All fixed inputs are defined in the input cell of `10_monte_carlo.ipynb`, each with source and type in a comment. A check cell confirms that the model function and the deterministic baseline give identical results.

---

## 2. Data

- **Free data only.** Anything paywalled is replaced by a proxy, a documented estimate, or a synthetic assumption with stated range.
- **Raw vs. processed:** Raw files are never edited. Analysis notebooks (`01`–`03`) write processed files, and the Monte Carlo notebook only reads processed files.

### Freight rate
- **Source:** Drewry World Container Index (WCI), Shanghai → Rotterdam spot rate (USD/FEU). Historical FBX11 data (Freightos) was not freely available.
- **Collection:** Collected manually from Drewry's weekly reports as republished in free shipping press. One observation per month, using the first weekly assessment (first Thursday) regardless of value, to avoid selecting weeks by outcome. Every value is stored with its source URL.
- **Sample period:** Jul 2021 – Aug 2026 (62 months). Free weekly coverage before mid-2021 is too sparse to collect reliably. The sample covers five regimes: COVID peak, crash, 2023 trough, Red Sea crisis and normalisation.
- **Data quality:** 49 values read directly, 7 derived exactly from the following week's report, 6 approximated from rounded percentage changes. All flagged in the `note` column. Deviation under 1% where approximations could be verified against exact values.
- **THC treatment:** Drewry's methodology table confirms that both Asia THC and EUR THC are excluded from the Shanghai–Rotterdam rate.
- **Consistency:** WCI includes the EU ETS surcharge, equipment management fee, port dues and Suez surcharge. Revenue therefore contains the carrier's recovery of these costs, which are modelled on the cost side. No double counting.
- **Source methodology change:** Drewry widened its outlier criteria in week 20 of 2021 (from ±20% to ±30% of median). Considered acceptable for V1.

### Fuel price
- **Source:** Brent crude (FRED `MCOILBRENTEU`), monthly, Jan 2019 – Aug 2026, used as a proxy because free historical VLSFO data is not available.
- **Method:** Fuel price = VLSFO baseline × (Brent that month / latest Brent). Brent is used as levels, not returns, so freight and fuel can be drawn from the same month.
- **VLSFO baseline:** \$822/mt, Singapore monthly average for Aug 2026 (Ship & Bunker). Must match the month of the latest Brent observation (Aug 2026, \$91.08), otherwise all simulated fuel prices are biased. Singapore chosen because the service calls Tanjung Pelepas, adjacent to the Singapore bunkering hub. G20 average for the same month was \$810/mt.
- **Extreme months kept:** COVID 2020 and Hormuz 2026 moves are real market events. Only the 2026 episode falls inside the freight sample period and enters the simulation.

### Fuel consumption
- **Source:** EU THETIS-MRV 2025. Container ships with zero time at sea removed (135 ships). Lowest 5% by days at sea flagged as low exposure (112 ships), as a precautionary screen.
- **Model input:** The reference vessel's own reported value (75.1 t/day), not a fleet-wide distribution. Fleet consumption ranges from 4 to 171 t/day, mainly by vessel size, so sampling from the whole fleet would be misleading.
- **Known bias:** MRV total fuel includes some fuel used in port, so the per-sea-day value slightly overstates sea consumption.

---

## 3. Reference vessel

- **NYK Venus** (IMO 9312793): 9,012 TEU, GT 97,825, built 2007. No scrubber fitted (ShipNext, single source), so burns VLSFO.
- **Class proxy:** NYK Venus operates on ONE's FE1 service from Southeast Asia (Laem Chabang–Cai Mep–Singapore–Rotterdam–Hamburg), not from Shanghai. The model keeps the Shanghai–Rotterdam route to match the freight data and uses NYK Venus to represent a 9,000 TEU non-eco vessel.
- **Why ~9,000 TEU and not an ultra-large vessel:** Vessels of this size are regularly chartered, so public charter benchmarks exist. Ultra-large vessels are mostly owned by carriers, with almost no public charter data.
- **Alternatives considered:** NYK Orion and ONE Hamburg (similar consumption, scrubber status unknown) and CMA CGM EXEMPLARITY (the earlier placeholder, which operates Mediterranean–Caribbean and is not representative of the route).

---

## 4. Inputs and costs

| Input | Value | Basis |
|---|---|---|
| Capacity | 9,012 TEU | NYK Venus |
| Fuel consumption at sea | 75.1 t/day | NYK Venus, MRV 2025 |
| Fuel consumption in port | 5 t/day | Synthetic, auxiliary engines |
| Sea days | 30 | Derived, see below |
| Port days | 5 | Placeholder, ~1–1.5 days per call |
| Charter rate | \$76,500/day | MSI 1-yr TC, 8.5k TEU non-eco, Dec 2025 |
| Suez toll | \$500,000 | Estimate within published range |
| War-risk premium | \$250,000 | ~0.5% of estimated \$50M hull value |
| EU ETS | ~\$270,000 | Derived |
| FuelEU Maritime | ~\$65,000 | Derived, penalty-based upper bound |
| Equipment | \$200/FEU | Synthetic |
| Origin handling (THC) | \$0 | Excluded from WCI |
| Port dues | \$30,000/call | Placeholder |

- **Sea days (30):** ONE's FE1 pro forma (31 days Laem Chabang–Rotterdam via Suez, ~9,550 nm, ~3.5 port days) implies ~14.5 knots. Applied to ~10,500 nm Shanghai–Rotterdam gives ~30 sea days. Consistent with UNCTAD's ~31 days Shenzhen–Rotterdam via Suez.
- **Charter rate:** NYK Venus is non-eco. The 2026 market was ~8% firmer by Aug 2026 (Harpex), so the value is likely conservative.
- **Suez toll:** Published range \$400–700k for container ships, incl. 12% container surcharge from 15 Jul 2026. Exact Suez Canal Net Tonnage is not public. Range \$400–600k for ~9k TEU.
- **War-risk premium:** Red Sea premiums ranged from ~0.2% (Dec 2025 ceasefire) to 0.7–1.0% in 2026, with ~3% reported for some transits from Sep 2026. Base 0.5%, range 0.2–1.0%. Hull value estimated from 2026 secondhand sales of older vessels.
- **EU ETS:** Applies to the Tanjung Pelepas → Rotterdam leg only. 50% coverage (voyage into EU from non-EU port), 100% phase-in in 2026, CO₂ factor 3.114, EUA \$96/t (Aug 2026 average €82.3). Leg share 0.8 estimated from distances. Rotterdam at-berth emissions not included.
- **FuelEU Maritime:** VLSFO is ~2.6% above the 2025–29 GHG intensity limit. Same scope as ETS. Penalty €2,400 per t VLSFO-equivalent. Treated as an upper bound, since pooling is cheaper.
- **Equipment:** Container lease ~\$50/FEU (lessor rates \$0.35–0.65/TEU/day, ~50 days per leg) + empty repositioning ~\$150/FEU (between a top-down estimate of \$100–160 and a bottom-up estimate of \$250–300). Weakest input in the model.
- **Origin handling:** Set to \$0 because THC is billed separately to shipper and consignee and is excluded from the WCI rate. Kept in the sensitivity analysis (\$0–150) in case THC revenue does not fully cover terminal cost.
- **Port dues:** Kept as placeholders because tripling them moves P(loss) by about 1 percentage point.
- **Excluded costs:** Broker commission (paid by the owner under time charter), demurrage (revenue to the carrier, not a cost), general insurance (covered in charter hire), overhead.

---

## 5. Uncertainty and sampling

- **Stochastic variables:** Freight rate and fuel price (paired bootstrap), utilization and port delay (triangular distributions). Everything else is fixed. Rule: only variables that are both uncertain and material are stochastic.
- **Paired bootstrap:** Each simulation draws one historical month and uses both the freight rate and the Brent level from that month. Preserves the observed co-movement (correlation 0.29). In the final model, paired gives 16.59% vs 17.25% with independent fuel draws (−0.66 pp).
- **Utilization:** Triangular 0.70 / 0.85 / 0.98, synthetic. No free route-level load-factor data exists.
- **Port delay:** Triangular 0 / 1 / 7 days, synthetic.
- **Reproducibility:** Seed 42. Scenario runs use their own generator with the same seed, so differences reflect the scenario only.

---

## 6. Sensitivity analysis

- **Method:** One input at a time, low and high value, using the same 100,000 draws (common random numbers). Distributions tested separately by re-drawing.
- **Largest drivers:** Utilization distribution (8.2 pp), charter rate (7.8 pp), equipment (7.6 pp), sea days (5.8 pp).
- **Smallest:** Port days, Suez toll, port dues, EUA price and ETS leg share all move P(loss) by less than 2 pp. Used to decide where further research was not worth the effort.
- **War risk vs. Suez toll:** The war-risk premium (3.4 pp) matters twice as much as the toll itself (1.7 pp).
- **Utilization asymmetry:** Weak utilization raises P(loss) by 5.3 pp, strong lowers it by only 2.9 pp, because utilization is capped near full capacity.

---

## 7. Scenarios

- **Market regime:** The COVID disruption period is defined by events (Jul 2021 – Aug 2022), not by a rate threshold, to avoid selecting data on the outcome. P(loss): full sample 16.59%, post-COVID 21.85%. Downside is nearly unchanged while upside roughly halves.
- **Route (Suez vs. Cape):** Cape = 40 sea days, no Suez toll, no war risk, ETS leg share 0.84. P(loss) Suez 16.59% / Cape 21.84% (full sample), Suez 21.85% / Cape 28.44% (post-COVID). Suez median result is ~\$680k higher per voyage. Cape becomes preferable when the war-risk premium exceeds ~1.9% of a \$50M hull value. Because the premium scales with hull value, older and cheaper vessels can return to Suez first, consistent with carriers sending older 8–9k TEU ships back through the canal in late 2026.
- **Utilization linked to freight:** In months with freight below the sample median (\$3,522), utilization is drawn from a weak distribution (0.60 / 0.75 / 0.90). P(loss): 21.86% (full sample), 28.51% (post-COVID), i.e. +5.3 to +6.7 pp. Presented as a scenario, not the base case, because the link itself is synthetic.
- **Presentation:** Results are reported as a range (~17–29%) rather than a single number, because no calm "normal" period exists in the data.

---

## 8. How P(loss) changed as the model developed

Each step changed one thing. Early steps used placeholder inputs, so their absolute values are not comparable to the final result.

| Step | P(loss) |
|---|---|
| Real freight data, freight and fuel drawn independently | 19.51% |
| Paired bootstrap, fuel as Brent levels | 17.17% |
| VLSFO baseline \$822 | 20.20% |
| EU ETS added | 23.34% |
| Equipment added * | 27.63% |
| Reference vessel NYK Venus * | 12.97% |
| FuelEU + origin handling, model function corrected | 18.13% |
| Origin THC = \$0 (confirmed by Drewry) | 13.03% |
| Sea days 30 | 14.33% |
| War-risk premium \$250k | **16.59%** |

\* Equipment was accidentally applied as a fixed amount rather than per FEU in these two runs. Found and corrected with the check cell.

**Lessons:**
- The vessel choice moved P(loss) more than any single cost, through economies of scale.
- P(loss) moves in steps rather than smoothly, because the freight bootstrap draws from 62 observed months. Several months cluster at \$1,590–1,650, so small cost changes can have large effects when break-even crosses them.

---

## 9. Limitations

- **Westbound leg only.** The eastbound return leg is not modelled, so results overstate round-trip profitability.
- **Utilization is synthetic** and is the most influential assumption.
- **Utilization independent of freight in the base case.** Likely understates P(loss) by 5–7 pp (see linked scenario).
- **Discrete freight sample.** 62 months from an unusually disrupted period.
- **Spot rates only.** A large share of Asia–Europe cargo moves on contract rates, which are less volatile.
- **Class proxy vessel.** NYK Venus specifications applied to a Shanghai-origin service.
- **Limited public information** for some estimates: scrubber status, hull value and the exact Suez toll.

---

## 10. Possible extensions (V2)

- [ ] Spot vs. contract mix of cargo
- [ ] Eastbound return leg for a round-trip result
- [ ] Better source for equipment and repositioning cost
- [ ] Data-based link between utilization and freight market
- [ ] "Current market" framing (returns around today's rate) alongside "through the cycle"
- [ ] Rotterdam at-berth emissions in EU ETS
