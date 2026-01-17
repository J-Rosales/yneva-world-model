# World Region Model

## Purpose
A **region** is a named territorial partition of a world map used for lore, gameplay logic, and simulation queries.

Regions are the **primary ownership/identity layer**: every location in the modeled extent belongs to exactly one region in the region label raster.

Physical geography (mountains, rivers, deserts, ocean currents, biomes) is modeled separately as **features**, which may overlap multiple regions. Cross-region relationships are modeled as **interfaces**.

## Core rules
1. **Partition rule**: the region layer is a complete, non-overlapping label raster (or equivalent polygon tiling). Each cell has one region id.
2. **Stable identity**: a region has a stable `id` used across versions; names can change.
3. **Feature independence**: regions should not attempt to “own” shared physical structures (e.g., an alpine chain). That structure is a feature.
4. **Interface semantics**: statements like “the Alps divide Northern Italy and Switzerland” or “the Po valley connects them” belong in interfaces, computed from border composition and refined by author overrides.

## Region data model (recommended fields)

### Minimal
- `id` (int or string)
- `name`
- `kind`: `land` | `sea` | `lake` | `island` | `archipelago` | `ice` (extend as needed)
- `aliases` (optional)

### Geometry
- `geometry`:
  - `type`: `raster_label` (recommended for your current pipeline)
  - `source`: filename/key for the region label raster
  - `value`: label id
  - `bbox_px`: `[x0, y0, x1, y1]` in the raster coordinate space used by `source`

### Topology
- `neighbors`: list of `{id, border_share}` where `border_share` is fraction of this region’s border touching that neighbor (optional but useful).

### Geo/Biosphere descriptor (`geo`)
The `geo` section is the region’s machine-usable environmental profile. It is **not** required to be fully accurate; it must be **internally consistent** and queryable.

Recommended structure:
- `summary`: short prose.
- `climate`:
  - `koppen` (optional): list of Köppen-like codes or your own enum.
  - `temp_c`: monthly means/ranges or seasonal stats.
  - `precip_mm`: monthly totals or seasonal totals.
  - `seasonality`:
    - `summer_dryness` (0..1)
    - `winter_wetness` (0..1)
    - `shoulder_seasons_length` (0..1) or `months`: `{spring:[...], autumn:[...]}`
- `terrain`:
  - `elevation_m`: `{mean, p10, p90}` or `{min, max}`
  - `ruggedness` (0..1)
  - `landforms`: weighted list
- `biosphere`:
  - `biomes`: weighted list
  - optional productivity proxy (e.g., `npp`)
- `hydrology`:
  - `drainage`: `exorheic` | `endorheic` | `cryptorheic` | `coastal`
  - `rivers`: list of major rivers (IDs or names)
  - `lakes`: list
  - `seasonality` (critical):
    - `baseflow_index` (0..1): higher means more stable flows (glacial/groundwater-fed).
    - `drying_risk_summer` (0..1): likelihood of low-flow / intermittent reaches in warm season.
    - `flood_risk_spring` (0..1)
    - `freeze_risk_winter` (0..1)
    - `notes` (optional)

#### Seasonality guidance
You can derive or author hydrology seasonality in three ways:
- **Direct authoring**: set indices based on intended worldbuilding.
- **Inference from climate** (recommended default):
  - high summer dryness + high temp → higher `drying_risk_summer`.
  - high winter snowfall + spring warming → higher `flood_risk_spring`.
  - low winter temps → higher `freeze_risk_winter`.
  - high precipitation spread across the year + high groundwater → higher `baseflow_index`.
- **Feature-driven overrides**: if a river is glacial-fed (a feature tag), increase `baseflow_index` and reduce `drying_risk_summer` for regions it crosses.

## How regions interact with features
- Regions may reference features via `feature_refs` for convenience, but **feature overlap metrics** should be computed and stored with the feature.
- Regions should not encode “shared ownership” of a feature; overlapping is not ownership.

## How regions interact with interfaces
- Interfaces are defined over **pairs of regions** (usually adjacent pairs).
- Regions do not store interface facts except convenience pointers.

## Build process (concrete)
1. Create/maintain a region label raster (`regions_labels.npy` or equivalent) for the target extent.
2. Compute per-region stats: area, bbox, centroid, border pixels.
3. Compute neighbor graph from border adjacency.
4. Author/compute `geo` descriptors:
   - optionally compute climate/terrain summaries from separate rasters (elevation, temperature, precipitation), or author them.
   - compute/infer hydrology seasonality indices using the rules above.
5. Keep region YAML small and stable; put derived matrices (overlap, borders) into build artifacts used to generate **features** and **interfaces**.
