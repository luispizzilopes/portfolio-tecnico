## 1. Fontes

- [x] 1.1 Adicionar `@import` do Google Fonts (Manrope e JetBrains Mono com os pesos corretos) no início de `_sass/ext/_fonts.scss` e verificar que o build Jekyll (`bundle exec jekyll build`) não reporta erros de CSS
- [x] 1.2 Verificar visualmente no browser que as fontes Manrope e JetBrains Mono estão sendo aplicadas (inspecionar elemento no body e em um bloco `<code>`)

## 2. Variáveis de cor e tipografia

- [x] 2.1 Criar o arquivo `_sass/_variables.scss` com todas as variáveis SASS mapeando os tokens de cor e tipografia do portfólio principal (conforme listado em `proposal.md`) e verificar que o arquivo é criado sem erros de sintaxe SASS
- [x] 2.2 Adicionar `@import "variables"` no entry point `assets/css/style.scss` antes dos outros imports e verificar que o build não reporta variáveis indefinidas

## 3. Estilos globais (plain.scss)

- [x] 3.1 Substituir a declaração `font-family` do `body` em `_sass/plain.scss` de `"Raleway"` para `"Manrope"` com o fallback correto e verificar no browser que o texto do body usa Manrope
- [x] 3.2 Substituir a variável `$linkColor` e os valores de cor de `a` em `_sass/plain.scss` pelos tokens `$accent-primary` e `$text-primary` e verificar que os links exibem a cor `#3b82f6`
- [x] 3.3 Aplicar a cor de fundo `$bg-primary` e a cor de texto `$text-primary` ao seletor `body` em `_sass/plain.scss` e verificar que o fundo da página é `#0a0e1a`

## 4. Tema escuro (dark.scss)

- [x] 4.1 Reescrever `_sass/dark.scss` aplicando os tokens de cor do portfólio principal diretamente no seletor `body` (sem depender da classe `.dark`): fundo `$bg-primary`, texto `$text-primary`, borda `$border-color` e verificar que o tema escuro é exibido sem necessidade de toggle
- [x] 4.2 Atualizar os sub-seletores de `dark.scss` (posts, labels, search, tabelas) para usar os novos tokens (`$bg-secondary`, `$bg-tertiary`, `$text-secondary`, `$text-muted`) e verificar visualmente que os componentes estão legíveis
- [x] 4.3 Atualizar o seletor de fonte monospace em `dark.scss` (bloco `pre.highlight-dark, code`) para usar `"JetBrains Mono", monospace` e verificar em um post com código que a fonte foi aplicada

## 5. Verificação final

- [x] 5.1 Rodar `bundle exec jekyll serve` e navegar pelas páginas principais (home, posts, about) verificando que: fundo é `#0a0e1a`, texto principal é `#f1f5f9`, links são `#3b82f6`, fontes são Manrope e JetBrains Mono, e nenhum elemento ficou com contraste insuficiente
- [x] 5.2 Verificar no DevTools (aba Network → Fonts) que Manrope e JetBrains Mono são carregadas com sucesso do Google Fonts
