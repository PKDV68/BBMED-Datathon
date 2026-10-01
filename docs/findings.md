# Findings

Full write-up: **From Messy Factory Data to Actionable Insight** —
[case study document](https://claude.ai/artifact/ba5807f0-cddc-44a6-9ba6-c633053e761c) ·
[slide carousel](https://claude.ai/artifact/AUoyqbkdFPGjWdny5zNP7W)

This file is a condensed version for the repo. Figures below are from
BBMED's real dataset (889 clean orders, Jan 2025–Sep 2026); the notebook in
this repo reproduces the method on synthetic sample data.

## 1. OEE: the machines are fast, they just don't run enough

Fleet OEE across 889 clean orders: **43.5%**. Performance (running speed)
sits at 90–97% on every line. Availability (share of time actually running)
ranges from 39% to 55% and explains almost all of the difference between
lines.

| Machine | Availability | Performance | OEE |
|---|---|---|---|
| KM1 | 55.2% | 89.7% | 49.9% |
| KM2 | 46.8% | 94.1% | 44.1% |
| AXO12 | 40.6% | 95.7% | 39.4% |
| AXO11 | 39.7% | 96.1% | 38.8% |
| AXO10 | 38.7% | 96.8% | 38.1% |

## 2. Order size is the cheapest lever

OEE rises from 33.5% (smallest quarter of orders) to 51.1% (largest
quarter), a **17.6 percentage point** gap driven almost entirely by
availability, not speed. Fixed setup/cleaning cost per order gets spread
over more units on a larger order. Order size is also the strongest
correlate of availability found (Pearson r = +0.33).

## 3. Why lines stop: changeovers, not breakdowns

| Line | Organisational share | Technical share |
|---|---|---|
| AXO12 | 82% | 18% |
| AXO11 | 78% | 22% |
| AXO10 | 77% | 23% |
| KM2 | 57% | 43% |
| KM1 | 41% | 59% |

KM1 is the only line where technical faults outweigh organisational
downtime (labeller and carton-folder faults specifically).

## 4. MES vs Excel: mostly consistent, with a known tail

263 orders appear in both sources. Median absolute quantity gap: **0.65%**.
Mean: 30%, dragged up entirely by 9 orders (3.4%) where MES shows 2–55x the
Excel quantity — most likely record-splitting on multi-day orders, not
systematic error. **Report medians, not means, on this kind of data.**

## 5. Energy on KM1

Five sub-machine meters draw ~3.2 kW combined; the capper and carton folder
account for 61% of it. One fully-traced order (702026, 9,112 bottles) gives
an energy intensity of **~2.8 kWh per 1,000 bottles**, of which about a
third was drawn while the line was stopped — power barely drops during
downtime.

## 6. Decoding the stoppage table (post-event follow-up)

`mes_stoppages` looked broken at the datathon (undefined type codes,
18-day "stoppages"). Tested against the period table afterwards, it is a
state-time ledger: type 3 = producing (matches `CalcProductionTime` on
100% of 6,023 work records), type 1 = classified stops, type 0 =
unclassified micro-stops, type 2 = idle time before the period started
(r = 0.92 against the gap since the previous run).

## Recommendations

**Operations:** batch small orders · run a changeover (SMED) study on the
AXO lines · target KM1 maintenance at the labeller and carton folder ·
add standby power modes for long stops.

**Data model:** make order number mandatory on every table · standardise
machine IDs across systems · log energy continuously, tagged to the active
work record · replace or tighten the Excel log.

**Validation rules:** bound performance to a plausible range · flag
MES/Excel gaps over 10% for review · require every stoppage row to
reference an existing work record · require a (possibly zero) reject count
on every work record.

## Limitations

- No reliable Quality factor (rejects link to <3% of work records) — OEE
  reported here is Availability × Performance, an upper bound.
- Energy findings rest on ~3.5 hours of meter data and one order.
- Only ~29% of MES orders have an Excel counterpart.
- Correlations in the write-up are pairwise, not causal.
