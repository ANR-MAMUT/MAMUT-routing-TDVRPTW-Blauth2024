# Changelog — Blauth2024 TDVRPTW BKS

All notable changes to the `Blauth2024` best-known solutions (BKS) are recorded here. Objective: **FleetCostDuration** (sum of per-route optimal durations plus `36000000 ms` per route; the depot departure time of each route is a decision variable). Costs are the authoritative output of the canonical checker (`mamut_routing_lib.td.check_td_solution`, mamut-routing-lib >= 0.9.0): exact IEEE-754 double arithmetic, no epsilon thresholds, routes in canonical order, total summed in that order, so any strict improvement is real. The upstream dollar objective is recovered exactly as `$ = cost_ms / 180000`.

## 2026-07-27

**Family published.** 20 instances hosted (n=10 and n=500, ten cities each) plus sha256 pins for the converter-materialized n=1000/2000 tier. Later the same day the pin table was completed: all twenty n=1000/2000 pins are published in the README, with the berlin and london pins at both sizes reproduced bit-identically by an independent conversion on a second machine.

**n=1000/2000 instance descriptors added.** The twenty big-size `<Name>.vrp.json` files (locations, time windows, fleet, metadata, recorded `atf_sha256` pin; under 200 KB each) are now hosted alongside their BKS sidecars, so only the oversized ATF sidecars remain converter-materialized. Each hosted descriptor's recorded pin matches the README table.

**24 BKS records at publication:**

- **14 imported reference solutions** (BonnTour high-effort mode, credited to Blauth, Held, Müller, Schlomberg, Traub, Tröbst and Vygen): all ten n=500 cities, plus berlin and london at n=1000 and n=2000. Import discipline: each upstream solution was re-evaluated with the pristine upstream evaluator's exact rational arithmetic (hard time windows, zero violations required), repriced by the canonical checker with per-route departure re-optimization (imported cost <= the upstream working-time cost at their published start times, gains of 49 to 173 ms at n=500), and cross-checked against the paper's published Table 2 values (each published dollar value is exactly the ceiling of the imported cost in dollars, fourteen for fourteen).
- **10 first-ever n=10 entries** (TD-ILS, [kayros](https://pypi.org/project/kayros/) 1.2.0, 600 s per run): upstream publishes no solutions at this size. Every city reached a bitwise-identical best value on 20 independent seeds (costs from 46078612.787004 ms, berlin, K=1, to 97367762.957588 ms, san_francisco, K=2).
