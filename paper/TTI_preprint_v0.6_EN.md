# Territorial Transformation Index (TTI): label-agnostic territorial change detection from satellite semantic embeddings

**National application to Brazil, 2017–2024**

---

**Author:** Vinicius Buri¹
**Affiliation:** ¹LuxVerso Research Initiative — Salvador, Bahia, Brazil
**ORCID:** 0009-0000-6006-1516
**Contact:** viniburilux@gmail.com
**Version:** Preprint v0.6 (EN)
**Date:** April 2026

---

## Abstract

This work introduces the **Territorial Transformation Index (TTI)** — a scalar, label-agnostic, spatially scale-invariant index defined over annual satellite semantic embeddings. TTI is computed as the complement of the cosine similarity between embedding vectors of two distinct years at the same location, providing a continuous measure of semantic dissimilarity between two territorial states. Unlike spectral indices such as NDVI/NDWI, which respond to specific physical properties, TTI operates on 64-dimensional learned representations produced by the *AlphaEarth Foundations* model (Google + DeepMind) and captures territorial change without depending on pre-defined classes or labeled supervision. We present a national implementation of TTI for Brazil for the period 2017–2024, built entirely on Google Earth Engine. The pixel-level distribution (N = 10,000 random samples over continental Brazil) exhibits median 0.030, P99 0.209 and maximum 0.865, with a heavy-tailed profile consistent with complex territorial systems. Aggregation by administrative unit reveals a counter-intuitive spatial pattern: Northeastern states (Sergipe, Alagoas, Pernambuco) lead mean territorial transformation, whereas states historically associated with the agricultural frontier (Mato Grosso ranked 19th, Amazonas 27th) occupy intermediate or low positions. We demonstrate the multi-scale robustness of the index through three aggregation methods (direct pixel→state, pixel→municipality→state, and 25 km regular grid), which converge when the analysis scale is kept uniform. We validate the index through four mechanisms: (i) four qualitative control points; (ii) a quantitative Sentinel-2 protocol covering 36 candidate locations (NDVI + SAR); (iii) a calibrator case with visual ground truth in the MATOPIBA region; and (iv) a pixel-level case study at the São Francisco River (Remanso/BA), whose maximum TTI of 0.69 corresponded to extreme water body retraction at the Sobradinho Lake. As a plausible hypothesis, we discuss the association between high TTI in Northeast Brazil and the expansion of renewable energy infrastructure (wind and photovoltaic) in the region during the period, without establishing causality. The index, the national dataset and the 2017–2024 Brazil raster are released openly under CC-BY 4.0.

**Keywords:** territorial change; semantic embeddings; remote sensing; change detection; Google Earth Engine; Brazil; renewable energy.

---

## 1. Introduction

Conventional territorial monitoring systems — PRODES, MapBiomas, spectral indices such as NDVI/NDWI — operate over pre-defined classes or specific physical properties. This confers considerable power within their intended domain (typically vegetation, water bodies, built-up area) and a structural limitation outside of it: they are blind to transformations that were not explicitly encoded.

This work starts from a different question: *what if, instead of classifying what a territory is, it were possible to measure how much it has become unrecognizable relative to itself?*

The proposal operationalizes this question as a semantic distance metric between two temporal states of the same territory, supported by learned satellite embeddings. While a spectral index asks *"how green is this pixel?"*, the index proposed here asks *"how different is this pixel now from what it was before?"* — without restricting the type of difference.

The contributions of this work are:

1. **Formal definition** of the *Territorial Transformation Index* (TTI), a scalar metric of territorial change based on cosine similarity between annual semantic embeddings.
2. **National implementation** of TTI over Brazil (2017–2024), via Google Earth Engine, with the resulting raster published as a reusable asset.
3. **Empirical characterization** of the index distribution through pixel-level sampling.
4. **Demonstration of multi-scale robustness** through three aggregation methods (direct pixel→state, pixel→municipality→state, and 25 km regular grid).
5. **Validation protocol** at four levels: qualitative control, quantitative Sentinel-2 protocol, calibrator case with visual ground truth, and pixel-level case study.
6. **Counter-intuitive empirical finding** on the geography of territorial transformation in Brazil: Northeastern dominance, with an associated hypothesis of renewable energy infrastructure expansion.

---

## 2. Concept: what is TTI

TTI is a scalar index that answers the question: *"how much did the semantic state of this territory change between time t₁ and time t₂?"*

Three properties define the index:

**Label-agnosticism.** TTI does not require thematic classes (forest, urban, agriculture). Any transformation that alters the vectorial representation of the territory is captured — be it urbanization, installation of energy infrastructure, agricultural conversion, environmental degradation, water body retraction, or any combination thereof.

**Invariance to spatial unit.** The index is defined pixel by pixel. Aggregations over regular grids, administrative meshes or arbitrary polygons are **operations of presentation**, not properties of the index. This distinction is central: TTI produces *a single continuous raster* over which any geometry can be queried.

**Continuity.** The index varies smoothly between full stability (TTI = 0) and maximum transformation (TTI = 2, though observed values rarely exceed 0.9 in practice). There is no fixed threshold; the definition of "significant transformation" is an application choice, not an index property.

---

## 3. Data source

The fundamental dataset is **GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL**, publicly available on Google Earth Engine. Produced by the *AlphaEarth Foundations* model (Google + DeepMind), the dataset represents each 10 m pixel of the terrestrial surface by a 64-dimensional vector that integrates annual series of Sentinel-1 (radar), Sentinel-2 (optical), Landsat and auxiliary sources. **The vectors are L2-normalized by construction (unit magnitude)**, which makes them directly comparable via dot product. The dataset covers globally the period 2017–2024 (inclusive) with annual cadence.

This is the sole primary source used in the TTI computation. Auxiliary sources (FAO/GAUL for administrative boundaries; Sentinel-2 and Sentinel-1 for independent validation) are specified below.

---

## 4. Method

### 4.1 Formal definition

For any spatial location *x* and two years *t₁* and *t₂*, let *E(x, t) ∈ ℝ⁶⁴* be the L2-normalized embedding vector associated with pixel *x* in year *t*. The *Territorial Transformation Index* is defined as:

$$\text{TTI}(x, t_1, t_2) = 1 - \frac{E(x, t_1) \cdot E(x, t_2)}{\|E(x, t_1)\| \cdot \|E(x, t_2)\|}$$

Because embeddings are L2-normalized (‖E‖ = 1), the expression reduces to:

$$\text{TTI}(x, t_1, t_2) = 1 - E(x, t_1) \cdot E(x, t_2)$$

The index satisfies TTI ∈ [0, 2], with values close to 0 indicating semantic stability and high values indicating pronounced transformation.

### 4.2 Brazil 2017–2024 implementation

The national implementation uses the following temporal pair:

- *E₁ = mosaic(SATELLITE_EMBEDDING, 2017-01-01, 2018-01-01)*
- *E₂ = mosaic(SATELLITE_EMBEDDING, 2024-01-01, 2025-01-01)*

The pair 2017 vs 2024 was chosen to **maximize the cumulative signal** within the available range of the dataset, capturing seven years of accumulated transformation and maximizing the signal-to-noise ratio for persistent changes.

The computation is pixel-level at 10 m native resolution:

```python
emb_2017 = dataset.filterDate('2017-01-01', '2018-01-01').mosaic()
emb_2024 = dataset.filterDate('2024-01-01', '2025-01-01').mosaic()
dot     = emb_2017.multiply(emb_2024).reduce(ee.Reducer.sum())
tti     = ee.Image(1).subtract(dot)  # embeddings already L2-normalized
```

Due to memory constraints on Google Earth Engine workers for national-scale operations, the resulting raster was **aggregated to 2000 m via bilinear resampling** and saved as a public asset in the GEE project (`GW_STT_2017_2024`). The asset covers the continental extent of Brazil (longitude −74.02 to −28.82; latitude −33.82 to 5.32). Point-wise validation (Section 8) confirms that aggregation to 2000 m preserves the discriminatory capacity of the index at municipal and broader scales. For analyses at finer scales, the computation can be re-executed locally over smaller clipped regions at native resolution.

### 4.3 Computational environment

The pipeline was constructed and executed in a resource-constrained computational environment — smartphone + Google Colab — without the use of dedicated workstations or paid computational resources beyond the standard free tier of Google Earth Engine. This circumstance is explicitly reported because it reinforces a practical property of the method: **low barrier to replication**.

---

## 5. Empirical distribution

To characterize the distribution of TTI over continental Brazil, uniform random sampling of N = 10,000 pixels was performed, with fixed seed (seed = 42) for reproducibility, at 500 m subsampling resolution justified by the spatial smoothness of the TTI signal at scales above ≈ 100 m. *This subsampling does not affect the distributional properties of the index*, as confirmed by comparison with the direct computation over the complete national raster (sample mean: 0.0429; raster mean: 0.0435; sample P99: 0.209; raster P99: 0.217).

**Main statistics (N = 10,000):**

| Statistic | Value |
|---|---|
| Mean | 0.0429 |
| Standard deviation | 0.0423 |
| Minimum | 0.0039 |
| Median (P50) | 0.0303 |
| P75 | 0.0551 |
| P90 | 0.0866 |
| P95 | 0.1182 |
| P99 | 0.2093 |
| Maximum (sample) | 0.6108 |

**Observations on the distribution:**

1. **Heavy-tail profile.** The ratio of P99 to median is ≈ 7×, and of sample maximum to median ≈ 20×. The vast majority of the Brazilian territory exhibits semantic stability between 2017 and 2024, with pronounced transformations concentrated in small fractions of space.

2. **Threshold positioning.** Values above ≈ 0.09 (P90) represent transformation above the general trend. Values above ≈ 0.21 (P99) represent the most-transformed 1% of the territory and correspond to transformations of notable magnitude. The range [0, 0.03] is interpretable as the index "background noise".

3. **Expected method behaviour.** Heavy-tail distributions are characteristic of complex geographical systems, in which most of space is stable and a minority concentrates the dynamics. That TTI yields this profile in an unsupervised manner suggests that the index captures the kind of signal a territorial change detector should capture.

---

## 6. Spatial aggregation

While TTI is defined at the pixel level, aggregations over larger spatial units are useful for communication and cross-reference with administrative data. We present three aggregations that jointly demonstrate the multi-scale robustness of the index.

### 6.1 Aggregation by federative unit (direct pixel→state)

Each of the 27 federative units had TTI aggregated directly from the raster via `reduceRegions` at uniform scale of 2000 m, with administrative matching performed by numerical GAUL code (ADM1_CODE).

**Table 1 — State ranking by mean TTI (direct pixel→state):**

| # | State | Mean TTI | SD | P50 | P90 | P99 |
|---|---|---|---|---|---|---|
| 1 | Sergipe | 0.1460 | 0.0624 | 0.1396 | 0.2293 | 0.3057 |
| 2 | Alagoas | 0.1426 | 0.0594 | 0.1397 | 0.2217 | 0.2918 |
| 3 | Pernambuco | 0.1135 | 0.0571 | 0.1006 | 0.1962 | 0.2861 |
| 4 | Espírito Santo | 0.0907 | 0.0325 | 0.0850 | 0.1279 | 0.2023 |
| 5 | Bahia | 0.0846 | 0.0521 | 0.0723 | 0.1465 | 0.2557 |
| 6 | Rio de Janeiro | 0.0739 | 0.0284 | 0.0693 | 0.1104 | 0.1573 |
| 7 | Paraíba | 0.0732 | 0.0377 | 0.0675 | 0.1220 | 0.1767 |
| 8 | Mato Grosso do Sul | 0.0689 | 0.0475 | 0.0566 | 0.1152 | 0.2754 |
| 9 | Paraná | 0.0651 | 0.0306 | 0.0606 | 0.0995 | 0.1698 |
| 10 | Rio Grande do Norte | 0.0623 | 0.0334 | 0.0527 | 0.1035 | 0.1622 |
| 11 | Roraima | 0.0591 | 0.0537 | 0.0449 | 0.1114 | 0.2868 |
| 12 | Rio Grande do Sul | 0.0568 | 0.0358 | 0.0488 | 0.0957 | 0.1893 |
| 13 | Santa Catarina | 0.0565 | 0.0284 | 0.0527 | 0.0800 | 0.1736 |
| 14 | Minas Gerais | 0.0561 | 0.0220 | 0.0537 | 0.0791 | 0.1280 |
| 15 | São Paulo | 0.0537 | 0.0336 | 0.0449 | 0.0878 | 0.1815 |
| 16 | Piauí | 0.0456 | 0.0276 | 0.0400 | 0.0713 | 0.1591 |
| 17 | Maranhão | 0.0444 | 0.0304 | 0.0371 | 0.0683 | 0.1738 |
| 18 | Ceará | 0.0412 | 0.0275 | 0.0371 | 0.0644 | 0.1231 |
| 19 | Mato Grosso | 0.0374 | 0.0500 | 0.0253 | 0.0722 | 0.2717 |
| 20 | Rondônia | 0.0347 | 0.0551 | 0.0214 | 0.0606 | 0.3299 |
| 21 | Distrito Federal | 0.0332 | 0.0162 | 0.0318 | 0.0503 | 0.0930 |
| 22 | Goiás | 0.0322 | 0.0215 | 0.0253 | 0.0526 | 0.1034 |
| 23 | Pará | 0.0315 | 0.0416 | 0.0214 | 0.0566 | 0.2129 |
| 24 | Tocantins | 0.0286 | 0.0183 | 0.0253 | 0.0487 | 0.0997 |
| 25 | Acre | 0.0255 | 0.0228 | 0.0175 | 0.0449 | 0.1189 |
| 26 | Amapá | 0.0249 | 0.0260 | 0.0135 | 0.0527 | 0.1270 |
| 27 | Amazonas | 0.0182 | 0.0309 | 0.0135 | 0.0252 | 0.1113 |

### 6.2 Municipal aggregation

Municipal aggregation was computed for the 5,506 Brazilian municipalities (via FAO/GAUL/2015/level2, matching by ADM2_CODE), at uniform scale of 2000 m.

**Table 2 — Top 15 municipalities by mean TTI:**

| # | Municipality | State | Mean TTI |
|---|---|---|---|
| 1 | Cumbe | SE | 0.2770 |
| 2 | São Francisco | SE | 0.2629 |
| 3 | Cachoeirinha | PE | 0.2616 |
| 4 | Tacaimbó | PE | 0.2447 |
| 5 | Manari | PE | 0.2422 |
| 6 | Paranatama | PE | 0.2412 |
| 7 | Calçado | PE | 0.2410 |
| 8 | Capoeiras | PE | 0.2392 |
| 9 | Itabi | SE | 0.2380 |
| 10 | Caetés | PE | 0.2380 |
| 11 | Itaíba | PE | 0.2370 |
| 12 | Nossa Sra. de Lourdes | SE | 0.2361 |
| 13 | Canhoba | SE | 0.2341 |
| 14 | Sanharó | PE | 0.2325 |
| 15 | Malhada dos Bois | SE | 0.2312 |

Among the top 20 municipalities by TTI, 19 are in the Northeast (Sergipe, Pernambuco, Alagoas, Bahia). This is the most salient result of the municipal ranking.

### 6.3 Robustness across aggregation methods

The state ranking can be obtained via at least two paths:

- **Path A:** direct pixel→state aggregation via `reduceRegions`.
- **Path C:** pixel→municipality→state aggregation (mean of municipal means within each state).

**Table 3 — Comparison between the two paths (top 5):**

| State | Path A | Path C | Difference |
|---|---|---|---|
| Sergipe | 0.1460 | 0.1486 | +0.003 |
| Alagoas | 0.1426 | 0.1375 | −0.005 |
| Pernambuco | 0.1135 | 0.1263 | +0.013 |
| Espírito Santo | 0.0907 | 0.0916 | +0.001 |
| Bahia | 0.0846 | 0.0900 | +0.005 |

The convergence between the two methods — with differences below 0.015 — is empirical evidence of the multi-scale robustness of the index when the analysis scale is kept uniform.

### 6.4 Regular 25 km grid

For a presentation independent of administrative boundaries, TTI was also aggregated over a regular 25 km × 25 km grid (SIRGAS 2000 / Brazil Polyconic projection, EPSG:5880), by IBGE region. This aggregation mitigates the *Modifiable Areal Unit Problem* (MAUP) associated with aggregation by administrative units of highly heterogeneous sizes. The result confirms the spatial pattern observed in the administrative aggregations.

---

## 7. Case study: São Francisco River / Remanso–BA

To illustrate the index behaviour at the pixel level, we present a case study centered on Remanso, northern Bahia, in the Sobradinho Lake region of the São Francisco River.

**AOI:** 30 km radius around (−42.10; −9.62).

**Statistics:**

| Statistic | Value | Ratio vs. Brazil |
|---|---|---|
| Mean TTI | 0.1326 | 3.1× |
| Median | 0.0679 | 2.2× |
| P90 | 0.3697 | 4.3× |
| P99 | 0.6358 | 3.0× |
| Maximum | 0.6898 | — |

The AOI exhibits mean TTI more than three times higher than the national mean, with a significantly heavier upper tail. Visual investigation in historical Google Earth imagery confirms that this region underwent extreme water body retraction during 2017–2024, associated with the São Francisco water crisis. The event was captured by TTI without the use of any water mask or prior classification.

This case demonstrates the utility of the index for detecting changes that do not follow classical urban patterns: the NDVI variation in the AOI is negative (riparian vegetation loss), but there is no new construction, no detectable economic activity via nightlights, and the land cover change does not qualify as deforestation in the conventional sense. A class-specific detector would struggle to register the event. TTI registers it.

---

## 8. Validation

Validation of the index operates at four complementary levels.

### 8.1 Point-wise validation (qualitative controls)

Four control points with clear qualitative expectation were tested:

| Point | Expectation | Observed TTI |
|---|---|---|
| Feira de Santana/BA | high (urban expansion) | 0.162 |
| Barreiras/BA | medium (agricultural frontier) | 0.042 |
| Salvador | low (consolidated capital) | 0.032 |
| Itaparica (island) | low (stable area) | 0.028 |

Observed values are consistent with qualitative expectations, confirming that the index correctly discriminates stable areas from areas undergoing transformation.

### 8.2 Quantitative Sentinel-2 protocol

An independent validation protocol was applied to 36 candidate points identified by the index as high transformation. The validation cross-references NDVI variations (Sentinel-2 Harmonized, 10 m) and SAR VV (Sentinel-1, 10 m) between 2019 and 2023, with 500 m analysis radius per point. The criteria are:

- **Class A (strong evidence):** NDVI_diff > 0.05 **and** SAR_diff > 0.5 dB
- **Class B (medium evidence):** one of the two
- **Class C (weak evidence):** neither

**Result:** 17 points (47.2%) in class A; 12 (33.3%) in class B; 7 (19.4%) in class C. In 80.5% of candidates, independent sources agree on at least one sensor.

### 8.3 MATOPIBA calibrator case

The point (−8.99; −46.35), in central Maranhão, was inspected in historical Google Earth imagery. The 2008 image shows dense cerrado vegetation; the 2024 image shows pivot circles and industrial agricultural expansion. TTI registers this point as pronounced transformation, a visually unequivocal correspondence.

### 8.4 São Francisco pixel-level case

Presented in Section 7. Demonstration of capture of an extreme hydrological event without the use of a water mask.

---

## 9. Discussion

### 9.1 Counter-intuitive finding: Northeastern dominance

The most salient result of the state ranking is the dominance of Sergipe, Alagoas and Pernambuco in the top positions, with mean TTI significantly higher than that of states historically associated with the agricultural frontier. Mato Grosso, Rondônia and Pará occupy intermediate or low positions. Amazonas ranks 27th.

This pattern contradicts the expectation that "territorial transformation" is synonymous with "Amazon deforestation". TTI measures something else: total semantic transformation, which includes — but is not reduced to — forest cover loss.

### 9.2 Plausible hypothesis: renewable energy expansion

A plausible hypothesis for Northeastern dominance is the **expansion of renewable energy infrastructure** (wind and photovoltaic) in the region between 2017 and 2024. Public ANEEL data indicate installed capacity in GW during the period for Bahia (≈ 10.5 GW), Rio Grande do Norte (≈ 7.6 GW), Pernambuco (≈ 1.7 GW) and other Northeastern states. Plant installation significantly modifies the territorial vectorial signature: it introduces geometrically regular physical structures (panels, towers, substations, access roads), alters soil texture and reflectance, and is often accompanied by native vegetation recovery in the vicinity of the plant. Semantic embeddings should capture this combination as pronounced transformation.

Quantitative confirmation of this hypothesis (regression TTI ~ MW installed per municipality, controlling for area, urbanization and vegetation change) is explicitly left as future work. The hypothesis is presented as a candidate explanation, not an established result.

### 9.3 Alternative or complementary hypotheses

Three non-exclusive hypotheses merit investigation:

1. **Extreme climate variability.** The Northeast underwent severe drought associated with the 2015–2016 El Niño followed by recovery. Hydric stress cycles may generate significant temporal variations in the semantic signature of the caatinga biome.

2. **Regional heterogeneity of the caatinga.** The caatinga is a transitional biome with high intra-annual variability. Globally-trained embeddings may be more sensitive to natural variations in the Northeast than in more stable biomes.

3. **MATOPIBA underestimation.** The conversion of cerrado vegetation into agriculture may produce smaller embedding variation than the installation of permanent physical structures.

### 9.4 What the index is, and what it is not

TTI does **not substitute** PRODES, MapBiomas or specific spectral indices. It **complements** those systems by answering a different question: *where did territorial transformation occur, with no restriction on the type of transformation?*

The index should be interpreted as a **detector** of transformation, not as a **classifier**. A high TTI value indicates that something changed significantly; identifying *what* changed is a subsequent task, typically involving visual inspection or cross-reference with auxiliary sources.

---

## 10. Limitations

**Restricted temporal horizon.** The *SATELLITE_EMBEDDING V1* dataset covers only 2017–2024. Long-cycle analyses are not possible within this implementation.

**National asset scale.** The `GW_STT_2017_2024` raster was aggregated to 2000 m due to GEE memory constraints. The native embedding resolution is 10 m; fine-scale aggregations require localized recomputation.

**No covariate correction.** Associations between TTI and specific drivers (renewables, urbanization, deforestation) are presented as hypotheses; no controlled multivariate regression was performed.

**Validation N = 36.** The Sentinel-2 protocol covers 36 candidate points, selected from high-TTI regions. Selection bias toward high magnitude exists; the false-positive rate at random points was not estimated.

**Untreated seasonal variability.** The annual embeddings integrate the calendar year, but mosaic quality differences between years (especially 2017) may produce residual noise. Preliminary analysis identified ≈ 161 municipalities with anomalously high values in the 2017→2018 pair — these cases were documented separately and are not included in the main analysis based on the 2017→2024 pair.

**Causal interpretation.** The index detects transformation, not the cause of transformation. Associations between high TTI and specific drivers are hypotheses to be confirmed with additional methods.

---

## 11. Data and code availability

Under CC-BY 4.0 license, available on Zenodo (DOI: pending):

- Aggregated national TTI raster (GeoTIFF, 2000 m, Brazil 2017–2024).
- `TTI_estadual_final.csv` — 27 states, mean TTI and percentiles.
- `TTI_municipal_fixed.csv` — 5,506 municipalities, mean TTI.
- `TTI_grid_25km_{north,northeast,central,southeast,south}.csv` — regular grid by IBGE region.
- `TTI_pixel_sample_brasil.csv` — random sample N = 10,000, seed = 42.
- `GhostWorks_Validacao_S2.csv` — 36 candidates, three evidence classes.
- Reproducible Google Colab notebook for Google Earth Engine.

---

## 12. Future work

- Multivariate regression TTI ~ drivers (ANEEL renewables, urbanization, Hansen deforestation, climate).
- Global expansion (the *SATELLITE_EMBEDDING* dataset is global; the methodology transfers without modification).
- Annual time series (consecutive pairs) for analysis of *when* transformation occurred.
- Unsupervised anomaly detector over the national TTI raster.
- Stratified validation with N > 200 random points to estimate false-positive rate.

---

## Acknowledgements

The author thanks the *Google Earth Engine* project and *AlphaEarth Foundations* (Google + DeepMind) for publicly releasing the *SATELLITE_EMBEDDING V1* dataset, without which this work would be unfeasible in an independent setting.

## Conflict of interest

The author declares no conflict of interest.

## Funding

This work was conducted without institutional funding, funding agency or private sponsorship.

---

*Preprint v0.6 — EN version.*
*LuxVerso Research Initiative — April 2026.*
