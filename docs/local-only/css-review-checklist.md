# Checklist de Revisao — CSS e Atomic Design

> ⚠️ **LOCAL ONLY** — NUNCA importar no Notion, Drive ou qualquer sistema compartilhado.
>
> **Para o agente**: Execute CADA checkbox como uma tarefa isolada. NAO pule etapas.
> NAO invente solucoes. Se nao sabe, pare e pergunte.
> Cada checkbox e uma micro-tarefa. Execute uma por vez, confirme o resultado, siga pra proxima.

---

## CONTEXTO

Temos 48 arquivos alterados no repo do Compre e Ganhe (Mercafe). Varios agentes diferentes criaram componentes e cada um usou um padrao de CSS diferente. O resultado e **4 formas de declarar estilo no mesmo projeto**, gerando confusao e impossibilitando futuras mudancas por CMS.

**Referencia de padrao correto**: repo `template-saas-ecommerce` em `/Users/fredleal/template-saas-ecommerce/`

---

## REGRA UNICA DE CSS (decore antes de comecar)

```
CSS Custom Properties (fonte da verdade)
        ↓
  Tailwind Config (consome as vars)
        ↓
  className no JSX (unico lugar onde estilo aparece)
```

**NUNCA**:
- Hex hardcoded em arquivo `.tsx` (ex: `#3b82f6`)
- Objeto JS com estilos inline (ex: `style={{ background: '#fff' }}`)
- Constante exportada com cores (ex: `export const primaryColor = '#3b82f6'`)
- Propriedade CSS escrita fora de `className` (ex: `style={variantStyle}`)

**SEMPRE**:
- Cores via CSS var: `bg-[var(--color-primary-500,#3b82f6)]`
- Ou via Tailwind registrado: `bg-primary-500` (se registrado no tailwind.config)
- Spacing/sizing via Tailwind: `px-4 py-2 text-sm`
- Tudo dentro de `className`

---

## PARTE 1 — IDENTIFICAR OS 4 PADROES ERRADOS

> Execute cada checkbox buscando no codigo. Anote TODOS os arquivos e linhas onde encontrar.

### 1.1 Buscar hex hardcoded em TSX

- [ ] Buscar regex `#[0-9a-fA-F]{3,8}` em todos os `.tsx` dentro de `components/`
- [ ] Para cada match encontrado, anotar: `arquivo:linha — valor encontrado — contexto (onde esta sendo usado)`
- [ ] **COMO IDENTIFICAR**: qualquer `#fff`, `#3b82f6`, `#e5e7eb` etc diretamente no codigo TSX
- [ ] **EXCECAO**: hex dentro de `var(--token, #fallback)` e CORRETO — nao marcar como erro
- [ ] **EXCECAO**: hex dentro de `tailwind.config` e aceito (config, nao componente)

**Exemplo ERRADO**:
```tsx
const primaryColor = '#3b82f6'
<div style={{ color: '#333' }}>
export const colors = { primary: '#e63946' }
```

**Exemplo CORRETO**:
```tsx
className="bg-[var(--color-primary-500,#3b82f6)]"
className="text-primary-500"
```

### 1.2 Buscar objetos JS de estilo

- [ ] Buscar regex `style\s*=\s*\{` em todos os `.tsx` dentro de `components/`
- [ ] Buscar regex `style\s*=\s*\{` em todos os `.tsx` dentro de `app/`
- [ ] Para cada match, anotar: `arquivo:linha — o que o style faz`
- [ ] **COMO IDENTIFICAR**: `style={{ background: ..., color: ..., fontSize: ... }}`
- [ ] **EXCECAO**: `style` usado pra `backgroundImage` com URL dinamica e ACEITO (Tailwind nao faz isso)
- [ ] **EXCECAO**: `style` usado pra CSS vars dinamicas em `layout.tsx` e ACEITO (futuro CMS)

**Exemplo ERRADO**:
```tsx
const variantStyle = variant === 'solid'
  ? { background: primaryColor }
  : { background: secondaryColor }
return <div style={variantStyle}>
```

**Exemplo CORRETO**:
```tsx
const variantClasses = {
  solid: 'bg-[var(--color-primary-500,#3b82f6)]',
  outline: 'border border-[var(--color-primary-500,#3b82f6)]',
}
return <div className={variantClasses[variant]}>
```

### 1.3 Buscar constantes exportadas com cores/estilos

- [ ] Buscar regex `export\s+(const|let)\s+\w*(color|Color|style|Style|theme|Theme|variant|Variant)` em `.tsx` e `.ts` dentro de `components/`
- [ ] Buscar regex `primaryColor|secondaryColor|backgroundColor|textColor` em todos os `.tsx`
- [ ] Para cada match, anotar: `arquivo:linha — nome da constante — valores que contem`
- [ ] **COMO IDENTIFICAR**: constantes que exportam objetos com hex, rgb, ou nomes de cores

**Exemplo ERRADO**:
```tsx
export const buttonStyles = {
  primaryColor: '#e63946',
  secondaryColor: '#6b7280',
  borderRadius: '8px',
}
```

**Exemplo CORRETO**: nao existe. Cores vivem no CSS (`:root`), nao em constantes JS.

### 1.4 Buscar arquivos de design token duplicados

- [ ] Listar TODOS os arquivos que contem "token" ou "theme" ou "design" no nome: `find . -name "*token*" -o -name "*theme*" -o -name "*design*"`
- [ ] Listar TODOS os arquivos `.css` no projeto (exceto `node_modules`): `find . -name "*.css" -not -path "*/node_modules/*"`
- [ ] Verificar: existe mais de 1 arquivo definindo `--color-*` ou `:root`?
- [ ] Verificar: existe arquivo `.ts` E arquivo `.css` definindo as mesmas cores?
- [ ] Anotar: quais arquivos definem tokens e onde estao

**Exemplo ERRADO**: ter `global.css` com `:root { --primary: #xxx }` E `designTokens.ts` com `primary: '#xxx'` E `tailwind.config` com `primary: '#xxx'` — tres fontes da verdade.

**Exemplo CORRETO**: ter apenas `base.css` com `:root { --color-primary-500: #xxx }` e `tailwind.config` referenciando `var(--color-primary-500)`.

---

## PARTE 2 — IDENTIFICAR PROBLEMAS DE ATOMIC DESIGN

> Para CADA componente nos 48 arquivos alterados, verificar classificacao e composicao.

### 2.1 Verificar se atomos estao corretos

Um ATOMO e:
- Um unico elemento HTML (button, input, span, img, a, label)
- NAO importa outros componentes custom
- Tem props de variant, size, className
- Exemplos corretos: Button, Input, Badge, Text, Label, Link, Icon

- [ ] Listar todos os arquivos dentro de `atoms/` (ou equivalente)
- [ ] Para CADA arquivo em atoms/, verificar:
  - [ ] Renderiza apenas 1 elemento HTML principal? (pode ter wrapper div simples)
  - [ ] NAO importa outros componentes custom do projeto?
  - [ ] Se importa outro componente → NAO e atomo, e molecula. Anotar pra mover.
  - [ ] Tem interface de props tipada (TypeScript)?
  - [ ] Tem variants via objeto de classes Tailwind?
  - [ ] Tem sizes via objeto de classes Tailwind?

**COMO IDENTIFICAR atomo errado (deveria ser molecula)**:
```tsx
// ERRADO — isso NAO e atomo, importa outro componente
import { Icon } from '../Icon/Icon'
import { Text } from '../Text/Text'

export const CampaignCard = () => (  // CampaignCard compoe Icon + Text = MOLECULA
  <div>
    <Icon name="star" />
    <Text>Campaign</Text>
  </div>
)
```

**Atomo CORRETO**:
```tsx
export const Button = ({ variant = 'primary', size = 'md', children }: ButtonProps) => {
  const variantClasses = {
    primary: 'bg-[var(--color-primary-500,#3b82f6)] text-white',
    outline: 'border border-[var(--color-primary-500,#3b82f6)]',
  }
  const sizeClasses = { sm: 'px-3 py-2 text-sm', md: 'px-4 py-2.5 text-base' }
  return (
    <button className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]}`}>
      {children}
    </button>
  )
}
```

### 2.2 Verificar se moleculas estao corretas

Uma MOLECULA e:
- Compoe 2+ atomos OU 1 atomo + HTML significativo
- NAO importa outros organismos
- Exemplos corretos: CampaignCard (Image + Text + Badge), FormField (Label + Input + Error), AccordionItem (Text + Icon + conteudo)

- [ ] Listar todos os arquivos dentro de `molecules/` (ou equivalente)
- [ ] Para CADA arquivo em molecules/, verificar:
  - [ ] Importa pelo menos 1 atomo do projeto?
  - [ ] Se NAO importa nenhum componente custom → provavelmente e atomo. Anotar pra mover.
  - [ ] Se importa um organismo → provavelmente e organismo. Anotar pra mover.
  - [ ] Composicao faz sentido? (molecula = grupo reutilizavel de atomos)

**COMO IDENTIFICAR molecula errada (deveria ser atomo)**:
```tsx
// ERRADO — nao compoe nada, e so um div estilizado = ATOMO
export const Container = ({ children }) => (
  <div className="max-w-7xl mx-auto px-4">{children}</div>
)
```

**Molecula CORRETA**:
```tsx
import { Text } from '@/components/atoms'
import { Badge } from '@/components/atoms'
import { Image } from '@/components/atoms'

export const CampaignCard = ({ title, image, status }: CampaignCardProps) => (
  <div className="rounded-lg border overflow-hidden">
    <Image src={image} alt={title} />
    <div className="p-4">
      <Text weight="semibold">{title}</Text>
      <Badge variant={status === 'active' ? 'success' : 'default'}>{status}</Badge>
    </div>
  </div>
)
```

### 2.3 Verificar se organismos estao corretos

Um ORGANISMO e:
- Compoe moleculas + atomos em uma secao completa da pagina
- Tem logica propria (state, handlers)
- Exemplos corretos: Header, Footer, HeroSection, CampaignForm, FAQSection

- [ ] Listar todos os arquivos dentro de `organisms/` (ou equivalente)
- [ ] Para CADA arquivo em organisms/, verificar:
  - [ ] Representa uma secao completa da pagina?
  - [ ] Compoe moleculas E/OU atomos?
  - [ ] Se e apenas 2 atomos simples sem logica → provavelmente e molecula. Anotar.

### 2.4 Verificar componentes fora da hierarquia

- [ ] Existem componentes soltos na raiz de `components/` (fora de atoms/molecules/organisms)?
- [ ] Se sim, para CADA um classificar: e atomo, molecula ou organismo?
- [ ] Anotar onde deveria estar

---

## PARTE 3 — VERIFICAR ESTRUTURA DE ARQUIVOS

### 3.1 Naming conventions

- [ ] Para CADA componente, verificar: nome do arquivo e PascalCase? (ex: `Button.tsx`, nao `button.tsx`)
- [ ] Para CADA componente, verificar: pasta tem mesmo nome do componente? (ex: `Button/Button.tsx`)
- [ ] Existem componentes sem pasta propria? (ex: `atoms/Button.tsx` solto, sem `atoms/Button/Button.tsx`)
- [ ] Anotar todos os que nao seguem o padrao

**Padrao CORRETO**:
```
atoms/
  Button/
    Button.tsx
  Input/
    Input.tsx
  Badge/
    Badge.tsx
```

**Padrao ERRADO**:
```
atoms/
  button.tsx          ← minusculo
  MyButton.tsx        ← solto, sem pasta
  components/
    Button.tsx        ← aninhamento errado
```

### 3.2 Export patterns

- [ ] Para CADA componente, verificar: usa `export const` (named export)?
- [ ] Algum componente usa `export default`? Se sim, anotar. Padrao correto e named export.
- [ ] Existe barrel file (`index.ts`) em `atoms/`, `molecules/`, `organisms/`?
- [ ] Se existe, todos os componentes da pasta estao listados no barrel?

**Export CORRETO**:
```tsx
// Button.tsx
export const Button = ({ ... }: ButtonProps) => { ... }

// atoms/index.ts
export { Button } from './Button/Button'
export { Input } from './Input/Input'
```

**Export ERRADO**:
```tsx
// Button.tsx
export default function Button() { ... }  // ← default export
```

### 3.3 Interface de props

- [ ] Para CADA componente, verificar: tem interface `XxxProps` exportada?
- [ ] A interface usa tipos explicitos (nao `any`)?
- [ ] Props opcionais tem `?` e valor default no destructuring?

**Props CORRETAS**:
```tsx
export interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  children: React.ReactNode
  className?: string
  isLoading?: boolean
}

export const Button = ({
  variant = 'primary',
  size = 'md',
  children,
  className = '',
  isLoading = false,
}: ButtonProps) => { ... }
```

**Props ERRADAS**:
```tsx
// Sem interface
export const Button = (props: any) => { ... }

// Props inline sem export
const Button = ({ color = 'red' }: { color: string }) => { ... }
```

---

## PARTE 4 — PADRONIZAR CSS (a parte principal)

> Para CADA componente dos 48 arquivos, executar TODOS os checks abaixo.

### 4.1 Verificar padrao de variant classes

- [ ] O componente tem variantes visuais (primary/secondary, solid/outline, etc)?
- [ ] Se sim, as variantes estao como objeto de Tailwind classes?

**Padrao CORRETO**:
```tsx
const variantClasses = {
  primary: 'bg-[var(--color-primary-500,#3b82f6)] text-white hover:bg-[var(--color-primary-600,#2563eb)]',
  secondary: 'bg-[var(--color-gray-500,#6b7280)] text-white',
  outline: 'border border-[var(--color-primary-500,#3b82f6)] text-[var(--color-primary-500,#3b82f6)]',
  ghost: 'text-[var(--color-primary-500,#3b82f6)] hover:bg-[var(--color-primary-50,#eff6ff)]',
}
```

**Padrao ERRADO (ternario inline)**:
```tsx
// ERRADO — ternario com style object
const variantStyle = variant === 'solid'
  ? { background: primary, color: 'white' }
  : { background: 'transparent', border: `1px solid ${primary}` }
```

**Padrao ERRADO (hex direto)**:
```tsx
// ERRADO — hex hardcoded
const variantClasses = {
  primary: 'bg-[#e63946] text-white',  // ← hex direto, sem CSS var
}
```

- [ ] Se o componente usa ternario com objetos de estilo → reescrever como objeto de classes Tailwind
- [ ] Se o componente usa hex hardcoded em classes → substituir por `var(--token, #fallback)`

### 4.2 Verificar padrao de size classes

- [ ] O componente tem tamanhos (sm/md/lg)?
- [ ] Se sim, os tamanhos estao como objeto de Tailwind classes?

**Padrao CORRETO**:
```tsx
const sizeClasses = {
  sm: 'px-3 py-2 text-sm',
  md: 'px-4 py-2.5 text-base',
  lg: 'px-6 py-3 text-lg',
}
```

**Padrao ERRADO**:
```tsx
// ERRADO — calculo JS
const fontSize = size === 'sm' ? '12px' : size === 'md' ? '16px' : '20px'
return <div style={{ fontSize }}>
```

- [ ] Se o componente calcula tamanho em JS → reescrever como objeto de classes Tailwind

### 4.3 Verificar montagem final do className

- [ ] O componente combina classes numa unica template literal?
- [ ] A ordem e: `base → variant → size → state → className`?

**Padrao CORRETO**:
```tsx
const classes = `${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${stateClasses} ${className}`
return <button className={classes}>
```

**Padrao ERRADO**:
```tsx
// ERRADO — className espalhado em multiplos elementos sem logica clara
return (
  <div className="flex">
    <span className={isPrimary ? 'text-blue-500' : 'text-gray-500'}>  // ← ternario inline
      <button style={{ padding: '8px 16px' }}>  // ← style inline
```

- [ ] Se classes estao espalhadas sem padrao → consolidar num unico ponto

### 4.4 Verificar CSS vars com fallback

- [ ] Toda cor no componente usa `var(--token, #fallback)`?
- [ ] O fallback e o hex correto (mesmo valor definido no `:root`)?

**CORRETO**: `bg-[var(--color-primary-500,#3b82f6)]`
**ERRADO**: `bg-[var(--color-primary-500)]` (sem fallback)
**ERRADO**: `bg-[#3b82f6]` (sem var, hex direto)
**ERRADO**: `bg-primary-500` que aponta pra hex fixo no tailwind.config (nao usa var)

- [ ] Se falta fallback → adicionar
- [ ] Se usa hex direto → envolver com `var(--token, #hex)`
- [ ] Se usa classe Tailwind que aponta pra hex fixo no config → ok por agora, mas anotar pra futuro

### 4.5 Verificar que NAO existe style={{}} (exceto excecoes)

- [ ] O componente usa `style={{ }}` em algum elemento?
- [ ] Se sim, e uma das excecoes permitidas?
  - `backgroundImage: url(...)` com URL dinamica → ACEITO
  - CSS vars dinamicas vindas de props (futuro CMS) → ACEITO
  - Qualquer outra coisa → ERRADO, converter pra className

### 4.6 Verificar imports de estilos

- [ ] O componente importa algum arquivo `.css` ou `.module.css` proprio?
- [ ] O componente importa algum arquivo de "tokens" ou "styles" em JS/TS?
- [ ] Se sim, anotar. Componente nao deve importar tokens diretamente — consome via Tailwind classes.

**ERRADO**:
```tsx
import { colors } from '@/design-tokens'  // ← componente nao importa tokens
import styles from './Button.module.css'   // ← CSS modules nao e o padrao
import './Button.css'                      // ← CSS proprio por componente nao e o padrao
```

**CORRETO**: componente nao importa nada de estilo. So usa className com Tailwind.

---

## PARTE 5 — ARQUIVO GLOBAL DE TOKENS

### 5.1 Verificar fonte unica de CSS vars

- [ ] Existe UM unico arquivo CSS com `:root { --color-* }`?
- [ ] Qual arquivo? Anotar path.
- [ ] Esse arquivo e importado no `globals.css`?
- [ ] Nenhum outro arquivo define `--color-*`?
- [ ] Se existem multiplos arquivos com `:root` → anotar pra consolidar

### 5.2 Verificar tailwind.config

- [ ] O `tailwind.config` tem cores definidas?
- [ ] As cores referenciam CSS vars (`var(--color-*)`) ou sao hex fixos?
- [ ] Se sao hex fixos → anotar. Ideal e referenciar vars.

**Config IDEAL**:
```js
colors: {
  primary: {
    500: 'var(--color-primary-500)',
    600: 'var(--color-primary-600)',
  }
}
```

**Config ACEITAVEL (por agora)**:
```js
colors: {
  primary: {
    500: '#3b82f6',  // hex fixo — funciona, mas nao responde a tema
  }
}
```

### 5.3 Verificar global.css

- [ ] `globals.css` importa o arquivo de tokens/base.css?
- [ ] `globals.css` tem as 3 diretivas do Tailwind (`@tailwind base/components/utilities`)?
- [ ] `body` usa CSS vars pra background/color/font?

---

## PARTE 6 — GERAR RELATORIO

Depois de executar TODAS as partes acima, gerar relatorio neste formato:

```markdown
# Relatorio de Revisao CSS + Atomic Design

## Resumo
- Total de arquivos revisados: ___
- Hex hardcoded encontrados: ___ ocorrencias em ___ arquivos
- Style objects encontrados: ___ ocorrencias em ___ arquivos
- Constantes de cor exportadas: ___ ocorrencias em ___ arquivos
- Fontes de tokens duplicadas: ___ arquivos
- Componentes mal classificados: ___ (lista abaixo)

## Hex hardcoded (remover)
| Arquivo | Linha | Valor | Contexto | Correcao |
|---|---|---|---|---|

## Style objects (converter pra className)
| Arquivo | Linha | O que faz | Correcao |
|---|---|---|---|

## Constantes de cor (eliminar)
| Arquivo | Linha | Constante | Correcao |
|---|---|---|---|

## Componentes mal classificados
| Componente | Classificacao atual | Classificacao correta | Motivo |
|---|---|---|---|

## Fontes de tokens duplicadas
| Arquivo | O que define | Manter ou remover |
|---|---|---|

## Arquivos sem padrao de naming
| Arquivo | Problema | Correcao |
|---|---|---|

## Prioridade de correcao
1. (listar por impacto: primeiro tokens duplicados, depois hex hardcoded, etc)
```

---

## REGRAS ANTI-ALUCINACAO

1. **NAO invente nomes de arquivo**. Se nao encontrou o arquivo, diga "nao encontrado".
2. **NAO assuma que um padrao existe**. Busque com grep/find antes de afirmar.
3. **NAO corrija codigo nesta etapa**. Esta etapa e SO DE DIAGNOSTICO. Anotar problemas, nao corrigir.
4. **NAO pule componentes**. Cada um dos 48 arquivos deve ser verificado.
5. **NAO misture diagnostico com correcao**. Primeiro roda o checklist inteiro, depois corrige.
6. **Se nao tem certeza se algo e erro**, anote como "DUVIDA" com contexto pra humano decidir.
7. **Cite SEMPRE arquivo:linha** — nunca "em alguns componentes" sem especificar quais.
8. **Um checkbox = uma busca = um resultado anotado**. Nao agrupe.

---

## REFERENCIA RAPIDA — template-saas-ecommerce

Quando tiver duvida de como algo deveria ser, consulte esses arquivos:

| Duvida | Arquivo de referencia |
|---|---|
| Como fazer variants de cor | `template-saas-ecommerce/src/components/atoms/Button/Button.tsx` |
| Como fazer sizes | `template-saas-ecommerce/src/components/atoms/Input/Input.tsx` |
| Como fazer estado de erro | `template-saas-ecommerce/src/components/atoms/Input/Input.tsx` |
| Como compor molecula | `template-saas-ecommerce/src/components/molecules/Card/Card.tsx` |
| Como organizar organismo | `template-saas-ecommerce/src/components/organisms/Header/Header.tsx` |
| Como definir CSS vars | `template-saas-ecommerce/src/design-system/themes/base.css` |
| Como configurar Tailwind | `template-saas-ecommerce/tailwind.config.ts` |
| Como exportar componentes | `template-saas-ecommerce/src/components/atoms/index.ts` |
| Como fazer accordion | `template-saas-ecommerce/src/components/organisms/FAQSection/FAQSection.tsx` |
| Como fazer props tipadas | `template-saas-ecommerce/src/components/atoms/Badge/Badge.tsx` |

---

## PARTE 7 — AUTO-AVALIACAO DO TRABALHO

> Depois de executar TODAS as partes (1-6), avalie a qualidade do seu proprio trabalho.
> Preencha a tabela abaixo com honestidade. A nota deve refletir o que REALMENTE foi feito, nao o que voce pretendia fazer.

### Como dar a nota

Para CADA criterio, siga a rubrica. Nao "arredonde pra cima". Se fez 80% dos arquivos, a nota e 8, nao 9.

| Nota | Significado |
|---|---|
| **10** | Perfeito. 100% dos itens verificados, zero item pulado, zero afirmacao sem evidencia, relatorio completo com arquivo:linha em todos os achados |
| **9** | Excelente. 95%+ verificado, no maximo 1-2 itens com evidencia incompleta, relatorio quase completo |
| **8** | Bom. 85%+ verificado, alguns itens sem arquivo:linha, relatorio tem gaps menores |
| **7** | Aceitavel. 70%+ verificado, varias afirmacoes genericas sem especificar arquivo, relatorio incompleto |
| **6** | Insuficiente. Menos de 70% verificado, ou afirmacoes sem evidencia, ou itens pulados sem justificativa |
| **≤5** | Falhou. Pulou partes inteiras, inventou dados, ou nao seguiu o checklist |

### Criterios de avaliacao

Preencha esta tabela no final do relatorio:

```markdown
## Auto-avaliacao

| # | Criterio | Referencia de qualidade | Nota (1-10) | Justificativa |
|---|---|---|---|---|
| 1 | **Cobertura de arquivos** | Todos os 48 arquivos alterados foram verificados individualmente? Nenhum pulado? | ___ | ___ |
| 2 | **Evidencia concreta** | Todo achado tem `arquivo:linha`? Nenhuma afirmacao generica tipo "em varios componentes"? | ___ | ___ |
| 3 | **Precisao de classificacao Atomic** | Cada componente foi classificado corretamente (atomo/molecula/organismo) com justificativa baseada em imports e composicao? Ref: Brad Frost "Atomic Design" — atomo = elemento HTML unico, molecula = grupo de atomos, organismo = secao completa com logica | ___ | ___ |
| 4 | **Deteccao de CSS redundante** | Encontrou TODAS as 4 formas de declaracao (hex hardcoded, style objects, constantes exportadas, tokens duplicados)? Usou regex/grep real, nao olho? Ref: Tailwind docs "Reusing Styles" — unica forma correta e className com utility classes | ___ | ___ |
| 5 | **Qualidade das correcoes sugeridas** | Cada problema tem uma correcao especifica (nao generica)? A correcao segue o padrao var(--token, #fallback)? Ref: MDN CSS Custom Properties + Tailwind "Arbitrary Values" | ___ | ___ |
| 6 | **Padrao de variant/size classes** | Verificou se CADA componente com variantes usa objeto de classes Tailwind (nao ternario com style)? Ref: template-saas-ecommerce Button.tsx, Badge.tsx — variantClasses como objeto, sizeClasses como objeto, combinados em template literal | ___ | ___ |
| 7 | **Naming e estrutura de pastas** | Verificou PascalCase, pasta por componente, named exports, barrel files? Ref: React community convention (PascalCase) + template-saas-ecommerce atoms/index.ts | ___ | ___ |
| 8 | **TypeScript e props** | Verificou interfaces exportadas, tipos explicitos (sem `any`), defaults no destructuring? Ref: TypeScript Handbook "Interfaces" + React TypeScript Cheatsheet | ___ | ___ |
| 9 | **Completude do relatorio** | O relatorio da Parte 6 esta completo? Todas as tabelas preenchidas? Resumo com numeros reais? Prioridade de correcao definida? | ___ | ___ |
| 10 | **Disciplina anti-alucinacao** | Seguiu as 8 regras? Nao inventou arquivo, nao assumiu padrao sem buscar, nao corrigiu codigo, nao agrupou achados? | ___ | ___ |

### Media final: ___ / 10
```

### Regras da auto-avaliacao

1. **Nota sem justificativa nao vale.** Se der 9, explique por que nao e 10. Se der 7, explique o que faltou.
2. **A justificativa deve citar exemplos concretos.** "Verifiquei todos" nao e justificativa. "Verifiquei 48/48 arquivos, listados na secao X do relatorio" e.
3. **Se a media for menor que 8, liste o que faria diferente** pra chegar a 9+ numa segunda rodada.
4. **Se a media for 9+, liste 1-2 melhorias residuais** que fariam chegar a 10.
5. **NAO infle a nota.** E melhor dar 7 honesto com plano de melhoria do que dar 9 inventado. O humano vai conferir.

### Referencias de boas praticas usadas na avaliacao

| Referencia | O que valida | Onde aplicar |
|---|---|---|
| **Brad Frost — Atomic Design** (bradfrost.com/blog/post/atomic-web-design) | Classificacao atomo/molecula/organismo. Atomo = indivisivel, molecula = grupo funcional, organismo = secao com identidade | Criterio 3 |
| **Tailwind CSS — Reusing Styles** (tailwindcss.com/docs/reusing-styles) | Utility classes como unica forma de estilizar. Sem CSS-in-JS, sem style objects, sem constantes de cor | Criterio 4 |
| **MDN — CSS Custom Properties** (developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) | `var(--token, fallback)` como padrao. Fallback obrigatorio. Definicao em `:root` | Criterio 5 |
| **Tailwind CSS — Arbitrary Values** (tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values) | Sintaxe `bg-[var(--color-X,#hex)]` pra integrar CSS vars com Tailwind | Criterio 5, 6 |
| **React TypeScript Cheatsheet** (react-typescript-cheatsheet.netlify.app) | Props interfaces, named exports, generics em componentes | Criterio 8 |
| **template-saas-ecommerce** (repo local de referencia) | Pattern completo: tokens em CSS, Tailwind config, variant/size objects, composicao Atomic | Criterios 3-8 |
| **Clean Code (Robert C. Martin)** — naming | PascalCase pra componentes, camelCase pra variaveis, nomes descritivos | Criterio 7 |
| **SOLID — Single Responsibility** | Cada componente faz uma coisa. Atomo nao compoe, molecula nao tem logica de pagina | Criterio 3 |

### Exemplo de auto-avaliacao nota 9

```
| 1 | Cobertura de arquivos | 9 | 47/48 verificados. Arquivo X.tsx nao foi encontrado no path indicado — pode ter sido renomeado. |
| 4 | Deteccao de CSS redundante | 9 | Encontrei 23 hex hardcoded, 8 style objects, 3 constantes. Usei grep com regex. Posso ter perdido hex dentro de template literals complexas. |
| 10 | Disciplina anti-alucinacao | 10 | Nenhum arquivo inventado. Todos os achados com path:linha. 2 itens marcados como DUVIDA pro humano. |
```

### Exemplo de auto-avaliacao nota 6 (honesta)

```
| 1 | Cobertura de arquivos | 6 | Verifiquei 30/48. Pulei os 18 da pasta organisms/ por limite de contexto. |
| 5 | Qualidade das correcoes | 6 | Algumas correcoes sao genericas ("usar Tailwind"). Faltou especificar a classe exata de substituicao. |
| 10 | Disciplina anti-alucinacao | 7 | Em 3 achados nao especifiquei a linha exata, apenas o arquivo. |
```
