# agents2ai-llamaindex-showcase

> 🇧🇷 Copilot de dados com **RAG multimodal** (PDF, DOCX, PPTX, XLSX e imagens) e **NL→SQL** com guardrail, construído com **LlamaIndex** (Workflows/AgentWorkflow) sobre **dados públicos**. Roda 100% local com SLM via Ollama; fallback OpenAI-compatível. Demonstra roteamento híbrido documentos × SQL, citações de fonte, aprovação humana (HITL), evals de RAG e SQL e tracing.
>
> 🇺🇸 Data copilot with **multimodal RAG** (PDF, DOCX, PPTX, XLSX, images) and guarded **NL→SQL**, built with **LlamaIndex** over **public data only**. Runs fully local with an Ollama SLM; OpenAI-compatible fallback. Showcases hybrid routing, source citations, human-in-the-loop, RAG + SQL evals and tracing.

**Status:** planejamento (v0). Arquitetura e specs em `docs/` · *planning stage; architecture and specs live in `docs/`.*

## Por que / Why
Projeto vitrine de **Luciano Teixeira** — AI & Data Engineer, fundador da **Agents2AI**. Irmão do `agents2ai-langgraph-showcase`: mesmo rigor (guardrails, HITL, evals), agora com o framework de referência em RAG.
*Showcase project by Luciano Teixeira, AI & Data Engineer and founder of Agents2AI.*

## Stack (planejada / planned)
Python 3.12 · uv · LlamaIndex · Ollama (SLM + bge-m3) · DuckDB (+ `vss`) · sqlglot · LlamaParse (opcional) · pytest · ruff · gitleaks.

## Rodando / Running
```bash
uv sync --group dev
uv run ruff check .
uv run pytest -q
```

## Licença / License
MIT — ver `LICENSE`. Dados de terceiros mantêm suas licenças (ver ADR-002).
