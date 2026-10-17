# ADR-003 — LLM: SLM local via Ollama por padrão, fallback OpenAI-compatível

| Campo | Valor |
|-------|-------|
| **Status** | Aceita (2026-10-16) — aceita por Luciano |
| **Data** | 2026-10-16 |
| **Decisores** | Luciano Teixeira (aceite) · Dev Grok Bot (proposta) |
| **Jira** | [A2LI-12](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-12) (aceite das ADRs, épico A2LI-1) |
| **Relacionados** | PRD e System Design `docs/contracts/202610/16/`; copilot model_factory; LangGraph showcase ADR-003 |

## Contexto (2026-10-16)
Soberania de dados é argumento central para jurídico/GED. O copilot já usa Ollama `qwen2.5:3b-instruct` como padrão.

## Decisão
1. `LLM_PROVIDER=ollama` (padrão) com `llama-index-llms-ollama==0.11.0`; **recomendado** `qwen2.5:7b-instruct` com GPU ≥ 8 GB e `qwen2.5:3b-instruct` em CPU (D-02 em aberto).
2. Fallback `openai_like` (`llama-index-llms-openai-like==0.8.1`): OpenRouter, Groq, vLLM, LM Studio, etc. **Padrão do fallback:** `nvidia/nemotron-3.5-lightning` via OpenRouter (mesmo do agents2ai-langgraph-showcase); chave só em `.env`.
3. Roteamento e geração de SQL usam saída estruturada (Pydantic) com `temperature=0`.
4. Testes usam `MockLLM`; nenhuma chamada real no CI.

## Alternativas consideradas
| Alternativa | Prós | Contras | Por que não |
|-------------|------|---------|-------------|
| **Recomendada:** Ollama + OpenAI-like | local, custo zero, troca simples | SLM erra mais | — |
| Só API (OpenRouter) | qualidade | dado sai da máquina | contra a tese |
| vLLM/llama.cpp direto | performance | setup pesado para demo | coberto via OpenAI-like |
| Llama 3.2 3B / Phi-4-mini / Gemma 3 | alternativas válidas | menos testadas em pt-BR aqui | avaliar nos evals |

## Consequências
Metas O4/O6 calibradas pelo SLM escolhido em D-02.
