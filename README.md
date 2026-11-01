# Territorial Transformation Index (TTI): Embedding-based territorial change detection — Application to Brazil (2017–2024)

**Author:** Vinicius Buri — LuxVerso Research Initiative, Salvador, Bahia, Brazil
**ORCID:** 0009-0000-6006-1516
**Contact:** viniburilux@gmail.com
**Version:** 0.6
**License:** CC-BY 4.0
**Date:** April 2026

---

## Overview

This repository contains the preprint, datasets, code and national raster for the **Territorial Transformation Index (TTI)**, a scalar, label-agnostic, spatially scale-invariant index of territorial change based on annual satellite semantic embeddings from the *AlphaEarth Foundations* model (Google + DeepMind).

TTI is defined as the complement of the cosine similarity between the 64-dimensional embedding vectors of two distinct years for the same pixel:

**TTI(x, t₁, t₂) = 1 − E(x, t₁) · E(x, t₂)**

(where E vectors are L2-normalized by construction.)

This deposit presents the full package supporting the national application of TTI to Brazil for the period 2017–2024, including the preprint, derived datasets, the aggregated national raster, a validation dataset, and a reproducible Colab notebook.

---

## Contents

```
TTI_Brazil_2017_2024/
├── paper/
│   ├── TTI_preprint_v0.6_EN.pdf            Main preprint (English)
│   ├── TTI_preprint_v0.6_EN.md             Markdown source (English)
│   ├── TTI_preprint_v0.6_PT.pdf            Portuguese version
│   └── TTI_preprint_v0.6_PT.md             Markdown source (Portuguese)
│
├── raster/
│   └── GW_STT_2017_2024.tif                National TTI raster, 2000 m, Brazil
│
├── datasets/
│   ├── TTI_estadual_final.csv              27 states, mean TTI + percentiles
│   ├── TTI_municipal_fixed.csv             5,506 municipalities, mean TTI
│   ├── TTI_grid_25km_north.csv             25 km regular grid — North region
│   ├── TTI_grid_25km_northeast.csv         25 km regular grid — Northeast region
│   ├── TTI_grid_25km_central.csv           25 km regular grid — Central region
│   ├── TTI_grid_25km_southeast.csv         25 km regular grid — Southeast region
│   ├── TTI_grid_25km_south.csv             25 km regular grid — South region
│   └── TTI_pixel_sample_brasil.csv         Random pixel sample, N=10,000, seed=42
│
├── validation/
│   └── GhostWorks_Validacao_S2.csv         36 candidates validated via Sentinel-2
│
├── code/
│   └── TTI_pipeline_colab.ipynb            Reproducible Google Earth Engine notebook
│
├── README.md                                This file
└── LICENSE                                  CC-BY 4.0
```

---

## Main findings

- **Empirical distribution (N = 10,000, Brazil):** median 0.030; P90 0.087; P99 0.209; sample maximum 0.611 (raster-level maximum: 0.865, concentrated in extreme transformation hotspots). Heavy-tailed profile consistent with complex territorial systems.

- **Counter-intuitive spatial pattern:** Northeastern states lead mean TTI — Sergipe (0.146), Alagoas (0.143), Pernambuco (0.114) — while Mato Grosso ranks 19th (0.037) and Amazonas 27th (0.018).

- **Top municipalities:** All 20 highest-TTI municipalities are in the Northeast, with Cumbe/SE leading (0.277).

- **Multi-scale robustness:** three aggregation methods (direct pixel→state, pixel→municipality→state, 25 km regular grid) converge within ≤ 0.015 absolute difference, providing empirical evidence of the index's scale-invariance when analysis is kept uniform.

- **Case study (Remanso/BA, São Francisco River):** TTI mean 0.133 (3.1× national mean), maximum 0.690, capturing extreme water body retraction at Sobradinho Lake without any water mask.

- **Validation:** 47.2% of 36 candidate points show strong independent evidence in Sentinel-2 NDVI + SAR; 80.5% show at least one independent sensor confirming transformation.

- **Main hypothesis (candidate, not established):** Northeastern dominance may be associated with the large expansion of renewable energy infrastructure (wind and photovoltaic) in the region during 2017–2024. Confirmation via controlled multivariate regression is left as future work.

---

## How to cite

```bibtex
@dataset{buri_2026_tti_brazil,
  author       = {Buri, Vinicius},
  title        = {{Territorial Transformation Index (TTI):
                   Embedding-based territorial change detection —
                   Application to Brazil (2017–2024)}},
  year         = {2026},
  publisher    = {Zenodo},
  version      = {v0.6},
  doi          = {10.5281/zenodo.XXXXXXX},
  url          = {https://doi.org/10.5281/zenodo.XXXXXXX}
}
```

Plain text:

> Buri, V. (2026). *Territorial Transformation Index (TTI): Embedding-based territorial change detection — Application to Brazil (2017–2024).* Zenodo. https://doi.org/10.5281/zenodo.XXXXXXX

---

## Reproducibility

The TTI pipeline can be reproduced on Google Earth Engine using any free-tier account:

```python
import ee
ee.Initialize()

dataset  = ee.ImageCollection('GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL')
emb_2017 = dataset.filterDate('2017-01-01', '2018-01-01').mosaic()
emb_2024 = dataset.filterDate('2024-01-01', '2025-01-01').mosaic()

dot = emb_2017.multiply(emb_2024).reduce(ee.Reducer.sum())
tti = ee.Image(1).subtract(dot)  # embeddings are L2-normalized
```

The full reproducible notebook is included in `code/TTI_pipeline_colab.ipynb`.

The national raster `GW_STT_2017_2024.tif` is provided at 2000 m resolution; it can also be accessed as a public Google Earth Engine asset at `projects/project-2715df31-12cf-4aed-908/assets/GW_STT_2017_2024`.

---

## License and attribution

This work is released under **CC-BY 4.0**. Any use must cite the work (see "How to cite").

Primary data source: **GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL**, released under CC-BY 4.0 by Google and Google DeepMind. Attribution: *AlphaEarth Foundation Satellite Embedding dataset, produced by Google and Google DeepMind.*

Administrative boundaries: **FAO/GAUL/2015**.

---

## Contact

Vinicius Buri — LuxVerso Research Initiative
Email: viniburilux@gmail.com
ORCID: 0009-0000-6006-1516

---

*Version 0.6 — April 2026*
