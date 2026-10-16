# SPEC-LI-05 — Criar pipeline de ingestão, embeddings e vector store

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-05 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §4 · ADR-005 |
| **Jira** | [A2LI-6](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-6) (épico A2LI-1) |
| **Branch** | `feat/li-05-pipeline-indice-vetorial` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-04 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
`ingest_folder(path, settings)` lê com `READERS`, aplica `SentenceSplitter(512, 64)` + embeddings, deduplica por sha256 e persiste em DuckDB (`DuckDBVectorStore`) ou Qdrant; `load_index(settings)` reabre o índice; `search(query, k, file_type=None)` retorna `NodeWithScore` com metadados.

## 2. Fora de escopo
Roteamento, síntese, SQL.

## 3. Contexto mínimo
`from llama_index.core.ingestion import IngestionPipeline`; `from llama_index.core.node_parser import SentenceSplitter`; `from llama_index.vector_stores.duckdb import DuckDBVectorStore` (import conferido em 0.7.0; parâmetros do construtor **não verificados** — confira assinatura e escale se divergir). Filtro: `MetadataFilters([ExactMatchFilter(key="file_type", value=...)])`. Testes com `MockEmbedding(embed_dim=8)`.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `pyproject.toml` / `uv.lock` | alterar | + `llama-index-vector-stores-duckdb==0.7.0`, `llama-index-vector-stores-qdrant==0.10.4` |
| `src/showcase/ingest/pipeline.py` | criar | `ingest_folder`, `load_index` |
| `src/showcase/retrieval.py` | criar | `search` |
| `tests/test_pipeline.py` | criar | §6 |

## 5. Passo a passo
Dependências → pipeline → retrieval → testes → portões → PR.

## 6. Testes
Mín. 6: ingestão dos fixtures cria nós com metadados; reingestão não duplica (sha256); `search` retorna k resultados; filtro `file_type=pptx` só traz slides; índice reabre após fechar; Qdrant só com `VECTOR_STORE=qdrant` (teste com cliente em memória `location=':memory:'`).

## 7. Definição de Pronto (DoD)
- [ ] Só os arquivos da §4 mudaram (`git diff --stat main`).
- [ ] `uv sync --locked --group dev`, `uv run ruff check .`, `uv run pytest -q` verdes localmente e no CI (incl. gitleaks).
- [ ] Nenhum teste com `skip`/`xfail` novo (exceto marcador `ocr` previsto na ADR-004); nenhuma chamada de rede, Ollama, LlamaParse ou LLM real nos testes.
- [ ] PR aberto (não mergeado) com o corpo da §10.

## 8. Ações proibidas
- Commitar em `main`, fazer merge, force-push, `git add -A`/`git add .`.
- Adicionar/atualizar dependências além das listadas na §4; usar `pip install`.
- Tocar arquivos fora da §4; reformatar/renomear arquivos vizinhos.
- Incluir chaves, `.env`, documentos ou dados de cliente; versionar PDFs de terceiros ou `data/raw/`.
- Afrouxar guardrail, allowlist ou HITL para um teste passar.

## 9. Regra de escalonamento
**Se a spec for ambígua, pare e pergunte; não improvise.** Pare se: um símbolo/API do LlamaIndex não existir como descrito (versões em System Design, cabeçalho); um teste só passar mudando algo fora da §4; o diff passar do limite; surgir decisão de produto, segurança, custo, licença ou dependência. Abra o PR em rascunho e comente citando a seção da spec.

## 10. PR
Título: ver abaixo. Corpo: Spec · O que foi feito · Evidência dos portões (saída real) · Desvios · Checklist DoD · Risco QA.

Título: `feat(indice): pipeline de ingestão, bge-m3 e vector store DuckDB [A2LI-6]`
