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

## Decisões pendentes (Luciano)
- **D-01** Chave Jira (sugestão `A2LI`); até lá `[pendente]`.
- **D-02** SLM padrão e hardware de referência.
- **D-03** Harness: Cursor + Composer ou Antigravity.
- **D-04** Vector store padrão: DuckDB (recomendado) ou Qdrant.
- **D-05** Destino OTel (sugestão Langfuse local).
- **D-06** Aceite das ADRs 001…007.
