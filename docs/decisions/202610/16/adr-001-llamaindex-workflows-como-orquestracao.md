# ADR-001 — LlamaIndex Workflows como orquestração (AgentWorkflow como modo opcional)

| Campo | Valor |
|-------|-------|
| **Status** | Proposto (2026-10-16) — aguardando aceite de Luciano |
| **Data** | 2026-10-16 |
| **Decisores** | Luciano Teixeira (aceite) · Dev Grok Bot (proposta) |
| **Jira** | [A2LI-12](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-12) (aceite das ADRs, épico A2LI-1) |
| **Relacionados** | PRD e System Design `docs/contracts/202610/16/`; LangGraph showcase ADR-001 |

## Contexto (2026-10-16)
O produto é RAG sobre documentos heterogêneos + SQL. LlamaIndex é o framework mais focado em ingestão/recuperação; em 27 de 730 vagas de IA verificadas no LinkedIn (2026-10-16) aparece ao lado de LangChain/LangGraph. SLMs locais erram mais em tool calling livre.

## Decisão
1. `llama-index-core==0.14.25` com **Workflows** (eventos e passos explícitos) como orquestração padrão: roteamento, regeneração de SQL e HITL determinísticos e testáveis com `MockLLM`.
2. **Recomendado:** workflow próprio `CopilotWorkflow`. `AgentWorkflow` + `FunctionAgent` com as mesmas tools (`search_docs`, `run_sql`) fica como flag `--agent`, para modelos fortes.
3. Sem LangChain no runtime (o paralelo com LangGraph é o outro showcase).

## Alternativas consideradas
| Alternativa | Prós | Contras | Por que não |
|-------------|------|---------|-------------|
| **Recomendada:** Workflows determinístico + AgentWorkflow opcional | testável, robusto com SLM, mostra as duas APIs | mais código que um agente pronto | — |
| Só `AgentWorkflow` (ReAct/Function) | menos código | instável com SLM 3–7B | meta O6 em risco |
| LlamaIndex + LangGraph | combinação comum no mercado | duplica o outro showcase | foco |
| `RouterQueryEngine` puro | simples | sem HITL nem retry | requisitos RF-07/08 |

## Consequências
Versões fixadas no `uv.lock`; API de Workflows muda rápido → specs marcam o que foi verificado.
