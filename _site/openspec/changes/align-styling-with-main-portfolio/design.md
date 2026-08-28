## Context

O site atual é um tema Jekyll chamado **plainwhite**. A estilização está distribuída em:
- `_sass/plain.scss` — estilos globais (fonte body: Raleway; cor de link: `#0a59b0`)
- `_sass/dark.scss` — overrides do modo escuro (fundo: `#171717`; texto: `#f6f6f6`)
- `_sass/ext/_fonts.scss` — apenas a font-face do ícone Fontello (sem Google Fonts)
- `assets/css/style.scss` — entry point que importa tudo

O portfólio principal (`portfolio-tailwindcss`) usa CSS custom properties (`--bg-primary`, `--accent-primary`, etc.) com fontes **Manrope** e **JetBrains Mono** via Google Fonts. Veja `proposal.md` para motivação.

## Goals / Non-Goals

**Goals:**
- Substituir a fonte `Raleway` por `Manrope` em todo o site
- Substituir a fonte monospace padrão por `JetBrains Mono` em blocos de código
- Atualizar a paleta de cores do modo escuro para a paleta do portfólio principal
- Tornar o tema escuro o padrão (remover dependência do toggle `.dark`)

**Non-Goals:**
- Alterar o layout, margens ou estrutura de componentes
- Migrar para Tailwind CSS
- Criar ou modificar temas claro (o site passará a ter apenas modo escuro)
- Alterar os ícones Fontello

## Decisions

### 1. Variáveis SASS em vez de CSS custom properties
O projeto já usa SASS e não tem suporte a CSS custom properties no pipeline atual. Introduzir CSS variables exigiria refatoração extensiva. Optamos por usar variáveis SASS (`$bg-primary`, `$accent-primary`, etc.) que mapeiam os mesmos tokens do portfólio principal, mantendo a abordagem existente e minimizando o risco.

**Alternativa considerada:** CSS custom properties — descartada porque exigiria mudanças em todos os seletores e potencial incompatibilidade com o plugin de syntax highlight.

### 2. Google Fonts via `@import` no SASS
A forma mais simples de importar Manrope e JetBrains Mono é adicionar um `@import` do Google Fonts no topo de `_sass/ext/_fonts.scss`, que já é o arquivo de fontes do projeto. Isso mantém a organização existente.

**Alternativa considerada:** `<link rel="preconnect">` no `_includes/head.html` — também válida e mais performática, mas requer edição de HTML. Ficará como Open Question.

### 3. Tema escuro como único tema (remover toggle)
O `dark.scss` atual é ativado via classe `.dark` no `body`, controlada por JS. Em vez de manter o toggle, aplicaremos os estilos escuros diretamente no seletor `body`, tornando o escuro o padrão sem dependência de JS. O arquivo `toggle.scss` e o script de toggle não serão removidos nesta fase para preservar compatibilidade — apenas o CSS do modo escuro passa a ser default.

**Alternativa considerada:** Manter o toggle e apenas atualizar as cores dentro de `.dark` — mais conservador, mas não cumpre a spec de "modo escuro como único tema".

## Risks / Trade-offs

- **Risco: Contraste insuficiente em alguns componentes** → A paleta é toda escura; alguns elementos de borda ou ícones podem ficar invisíveis. Mitigação: revisar visualmente após aplicar.
- **Risco: Fonte Manrope não carregar offline** → O Google Fonts pode falhar sem internet. Mitigação: declarar fallback adequado (`-apple-system, BlinkMacSystemFont, sans-serif`).
- **Risco: Quebra do syntax highlight** → `_syntax.scss` e `_sass/ext/_solarized-dark.scss` têm cores próprias que podem conflitar. Mitigação: manter os arquivos de syntax highlight inalterados nesta fase.

## Migration Plan

1. Adicionar import do Google Fonts em `_sass/ext/_fonts.scss`
2. Criar arquivo de variáveis SASS `_sass/_variables.scss` com os tokens de cor e tipografia
3. Atualizar `_sass/plain.scss` para usar as novas variáveis (fonte body, cores de link e texto)
4. Reescrever `_sass/dark.scss` para aplicar os tokens no seletor `body` diretamente (sem `.dark`)
5. Importar `_variables.scss` no entry point `assets/css/style.scss`
6. Build e revisão visual (`bundle exec jekyll serve`)

**Rollback:** Git revert nos arquivos SASS modificados — sem mudanças de schema ou dados.

## Open Questions

- Usar `<link rel="preconnect">` no `<head>` HTML em vez de `@import` CSS para melhor performance de carregamento das fontes? Pode ser feito em uma iteração posterior sem afetar os requisitos.
