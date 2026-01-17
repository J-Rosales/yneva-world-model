Below is a concise documentation note you can ship alongside the file.

---

# Elevation Map (`elevation_map.v1.npy`)

## Purpose

This elevation map encodes **relative terrain height** for the Yneva sub-region.
It is **aesthetic and narrative-first**, not a physically exact DEM. Its role is to:

* Constrain geographic reasoning (drainage, rain shadows, passes).
* Provide a coherent altitude signal for LLM-driven inference.
* Serve as an input layer for feature extraction (mountains, plateaus, basins).

It may be **subverted or locally corrected** if downstream models demonstrate that a configuration is implausible.

---

## Data Format

* **File**: `elevation_map.v1.npy`
* **Type**: NumPy array, `float32`
* **Shape**: `700 × 700`
* **Units**: meters above/below sea level
* **Coordinate space**: pixel-aligned with `colorcodes.png` and other derived rasters

Each cell represents the *typical* elevation of that location, not micro-relief.

---

## Source

The map is derived from a hand-authored **height preference image** (`croppedHeightPreference.png`), where colors represent comparative altitude bands.
Pixel colors were mapped to elevation values using nearest-color matching to tolerate fuzziness and gradients.

---

## Color → Elevation Mapping

| Color (hex) | Interpretation                                | Elevation (m) |
| ----------- | --------------------------------------------- | ------------- |
| `#000000`   | Deep sea                                      | −4000         |
| `#404040`   | Shallow sea / deep lake                       | −200          |
| `#81e232`   | Lowlands / deltas (Netherlands-like)          | 10            |
| `#e2e223`   | Flatlands / grasslands                        | 120           |
| `#f1ae11`   | Foothills                                     | 600           |
| `#ff7b00`   | High plateau / low mountains (Anatolian-like) | 1700          |
| `#ff0000`   | High mountains (Alps-like peaks)              | 3500          |

These values are **representative**, not absolute. Their main purpose is to establish *ordering* and *relative magnitude*.

---

## How to Interpret the Map

### What it *means*

* Higher values imply:

  * cooler temperatures
  * greater orographic influence
  * potential snowpack or glacial headwaters
* Lower values imply:

  * warmer conditions
  * flatter terrain
  * higher likelihood of deltas, wetlands, or agriculture

### What it does *not* mean*

* It is not a high-resolution DEM.
* It does not encode slopes, cliffs, or erosion explicitly.
* Small pixel-to-pixel variation should not be over-interpreted.

Treat it as a **terrain envelope**, not a ground survey.

---

## Intended Uses

This elevation map is suitable for:

* **Feature extraction**

  * mountain ranges (contiguous high bands)
  * plateaus vs valleys
* **Hydrology inference**

  * drainage direction (downhill toward seas)
  * snowmelt vs rainfall-fed rivers
* **Climate inference**

  * rain shadows
  * alpine vs lowland biomes
* **Interface generation**

  * mountain barriers vs corridors between regions

It pairs directly with:

* region label rasters
* feature label rasters
* adjacency/interface models

---

## Authorial Priority

This map encodes **authorial intent** about the world’s shape and feel.
Automated systems should respect it **unless** they can demonstrate:

* physical impossibility (e.g. endorheic basin draining uphill),
* or strong global inconsistency (e.g. tropical rainforest above permanent snowline).

In such cases, systems should propose adjustments rather than silently overriding it.

---

## Summary

Think of `elevation_map.v1.npy` as the **terrain skeleton** of the world:

* simple,
* legible,
* constraint-forming,
* and designed to support inference rather than replace judgment.