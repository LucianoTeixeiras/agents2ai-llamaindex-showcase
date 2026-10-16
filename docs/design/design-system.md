# Design System — Agents2AI LlamaIndex Showcase (tema claro, alto contraste)

| Campo | Valor |
|-------|-------|
| **Status** | Proposto (2026-10-16) |
| **Origem** | `agents2ai-data-copilot`: `config/brand.yaml`, `webapp/tailwind.config.ts`, `webapp/src/app/globals.css` |
| **Uso** | Fase 3 (UI web/API, fora do MVP) e artefatos visuais da vitrine (README, diagramas, GIF, relatório de evals em HTML) |
| **Requisito do dono** | **Somente tema claro**, alto contraste: Luciano tem dificuldade de leitura em fundos escuros. Não haverá dark mode. |

## 1. Princípios
1. **Claro sempre:** `color-scheme: light`; nenhum fundo escuro em telas, blocos de código, tooltips ou diagramas.
2. **Contraste WCAG AAA para texto (≥ 7:1)** e ≥ 3:1 para bordas de controles e ícones. Valores abaixo calculados em 2026-10-16.
3. **Auditável:** toda resposta mostra a fonte (citação) ou o SQL; o design dá destaque a isso, não esconde.
4. **White-label:** cores de marca vêm de `config/brand.yaml` via CSS variables, como no copilot.

## 2. Marca (de `config/brand.yaml`)
```yaml
product_name: LlamaIndex Copilot (showcase)
company_name: Agents2AI
tagline: Pergunte aos seus documentos e dados. Respostas com fonte.
logo: brand-mark.svg
colors:
  primary: "#4338ca"       # era primary_dark no copilot; promovido para AAA
  primary_hover: "#3730a3"
```

## 3. Tokens de cor
| Token | Copilot | **Showcase (alto contraste)** | Contraste | Uso |
|-------|---------|-------------------------------|-----------|-----|
| `canvas` | `#f8fafc` | `#f8fafc` | — | fundo da aplicação |
| `surface` | `#ffffff` | `#ffffff` | — | cards, painéis, blocos de código |
| `ink` | `#0f172a` | `#0f172a` | 17,9:1 em branco | texto principal |
| `muted` | `#64748b` (4,8:1) | **`#475569`** | 7,6:1 | texto secundário, legendas |
| `primary` | `#4f46e5` (6,3:1) | **`#4338ca`** | 7,9:1 (texto branco sobre ele também 7,9:1) | botões, links, foco |
| `accent` | `#0369a1` (5,9:1) | **`#075985`** | 7,6:1 | citações `[n]`, chips de fonte |
| `success` | `#166534` | `#166534` | 7,1:1 | SQL aprovado, aprovação concedida |
| `warning` | — | `#92400e` sobre `#fffbeb` | 7,1:1 (em branco) | HITL pendente |
| `danger` | — | `#991b1b` sobre `#fef2f2` | > 7:1 | guardrail reprovado, erro |
| `border` | `#e2e8f0` (1,2:1) | `#cbd5e1` decorativa · **`#64748b`** em inputs/controles | 4,8:1 | separadores · campos |
| `focus` | — | anel 3 px `#4338ca` + offset 2 px branco | 7,9:1 | foco de teclado sempre visível |

Proibido: texto em `#94a3b8` ou mais claro (2,6:1), texto sobre `primary` claro, fundos `slate-800/900`.

## 4. Tipografia
- Família: `Inter, system-ui, sans-serif`; mono: `ui-monospace, SFMono-Regular, Menlo, Consolas, monospace` (do copilot).
- Tamanho base **16 px** (mínimo 14 px só para metadados); altura de linha 1,6; peso de texto 400/500, títulos 600.
- Escala: 14 · 16 · 18 · 20 · 24 · 30 px. Sem itálico em blocos longos.
- Código/SQL: fundo `surface` com borda `#cbd5e1`, texto `ink`, realce de sintaxe em tema **claro** (ex.: GitHub Light), nunca tema escuro.

## 5. Forma e elevação
- `rounded-card` 12 px; botões e inputs 8 px.
- Sombra `enterprise`: `0 4px 6px -1px rgba(15,23,42,.05), 0 2px 4px -1px rgba(15,23,42,.03)`.
- Espaçamento em múltiplos de 4 px; alvo de clique ≥ 44 × 44 px.

## 6. Componentes
| Componente | Regras |
|------------|--------|
| **Resposta** | texto `ink`; citações `[n]` em `accent` sublinhado, clicáveis, abrindo o trecho |
| **Painel de fontes** | lista `arquivo · p.X / slide Y / aba Z`, ícone por tipo (PDF, DOCX, PPTX, XLSX, imagem) com rótulo em texto (não só cor) |
| **Bloco SQL** | título "SQL executado", badge `success` "Aprovado pelo guardrail" ou `danger` com a mensagem exata do guardrail |
| **Tabela de resultados** | cabeçalho `#f1f5f9` com texto `ink` 600; zebra `#f8fafc`; máx. 50 linhas com aviso |
| **Cartão HITL** | fundo `#fffbeb`, borda `#92400e`, texto `ink`; botões "Aprovar" (primary) e "Negar" (contorno `danger`) |
| **Trace** | linha do tempo por passo com ms e tokens; barras em `primary` sobre `#e0e7ff` |
| **Status de ingestão** | texto + ícone (não só cor): "Indexado", "OCR aplicado", "Rejeitado: unsupported_type" |

## 7. Tailwind (proposta para fase 3)
```ts
colors: { brand: {
  primary: 'var(--brand-primary, #4338ca)', primaryHover: 'var(--brand-primary-hover, #3730a3)',
  canvas: '#f8fafc', surface: '#ffffff', ink: '#0f172a', muted: '#475569',
  border: '#cbd5e1', control: '#64748b', accent: '#075985',
  success: '#166534', warning: '#92400e', danger: '#991b1b',
} }
```
`globals.css`: `:root { color-scheme: light; }` e **sem** `@media (prefers-color-scheme: dark)`.

## 8. Diagramas e mídia da vitrine
Mermaid com tema `default`/`neutral` (fundo branco), texto `ink`; GIF/capturas sempre do tema claro; relatório HTML de evals com os mesmos tokens.

## 9. Acessibilidade (checklist)
- [ ] Texto ≥ 7:1; controles ≥ 3:1; foco visível.
- [ ] Informação nunca só por cor.
- [ ] Zoom 200% sem perda; navegação por teclado completa.
- [ ] Nenhum fundo escuro em nenhum estado.
