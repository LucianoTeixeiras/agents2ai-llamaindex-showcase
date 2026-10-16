# SPEC-LI-09 — Adicionar tracing, evals de RAG/SQL/roteamento/adversarial e README

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-09 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §9 · ADR-007 |
| **Jira** | [A2LI-10](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-10) (épico A2LI-1) |
| **Branch** | `feat/li-09-tracing-evals-readme` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-08 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
Decorador `traced` por passo e log JSON; OpenInference/OTel opcional por env; módulo `showcase.evals` com golden sets e relatório (hit rate@k, MRR, faithfulness, acurácia SQL, roteamento, adversarial, p50/p95); README de vitrine com diagrama e "como rodar" em ≤ 3 comandos.

## 2. Fora de escopo
Dashboard próprio; Ragas/DeepEval.

## 3. Contexto mínimo
`from llama_index.core.evaluation import FaithfulnessEvaluator, RelevancyEvaluator` (só no modo `--online`). OpenInference: `LlamaIndexInstrumentor().instrument()` só se `OTEL_EXPORTER_OTLP_ENDPOINT` definido. Golden sets mínimos: `evals/rag.yaml` 20, `evals/sql.yaml` 20, `evals/routing.yaml` 20, `evals/adversarial.yaml` 8 (inclui documento sintético malicioso em `tests/fixtures/`).

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `src/showcase/tracing.py` | criar | `traced`, logger JSON |
| `src/showcase/evals/__init__.py`, `__main__.py`, `metrics.py` | criar | runner e métricas |
| `evals/*.yaml` | criar | golden sets |
| `pyproject.toml` / `uv.lock` | alterar | + `pyyaml>=6`, grupo opcional `otel` com `openinference-instrumentation-llama-index==4.6.0`, `opentelemetry-exporter-otlp>=1.27` |
| `README.md` | alterar | vitrine |
| `tests/test_tracing.py`, `tests/test_evals.py` | criar | §6 |

## 5. Passo a passo
Tracing → métricas → golden sets → runner → README → testes → portões → PR.

## 6. Testes
Mín. 8: trace tem um item por passo executado; log JSON sem prompt/texto de documento; hit rate e MRR calculados certo em exemplo fixo; acurácia SQL compara resultados; adversarial: 0 execução perigosa; runner offline roda no CI em < 60 s.

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

Título: `feat(evals): tracing por passo e evals de RAG e SQL [A2LI-10]`
