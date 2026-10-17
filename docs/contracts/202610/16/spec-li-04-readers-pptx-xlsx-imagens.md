# SPEC-LI-04 — Implementar readers de PPTX, XLSX e imagens

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-04 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §4, §5 · ADR-004 |
| **Jira** | [A2LI-5](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-5) (épico A2LI-1) |
| **Branch** | `feat/li-04-readers-pptx-xlsx-imagens` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-03 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
`read_pptx` (1 `Document` por slide, texto + notas, `slide`), `read_xlsx` (1 `Document` por aba em markdown truncado a 200 linhas **e** registro de tabela DuckDB `sheet_<arquivo>_<aba>` normalizado em snake_case), `read_image` (OCR, `ocr=True`).

## 2. Fora de escopo
Legenda por modelo de visão; chunking; índice.

## 3. Contexto mínimo
python-pptx: `Presentation(path).slides`, `slide.shapes` (`has_text_frame`), `slide.has_notes_slide`, `slide.notes_slide.notes_text_frame.text`. XLSX: `pandas.read_excel(path, sheet_name=None)`; rejeitar `.xlsm`. Registro de tabela: função `register_sheet_tables(con, path) -> list[str]` em conexão DuckDB de **escrita apenas na ingestão**.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `src/showcase/ingest/readers.py` | alterar | + `read_pptx`, `read_xlsx`, `read_image`, `READERS: dict[str, Callable]` |
| `src/showcase/ingest/tables.py` | criar | `register_sheet_tables` |
| `pyproject.toml` / `uv.lock` | alterar | + `pandas>=2.2` |
| `tests/test_readers_pptx_xlsx_img.py` | criar | §6 |

## 5. Passo a passo
Readers → tables → testes → portões → PR.

## 6. Testes
Mín. 8: PPTX fixture gera 4 docs com `slide` 1..4 e notas; XLSX gera 2 docs com `sheet`; tabelas `sheet_vendas_faturamento` e `sheet_vendas_generos` existem; nomes normalizados; truncamento a 200 linhas; PNG gera texto via OCR (`ocr`); `.xlsm` rejeitado; `READERS` cobre as 7 extensões.

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

Título: `feat(ingestao): readers de PPTX, XLSX e imagens [A2LI-5]`
