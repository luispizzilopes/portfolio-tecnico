## Context

O blog usa o tema plainwhite-jekyll configurado via `_config.yml`. Toda a identidade do autor (nome, tagline, links sociais, idioma, título) é controlada por esse arquivo de configuração YAML — sem necessidade de alterar layouts ou SCSS. O post placeholder `_posts/2019-03-23-welcome-to-jekyll.markdown` é genérico e deve ser substituído. Ver `proposal.md` para motivação.

## Goals / Non-Goals

**Goals:**

- Substituir todos os dados de identidade do autor original no `_config.yml`
- Garantir que `html_lang`, `title`, `description`, `author`, `email` e `social_links` estejam corretos
- Substituir o post placeholder por um post inicial autoral
- Manter a funcionalidade do tema intacta (dark mode toggle, search, sitemap)

**Non-Goals:**

- Personalização visual (cores, fontes, layout) — escopo futuro
- Foto de perfil definitiva — placeholder mantido por ora
- Criação de páginas adicionais (about, portfolio) — escopo futuro
- Configuração de domínio ou hospedagem

## Decisions

### Tudo via `_config.yml`

O plainwhite-jekyll expõe todas as configs de identidade em `plainwhite:` e nas chaves raiz do `_config.yml`. Não há necessidade de tocar em layouts Liquid ou SCSS — a mudança é puramente de configuração.

**Alternativa considerada**: editar `_layouts/default.html` diretamente. Rejeitado porque quebraria a capacidade de atualizar o tema e viola a separação configuração/template.

### LinkedIn no formato `in/<slug>`

A config `social_links.linkedIn` do tema espera o path relativo (ex: `in/usuario`), não a URL completa. O valor correto é `in/luis-felipe-pizzi-lopes`.

### Post inicial em português

Criar um novo post em pt-BR no lugar do placeholder, datado adequadamente, que serve como inauguração do blog.

**Alternativa considerada**: deletar o post sem criar outro. Rejeitado — um blog sem nenhum post na home parece vazio/quebrado ao carregar.

## Risks / Trade-offs

- **[Risco]** Foto de perfil placeholder (`assets/portfolio.png`) continua sendo a do tema original → Aceitável por ora; será substituída em mudança futura quando o autor tiver uma imagem pronta
- **[Risco]** Email público no blog pode receber spam → Mitigação: é decisão consciente do autor; o tema já prevê essa opção

## Migration Plan

1. Editar `_config.yml` com os novos valores
2. Substituir/criar post inicial em `_posts/`
3. Verificar `index.md` quanto a referências ao template
4. Testar localmente com `bundle exec jekyll serve` para confirmar renderização correta
5. Sem rollback especial necessário — mudanças são reversíveis via git
