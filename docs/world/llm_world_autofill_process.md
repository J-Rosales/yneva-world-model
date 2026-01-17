# Sparse World Autofill Agent Protocol

## What you get
- **Regions/topology**: region IDs, names, kinds (land/sea/lake/island), geometry summaries (area/bbox/centroid), and adjacency edges with rough direction of contact.
- **Sparse region descriptions**: natural-language facts for some regions (others may be blank).
- **Global assumptions**: world-scale rules (prevailing wind directions, coarse seasonality bands, ocean moisture/temperature moderation).

## Your job
Generate **autofill candidates** for missing/underspecified properties so the world is physically plausible.

## Hard rules
1) **Do not overwrite authored facts.** Keep them as `authored_facts`.
2) **Separate inference from authorship.** Put all new proposals under `autofill.candidates`.
3) **Always include rationale.** Every candidate must cite which inputs caused the suggestion.
4) **Always provide alternatives when uncertain.** Prefer 2–4 candidates.

## Minimal per-region output schema
Use this structure per region that needs completion:

```yaml
region_id: <id>
region_name: <name>
authored_facts:  # parsed from the sparse description file
  biome: <optional>
  terrain: <optional>
  hydrology: <optional>
autofill:
  candidates:
    - id: C1
      geo:
        biome: {<biome_enum>: <weight>, ...}
        climate:
          wetness: <humid|semi_arid|arid|variable>
          temperature: <hot|temperate|cold|variable>
          seasonality: <wet_dry|monsoonal|temperate|snowmelt|mixed>
        hydrology_seasonality:
          baseflow_index: <0..1>
          drying_risk_summer: <0..1>
          flood_risk_spring: <0..1>
          freeze_risk_winter: <0..1>
      confidence: <0..1>
      rationale:
        - <bullet list of specific constraints>
    - id: C2
      ...
```

## Inference procedure
### Step 1: Build constraints from topology
From regions/topology, derive:
- **Coastal vs inland**: land region adjacent to any sea region is coastal.
- **Deep inland**: no sea neighbors within 2 adjacency steps.
- **Directionality**: use centroid deltas or directional contact to label neighbors as roughly N/E/S/W.

### Step 2: Apply global assumptions
Use the assumptions file to determine:
- prevailing wind direction in the region’s latitude band (or global default)
- expected seasonality regime (wet/dry vs temperate vs snow-dominated)
- strength of maritime moderation

If an assumption is missing, lower confidence and explicitly state what is missing.

### Step 3: Use authored seed facts (and neighbor facts)
Treat authored facts as high-confidence anchors. Propagate only as *suggestions*:
- If a neighbor is “humid jungle” and winds blow from that neighbor toward the target, the target is more likely humid *unless* a barrier exists.
- If a neighbor is “arid” and the target is windward/coastal, allow a wetter alternative.

### Step 4: Causal feature reasoning (rain shadow example)
If either (a) the sparse descriptions mention a mountain barrier on the border, or (b) a provided feature dataset tags a mountain range as an orographic barrier:
- Determine windward vs leeward side from prevailing winds.
- If target is **leeward** of a humid region across a barrier, propose drier biomes (steppe/desert) and higher `drying_risk_summer`.
- If target is **windward** or coastal with no barrier, propose wetter biomes and lower drying risk.

### Step 5: Always produce multiple candidates
When the data underdetermines the result, provide alternatives, e.g.:
- **Leeward steppe** (semi-arid) vs **true desert** (arid) depending on distance to sea and barrier strength.
- **Monsoonal savanna** vs **humid forest** depending on the seasonality band.

## Rationale requirements
Each candidate rationale must reference concrete inputs, for example:
- “Adjacency: region 18 is east of region 7 (centroid dx>0).”
- “Global assumptions: winds W→E in this band.”
- “Border described as high mountains: rain shadow plausible.”
- “Target is 2 steps from sea: higher continentality.”

## Confidence rubric
- **0.80–0.95**: directly implied by authored facts + strong constraints.
- **0.55–0.80**: implied, but multiple plausible outcomes.
- **0.30–0.55**: weakly constrained; keep candidates broad.
- **<0.30**: only list ideas; do not commit to numeric ranges.
