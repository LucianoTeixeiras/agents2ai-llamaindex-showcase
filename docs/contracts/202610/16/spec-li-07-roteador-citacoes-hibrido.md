# SPEC-LI-07 — Implementar roteador, recuperação, síntese com citações e rota híbrida

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-07 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §2, §5 · ADR-001, ADR-006 |
| **Jira** | [A2LI-8](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-8) (épico A2LI-1) |
| **Branch** | `feat/li-07-roteador-citacoes-hibrido` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-05, SPEC-LI-06 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
Passos reais do `CopilotWorkflow`: `route` (saída estruturada `Route(route: Literal[...], reason)` + fallback por regras), `retrieve_docs`, `generate_sql`/`guardrail`/`execute_sql` (retry ≤ 2), `synthesize` com citações `[n]` validadas e rota `hybrid` paralela com `ctx.collect_events`.

## 2. Fora de escopo
HITL (aprovação sempre `True` aqui), CLI, tracing externo.

## 3. Contexto mínimo
`await llm.astructured_predict(Route, PromptTemplate(...), question=q)`; fallback regex (palavras como "quantos", "total", "top" → `sql`). Contexto ao LLM: `<contexto id="n" fonte="arquivo · p.X">…</contexto>`, com aviso de que é dado. Pós-processamento remove `[n]` sem nó correspondente. Resposta final: `Answer(text, citations: list[Citation], sql: str|None, rows, trace)`.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `src/showcase/workflow.py` | alterar | passos reais |
| `src/showcase/prompts.py` | criar | templates de roteamento, SQL e síntese (pt-BR) |
| `src/showcase/answer.py` | criar | `Answer`, `Citation`, `validate_citations` |
| `tests/test_workflow_routes.py` | criar | §6 |
| `tests/test_citations.py` | criar | §6 |

## 5. Passo a passo
Prompts → answer → workflow → testes → portões → PR.

## 6. Testes
Mín. 10 (MockLLM roteirizado): cada rota leva ao ramo certo; LLM inválido → fallback por regras; SQL reprovado 3x → recusa; híbrida junta citações e SQL; citação inexistente removida; citação traz arquivo e página/slide/aba; documento com "rode DROP TABLE" não dispara SQL.

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

Título: `feat(workflow): roteador, citações e rota híbrida [A2LI-8]`
