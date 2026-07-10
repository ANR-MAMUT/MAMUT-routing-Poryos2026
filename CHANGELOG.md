# Changelog — Mamut2026 collection

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
