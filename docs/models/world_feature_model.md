# World Feature Model

## Purpose
A **feature** is a physical, climatic, or biospheric structure that can:

- **Overlap multiple regions** (e.g., a mountain range crossing borders).
- **Be partially present** in a region without implying ownership.
- **Explain why interfaces behave a certain way** (barriers, corridors, climate gradients).

Features are the **world-physics layer**. Regions are the **identity/administrative layer**.

## Core rules
1. **Overlapping allowed**: features may overlap each other and regions.
2. **Geometry is explicit**: each feature has a geometry representation in the same coordinate space as the region raster (or a declared transform).
3. **Derived overlap is authoritative**: per-region coverage fractions are computed from geometry; do not hand-maintain them unless necessary.

## Feature kinds
Use an enum (extend as needed):

- `mountain_range`, `plateau`, `basin`, `plain`, `canyon`
- `river`, `river_basin`, `lake`, `wetland`, `aquifer`
- `desert`, `steppe_belt`, `rainforest_belt`, `taiga_belt`, `tundra_belt`
- `monsoon_zone`, `trade_wind_belt`, `ocean_current`
- `glacier`, `ice_sheet`
- `pass` (corridor feature), `strait`, `isthmus`

## Feature data model (recommended fields)

### Minimal
- `id`
- `kind`
- `name`
- `tags`: free-form but prefer a controlled vocabulary (e.g., `barrier`, `corridor`, `glacial_fed`, `navigable`, `seasonal`).

### Geometry
Pick one representation per feature; allow multiple if needed:

- `raster_mask`: mask/label raster in the same crop/extent as regions.
  - `{source, value}` (value is the feature label id)
- `polyline`: for rivers/ridges/roads (optionally with width)
- `polygon`: for basins/deserts/biome belts
- `point`: for passes, springs, chokepoints

### Environmental semantics
A feature can carry semantics used to infer region and interface properties.

Suggested fields (sparse allowed):
- `orography` (for mountain ranges): `{mean_elev_m, max_elev_m, glaciation_frac}`
- `hydrology` (for rivers/lakes/aquifers):
  - `regime`: `pluvial` | `nival` | `glacial` | `mixed`
  - `permanence`: `perennial` | `intermittent` | `ephemeral`
  - `seasonality` (critical):
    - `low_flow_months`: list `[1..12]` (optional)
    - `peak_flow_months`: list `[1..12]` (optional)
    - `drying_probability_summer` (0..1)
    - `freeze_probability_winter` (0..1)
    - `notes`
  - `navigability`: `{class: none|raft|barge|ship, seasonal: true|false}` (optional)
- `climate_effects`:
  - list of `{effect, strength}` where `effect` is e.g. `rain_shadow`, `orographic_rain`, `marine_moderation`.

### Derived relations (computed)
- `relations.overlaps_regions`: list of `{region_id, coverage_frac}` computed from feature geometry.
- `relations.border_presence`: optional list of `{a, b, share}` computed from border pixels (used to seed interfaces).

## How features interact with regions
- Regions may reference features for lore, but machine-useful overlap should come from the feature’s computed relations.
- Region `geo` may be **inferred/adjusted** using feature tags:
  - `glacial_fed` rivers increase `baseflow_index` and reduce `drying_risk_summer` in regions they cross.
  - `rain_shadow` from a mountain range reduces region precipitation on the lee side (if you model wind direction).
- Use the centralized tag-to-geo adjustment map in `data/source/yaml/feature_effects.v1.yaml` to keep feature-driven adjustments consistent across regions and any derived transition metrics.

## How features interact with interfaces
Interfaces describe the relationship between two regions. Feature geometry explains **why**:

- If a region border is mostly co-located with a `mountain_range` mask, the interface segment is a `mountain_barrier`.
- If the border overlaps a `pass` point or corridor mask, permeability increases seasonally.
- If the border follows a `river` polyline, interface type can be `river_boundary` and permeability can depend on river seasonality.

## Build process (concrete)
1. Create feature geometries (mask labels, polylines, points) in the same coordinate space as the region labels (or provide transforms).
2. Compute per-feature overlap with regions: `coverage_frac`.
3. For each adjacent region pair, compute border pixels and tally feature labels along the border.
4. Store computed overlap/border presence in `relations`.
5. Apply feature tag adjustments from `data/source/yaml/feature_effects.v1.yaml` when deriving region `geo` summaries and transition metrics.
6. Use these relations to auto-generate initial interface segments; then allow hand overrides in the interface YAML.
