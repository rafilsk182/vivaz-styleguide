# Pente fino Figma × Style Guide

**Data:** 23/04/2026
**Referência:** Arquivo Figma "Protótipos - Validação" (`HvlBUS26BKcErgLLrUHtEL`) — página Versão final / Ready for development
**Análise:** via Claude in Chrome no Dev Mode, Inspect → Code (CSS)
**Destino:** site Vivaz em Drupal (Pantheon + Buddy + Bitbucket)

---

## Resumo executivo

O Figma está **muito mais maduro** do que o style guide atual reflete. O arquivo usa um sistema de **variáveis nomeadas com modes** (temas Light/Dark + tamanhos responsivos automáticos) — o que é exatamente a arquitetura que um tema Drupal precisa. Nosso style guide hoje tem tokens soltos, nomes genéricos (`--color-roxo-500`) e valores que **não batem** com o que o Figma usa em produção.

A boa notícia: portar isso para Drupal fica mais fácil agora, porque podemos copiar a estrutura semântica do Figma quase 1:1.

---

## 1. Descobertas críticas (o que está errado hoje)

### 1.1. Cor de fundo dark está errada

| Token atual no style guide | Valor atual | Valor real no Figma | Fonte |
|---|---|---|---|
| `--color-dark-900` (usado como BG do dark theme) | `#0A0C10` | `#4D4D4F` | `var(--BG-Default, #4D4D4F)` exposto no Inspect da Home Desktop |

**Impacto:** tudo que usa `--color-dark-900` como fundo está renderizando mais preto do que o Figma especifica. O dark real é um cinza escuro médio, não preto puro.

### 1.2. Escala de spacing está sobredimensionada

Nosso style guide tem `--space-1` até `--space-20` (4px → 80px) com 20 níveis. O Figma usa uma escala enxuta com prefixo `--x`:

| Variável Figma | Valor observado |
|---|---|
| `--x0` | 0 |
| `--x3` | 24px (gap de cards, padding top) |
| `--x4` | 32px (gap da seção Cards Empreendimento) |
| `--xMargins` | responsivo (margem lateral que muda por breakpoint — 51px em desktop) |

**Padrão provável** (múltiplos de 8): `--x1 = 8, --x2 = 16, --x3 = 24, --x4 = 32, --x5 = 40, --x6 = 48...`

**Recomendação:** simplificar nossa escala para `--x0` a `--x8` + adicionar `--x-margins` responsivo via `@media`. Drupal gosta dessa nomenclatura curta porque facilita escrever utility classes.

### 1.3. O "Roxo" não é Primary — é Secondary

Olhar o painel de variáveis do Figma revelou uma nomenclatura com implicações semânticas:

- `Primary Roxo` existe — mas está sendo usado em **gradientes**, não em componentes base
- `Primary Turquesa` também — mesmo padrão
- As cores **usadas nos CTAs / botões principais** estão sob nomes como `Secondary-Purple-Yellow`, `Secondary-Yellow-White`, `Secondary-Purple-White` — indicando tokens **contextuais** que mudam dependendo do tema (light/dark) ou do fundo

**Conclusão:** o sistema Vivaz trata o roxo como cor secundária/contextual, e as primárias são reservadas para gradientes de marca. Nosso `--color-roxo-500: #960064` está certo como valor, mas o **nome semântico** precisa mudar.

### 1.4. Tokens hardcoded que deveriam existir no style guide

Durante a navegação apareceram valores que **não estão** no nosso `:root`:

| Valor | Onde aparece no Figma | Proposta de token |
|---|---|---|
| `#2FDADA` | Usada solta em elementos | `--color-turquesa-400` (hoje só temos 500) |
| `#51A7E1` | Gradientes + ilustrações | `--color-azul-300` (não existe hoje) |
| `#3146A3` | Gradientes + ilustrações | `--color-azul-700` (mesmo `--color-info` atual) |
| `#6EBB40` | Gradientes feedback positivo | `--color-verde-500` |
| `#1F6743` | Gradientes feedback positivo escuro | `--color-verde-800` (mesmo `--color-success` atual) |
| `#ED8721` | Gradientes laranja | `--color-laranja-500` |
| `#EDEE27` | Amarelo esverdeado (feedback?) | revisar se é realmente necessário |

---

## 2. Arquitetura de tokens recomendada (3 camadas)

O Figma já faz isso — nosso style guide precisa seguir o mesmo padrão. Isso é **essencial** para um tema Drupal bem mantido.

### Camada 1 — Primitivos (`--viv-*`)

Valores brutos. Nunca consumidos diretamente em componentes.

```css
:root {
  /* Marca */
  --viv-white:            #FFFFFF;
  --viv-black:            #000000;
  --viv-yellow:           #FFD200;
  --viv-purple:           #960064;
  --viv-turquoise:        #32C8C8;
  --viv-feedback-green:   #38C12C;

  /* Grays */
  --viv-grey-1:           #F4F4F4;
  --viv-grey-3:           #ACAEB2;
  --viv-text-grey:        #66676A;
  --viv-text-black:       #0A0C10;

  /* Alphas */
  --viv-white-75:         rgba(255,255,255,0.75);
}
```

### Camada 2 — Semânticos (o que os componentes usam)

Dão nome ao **papel** da cor. Mudam automaticamente entre light/dark.

```css
/* Tema dark (padrão no projeto Vivaz) */
:root {
  --bg-default:       #4D4D4F;       /* CORRIGIDO */
  --bg-g1-g3:         /* gradiente */;
  --bg-g2-purp:       /* gradiente */;
  --bg-white-purple:  /* gradiente */;

  --text:             var(--viv-white);
  --text-2:           var(--viv-grey-3);
  --text-3:           var(--viv-text-grey);

  --outline:          rgba(255,255,255,0.2);
}

[data-theme="light"] {
  --bg-default:       var(--viv-white);
  --text:             var(--viv-text-black);
  --text-2:           var(--viv-text-grey);
  --outline:          rgba(0,0,0,0.12);
}
```

### Camada 3 — Contextuais (combinações específicas)

Pares de cor que o Figma já define como `Secondary-Purple-Yellow`, etc. — usados para botões e badges que precisam de comportamento específico:

```css
:root {
  --secondary-purple-yellow: /* botão roxo em fundo amarelo */;
  --secondary-yellow-white:  /* botão amarelo em fundo branco */;
  --secondary-purple-white:  /* botão roxo em fundo branco */;
}
```

---

## 3. Tipografia — padrão "Auto (Size)" do Figma

O inspetor mostrou `Typography: Auto (Size)` e `Type Scale: Auto (Mode 1)`. Isso significa que o Figma tem escala tipográfica **responsiva automática**.

**Recomendação para o style guide / Drupal:** implementar via `clamp()` com base em breakpoints:

```css
:root {
  --fs-display:  clamp(2.5rem, 4vw + 1rem, 4rem);    /* 40px → 64px */
  --fs-h1:       clamp(2rem, 3vw + 1rem, 3rem);      /* 32px → 48px */
  --fs-h2:       clamp(1.5rem, 2vw + 0.75rem, 2.25rem);
  --fs-body:     1rem;                               /* 16px fixo */
  --fs-small:    0.875rem;                           /* 14px */
  --fs-caption:  0.75rem;                            /* 12px */
}
```

Falta validação de line-heights e letter-spacings (ficaram para uma segunda rodada de inspeção se você quiser).

---

## 4. Componentes faltando no style guide

Frames aprovados para dev (desktop + mobile) que têm componentes ainda **não documentados** no `index.html`:

| Componente | Visto em | Prioridade |
|---|---|---|
| Menu Mobile | Frame `2993-52407` | Alta — só existe em mobile |
| Hero (principal + PDP) | Home + Página de Produto | Alta — aparece em quase toda página |
| Filter Bar (PDP / Categoria) | `FilterBar-PDP` | Alta — base da Categoria |
| Pagination (`1 de 36`) | Rodapé dos Wireframes Mobile | Média |
| Breadcrumbs | PDP Desktop | Média |
| Formulário Simulador | `2993-59750` / `2993-59783` | Alta |
| Modal / Dialog | Vários | Média |
| Accordion / FAQ | Sobre a Vivaz | Média |
| Tabs | Instituto Cyrela | Baixa |
| Toggle / Checkbox / Radio | Simulador + Filtros | Alta |
| Footer | Todos | Alta |
| Timeline | Instituto Cyrela | Baixa |
| Card Empreendimento — variante desktop | Seção home | Média (hoje só tem mobile) |

---

## 5. Recomendações específicas para Drupal

### 5.1. Usar CSS custom properties em `:root`, não Sass variables
Drupal 10/11 com temas baseados em Olivero/Claro se beneficia de **variáveis CSS nativas** — permite trocar tema via atributo (`[data-theme="dark"]`) sem rebuild. Evitar `$variable` de Sass para tokens semânticos.

### 5.2. Nomenclatura BEM-compatível
Os nomes do Figma (`Secondary-Purple-Yellow`) viram `.btn--secondary-purple-yellow` direto no Twig. Manter esse mapeamento 1:1 facilita handoff designer ↔ dev.

### 5.3. Focus states (WCAG 2.2 AA)
**Não identificados no Figma ainda** — precisa perguntar à designer. Mas o style guide precisa de:

```css
:focus-visible {
  outline: 2px solid var(--viv-yellow);
  outline-offset: 2px;
}
```

### 5.4. Breakpoints
O Figma tem desktop 1366px e mobile 375px. Sugestão:

```css
--bp-mobile:   375px;
--bp-tablet:   768px;
--bp-desktop:  1024px;
--bp-wide:     1366px;
```

### 5.5. Z-index scale
Faltando hoje. Proposta:

```css
--z-base:     1;
--z-dropdown: 100;
--z-sticky:   200;
--z-overlay:  300;
--z-modal:    400;
--z-toast:    500;
```

### 5.6. Transitions
Faltando hoje. Proposta:

```css
--transition-fast: 120ms ease-out;
--transition-base: 200ms ease-out;
--transition-slow: 400ms ease-in-out;
```

---

## 6. Próximos passos sugeridos

1. **Validar com designer Vivaz** os valores que inferi (especialmente `--x1` a `--x8`, line-heights, e os nomes da camada Secondary).
2. **Refatorar `:root`** no `index.html` para a arquitetura de 3 camadas — posso fazer isso no próximo passo.
3. **Corrigir `--color-dark-900`** de `#0A0C10` para `#4D4D4F` (ou renomear e criar o novo token semântico `--bg-default`).
4. **Documentar os componentes faltantes** no style guide — sugiro começar por Menu Mobile, Hero, Filter Bar e Footer (os mais recorrentes).
5. **Abrir a tabela de variáveis do Figma** com a designer presente para capturar os 40 tokens completos (o Dev Mode mostra só as variáveis usadas na seleção atual).

---

*Gerado em 2026-04-23 por inspeção direta no Figma via Claude in Chrome · Usa apenas valores observados no painel Inspect → Code (CSS).*
