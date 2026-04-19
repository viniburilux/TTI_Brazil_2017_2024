# Territorial Transformation Index (TTI): detecção agnóstica de transformação territorial a partir de embeddings semânticos de satélite

**Aplicação nacional ao Brasil, 2017–2024**

---

**Autor:** Vinicius Buri¹
**Afiliação:** ¹LuxVerso Research Initiative — Salvador, Bahia, Brasil
**ORCID:** 0009-0000-6006-1516
**Contato:** viniburilux@gmail.com
**Versão:** Preprint v0.6 (PT-BR)
**Data:** Abril de 2026

---

## Resumo

Este trabalho introduz o **Territorial Transformation Index (TTI)** — um índice escalar, agnóstico a rótulos e invariante à unidade espacial, definido sobre embeddings semânticos anuais de satélite. O TTI é calculado como o complemento da similaridade de cosseno entre vetores de embedding de dois anos distintos para a mesma localização, fornecendo uma medida contínua da dissimilaridade semântica entre dois estados do território. Diferente de índices espectrais como NDVI/NDWI, que respondem a propriedades físicas específicas, o TTI opera sobre representações aprendidas de 64 dimensões produzidas pelo modelo *AlphaEarth Foundations* (Google + DeepMind) e captura transformação territorial sem depender de classes predefinidas ou supervisão rotulada. Apresentamos uma implementação nacional do TTI para o Brasil no período 2017–2024, construída inteiramente sobre o Google Earth Engine. A distribuição pixel-level (N = 10.000 amostras aleatórias sobre o Brasil continental) apresenta mediana de 0,030, P99 de 0,209 e máximo de 0,865, com perfil *heavy-tail* consistente com sistemas territoriais complexos. A agregação por unidade administrativa revela um padrão espacial contraintuitivo: estados do Nordeste brasileiro (Sergipe, Alagoas, Pernambuco) lideram a transformação territorial média, enquanto estados historicamente associados à fronteira agrícola (Mato Grosso em 19º, Amazonas em 27º) ocupam posições intermediárias ou baixas. Demonstramos a robustez multiescalar do índice através de três métodos de agregação (pixel→UF direto, pixel→município→UF, e grid regular 25 km), os quais convergem quando a escala de análise é mantida uniforme. Validamos o índice por quatro mecanismos: (i) quatro pontos de controle qualitativos; (ii) protocolo quantitativo com 36 candidatos validados por Sentinel-2 (NDVI + SAR); (iii) caso calibrador com *ground truth* visual no MATOPIBA; e (iv) estudo de caso pixel-level no Rio São Francisco (Remanso/BA), cujo TTI máximo de 0,69 correspondeu à retração extrema do lago de Sobradinho. Discutimos como hipótese plausível a associação entre alto TTI no Nordeste e a expansão de infraestrutura de energia renovável (eólica e fotovoltaica) na região durante o período, sem estabelecer causalidade. O índice, o dataset nacional e o raster Brasil 2017–2024 são disponibilizados abertamente sob licença CC-BY 4.0.

**Palavras-chave:** transformação territorial; embeddings semânticos; sensoriamento remoto; *change detection*; Google Earth Engine; Brasil; energia renovável.

---

## 1. Introdução

Sistemas de monitoramento territorial convencionais — PRODES, MapBiomas, índices espectrais como NDVI/NDWI — operam sobre classes pré-definidas ou propriedades físicas específicas. Isso lhes confere grande poder dentro do domínio para o qual foram projetados (tipicamente vegetação, corpos d'água, área construída) e uma limitação estrutural fora dele: são cegos para transformações que não foram explicitamente codificadas.

Este trabalho parte de uma pergunta diferente: **e se, em vez de classificar o que o território é, fosse possível medir o quanto ele se tornou irreconhecível em relação a si mesmo?**

A proposta operacionaliza essa pergunta como uma métrica de distância semântica entre dois estados temporais do mesmo território, apoiada em embeddings aprendidos de satélite. Enquanto um índice espectral pergunta *"quão verde é este pixel?"*, o índice aqui proposto pergunta *"quão diferente este pixel é agora do que ele era antes?"* — sem restringir o tipo de diferença.

As contribuições deste trabalho são:

1. **Definição formal** do *Territorial Transformation Index* (TTI), uma métrica escalar de transformação territorial baseada em similaridade de cosseno entre embeddings semânticos anuais.
2. **Implementação nacional** do TTI sobre o Brasil (2017–2024), via Google Earth Engine, com o raster resultante publicado como *asset* reutilizável.
3. **Caracterização empírica** da distribuição do índice a partir de amostragem pixel-level.
4. **Demonstração de robustez multiescalar** através de três métodos de agregação (pixel→UF direto, pixel→município→UF, e grid regular 25 km).
5. **Protocolo de validação** em quatro níveis: controle qualitativo, protocolo quantitativo Sentinel-2, caso calibrador com *ground truth* visual, e estudo de caso pixel-level.
6. **Achado empírico contraintuitivo** sobre a geografia da transformação territorial no Brasil: dominância do Nordeste, com hipótese associada à expansão de infraestrutura de energia renovável.

---

## 2. Conceito: o que é o TTI

O TTI é um índice escalar que responde à pergunta: *"quanto o estado semântico deste território mudou entre o tempo t₁ e o tempo t₂?"*

Três propriedades definem o índice:

**Agnosticismo a rótulos.** O TTI não requer classes temáticas (floresta, urbano, agricultura). Qualquer transformação que altere a representação vetorial do território é captada — seja urbanização, instalação de infraestrutura energética, conversão agrícola, degradação ambiental, retração de corpo d'água ou qualquer combinação.

**Invariância à unidade espacial.** O índice é definido pixel a pixel. Agregações em grid regular, malha administrativa ou recortes arbitrários são **operações de apresentação**, não propriedades do índice. Esta distinção é central: o TTI produz *um único raster contínuo* sobre o qual qualquer geometria pode ser consultada.

**Continuidade.** O índice varia suavemente entre estabilidade total (TTI = 0) e transformação máxima (TTI = 2, embora na prática os valores observados raramente excedam 0,9). Não há limiar fixo; a definição de "transformação significativa" é uma escolha de aplicação, não do índice.

---

## 3. Fonte de dados

O dataset fundamental é o **GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL**, disponível publicamente no Google Earth Engine. Produzido pelo modelo *AlphaEarth Foundations* (Google + DeepMind), o dataset representa cada pixel de 10 m da superfície terrestre por um vetor de 64 dimensões que integra séries anuais de Sentinel-1 (radar), Sentinel-2 (óptico), Landsat e fontes auxiliares. **Os vetores são L2-normalizados por construção (magnitude unitária)**, o que os torna diretamente comparáveis via produto escalar. O dataset cobre globalmente o período 2017–2024 (inclusive) com cadência anual.

Esta é a única fonte primária utilizada no cálculo do TTI. Fontes auxiliares (FAO/GAUL para fronteiras administrativas; Sentinel-2 e Sentinel-1 para validação independente) são explicitadas adiante.

---

## 4. Método

### 4.1 Definição formal

Para qualquer localização espacial *x* e dois anos *t₁* e *t₂*, seja *E(x, t) ∈ ℝ⁶⁴* o vetor de embedding L2-normalizado associado ao pixel *x* no ano *t*. O *Territorial Transformation Index* é definido como:

$$\text{TTI}(x, t_1, t_2) = 1 - \frac{E(x, t_1) \cdot E(x, t_2)}{\|E(x, t_1)\| \cdot \|E(x, t_2)\|}$$

Como os embeddings são L2-normalizados (‖E‖ = 1), a expressão se reduz a:

$$\text{TTI}(x, t_1, t_2) = 1 - E(x, t_1) \cdot E(x, t_2)$$

O índice satisfaz TTI ∈ [0, 2], com valores próximos a 0 indicando estabilidade semântica e valores elevados indicando transformação pronunciada.

### 4.2 Implementação Brasil 2017–2024

A implementação nacional utiliza como pares temporais:

- *E₁ = mosaic(SATELLITE_EMBEDDING, 2017-01-01, 2018-01-01)*
- *E₂ = mosaic(SATELLITE_EMBEDDING, 2024-01-01, 2025-01-01)*

O par 2017 vs 2024 foi escolhido por **maximizar o sinal cumulativo** dentro do intervalo disponível do dataset, capturando transformação acumulada de sete anos e maximizando a razão sinal-ruído para mudanças persistentes.

O cálculo é pixel-level em escala nativa de 10 m:

```python
emb_2017 = dataset.filterDate('2017-01-01', '2018-01-01').mosaic()
emb_2024 = dataset.filterDate('2024-01-01', '2025-01-01').mosaic()
dot     = emb_2017.multiply(emb_2024).reduce(ee.Reducer.sum())
tti     = ee.Image(1).subtract(dot)  # embeddings já L2-normalizados
```

Por restrição de memória dos *workers* do Google Earth Engine para operações nacionais, o raster resultante foi **agregado para escala de 2000 m via reamostragem bilinear** e salvo como *asset* público no projeto GEE (`GW_STT_2017_2024`). O asset cobre a extensão continental do Brasil (longitude −74,02 a −28,82; latitude −33,82 a 5,32). A validação pontual (Seção 8) confirma que a agregação para 2000 m preserva a capacidade discriminatória do índice em escalas municipais e superiores. Para análises em escalas mais finas, o cálculo pode ser reexecutado localmente sobre recortes menores em resolução nativa.

### 4.3 Ambiente computacional

O pipeline foi construído e executado em ambiente computacional restrito — smartphone + Google Colab — sem uso de estação de trabalho dedicada ou recursos computacionais pagos além do nível gratuito padrão do Google Earth Engine. Esta circunstância é relatada explicitamente porque reforça uma propriedade prática do método: **baixa barreira para replicação**.

---

## 5. Distribuição empírica

Para caracterizar a distribuição do TTI sobre o Brasil continental, foi realizada amostragem aleatória uniforme de N = 10.000 pixels, com semente fixa (seed = 42) para reprodutibilidade, em subamostragem de 500 m justificada pela suavidade espacial do sinal TTI em escalas acima de ≈ 100 m. *Esta subamostragem não altera as propriedades distribucionais do índice*, como confirmado pela comparação com o cálculo direto sobre o raster nacional completo (média amostral: 0,0429; média do raster: 0,0435; P99 amostral: 0,209; P99 do raster: 0,217).

**Estatísticas principais (N = 10.000):**

| Estatística | Valor |
|---|---|
| Média | 0,0429 |
| Desvio-padrão | 0,0423 |
| Mínimo | 0,0039 |
| Mediana (P50) | 0,0303 |
| P75 | 0,0551 |
| P90 | 0,0866 |
| P95 | 0,1182 |
| P99 | 0,2093 |
| Máximo (amostral) | 0,6108 |

**Observações sobre a distribuição:**

1. **Perfil *heavy-tail*.** A razão entre P99 e mediana é de ≈ 7×, e entre máximo amostral e mediana é de ≈ 20×. A grande maioria do território brasileiro apresenta estabilidade semântica entre 2017 e 2024, com transformações pronunciadas concentradas em pequenas frações do espaço.

2. **Posicionamento de limiares.** Valores acima de ≈ 0,09 (P90) já representam transformação acima da tendência geral. Valores acima de ≈ 0,21 (P99) representam o 1% mais transformado do território e correspondem a transformações de magnitude notável. O intervalo [0; 0,03] é interpretável como "ruído de fundo" do índice.

3. **Comportamento esperado do método.** Distribuições *heavy-tail* são características de sistemas geográficos complexos, nos quais a maior parte do espaço é estável e uma minoria concentra a dinâmica. O fato de o TTI produzir esse perfil de forma não supervisionada é um indicativo de que o índice captura o tipo de sinal que um detector de mudança territorial deveria capturar.

---

## 6. Agregação espacial

Embora o TTI seja definido a nível de pixel, agregações em unidades espaciais maiores são úteis para comunicação e para cruzamento com dados administrativos. Apresentamos três agregações que, em conjunto, demonstram a robustez multiescalar do índice.

### 6.1 Agregação por unidade da federação (pixel→UF direto)

Cada uma das 27 unidades da federação teve o TTI agregado diretamente a partir do raster, via `reduceRegions` em escala uniforme de 2000 m, com *matching* administrativo realizado por código numérico GAUL (ADM1_CODE).

**Tabela 1 — Ranking estadual pelo TTI médio (pixel→UF direto):**

| # | Estado | TTI médio | Desvio-padrão | P50 | P90 | P99 |
|---|---|---|---|---|---|---|
| 1 | Sergipe | 0,1460 | 0,0624 | 0,1396 | 0,2293 | 0,3057 |
| 2 | Alagoas | 0,1426 | 0,0594 | 0,1397 | 0,2217 | 0,2918 |
| 3 | Pernambuco | 0,1135 | 0,0571 | 0,1006 | 0,1962 | 0,2861 |
| 4 | Espírito Santo | 0,0907 | 0,0325 | 0,0850 | 0,1279 | 0,2023 |
| 5 | Bahia | 0,0846 | 0,0521 | 0,0723 | 0,1465 | 0,2557 |
| 6 | Rio de Janeiro | 0,0739 | 0,0284 | 0,0693 | 0,1104 | 0,1573 |
| 7 | Paraíba | 0,0732 | 0,0377 | 0,0675 | 0,1220 | 0,1767 |
| 8 | Mato Grosso do Sul | 0,0689 | 0,0475 | 0,0566 | 0,1152 | 0,2754 |
| 9 | Paraná | 0,0651 | 0,0306 | 0,0606 | 0,0995 | 0,1698 |
| 10 | Rio Grande do Norte | 0,0623 | 0,0334 | 0,0527 | 0,1035 | 0,1622 |
| 11 | Roraima | 0,0591 | 0,0537 | 0,0449 | 0,1114 | 0,2868 |
| 12 | Rio Grande do Sul | 0,0568 | 0,0358 | 0,0488 | 0,0957 | 0,1893 |
| 13 | Santa Catarina | 0,0565 | 0,0284 | 0,0527 | 0,0800 | 0,1736 |
| 14 | Minas Gerais | 0,0561 | 0,0220 | 0,0537 | 0,0791 | 0,1280 |
| 15 | São Paulo | 0,0537 | 0,0336 | 0,0449 | 0,0878 | 0,1815 |
| 16 | Piauí | 0,0456 | 0,0276 | 0,0400 | 0,0713 | 0,1591 |
| 17 | Maranhão | 0,0444 | 0,0304 | 0,0371 | 0,0683 | 0,1738 |
| 18 | Ceará | 0,0412 | 0,0275 | 0,0371 | 0,0644 | 0,1231 |
| 19 | Mato Grosso | 0,0374 | 0,0500 | 0,0253 | 0,0722 | 0,2717 |
| 20 | Rondônia | 0,0347 | 0,0551 | 0,0214 | 0,0606 | 0,3299 |
| 21 | Distrito Federal | 0,0332 | 0,0162 | 0,0318 | 0,0503 | 0,0930 |
| 22 | Goiás | 0,0322 | 0,0215 | 0,0253 | 0,0526 | 0,1034 |
| 23 | Pará | 0,0315 | 0,0416 | 0,0214 | 0,0566 | 0,2129 |
| 24 | Tocantins | 0,0286 | 0,0183 | 0,0253 | 0,0487 | 0,0997 |
| 25 | Acre | 0,0255 | 0,0228 | 0,0175 | 0,0449 | 0,1189 |
| 26 | Amapá | 0,0249 | 0,0260 | 0,0135 | 0,0527 | 0,1270 |
| 27 | Amazonas | 0,0182 | 0,0309 | 0,0135 | 0,0252 | 0,1113 |

### 6.2 Agregação municipal

A agregação municipal foi calculada para os 5.506 municípios brasileiros (via FAO/GAUL/2015/level2, *matching* por ADM2_CODE), em escala uniforme de 2000 m.

**Tabela 2 — Top 15 municípios por TTI médio:**

| # | Município | UF | TTI médio |
|---|---|---|---|
| 1 | Cumbe | SE | 0,2770 |
| 2 | São Francisco | SE | 0,2629 |
| 3 | Cachoeirinha | PE | 0,2616 |
| 4 | Tacaimbó | PE | 0,2447 |
| 5 | Manari | PE | 0,2422 |
| 6 | Paranatama | PE | 0,2412 |
| 7 | Calçado | PE | 0,2410 |
| 8 | Capoeiras | PE | 0,2392 |
| 9 | Itabi | SE | 0,2380 |
| 10 | Caetés | PE | 0,2380 |
| 11 | Itaíba | PE | 0,2370 |
| 12 | Nossa Sra. de Lourdes | SE | 0,2361 |
| 13 | Canhoba | SE | 0,2341 |
| 14 | Sanharó | PE | 0,2325 |
| 15 | Malhada dos Bois | SE | 0,2312 |

Entre os 20 municípios de maior TTI, 19 estão no Nordeste (Sergipe, Pernambuco, Alagoas, Bahia). Este é o resultado mais saliente do ranking municipal.

### 6.3 Robustez entre métodos de agregação

O ranking estadual pode ser obtido por pelo menos dois caminhos:

- **Caminho A:** agregação direta pixel→UF via `reduceRegions`.
- **Caminho C:** agregação pixel→município→UF (média das médias municipais da UF).

**Tabela 3 — Comparação entre os dois caminhos (top 5):**

| Estado | Caminho A | Caminho C | Diferença |
|---|---|---|---|
| Sergipe | 0,1460 | 0,1486 | +0,003 |
| Alagoas | 0,1426 | 0,1375 | −0,005 |
| Pernambuco | 0,1135 | 0,1263 | +0,013 |
| Espírito Santo | 0,0907 | 0,0916 | +0,001 |
| Bahia | 0,0846 | 0,0900 | +0,005 |

A convergência entre os dois métodos — com divergências inferiores a 0,015 — é evidência empírica da robustez multiescalar do índice quando a escala de análise é mantida uniforme.

### 6.4 Grid regular de 25 km

Para uma apresentação independente de fronteiras administrativas, o TTI foi também agregado em grid regular de 25 km × 25 km (projeção SIRGAS 2000 / Brasil Polyconic, EPSG:5880), por região do IBGE. Esta agregação, quando visualizada, mitiga o *Modifiable Areal Unit Problem* (MAUP) associado à agregação por unidades administrativas de tamanhos muito heterogêneos. O resultado confirma o padrão espacial observado nas agregações administrativas.

---

## 7. Estudo de caso: Rio São Francisco / Remanso–BA

Para ilustrar o comportamento do índice em nível pixel-level, apresentamos um estudo de caso centrado em Remanso, no norte da Bahia, região do lago de Sobradinho no Rio São Francisco.

**AOI:** círculo de 30 km de raio centrado em (−42,10; −9,62).

**Estatísticas:**

| Estatística | Valor | Razão vs. Brasil |
|---|---|---|
| TTI médio | 0,1326 | 3,1× |
| Mediana | 0,0679 | 2,2× |
| P90 | 0,3697 | 4,3× |
| P99 | 0,6358 | 3,0× |
| Máximo | 0,6898 | — |

A AOI apresenta TTI médio mais de três vezes superior à média nacional, com cauda superior significativamente mais pesada. Investigação visual em Google Earth histórico confirma que esta região sofreu retração extrema do corpo d'água no período 2017–2024, associada à crise hídrica do São Francisco. O evento foi capturado pelo TTI sem uso de máscara de água ou classificação prévia.

Este caso demonstra a utilidade do índice para detecção de mudanças que não seguem padrões urbanos clássicos: a variação de NDVI na AOI é negativa (perda de vegetação ripária), mas não há construção nova, não há atividade econômica detectável por *nightlights*, e a mudança de cobertura não é desmatamento no sentido convencional. Um detector baseado em classes específicas teria dificuldade de registrar o evento. O TTI registra.

---

## 8. Validação

A validação do índice opera em quatro níveis complementares.

### 8.1 Validação pontual (controles qualitativos)

Quatro pontos de controle com expectativa qualitativa clara foram testados:

| Ponto | Expectativa | TTI observado |
|---|---|---|
| Feira de Santana/BA | alto (urbano em expansão) | 0,162 |
| Barreiras/BA | médio (fronteira agrícola) | 0,042 |
| Salvador | baixo (capital consolidada) | 0,032 |
| Itaparica (ilha) | baixo (área estável) | 0,028 |

Os valores observados são consistentes com as expectativas qualitativas, confirmando que o índice discrimina corretamente áreas estáveis de áreas em transformação.

### 8.2 Protocolo quantitativo via Sentinel-2

Um protocolo de validação independente foi aplicado a 36 pontos candidatos identificados pelo índice como de alta transformação. A validação cruza variações de NDVI (Sentinel-2 Harmonized, 10 m) e SAR VV (Sentinel-1, 10 m) entre 2019 e 2023, com raio de análise de 500 m por ponto. Os critérios são:

- **Classe A (evidência forte):** NDVI_diff > 0,05 **e** SAR_diff > 0,5 dB
- **Classe B (evidência média):** um dos dois
- **Classe C (evidência fraca):** nenhum

**Resultado:** 17 pontos (47,2 %) em classe A; 12 (33,3 %) em classe B; 7 (19,4 %) em classe C. Em 80,5 % dos candidatos, fontes independentes concordam em pelo menos um sensor.

### 8.3 Caso calibrador MATOPIBA

O ponto (−8,99; −46,35), no Maranhão central, foi inspecionado no Google Earth histórico. A imagem de 2008 mostra vegetação densa de cerrado; a imagem de 2024 mostra círculos de pivô e expansão agrícola industrial. O TTI registra este ponto como transformação pronunciada, correspondência visualmente inequívoca.

### 8.4 Caso pixel-level São Francisco

Apresentado na Seção 7. Demonstração de captura de evento hidrológico extremo sem uso de máscara de água.

---

## 9. Discussão

### 9.1 Achado contraintuitivo: dominância nordestina

O resultado mais saliente do ranking estadual é a dominância de Sergipe, Alagoas e Pernambuco nas primeiras posições, com TTI médio significativamente superior ao de estados historicamente associados à fronteira agrícola. Mato Grosso, Rondônia e Pará ocupam posições intermediárias ou baixas. Amazonas aparece em 27º.

Este padrão contraria a expectativa de que "transformação territorial" seja sinônimo de "desmatamento amazônico". O TTI mede outra coisa: transformação semântica total, que inclui — mas não se reduz a — perda de cobertura florestal.

### 9.2 Hipótese plausível: expansão de energia renovável

Uma hipótese plausível para a dominância nordestina é a **expansão de infraestrutura de energia renovável** (eólica e fotovoltaica) na região entre 2017 e 2024. Dados públicos da ANEEL indicam capacidade instalada em GW no período para Bahia (≈ 10,5 GW), Rio Grande do Norte (≈ 7,6 GW), Pernambuco (≈ 1,7 GW) e outros estados nordestinos. A instalação de parques modifica significativamente a assinatura vetorial do território: introduz estruturas físicas geometricamente regulares (painéis, torres, subestações, acessos), altera a textura e a refletância do solo, e em muitos casos é acompanhada de recuperação da vegetação nativa no entorno do parque. Embeddings semânticos devem capturar esta combinação como transformação pronunciada.

A confirmação quantitativa desta hipótese (regressão TTI ~ MW instalados por município, controlando por área, urbanização e mudança de vegetação) é deixada explicitamente como trabalho futuro. A hipótese é apresentada como explicação candidata, não como resultado estabelecido.

### 9.3 Hipóteses alternativas ou complementares

Três hipóteses não excludentes merecem investigação:

1. **Variabilidade climática extrema.** O Nordeste experimentou seca severa associada ao El Niño 2015–2016 seguida de recuperação. Ciclos de estresse hídrico podem gerar variações temporais significativas na assinatura semântica da caatinga.

2. **Heterogeneidade regional da caatinga.** A caatinga é um bioma de transição com alta variabilidade intra-anual. Embeddings treinados em escala global podem ser mais sensíveis a variações naturais no Nordeste do que em biomas mais estáveis.

3. **Subestimação do MATOPIBA.** A conversão de vegetação de cerrado para agricultura pode produzir variação de embedding menor do que a instalação de estruturas físicas permanentes.

### 9.4 O que o índice é, o que não é

O TTI **não substitui** PRODES, MapBiomas ou índices espectrais específicos. Ele **complementa** esses sistemas ao responder a uma pergunta diferente: *onde ocorreu transformação territorial, sem restrição do tipo de transformação?*

O índice deve ser interpretado como **detector** de transformação, não como **classificador**. Um TTI elevado indica que algo mudou significativamente; identificar *o que* mudou é tarefa subsequente, usualmente envolvendo inspeção visual ou cruzamento com fontes auxiliares.

---

## 10. Limitações

**Horizonte temporal restrito.** O dataset *SATELLITE_EMBEDDING V1* cobre apenas 2017–2024. Análises de ciclos longos não são possíveis dentro desta implementação.

**Escala do asset nacional.** O raster `GW_STT_2017_2024` foi agregado para 2000 m por restrição de memória do GEE. A resolução nativa dos embeddings é 10 m; agregações em escalas finas requerem recálculo localizado.

**Ausência de correção por covariáveis.** As associações entre TTI e drivers específicos (renováveis, urbanização, desmatamento) são apresentadas como hipóteses; não foi realizada regressão multivariada controlada.

**Validação N = 36.** O protocolo Sentinel-2 cobre 36 pontos candidatos, selecionados em regiões de alto TTI. Há *selection bias* para magnitude elevada; a taxa de falsos positivos em pontos aleatórios não foi estimada.

**Variabilidade sazonal não tratada.** Os embeddings anuais integram o ano civil, mas diferenças de qualidade do mosaico entre anos (especialmente 2017) podem produzir ruído residual. Análise preliminar identificou ≈ 161 municípios com valores anomalamente altos no par 2017→2018 — esses casos foram documentados separadamente e não entram na análise principal baseada no par 2017→2024.

**Interpretação causal.** O índice detecta transformação, não a causa da transformação. Associações entre alto TTI e drivers específicos são hipóteses a confirmar com métodos adicionais.

---

## 11. Disponibilidade de dados e código

Sob licença CC-BY 4.0, disponibilizamos no Zenodo (DOI: a obter):

- Raster nacional TTI agregado (GeoTIFF, 2000 m, Brasil 2017–2024).
- `TTI_estadual_final.csv` — 27 UFs, TTI médio e percentis.
- `TTI_municipal_fixed.csv` — 5.506 municípios, TTI médio.
- `TTI_grid_25km_{north,northeast,central,southeast,south}.csv` — grid regular por região IBGE.
- `TTI_pixel_sample_brasil.csv` — amostra aleatória N = 10.000, seed = 42.
- `GhostWorks_Validacao_S2.csv` — 36 candidatos, três classes de evidência.
- Notebook Colab reprodutível via Google Earth Engine.

---

## 12. Trabalho futuro

- Regressão multivariada TTI ~ drivers (renováveis ANEEL, urbanização, desmatamento Hansen, clima).
- Expansão global (o dataset *SATELLITE_EMBEDDING* é global; a metodologia transfere sem modificação).
- Série temporal anual (pares consecutivos) para análise do *quando* da transformação.
- Detector de anomalia não supervisionado sobre o raster TTI nacional.
- Validação estratificada com N > 200 pontos aleatórios para estimar taxa de falsos positivos.

---

## Agradecimentos

O autor agradece ao projeto *Google Earth Engine* e ao *AlphaEarth Foundations* (Google + DeepMind) por disponibilizarem publicamente o dataset *SATELLITE_EMBEDDING V1*, sem o qual este trabalho seria inviável em ambiente independente.

## Conflito de interesses

O autor declara ausência de conflito de interesses.

## Financiamento

Este trabalho foi conduzido sem financiamento institucional, agência de fomento ou patrocínio privado.

---

*Preprint v0.6 — versão PT-BR.*
*LuxVerso Research Initiative — Abril de 2026.*
