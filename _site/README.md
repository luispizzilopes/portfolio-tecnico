# Luis Felipe — Dev Blog

Blog técnico pessoal construído com Jekyll, baseado no tema [plainwhite](https://github.com/samarsault/plainwhite-jekyll), com identidade visual personalizada alinhada ao portfólio principal.

## Visão geral

Site estático que reúne publicações sobre código, arquitetura, ferramentas e aprendizados do dia a dia como desenvolvedor Full Stack. Não depende de banco de dados nem de servidor de aplicação — o build gera HTML puro.

## Tecnologias

- **Jekyll** — gerador de site estático
- **SASS** — pré-processador CSS
- **Tema plainwhite** (v0.13) — base de layout e estrutura
- **Google Fonts** — Manrope (texto) e JetBrains Mono (código)
- **jekyll-seo-tag** — meta tags de SEO automáticas

## Estrutura do projeto

```
├── _config.yml          # Configurações do site (título, autor, redes sociais)
├── _layouts/            # Templates HTML (default, home, page, post)
├── _includes/           # Fragmentos reutilizáveis (head, etc.)
├── _posts/              # Publicações em Markdown (YYYY-MM-DD-titulo.md)
├── _sass/
│   ├── _variables.scss  # Tokens de cor e tipografia
│   ├── plain.scss       # Estilos globais
│   ├── dark.scss        # Tema escuro (padrão único)
│   ├── search.scss      # Estilos da busca
│   └── ext/             # Fontes e syntax highlight
├── assets/
│   ├── css/style.scss   # Entry point do CSS
│   ├── js/              # Scripts (busca)
│   └── portfolio.png    # Foto de perfil
└── index.md             # Página inicial
```

## Identidade visual

O site usa exclusivamente tema escuro, com a paleta do portfólio principal:

| Token | Cor | Uso |
|---|---|---|
| Background primário | `#0a0e1a` | Fundo da página |
| Background secundário | `#111827` | Cards, tabelas |
| Background terciário | `#1a2234` | Elementos internos |
| Acento primário | `#3b82f6` | Links, destaques |
| Acento secundário | `#06b6d4` | Elementos complementares |
| Texto primário | `#f1f5f9` | Corpo do texto |
| Texto secundário | `#94a3b8` | Subtítulos, metadados |
| Texto muted | `#64748b` | Datas, rótulos |
| Borda | `#1e293b` | Separadores |

**Fontes:** Manrope (sans-serif) · JetBrains Mono (código)

Todos os tokens ficam em `_sass/_variables.scss`.

## Pré-requisitos

- Ruby >= 3.0
- Bundler

## Instalação

```bash
bundle install
```

## Desenvolvimento

```bash
bundle exec jekyll serve
```

O site fica disponível em `http://localhost:4000`.

## Build para produção

```bash
bundle exec jekyll build
```

O output é gerado em `_site/`.

## Como criar um post

Crie um arquivo em `_posts/` seguindo o padrão de nome:

```
_posts/YYYY-MM-DD-titulo-do-post.md
```

Cabeçalho mínimo do arquivo:

```markdown
---
layout: post
title: "Título do Post"
categories: [categoria]
---

Conteúdo em Markdown...
```

## Configuração principal

As principais opções ficam em `_config.yml`, na chave `plainwhite`:

| Chave | Descrição |
|---|---|
| `name` | Nome exibido no sidebar |
| `tagline` | Subtítulo abaixo do nome |
| `portfolio_image` | Caminho da foto de perfil |
| `search` | Ativa busca nos posts (`true`/`false`) |
| `social_links` | Links para GitHub, LinkedIn, e-mail |
| `condensed_mobile` | Layouts que usam sidebar comprimida no mobile |

## Licença

Tema plainwhite licenciado sob [MIT](LICENSE.txt).
