# SPEC-LI-02 — Baixar corpus público, gerar documentos sintéticos e registrar manifest

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-02 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §7 · ADR-002 |
| **Jira** | [A2LI-3](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-3) (épico A2LI-1) |
| **Branch** | `feat/li-02-corpus-publico-manifest` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-01 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
`scripts/fetch_corpus.py` baixa Chinook, IBGE e (opcional, `--with-arxiv`) arXiv para `data/raw/` com sha256 e licença em `data/manifest.json`; `scripts/make_synthetic_docs.py` gera DOCX, PPTX, XLSX e PNGs a partir do Chinook; fixtures mínimos em `tests/fixtures/`.

## 2. Fora de escopo
Ler/indexar documentos; converter SQLite→DuckDB na execução da consulta (só o arquivo `data/raw/chinook.duckdb`).

## 3. Contexto mínimo
URLs e licenças exatas: tabela da ADR-002 (verificadas em 2026-10-16). DuckDB: `INSTALL sqlite; LOAD sqlite; ATTACH 'Chinook_Sqlite.sqlite' AS src (TYPE sqlite); CREATE TABLE ... AS SELECT * FROM src.<t>` (só no script, nunca no runtime). Fixtures devem ter < 100 KB cada.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `pyproject.toml` / `uv.lock` | alterar | + `duckdb==1.5.6`, `python-docx==1.2.0`, `python-pptx==1.0.2`, `openpyxl>=3.1`, `matplotlib>=3.9`, `httpx>=0.27` |
| `scripts/fetch_corpus.py` | criar | download com timeout, sha256, `data/manifest.json`; `--with-arxiv` |
| `scripts/make_synthetic_docs.py` | criar | gera `relatorio_vendas.docx` (títulos+tabela), `apresentacao_vendas.pptx` (4 slides + notas), `vendas.xlsx` (abas `Faturamento`, `Generos`), `grafico_generos.png`, `pagina_escaneada.png` |
| `tests/fixtures/` | criar | versões pequenas dos 5 arquivos acima + `LICENSE-fixtures.md` (MIT, derivados do Chinook) |
| `data/README.md` | criar | fontes, licenças, como baixar |
| `tests/test_corpus.py` | criar | §6 |

## 5. Passo a passo
Dependências → scripts → gerar fixtures → testes → portões → PR.

## 6. Testes
Mín. 6 (sem rede; `httpx` mockado): manifest tem url/sha256/licença; sha256 confere; arXiv só com flag; sintéticos abrem com as libs; xlsx tem 2 abas; fixtures < 100 KB.

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

Título: `feat(dados): corpus público, manifest e documentos sintéticos [A2LI-3]`
