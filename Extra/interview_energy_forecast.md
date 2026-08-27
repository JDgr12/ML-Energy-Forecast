## Nowcasting Italian Fuel and Heating Inflation from Energy Markets

---

## Context and Motivation

The energy component of Italian consumer price inflation is published monthly by
Eurostat as part of the Harmonised Index of Consumer Prices (HICP). The figure for a
given month is released around the middle of the *following* month, so for roughly
six weeks policy makers, analysts and treasuries operate without knowing what
consumer energy prices have just done.

Energy *markets*, on the other hand, print every business day. Crude oil, refined
product spot prices and the euro/dollar exchange rate are observable in real time,
and they sit upstream of what Italian households eventually pay at the pump and for
heating fuel. The transmission is neither instantaneous nor one-to-one: it passes
through refining margins, distribution costs, excise duties and VAT, each of which
delays, dampens or distorts the pass-through.

This project asks you to exploit that gap: **estimate Italian fuel and heating
inflation from market data already observable, before the official figure exists.**

The problem naturally splits in two. Consumer prices for a month depend on how the
markets behave across *that whole month*, but at any moment inside the month only
part of it has printed: a nowcast made on the 15th has seen fifteen days of trading
and must say something about the other fifteen. So before markets can be mapped onto
consumer prices, the market series themselves have to be understood and projected
forward. The work is organised accordingly: **Part 1 models each market series in its
own right; Part 2 asks how those models get you to the consumer price index.**

---

## Understanding the Market Data

Before modelling anything, it is worth being clear about what these numbers are.
Half of the work in this problem is knowing what you are looking at.

The six market series are **wholesale market quotations**, upstream of the consumer.
Nothing here is a price any household pays.

| `series_id` | What it is | Unit |
|---|---|---|
| `brent` | Spot price of Brent crude oil, FOB — the benchmark grade for European refineries and the reference against which most internationally traded crude is priced. | USD per barrel |
| `wti` | Spot price of West Texas Intermediate crude at Cushing, Oklahoma — the US benchmark grade. Tracks Brent closely, but the spread between them moves with US logistics and export conditions. | USD per barrel |
| `us_gasoline_spot` | Spot price of conventional gasoline on the US Gulf Coast: a **refined product**, one step further down the chain than crude. | USD per gallon |
| `us_diesel_spot` | Spot price of ultra-low-sulphur diesel on the US Gulf Coast. Also a refined product. | USD per gallon |
| `heating_oil_ny` | Spot price of No. 2 heating oil at New York Harbor — a distillate very close to road diesel, and the standard benchmark for heating gasoil. | USD per gallon |
| `eurusd` | The ECB euro reference exchange rate: how many US dollars one euro buys. Fixed once per business day at 16:00 CET. | USD per EUR |

**Every commodity series is quoted in dollars**: refined product quotations for the Italian and northwest-European markets are not
openly published. The US Gulf Coast and New York Harbor benchmarks are the closest
freely available daily proxies, and they co-move strongly with the European ones.

---

## Part 1 — Model the market series

**Build a predictive model for the six market series** — `brent`, `wti`,
`us_gasoline_spot`, `us_diesel_spot`, `heating_oil_ny`, `eurusd` — treating each one
as a forecasting problem in its own right. If you are short on time, you can choose a
subset of the series, or even a single one. For each series:

* Choose the **forecast horizon**, it can be daily or monthly.
* Decide the target to predict.
* Check your models against simple baselines of your choice
* Report your results relative to those baselines.

---

## Understanding the HICP

Part 2 involves the three monthly consumer price series, which are a different kind
of object from the market quotations above.

The **Harmonised Index of Consumer Prices** is the official measure of consumer price
inflation in the European Union. It is compiled to a methodology common to all member
states — that is what "harmonised" means — so that national figures are comparable
and can be aggregated into the euro area index the ECB targets. For Italy the
underlying collection is done by ISTAT and the harmonised series is published by
Eurostat, the statistical office of the European Union.

Four properties matter for this exercise:

* **It is an index, not a price.** A value of `120.0` means "prices in this category
  are 20% above their level in the base year". The reference is 2015, set as 100. The
  absolute level carries no information on its own; only *changes* in the index are
  economically meaningful. This is why inflation is a growth rate, not a level.

* **It is broken down by category** using **COICOP**, the international
  classification of household consumption. Each series here is one COICOP class, and
  each has a weight in the aggregate — weights that are revised annually to track how
  household spending shifts.

* **It is published with a lag.** The figure for a given month is released around the
  middle of the following month. This lag is the reason the problem exists.

| `series_id` | COICOP | What it measures |
|---|---|---|
| `hicp_transport_fuels_it` | 0722 | Fuels and lubricants for personal transport equipment: petrol, diesel, LPG and methane bought at Italian filling stations. The category most households experience directly, and the one that reacts fastest to crude oil. |
| `hicp_heating_it` | 0453 | Liquid fuels for domestic heating: primarily heating gasoil (*gasolio da riscaldamento*) burnt in household boilers. Chemically close to road diesel, but taxed differently and bought in bulk deliveries rather than continuously, which changes both the level and the timing of pass-through. |
| `hicp_headline_it` | CP00 | The all-items index: the single number reported as "Italian inflation". Provided as context — note that it is an aggregate which *contains* the two target components, alongside food, services, housing and everything else. |

All three are monthly, index points with 2015 = 100, spanning 1996-01 to 2025-12.

---

## Part 2 — From markets to consumer prices

**Work out how you would predict Italian consumer price inflation** for **transport
fuels** (`hicp_transport_fuels_it`) and **heating fuels** (`hicp_heating_it`), using
the market series and possibly the data and the models you built in Part 1.

This part is deliberately open. Reason about what could be used and how: which series
carry information about consumer prices, how the models from Part 1 could feed into a
prediction, and what the obstacles are. How far you take it — a worked-out design, a
first implementation, a full model — is up to you and to the time you have.

A simple approach you can explain is worth more than an elaborate one you cannot.

---

## Dataset

Nine time series in `data/`, in two interchangeable formats — use whichever you
prefer:

* `data/<series_id>.parquet` — one file per series
* `data/energy_panel.csv` — all series stacked in a single long table

### Schema

Every row is one observation of one series. Five columns, tidy long format:

| Column | Type | Description |
|--------|------|-------------|
| `date` | date | Observation date. Daily series carry the actual business day. Monthly series are stamped on the first day of the month they refer to — a labelling convention, not an observation date and not a publication date. |
| `series_id` | string | Identifier of the series. |
| `value` | float | The observed value, in the unit given above. |
| `source` | string | Publishing institution: `fred`, `ecb` or `eurostat`. |
| `frequency` | string | Native frequency: `daily` or `monthly`. |

### Coverage

| `series_id` | Role | Frequency | Observations | Coverage |
|---|---|---|---|---|
| `brent` | market | daily | 9958 | 1987-05 → 2026-08 |
| `wti` | market | daily | 10226 | 1986-01 → 2026-08 |
| `us_gasoline_spot` | market | daily | 10099 | 1986-06 → 2026-08 |
| `us_diesel_spot` | market | daily | 5063 | 2006-06 → 2026-08 |
| `heating_oil_ny` | market | daily | 10100 | 1986-06 → 2026-08 |
| `eurusd` | market | daily | 7075 | 1999-01 → 2026-08 |
| `hicp_transport_fuels_it` | consumer price | monthly | 360 | 1996-01 → 2025-12 |
| `hicp_heating_it` | consumer price | monthly | 360 | 1996-01 → 2025-12 |
| `hicp_headline_it` | context | monthly | 360 | 1996-01 → 2025-12 |

`market` series are the targets of Part 1 and the inputs of Part 2; `consumer price`
series are the targets of Part 2.

The daily series carry business days only, but they do not all observe the same
holidays: the commodity series follow the US calendar, the exchange rate follows the
ECB one.

---

## Deliverables

You are free to structure the work as you see fit. By the end you should be able to
answer three questions:

1. **How predictable are the market series?** What did you build,
   what did you measure it against, and what does your evaluation actually establish?
2. **How would you get from the market series to the consumer price index?** Which
   series would you use, how would the models from Part 1 fit in, and what makes it
   hard? However far you took it, be ready to explain the reasoning.
3. **What did you do to the data, and what went wrong?** Which problems did the raw
   series force you to solve, and how did you solve them?

**Deliverable:** a public GitHub repository containing the code and at least one
document describing the process and the results — notebook, report, or any other
format you consider appropriate. You will present and discuss the work at the
interview.

---

## Practical Notes

* The two parts operate at very different sample sizes. Part 1 has thousands of daily
  observations per series; Part 2 has 360 monthly ones per target, some of which the
  information-set rule and your own feature construction will consume. This is
  deliberate, and what works in one part will not automatically work in the other.
  Neither is a large tabular problem, and both reward different choices from one.
* Daily and monthly series live on different calendars and different frequencies.
  Reconciling them is the first real decision you have to make, and it does not have
  a single correct answer.
* The series are given raw, exactly as published: not cleaned, not aligned, not
  filled. Anything you find in them is really there.
* Complete freedom in language, libraries and algorithmic approach.
* Perfect results are not expected. What matters is the clarity of the reasoning, the
  quality of the analysis, and the ability to communicate the decisions taken.

---

*Good luck!*
