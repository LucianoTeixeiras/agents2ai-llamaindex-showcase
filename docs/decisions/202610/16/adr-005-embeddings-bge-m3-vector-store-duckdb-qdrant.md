# ADR-005 — Embeddings bge-m3 local e vector store DuckDB (Qdrant opcional)

| Campo | Valor |
|-------|-------|
| **Status** | Proposto (2026-10-16) — aguardando aceite de Luciano |
| **Data** | 2026-10-16 |
| **Decisores** | Luciano Teixeira (aceite) · Dev Grok Bot (proposta) |
| **Jira** | [A2LI-12](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-12) (aceite das ADRs, épico A2LI-1) |
| **Relacionados** | PRD e System Design `docs/contracts/202610/16/`; LangGraph showcase ADR-006 (DuckDB vss) |

## Contexto (2026-10-16)
Corpus pt-BR + inglês, execução local, demo simples de rodar. O LangGraph showcase já usa DuckDB `vss` para schema retrieval.

## Decisão
1. **Embeddings recomendados:** `bge-m3` (multilíngue, 1024 dim) servido pelo Ollama via `llama-index-embeddings-ollama==0.10.0`. Alternativa sem Ollama: `BAAI/bge-m3` via `llama-index-embeddings-huggingface==0.8.0`.
2. **Vector store recomendado:** DuckDB via `llama-index-vector-stores-duckdb==0.7.0`, arquivo `data/index/vectors.duckdb` (fora do Git); mesmo motor das tabelas SQL, zero serviço extra. Uso de índice HNSW (`vss`) **não verificado** nesse integration; se ausente, busca exata é suficiente para o corpus de demo.
3. **Opcional:** Qdrant (`VECTOR_STORE=qdrant`, profile do compose) para mostrar caminho de escala e filtros ricos.
4. Testes: `MockEmbedding(embed_dim=8)` e store em memória.

## Alternativas consideradas
| Alternativa | Prós | Contras | Por que não |
|-------------|------|---------|-------------|
| **Recomendada:** bge-m3 + DuckDB | local, multilíngue, 1 arquivo | escala limitada | — |
| bge-m3 + Qdrant padrão | escala, híbrido sparse | serviço extra | fica opcional (D-04) |
| `nomic-embed-text` | leve | inglês melhor que pt-BR | qualidade pt-BR |
| pgvector | padrão de mercado | Postgres extra | não necessário |
| Chroma | popular | pouco diferencial | — |

## Consequências
Reindexar ao trocar modelo de embedding (dim registrada no manifest do índice).
