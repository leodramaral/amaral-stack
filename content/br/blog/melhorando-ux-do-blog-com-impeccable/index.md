---
title: "Como usei o Impeccable pra revisar a UX do meu próprio blog"
date: 2026-09-02T09:00:00-03:00
draft: false
translationKey: "improving-ux-with-impeccable"
tags: ["ux", "design", "ia", "hugo", "acessibilidade"]
description: "Como rodei uma sessão de revisão de design guiada por IA neste blog usando o Impeccable, e o que ela encontrou e corrigiu — de bugs escondidos a acessibilidade."
---

Eu não sou designer. Construí este blog sozinho, com ajuda de LLMs pra estruturar o Hugo, escolher o tema e escrever os templates, mas nunca parei pra olhar o resultado com um olhar crítico de UX. Sabia que tinha coisa errada — algumas até visíveis a olho nu, tipo tags que pareciam clicáveis e não eram — mas não tinha um processo pra encontrar o resto.

Foi aí que resolvi testar o **Impeccable**, uma skill do Claude Code feita pra revisão e refinamento de design de interface. A ideia é simples: em vez de eu pedir "melhora o design" (que é vago demais pra qualquer IA fazer bem), o Impeccable tem comandos específicos — um pra diagnosticar problemas, outros pra corrigir categorias específicas (tipografia, espaçamento, acessibilidade, etc.) — e um jeito de verificar se a correção realmente funcionou antes de considerar terminado.

Rodei uma sessão inteira nele, comando por comando, corrigindo e validando um item de cada vez antes de passar pro próximo. Esse post é o resumo de como foi.

## O diagnóstico: `/impeccable critique`

Comecei sem apontar pra nada específico, e o Impeccable escolheu a homepage como alvo — faz sentido, é a porta de entrada do site. O comando `critique` faz duas coisas ao mesmo tempo: uma "leitura" de design de verdade (como um diretor de design analisando hierarquia, clareza, acessibilidade) e um scanner automático procurando padrões conhecidos de problema no código. As duas rodam separadas uma da outra, sem uma influenciar a outra, e só depois eu vejo as duas juntas.

O resultado veio com uma nota de qualidade (16 de 28 pontos aplicáveis — "aceitável", nem bom nem péssimo) e uma lista de problemas, do mais grave pro mais leve:

- A seção de bio da homepage está completamente vazia (ainda não escrevi esse conteúdo — fica pra depois, quando tiver o material pronto).
- Não existia nenhum caminho da homepage pros posts do blog além de um link escondido no menu.
- As tags dos posts (tipo `#Groq`, `#OpenAI`) pareciam botões clicáveis, mas eram só texto decorativo sem link nenhum.
- Um bug de CSS bobo no cabeçalho, que em teoria deveria ficar fixo no topo ao rolar a página.
- Rótulos de acessibilidade em inglês misturados no meio do site em português, e nenhuma descrição configurada pra quando alguém compartilha o link da home nas redes.

## Corrigindo o que a critique encontrou

Fui um por um, validando cada correção antes de seguir pra próxima:

**Tags viraram links de verdade.** Cada tag agora aponta pra uma página real listando todos os posts daquele assunto — inclusive numa página que eu nem sabia que o Hugo já gerava automaticamente.

**A homepage ganhou uma prévia dos últimos posts.** Enquanto mexia nisso, descobri um bug bem maior escondido: as traduções da interface (as strings de "compartilhar", "posts relacionados" etc.) estavam completamente quebradas em todo o site — um arquivo tinha ido parar na pasta errada meses atrás e ninguém tinha notado porque o site simplesmente mostrava texto vazio no lugar, sem erro nenhum.

**O cabeçalho ficou realmente fixo.** O que parecia ser só uma classe CSS contraditória virou uma investigação mais funda: mesmo removendo a classe errada, o cabeçalho continuava rolando junto com a página. A causa real era estrutural — o elemento pai não deixava espaço suficiente pro efeito de "grudar no topo" funcionar. Só descobri isso testando de verdade num navegador, rolando a página, em vez de confiar só no código.

**Acessibilidade e compartilhamento.** Rótulos traduzidos corretamente pros dois idiomas, e uma descrição padrão pra quando a homepage aparece compartilhada no WhatsApp ou LinkedIn.

## `/impeccable typeset`: a fonte errada pro trabalho errado

Depois pedi especificamente pra revisar a tipografia, porque sentia que a leitura dos posts não estava boa. O motivo: o blog inteiro — inclusive o texto corrido dos artigos — usava uma fonte monoespaçada (tipo as de terminal/código). Isso combina com a identidade visual `{amaral stack}` do site, mas é reconhecidamente pior pra ler parágrafos longos, porque cada letra ocupa a mesma largura e o olho tem mais dificuldade de reconhecer o formato das palavras.

A correção manteve a fonte monoespaçada onde ela faz sentido — logo, menu, datas, código — e trouxe uma fonte própria pra leitura (IBM Plex Sans) só pro corpo dos posts. Também dei peso de verdade aos títulos dentro dos artigos (antes eram do mesmo peso do texto normal, só maiores) e estreitei a largura da coluna de leitura, que estava larga demais pro confortável.

## `/impeccable polish`: o passe de qualidade geral

Esse comando faz uma varredura ampla, testando o site inteiro como um usuário real usaria. Encontrou dois problemas que eu não tinha visto:

- As tags no topo de cada post individual (diferente das da lista) tinham um bug de digitação no link, gerando URLs quebradas em todo post publicado.
- Praticamente nenhum elemento clicável do site tinha um indicador visual de foco pra quem navega pelo teclado — só 2 botões, de mais de 20 elementos, tinham esse cuidado. Isso é importante pra acessibilidade: sem foco visível, alguém navegando só com teclado não consegue saber onde está na página.

## `/impeccable layout`: espaçamento e hierarquia

Por último, uma revisão de estrutura e espaçamento. Achou dois problemas de layout que só apareciam em larguras de tela intermediárias (tipo tablet): as tags de um post com muitas etiquetas quebravam numa segunda linha alinhada errado, deixando uma tag "órfã" sozinha; e títulos longos quebravam no meio de uma palavra composta (tipo "GLM-5.1" virando "GLM-" numa linha e "5.1" na outra). Também ajustou o card de "posts relacionados" pra não sobrar um espaço vazio estranho quando só existe um post relacionado.

## O que ainda falta

A bio da homepage continua vazia de propósito — ainda não escrevi esse texto. E a página de listagem de tags (aquela que os chips agora apontam corretamente) ainda usa o visual padrão e cru do tema, sem o cuidado visual do resto do site. Ambos ficam pra uma próxima rodada.

## Valeu a pena?

O que mais me chamou atenção foi quantos dos problemas reais não eram "feios" — eram invisíveis até alguém testar de verdade: o cabeçalho que não grudava, as traduções quebradas, o link de tag com um espaço a mais no meio da URL. Nenhum desses aparece só olhando o código com calma; só depois de rodar o site num navegador e realmente tentar usá-lo. Ter um processo estruturado pra isso — diagnosticar, corrigir por categoria, validar de verdade antes de seguir — foi o que fez a diferença pra alguém como eu, que não tem instinto de design treinado mas consegue reconhecer um problema quando é mostrado.
