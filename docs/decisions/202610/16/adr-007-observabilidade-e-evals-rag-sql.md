# ADR-007 — Observabilidade e evals: trace por passo, OpenInference/OTel opcional, evals RAG+SQL

| Campo | Valor |
|-------|-------|
| **Status** | Proposto (2026-10-16) — aguardando aceite de Luciano |
| **Data** | 2026-10-16 |
| **Decisores** | Luciano Teixeira (aceite) · Dev Grok Bot (proposta) |
| **Jira** | pendente — D-01 |
| **Relacionados** | PRD e System Design `docs/contracts/202610/16/`; LangGraph showcase ADR-004; copilot ADR-009 (OTel) |

## Contexto (2026-10-16)
Recrutadores valorizam métricas visíveis de RAG (recuperação e fidelidade) e latência por etapa. CI não pode ter segredo nem rede.

## Decisão
1. Decorador `traced` por passo do workflow: `{"step", "ms", "tokens_in", "tokens_out", "k"}` no `trace`; uma linha JSON `INFO` por execução (logger `showcase`), sem prompt, sem texto de documento, sem SQL com literais.
2. **Opcional:** `openinference-instrumentation-llama-index==4.6.0` → OTLP (`OTEL_EXPORTER_OTLP_ENDPOINT`); destino em D-05 (sugestão Langfuse local).
3. **Evals próprios** (`src/showcase/evals/`), sem framework externo obrigatório:
   - RAG: hit rate@k e MRR (determinísticos, arquivo+página esperados); faithfulness e relevância via `FaithfulnessEvaluator`/`RelevancyEvaluator` do LlamaIndex com juiz configurável (online).
   - SQL: acurácia de execução (resultado igual ao SQL de referência).
   - Roteamento: acurácia da rota.
   - Adversarial: 0 SQL perigoso executado, 0 tool disparada por documento.
   - Latência p50/p95 por passo a partir do `trace`.
4. Offline no CI com fakes; online por `uv run python -m showcase.evals --online`.

## Alternativas consideradas
| Alternativa | Prós | Contras | Por que não |
|-------------|------|---------|-------------|
| **Recomendada:** trace próprio + OpenInference opcional + evals próprios | sem dependência, testável | menos visual | — |
| Ragas / DeepEval obrigatório | métricas prontas | dependências pesadas, LLM juiz | pode entrar depois |
| LlamaTrace/Arize Phoenix padrão | UI rica | serviço extra | opcional via OTel |

## Consequências
Golden sets versionados em `evals/*.yaml`, ligados ao manifest (sha256) para detectar corpus alterado.
