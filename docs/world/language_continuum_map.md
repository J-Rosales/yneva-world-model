# Language continuum map (8th Century B.R.)

## Overview

The language continuum map represents general areas of language distributions in the 8th Century B.R.
It is a raster aligned to the same spatial extent as `colorcodes.png`, with each pixel storing a
language index defined in `data/source/yaml/language_continuum_palette.v1.yaml`.

**Primary assets**

- Index raster: `data/source/rasters/regions/colorLanguageContinuum_indices.npy`
- Visual reference: `data/source/rasters/regions/colorcodes.png`
- Index palette: `data/source/yaml/language_continuum_palette.v1.yaml`

## Philological taxonomy

```
enta
└─ felerinic

aluterian
└─ ynevo_vestish
   ├─ vestish
   │  ├─ peshic
   │  └─ adinian
   └─ ancient_ynevan
      ├─ dalian
      │  ├─ undic
      │  └─ lobric
      ├─ old_romish
      │  ├─ romish
      │  ├─ ombrish
      │  └─ oscan
      └─ ynedric
         └─ goddish
            ├─ normish
            └─ genlish

nyelfish
├─ proto_sakavish
│  ├─ sakavish
│  │  └─ rendish
│  ├─ lacurian
└─ old_talavish
   └─ talavish
```

## Notes

- Index 2 in the palette is an interaction region (Rommish-Lacurian dialect), not a standalone language.
- Unassigned areas are encoded as index 0.
