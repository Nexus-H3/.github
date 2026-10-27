<div align="center">

<img src="[assets/nexus_h3_logo.jpg](https://github.com/user-attachments/assets/284380e8-ac2d-4d22-9f67-56ab45ea6cdf)" alt="Nexus-H3 — Urban Mobility Observatory" width="520"/>

# NEXUS-H3

**A Resource-Frugal Urban Mobility Observatory for Continuous Telemetry Preservation and Reproducible Benchmarking**

*Continuously preserving Grand Paris' ephemeral mobility telemetry as an open, machine-learning-ready benchmark — for less than €5 a month.*

[![Cadence](https://img.shields.io/badge/ingestion-3%20min%20cadence-ff69b4)](#-the-benchmark) [![Spatial Index](https://img.shields.io/badge/spatial%20index-Uber%20H3%20Res--9-blue)](#-the-benchmark) [![Cells](https://img.shields.io/badge/benchmark%20cells-8%2C277-9cf)](#-the-benchmark) [![Features](https://img.shields.io/badge/schema-37%20features-success)](#-the-benchmark) [![Cost](https://img.shields.io/badge/operating%20cost-%E2%82%AC5%2Fmonth-brightgreen)](#-cost) [![Workbench](https://img.shields.io/badge/live%20workbench-nexus--3.vercel.app-8A2BE2)](https://nexus-h3.vercel.app)

</div>

---

> **TL;DR** — Real-time mobility APIs are *ephemeral*: IDFM transit delays (~30,000 departures every 3 minutes) and Vélib' station availability are broadcast, then discarded forever. **Nexus-H3** is an open observatory that polls, harmonizes, and archives these streams onto an Uber H3 Resolution-9 hexagonal grid across Grand Paris — and publishes them as a reproducible spatiotemporal benchmark (8,277 cells × 37 features, 3-minute cadence, ~5.7M observations/day). It runs 24/7 on a 1 GB RAM ARM64 virtual machine, with heavy compilation offloaded to *your* browser (DuckDB-Wasm) and *your* free Google Colab runtime. No enterprise cloud. No egress fees. No PII.

---

## 📡 Why Nexus-H3?

Modern metropolitan mobility is a tightly coupled multi-modal network — road traffic, transit reliability, shared micromobility, and weather shocks interacting at sub-kilometer, minute-level scales. Studying it hits three structural barriers:

| Barrier | Reality | Nexus-H3's answer |
|:---|:---|:---|
| **The Ephemeral API Problem** | IDFM's SIRI Lite departures and Vélib' GBFS availability are **never archived publicly**. If nobody is polling, the data cannot be reconstructed — ever. | A fault-tolerant observatory polls and preserves every 3 minutes, continuously. |
| **Benchmark Stagnation** | METR-LA, PeMS-BAY, NYC Taxi, and CitiBike are static, single-modality snapshots — no transit, no morphology, no 5G, no weather. | A **continuously growing** multi-modal benchmark fusing 7 dynamic streams with 24 static infrastructure layers. |
| **The Cloud Cost Wall** | City-scale 24/7 ingestion generates hundreds of millions of records per month; cloud warehouses make long-term hosting cost-prohibitive. | **Frugal disaggregation**: a <€5/month VM + free-tier services + client-side compute. |

## 🏙️ The Benchmark

<img src="https://github.com/user-attachments/assets/e12d5069-4524-46e2-8fcd-16fdfca4a504" alt="Grand Paris multi-modal benchmark on the H3 hexagonal grid" width="100%"/>

<div align="center">

| Statistic | Value |
|:---|:---|
| Spatial tessellation | Uber H3 Resolution-9 (~0.105 km² per cell) |
| Benchmark extent | **8,277** contiguous Grand Paris H3 cells (Depts. 75, 92, 93, 94) |
| Compiled schema | **37 columns** (6 spatiotemporal keys + 7 dynamic + 24 static) |
| Native cadence | **3 minutes** (480 observation windows per day) |
| Daily volume | **~5.7 million** multi-modal observations |
| Coordinate systems | WGS84 (geodetic) + Lambert-93 (metric) |
| Regime boundary | Mode B / Mode A split at **2026-10-17T00:00Z** |

</div>

> ℹ️ The 7,678-cell figure shown in the illustration refers to cells carrying OSM morphology coverage in the `complete_h3_grid` export; the full Grand Paris grid spans **8,277** cells.

**Layer 1 — 7 dynamic mobility & meteorological features** (live, 3-minute windows): `idfm_active_departures`, `idfm_mean_delay_sec`, `velib_mechanical_bikes`, `velib_ebikes`, `velib_available_docks`, `temperature_2m`, `precipitation`.

**Layer 0 — 24 static infrastructure & morphology features**: 18 OSM-derived 3D urban morphology features (building counts, footprints, heights, levels, land use, green/water area, road/cycleway/rail lengths, POI densities) and 6 critical-infrastructure features (transit stop density & modal diversity, 4G/5G antenna counts & operators, monitored road-loop arcs).

<details>
<summary><b>📋 Full 37-feature data dictionary (click to expand)</b></summary>

| # | Column | Layer | Description |
|---:|:---|:---|:---|
| 1 | `window_start` | Key | UTC observation window start (3m/15m/1h) |
| 2 | `h3_index_r9` | Key | Uber H3 Res-9 cell index |
| 3 | `cell_lat` / 4 `cell_lon` | Key | WGS84 cell centroid (EPSG:4326) |
| 5 | `x_193` / 6 `y_193` | Key | Lambert-93 metric coordinates (EPSG:2154) |
| 7 | `velib_mechanical_bikes` | L1 — Vélib' GBFS | Mechanical bikes available (NULL in Mode B) |
| 8 | `velib_ebikes` | L1 — Vélib' GBFS | E-bikes available (NULL in Mode B) |
| 9 | `velib_available_docks` | L1 — Vélib' GBFS | Empty return docks (NULL in Mode B) |
| 10 | `idfm_active_departures` | L1 — IDFM SIRI | Active transit departures (NULL in Mode B) |
| 11 | `idfm_mean_delay_sec` | L1 — IDFM SIRI | Mean departure delay in seconds (NULL in Mode B) |
| 12 | `temperature_2m` | L1 — Open-Meteo | Ambient temperature at 2 m (°C) |
| 13 | `precipitation` | L1 — Open-Meteo | Liquid-equivalent precipitation (mm) |
| 14 | `transit_stop_count` | L0 — IDFM GTFS | Physical transit stop points in cell |
| 15 | `distinct_transit_modes` | L0 — IDFM GTFS | Distinct modes (Metro/RER/Tram/Bus) |
| 16 | `cellular_antenna_count` | L0 — ANFR | Active cellular antennas in cell |
| 17 | `antenna_5g_count` | L0 — ANFR | Active 5G NR antennas in cell |
| 18 | `distinct_telecom_operators` | L0 — ANFR | Distinct mobile operators (max 4) |
| 19 | `monitored_road_arc_count` | L0 — Paris Data | Permanent inductive-loop counting arcs |
| 20 | `building_count` | L0 — OSM 3D | Buildings with centroid in cell |
| 21 | `total_building_footprint_sqm` | L0 — OSM 3D | Cumulative building footprint (m²) |
| 22 | `avg_building_height_m` | L0 — OSM 3D | Mean building height (m) |
| 23 | `avg_building_levels` | L0 — OSM 3D | Mean above-ground floor count |
| 24 | `dominant_building_type` | L0 — OSM 3D | Modal building typology |
| 25 | `green_and_field_area_sqm` | L0 — OSM Landuse | Parks, forest, grass, vegetation (m²) |
| 26 | `commercial_industrial_area_sqm` | L0 — OSM Landuse | Commercial/retail/industrial land use (m²) |
| 27 | `water_area_sqm` | L0 — OSM Landuse | River, canal, basin surface water (m²) |
| 28 | `dominant_landuse` | L0 — OSM Landuse | Primary land-use classification |
| 29 | `total_road_length_m` | L0 — OSM Network | Highway/street centerline length (m) |
| 30 | `cycleway_length_m` | L0 — OSM Network | Dedicated cycleway length (m) |
| 31 | `rail_transit_length_m` | L0 — OSM Network | Heavy rail/metro/tram track length (m) |
| 32 | `pipeline_and_power_length_m` | L0 — OSM Network | Power/utility corridor length (m) |
| 33 | `total_poi_count` | L0 — OSM Amenity | Amenities, shops, offices, tourism |
| 34 | `market_and_retail_count` | L0 — OSM Amenity | Supermarkets, markets, retail |
| 35 | `food_and_beverage_count` | L0 — OSM Amenity | Restaurants, cafés, bakeries, bars |
| 36 | `education_and_health_count` | L0 — OSM Amenity | Schools, universities, hospitals, pharmacies |
| 37 | `utility_and_infrastructure_nodes` | L0 — OSM Amenity | Substations, telecom towers, water works |

</details>

## 🏗️ Architecture — Frugal Disaggregation

<img src="https://github.com/user-attachments/assets/80b3cc4e-9e37-4e0c-a38e-c9d06b7c3c35" alt="Nexus-H3 resource-frugal disaggregated architecture" width="100%"/>

Coordination, storage, ingestion, and analytical compilation are deliberately split across low-cost and resource-bounded tiers:

| Tier | Technology | Envelope |
|:---|:---|:---|
| **Edge ingestion** | ARM64 VM (Azure B2pts_v2, 2 vCPU) polling IDFM, Vélib', Open-Meteo every 3 min | **1 GB RAM** + 4 GB swap |
| **Processing** | Polars (vectorized transforms) + DuckDB Spatial (H3 r9 projection, CRS reprojection) | ~200 MB worker envelope |
| **Control plane** | FastAPI + serverless Neon PostgreSQL (leases, lineage, metrics) | Atomic, duplicate-safe by construction |
| **Object storage** | Google Drive (OAuth-managed), ZSTD Parquet, 24-slice daily compaction | Zero egress fees |
| **Edge protection** | nginx: TLS, 5 req/s per IP, 20 max connections, 5 MB payload cap | — |
| **Compiler — Tier 1** | DuckDB-Wasm in the researcher's browser (HTTP range-request Parquet streaming) | Interactive 5-row sampling |
| **Compiler — Tier 2** | Native DuckDB server compiler, one export at a time | 512 MB / 2 threads, disk spill |
| **Compiler — Tier 3** | Deterministic script synthesis on ephemeral Google Colab runtimes | 10 GB RAM, burn-on-claim keys |

### Duplicate-safe ingestion

Every 3-minute window is a PostgreSQL lease (`batch_leases`) acquired through an idempotent `FOR UPDATE SKIP LOCKED` claim predicate: PENDING → PROCESSING → COMPLETED, with TTL expiry safely returning crashed workers' locks to the pool. An explicit **EMPTY** lease status distinguishes verified upstream pauses from failures — *no data is never confused with error*. The `parquet_lineage` catalog records SHA-256, row count, size, and window bounds for every file, so researchers can cryptographically verify any download against the observatory's ledger.

### Constant-memory streaming extraction

The 68 MB IDFM SIRI Lite payload (30,000 departures) is parsed with streaming `ijson` from disk after a 45-second asynchronous prefetch — bounding peak heap to a fraction of a megabyte instead of ~493 MB for naive `json.loads()`, and cutting in-window latency from over 160 seconds to ~3.3 seconds.

### ⏳ Temporal regime separation

Nexus-H3 deliberately preserves a real-world **modality shift** instead of hiding it:

- **Mode B (historical backfill, before 2026-10-17T00:00Z)** — sparse road-loop archives; Vélib' and IDFM delay features are structurally `NULL`, never imputed.
- **Mode A (live multi-modal, from 2026-10-17T00:00Z)** — rich, high-frequency telemetry.

The two regimes are strictly disjoint, and cross-epoch exports are blocked in both the web UI and the worker. The dataset is thus a clean empirical foundation for **cross-regime spatiotemporal transfer learning** — the regimes are explicitly *not poolable*, and that is a design principle, not a defect.

## 📊 How It Compares

| Dataset | Continuous Updates | Multi-Modal | Weather Fused | 3D Morphology | Infrastructure | Spatial Indexing | Public Access |
|:---|:---:|:---:|:---:|:---:|:---:|:---|:---:|
| METR-LA | ✗ | ✗ | ✗ | ✗ | ✗ | Point / Graph | ✓ |
| PeMS-BAY | ✗ | ✗ | ✗ | ✗ | ✗ | Point / Graph | ✓ |
| NYC Taxi | ✗ | ✗ | ✗ | ✗ | ✗ | Custom zones | ✓ |
| CitiBike (NYC) | ✗ | ✗ | ✗ | ✗ | ✗ | Lat/Lon points | ✓ |
| **Nexus-H3** | **✓ 3-min telemetry** | **✓ transit · bikes · traffic** | **✓ Open-Meteo** | **✓ OSM buildings** | **✓ stops · 5G** | **Uber H3 (Res-9)** | **✓ zero-egress** |

## 📈 Measured Results

<div align="center">

| Evaluation | Result |
|:---|:---|
| Traced heap during SIRI Lite parsing (82 MB payload) | **642.7× reduction** (493.23 MB → 0.77 MB) |
| In-window ingestion latency (critical path) | **160–250 s → ~3.3 s** |
| Storage: key-sorted ZSTD compaction (3.89M rows) | **40.9% smaller** (287.46 MB → 170.02 MB) |
| Daily commit time via 24-slice batch completion | **96 s → 13 s** (95.8% fewer Drive/Postgres transactions) |
| Tier 3 vs Tier 2 compiled file size | **4× smaller** (30.0 MB → 7.6 MB, same 731,772 rows) |
| Initial 7-day live deployment uptime | **95.65%** (3,214 / 3,360 windows) |

</div>

### 💶 Cost

| Architecture (1 TB telemetry workload) | Monthly cost |
|:---|:---:|
| **Nexus-H3** — Azure micro-VM + free ecosystem | **< €5.00** |
| GCP BigQuery (storage + on-demand query) | ~€95.00 |
| AWS S3 + Snowflake compute | ~€140.00 |

### 🤖 Forecasting baselines (30-min bike-availability, 8,277-cell graph)

| Baseline | MAE ↓ | RMSE ↓ | MAPE ↓ |
|:---|---:|---:|---:|
| Historical Average | 6.607 | 9.629 | 91.36% |
| Persistence (LOCF) | 2.505 | 5.776 | 22.53% |
| LightGBM (dynamic only) | 2.151 | 4.487 | 20.78% |
| Temporal GRU | 2.153 | 4.549 | 20.96% |
| LightGBM + static morphology + H3 spatial lags | 2.147 | **4.477** | 21.06% |
| ST-GNN (H3 1-ring graph + static) | **2.144** | 4.541 | 20.91% |

All learned models cut MAE by up to **14.4%** and RMSE by **22.5%** vs. persistence. Zero-shot transfer (trained on Paris Intra-Muros, 814 cells → evaluated on suburbs, 369 cells) reduces RMSE by **38.8%** vs. the persistence baseline, with static morphology improving MAPE (15.68% vs. 15.94%). Full protocol: `notebooks/section8_benchmarks.ipynb`, fixed seed (SEED = 42).

## 🚀 Getting the Data

The [interactive workbench](https://nexus-h3.vercel.app) compiles custom slices of the benchmark — no infrastructure required:

1. **Configure** — pick regime (Mode A live / Mode B backfill), cadence (3m / 15m / 1h), spatial scope (Grand Paris / Intra-Muros), join topology, null policy, and output format (Parquet / CSV).
2. **Preview** — DuckDB-Wasm streams Parquet footers straight into your browser for instant schema inspection and 5-row samples.
3. **Compile large exports** — "Download with Colab" mints a burn-on-claim session (job ID + ephemeral secret) and synthesizes a deterministic Python script; paste it into a free Colab runtime (10 GB RAM), which streams the authorized partitions, verifies SHA-256 checksums against the lineage ledger, and exports ZSTD-compressed Parquet to your browser.
4. **Reproduce** — run the companion evaluation notebook on the exported artifact to regenerate the forecasting and transfer-learning tables.

## 📁 Repository Layout

```text
nexus-h3-observatory/
├── control-plane/       # FastAPI: lease state machine, lineage catalog, compiler, Colab brokering
├── ingestion-engine/    # Polars/DuckDB workers: live 3-min loop, Kafka backfill, static topology
├── web-portal/          # Vite + React SPA: dataset generator workbench & admin console
├── notebooks/           # Reproducible benchmark & systems evaluations (SEED = 42)
├── reverse-proxy/       # nginx edge configuration (TLS, rate limits)<img width="2172" height="896" alt="nexus_h3_logo" src="https://github.com/user-attachments/assets/893728c2-2141-4d4e-a864-528fefa497b8" />

```

## 🗺️ Roadmap

- [x] Explicit **EMPTY** lease status (verified upstream pauses ≠ failures)
- [ ] Fault-isolated local disk spooling + Kafka replay (10 GB write-ahead ring buffer, ~66 h offline capacity)
- [ ] Fine-grained lease taxonomy: `EMPTY_UPSTREAM` vs. `MISSED` for automated completeness auditing
- [ ] Standalone **Rust compiler** binaries (Windows/macOS/Linux) with content-addressable slice caching — no third-party notebook needed
- [ ] Automated multi-tier live compaction (48-hour grace → daily ZSTD archives)
- [ ] Longitudinal **9-month cross-regime benchmark** (autumn → winter → spring commuting cycles)

## 🔒 Privacy & Licensing

- **Zero PII / GDPR compliant** — the observatory ingests only macroscopic infrastructure states and aggregate station/stop telemetry. No rider trajectories, smart-card validations, or device identifiers. H3 aggregation (~0.105 km² per cell) adds a further layer of re-identification protection.
- **Open licenses throughout** — IDFM, Smovengo, Paris Data, and ANFR streams under the French **Licence Ouverte / Open Licence 2.0 (Etalab)**; OSM morphology under **ODbL 1.0** (share-alike preserved); Open-Meteo under **CC BY 4.0**.
- **Honest missingness** — upstream gaps are preserved as explicit null spans and documented in a completeness manifest. No synthetic imputation, ever.

## 📜 Citation

If you use Nexus-H3 in your research, please cite:

```bibtex
@misc{islam2026nexush3,
  title       = {Nexus-H3: A Resource-Frugal Urban Mobility Observatory for
                 Continuous Telemetry Preservation and Reproducible Benchmarking},
  author      = {Islam, Saiful},
  year        = {2026},
  institution = {Institut Polytechnique de Paris, Palaiseau, France},
  note        = {Preprint, October 2026},
  url         = {https://github.com/SaifulShohag/nexus-h3-observatory}
}
```

## 🙏 Acknowledgements

Nexus-H3 is built on open public feeds: **Paris Open Data** (road loops, multimodal counters), **Île-de-France Mobilités** (SIRI Lite real-time transit), **Smovengo / Vélib'** (GBFS), **Open-Meteo** (meteorology), **OpenStreetMap** contributors (ODbL), and **ARCEP/ANFR** (cellular antenna registries).

<div align="center">

**NEXUS-H3 · Urban Mobility Observatory**

*Preserving the ephemeral, one 3-minute window at a time.*

</div>
