# SPEC-LI-01 — Criar a fundação: dependências, settings, fábrica de modelos e workflow com passos stub

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-01 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §2, §3, §8 · ADR-001, ADR-003 |
| **Jira** | pendente — D-01 |
| **Branch** | `feat/li-01-fundacao-workflow` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-low` |
| **Depende de** | PR de planejamento mergeado |
| **Tamanho máximo** | ≤ 300 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
Repositório passa a ter `Settings`, fábrica de LLM/embeddings (Ollama | OpenAI-like | Mock) e um `CopilotWorkflow` com os passos do System Design §2 em versão stub, executável com `MockLLM`.

## 2. Fora de escopo
Readers, índice, SQL real, HITL real, CLI.

## 3. Contexto mínimo
`from llama_index.core.workflow import Workflow, step, StartEvent, StopEvent, Event, Context` (conferido em 0.14.25). Passo: `@step async def route(self, ctx: Context, ev: StartEvent) -> RouteEvent`. Execução: `await wf.run(question="...")`. Mocks: `from llama_index.core.llms import MockLLM`; `from llama_index.core.embeddings import MockEmbedding`.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `pyproject.toml` / `uv.lock` | alterar | + `llama-index-core==0.14.25`, `llama-index-llms-ollama==0.11.0`, `llama-index-llms-openai-like==0.8.1`, `llama-index-embeddings-ollama==0.10.0`, `pydantic-settings>=2.6` |
| `src/showcase/settings.py` | criar | `Settings(BaseSettings)` com as variáveis do System Design §8 e defaults |
| `src/showcase/models.py` | criar | `build_llm(s) -> LLM`, `build_embed_model(s) -> BaseEmbedding`; provider `mock` para testes |
| `src/showcase/events.py` | criar | eventos do System Design §3 |
| `src/showcase/workflow.py` | criar | `CopilotWorkflow` com passos stub; `route` retorna `out_of_scope` para string vazia |
| `tests/test_workflow.py` | criar | §6 |

## 5. Passo a passo
Dependências → settings → models → events → workflow → testes → portões → PR.

## 6. Testes
Mín. 6: settings lê env e defaults; `build_llm` com `mock` retorna `MockLLM`; provider inválido → `ValueError`; workflow roda até `StopEvent` com pergunta stub; pergunta vazia → rota `out_of_scope`; `trace` não vazio no resultado.

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

Título: `feat(fundacao): settings, fábrica de modelos e workflow stub em LlamaIndex [pendente]`
