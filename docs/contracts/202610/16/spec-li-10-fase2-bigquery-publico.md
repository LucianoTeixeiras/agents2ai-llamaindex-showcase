# SPEC-LI-10 — [BLOQUEADA] Fase 2: backend BigQuery público com dry-run de custo

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-10 |
| **Status** | Bloqueada |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §6, §8 · ADR-002, ADR-006 |
| **Jira** | [A2LI-11](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-11) (épico A2LI-1) |
| **Branch** | `feat/li-10-fase2-bigquery-publico` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-high` |
| **Depende de** | SPEC-LI-09 e decisão de projeto GCP de testes |
| **Tamanho máximo** | ≤ 300 linhas de diff, ≤ 6 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
Backend `bigquery` para `SqlTool` restrito a `bigquery-public-data.*`, dry-run com bytes estimados, `maximum_bytes_billed` e HITL acima de `MAX_BYTES_WITHOUT_APPROVAL`.

## 2. Fora de escopo
Qualquer dataset que não seja `bigquery-public-data`; escrita; credenciais no repo.

## 3. Contexto mínimo
**Bloqueada** até Luciano definir projeto GCP de testes da Agents2AI (como SPEC-LG-08). Reaproveitar o desenho da SPEC-LG-08 do LangGraph showcase.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| a definir | — | detalhar quando desbloqueada |

## 5. Passo a passo
—

## 6. Testes
BigQuery mockado; nenhum teste live no CI.

## 7. Definição de Pronto (DoD)
- [ ] Testes usam só `MockLLM`/`MockEmbedding` (ou dublês locais); **zero rede** — sem Ollama, OpenRouter, LlamaParse, BigQuery ou download no `pytest`.
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

Título: `feat(sql): backend BigQuery público com dry-run [A2LI-11]`
