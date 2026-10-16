# PRD — Copilot RAG multimodal + NL→SQL em LlamaIndex sobre dados públicos (showcase)

| Campo | Valor |
|-------|-------|
| **Status** | Proposto (2026-10-16) — aguardando aceite de Luciano |
| **Autor (arquiteto)** | Dev Grok Bot (planner) |
| **Dono do produto** | Luciano Teixeira |
| **Jira (épico)** | [A2LI-1](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-1) (projeto `A2LI`, Kanban team-managed) |
| **Nível de risco QA** | `qa:risk-medium` (ingere documentos de terceiros, gera e executa SQL; repositório público) |
| **Documentos relacionados** | `system-design-copilot-rag-multimodal-llamaindex.md`, ADR-001…007, specs `spec-li-01…10`, `docs/design/design-system.md` |

## 1. Problema
O núcleo de valor do produto Agents2AI é responder perguntas sobre **documentos heterogêneos** (PDF, Word, PowerPoint, Excel, imagens) **e** bancos de dados, com fonte citada. O `agents2ai-data-copilot` (Pydantic AI) cobre hoje só planilhas e SQL, sem RAG vetorial. LlamaIndex é o framework de referência em ingestão e recuperação e aparece em vagas de AI Engineer ao lado de LangGraph. Falta uma vitrine aberta que mostre RAG multimodal + SQL seguro, rodando **local** com SLM (requisito típico de jurídico, GED e contratos, onde o dado não pode sair do cliente).

## 2. Objetivo e métricas de sucesso

| ID | Objetivo | Métrica | Linha de base | Alvo | Como medir |
|----|----------|---------|---------------|------|------------|
| O1 | Respostas de documentos fundamentadas | faithfulness (juiz) no golden set RAG | n/a | ≥ 0,85 | `uv run python -m showcase.evals rag` |
| O2 | Recuperação boa | hit rate@5 / MRR no golden set RAG | n/a | ≥ 0,80 / ≥ 0,60 | idem |
| O3 | Citações verificáveis | respostas de documento com ≥ 1 citação válida (arquivo + página/slide/aba) | n/a | 100% | `tests/test_citations.py` + evals |
| O4 | SQL correto | acurácia de execução no golden set SQL | n/a | ≥ 75% (SLM local padrão) | `uv run python -m showcase.evals sql` |
| O5 | Zero SQL perigoso | escrita/DDL/multi-statement que passa o guardrail | n/a | 0 | `tests/test_guardrails.py` |
| O6 | Roteamento certo | pergunta → rota (`docs`/`sql`/`hybrid`/`out_of_scope`) | n/a | ≥ 90% | evals de roteamento |
| O7 | Reprodutível e soberano | CI verde sem rede, sem chave e sem Ollama; demo roda offline | n/a | sim | CI (MockLLM/MockEmbedding) |
| O8 | Vitrine | README com diagrama/GIF e "como rodar" em ≤ 3 comandos | n/a | sim | revisão do arquiteto |

## 3. Usuários e cenários
- Como **recrutador técnico**, quero ver em 5 minutos um pipeline de ingestão multimodal e um workflow de agente legível.
- Como **cliente potencial (jurídico/GED)**, quero perguntar "o que a Constituição diz sobre X?" ou "qual o ranking de vendas por gênero no relatório?" e ver a resposta com **arquivo e página citados**.
- Como **analista**, quero perguntar "quais os 5 países com maior faturamento?" e ver o SQL executado e as linhas.
- Como **operador**, quero aprovar ou negar consultas grandes e o envio de um arquivo para parsing em nuvem.
- Como **dev avaliador**, quero rodar evals de RAG e SQL e ver métricas e latência por etapa.

## 4. Escopo
### 4.1 Dentro do escopo
- CLI: `uv run showcase ingest <pasta>`, `uv run showcase ask "<pergunta>"` (com aprovação interativa).
- Ingestão: PDF (texto + tabelas), DOCX, PPTX (texto por slide + notas), XLSX (aba como tabela e como texto), imagens (OCR local; legenda por modelo de visão opcional). Readers locais por padrão; LlamaParse opcional por flag + HITL.
- Metadados por nó: `file_name`, `file_type`, `page`/`slide`/`sheet`, `sha256`, `source_url`, `license`.
- Embeddings locais `bge-m3` (Ollama) e vector store DuckDB `vss` (padrão) ou Qdrant (opcional).
- Ferramenta NL→SQL sobre DuckDB (Chinook) com guardrail sqlglot próprio (não confiar no engine).
- Workflow/AgentWorkflow com roteamento `docs` | `sql` | `hybrid` | `out_of_scope`, síntese com citações e HITL.
- Evals offline (fakes) e online opcionais (Ollama); tracing por etapa em log JSON; OpenTelemetry/OpenInference opcional.
- Fase 2: BigQuery público (`bigquery-public-data.*`) com dry-run de custo (SPEC-LI-10, Bloqueada).

### 4.2 Fora do escopo (explícito)
- UI web e API HTTP (fase 3, seguindo `docs/design/design-system.md`); autenticação multiusuário; multi-tenant.
- Qualquer documento, dado ou nome de cliente; escrita em qualquer banco.
- Fine-tuning; LlamaCloud Index gerenciado; LlamaDeploy.
- Portar código do `agents2ai-data-copilot` ou do `agent-kit-ged` (vitrine independente).

## 5. Requisitos funcionais
- **RF-01** Ingerir uma pasta, detectar tipo por extensão + assinatura, rejeitar tipos/tamanhos fora da allowlist.
- **RF-02** Extrair texto e estrutura por formato, preservando página/slide/aba nos metadados.
- **RF-03** OCR de imagens e de páginas PDF sem camada de texto.
- **RF-04** Chunking com `SentenceSplitter` (tamanho/sobreposição configuráveis); ingestão idempotente por `sha256`.
- **RF-05** Indexar embeddings e recuperar top-k com filtro por metadados (tipo de arquivo).
- **RF-06** Classificar a pergunta em rota; `out_of_scope` recusa sem chamar tools.
- **RF-07** Gerar SQL no dialeto do backend, validar no guardrail, até 2 regenerações com feedback; depois recusa.
- **RF-08** Pausar para aprovação humana quando: linhas estimadas > `MAX_ROWS_WITHOUT_APPROVAL`, BigQuery acima de `MAX_BYTES_WITHOUT_APPROVAL` (fase 2) ou uso de LlamaParse.
- **RF-09** Responder em pt-BR com citações `[n]` ligadas a `arquivo · página/slide/aba`; rota SQL exibe SQL e até 50 linhas; rota híbrida combina ambas.
- **RF-10** Registrar trace por etapa (nome, ms, tokens, k recuperados) no resultado e em log JSON.

## 6. Requisitos não funcionais
- Python 3.12, uv, ruff, pytest; CI sem segredos e sem rede; gitleaks.
- Latência p95 < 20 s com SLM local em CPU/GPU modesta (medida nos evals online; depende de D-02).
- Ingestão do corpus de demo < 10 min em notebook sem GPU.
- Custo zero por padrão (tudo local); nuvem só por opt-in.

## 7. Segurança e privacidade
Ver ADR-006. Somente dados públicos (ADR-002); segredos só em `.env`; documento é dado, não instrução; SQL só leitura; PII bloqueada; envio a nuvem só com HITL.

## 8. Riscos
| Risco | Mitigação |
|-------|-----------|
| Prompt injection dentro de documento | delimitação de contexto, tools só por decisão do workflow, guardrail SQL determinístico, eval adversarial |
| SLM pequeno erra roteamento/tool calling | roteador com saída estruturada + fallback por regras; workflow determinístico em vez de agente livre (ADR-001) |
| API do LlamaIndex muda rápido | versões fixadas no `uv.lock`; ADR-001 registra versões verificadas |
| OCR/tabelas de baixa qualidade | evals por formato; LlamaParse opcional |
| Licença de corpus | só fontes com licença registrada; arXiv baixado sob demanda, não versionado |

## 9. Decisões em aberto
- **D-01** ~~Chave Jira~~ **Decidido (2026-10-16):** projeto `A2LI`, épico A2LI-1; specs LI-01…LI-10 = A2LI-2…A2LI-11; aceite das ADRs = A2LI-12.
- **D-02** SLM padrão e hardware de referência (sugestão: `qwen2.5:7b-instruct` com GPU ≥ 8 GB; `qwen2.5:3b-instruct` em CPU). Afeta metas O4/O6 e latência.
- **D-03** Harness de execução: Cursor + Composer (como no LangGraph showcase) ou Antigravity.
- **D-04** Vector store padrão: DuckDB `vss` (recomendado) ou Qdrant (ADR-005).
- **D-05** Destino OTel (sugestão: Langfuse local, alinhado ao copilot e ao LangGraph showcase).
- **D-06** Aceite das ADRs 001…007 (todas **Propostas**).
