## Why

O site técnico atual (Jekyll + plainwhite) possui layout e disposição satisfatórios, mas usa cores e fontes genéricas que não correspondem à identidade visual já estabelecida no portfólio principal do usuário (`portfolio-tailwindcss`). Unificar a estilização cria consistência visual entre os dois sites e reforça a identidade pessoal do desenvolvedor.

## What Changes

- Substituir as fontes atuais pelas fontes **Manrope** (sans-serif principal) e **JetBrains Mono** (monospace) via Google Fonts
- Substituir a paleta de cores atual pela paleta do portfólio principal:
  - Backgrounds: `#0a0e1a` (primário), `#111827` (secundário), `#1a2234` (terciário)
  - Acentos: `#3b82f6` (primário), `#06b6d4` (secundário), `#60a5fa` (glow)
  - Textos: `#f1f5f9` (primário), `#94a3b8` (secundário), `#64748b` (muted)
  - Borda: `#1e293b`; Card hover: `#1e2938`
  - Status: success `#10b981`, warning `#f59e0b`, error `#ef4444`
- Atualizar os arquivos SASS (`_sass/plain.scss`, `_sass/dark.scss` e afins) para usar os novos tokens
- Importar as fontes no cabeçalho ou via SASS

## Capabilities

### New Capabilities

- `visual-identity`: Define a paleta de cores e tipografia que o site deve seguir, alinhada ao portfólio principal do usuário

### Modified Capabilities

<!-- Nenhuma capability existente com spec anterior -->

## Impact

- Arquivos SASS: `_sass/plain.scss`, `_sass/dark.scss`, `_sass/ext/_fonts.scss`, `assets/css/style.scss`
- Layout HTML (`_layouts/`, `_includes/`) pode precisar de ajustes pontuais de classe caso use valores de cor hard-coded
- `_config.yml` não é afetado
- Sem mudanças de dependências de gems; fontes carregadas via CDN Google Fonts
