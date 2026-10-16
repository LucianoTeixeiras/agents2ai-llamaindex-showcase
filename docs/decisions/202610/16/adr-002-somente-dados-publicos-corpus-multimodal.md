# ADR-002 — Somente dados públicos: corpus multimodal e Chinook em DuckDB

| Campo | Valor |
|-------|-------|
| **Status** | Proposto (2026-10-16) — aguardando aceite de Luciano |
| **Data** | 2026-10-16 |
| **Decisores** | Luciano Teixeira (aceite) · Dev Grok Bot (proposta) |
| **Jira** | pendente — D-01 |
| **Relacionados** | PRD e System Design `docs/contracts/202610/16/`; LangGraph showcase ADR-002 |

## Contexto (2026-10-16)
Repositório público. Precisamos de PDF real (com tabelas), DOCX, PPTX, XLSX, imagens e um banco relacional, todos com licença clara. URLs abaixo verificadas em 2026-10-16 (HTTP 200).

## Decisão
| Fonte | Formato | Uso | Licença | URL | Versionado? |
|-------|---------|-----|---------|-----|-------------|
| Chinook Database v1.4.5 (Luis Rocha) | SQLite → DuckDB | NL→SQL e base dos sintéticos | MIT | https://github.com/lerocha/chinook-database/releases/download/v1.4.5/Chinook_Sqlite.sqlite | não (baixado por script) |
| IBGE — *Brasil em Números* v.31 (2023) | PDF (~11,7 MB), texto + tabelas, pt-BR | RAG em PDF com tabelas | reprodução permitida citando a fonte (termos de uso do IBGE) | https://biblioteca.ibge.gov.br/visualizacao/periodicos/2/bn_2023_v31.pdf | não |
| Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks* (arXiv:2005.11401) | PDF em inglês | RAG em PDF técnico/multilíngue | arXiv non-exclusive distribution 1.0 (não redistribuir) | https://arxiv.org/pdf/2005.11401 | **não**, só download sob demanda |
| Documentos sintéticos gerados de Chinook (`scripts/make_synthetic_docs.py`) | DOCX (relatório), PPTX (apresentação com notas), XLSX (2 abas), PNG (gráfico) e PNG "escaneado" (página renderizada) | cobrir todos os formatos de forma reproduzível | MIT (derivados de Chinook MIT) | gerados localmente | só fixtures pequenos em `tests/fixtures/` |

Colunas PII do Chinook (`Email`, `Phone`, `Fax`, `Address`, `PostalCode`) são bloqueadas pelo guardrail mesmo sendo fictícias (demonstração). `data/manifest.json` registra URL, sha256 e licença de cada arquivo.

## Alternativas consideradas
| Alternativa | Prós | Contras | Por que não |
|-------------|------|---------|-------------|
| **Recomendada:** Chinook + IBGE + arXiv sob demanda + sintéticos | licenças claras, pt-BR, todos os formatos | sintéticos são "limpos" | — |
| Olist (Kaggle) | e-commerce BR real | CC BY-NC-SA e login Kaggle | licença/automação |
| CF/88 PDF do Senado | jurídico, pt-BR | URL testada devolveu HTML (2026-10-16) | não verificável |
| thelook_ecommerce (como no LangGraph showcase) | coerência | exige GCP na fase 1 | fica para fase 2 |

## Consequências
Nenhum PDF de terceiros vai para o Git; CI usa só fixtures sintéticos.
