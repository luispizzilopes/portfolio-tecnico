## 1. Atualizar configuração de identidade

- [x] 1.1 Em `_config.yml`, atualizar `title` para "Luis Felipe - Dev Blog" e verificar que aparece na aba do navegador ao servir o site localmente
- [x] 1.2 Em `_config.yml`, atualizar `author` para "Luis Felipe Pizzi Lopes", `email` para `luisfelipe1203lf@gmail.com` e `description` para "Publicações sobre código, arquitetura, ferramentas e aprendizados do dia a dia como desenvolvedor." — verificar que os metadados SEO refletem os novos valores no HTML gerado
- [x] 1.3 Em `_config.yml`, atualizar `plainwhite.name` para "Luis Felipe Pizzi Lopes" e `plainwhite.tagline` para "Full Stack Developer" — verificar que a sidebar exibe os valores corretos
- [x] 1.4 Em `_config.yml`, atualizar `plainwhite.html_lang` para `pt-BR` — verificar que `<html lang="pt-BR">` está presente no HTML gerado
- [x] 1.5 Em `_config.yml`, configurar `plainwhite.social_links` com `github: luispizzilopes`, `linkedIn: in/luis-felipe-pizzi-lopes` e `email: luisfelipe1203lf@gmail.com` (remover twitter e demais redes não utilizadas) — verificar que os três links aparecem no rodapé

## 2. Ajustar post inicial

- [x] 2.1 Remover o arquivo `_posts/2019-03-23-welcome-to-jekyll.markdown` (post placeholder do Jekyll)
- [x] 2.2 Criar novo arquivo `_posts/<data-atual>-ola-mundo.md` com um post inaugural em pt-BR apresentando o blog — verificar que o post aparece na home e é renderizado corretamente

## 3. Verificar arquivos de suporte

- [x] 3.1 Inspecionar `index.md` e remover ou atualizar quaisquer referências ao template original — verificar que a home renderiza sem conteúdo residual do template

## 4. Validação final

- [x] 4.1 Executar `bundle exec jekyll serve` e verificar visualmente que: nome, tagline, links sociais, idioma e título estão corretos; dark mode toggle funciona; post inaugural aparece na home
