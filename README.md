# Blauth2024 TDVRPTW benchmark family (OSM + Uber Movement time-dependent city instances)

Satellite benchmark repository of [MAMUT-routing](https://github.com/ANR-MAMUT/MAMUT-routing) (ANR MAMUT, ANR-22-CE22-0016), mounted there as `benchmarks/TDVRPTW/Blauth2024`. Value-exact conversion of the delivery-only instances of the **vrptdt-benchmark** of Blauth, Held, Müller, Schlomberg, Traub, Tröbst and Vygen (2024): 10 cities on four sub-continents, real OpenStreetMap road networks, hourly Uber Movement speeds, dense per-arc piecewise-linear arrival-time functions.

> **License caveat, read first**: this family is **CC BY-NC 4.0**, unlike the MIT-licensed MAMUT-routing code and most other MAMUT families. The Uber Movement speed data it derives from is CC BY-NC 3.0, so every instance, sidecar and best-known-solution file here is **non-commercial use only**, with attribution (see `LICENSE` for the full block: paper, upstream GitLab repository, bonndata DOI, OSM ODbL, Uber Movement).

## Why this family

The vrptdt-benchmark is the major modern reference for realistic time-dependent vehicle routing: travel times come from real road networks and measured speeds, not synthetic congestion profiles. It also carries a preservation argument: Uber Movement was shut down in 2023, so the published arrival-time matrices can no longer be reproduced from public sources and are the sole surviving artifact of that speed data. This conversion makes the family usable with the MAMUT-routing canonical format, checker and BKS machinery while keeping the original semantics bit-exact.

Upstream: https://gitlab.com/muelleratorunibonnde/vrptdt-benchmark (converted at commit `45f8f57e4d643af5c4e7641e77c2aad197233dc2`), archived at bonndata DOI [10.60507/FK2/X22BKR](https://doi.org/10.60507/FK2/X22BKR). Paper: [Discrete Optimization 53 (2024), 100848](https://doi.org/10.1016/j.disopt.2024.100848).

## The instances

Ten cities (berlin, cincinnati, kyiv, london, madrid, nairobi, new_york, san_francisco, sao_paulo, seattle) at sizes n = 10, 500, 1000 and 2000 customers, named `Blauth-<city>` under `n=<size>/`. Delivery-only TDVRPTW: an unlimited identical uncapacitated fleet (encoded as `num_vehicles: null`, demands all 1, capacity n), depot departures at or after 15:00 with free return, 3 minutes of service per delivery, and hard time windows (50% one-hour windows, 50% wide 15:30 to 21:00; service must begin within the window, waiting at early arrivals). Travel times are dense (n+1)^2 matrices of piecewise-linear arrival-time functions spanning 15:00 to 22:00.

Two deviations from the habits of other MAMUT TD families, both declared in each instance:

- **Time unit is integer milliseconds since midnight** (`metadata.time_unit = "millisecond"`). 91% of the upstream breakpoints are not whole seconds, so any rescale would break bit-exactness; the MAMUT checker is unit-agnostic.
- **No TDVRP twin.** Upstream defines no time-window-free variant and the time windows are intrinsic to the instance generation, so a synthetic twin would misrepresent the family.

The upstream project also publishes a pickup-and-delivery variant of every instance. It is out of the node-routing scope of MAMUT-routing and is not converted here; see the upstream repository.

## Objective: FleetCostDuration

Best-known solutions are published under the `FleetCostDuration` objective: `cost = sum of route durations + F * K` with K the number of routes and `F = fleet_fixed_cost = 36000000 ms` (10 hours) carried by each instance. Route durations are the canonical checker's per-route optimal durations (each route's depot departure time is a decision variable, exactly as in the `Duration` objective). This reproduces the upstream dollar objective ($200 per vehicle plus $20 per working hour) exactly: **cost in $ = cost_ms / 180000**. All breakpoints are integer milliseconds, so checker costs are exact.

BKS sidecars are named `<Name>.bks.FleetCostDuration.json`; costs are always the authoritative output of the canonical checker (`mamut_routing_lib.td.check_td_solution`, mamut-routing-lib >= 0.9.0, exact IEEE-754 arithmetic, no epsilons).

## Hosting: n=10 and n=500 complete, n=1000 and n=2000 without the ATF sidecar

GitHub file-size limits make the big sidecars unhostable (a single n=1000 ATF sidecar exceeds 100 MiB gzipped; n=2000 sidecars reach 386 MiB). The family is therefore hybrid:

- `n=10/` and `n=500/` ship complete: `<Name>.vrp.json` instance, `<Name>.atf.json.gz` canonical ATF sidecar (gzipped), and BKS sidecars.
- `n=1000/` and `n=2000/` ship the `<Name>.vrp.json` instance descriptors (locations, time windows, fleet, metadata, recorded `atf_sha256` pin), BKS sidecars, and the sha256 pins below; **only the oversized ATF sidecars are regenerated locally** with the deterministic converter and verified against the pins.

Note that loading a big-size instance with `mamut_routing_lib.td.load_td_instance` fails with a missing-sidecar error until the ATF sidecar is materialized: run the converter below with `--output-dir` pointing at this repository's checkout (or copy the emitted `<Name>.atf.json.gz` next to the hosted `<Name>.vrp.json`). The emitted `.vrp.json` files are byte-identical to the hosted ones, and the sidecar is verified against the recorded `atf_sha256` on every load.

To materialize the big sizes, clone the upstream repository at the pinned commit and run the converter from [MAMUT-routing-tools](https://github.com/ANR-MAMUT/MAMUT-routing-tools):

```bash
git clone https://gitlab.com/muelleratorunibonnde/vrptdt-benchmark
git -C vrptdt-benchmark checkout 45f8f57e4d643af5c4e7641e77c2aad197233dc2
mamut-tools convert blauth2024 vrptdt-benchmark --output-dir <dir> --sizes 1000,2000
```

Two converter runs produce bit-identical canonical bytes; conversion re-loads every emitted instance with sha verification. Verified pins (`atf_sha256` is the hash of the uncompressed canonical sidecar bytes, recorded in each instance's `td` block):

| instance | size | atf_sha256 |
|---|---|---|
| Blauth-berlin | 1000 | `87584665aae00049ead7e2c6cfa1cd96fc348ee4a4573ce5feb91eb69bca9bc8` |
| Blauth-cincinnati | 1000 | `a33d787a2c64cfd6758c64265298e025c133ce75b294d2c70a17df4855cd324c` |
| Blauth-kyiv | 1000 | `6ef5927c9ae6219601ba31b37db3d396120ca3694868c5b4f2cb9a631bb84861` |
| Blauth-london | 1000 | `7e6a53c509f81c9c3a508ddd3f4ef41adf998b63f18be96fce9675eb2af7f530` |
| Blauth-madrid | 1000 | `e6a218163ca52992b80c72d317c88ad8cfaaa71ef5b7067b7afb5c9ddf12b97b` |
| Blauth-nairobi | 1000 | `6d19df18e07fee33e7240d0f523b3ed4f9963a3584b68bdbb980536b8edc5e37` |
| Blauth-new_york | 1000 | `7fc60cd36b883953f78b2bdc889aa6b2f00be184107fd8538552aca4b14da004` |
| Blauth-san_francisco | 1000 | `42a9d9759dfe1e980ae54695497ab4dc0f5c9944d281891a2f8c7c0be195c167` |
| Blauth-sao_paulo | 1000 | `e6fb03915e85f3b226bba56c7912760a39bfb3c964d54f9d20655776e3ec399c` |
| Blauth-seattle | 1000 | `d2448e0cb544a6d53ad5cea16996eeaf69a365b053e7b2a7ce13caada1c322c8` |
| Blauth-berlin | 2000 | `7ea3fa2bc2d899982f57744b624fda9c0d5e5002ed42a74143b7fbf1fc8bb5a9` |
| Blauth-cincinnati | 2000 | `66d02cad2b09d5b274475cfb3c35275aa126ad0df75a52fadfb0d4f8926f48fe` |
| Blauth-kyiv | 2000 | `855687244efc58448cf540bc2a9e2ea0d49c01ebcc1b960efa44098117d37053` |
| Blauth-london | 2000 | `3e686a2b66b12ebbbe828321c91f1da718658df532753e65dc69acaf6ddac73e` |
| Blauth-madrid | 2000 | `1b36c82cdc29d147dc6dafc7fd28e99ae572932b7f32761237a9b2c88cfd5e05` |
| Blauth-nairobi | 2000 | `60ce865ff70056389de235964d46dd208ca964d63ef6d8f1e0383d5c2df47d87` |
| Blauth-new_york | 2000 | `7040ce56a475c95d2aa468ed1e27a10d6c6f8fc2a47fdb230c9a8af92ce25e5a` |
| Blauth-san_francisco | 2000 | `21cca22b684efd686ab7b992b425c91862afee3529d5f11b1d54125cf8c8ebb9` |
| Blauth-sao_paulo | 2000 | `6b3ca7c2e94ddcd3f9e848ec0f1e73b16258ca5e2aa00ccfa3c95a367e0af27b` |
| Blauth-seattle | 2000 | `58ca0959e055776c7c6ea6b0b61a331aa5b75573f40a414db5a2e5ba1f4d64ff` |

All twenty n=1000/2000 pins are published above; each instance's own `metadata.generator.upstream_files` block additionally pins the sha256 of the exact upstream source files. The berlin and london pins at both sizes were additionally reproduced bit-identically by an independent conversion on a second machine.

## Conversion semantics (value-exact port)

The converter performs a pure integer relabeling of the upstream data; no numeric transformation is applied to any travel-time value. The details, all recorded per instance in `metadata`:

- Upstream items "1".."n" map to customers 1..n, "depot" to node 0; upstream `atf_leave_time`/`atf_arrive_time` breakpoints are carried verbatim per arc. Identity self-arcs are dropped (MAMUT sidecars cover i != j only; a strict gate asserts each dropped self-arc is exactly the identity).
- Arcs between co-located addresses (upstream encodes them as full identities) are kept as zero-travel arcs restricted to the horizon; the items stay distinct customers because each 3-minute service must begin within its own time window (`metadata.co_located_arcs`).
- Some upstream arc domains end before 22:00 (coverage is only guaranteed past 21:33). They are completed to 22:00 at constant travel time, which is provably value-identical: the latest time window ends at 21:00 and service takes 3 minutes, so no feasible departure exists after 21:03 and the completed region is unreachable (`metadata.domain_extended_arcs`; arcs ending before 21:03 are refused).
- The depot due date is midnight (`time_windows[0] = [54000000, 86400000]`): upstream imposes no return deadline, and midnight is provably non-binding because every return arc evaluated exactly (integer rationals) at the 21:03 latest-departure bound arrives strictly before midnight (`metadata.depot_due_date.max_feasible_return_ceil`; family worst: london, 22.53 h).
- The upstream fastest-path distance channel (`atf_dist_end_time`/`atf_distance`, informative meters) is not carried into the sidecars; the upstream source files are sha-pinned per instance for anyone needing it.

## Best-known solutions

- **n=500, 1000, 2000**: imported from the upstream reference solutions (BonnTour high-effort mode, credited to the seven Blauth et al. authors). The upstream per-tour `start_time` is dropped and each route's depot departure is re-optimized by the checker, so the imported costs are less than or equal to the published dollar values; the published Table 2 values of the paper are exactly the ceilings of the imported costs in dollars. Every imported solution passed a hard-time-window re-evaluation with the upstream evaluator's exact rational arithmetic before import.
- **n=10**: upstream publishes no solutions at this size, so these entries are the first published results for the tier.

See `CHANGELOG.md` for the BKS history.
