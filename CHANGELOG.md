# Changelog — Poryos2026 collection

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
