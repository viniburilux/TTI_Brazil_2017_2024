# Leitura estratégica — TTI_Brazil_2017_2024

**Data:** 17 de agosto de 2026  |  **Repositório:** [TTI_Brazil_2017_2024](https://github.com/viniburilux/TTI_Brazil_2017_2024)  |  **Autor:** Manus AI

> Este documento é uma auditoria de inventário e potencial. Ele não altera o código existente e não afirma que funcionalidades foram executadas ou validadas quando isso não aparece na evidência observada.

## Síntese executiva

Repositório de pesquisa que implementa e distribui o Territorial Transformation Index (TTI) aplicado ao Brasil (2017–2024). Contém o preprint (PDF/MD), um notebook reproduzível para Google Earth Engine (Colab), um raster nacional (GW_STT_2017_2024.tif), conjuntos agregados (estados, municípios, grids regionais), uma amostra de pixels (N=10.000) e um pequeno conjunto de validação (36 pontos) com evidência em Sentinel‑2. Estrutura e documentação orientam reprodução via GEE; não há sinais de infraestrutura de produção, testes automatizados ou pipelines fora do notebook Colab.

## Domínio e propósito aparente

Detecção e quantificação de transformação territorial usando embeddings anuais de satélite (AlphaEarth / GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL). Domínio: sensoriamento remoto, monitoramento de mudanças territoriais e análise multi‑escala (pixel → município → estado → grade regular). Público‑alvo principal: pesquisadores, analistas geoespaciais e iniciativas de pesquisa aplicada (p.ex. LuxVerso/GhostWorks).

## Indicadores do snapshot

| Indicador | Valor |
|---|---:|
| Arquivos contabilizados | 22 |
| Tamanho no snapshot | 26376051 bytes |
| Último commit observado | 6c1f4788644227e2803bb49c0d5be309d69f6ce1	2026-04-19T15:08:05-04:00	Initial commit: TTI Brazil 2017-2024 project structure and files |
| Prioridade sugerida | alta |

## Evidências observadas

- README.md descreve formulação matemática do índice (TTI = 1 − dot(E_t1, E_t2)) e lista do conteúdo do repositório.
- paper/TTI_preprint_v0.6_EN.pdf e paper/TTI_preprint_v0.6_PT.pdf: preprint completo em inglês e português (presumível metodologia, resultados e discussões).
- code/TTI_pipeline_colab.ipynb: notebook Colab com etapas de autenticação GEE, cálculo do TTI, amostragem de pixels, agregação municipal e estadual, e exportação para Drive.
- raster/GW_STT_2017_2024.tif: raster nacional em 2000 m incluído no repositório (arquivo grande presente).
- datasets/TTI_estadual_final.csv, datasets/TTI_municipal_fixed.csv, datasets/TTI_grid_25km_*.csv e datasets/TTI_pixel_sample_brasil.csv: produtos derivados e agregados disponibilizados em CSV.
- validation/GhostWorks_Validacao_S2.csv: arquivo de validação com 36 candidatos e alegada validação via Sentinel‑2.
- TTI_pixel_stats_brasil.json contém estatísticas resumidas (mín, max, p50, p90, etc.) usadas no README.

## Ativos e capacidades

- Cálculo do TTI a partir de embeddings anuais em GEE (exemplo implementado no notebook Colab).
- Raster nacional TTI (GW_STT_2017_2024.tif) pronto para uso em análises geoespaciais a 2 km de resolução.
- Conjuntos agregados prontos para análise: estados (27), municípios (5.506), grades regionais (25 km) e amostra aleatória de pixels (N=10.000, seed=42).
- Documentação científica (preprint em PT/EN) que descreve metodologia e resultados, incluindo estudos de caso (Remanso/BA) e estatísticas de distribuição.
- Arquivo de validação (36 pontos) com comparações independentes em Sentinel‑2 e SAR (descrito no README e presente no diretório validation).
- Reprodutibilidade orientada via Google Colab: o notebook contém células executáveis e instruções para executar no GEE (inclui comandos ee.Authenticate()/ee.Initialize()).
- Licença CC‑BY 4.0 aplicada ao repositório (arquivo LICENSE presente).

## Maturidade observável

Pesquisa / protótipo em estágio avançado de documentação científica e entrega de artefatos, mas não maturado para produção. Evidências mostram: artigos e produtos finais (raster e CSVs) entregues; pipeline executável, porém encapsulado apenas em um notebook Colab. Falta automação (CI/CD), modularização de código (scripts reutilizáveis ou pacote), testes unitários/integrados, contêineres/infraestrutura de deploy, metadados técnicos formais (ex.: esquema/CRS/proveniência detalhada em arquivo machine‑readable) e política de governança de dados. Validação existe, mas é pequena (36 pontos) e indicada como candidata/experimental no README. Portanto: robustez reprodutível para usuários técnicos com acesso a GEE, porém baixo preparo operacional (documentado) para integração produtiva em pipelines de produção.

## Potencial de aproveitamento

- Integração com o ecossistema LuxVerso/GhostWorks como camada de detecção rápida de mudança territorial: o raster nacional e os CSVs permitem alimentar painéis, sistemas de alerta e análises exploratórias.
- Uso do notebook e da metodologia como base para automação de ingestão periódica (p.ex. monitoramento anual ou semi‑anual) aproveitando embeddings AlphaEarth em GEE.
- Combinação com workflows GhostWorks de validação (ex.: estender a validação Sentinel‑2/SAR, incorporar validação comunitária) para aumentar confiança e rotular exemplos para treinamento de modelos supervisionados.
- Criação de APIs/tile services a partir do raster (ex.: hospedagem em cloud storage + TileJSON) para integração em dashboards e serviços IA (monitoramento por município, alertas por thresholds).
- Aproveitamento dos datasets agregados para modelos estatísticos e análises causais mencionadas no preprint (p.ex. regressões multivariadas para testar hipóteses sobre infraestrutura renovável).
- Transformação do notebook em módulo reutilizável (Python package) para permitir execução fora do Colab e inclusão em pipelines CI/CD.

## Riscos e lacunas

- Dependência chave do dataset proprietário/terceiro (GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL): reprodutibilidade completa exige acesso público/estável ao asset GEE; mudanças no naming/versioning do asset podem quebrar reproduções.
- Validação limitada: apenas 36 pontos validados (arquivo validation) — insuficiente para aferir precisão global ou para diferentes tipologias de mudança; riscos de viés espacial (nordeste destacado no resumo) e de interpretação causal não verificada (autor indica hipóteses não estabelecidas).
- Ausência de testes automatizados, integração contínua e scripts CLI/contêineres: notebook é bom para reprodução manual, mas frágil para automação, reuso em produção e verificação regressiva.
- Falta de metadados machine‑readable e documentação técnica detalhada sobre CRS, resolução efetiva, tratamento de valores ausentes/no‑data, e política de agregação (por exemplo, como lidar com áreas cobertas por nuvens ou máscaras).
- Governança de dados e proveniência incompletas: README e paper dão descrições, mas não há manifest/Provenance JSON, versão DOI efetivo (placeholder no README), ou registro de transformações executadas para produzir os CSVs e TIFFs.
- Segurança e controle de acesso: o repositório é público e contém instruções para autenticar em GEE, mas não há diretrizes sobre credenciais GEE, gerenciamento de projetos, quotas ou práticas de acesso seguro (risco operacional ao incentivar uso de projeto pessoal sem melhores práticas).
- Licenciamento de entrada/saída: o trabalho está sob CC‑BY 4.0 (ok), mas artefatos derivados que incorporem dados de terceiros (p.ex. Google embeddings) devem observar termos de atribuição do fornecedor; isso não está formalizado no repositório além de menção no README.
- Escalabilidade operacional: cálculo de TTI em escala nacional exige GEE e possíveis quotas/limites de exportação; não há orientação sobre custos, limites de export ou processamento paralelo.

## Próximos passos recomendados

- Modularizar o pipeline: extrair o conteúdo do notebook para scripts Python moduláveis (p.ex. tti/core.py, tti/compute.py, tti/export.py) e fornecer um CLI mínimo para executar etapas (calc, sample, aggregate, export).
- Adicionar documentação técnica machine‑readable: incluir um arquivo METADATA.json com CRS, resolução nominal e efetiva, extensão geográfica, reprojeção aplicada, esquema dos CSVs (colunas, tipos), e hash/size dos artefatos principais (TIFF/CSV) para verificação de integridade.
- Implementar testes básicos e CI: criar testes unitários que validem funções matemáticas (dot product → TTI), pequenas execuções mockadas (unit test com arrays de embeddings), e um pipeline CI (GitHub Actions) que rode testes e verifica presença mínima de arquivos de saída.
- Publicar versão com DOI resolvido e manifest de versão: preencher DOI/Zenodo corretamente (README aponta placeholder) e versionar artefatos para rastreabilidade (ex.: v0.6 → v0.6.1 com changelog).
- Fortalecer a validação: ampliar amostragem de validação fora dos 36 pontos; documentar método de validação (protocolos de verificação em S2/SAR), adicionar metadados de confiança por ponto e estimativas de erro/precisão por tipologia de mudança.
- Documentar dependências operacionais e riscos de custeio: explicar requisitos GEE (asset names, quotas, projeto GCP associado se houver), estimativa de tempo/custo para exportar o raster em escala e recomendações sobre uso de contas de serviço com least privilege.
- Incluir proveniência completa: gerar e disponibilizar um registro de transformação (workflow provenance) para o raster e CSVs — p.ex. um METAPROV (JSON) contendo comandos/executions do GEE, versões de datasets de entrada, e hashes de outputs.
- Preparar artefatos para integração: exportar tiles (XYZ/Cloud-optimized GeoTIFF/COG) ou hospedar o TIFF em object storage para servir via TileJSON/WMTS, permitindo integração com dashboards e APIs GhostWorks/LuxVerso.
- Estabelecer política de atribuição/licenciamento operacional: documentar claramente como citar Google/AlphaEarth e FAO/GAUL ao reutilizar os derivados, incluindo notas sobre restrições de reuso (se houver) e conformidade com CC‑BY.
- Planejar uma versão ‘production‑ready’ roadmap: definir milestones (1) modularização + CI, (2) validação ampliada, (3) API/serviço de tiles, (4) integração com GhostWorks validation/labeling pipeline e (5) monitoramento periódico com alertas automatizados.

## Método e limites

A leitura foi feita sobre um snapshot de profundidade 1 e sobre arquivos textuais selecionados por relevância estrutural, incluindo README, manifests e amostras de código. Dependências, notebooks, binários, dados grandes e integrações externas podem exigir uma rodada posterior de execução controlada. Nenhum código do repositório foi executado durante a auditoria.

**Fonte primária:** [TTI_Brazil_2017_2024](https://github.com/viniburilux/TTI_Brazil_2017_2024)
