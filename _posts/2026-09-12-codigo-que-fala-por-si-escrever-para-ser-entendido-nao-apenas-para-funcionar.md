---
layout: post
title: "Código que fala por si: escrever para ser entendido, não apenas para funcionar"
date: 2026-09-12
categories: [Arquitetura, Clean Architecture, Engenharia de Software]
permalink: /2026/09/12/codigo-que-fala-por-si-escrever-para-ser-entendido-nao-apenas-para-funcionar/
---

Existem projetos que passam pela nossa carreira e outros que ficam. Este é sobre um dos que ficou, não pela tecnologia em si, mas pelo tanto que me ensinou sobre entender um problema antes de tentar resolvê-lo, e sobre escrever código pensando em quem vai lê-lo depois de nós.

## O desafio: engenharia reversa de uma cadeia fiscal inteira

Fui responsável por projetar, de ponta a ponta, um sistema cujo objetivo era gerar um relatório fiscal cobrindo toda a cadeia de fabricação de um produto. Na teoria, a frase parece simples. Na prática, era um quebra-cabeça de engenharia reversa.

O ponto de partida era o produto acabado. A partir dele, o sistema precisava chegar até a nota fiscal de saída daquele produto, analisar linha a linha essa nota, e então caminhar no sentido inverso: identificar as notas fiscais de entrada que haviam originado a matéria-prima usada na fabricação. Isso envolvia cruzar lote, quantidade, unidade de medida e uma série de outras informações que não estavam todas concentradas em um único lugar; muitas vinham de fontes externas ao sistema, com formatos e granularidades diferentes.

Não existia atalho possível. Cada regra de negócio dependia de um vínculo forte entre campos que, isoladamente, pareciam desconexos, mas que juntos reconstruíam a história fiscal completa de um produto: da matéria-prima que entrou pela porta dos fundos até a nota que saiu pela porta da frente. Um único produto podia carregar consigo dezenas de notas de entrada diferentes, cada uma com suas particularidades, e o sistema precisava ser capaz de reconstruir esse caminho de forma confiável, auditável e replicável.

Esse tipo de problema tem uma característica interessante: ele não perdoa meio-entendimento. Ou você entende profundamente como o processo fiscal e produtivo da empresa funciona, ou o sistema vai quebrar silenciosamente em algum caso de borda que ninguém previu, e em um contexto fiscal, "quebrar silenciosamente" não é uma opção.

## O mapeamento vai muito além do que o negócio consegue te contar

Antes de qualquer linha de código, veio uma fase de levantamento longa: reuniões extensas, riqueza de detalhes sobre o processo produtivo e fiscal, muita conversa com quem vivia aquele processo no dia a dia. E aqui está o primeiro ponto que quero destacar, porque foi uma das maiores lições desse projeto: o mapeamento de um sistema como esse não se esgota no que a área de negócio consegue formalizar em uma reunião.

O pessoal de negócio conhece o processo na prática, mas nem sempre tem clareza de como cada etapa se conecta tecnicamente, quais exceções existem no meio do caminho, ou como um dado que "sempre foi preenchido daquele jeito" pode quebrar uma regra em um caso específico que só aparece uma vez a cada mil. Cabe ao time técnico ir além da entrevista inicial: entender a cadeia de informações de forma robusta, questionar os porquês, simular cenários hipotéticos, levantar exceções antes que elas se tornem bugs em produção, e validar hipóteses com quem vive o processo, não apenas documentá-lo passivamente.

Esse tipo de mapeamento é quase um trabalho de investigação. Você não está apenas anotando o que o stakeholder diz; está reconstruindo, junto com ele, um modelo mental do processo que muitas vezes nem ele tinha formalizado por completo. Perguntas como "e se essa nota vier sem esse campo preenchido?", "e se o lote for dividido entre duas notas de entrada diferentes?", "quem garante que essa informação externa está sempre atualizada?" não nascem de uma lista de requisitos. Nascem de sentar, entender o processo de ponta a ponta, e desconfiar de qualquer regra que pareça simples demais.

Foi exatamente esse entendimento aprofundado, construído em conjunto com os stakeholders, que permitiu antecipar boa parte da complexidade que apareceria mais adiante no desenvolvimento. Quando você entende o processo de negócio na essência, e não só na superfície, a arquitetura que nasce a partir disso tende a refletir a realidade do domínio, e não uma abstração ingênua e simplificada dela.

## Arquitetura limpa como resposta à complexidade, não como modismo

Enquanto o levantamento de negócio avançava, a estrutura do código já começava a nascer em paralelo, e isso não foi coincidência. Optamos por uma arquitetura de software limpa, com foco explícito em desacoplamento e organizada em torno dos casos de uso que o projeto exigiria. Essa escolha não foi acadêmica, nem "porque é assim que se faz hoje em dia". Foi uma necessidade prática diante da complexidade das regras que estávamos lidando.

Um sistema com esse volume de regras de negócio, dependências entre campos e fontes de dados externas tende a virar um emaranhado difícil de manter se a estrutura não for pensada com cuidado desde o início. Por isso, cada caso de uso do sistema foi tratado como uma unidade isolada de responsabilidade: buscar as notas de saída, extrair as linhas relevantes, rastrear a origem da matéria-prima, validar o vínculo de lote, montar o relatório final, cada uma dessas etapas vivendo em seu próprio espaço, sem se misturar com as outras.

Durante o desenvolvimento, era comum surgirem casos de uso muito específicos, cuja lógica fazia sentido em apenas um ponto isolado do sistema: uma regra que só se aplicava a um tipo de produto, uma validação que só existia para um formato específico de nota externa. Para esses cenários, recorremos a design patterns focados em comportamento e estrutura, sempre com o objetivo de facilitar o desenvolvimento e manter a legibilidade, nunca para "seguir a cartilha" ou aplicar padrão por aplicar. A ideia sempre foi: qual estrutura deixa essa regra específica isolada, testável e fácil de entender sem que a pessoa precise ler o sistema inteiro para captar o contexto?

O resultado prático dessa abordagem:

* Os casos de uso ficaram bem divididos e isolados, cada um com uma responsabilidade clara e um nome que já dizia o que fazia.
* O acesso a dados ficou desacoplado da lógica de negócio, o que facilitava tanto a manutenção quanto os testes; trocar uma fonte de dados externa, por exemplo, não exigia tocar na regra de negócio em si.
* Mesmo em um sistema robusto e com muita regra embutida, a leitura do código permanecia clara, quase didática.
* Regras muito específicas, que em outro contexto poderiam virar "gambiarras" espalhadas pelo código, ganhavam um lugar próprio e explícito.

E isso importa porque, num sistema fiscal como esse, a lógica de negócio é o produto. Não existe margem para um código que funciona, mas que ninguém além de quem o escreveu consegue entender. Um erro de interpretação em uma regra fiscal não é só um bug: pode significar uma informação incorreta em uma obrigação legal da empresa.

## Escopo que cresce, prazo que aperta, e a decisão de não abrir mão da qualidade

Como em praticamente todo projeto real, coisas além do escopo inicial foram aparecendo ao longo do caminho: novas regras que ninguém tinha mencionado nas reuniões iniciais, dúvidas técnicas sobre como tratar certos formatos de dado externo, dúvidas de negócio que só se revelavam quando um caso real batia de frente com a regra que havíamos desenhado no papel. Isso é absolutamente normal e esperado, faz parte de qualquer levantamento, por mais detalhado e cuidadoso que tenha sido.

Nenhum levantamento, por melhor que seja, antecipa cem por cento da realidade. E tudo bem. O que fez diferença, na minha visão, foi a postura da equipe diante disso. Mesmo com prazos apertados, e eles estavam sempre apertados, o compromisso era manter a qualidade do código que estávamos entregando. Não abrir mão do desacoplamento, não abrir mão da clareza, não abrir mão de casos de uso bem definidos só porque "dava para resolver mais rápido de outro jeito, remendando ali".

Esse tipo de decisão parece custar tempo no curto prazo. Escrever um caso de uso isolado para uma regra específica, ao invés de simplesmente enfiar um `if` a mais em algum lugar que já existe, leva mais tempo naquele momento. Mas é exatamente esse tipo de escolha que salva o projeto no médio e longo prazo, e, como vou contar a seguir, salvou literalmente a continuidade desse projeto depois que eu saí dele.

## O teste real: o código explicando a si mesmo

Não consegui finalizar esse projeto: surgiu uma nova oportunidade de carreira no meio do caminho, como acontece na vida de qualquer desenvolvedor. E é justamente aí que veio a validação mais honesta de todo esse esforço de mapeamento e arquitetura.

Na hora de repassar o projeto para um colega, o processo foi surpreendentemente simples. Não porque eu tive tempo de fazer uma transição perfeita, com documentação extensa, diagramas atualizados e reuniões de handover intermináveis, mas porque o próprio código já dizia o que precisava ser dito. Os casos de uso bem nomeados, a separação clara de responsabilidades, o desacoplamento entre lógica de negócio e acesso a dados, tudo isso fez o trabalho que, em outras circunstâncias, teria ficado inteiramente sobre os ombros de uma documentação que talvez nem existisse mais, ou que estivesse desatualizada desde a segunda semana do projeto.

Esse momento foi, para mim, a prova concreta de que o tempo investido no início, nas longas reuniões de mapeamento, nas conversas repetidas com os stakeholders, na decisão de desenhar uma arquitetura desacoplada mesmo sob pressão de prazo, não tinha sido tempo perdido. Tinha sido, na verdade, o investimento que permitiu que o projeto continuasse de pé sem depender da minha presença.

## Reflexão final

Se eu pudesse resumir esse case em uma ideia, seria esta: o mapeamento profundo do negócio e a clareza do código não são etapas separadas do projeto, são a mesma disciplina aplicada em momentos diferentes. Entender a cadeia de informações além do que o negócio consegue formalizar em uma reunião é o que permite desenhar uma arquitetura que faz sentido. E uma arquitetura que faz sentido é o que permite escrever um código que continua fazendo sentido mesmo quando quem o escreveu não está mais lá para explicá-lo.

Prazos apertam, escopos crescem, pessoas saem do projeto: isso é inevitável, faz parte da realidade de qualquer time de desenvolvimento. O que não precisa ser inevitável é deixar que essas pressões corroam a qualidade daquilo que construímos. Um código bem pensado, nascido de um entendimento robusto do processo e escrito com cuidado com quem vai lê-lo depois, é uma forma de respeito com quem vem a seguir.

Nos vemos no próximo post.
