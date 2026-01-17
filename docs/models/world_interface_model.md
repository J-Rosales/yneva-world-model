# World Interface Model

## Purpose
An **interface** describes the relationship between **two regions** (typically adjacent) and is the primary place to encode statements like:

- “A mountain chain divides Region A and Region B.”
- “A river valley connects them.”
- “Crossing is easy in summer but difficult in winter.”

Interfaces are derived from:
- **Region adjacency** (from the region label raster) and
- **Feature composition along the shared border** (from feature geometries),
then refined with author overrides.

## Core rules
1. **Pair identity**: an interface is keyed by `(a, b)` with canonical ordering (e.g., ascending id).
2. **Adjacency-aware**: most interfaces are between adjacent regions; non-adjacent interfaces are allowed for “trade corridors” or “sea routes”, but must set `adjacency: false`.
3. **Segmented border**: the shared boundary can be decomposed into segments associated with features (mountain barrier, river boundary, coastal plain).
4. **Seasonality is first-class**: permeability and hazards may vary by season.

## Interface data model (recommended fields)

### Minimal
- `a`, `b`: region ids
- `adjacency`: boolean
- `summary`: short prose

### Border composition
- `border`:
  - `length_px` (or km if you have a distance model)
  - `segments`: list of segments, each with:
    - `share` (0..1): fraction of the border represented by this segment type
    - `type`: enum such as:
      - `mountain_barrier`, `mountain_pass`, `river_boundary`, `coastal_plain`, `desert_frontier`, `ice_wall`, `marsh_belt`, `open_sea`
    - `feature_id` (optional): feature responsible for the segment
    - `permeability` (0..1): baseline ease of crossing (1 = trivial)
    - `seasonality` (optional): seasonal modifiers

### Seasonal modifiers
Two compatible approaches:

**A) Multipliers** (simple):
- `seasonality`:
  - `winter`: `{permeability_mult: 0.3, notes: "passes snowed in"}`
  - `spring`: `{permeability_mult: 0.6, hazard: ["flooding"]}`
  - `summer`: `{permeability_mult: 1.0, hazard: ["rockfall"]}`
  - `autumn`: `{permeability_mult: 0.8}`

**B) Regime tags** (very compact):
- `seasonality`: `{regime: "snow_pass", severity: 0.7}`

Use A when you need numeric behavior; use B when you only need classification.

## How interfaces interact with regions
- Interfaces reference region ids; regions do not embed interface semantics.
- Region `geo.hydrology.seasonality` can influence interface defaults:
  - high `freeze_risk_winter` → reduce winter permeability for `river_boundary`
  - high `flood_risk_spring` → reduce spring permeability for `river_boundary` and `marsh_belt`

## How interfaces interact with features
- Interfaces should point to the feature(s) responsible for barrier/corridor behavior.
- A feature can explain both **division** and **connection** depending on sub-features:
  - `mountain_range` creates `mountain_barrier` segments
  - `pass` features create small `mountain_pass` segments with high permeability in summer

## Build process (concrete)
1. Compute adjacency pairs and shared-border pixel sets from the region label raster.
2. For each adjacency pair, sample feature geometries along the border to get a histogram of feature ids.
3. Map feature kinds/tags to default segment `type` and baseline `permeability`.
4. Derive initial seasonal modifiers from region climate/hydrology and feature tags:
   - feature tag `glacial_fed` lowers summer drying hazards
   - feature tag `ephemeral` increases summer crossing hazards for rivers
   - feature tag `snow_bound` decreases winter permeability for passes
5. Write the interface YAML and allow manual edits for lore-specific claims (named passes, tunnels, ferries, political restrictions).
