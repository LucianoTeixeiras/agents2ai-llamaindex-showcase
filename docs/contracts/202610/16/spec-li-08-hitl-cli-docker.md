# SPEC-LI-08 — Adicionar HITL, CLI e Docker

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-08 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §2, §8a · ADR-006 |
| **Jira** | pendente — D-01 |
| **Branch** | `feat/li-08-hitl-cli-docker` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-07 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
Aprovação humana via `InputRequiredEvent`/`HumanResponseEvent` para SQL acima de `MAX_ROWS_WITHOUT_APPROVAL` e uso de LlamaParse; CLI `showcase ingest` e `showcase ask` (prompt s/N); `Dockerfile` e `docker-compose.yml` com Ollama (profile `qdrant` opcional).

## 2. Fora de escopo
UI web, API HTTP, BigQuery.

## 3. Contexto mínimo
Eventos conferidos em 0.14.25: `from llama_index.core.workflow import InputRequiredEvent, HumanResponseEvent`. Padrão: o passo emite `ctx.write_event_to_stream(InputRequiredEvent(prefix=...))` e aguarda `await ctx.wait_for_event(HumanResponseEvent)` (assinatura **não verificada** — confira e escale se divergir). Efeito colateral nunca antes da aprovação. CLI com `argparse` (sem dependência nova).

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `src/showcase/workflow.py` | alterar | passo `human_approval` |
| `src/showcase/cli.py` | criar | `main()`; `[project.scripts] showcase = "showcase.cli:main"` |
| `pyproject.toml` / `uv.lock` | alterar | entry point |
| `Dockerfile` | criar | python 3.12-slim + uv + tesseract |
| `docker-compose.yml` | criar | `ollama` (healthcheck), `app`, profile `qdrant` |
| `.env.example` | criar | variáveis do System Design §8 sem valores secretos |
| `tests/test_hitl.py` | criar | §6 |

## 5. Passo a passo
Workflow → CLI → Docker → testes → portões → PR.

## 6. Testes
Mín. 6: acima do limite pausa; aprovado executa; negado responde recusa sem executar; abaixo do limite não pausa; LlamaParse sem aprovação não é chamado; `showcase --help` funciona.

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

Título: `feat(hitl): aprovação humana, CLI e Docker Compose com Ollama [pendente]`
