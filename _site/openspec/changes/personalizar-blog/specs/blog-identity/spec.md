## Purpose

Define a identidade pública do blog de Luis Felipe Pizzi Lopes: nome do autor, tagline, descrição, idioma, título do site e links de redes sociais exibidos ao visitante.

## ADDED Requirements

### Requirement: Identidade do autor exibida corretamente

O blog SHALL exibir o nome "Luis Felipe Pizzi Lopes" e a tagline "Full Stack Developer" na sidebar/cabeçalho em todas as páginas.

#### Scenario: Visitante acessa qualquer página

- **WHEN** um visitante carrega qualquer página do blog
- **THEN** o nome "Luis Felipe Pizzi Lopes" e "Full Stack Developer" são exibidos na área de identidade do autor

### Requirement: Título e descrição do site configurados

O site SHALL ter o título "Luis Felipe - Dev Blog" e a descrição "Publicações sobre código, arquitetura, ferramentas e aprendizados do dia a dia como desenvolvedor." nos metadados HTML e SEO.

#### Scenario: Metadados de SEO corretos

- **WHEN** um motor de busca ou ferramenta de preview acessa o blog
- **THEN** os metadados `<title>` e `<meta name="description">` refletem o título e descrição configurados

### Requirement: Idioma do site configurado para pt-BR

O atributo `lang` da tag `<html>` SHALL ser `pt-BR` em todas as páginas.

#### Scenario: Atributo de idioma presente

- **WHEN** qualquer página é carregada
- **THEN** `<html lang="pt-BR">` está presente no HTML renderizado

### Requirement: Links sociais corretos no rodapé

O blog SHALL exibir links para GitHub (`luispizzilopes`), LinkedIn (`in/luis-felipe-pizzi-lopes`) e email (`luisfelipe1203lf@gmail.com`) no rodapé.

#### Scenario: Links sociais acessíveis

- **WHEN** um visitante visualiza o rodapé do blog
- **THEN** estão presentes links funcionais para o GitHub, LinkedIn e email do autor

### Requirement: Dark mode com padrão claro

O blog SHALL iniciar em tema claro por padrão, com toggle disponível para que o usuário alterne para dark mode manualmente.

#### Scenario: Carregamento inicial em tema claro

- **WHEN** um visitante acessa o blog pela primeira vez sem preferência salva
- **THEN** o tema claro é aplicado e o toggle de dark mode está visível e funcional
