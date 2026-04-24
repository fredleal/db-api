# Staging pro PC de trabalho

Pasta pronta pra enviar. Envie por **email corporativo → corporativo** (um email só, com `to-work-pc.zip` em anexo).

---

## No PC de trabalho

### 1. `local-only/` — **3 arquivos, NUNCA subir em lugar nenhum**

Todos os arquivos desta pasta tem `⚠️ LOCAL ONLY` na primeira linha.
**Nunca**: importar no Notion, subir no Drive, anexar em emails, compartilhar.

| Arquivo | O que é | Como usar |
|---|---|---|
| `gemini-context.md` | Contexto de documentação pro Gemini | Colar no início de cada sessão Gemini (CLI ou Chat) pra gerar docs (Daily, ADR, BR, etc) |
| `compre-e-ganhe-migration-context.md` | Plano da migração Builder.io → Next.js | Colar no início da sessão do agente quando trabalhar na migração do Compre e Ganhe |
| `css-review-checklist.md` | Checklist de revisão de CSS + Atomic Design | Colar quando precisar auditar/padronizar componentes. Complementa o migration-context |

- Salvar todos em `%USERPROFILE%\Documents\` (local only)
- Alternativa pro gemini-context: salvar como Gem customizada se a 3C tiver Gemini Workspace

### 2. `notion-import/` — importar no Notion corporativo

**Método seguro (NÃO usar copiar e colar — Notion quebra markdown):**

1. Abrir Notion corporativo
2. Menu lateral → **Import**
3. Escolher **Text & Markdown**
4. Selecionar os 9 arquivos de `notion-import/` (incluindo subpastas)
5. Notion cria páginas nativas

**⚠️ NUNCA selecionar na janela de import:**
- Este `INSTRUCTIONS.md`
- Qualquer coisa de `local-only/`

---

## O que tem aqui (9 arquivos pro Notion + 3 locais)

### `notion-import/` (vai pro Notion)
```
home-moc.md                     → landing page
templates/
  1on1.md                       → template do database "1:1 com Neuliane"
  adr-madr-short.md             → template do database "ADRs"
  daily-work-log.md             → template do database "Daily Work Log"
  business-rule.md              → template do database "Business Rules"
onboarding/
  stack-map.md                  → página onboarding
  glossario.md                  → página onboarding (vazia, preencher)
  repo-map.md                   → página onboarding (vazia, preencher)
  people-map.md                 → página onboarding
```

### `local-only/` (FICA NO PC, NUNCA SOBE)
```
gemini-context.md                          → contexto de docs pro Gemini
compre-e-ganhe-migration-context.md        → plano da migração Builder.io → Next.js
css-review-checklist.md                    → auditoria de CSS + Atomic Design (complementa o migration-context)
```

---

## Ordem de execução no PC

1. **Salvar `gemini-context.md`** em `Documents\` (local only)
2. **Abrir Notion corporativo** → Import → selecionar `notion-import/` inteiro
3. **Criar os databases** e aplicar cada template como page template do seu database:
   - Daily Work Log → database "Daily Work Log"
   - 1:1 → database "1:1 com Neuliane"
   - ADR → database "ADRs"
   - Business Rule → database "Business Rules"
4. **Abrir Git Bash** no PC → `npm install -g @google/gemini-cli`
5. **Testar**: `gemini` → colar o conteúdo de `gemini-context.md` → pedir "Gere um Daily Work Log de exemplo"
6. **Verificar**: o output deve ter as seções Done, Decisions, Blockers, Learnings, Highlights / Impact, Captura rápida, Win candidate
7. **Primeiro prompt real** (depois da verificação): usar o prompt do plano-mãe pra vasculhar docs existentes do time e mapear BRs/termos/ADRs

---

## Depois do setup

- **Dia a dia**: usar Gemini CLI pra gerar/refinar docs diretamente no Notion (copiar do output)
- **Sexta/weekly**: exportar páginas elegíveis (BRs, ADRs aceitos, Runbooks, Tech Notes de stack) do Notion → colar no Drive como Google Doc em `3C-Research-Sources/`
- **NotebookLM**: criar notebook "3C Engineering Knowledge" apontado pra essa pasta do Drive
- **Nunca exportar** pro Drive: Daily Work Log, 1:1 notes

---

## Checklist de segurança (antes de apertar Import)

- [ ] `INSTRUCTIONS.md` NÃO está selecionado
- [ ] `local-only/` NÃO está selecionado
- [ ] Só arquivos de `notion-import/` estão marcados
- [ ] Depois do import, conferir que o Notion não listou nenhum arquivo além dos 9 esperados
