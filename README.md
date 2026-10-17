# Shipping Voyage Monte Carlo — Asia → Northern Europe

**What is the probability that a westbound Asia–Northern Europe container voyage produces a negative voyage result?**

A Monte Carlo simulation of 100,000 voyages on the Shanghai → Rotterdam trade, built on free public data: historical freight rates, oil prices, EU emissions data and published market benchmarks.

---

## Key result

Depending on market regime, routing and how vessel utilization behaves, the probability of a loss-making westbound voyage ranges from roughly **17% to 29%**.

| Scenario | P(loss) | Median result |
|---|---|---|
| **Base case** — Suez, full sample (Jul 2021 – Aug 2026) | **16.6%** | $6.5M |
| Post-COVID sample only (Sep 2022 – Aug 2026) | 21.9% | $3.8M |
| Cape of Good Hope routing | 21.8% | $5.9M |
| Utilization linked to freight market | 21.9% | $6.0M |
| Post-COVID + Cape routing | 28.4% | $3.1M |
| Post-COVID + linked utilization | 28.5% | $2.7M |

Base case distribution: mean $13.3M, 5th percentile −$1.5M, 95th percentile $46.9M. The distribution is strongly right-skewed. A typical voyage earns around $6.5M, while the 2021–22 boom months pull the mean far above the median.

**Break-even freight rate** at baseline assumptions: about **$1,700 per FEU**.

---

## Key findings

**1. Freight rate is the dominant risk, and P(loss) moves in steps.**
Loss probability is driven mainly by how often the market falls below break-even. Because the freight sample consists of 62 observed months, several of which cluster at $1,590–1,650 (2023 and autumn 2025), small cost changes can move P(loss) by more than expected when break-even crosses one of these months.

**2. The vessel choice mattered more than any single cost.**
Moving from a ~7,000 TEU placeholder to a 9,000 TEU reference vessel cut break-even by roughly $550/FEU and P(loss) by more than half. Economies of scale are not just theory in this market.

**3. Suez beats Cape, unless war-risk insurance exceeds ~1.9% of hull value.**
Suez saves around $680k per voyage after paying the canal toll and a $250k war-risk premium. Cape becomes preferable only when the premium exceeds roughly 1.9% of a $50M hull. Because the premium is a percentage of hull value, older and cheaper vessels can return to Suez first. This is consistent with carriers sending older 8–9k TEU ships back through the canal in late 2026 while larger, newer ships stay on the Cape.

**4. Insurance uncertainty matters more than the canal toll itself.**
In the sensitivity analysis, the war-risk premium moves P(loss) twice as much as the Suez toll.

**5. Utilization is the most important assumption, and the least data-backed.**
Shifting the utilization distribution moves P(loss) by 8.2 percentage points, more than any cost input. Linking utilization to the freight market (weaker load factors when rates are below median) raises P(loss) by 5–7 points. The independent base case therefore likely understates risk.

---

## What drives the risk

One-at-a-time sensitivity of P(loss), using the same 100,000 random draws for every test:

| Input | Range tested | Swing in P(loss) |
|---|---|---|
| Utilization distribution | weak (0.60/0.75/0.90) to strong (0.80/0.90/0.98) | 8.2 pp |
| Charter rate | $65,000 – $90,500/day | 7.8 pp |
| Equipment cost | $100 – $350/FEU | 7.6 pp |
| Sea days | 28 – 33 | 5.8 pp |
| Fuel consumption | 65 – 86 t/day | 4.6 pp |
| Origin handling (THC) | $0 – $150/FEU | 4.1 pp |
| War-risk premium | $100k – $500k | 3.4 pp |
| Delay distribution | low to high | 2.6 pp |
| Port days | 4 – 7 | 2.0 pp |
| Suez toll | $400k – $600k | 1.7 pp |
| Port dues | $80k – $240k | 1.3 pp |
| EUA price | $70 – $110/t CO₂ | 1.0 pp |
| ETS leg share | 0.7 – 0.9 | 0.7 pp |

The sensitivity analysis was used to decide where further research was worth the effort. Port dues, for example, remain placeholders because tripling them moves P(loss) by only about one percentage point.

---

## Method

### Economic model

```
Voyage result = Revenue − Voyage costs

Revenue      = Freight rate (USD/FEU) × Loaded FEU
Loaded FEU   = Capacity (TEU) × Utilization / 2

Voyage costs = Fuel + Charter hire + Port dues + Suez toll + War-risk premium
             + EU ETS + FuelEU Maritime + Equipment + Origin handling
```

The model takes a **time-charter perspective**. The operator pays charter hire, while crew, maintenance and hull insurance are covered by the owner within the hire. Adding them separately would double count.

The result is a **voyage result for the westbound leg**, not company profit. Overhead is excluded, and the eastbound return leg, with lower rates and utilization, is not modelled.

### Uncertainty

| Variable | Method |
|---|---|
| Freight rate + fuel price | **Paired bootstrap**: each simulation draws one historical month and uses both the freight rate and the Brent level from that month |
| Utilization | Triangular distribution (0.70 / 0.85 / 0.98), synthetic |
| Port delay | Triangular distribution (0 / 1 / 7 days), synthetic |

Fuel price = VLSFO baseline × (Brent that month / latest Brent). The pairing preserves the observed co-movement between freight and oil (correlation 0.29). Sampling them independently would combine, for example, a 2023 trough rate with 2022 peak fuel prices, a combination that never occurred.

### Reference vessel

**NYK Venus** (IMO 9312793): 9,012 TEU, built 2007, no scrubber (burns VLSFO). The vessel operates on ONE's FE1 service from Southeast Asia, not from Shanghai. It is used as a **class proxy** for a 9,000 TEU non-eco vessel, while the route is kept as Shanghai–Rotterdam to match the freight data.

---

## Data and assumptions

Every fixed number in the model carries its source and type in a comment in the input cell, and all decisions are logged in [`decisions.md`](decisions.md).

| Input | Value | Source | Type |
|---|---|---|---|
| Freight rate | 62 months | Drewry WCI Shanghai–Rotterdam, first assessment each month, Jul 2021 – Aug 2026 | Data |
| Oil price | Monthly | FRED `MCOILBRENTEU` (Brent) | Proxy for VLSFO |
| VLSFO baseline | $822/mt | Singapore monthly average, Aug 2026 (Ship & Bunker) | Data |
| Fuel consumption | 75.1 t/day | EU THETIS-MRV 2025, NYK Venus | Data |
| Charter rate | $76,500/day | MSI 1-yr TC, 8.5k TEU non-eco, Dec 2025 | Benchmark |
| Sea days | 30 | Derived from ONE FE1 pro forma speed (~14.5 kn) | Derived |
| Suez toll | $500,000 | Published range for ~9k TEU incl. 2026 surcharge | Estimate |
| War-risk premium | $250,000 | ~0.5% of estimated $50M hull value | Estimate |
| EU ETS | ~$270,000 | 50% coverage, 100% phase-in 2026, EUA $96/t | Derived |
| FuelEU Maritime | ~$65,000 | Penalty-based upper bound | Derived |
| Equipment | $200/FEU | Lease + empty repositioning | Synthetic |
| Origin THC | $0 | Excluded from WCI on Asia–Europe (Drewry methodology) | Data |
| Port dues | $30,000/call | Placeholder, low sensitivity | Placeholder |

**Freight data quality:** of the 62 monthly values, 49 were read directly from Drewry reports, 7 were derived exactly from the following week's report, and 6 are approximations from rounded percentages (all flagged, deviation under 1% where verified).

**Consistency check:** Drewry's WCI explicitly includes the EU ETS surcharge, equipment management fee, port dues and Suez surcharge, while excluding THC on this route. Revenue therefore contains the carrier's recovery of the costs modelled on the cost side, with no double counting.

---

## Limitations

- **Westbound leg only.** The return leg is not modelled, so results overstate round-trip profitability.
- **Utilization is synthetic.** No free route-level load-factor data exists. It is the most influential assumption in the model.
- **Utilization is independent of freight in the base case.** The linked scenario shows this likely understates P(loss) by 5–7 points.
- **Discrete freight sample.** 62 months from a period dominated by unusual events (COVID, Red Sea crisis, Hormuz). No calm, "normal" period exists in the data.
- **Spot rates only.** A large share of Asia–Europe cargo moves on contract rates, which are less volatile.
- **Class proxy vessel.** NYK Venus's specifications are used for a Shanghai-origin service it does not operate on.
- **Methodology change in source data.** Drewry widened its outlier criteria in week 20 of 2021.

---

## Repository structure

```
shipping-route-monte-carlo-V1/
├── data/
│   ├── raw/                  ← data as received, never edited
│   └── processed/            ← cleaned data read by the model
├── notebooks/
│   ├── 01_fuel_price.ipynb          ← Brent analysis
│   ├── 02_fuel_consumption.ipynb    ← EU MRV analysis
│   ├── 03_freight_rate.ipynb        ← Drewry WCI analysis
│   └── 10_monte_carlo.ipynb         ← simulation, sensitivity, scenarios
├── decisions.md              ← log of every modelling decision and why
└── README.md
```

The pipeline is: **raw data → analysis notebook → processed data → Monte Carlo notebook**.

## How to run

Open `notebooks/10_monte_carlo.ipynb` in Google Colab and select **Runtime → Run all**. All data is read directly from this repository, so no uploads are needed. Results are reproducible with seed 42.


# shipping-route-monte-carlo-V1
- **Limitation – discrete freight sample:** The freight bootstrap draws from 62 observed months. P(loss) therefore changes in steps when break-even crosses an observed rate. Several months cluster at $1,590–1,650 (2023 and autumn 2025), which makes the result sensitive to small cost changes in that range.
