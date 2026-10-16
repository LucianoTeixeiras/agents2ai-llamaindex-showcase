# ADR-004 — Ingestão multimodal: readers locais por padrão, LlamaParse opcional

| Campo | Valor |
|-------|-------|
| **Status** | Proposto (2026-10-16) — aguardando aceite de Luciano |
| **Data** | 2026-10-16 |
| **Decisores** | Luciano Teixeira (aceite) · Dev Grok Bot (proposta) |
| **Jira** | [A2LI-12](https://lucianoteixeiras-1773934569563.atlassian.net/browse/A2LI-12) (aceite das ADRs, épico A2LI-1) |
| **Relacionados** | PRD e System Design `docs/contracts/202610/16/`; ADR-006 (envio a nuvem via HITL) |

## Contexto (2026-10-16)
PDF com tabelas, PPTX com notas, XLSX com várias abas e imagens precisam virar texto com localização (página/slide/aba) para citar a fonte. LlamaParse é forte em layout, mas é nuvem e exige chave.

## Decisão
1. **Recomendado (padrão):** readers locais próprios, finos sobre bibliotecas maduras: PDF `PyMuPDF` (texto e `find_tables`), DOCX `python-docx` (parágrafos, títulos, tabelas), PPTX `python-pptx` (slide + notas), XLSX `pandas`/`openpyxl` (aba → tabela DuckDB **e** `Document` markdown), imagens Tesseract (`pytesseract`, idioma `por+eng`).
2. PDF sem camada de texto na página → renderiza e aplica OCR (`ocr=True` no metadado).
3. LlamaParse (`llama-cloud-services==0.6.94`) por `USE_LLAMAPARSE=true`, chave presente **e** aprovação HITL por arquivo.
4. Legenda de imagem por modelo de visão local (ex.: `qwen2.5vl` no Ollama) como extensão opcional, fora do MVP.

## Alternativas consideradas
| Alternativa | Prós | Contras | Por que não |
|-------------|------|---------|-------------|
| **Recomendada:** locais + LlamaParse opt-in | offline, sem custo, auditável | layout complexo pior | — |
| LlamaParse padrão | melhor qualidade em layout | nuvem, chave, custo | soberania |
| `SimpleDirectoryReader` com readers default | zero código | metadados inconsistentes por formato | citação exige página/slide/aba |
| Docling / Unstructured | ótimo layout local | dependências pesadas | avaliar se evals de PDF falharem |

## Consequências
Tesseract é dependência de sistema (Docker e README); testes de OCR são marcados `ocr` e pulam se o binário faltar no CI local, mas rodam no CI (instalado no job).
