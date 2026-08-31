---
name: agente-de-ia
description: Monta, revisa e corrige o Agente de IA do LionChat de ponta a ponta pelo conector - modelo, instrucoes, protecoes, cenarios, base de conhecimento, biblioteca de midia, ferramentas, transferencia para humano, acompanhamento automatico, teste no Playground e revisao de qualidade. Use quando o usuario disser "monta meu agente de IA", "cria uma IA que atende meu WhatsApp", "a IA esta inventando informacao", "a IA nao responde", "quero que a IA agende", "quero que a IA envie o catalogo", "quero comparar duas IAs", ou pedir cenario, base de conhecimento, follow-up, resposta em audio ou ferramenta da IA. Sempre pergunta antes e so cria depois de confirmacao explicita.
---

# Agente de IA do LionChat

O Agente de IA e o atendente virtual que responde os clientes sozinho dentro das conversas, em qualquer
canal (WhatsApp, Instagram, chat do site, e-mail). Ele le a conversa inteira, consulta a base de
conhecimento que voce cadastrou e, alem de responder, ele AGE: coloca etiqueta, cria card no funil, grava
dado na ficha do contato, marca compromisso, envia arquivo e transfere para uma pessoa quando trava. Voce
monta o comportamento dele escrevendo em portugues.

Montar um agente bom nao e escrever um texto grande. E montar quatro coisas que se apoiam: **instrucao**
(quem ele e), **cenarios** (o passo a passo de cada situacao), **base de conhecimento** (a verdade sobre o
negocio) e **ferramentas** (o que ele consegue de fato executar). Quando qualquer uma dessas quatro falta,
a IA preenche o buraco inventando.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 - Entender

Antes de criar qualquer coisa, confira os pre-requisitos e faca as perguntas. **1 ou 2 por vez** - disparar
a lista inteira cansa e o cliente responde mal.

Confira primeiro, com `lionchat_account_show` e `lionchat_captain_assistants_list`:

- A conta tem o recurso de IA liberado e ainda ha vaga no plano? Sem vaga, a criacao volta erro de limite.
- Ja existe algum agente? Nunca crie um segundo achando que nao existe nenhum.
- A conta tem chave da OpenAI cadastrada (Integracoes, OpenAI)? **Sem a chave, nada funciona**: documento
  nao e indexado e a IA fica muda. Isso NAO se cadastra pelo conector - se faltar, PARE e peca ao cliente
  antes de montar.

Perguntas ao cliente:

1. Qual o papel deste agente numa frase? (vender, tirar duvida, agendar, triar e passar adiante)
2. Qual o negocio e quem e o cliente que vai falar com ele?
3. Onde ele deve atender: em toda conversa nova de uma caixa, so depois de uma triagem, ou so quando um
   atendente ligar na mao?
4. O que ele NUNCA pode fazer ou falar?
5. Quando ele nao souber, o que acontece: transfere para quem, ou pergunta para a equipe por nota interna?
6. Voce tem material escrito para virar base de conhecimento (precos, enderecos, horarios, politica de
   troca, perguntas frequentes)? Sem isso ele vai depender so das instrucoes e vai errar mais.
7. Ele precisa AGIR ou so responder? Etiquetar, criar card, gravar dado na ficha, agendar, mandar arquivo?
8. Se precisa agendar: ja existe agenda configurada? Sem agenda, ele nao agenda.
9. Se precisa consultar um sistema seu (estoque, pedido): existe uma API pronta?
10. Ele deve cobrar quem sumiu? Em quanto tempo, quantas vezes, e quem NAO deve ser cobrado?
11. A caixa e WhatsApp Oficial (Meta) ou por QR Code? No Oficial ha limites de janela que mudam a resposta.
12. Quando um atendente humano entrar na conversa, a IA some de vez ou fica calada por um tempo?
13. Ele responde em audio?
14. Vamos criar UM agente ou varios? Varios = comparacao. Um por PAPEL, nunca um por canal.

Depois, levante o terreno (e o que decide quais ferramentas vao existir): `lionchat_labels_list`,
`lionchat_teams_list`, `lionchat_funnels_list`, `lionchat_booking_event_types_list`,
`lionchat_custom_attributes_list`, `lionchat_offers_list`, `lionchat_captain_media_assets_list`.

### Etapa 2 - Decidir

- **Papel unico.** Um agente por papel. Vendas e suporte no mesmo texto viram um agente confuso.
- **Modelo e criatividade** pelo tipo de atendimento: roteiro fixo, agendamento ou cobranca de dado pedem
  criatividade baixa; venda conversacional pede mais solta. Tabela e regras em
  `references/modelo-e-ajustes.md`.
- **O que vira cenario e o que fica na instrucao.** Instrucao = quem ele e e como fala, sempre. Cenario =
  situacao com passo a passo. Se a instrucao passa de umas 2.500 letras sem nenhum cenario, quebre em
  cenarios - e nao por causa de custo, e porque a IA perde o "quando" de cada regra.
- **So prometa o que a ferramenta existe para executar.** Se nao ha funil, a IA nao cria card. Se nao ha
  agenda, ela nao agenda. Se nao ha arquivo liberado, ela nao envia arquivo. Ver
  `references/catalogo-ferramentas.md`.
- **Transferencia para humano tem que ter destino.** Decida: atendente, time ou nota interna pedindo ajuda.

### Etapa 3 - Propor

Mostre a proposta inteira em texto estruturado, com o porque de cada decisao em uma frase:

```
AGENTE "[nome]"
  Papel: [uma frase - vai inteira para o comportamento dele]
  Modelo: [modelo] - criatividade [valor] porque [motivo]

INSTRUCAO (resumo do que vai escrito)
PROTECOES (X) - o que ele nunca faz
DIRETRIZES DE RESPOSTA (X) - como ele escreve

BASE DE CONHECIMENTO (X paginas)
  1. [assunto] - [o que tem dentro]
PERGUNTAS FREQUENTES (X)

CENARIOS (X)
  1. "[titulo]" - quando: [gatilho] / faz: [passo a passo resumido]

FERRAMENTAS QUE ELE VAI TER
  Ja existem: [...]
  Precisam ser criadas antes: [funil / agenda / time com descricao / etiqueta com descricao / arquivo]

ACOMPANHAMENTO AUTOMATICO: [X etapas, tempos] ou "nao"
RESPOSTA EM AUDIO: [modo] ou "nao"
QUANDO O HUMANO ASSUME: [a IA para de vez / cala por N minutos]

COMO ELE COMECA A ATENDER: [automacao em toda conversa nova / bloco de fluxo / na mao pelo atendente]
```

Termine com: **"Confirma que posso criar tudo isso? (sim, ou me diga o que mudar)"**. Nao avance sem um
sim explicito ("sim", "pode", "confirmado", "beleza", "manda ver"). Se pedir ajuste, refaca e pergunte
de novo.

### Etapa 4 - Executar

Nesta ordem. A ordem importa: cenario que cita ferramenta inexistente e recusado, e ferramenta so existe
se o recurso da conta existir antes.

1. **Criar o agente** - `lionchat_captain_assistants_create` com `name` e `description`. A descricao vai
   INTEIRA para o comportamento dele; escreva o papel, nunca um rotulo de bastidor.
2. **Modelo e criatividade** - `lionchat_captain_assistants_update`. Ler `references/modelo-e-ajustes.md`
   antes: ha modelos em que o ajuste de criatividade nao faz nada e uma familia que deixa a IA 100% muda.
3. **Instrucao, protecoes e diretrizes** - `config.instructions`, `guardrails`, `response_guidelines`.
   Como escrever: `references/escrever-cenario.md`.
4. **Base de conhecimento e perguntas frequentes** - `lionchat_captain_documents_create` (uma pagina por
   assunto) e `lionchat_captain_assistants_create_2`. Espere o documento ficar disponivel antes de
   apontar para ele. Detalhe em `references/base-de-conhecimento-e-midia.md`.
5. **Criar o terreno que falta** - etiqueta e time COM DESCRICAO, funil, agenda, campo personalizado,
   oferta. Antes dos cenarios, sempre.
6. **Biblioteca de midia** - cadastrar o arquivo e depois LIBERAR os ids no agente
   (`config.media_asset_ids`). Cadastrar sem liberar = a IA nao enxerga o arquivo.
7. **Ferramentas proprias** - de API (`lionchat_captain_assistants_create_5`, teste com
   `lionchat_captain_custom_tools_test`) e de fluxo (`lionchat_flow_tools_create`, teste com
   `lionchat_flow_tools_run` ANTES de vincular, vincule com `lionchat_flow_tools_assistants_update`
   mandando a lista COMPLETA de agentes).
8. **Conferir o que ele tem** - `lionchat_captain_assistants_tools` com o `assistant_id`. Desligue o que
   ele nao deve poder fazer em `config.disabled_tools`.
9. **Cenarios** - `lionchat_captain_assistants_create_1` (esse e o nome mesmo). Um por situacao.
10. **Conferir as variaveis** - `lionchat_captain_liquid_variables_list`. So use `{{ }}` que aparece la.
11. **Acompanhamento automatico e audio**, se pedido.
12. **Testar no Playground** - `lionchat_captain_assistants_playground`, com pelo menos 5 casos reais.
13. **Revisao de qualidade** - `lionchat_captain_assistants_quality`. So leitura, nao gasta IA.
14. **Ligar** - so depois de confirmar de novo com o cliente, porque a partir daqui a IA fala com cliente
    de verdade. Ver `references/ativar-e-diagnosticar.md`.

Em cada erro: 403 e limite do plano (acabou a vaga de agente) e 401 e falta de permissao (quem esta
executando nao e administrador nem tem o papel de gerenciar IA) - nos dois casos, pare e avise. Se voltar
422 dizendo que o texto cita ferramenta invalida, a mencao no cenario aponta para algo que este agente nao
tem: conserte o texto, nao insista. Se um nome de ferramenta do conector nao existir, NAO invente outro - descreva a acao
e pergunte.

### Etapa 5 - Conferir e resumir

Antes de dizer que acabou: reabra o agente (`lionchat_captain_assistants_show`) e confira que o que voce
mandou esta gravado. Campo que nao e aceito some SEM erro - a resposta diz sucesso e o valor nunca chegou.

Resumo final: o que foi criado (com os nomes), o que falhou e por que, o que so da para fazer na mao no
painel (cadastrar a chave da OpenAI, fixar agenda por cenario, tocar amostra de voz), como desligar em
emergencia (`paused: true` no agente para tudo sem perder configuracao) e onde acompanhar
(`lionchat_captain_supervisor_show`).

## Regras que nao podem ser violadas

1. **NUNCA crie ou altere nada sem confirmacao explicita do cliente** - ainda mais o passo de ligar a IA,
   que faz ela falar com cliente de verdade.
2. **NUNCA apague nada.** Para desligar, use `paused: true` no agente ou `enabled: false` no cenario, no
   arquivo e na ferramenta. Exclusao e decisao do cliente, no painel.
3. **NUNCA invente nome de ferramenta do conector.** Os nomes desta area sao enganosos de proposito
   (cenario, pergunta frequente e ferramenta propria vivem todos sob nomes de "assistants"). Mapa completo
   em `references/nomes-mcp.md`. Se nao achar o nome, descreva a acao em palavras.
4. **NUNCA prometa no texto do agente um caminho que a ferramenta nao executa.** Sem funil nao ha card, sem
   agenda nao ha agendamento, sem arquivo liberado nao ha envio de arquivo. A IA obedece o texto e inventa
   a execucao.
5. **NUNCA use a familia de modelos gpt-5.6.** Ela recusa ferramentas e a IA fica 100% muda, em silencio.
6. **NUNCA prometa criatividade em modelo de raciocinio** (o1, o3, o3-mini, o4-mini, gpt-5, gpt-5-mini,
   gpt-5.5). O valor ate grava, mas o sistema nao manda o ajuste para esses modelos: ele nao faz nada.
   Se o cliente quer controlar o tom pela criatividade, escolha outro modelo.
7. **NUNCA deixe `{{alguma.coisa}}` no texto sem rodar a revisao de qualidade depois.** Variavel que o
   sistema nao preenche nao da erro: some do texto e a IA entrega a frase com o buraco para o cliente.
8. **NUNCA deixe as protecoes vazias.** Sem elas nao existe limite escrito para o que ele nao pode falar.
9. **NUNCA mande a lista de agentes de uma ferramenta de fluxo com so o id novo** - aquele endereco
   substitui a lista inteira e desvincula todos os outros, em silencio.
10. **SEMPRE reabra e confira depois de gravar.** Configuracao nao esta pronta enquanto ninguem releu.
11. **SEMPRE avise o cliente sobre efeito colateral de ligar a IA numa caixa**: com agente ativo na
    conversa, a saudacao, o aviso de ausencia e a coleta de e-mail daquela caixa param de ser enviados -
    quem da as boas-vindas passa a ser a propria IA.
12. **SEMPRE em portugues do Brasil**, sem emoji, e usando os nomes da tela: Agente de IA, Cenarios,
    Documentos, FAQs, Ferramentas, Biblioteca de Midia, Playground, Qualidade da construcao, Supervisor.

## Armadilhas

Todas ja aconteceram com cliente de verdade. As piores sao as que falham **em silencio**: a coisa e criada,
parece certa na tela e nao funciona. Detalhe e mais casos em `references/armadilhas.md`.

- **Se voce montar o agente e nao ligar em lugar nenhum, ele nunca atende ninguem.** Nao existe mais
  vincular agente a uma caixa de entrada. A IA so responde numa conversa que tem o agente atribuido - por
  automacao, por bloco de fluxo, pela lista suspensa da conversa ou por acao em massa. Documentacao antiga
  ainda ensina o passo de "vincular a caixa"; ele nao existe mais.
- **Se voce cadastrar um arquivo na Biblioteca de Midia e nao liberar o id no agente**, o arquivo aparece
  na tela, a IA nao enxerga e nao envia. Vale igual para produtos e agendas.
- **Se voce escrever no cenario "confirme o agendamento" contando que ela chame a ferramenta**, ela pode
  consultar os horarios, validar tudo e escrever "sua reuniao esta confirmada" sem marcar nada. Ja
  aconteceu: cliente em sala vazia. Escreva o passo a passo citando a ferramenta pelo nome.
- **Se voce achar que "nao invente" na instrucao trava a IA**, nao trava. Ja houve um caso com a regra
  escrita NOVE vezes no texto e a IA passando endereco inexistente. O que segura e estrutura: base
  organizada, cenario com passo a passo, e ferramenta de verdade para executar.
- **Se voce mandar um campo que o servidor nao aceita**, ele some sem erro: a resposta diz sucesso e o
  valor nunca foi gravado. Reabra sempre.
- **Se voce somar mais de 23 horas no acompanhamento automatico em caixa WhatsApp Oficial**, a cobranca cai
  fora da janela de envio e some sem aviso. O sistema hoje recusa acima disso ao salvar, mas configuracao
  antiga acima do teto continua ativa e falhando.
- **Se voce deixar uma linha com tres tracos sozinha na instrucao ou no cenario**, esse e o sinal de quebrar
  a resposta em mensagens: o cliente recebe a mensagem picada.
- **Se a IA "nao responde", nem sempre o problema e o texto.** Em caixa WhatsApp Oficial fora da janela de
  24 horas ela nem gera resposta, de proposito. Comentario de post do Instagram tambem nunca aciona a IA.
  Lista completa de motivos em `references/ativar-e-diagnosticar.md` - confira ANTES de mexer no prompt.
- **Se voce criar time ou etiqueta sem descricao**, a IA nao sabe quando usar: e a descricao, nao o nome,
  que ela le para decidir rotear e etiquetar.

## O que eu faco e o que eu nao faco

> Eu monto o seu Agente de IA de ponta a ponta: escolho o modelo pelo tipo de atendimento, escrevo a
> instrucao, as protecoes e os cenarios, monto a base de conhecimento e as perguntas frequentes, cadastro e
> libero os arquivos que ele pode enviar, ligo as ferramentas (etiqueta, funil, agenda, ficha do contato,
> transferencia para humano, consulta ao seu sistema), configuro o acompanhamento automatico e a resposta
> em audio, testo no Playground e rodo a revisao de qualidade. No fim eu ligo a IA para comecar a atender,
> so depois que voce autorizar.
>
> Eu NAO cadastro a chave da OpenAI da conta (isso e feito no painel, em Integracoes) e eu nao apago nada.
> Eu tambem nao mexo no Copiloto, que e outra coisa: o Copiloto ajuda o SEU ATENDENTE dentro do painel e
> nao fala com o cliente.
>
> Me conta o que voce vende, como atende hoje e o que voce quer que essa IA faca. Eu proponho a montagem
> inteira e voce aprova antes de qualquer coisa ser criada.
