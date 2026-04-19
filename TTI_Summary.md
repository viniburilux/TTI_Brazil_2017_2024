# TTI — Summary para Preprint v1
**Gerado em:** 2026-04-19T13:07:39.140685
**Asset GEE:** `projects/project-2715df31-12cf-4aed-908/assets/GW_STT_2017_2024`

## 1. Metadata do raster
- Banda: `stt` (1 banda, float32)
- CRS: EPSG:4326
- Bounds: longitude -74.02 a -28.82, latitude -33.82 a 5.32
- Escala nativa: 10m (SATELLITE_EMBEDDING V1)
- Cobertura: Brasil continental

## 2. Distribuição pixel-level (N=10.000, Brasil)
- N válido: 10000
- Média: 0.0429
- Desvio-padrão: 0.0423
- Mínimo: 0.0039
- Mediana: 0.0303
- P90: 0.0866
- P99: 0.2093
- Máximo: 0.6108

## 3. Ranking estadual (pixel→UF direto, 27 UFs)

|   ADM1_CODE | estado              |   stt_mean |   stt_stdDev |   stt_p50 |   stt_p90 |   stt_p99 |
|------------:|:--------------------|-----------:|-------------:|----------:|----------:|----------:|
|         690 | Sergipe             |     0.146  |       0.0624 |    0.1396 |    0.2293 |    0.3057 |
|         666 | Alagoas             |     0.1426 |       0.0594 |    0.1397 |    0.2217 |    0.2918 |
|         681 | Pernambuco          |     0.1135 |       0.0571 |    0.1006 |    0.1962 |    0.2861 |
|         672 | Espirito Santo      |     0.0907 |       0.0325 |    0.085  |    0.1279 |    0.2023 |
|         669 | Bahia               |     0.0846 |       0.0521 |    0.0723 |    0.1465 |    0.2557 |
|         683 | Rio De Janeiro      |     0.0739 |       0.0284 |    0.0693 |    0.1104 |    0.1573 |
|         679 | Paraiba             |     0.0732 |       0.0377 |    0.0675 |    0.122  |    0.1767 |
|         676 | Mato Grosso Do Sul  |     0.0689 |       0.0475 |    0.0566 |    0.1152 |    0.2754 |
|         680 | Parana              |     0.0651 |       0.0306 |    0.0606 |    0.0995 |    0.1698 |
|         684 | Rio Grande Do Norte |     0.0623 |       0.0334 |    0.0527 |    0.1035 |    0.1622 |
|         687 | Roraima             |     0.0591 |       0.0537 |    0.0449 |    0.1114 |    0.2868 |
|         685 | Rio Grande Do Sul   |     0.0568 |       0.0358 |    0.0488 |    0.0957 |    0.1893 |
|         688 | Santa Catarina      |     0.0565 |       0.0284 |    0.0527 |    0.08   |    0.1736 |
|         677 | Minas Gerais        |     0.0561 |       0.022  |    0.0537 |    0.0791 |    0.128  |
|         689 | Sao Paulo           |     0.0537 |       0.0336 |    0.0449 |    0.0878 |    0.1815 |
|         682 | Piaui               |     0.0456 |       0.0276 |    0.04   |    0.0713 |    0.1591 |
|         674 | Maranhao            |     0.0444 |       0.0304 |    0.0371 |    0.0683 |    0.1738 |
|         670 | Ceara               |     0.0412 |       0.0275 |    0.0371 |    0.0644 |    0.1231 |
|         675 | Mato Grosso         |     0.0374 |       0.05   |    0.0253 |    0.0722 |    0.2717 |
|         686 | Rondonia            |     0.0347 |       0.0551 |    0.0214 |    0.0606 |    0.3299 |
|         671 | Distrito Federal    |     0.0332 |       0.0162 |    0.0318 |    0.0503 |    0.093  |
|         673 | Goias               |     0.0322 |       0.0215 |    0.0253 |    0.0526 |    0.1034 |
|         678 | Para                |     0.0315 |       0.0416 |    0.0214 |    0.0566 |    0.2129 |
|         691 | Tocantins           |     0.0286 |       0.0183 |    0.0253 |    0.0487 |    0.0997 |
|         665 | Acre                |     0.0255 |       0.0228 |    0.0175 |    0.0449 |    0.1189 |
|         667 | Amapa               |     0.0249 |       0.026  |    0.0135 |    0.0527 |    0.127  |
|         668 | Amazonas            |     0.0182 |       0.0309 |    0.0135 |    0.0252 |    0.1113 |

## 4. Nota metodológica

A agregação pixel→UF preserva área como unidade de medida.
A agregação pixel→município→UF introduz viés de média-de-médias.
Por isso o ranking estadual final usa agregação direta pixel→UF.

## 5. Estudo de caso — Remanso/BA (Rio São Francisco)
- STT médio (AOI 30km): 0.1326
- STT mediana: 0.0679
- STT P90: 0.3697
- STT P99: 0.6358
- STT máximo: 0.6898

## 6. Datasets utilizados
- Raster STT (GW_STT_2017_2024)
- Pixel sample Brasil (N=10.000)
- Ranking estadual (27 UFs)
- Validação Sentinel-2 N=36
- Caso calibrador MATOPIBA

## 7. Outputs gerados
- `TTI_estadual_final.csv`
- `TTI_pixel_sample_brasil.csv`
- `TTI_Summary.md`