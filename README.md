# Poryos2026 — the MAMUT-routing city benchmark collection

Poryos2026 is one benchmark family with four problem-type variants (CVRP, VRPTW, TDVRP, TDVRPTW) generated from real OpenStreetMap city road networks. Every variant of a base instance shares the same customers, coordinates, demands and vehicle capacity, so the family supports controlled paired comparisons: CVRP vs TDVRP isolates the cost of ignoring time, VRPTW vs TDVRPTW the cost of time dependence, and light vs heavy traffic the marginal cost of congestion on identical time windows.

Grid: 5 cities (Lyon, Paris, San-Francisco, Hong-Kong, Tokyo) × n ∈ {10, 25, 50, 100, 500, 1000} customers × 2 sampling methods (`poi`: OSM points of interest; `hyb`: hybrid POI + parametric) = 60 base instances.

| Problem type | Instances | Axes |
|---|---|---|
| CVRP | 180 | 3 arc-cost metrics (euclidean, shortest, fastest) |
| VRPTW | 180 | fastest metric, 3 time-window sets (`td-shared`, `tight`, `spread`) |
| TDVRP | 360 | 6 traffic subinstances (bpr/wave × light/moderate/heavy) |
| TDVRPTW | 360 | 6 traffic subinstances, windows shared with the `td-shared` VRPTW set |

## Naming

Base name: `mamut-<city>-n<N>-<method>` (e.g. `poryos-lyon-n10-hyb`). CVRP and the TD-paired VRPTW instance carry the base name verbatim (the metric or TW set is distinguished by path). TD subinstances append `-<model>-<intensity>` (e.g. `poryos-lyon-n10-hyb-bpr-heavy`). Static-only VRPTW time-window sets append `-tw-<set>` (e.g. `poryos-lyon-n10-hyb-tw-tight`); the reserved `tw-` tag marks by name that the instance is not TD-paired.

## Layout

```text
mamut-collection.json                 collection marker (family, layout version)
sidecars/<city>/n=<N>/<base>/
    <base>.geo.json.gz                geo sidecar (WGS84 nodes, reference frame, road cache for plotting; complete node-pair cache for n <= 100)
    <base>.road.json.gz               road graph (static trimmed subgraph: edges with length and free-flow speed limit, vertex coordinates)
    <base>.traffic-<model>-<intensity>.json.gz   6 per-edge 24-bin speed overlays per base
    <base>.distances-fastest.json.gz  free-flow fastest travel-time matrix (seconds)
    <base>.distances-shortest.json.gz shortest-path distance matrix (meters)
CVRP/<metric>/<city>/n=<N>/<base>/    metric in {euclidean, shortest, fastest}
    <base>.vrp.json                   canonical instance (arc costs by sidecar reference or euclidean rule)
    <base>.vrp                        CVRPLIB explicit-matrix export, committed for n <= 100 only
VRPTW/fastest/<city>/n=<N>/<base>/
    <base>.vrp.json                   td-shared TW set (the TDVRPTW windows)
    <base>-tw-tight.vrp.json          static-only tight TW set
    <base>-tw-spread.vrp.json         static-only spread TW set
TDVRP/<city>/n=<N>/<base>/<model>-<intensity>/<base>-<model>-<intensity>.vrp.json
TDVRPTW/<city>/n=<N>/<base>/<model>-<intensity>/<base>-<model>-<intensity>.vrp.json
```

Instances reference their shared sidecars by collection-root-relative path with a pinned sha256; the marker file makes the root discoverable by walking up from any instance. A standalone clone of this repository is fully self-contained.

## Value conventions: why float arc costs

Arc costs and distance matrices are 3-decimal floats (seconds for travel times, meters for shortest distances). This is a deliberate departure from the CVRPLIB integer tradition, and it buys a single cost model across all four problem types: the time-dependent travel times are inherently real-valued, and rounding the static variants to integers would have created a second, coarser cost model next to the float TD world, confounding exactly the paired CVRP-vs-TDVRP and VRPTW-vs-TDVRPTW comparisons the family is built for. Solvers that require integer data can scale all costs (and time windows, service times, and the horizon) by 1000: 3-decimal values are exact in binary64, so the scaling is lossless and 100 % reproducible. Time windows and service times are integer seconds; demands and capacities are integers.

## Time windows: three sets, one pairing rule

Service times are a base property (one set per base, shared by every VRPTW and TDVRPTW instance). Each base carries three time-window sets over those service times:

- `td-shared` (file `<base>.vrp.json`): route-centered windows over the free-flow fastest matrix, anchored on deterministic capacity-and-horizon-feasible routes and globally certified under all 6 traffic overlays (`metadata.tw_anchor` and `metadata.tw_repair`). **These are exactly the windows of the base's 6 TDVRPTW subinstances.**
- `tight` (`<base>-tw-tight.vrp.json`): same route-centered window centers, much narrower widths (mean about 1.2 h vs about 4.8 h). A controlled width comparison on identical demand geometry.
- `spread` (`<base>-tw-spread.vrp.json`): window centers drawn uniformly over each customer's feasible interval, td-shared widths. Breaks the route structure of the placement.

**Pairing rule**: only the bare-base-named VRPTW instance shares its windows with the TDVRPTW twins. The `tight` and `spread` sets are static-only: they are never audited under traffic, and results on them must not be compared against time-dependent results. Every VRPTW instance states its set and pairing status in `metadata.tw_set`.

Every client time window has strictly positive width. Route-centered sets ship a deterministic feasible anchor solution: routes start independently at time zero, append the nearest capacity-and-horizon-feasible unserved customer (node-index tie-break), and cover every customer exactly once. For `td-shared`, shared deadline lifts are the only permitted traffic repair; earliest bounds are never reduced, and the complete anchor solution is checked under every overlay.

## Capacity policy

Poryos2026 instances are genuine vehicle-routing instances by construction. The generator targets 3–5 customers per route at `n=10`, 5–8 at `n=25`, and 12–16 from `n=50` upward. Every published base passes the hard gate `LB_cap = ceil(sum(q_i) / Q) >= 2`; the exact lower bound is recorded as `metadata.num_vehicles_lb`.

## Time-dependent model

TD instances use the `road-graph` model: the instance pins, per customer pair, the free-flow fastest path through the trimmed city road graph (`<base>.road.json.gz`), and the traffic overlay (`<base>.traffic-<model>-<intensity>.json.gz`) drives time-varying speeds over those pinned paths. Arrival-time functions are materialized deterministically at load time and verified against the pinned `atf_sha256`. Both traffic models (`bpr`, `wave`) are clamped at each edge's free-flow speed: traffic only ever slows you down, and the horizon's night bins are free-flow, which makes the TD variants exactly comparable to their static twins. The 24 h horizon is `[0, 86400]` seconds.

## Loading

The reference loader is [mamut-routing-lib](https://github.com/ANR-MAMUT/MAMUT-routing-lib) (Python, >= 0.6.0):

```python
from pathlib import Path
from mamut_routing_lib.artifacts import discover_benchmark_instances, load_benchmark_instance, resolve_arc_costs
from mamut_routing_lib.td import load_td_instance

items = discover_benchmark_instances(Path("path/to/Poryos2026"))
static = load_benchmark_instance(items[0].instance_path)      # CVRP / VRPTW
matrix = resolve_arc_costs(static, items[0].instance_path)    # hydrate from sidecar
td = load_td_instance("TDVRPTW/lyon/n=10/poryos-lyon-n10-hyb/bpr-heavy/poryos-lyon-n10-hyb-bpr-heavy.vrp.json")
```

CVRPLIB `.vrp` exports with explicit matrices are committed for n <= 100; larger sizes can be exported with the MAMUT-routing workbench CLI (about 8 MB of text per instance at n = 1000).

## Best known solutions

BKS files sit next to their instance as `<instance>.bks.<objective>.json` (`MonoCost` for CVRP/VRPTW, `Duration` for TD), are checker-validated before storage, and only ever improve. Contributions are welcome through pull requests with reproducible solution files.

## Provenance and reproducibility

The collection is generated by the MAMUT-routing workbench pipeline (version 3): OSM extracts are sampled per city, the road graph is trimmed on static free-flow weights, and every published byte is canonical JSON written by a single deterministic Python serializer, seeded from instance names. Published artifacts are internally consistent and pinned by sha256; loading is deterministic across machines. Regenerating from raw OSM requires the pinned toolchain and may differ in synthetic OSM node identifiers across machines; the published bytes are the canonical reference.

## License

Benchmark data: ODbL 1.0 (OpenStreetMap-derived; see LICENSE). Data from OpenStreetMap and its contributors, https://www.openstreetmap.org/copyright.
