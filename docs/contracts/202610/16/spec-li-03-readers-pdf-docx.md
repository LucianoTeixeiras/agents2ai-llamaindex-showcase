# SPEC-LI-03 — Implementar readers de PDF e DOCX com metadados de citação e OCR de página

| Campo | Valor |
|-------|-------|
| **ID** | SPEC-LI-03 |
| **Status** | Pronta (após aceite das ADRs) |
| **PRD / System Design / ADRs** | `prd-copilot-rag-multimodal-llamaindex.md` · `system-design-copilot-rag-multimodal-llamaindex.md` System Design §4, §5 · ADR-004 |
| **Jira** | [A2LI-4](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-4) (épico A2LI-1) |
| **Branch** | `feat/li-03-readers-pdf-docx` |
| **Executor sugerido** | `spec-executor` em Cursor + Composer (D-03) |
| **Risco QA** | `qa:risk-medium` |
| **Depende de** | SPEC-LI-02 |
| **Tamanho máximo** | ≤ 400 linhas de diff, ≤ 8 arquivos (acima disso: pare e escale) |
| **Commit de referência** | `main` após merge do PR de planejamento |

## 1. Objetivo
`read_pdf` e `read_docx` retornam `list[Document]` com os metadados obrigatórios (System Design §5); página PDF sem texto passa por OCR; `detect_type` aplica allowlist, tamanho e magic bytes.

## 2. Fora de escopo
PPTX, XLSX, imagens soltas, chunking, índice, LlamaParse.

## 3. Contexto mínimo
PyMuPDF: `pymupdf.open(path)`, `page.get_text()`, `page.find_tables()`, `page.get_pixmap(dpi=200)`. OCR via `ocr_image(png_bytes) -> str` (pytesseract, `lang="por+eng"`). Rejeitar `.docm`. Erros exatos: `"unsupported_type:<ext>"`, `"file_too_large"`, `"signature_mismatch"`.

## 4. Arquivos e interfaces
| Arquivo | Ação | O que muda |
|---------|------|------------|
| `pyproject.toml` / `uv.lock` | alterar | + `pymupdf>=1.24`, `pytesseract==0.3.13`, `pillow>=10` |
| `src/showcase/ingest/__init__.py` | criar | vazio |
| `src/showcase/ingest/detect.py` | criar | `detect_type(path) -> str`, `IngestError` |
| `src/showcase/ingest/ocr.py` | criar | `ocr_image` |
| `src/showcase/ingest/readers.py` | criar | `read_pdf`, `read_docx` (tabelas DOCX em markdown) |
| `.github/workflows/ci.yml` | alterar | `sudo apt-get install -y tesseract-ocr tesseract-ocr-por` no job de teste |
| `tests/test_readers_pdf_docx.py` | criar | §6 |

## 5. Passo a passo
Dependências → detect → ocr → readers → CI → testes → portões → PR.

## 6. Testes
Mín. 8: PDF fixture gera 1 doc por página com `page`; tabela vira markdown; página escaneada → `ocr=True` e texto não vazio (marcador `ocr`); DOCX preserva títulos e tabela; `.docm` → `unsupported_type`; >50 MB → `file_too_large`; PDF renomeado para .docx → `signature_mismatch`; metadados obrigatórios presentes.

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

Título: `feat(ingestao): readers de PDF e DOCX com OCR e metadados de citação [A2LI-4]`
