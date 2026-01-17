# yneva-world-model

`yneva-world-model` is a **conversation-driven world representation repository**.
It contains the authoritative geographic, environmental, and structural model of the world of **Yneva**, expressed through a combination of authored inputs, spatial data, and LLM-assisted derivations.

This repository is **not** a codebase and **not** a build system.
Its primary mode of evolution is **human–LLM collaboration**, with artifacts produced, refined, and curated through dialogue.

---

## Purpose

The repository exists to:

* Encode the **geographic reality** of the world (regions, terrain, elevation, adjacency).
* Capture **authorial intent** in a form that is both human-legible and machine-interpretable.
* Provide a stable substrate from which **world fragments** (geography, climate, interfaces, constraints) can be generated and reasoned about.
* Allow incomplete or suggestive inputs to be **coherently completed** through inference, while preserving provenance and uncertainty.

It intentionally separates:

* what is *asserted*,
* what is *suggested*,
* and what is *accepted*.

---

## Core concepts

The world model is organized around three primary abstractions:

### Regions

Regions are **exclusive spatial partitions** of the map.
Every point belongs to exactly one region.

Regions carry:

* identity (id, name),
* topology (neighbors, borders),
* and optionally geographic descriptors (climate, terrain, hydrology).

They do **not** own shared features such as mountain ranges or rivers.

### Features

Features are **overlapping physical or environmental entities**:

* mountain ranges,
* rivers,
* valleys,
* plateaus,
* wetlands,
* deserts,
* climatic belts.

Features may cross multiple regions and are the primary carriers of **causal geography**.

### Interfaces

Interfaces describe **how two adjacent regions relate along their shared border**:

* barriers vs corridors,
* dominant features along the border,
* seasonal permeability (passes closed by snow, rivers flooding, etc.).

Interfaces explain *why* regions differ or connect, without duplicating features.

---

## Data philosophy

### Conversation-first

Most derived content in this repository is produced via **LLM conversation**, not scripts.
Prompts, transcripts, and intermediate outputs are first-class artifacts.

### Authoritative vs derived

The repository distinguishes clearly between layers:

* **Source**
  Immutable inputs: rasters, region labels, adjacency summaries.

* **Authored**
  Human-written descriptions, overrides, and decisions.

* **Derived**
  LLM-generated suggestions, inferences, and candidate models.

* **Curated outputs**
  Material that has been reviewed and accepted as part of the world state.

Nothing in `derived/` is automatically authoritative.

### Aesthetic priority, physical plausibility

Spatial inputs (such as elevation) encode **aesthetic and narrative preference first**.
They are allowed to be fuzzy or suggestive, and may be overridden **only when physical impossibility or strong inconsistency is demonstrated**.

---

## Repository structure (high level)

```
docs/        → world scope, assumptions, conceptual models, conversations
data/
  source/    → immutable spatial and structural inputs
  authored/  → human-written world content
  derived/   → LLM-generated suggestions and candidates
schemas/     → reference schemas for regions, features, interfaces
outputs/     → curated, accepted world state snapshots
```

See the directory tree for full detail.

---

## Elevation and spatial data

Elevation is represented as a **relative terrain envelope**, not a survey-grade DEM.

* Stored as NumPy rasters aligned to region labels.
* Used to support reasoning about:

  * drainage,
  * rain shadows,
  * passes,
  * plateaus vs basins,
  * climate gradients.

Elevation, regions, and features are designed to be **combined**, not interpreted in isolation.

---

## Versioning and change

* Files are versioned explicitly (`.v1`, `.v2`, etc.).
* Changes are documented narratively in `docs/world/changelog.md`.
* Promotion from `derived/` to `outputs/curated/` is an editorial decision, not an automatic process.

---

## What this repository is not

* Not a simulation engine.
* Not a deterministic build pipeline.
* Not a database snapshot.
* Not tied to any specific downstream consumer.

It is a **world model notebook**: structured, constrained, and intentionally incomplete where imagination and inference are expected to operate.

---

## Status

This repository is **active and evolving**.
Incompleteness is intentional.
Ambiguity is preserved where it is productive.

