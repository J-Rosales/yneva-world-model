# World Coherence QA Validation Spec

This document defines measurable, automatable checks for world-model coherence across regions. Each check includes thresholds, examples, and pass/fail criteria so QA can be automated.

## 1. Region-to-Region Gradient Checks

### 1.1 Temperature Gradient Consistency
**Intent:** Adjacent regions should not show unrealistic temperature discontinuities without a strong physical boundary (e.g., sharp elevation change, ocean/land boundary, or current).

**Method:** For each pair of adjacent regions:
- Compute mean annual temperature difference \(\Delta T\) in °C.
- Compute mean elevation difference \(\Delta z\) in meters.

**Thresholds:**
- Base limit: \(|\Delta T| \le 8°C\).
- If \(|\Delta z| > 1000 m\), allow \(|\Delta T| \le 12°C\) (lapse-rate-driven change).
- If a coastal boundary (land-to-ocean) is present, allow \(|\Delta T| \le 10°C\).

**Pass/Fail:**
- **Pass** if all adjacent pairs meet thresholds or have justified boundary tags.
- **Fail** if any pair exceeds thresholds without justification.

**Example:**
- Region A (15°C) adjacent to Region B (6°C), \(\Delta T=9°C\). If \(\Delta z=1200 m\), **Pass**. If \(\Delta z=200 m\), **Fail**.

### 1.2 Precipitation Gradient Consistency
**Intent:** Adjacent regions should not have abrupt precipitation shifts unless terrain or rain shadow effects exist.

**Method:** For each adjacent pair:
- Compute annual precipitation difference \(\Delta P\) in mm/year.
- Detect orographic barrier between regions (mountain chain or ridge).

**Thresholds:**
- Base limit: \(|\Delta P| \le 800 mm/year\).
- If orographic barrier present, allow \(|\Delta P| \le 1500 mm/year\).

**Pass/Fail:**
- **Pass** if thresholds satisfied or orographic barrier is tagged.
- **Fail** if precipitation difference exceeds thresholds without barrier.

**Example:**
- Windward 2200 mm vs leeward 500 mm (\(\Delta P=1700\)) with mountain barrier: **Fail** (exceeds 1500). With a larger barrier but still no cap, **Fail**. Lower to 1400: **Pass**.

### 1.3 Biome Adjacency Plausibility
**Intent:** Adjacent biomes should follow realistic transitions (e.g., desert → steppe → grassland → forest).

**Method:** Validate adjacency against a permitted transition matrix.

**Allowed Transitions (non-exhaustive):**
- Desert ↔ Steppe ↔ Grassland ↔ Temperate Forest
- Tundra ↔ Boreal Forest
- Tropical Rainforest ↔ Tropical Seasonal Forest ↔ Savanna
- Alpine ↔ Tundra ↔ Boreal Forest

**Disallowed Direct Transitions:**
- Desert ↔ Tropical Rainforest (without mountain or coastal modifiers)
- Tundra ↔ Tropical Rainforest

**Pass/Fail:**
- **Pass** if adjacency is allowed or has a tagged transitional modifier (e.g., coastal upwelling).
- **Fail** if disallowed transitions appear without modifiers.

## 2. Feature-Driven Expectations

### 2.1 Rain Shadow vs. Precipitation
**Intent:** Mountain ranges should create windward wet and leeward dry patterns consistent with orographic lift and rain shadow effects.

**Method:**
- Identify prevailing wind direction (from climate model or regional metadata).
- Identify mountain ranges and classify each adjacent region as windward or leeward.
- Compute precipitation ratios: \(P_{windward} / P_{leeward}\).

**Thresholds:**
- Expected ratio \(\ge 2.0\) for strong rain shadow; \(1.2 - 2.0\) for moderate.
- If ratio < 1.2, flag as inconsistent unless a maritime moisture source is present on leeward side.

**Pass/Fail:**
- **Pass** if ratio meets expected range or leeward has compensating moisture source.
- **Fail** if ratio < 1.2 without moisture source.

**Example:**
- Windward 1800 mm, leeward 600 mm → ratio 3.0: **Pass**.
- Windward 900 mm, leeward 850 mm → ratio 1.06: **Fail** unless leeward is coastal.

### 2.2 Coastal Upwelling vs. Aridity
**Intent:** Cold ocean currents and upwelling can produce coastal deserts (e.g., Namib/Atacama analogs).

**Method:**
- Identify coastal regions with cold current or upwelling tags.
- Check if adjacent coastal biome is arid or semi-arid and precipitation < 300 mm/year.

**Thresholds:**
- Coastal desert expectation: \(P < 300\) mm/year and mean annual temp 12–24°C.
- If upwelling present but precipitation > 600 mm/year, flag inconsistent.

**Pass/Fail:**
- **Pass** if upwelling coast matches arid criteria or has explicit warm current override.
- **Fail** if upwelling coast is humid without override.

### 2.3 River Network vs. Topography
**Intent:** Rivers should flow downhill to sea or terminal basins, with drainage density consistent with climate.

**Method:**
- For each river segment, check monotonic downhill gradient (allowing local noise ≤ 50 m).
- Check river termini: ocean, lake, or internal basin.
- Compute drainage density (km river per km²) per region.

**Thresholds:**
- Elevation gradient: net downhill from source to mouth, no uphill jumps > 50 m.
- Drainage density: humid regions > 0.5 km/km²; arid regions < 0.2 km/km².

**Pass/Fail:**
- **Pass** if gradient is downhill and drainage density matches biome/aridity.
- **Fail** if rivers flow uphill or arid regions are over-drained.

## 3. Elevation vs. Biome Constraints

### 3.1 Lapse Rate Compatibility
**Intent:** Temperature decreases with elevation (~6.5°C per 1000 m in the troposphere). Biomes should reflect this.

**Method:**
- For each region, compute expected temperature from reference lowland temperature and elevation.
- Compare observed biome and temperature bands.

**Thresholds:**
- Expected \(T_{expected} = T_{lowland} - 6.5°C \times (\Delta z / 1000 m)\).
- Allow deviation up to ±3°C due to local effects.

**Pass/Fail:**
- **Pass** if regional temperatures align with lapse-rate bounds.
- **Fail** if high elevations show tropical temperatures without geothermal exception.

**Example:**
- Lowland 24°C at 0 m, highland 3000 m → expected 4.5°C ±3°C. If actual 16°C: **Fail**.

### 3.2 Alpine and Snowline Constraints
**Intent:** Permanent snowline and alpine biomes should appear above certain elevation thresholds depending on latitude and climate.

**Method:**
- Identify snow/ice or alpine biomes.
- Compare region elevation and latitude.

**Thresholds:**
- Low latitudes (0–20°): snowline > 4500 m.
- Mid latitudes (20–50°): snowline > 3000 m.
- High latitudes (>50°): snowline > 1500 m.

**Pass/Fail:**
- **Pass** if snow/ice biomes appear above thresholds.
- **Fail** if snow/ice appears far below thresholds without polar maritime tag.

## 4. Climate Classification Consistency

### 4.1 Köppen-Like Classification vs. Inputs
**Intent:** Biomes should align with temperature and precipitation ranges consistent with common climate classifications.

**Method:**
- Derive climate class from temperature and precipitation inputs.
- Compare to assigned biome.

**Thresholds (simplified):**
- Tropical rainforest: mean annual T > 18°C and P > 2000 mm.
- Desert: P < 250 mm.
- Tundra: warmest month < 10°C.

**Pass/Fail:**
- **Pass** if biome matches climate class thresholds.
- **Fail** if mismatched.

**Example:**
- Region with P=180 mm labeled as temperate forest → **Fail**.

## 5. Conflict Rules

### 5.1 Hydrology vs. Aridity
**Intent:** High precipitation and high aridity should not co-occur unless supported by unique mechanisms.

**Method:**
- Check if region has arid biome but precipitation > 600 mm/year.
- Check if humid biome but precipitation < 300 mm/year.

**Pass/Fail:**
- **Pass** if biome and precipitation align or override tags exist (e.g., karst infiltration or seasonal fog deserts).
- **Fail** if contradictory without override.

### 5.2 Temperature vs. Ice Coverage
**Intent:** Permanent ice requires low temperatures.

**Method:**
- If biome is ice sheet or permanent snow, check mean annual temperature.

**Thresholds:**
- Mean annual temperature must be < 0°C.
- If 0–2°C, allow only with high elevation (>3000 m) or high latitude (>60°).

**Pass/Fail:**
- **Pass** if conditions met.
- **Fail** otherwise.

### 5.3 Vegetation vs. Soil Moisture
**Intent:** Dense forests require sufficient moisture.

**Method:**
- If biome is dense forest, check precipitation and soil moisture index.

**Thresholds:**
- Dense forest requires P > 800 mm/year and soil moisture index > 0.5.

**Pass/Fail:**
- **Pass** if thresholds met.
- **Fail** if insufficient moisture.

## 6. Examples of Automated QA Outputs

### Pass Example
- Adjacent regions show \(|\Delta T|=6°C\) and \(|\Delta P|=400 mm\) → **Pass**.
- Windward/leeward precipitation ratio = 2.3 → **Pass**.
- Alpine biome at 3200 m and 35° latitude → **Pass**.

### Fail Example
- Desert adjacent to tropical rainforest with no transitional modifier → **Fail**.
- River segment with 120 m uphill step → **Fail**.
- Upwelling coast with 1200 mm precipitation and humid forest biome → **Fail**.

## 7. QA Implementation Notes

- **Adjacency Graph:** Ensure region adjacency is derived from shared borders, not only corner touches.
- **Normalization:** Use consistent units (°C, mm/year, m).
- **Overrides:** Allow metadata flags (e.g., geothermal hotspot, coastal warm current) that justify exceptions.
- **Audit Trail:** QA should emit a machine-readable report listing failing region IDs, metrics, and thresholds.
