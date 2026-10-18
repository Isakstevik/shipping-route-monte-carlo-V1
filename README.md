# Shipping Voyage Monte Carlo: Asia to Northern Europe

How likely is it that a container ship loses money on a westbound voyage from Shanghai to Rotterdam?

I built this project to answer that question with free, public data. The model simulates 100,000 voyages, each with a different combination of freight rate, fuel price, cargo volume and port delay, and counts how many of them end in a loss.

## The short answer

Somewhere between **17% and 29%**, depending on the market, the route and how full the ship is.

| Scenario | P(loss) | Median result |
|---|---|---|
| **Base case** (Suez, all data Jul 2021 to Aug 2026) | **16.6%** | \$6.5M |
| Without the COVID boom (Sep 2022 to Aug 2026) | 21.9% | \$3.8M |
| Around the Cape of Good Hope | 21.8% | \$5.9M |
| Cargo volume drops when rates are low | 21.9% | \$6.0M |
| No COVID boom, Cape routing | 28.4% | \$3.1M |
| No COVID boom, cargo volume linked to rates | 28.5% | \$2.7M |

In the base case, a typical voyage earns about \$6.5M. The average is much higher, at \$13.3M, because the 2021 and 2022 boom months produced enormous results. In the worst 5% of cases the voyage loses \$1.5M or more.

The ship breaks even at a freight rate of about **\$1,700 per FEU** (40-foot container).

## What I found

**Freight rates drive almost everything.** The risk of a loss comes down to how often the market falls below break-even. Because the model draws from 62 real months, and several of them sit just above break-even (\$1,590 to \$1,650 in 2023 and autumn 2025), small cost changes can move the result more than you would expect.

**Ship size mattered more than any single cost.** Switching from a 7,000 TEU placeholder to a 9,000 TEU ship lowered break-even by about \$550 per FEU and cut the probability of a loss by more than half.

**Suez is better than the Cape, unless insurance gets very expensive.** Going through Suez saves about \$680k per voyage, even after the canal toll and war-risk insurance. The Cape only wins if the insurance premium rises above roughly 1.9% of the ship's value. Since the premium is a percentage of value, older and cheaper ships can go back through Suez first. That matches what carriers actually did in late 2026, when older 8,000 to 9,000 TEU ships returned to the canal while larger, newer ones stayed on the Cape route.

**Insurance matters more than the canal toll.** In the sensitivity analysis, uncertainty about the war-risk premium moves the result twice as much as the toll itself.

**The most important assumption is also the weakest.** How full the ship is (utilization) has more effect than any cost, but there is no free data on it. If utilization falls when rates are low, which is likely, the probability of a loss rises by 5 to 7 percentage points. The base case probably understates the risk.

## What moves the result

Each input was set to a low and a high value while everything else stayed the same.

| Input | Range tested | Change in P(loss) |
|---|---|---|
| Utilization | weak to strong | 8.2 pp |
| Charter rate | \$65,000 to \$90,500 per day | 7.8 pp |
| Equipment cost | \$100 to \$350 per FEU | 7.6 pp |
| Sea days | 28 to 33 | 5.8 pp |
| Fuel consumption | 65 to 86 t/day | 4.6 pp |
| Origin terminal handling | \$0 to \$150 per FEU | 4.1 pp |
| War-risk premium | \$100k to \$500k | 3.4 pp |
| Port delay | low to high | 2.6 pp |
| Port days | 4 to 7 | 2.0 pp |
| Suez toll | \$400k to \$600k | 1.7 pp |
| Port dues | \$80k to \$240k | 1.3 pp |
| Carbon price (EUA) | \$70 to \$110 per t CO₂ | 1.0 pp |
| EU emissions share | 0.7 to 0.9 | 0.7 pp |

I used this table to decide where more research was worth it. Port dues are still rough estimates, for example, because tripling them barely changes the answer.

## How the model works

Each voyage is calculated like this:

```
Voyage result = Revenue − Voyage costs

Revenue       = Freight rate × Containers carried
Costs         = Fuel + Ship charter + Port dues + Suez toll + War-risk insurance
              + EU carbon costs (ETS and FuelEU) + Container equipment
```

The model looks at the voyage from the point of view of a carrier that charters the ship. Crew, maintenance and hull insurance are paid by the owner and included in the daily charter rate, so they are not added separately.

The result covers the **westbound leg only**. It is not company profit. Overhead and the return trip to Asia, where rates and volumes are lower, are left out.

**Where the uncertainty comes from**

- **Freight rate and fuel price** are drawn together from the same historical month. If a simulation picks June 2022, it uses both the freight rate and the oil price from June 2022. This keeps combinations realistic. Drawing them separately could pair a 2023 low freight rate with 2022 peak fuel prices, which never happened.
- **Utilization** and **port delay** are drawn from triangular distributions, since no free data exists.

**The reference ship** is NYK Venus: 9,012 TEU, built 2007, no scrubber, so it burns standard low-sulphur fuel (VLSFO). In reality it sails from Southeast Asia, not Shanghai. I use it to represent a typical ship of its size, while keeping the Shanghai to Rotterdam route so the freight data matches.

## Data

| Input | Value | Source |
|---|---|---|
| Freight rate | 62 monthly values | Drewry World Container Index, Shanghai to Rotterdam |
| Oil price | Monthly | FRED (Brent crude), used as a proxy for marine fuel |
| Fuel price today | \$822 per tonne | Singapore VLSFO, Aug 2026 (Ship & Bunker) |
| Fuel consumption | 75.1 t/day | EU emissions data (MRV 2025) for NYK Venus |
| Charter rate | \$76,500 per day | Market benchmark for this ship size (MSI, Dec 2025) |
| Sea days | 30 | Calculated from ONE's published schedule |
| Suez toll | \$500,000 | Estimate from published ranges |
| War-risk insurance | \$250,000 | About 0.5% of an estimated \$50M ship value |
| EU carbon costs | about \$335,000 | Calculated from EU rules and the Aug 2026 carbon price |
| Equipment | \$200 per FEU | Own estimate (container lease and empty returns) |
| Port dues | \$30,000 per call | Rough estimate, low impact |

Free historical freight data does not exist as a download, so I collected the 62 monthly values by hand (with help from Claude) from Drewry's weekly reports in the shipping press. I always used the first report of each month, so no month was picked because of its value. 49 values were read directly, 7 were calculated exactly from the next week's report, and 6 are close approximations. Every value has its source link in the raw file.

Drewry's index includes surcharges for EU carbon costs, equipment, port dues and Suez, so the revenue side already contains what carriers charge to cover the costs the model subtracts. Nothing is counted twice.

Every modelling decision and the reasoning behind it is written down in [`decisions.md`](decisions.md).

## Limitations

- Only the westbound leg is modelled, so the results look better than a full round trip would.
- Utilization has no data behind it and is the assumption that matters most.
- In the base case, utilization does not depend on freight rates, which likely understates the risk.
- The freight data covers an unusual period (COVID, the Red Sea crisis, Hormuz). There is no calm, normal period in the sample.
- Only spot rates are used. Much Asia to Europe cargo moves on contracts, which are more stable.
- NYK Venus represents a ship class. It does not actually sail this exact route.

## Repository

```
shipping-route-monte-carlo-V1/
├── data/
│   ├── processed/                   cleaned data used by the model
│   └── raw/                         data as downloaded or collected
├── notebook/
│   ├── 01_fuel_price.ipynb          oil price analysis
│   ├── 02_fuel_consumption.ipynb    fuel consumption and choice of ship
│   ├── 03_freight_rate.ipynb        freight rate analysis
│   └── 10_monte_carlo.ipynb         simulation, sensitivity and scenarios
├── README.md
└── decisions.md                     every modelling decision and why
```

## How to run it

Open `notebooks/10_monte_carlo.ipynb` in Google Colab and choose **Runtime → Run all**. The notebook reads all data directly from this repository, so nothing needs to be uploaded. Results are reproducible with seed 42.
