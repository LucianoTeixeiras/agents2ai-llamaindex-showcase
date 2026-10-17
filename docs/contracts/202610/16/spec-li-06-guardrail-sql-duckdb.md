# SPEC-LI-06 — Implementar guardrail sqlglot e ferramenta NL→SQL sobre DuckDB

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-06 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §6 · ADR-006 |
| **Jira** | [A2LI-7](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-7) (épico A2LI-1) |
| **Branch** | `feat/li-06-guardrail-sql-duckdb` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-01, SPEC-LI-02 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
`validate_sql` com o mesmo contrato da SPEC-LG-02 (mensagens exatas) e `SqlTool`: gera SQL com `NLSQLTableQueryEngine(sql_only=True)`, valida, executa em DuckDB `read_only=True` e devolve `SqlResult(sql, columns, rows[:50], row_count_estimate)`.

## 2. Fora de escopo
Roteamento, HITL (só a estimativa de linhas), BigQuery.

## 3. Contexto mínimo
Mensagens: `"parse_error"`, `"multiple_statements"`, `"not_select"`, `"forbidden_statement"`, `"table_not_allowed:<nome>"`, `"pii_column:<nome>"`, `"star_on_pii_table:<nome>"`. PII Chinook: `{"email","phone","fax","address","postalcode"}` (comparar em minúsculas). Allowlist = tabelas Chinook + `sheet_*`. `NLSQLTableQueryEngine` exige `SQLDatabase` (SQLAlchemy); usar `duckdb-engine` só para leitura de schema/geração. Estimativa: `EXPLAIN` ou `SELECT count(*) FROM (<sql sem LIMIT>)`.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `pyproject.toml` / `uv.lock` | alterar | + `sqlglot==30.23.0`, `duckdb-engine>=0.13`, `sqlalchemy>=2.0` |
| `src/showcase/guardrails.py` | criar | `GuardrailResult`, `validate_sql`, `PII_COLUMNS` |
| `src/showcase/sql_tool.py` | criar | `SqlTool.generate`, `SqlTool.run`, `SqlResult` |
| `tests/test_guardrails.py` | criar | casos da SPEC-LG-02 adaptados ao Chinook |
| `tests/test_sql_tool.py` | criar | §6 |

## 5. Passo a passo
Dependências → guardrail → tool → testes → portões → PR.

## 6. Testes
Mín. 16: os 14 casos da SPEC-LG-02 adaptados (ex.: `SELECT Email FROM Customer` → `pii_column:email`); conexão é read-only (INSERT falha mesmo sem guardrail); `MockLLM` devolvendo `DROP TABLE` nunca executa; resultado limitado a 50 linhas.

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

Título: `feat(sql): guardrail sqlglot e ferramenta NL→SQL em DuckDB [A2LI-7]`
