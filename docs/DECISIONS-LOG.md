# Log de decisões

## 2026-10-16 — Planejamento inicial (PR docs/planejamento-inicial)
Contexto: Luciano escolheu **showcase público e separado**, somente dados públicos, **não** fundido ao `agent-kit-ged`.
- **ADR-001** LlamaIndex Workflows como orquestração; AgentWorkflow opcional (**Proposto**).
- **ADR-002** Somente dados públicos: Chinook (MIT), IBGE *Brasil em Números* 2023, arXiv:2005.11401 sob demanda, sintéticos MIT (**Proposto**).
- **ADR-003** SLM local via Ollama; fallback OpenAI-compatível (**Proposto**).
- **ADR-004** Readers locais por padrão; LlamaParse opcional com HITL (**Proposto**).
- **ADR-005** Embeddings `bge-m3` local + DuckDB vector store; Qdrant opcional (**Proposto**).
- **ADR-006** Guardrails: SQL só leitura, documento como dado, HITL (**Proposto**).
- **ADR-007** Trace por passo, OpenInference/OTel opcional, evals RAG+SQL (**Proposto**).
- Design system claro de alto contraste (AAA) derivado do copilot: `docs/design/design-system.md`.

## 2026-10-16 — Jira (D-01 decidida)
- **D-01** Projeto Jira **`A2LI`** criado por Luciano (Kanban team-managed). Épico **A2LI-1**.

| Spec | Jira |
|------|------|
| SPEC-LI-01 fundação | A2LI-2 |
| SPEC-LI-02 corpus/manifest | A2LI-3 |
| SPEC-LI-03 readers PDF/DOCX | A2LI-4 |
| SPEC-LI-04 readers PPTX/XLSX/imagens | A2LI-5 |
| SPEC-LI-05 índice vetorial | A2LI-6 |
| SPEC-LI-06 guardrail + NL→SQL | A2LI-7 |
| SPEC-LI-07 roteador/citações/híbrido | A2LI-8 |
| SPEC-LI-08 HITL/CLI/Docker | A2LI-9 |
| SPEC-LI-09 tracing/evals/README | A2LI-10 |
| SPEC-LI-10 fase 2 BigQuery (**bloqueada**) | A2LI-11 |
| Aceite ADRs 001–007 e D-02…D-06 | A2LI-12 |

## Decisões pendentes (Luciano)
- **D-02** SLM padrão e hardware de referência.
- **D-03** Harness: Cursor + Composer ou Antigravity.
- **D-04** Vector store padrão: DuckDB (recomendado) ou Qdrant.
- **D-05** Destino OTel (sugestão Langfuse local).
- **D-06** Aceite das ADRs 001…007.
