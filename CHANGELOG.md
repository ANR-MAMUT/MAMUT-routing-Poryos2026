# Changelog — Poryos2026 collection

## 2026-11-26. Improved Duration records

- Florian Rascoussier improved the total route duration for 58 TDVRPTW instances. The Kayros development build produced 57 solutions. Kayros 1.7.1 produced the solution for Lyon n=100 POI, BPR-light. [Kayros development revision `dba5688`](https://github.com/0nyr/kayros/commit/dba56886167c854d2ae36b627128d6d419f811f4) · [Kayros 1.7.1](https://github.com/0nyr/kayros/commit/2c9f4aedab10bf64365035c300f896ae9e597244)
- Replaying those routes without time windows on the corresponding TDVRP instances also improved 23 TDVRP incumbents. The native and Python Fraction evaluators independently checked the exact durations and the earliest minimizing departure for each route under the edition `r1` profile, `td-binary64-exact-v1`.

## 2026-11-25. Optimality claims withdrawn from edition r1

- The 238 solver-reported optimality claims (`metadata.optimality`) of this family's best-known solutions are withdrawn, 115 TDVRP and 123 TDVRPTW. They targeted the historical `td-fold/1` evaluation of earlier Kayros versions, not the `td-binary64-exact-v1` profile of edition `r1`. New claims will be derived with Kayros 2.0 under `td-binary64-exact-v1`.
- The 238 content-addressed evidence files under `.mamut/history/` that only these claims referenced are removed. The archive of the original pre-migration best-known solutions, indexed by `.mamut/archives/r1-original-bks.json`, is unchanged.
- No route, cost, route witness, manifest or instance file changed.

## 2026-11-24. Hong Kong 25-customer CVRP improvement on v013

The `fastest` CVRP incumbent for `poryos-hong_kong-n25-hyb` now uses the four routes from its time-window variant. On the CVRP edition `r1`, the C++ and Python Fraction backends both found a strict reduction of $5/2^{49}$ seconds under `binary64-exact-v1`. Both costs display as 9150.837 seconds. The new BKS records the source routes and the exact evaluation on the CVRP manifest. [CVRP incumbent](CVRP/fastest/hong_kong/n=25/poryos-hong_kong-n25-hyb/poryos-hong_kong-n25-hyb.bks.MonoCost.json) and [source VRPTW routes](VRPTW/fastest/hong_kong/n=25/poryos-hong_kong-n25-hyb/poryos-hong_kong-n25-hyb.bks.MonoCost.json).

## 2026-11-04: community BKS improvement

Accepted [vizzito's contribution](https://github.com/ANR-MAMUT/MAMUT-routing-Poryos2026/issues/2) for `poryos-lyon-n1000-hyb`, CVRP/fastest: exact cost improves from 60163.837 to 60120.025 seconds (43.812 seconds), with 64 routes. Re-validated the candidate and incumbent with mamut-routing-lib 0.12.0 against the sha256-pinned fastest-distance sidecar. An independent Decimal sum of the raw sidecar entries confirms both costs, and independent coverage, capacity and fleet checks pass. Stored the candidate through the canonical improve-only BKS store. The remaining 1079 BKS are unchanged. [Verification record](contributions/2026-11-04-issue-2.json).

## 2026-09-24 — TD BKS re-priced under the td-fold/2 checker contract; relaxation-twin cross-evaluation

- **Re-pricing, no route changed.** mamut-routing-lib 0.12.0 replaces the TD checker's route fold (contract
  `td-fold/1` -> `td-fold/2`): waiting and service at a vertex are applied exactly to the accumulated arrival times
  instead of being composed through a ratio interpolation, and slope-one travel is computed by addition. Every TD
  BKS was re-priced with `mamut-routing bks reprice-td`: TDVRPTW 311 of 360 files rewritten (93 cost moves), TDVRP
  286 of 360 (108 cost moves), all float rounding (at most 4.1e-16 relative); the others only refreshed
  `route_durations` / `route_departure_times` by ulps. Each BKS whose cost moved records `metadata.repriced`
  (previous cost, checker, contract, date). A second pass reports every file unchanged.
- **Optimality stamps.** 48 stamped BKS (29 TDVRPTW, 19 TDVRP) moved by at most 7.3e-12: `proven_optimum` now equals
  the re-priced cost, and a `note` records that the proof was obtained under `td-fold/1`, that `dual_bound` is the
  prover's value in that arithmetic, and that re-certification under `td-fold/2` is pending.
- **Relaxation-twin cross-evaluation: 41 TDVRP BKS and 1 CVRP BKS improved.** A TDVRPTW solution is a feasible
  TDVRP solution of no larger duration (the TDVRP twin drops the windows; the arrival-time functions are FIFO),
  and a VRPTW solution of any TW set is a CVRP solution of the same cost on the same metric. Every TDVRPTW BKS was
  therefore priced on its TDVRP twin, and every VRPTW BKS on its CVRP twin, and offered through the improve-only
  stores (`save_td_solution_as_bks_if_improved`, `save_solution_as_bks_if_improved`; exact comparison, checker
  cost authoritative) after a structural check that the pair really is a relaxation. 41 TDVRP BKS improved (2 at
  n=50, 2 at n=100, 15 at n=500, 22 at n=1000; mean -0.48 %, median -0.18 %, largest -1.74 % on
  `poryos-hong_kong-n500-poi-wave-heavy`), and CVRP/fastest `poryos-hong_kong-n500-poi` improved from 51998.37 to
  51865.004 (-0.26 %) with the routes of its VRPTW twin. No optimality-stamped BKS was affected. The new records keep
  the twin's authors and carry `method: twin-cross-evaluation`, a `derived_from` block naming the source instance,
  objective, cost and date, and `campaign: 2026-09 relaxation-twin cross-evaluation`. The decimal tie
  `poryos-hong_kong-n25-hyb` (CVRP) was kept.

## 2026-08-08 — first optimality certificates: 238 TD instances proven optimal, 4 BKS improved

- First exact-solver campaign over the TDVRP and TDVRPTW families at n <= 100 (480 instances; Grid'5000 abaca, jobs 6832834, 6832960, 6835714 and 6837251). 238 instances now carry an optimality certificate in their BKS metadata (`metadata.optimality`): TDVRPTW 123 (60 at n=10, 57 at n=25, 6 at n=50) and TDVRP 115 (59 at n=10, 56 at n=25). These are the first proven optima in the collection's TD families.
- Prover: kayros 1.5.1 lera branch-price-and-cut (HiGHS backend). Every certificate required four independent solves (cold and warm starts × asymmetric and symmetric labeling modes) agreeing on one checker-exact value, each with an audited exact-pricing phase; the reference `mamut-routing-lib` td Duration checker re-validated every stamped solution and value.
- 4 TDVRPTW best known solutions improved to their certified optima (improve-only store discipline, checker re-priced costs authoritative): `poryos-san_francisco-n25-poi-bpr-heavy` (-0.256 percent), `poryos-tokyo-n50-hyb-bpr-heavy` (-0.206 percent), `poryos-tokyo-n50-hyb-bpr-moderate` (-0.281 percent) and `poryos-tokyo-n50-hyb-wave-heavy` (-0.181 percent). On the other 234 certified instances the stored heuristic BKS already equalled the exact optimum.
- The remaining 242 TD instances at n <= 100 keep their heuristic BKS and stay open: 199 exceeded the time budget (up to 7200 s per solve), 27 exceeded the memory budget, and on 16 the certification audit declined to stamp because at least one solve concluded without a fully audited exact-pricing trail; their stored solutions are unchanged. TD instances at n >= 500 and all static families were out of scope.

- Improved 19 TDVRPTW best known solutions (12 at `n=100`, 6 at `n=500`, 1 at `n=1000`), found during a controlled comparison of several time-dependent VRPTW solvers at a one-hour budget per run. Improvements range from 0.512 percent down to 0.0001 percent, mean 0.069 percent; the largest is `poryos-hong_kong-n500-poi-bpr-heavy` at -0.512 percent.
- Contributors: kayros TD-ILS (versions 1.3.0 and 1.5.0) supplied 18 of them and Timefold Solver 2.3.0 supplied one (`poryos-lyon-n100-hyb-wave-light`). Each stored solution names the solver, version, seed and time limit that produced it.
- Every solution was re-validated with the reference `mamut-routing-lib` time-dependent Duration checker before storage, and written under the improve-only store discipline: the checker's re-priced cost is authoritative, and a candidate is stored only if it is strictly better than the entry it replaces.
- The remaining 1061 instances are unchanged. Coverage stays complete: every instance carries a checker-validated BKS.

## 2026-07-22: benchmark-family identity renamed from Mamut2026 to Poryos2026

- Renamed the family, repository, submodule path, metadata identity, and generated instance prefix from `Mamut2026`/`mamut-<city>` to `Poryos2026`/`poryos-<city>` so the dataset is clearly identified as Florian Rascoussier's benchmark family rather than a collective MAMUT project benchmark.
- Re-pinned every affected canonical sidecar and arrival-time-function hash after the identity change. All 1,080 benchmark instances retain their data, problem variants, and complete checker-validated BKS coverage.
- Updated the reference library, generation tools, publisher, tests, and documentation to use the Poryos2026 identity. Project-level names and artifact-format identifiers such as `MAMUT-routing`, `mamut-routing-lib`, and `mamut-collection.json` remain unchanged by design.

## 2026-07-17 — BKS re-seeding for the regenerated instances (coverage 1080/1080)

- Recomputed the best known solutions of all 680 regenerated instances; the 400 byte-identical instances keep their existing BKS. Coverage is complete again: every instance carries a checker-validated BKS (`MonoCost` for CVRP/VRPTW, `Duration` for TDVRP/TDVRPTW).
- Campaign: PyVRP HGS for the static instances and kayros TD-ILS for the TD instances, seeds {42, 123, 456, 1729, 2026, 31416} at the established per-size time limits (120 s at n=10 up to 3600 s at n=1000), one fast full-coverage pass (seed 7, limits capped at 900 s), and a partial doubled-limit pass (seed 9001) on the n >= 500 instances. Best of runs per instance, re-validated by the reference checker before storage (improve-only store discipline).
- The regenerated small instances now have genuinely multi-route optima: every n=10 BKS uses 3 to 4 routes and every n=25 BKS at least 4, in line with the restored `LB_cap` bounds.

## 2026-07-16 — instance-generation repair (pipeline v3)

- Replaced the fixed 12–16-customer route-size setting with targets of 3–5 customers at `n=10`, 5–8 at `n=25`, and 12–16 from `n=50` upward. Every base now passes the hard `LB_cap >= 2` publication gate; the regenerated `n=10` bases have `LB_cap` 3–4 and `n=25` bases 4–5.
- Replaced the single giant nearest-neighbour TW anchor tour with deterministic capacity-and-horizon-feasible multi-route anchors, applied consistently to every `td-shared` and `tight` set.
- Replaced individual singleton serveability repair with a global anchor certificate under all six traffic overlays. Repairs only lift shared deadlines; earliest bounds are never reduced.
- Added hard publication gates for strictly positive client-window widths, paired-base demand/capacity consistency, anchor feasibility, and `LB_cap`.
- Regenerated exactly 680 affected instances while retaining 400 byte-identical instances. All 540 TW-bearing instances now contain zero zero-width windows.

## 2026-07-10 — initial BKS seeding (coverage 1080/1080)

- Every instance now carries a checker-validated best known solution: `MonoCost` for the 360 static CVRP and VRPTW instances (PyVRP HGS on the documented 1000-scaled integer twin, re-priced in the canonical float domain), `Duration` for the 720 TDVRP and TDVRPTW instances (kayros TD-ILS on the materialized arrival-time functions).
- Seeding campaign: seeds {42, 123, 456}, per-size time limits from 120 s (n=10) to 3600 s (n=1000); each stored solution is the best of its runs, re-validated by the reference checker before storage (improve-only store discipline).

## 2026-07-09 — initial release (collection layout v1, pipeline v2)

- 60 base instances: 5 cities × n ∈ {10, 25, 50, 100, 500, 1000} × {poi, hyb}.
- 1080 instances: CVRP 180 (3 metrics), VRPTW 180 (3 TW sets over the fastest metric), TDVRP 360, TDVRPTW 360.
- Shared sidecars per base: geo, road graph, 6 traffic overlays, fastest and shortest distance matrices; all sha256-pinned, referenced collection-root-relative.
- Value conventions: 3-decimal float arc costs family-wide (see README), integer-second time windows and service times.
- VRPTW TW sets: `td-shared` (paired with the TDVRPTW subinstances, traffic-audited with minimal shared repair), `tight` and `spread` (static-only).
- CVRPLIB `.vrp` exports committed for n <= 100.
