## Purpose

Define a paleta de cores e tipografia que o site técnico (Jekyll) deve usar, garantindo alinhamento visual com o portfólio principal do desenvolvedor.

## ADDED Requirements

### Requirement: Paleta de cores alinhada ao portfólio principal
O site SHALL usar os seguintes tokens de cor como base de toda a estilização:
- Background primário: `#0a0e1a`
- Background secundário: `#111827`
- Background terciário: `#1a2234`
- Acento primário: `#3b82f6`
- Acento secundário: `#06b6d4`
- Acento glow: `#60a5fa`
- Texto primário: `#f1f5f9`
- Texto secundário: `#94a3b8`
- Texto muted: `#64748b`
- Cor de borda: `#1e293b`
- Card hover: `#1e2938`
- Success: `#10b981`
- Warning: `#f59e0b`
- Error: `#ef4444`

#### Scenario: Fundo da página usa cor primária
- **WHEN** o usuário acessa qualquer página do site
- **THEN** o fundo SHALL ser renderizado na cor `#0a0e1a`

#### Scenario: Texto principal é legível sobre fundo escuro
- **WHEN** o usuário lê o conteúdo principal da página
- **THEN** o texto SHALL ser exibido na cor `#f1f5f9` sobre fundo `#0a0e1a`

#### Scenario: Links e destaques usam acento primário
- **WHEN** o usuário visualiza um link ou elemento de destaque
- **THEN** o elemento SHALL ser exibido na cor `#3b82f6`

### Requirement: Tipografia alinhada ao portfólio principal
O site SHALL usar **Manrope** como fonte sans-serif principal e **JetBrains Mono** como fonte monospace, carregadas via Google Fonts com os pesos 300, 400, 500, 600, 700 e 800 para Manrope e 300, 400, 600, 700 para JetBrains Mono.

#### Scenario: Fonte principal aplicada ao corpo do texto
- **WHEN** o usuário visualiza qualquer conteúdo textual
- **THEN** o texto SHALL ser renderizado em Manrope como fonte primária, com fallback para `-apple-system, BlinkMacSystemFont, sans-serif`

#### Scenario: Fonte monospace aplicada a blocos de código
- **WHEN** o usuário visualiza um bloco de código ou trecho inline de código
- **THEN** o texto SHALL ser renderizado em JetBrains Mono, com fallback para `monospace`

#### Scenario: Fontes carregadas via Google Fonts
- **WHEN** o site é carregado no navegador
- **THEN** as fontes Manrope e JetBrains Mono SHALL ser importadas do Google Fonts sem bloqueio de renderização (preconnect ou display=swap)

### Requirement: Modo escuro como padrão e único tema
O site SHALL exibir exclusivamente o tema escuro usando a paleta definida acima, sem oferecer alternância para tema claro.

#### Scenario: Ausência de toggle de tema claro
- **WHEN** o usuário acessa o site
- **THEN** nenhum controle de alternância de tema SHALL estar presente ou funcional para mudar para modo claro
