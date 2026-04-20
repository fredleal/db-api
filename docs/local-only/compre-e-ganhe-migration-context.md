# Compre e Ganhe — Migracao Builder.io → Next.js nativo

> ⚠️ **LOCAL ONLY** — NUNCA importar no Notion, Drive ou qualquer sistema compartilhado.
> Cole este documento no inicio da sessao do agente (Gemini CLI, Code Assist ou similar).
> Self-contained: tudo que voce precisa pra executar esta aqui.

---

## Projeto

**Compre e Ganhe** (compreeganhe.mercafe.com.br) — paginas promocionais do Mercafe.
Atualmente construido com **Builder.io**. SDK causa load lento (mobile principalmente), instabilidade, e custo de plataforma.

**Objetivo**: migrar todas as paginas para Next.js nativo dentro do repo Mercafe existente. Eliminar dependencia do Builder.io.

**Dev responsavel**: Fred (fullstack senior, 2a semana no time)
**Dev de referencia no repo**: Cadu (conhece o front do Mercafe)
**Ambiente de homolog**: disponivel
**Infra**: on-premises (servidor da empresa, nao Vercel/cloud)

---

## Decisao arquitetural: SSG puro, sem SSR

A infra e on-prem e o conteudo e identico pra todos os usuarios. SSR nao faz sentido.

- Todas as paginas usam **Static Site Generation** (`generateStaticParams` + build-time rendering)
- HTML gerado no build, servido como arquivo estatico — zero custo de CPU por request
- Formularios sao **Client Components** (`"use client"`) — interatividade roda no browser
- API calls dos forms vao direto do browser pra API backend, sem passar pelo Next.js server
- Troca de campanha = atualizar JSON + redeploy (build gera HTML novo)
- Picos de acesso (campanhas promocionais) nao sobrecarregam o server

---

## Paginas a migrar

| # | Pagina | Tipo | Rendering | Complexidade |
|---|---|---|---|---|
| 1 | Home hub (`/compre-e-ganhe`) | Landing com cards de campanhas | SSG | Media |
| 2 | Campanha ativa (ex: Dia das Maes) | Hero + regras + CTA | SSG | Media |
| 3 | Formulario de cadastro | Form multi-step + validacao + API | SSG shell + Client Component | Alta |
| 4 | Formulario de resgate | Form + consulta + API | SSG shell + Client Component | Alta |
| 5 | Regulamento | Texto estatico longo | SSG | Baixa |
| 6 | FAQ / Duvidas | Accordion estatico | SSG | Baixa |

---

## Arquitetura de pastas (modelo — adaptar ao repo real)

**IMPORTANTE**: antes de criar qualquer pasta ou componente, verificar convencoes existentes do repo Mercafe. Se ja existem `components/shared/`, `components/ui/`, ou outra convencao, seguir. O esquema abaixo e o modelo de referencia.

```
src/
  app/
    compre-e-ganhe/
      layout.tsx                # Layout compartilhado (Header + Footer)
      page.tsx                  # Home hub (SSG)
      regulamento/page.tsx      # Texto estatico (SSG)
      faq/page.tsx              # Accordion (SSG)
      [campanha]/               # Dynamic route por campanha
        page.tsx                # Landing da campanha (SSG)
        cadastro/page.tsx       # Form cadastro (SSG shell + "use client")
        resgate/page.tsx        # Form resgate (SSG shell + "use client")
  components/
    compre-e-ganhe/
      atoms/                    # Button, Input, Badge, Tag
      molecules/                # CampaignCard, FormField, AccordionItem
      organisms/                # Header, Footer, HeroSection, CampaignForm
  content/
    compre-e-ganhe/
      campanhas.json            # Dados das campanhas (titulo, regras, datas, assets)
      regulamento.json          # Texto do regulamento
      faq.json                  # Perguntas e respostas
  lib/
    compre-e-ganhe/
      api.ts                    # Client da API de formularios (ja mapeada)
      validation.ts             # Schemas de validacao (zod)
      types.ts                  # Tipos compartilhados
```

---

## Convencoes tecnicas

- **Styling**: seguir o padrao que o repo Mercafe ja usa (Tailwind, CSS Modules, styled-components — verificar antes de comecar)
- **TypeScript**: strict mode
- **Componentes**: reusar existentes do Mercafe antes de criar novos
- **JSON content**: conteudo das paginas vive em `content/compre-e-ganhe/` — separado dos componentes
- **SSG**: usar `generateStaticParams` pra rotas dinamicas. Nenhuma pagina usa `getServerSideProps` ou equivalente
- **Client Components**: so pra formularios e interacoes que exigem estado no browser. Marcar com `"use client"` no topo do arquivo
- **Acessibilidade**: labels em forms, contraste WCAG AA, keyboard navigation, heading structure semantica

---

## Estrategia de deploy: incremental, pagina a pagina

Cada fase gera PR independente. Builder.io continua ativo em paralelo ate a ultima fase. Rollback e trivial: reverter redirect.

```
Fase 1 (layout)        → PR → homolog → validacao → prod
Fase 2 (estaticas)     → PR → homolog → validacao → prod
Fase 3 (home+campanha) → PR → homolog → validacao → prod
Fase 4 (forms)         → PR → homolog → validacao com Cadu → prod
Fase 5 (cutover)       → PR → homolog → validacao final → prod → remove Builder.io SDK
```

---

## Fase 1 — Layout compartilhado (1-2 dias)

**Objetivo**: estrutura base de navegacao.

### Tarefas
- [ ] Verificar componentes existentes no repo (Header, Footer, Button, Container)
- [ ] Criar `app/compre-e-ganhe/layout.tsx` com Header + Footer
- [ ] Criar atoms novos so se nao existirem equivalentes no repo
- [ ] Setup de rotas e metadata SEO (`generateMetadata`)
- [ ] Tokens de design: cores e tipografia da campanha Compre e Ganhe

### Criterio de pronto
- [ ] Layout renderiza em homolog
- [ ] Navegacao entre todas as rotas do Compre e Ganhe funciona
- [ ] Metadata (title, description, OG tags) presente em todas as paginas

### Prompt sugerido pro agente
```
Analise o repo Mercafe e identifique:
1. Componentes existentes reusaveis (Header, Footer, Button, etc)
2. Convencao de pastas pra componentes
3. Sistema de styling usado (Tailwind? CSS Modules? styled-components?)
4. Como metadata/SEO e configurado nas paginas existentes

Com base nisso, crie o layout.tsx do Compre e Ganhe reusando o maximo possivel.
Todas as paginas sao SSG — nenhum getServerSideProps.
```

---

## Fase 2 — Paginas estaticas: Regulamento + FAQ (1-2 dias)

**Objetivo**: 2 paginas simples publicaveis.

### Tarefas
- [ ] Extrair conteudo do Regulamento atual (Builder.io) → `content/compre-e-ganhe/regulamento.json`
- [ ] Extrair conteudo do FAQ atual → `content/compre-e-ganhe/faq.json`
- [ ] Criar pagina Regulamento: render SSG do JSON
- [ ] Criar pagina FAQ: AccordionItem com expand/collapse (pode ser Client Component leve ou CSS-only)
- [ ] Heading structure semantica (h1, h2, h3)
- [ ] Responsividade mobile

### Criterio de pronto
- [ ] Ambas as paginas renderizam em homolog com conteudo real
- [ ] Lighthouse Performance >=90 mobile
- [ ] Texto acessivel (contraste, headings, keyboard nav no accordion)

### Prompt sugerido pro agente
```
Crie as paginas de Regulamento e FAQ do Compre e Ganhe.
Conteudo vem de arquivos JSON em content/compre-e-ganhe/.
Ambas sao SSG puro. FAQ tem accordion expand/collapse.
Mantenha acessibilidade: headings semanticos, contraste WCAG AA, keyboard nav.
Siga convencoes do repo Mercafe.
```

---

## Fase 3 — Home hub + Landing de campanha (2-3 dias)

**Objetivo**: paginas principais com conteudo visual da campanha.

### Tarefas
- [ ] Criar `content/compre-e-ganhe/campanhas.json` com dados da campanha ativa (titulo, descricao, regras, datas, imagens, CTAs)
- [ ] Home hub: grid de CampaignCards linkando pra campanhas ativas
- [ ] Landing da campanha: HeroSection + secao de regras + CTAs pra cadastro/resgate
- [ ] `generateStaticParams` retornando slugs das campanhas ativas
- [ ] Assets: verificar onde imagens estao (CDN? public/? URLs externas?)
- [ ] Responsividade mobile-first
- [ ] OG tags por campanha (compartilhamento social)

### Criterio de pronto
- [ ] Home e landing renderizam em homolog com conteudo e imagens reais
- [ ] Lighthouse Performance >=90 mobile
- [ ] Cards na home linkam corretamente pra landing
- [ ] CTAs na landing linkam pra formularios

### Prompt sugerido pro agente
```
Crie a home hub e a landing page de campanha do Compre e Ganhe.
Dados vem de campanhas.json. Rotas dinamicas com generateStaticParams.
Home mostra grid de campanhas. Landing mostra hero + regras + CTAs.
SSG puro. Mobile-first. OG tags por campanha.
Verifique onde os assets (imagens) estao armazenados no repo antes de referenciar.
```

---

## Fase 4 — Formularios: cadastro + resgate (2-3 dias)

**Objetivo**: formularios funcionais integrados com API real.

### Tarefas
- [ ] Criar CampaignForm como Client Component (`"use client"`)
- [ ] Pagina SSG renderiza shell estatico, form hidrata no client
- [ ] Validacao client-side com zod (schemas em `lib/compre-e-ganhe/validation.ts`)
- [ ] Integracao com API de formularios (chamadas direto do browser, sem proxy server)
- [ ] Estados visuais: idle, loading, success, error, timeout
- [ ] Form de cadastro: campos conforme API mapeada
- [ ] Form de resgate: campos conforme API mapeada
- [ ] Feedback visual claro pro usuario em cada estado
- [ ] Mascaras de input se necessario (CPF, telefone)

### Criterio de pronto
- [ ] Ambos os formularios submetem com sucesso na API real em homolog
- [ ] Validacao client-side impede submit invalido
- [ ] Estados de loading/success/error funcionam
- [ ] Acessibilidade: labels, aria-describedby pra erros, focus management
- [ ] Testado em mobile

### Prompt sugerido pro agente
```
Crie os formularios de cadastro e resgate do Compre e Ganhe.
Sao Client Components ("use client") dentro de paginas SSG.
API calls vao direto do browser pra API backend — sem proxy pelo Next.js server.
Validacao com zod. Estados: idle, loading, success, error, timeout.
Campos e endpoints estao mapeados em lib/compre-e-ganhe/api.ts.
Acessibilidade: labels, aria, focus management apos submit.
```

---

## Fase 5 — Cutover: redirects + remocao Builder.io (1-2 dias)

**Objetivo**: corte final. Builder.io desativado.

### Tarefas
- [ ] Mapear todas as URLs do Builder.io que precisam de redirect
- [ ] Configurar redirects (next.config.js, middleware, ou infra — confirmar com Cadu)
- [ ] Configurar analytics equivalente ao atual (eventos, pageviews)
- [ ] Validar todas as paginas em producao (checklist completo abaixo)
- [ ] Remover SDK Builder.io do bundle
- [ ] Medir e documentar bundle size antes vs depois
- [ ] PR review com Cadu

### Checklist de validacao final (todas as paginas)
- [ ] Renderiza com conteudo correto
- [ ] Links internos funcionam
- [ ] Meta tags / OG tags corretos
- [ ] Responsivo mobile
- [ ] Lighthouse >=90 mobile
- [ ] Analytics disparando eventos
- [ ] Redirects de URLs antigas funcionando
- [ ] Zero erro no console
- [ ] Zero regressao nas outras paginas do Mercafe

### Prompt sugerido pro agente
```
Fase final da migracao Compre e Ganhe.
1. Identifique todas as URLs do Builder.io que precisam redirect
2. Configure redirects no next.config.js (ou middleware, conforme padrao do repo)
3. Remova o SDK do Builder.io do bundle
4. Verifique bundle size antes e depois
5. Liste qualquer referencia restante ao Builder.io no codebase
```

---

## Fluxo de campanha futura

Quando a campanha mudar (ex: Dia das Maes → Dia dos Namorados):

1. Atualizar `content/compre-e-ganhe/campanhas.json` com dados da nova campanha
2. Adicionar assets (imagens, logos) da nova campanha
3. PR + build → HTML novo gerado no build (SSG)
4. Deploy via pipeline normal

**Quem edita**: dev via PR. Se no futuro marketing precisar editar direto, substituir JSON por headless CMS — componentes nao mudam, so a fonte de dados.

---

## Perguntas pra Cadu (resolver antes da Fase 1)

| # | Pergunta | Impacto |
|---|---|---|
| 1 | Quais componentes existentes posso reusar? (Header, Footer, Button, etc) | Fase 1 — evitar duplicacao |
| 2 | Onde estao os assets das campanhas? (CDN, public/, URLs externas) | Fase 3 — referencia de imagens |
| 3 | Como redirects sao configurados? (next.config, middleware, infra/DNS) | Fase 5 — pode precisar de outro time |
| 4 | Qual analytics esta ativo? (GA4, Tag Manager, eventos customizados) | Fase 5 — paridade de tracking |
| 5 | Branch strategy: feature branch a partir de qual base? | Todas as fases |
| 6 | Quem valida em homolog alem de voce? (PO, marketing) | Todas as fases |

---

## Resumo de estimativa

| Fase | Dias | Acumulado |
|---|---|---|
| 1 — Layout | 1-2 | 1-2 |
| 2 — Estaticas | 1-2 | 2-4 |
| 3 — Home + Campanha | 2-3 | 4-7 |
| 4 — Formularios | 2-3 | 6-10 |
| 5 — Cutover | 1-2 | 7-12 |

**Total estimado: 7-12 dias uteis** (inclui margem pra curva de aprendizado do repo).
