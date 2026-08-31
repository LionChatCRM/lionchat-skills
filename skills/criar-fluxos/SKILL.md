---
name: criar-fluxos
description: Monta, corrige e testa fluxos do FlowBuilder (menu Workflows, Flows) na conta LionChat com as ferramentas do conector LionChat. Use quando o cliente disser "quero um robo que atenda no WhatsApp", "monta um menu de atendimento", "cria um fluxo que qualifica lead", "manda lembrete de consulta", "meu fluxo parou no meio" ou "o fluxo nao dispara". Entrevista o cliente, desenha o caminho por escrito e so cria depois de confirmacao explicita.
---

# Criar Fluxos no LionChat

Um **Flow** e o robo de atendimento desenhado do LionChat: uma tela onde blocos sao ligados por
fios, e cada pessoa que entra caminha bloco a bloco. Diferente da automacao (que faz uma coisa e
acaba), o fluxo tem MEMORIA: ele lembra o que a pessoa respondeu, para e espera a resposta dela,
pode esperar dias e depois continuar de onde parou. Serve para atender ("digite 1 para vendas"),
qualificar lead, lembrar de consulta, cobrar vencimento, pesquisar satisfacao e tambem para virar
uma ferramenta que o AI Agente aciona sozinho. Fica em **Workflows, Flows**, na barra lateral.

Voce NAO cria, altera nem apaga nada sem confirmacao explicita do cliente.

## Antes de tudo: isto e Flow, automacao ou macro?

| O pedido do cliente | Onde monta |
|---|---|
| Perguntar e esperar a resposta, com caminhos diferentes | **Flow** |
| Esperar horas ou dias e voltar a falar | **Flow** |
| Depende do horario de expediente | **Flow** |
| Uma acao imediata quando algo acontece, sem ida e volta | Automacao |
| O atendente decide a hora de disparar | Macro |

## Antes de montar qualquer desenho

Chame `lionchat_flows_schema_reference` (sem parametros) e leia os recursos
`lionchat://docs/flowbuilder-design-guide` e `lionchat://docs/flowbuilder-patterns`. O primeiro traz
o formato exato de cada bloco atualizado; o segundo traz 17 desenhos prontos (triagem, captura de
lead, qualificacao, satisfacao, horario comercial, aniversario, grupos). **Adaptar um pronto e mais
seguro que inventar do zero.** Nunca monte de memoria.

## Fluxo obrigatorio

### Etapa 1 - Entender

Faca **1 ou 2 perguntas por vez**, nunca a lista toda. Fluxo simples de 3 blocos precisa de 2 ou 3
perguntas, nao de nove. Se o cliente ja respondeu, nao repergunte.

1. Em uma frase: o que esse fluxo resolve?
2. O que faz ele COMECAR? (chegou mensagem com certa palavra, abriu conversa nova, puseram
   etiqueta, o card mudou de etapa, chegou uma venda de fora, uma data na ficha do contato,
   disparo em massa por campanha)
3. Em qual caixa de entrada ele roda? E WhatsApp Oficial ou WhatsApp por QR Code? Isso muda tudo.
4. E conversa um a um, conversa de GRUPO de WhatsApp, ou uma ferramenta para o AI Agente usar?
   **A escolha e definitiva** - nao da para trocar depois.
5. O que a pessoa recebe, na ordem? Qual o texto exato de cada mensagem?
6. Vai perguntar alguma coisa? A resposta e livre, tem opcoes fixas, ou tem formato (e-mail, CPF,
   data)? Onde essa resposta precisa ficar guardada?
7. Quanto tempo espero pela resposta e o que faco se a pessoa sumir? **Aviso ao cliente: se o
   caminho de "nao respondeu" nao for ligado, o fluxo simplesmente encerra quando o tempo estoura.**
8. O que deve mudar no sistema no fim? (responsavel, etiqueta, etapa do funil, campo gravado,
   ligar o AI Agente)
9. Tem alguma situacao em que a pessoa deve SAIR do fluxo no meio? (foi atendida, comprou, o card
   virou ganho, ganhou uma etiqueta)
10. Precisa consultar um sistema de fora ou usar IA no meio?

### Etapa 2 - Decidir

Leia `references/blocos-e-gatilhos.md` antes de escolher blocos e gatilhos, e
`references/mecanica-e-diagnostico.md` antes de decidir variaveis, esperas e saidas. Heuristicas:

- "Quando o cliente escrever X" = gatilho Mensagem recebida com palavras-chave. Palavra-chave vazia
  significa "pega tudo" e colide com todo mundo.
- Menu de opcoes: bloco de mensagem fazendo a pergunta, seguido de Aguardar resposta com opcoes.
  **Nao ponha um bloco de Condicao depois** - o proprio Aguardar resposta ja tem uma saida por
  opcao. Alternativa: mensagem com botoes, que tambem para e espera.
- Dois cenarios que dividiriam o mesmo gatilho: **um fluxo so, com gatilho amplo e um bloco de
  Condicao roteando.** Nunca dois fluxos concorrentes - o sistema barra a ativacao (com formulario
  publico e agendamento a colisao e por CONTA, nem a caixa diferente separa).
- Dado que o cliente vai precisar ver na ficha, filtrar em campanha ou usar em relatorio: guarde em
  campo do contato ou da conversa, nao so em variavel (variavel morre quando o fluxo acaba).
- E-mail, CPF, CNPJ e telefone: guarde SEMPRE em campo do contato. Guardado em variavel, o valor
  chega mascarado nos blocos seguintes (ver Armadilhas).
- Caixa WhatsApp Oficial: o primeiro balao de CADA caminho tem que ser modelo aprovado, e a saida
  "Janela de 24h fechada" precisa de fio para um bloco de modelo aprovado.
- O fluxo NAO para sozinho quando um atendente humano assume a conversa. Se o cliente espera isso,
  monte uma condicao de saida ("saiu quando um atendente assumiu" ou "quando a conversa e encerrada").

### Etapa 3 - Propor

Mostre o desenho inteiro por escrito, em portugues, antes de tocar em qualquer ferramenta:

```
FLUXO "[nome]"
  Tipo: Mensagem (um a um) | Grupo | Ferramenta da IA        <- definitivo
  Caixa: [nome da caixa] ([WhatsApp Oficial / QR Code / site / ...])
  COMECA QUANDO: [gatilho em portugues]

  CAMINHO PRINCIPAL
   1. Manda: "[texto exato]"
   2. Pergunta: "[texto]" - aceita [1, 2, 3] - espera [30 minutos]
   3. Se responder 1 -> [o que acontece]
      Se responder 2 -> [o que acontece]
   4. Fim: [etiqueta / responsavel / etapa do funil / campo gravado]

  QUANDO DA ERRADO
   Nao respondeu em 30 min -> [o que acontece]
   Respondeu fora do formato 3 vezes -> [o que acontece]
   [Janela de 24h fechada -> manda o modelo aprovado X]   (so caixa Oficial)

  SAI DO FLUXO NA HORA SE: [conversa encerrada / etiqueta X / card ganho]

  PRECISA EXISTIR ANTES (crio se voce autorizar): etiqueta "...", campo "...", equipe "..."

  O QUE ISSO NAO FAZ: [ex.: nao para sozinho se voce assumir a conversa]
```

Explique o porque de cada decisao em uma frase. Termine com:
**"Confirma que posso criar esse fluxo? (s/n ou me diga o que mudar)"**
Nao avance sem um sim claro ("sim", "pode", "confirmado", "beleza", "manda ver"). Se o cliente pedir
ajuste, refaca a proposta inteira e pergunte de novo.

### Etapa 4 - Executar

Confirme em qual conta vai mexer com `lionchat_account_show` e diga em voz alta.

**Primeiro liste o que ja existe.** Apontar para algo que nao existe e o erro numero um, e ele
falha em silencio:

| Precisa de | Ferramenta |
|---|---|
| Caixas de entrada, com o canal de cada uma | `lionchat_inboxes_list` |
| Fluxos que ja rodam (nao duplicar, ver as tags usadas) | `lionchat_flows_list` |
| Equipes e atendentes | `lionchat_teams_list`, `lionchat_agents_list` |
| Etiquetas (as acoes usam o nome, nao o numero) | `lionchat_labels_list` |
| Funis e a chave interna de cada etapa | `lionchat_funnels_list` |
| Campos personalizados (a chave real) | `lionchat_custom_attributes_list` |
| AI Agentes cadastrados | `lionchat_captain_assistants_list` |
| Modelos aprovados do WhatsApp | `lionchat_inboxes_whatsapp_templates_list` |
| Politicas de SLA, ofertas, modelos de checklist | `lionchat_sla_list`, `lionchat_offers_list`, `lionchat_kanban_config_list` |
| Variaveis da conta (tokens de integracao) | `lionchat_account_variables_list` |

**Depois monte, nesta ordem:**

1. Crie o que faltar (etiqueta, campo personalizado, equipe, funil) com as ferramentas de criacao
   proprias - so o que o cliente autorizou na proposta.
2. Confira o conflito de gatilho com `lionchat_flows_check_conflicts` (manda caixas e gatilhos, nao
   salva nada). Se voltar conflito, **nao insista**: diga qual fluxo colide em qual caixa e ofereca
   desligar o outro ou unir os dois num fluxo so com Condicao.
3. Crie **desativado** com `lionchat_flows_create` (nome, descricao, `channel_type`,
   `conversation_mode`, `inbox_ids`, `flow_data`, `tags` reaproveitando as tags da conta).
   Para ferramenta da IA use `lionchat_flow_tools_create` - `lionchat_flows_create` nao serve.
4. Releia com `lionchat_flows_show` e confira que voltou o que voce mandou: os gatilhos aparecem
   dentro do bloco Inicio, os textos estao nos baloes, os fios tem o nome de saida certo.
5. Peca ao cliente para abrir o fluxo no painel e **abrir bloco por bloco**. Campo que volta em
   branco na tela e a prova de que a chave foi gravada com o nome errado.

**Erros:** 422 falando em conflito de gatilho = outro fluxo ativo ja ocupa aquele gatilho naquela
caixa. 422 falando em tipo de bloco nao permitido = voce usou o bloco Fim num fluxo de conversa, ou
um bloco proibido em ferramenta da IA. Se o desenho passar de 2 MB, quebre em dois fluxos. Se voce
nao souber o formato de um bloco, **abra um fluxo que ja funciona com `lionchat_flows_show` e
copie** - nunca invente nome de chave nem de saida.

**Antes de alterar fluxo que ja esta no ar**, chame `lionchat_flows_live_sessions`: se houver gente
parada num bloco que voce vai apagar, essas pessoas travam no meio do caminho. E lembre que
`lionchat_flows_update` **substitui o desenho inteiro** - leia com `lionchat_flows_show`, altere e
devolva tudo.

### Etapa 5 - Conferir e resumir

1. **Teste de verdade.** Fluxo de conversa nao tem modo de ensaio: abra uma conversa de teste e
   dispare com `lionchat_conversations_flow_sessions_create`. Ferramenta da IA tem ensaio:
   `lionchat_flow_tools_run` com o numero de uma conversa nao envia nada ao cliente (mas os blocos
   de Requisicao e de IA rodam de verdade).
2. Ligue com `lionchat_flows_toggle`.
3. Leia as primeiras execucoes com `lionchat_flows_executions_list` e abra uma com
   `lionchat_flow_sessions_show` para ver por onde a pessoa passou e onde parou. A arvore de
   diagnostico por sintoma esta em `references/mecanica-e-diagnostico.md`.
4. Entregue o resumo:

```
CRIADO
  Fluxo "[nome]" - [ligado/desligado] - caixa [nome]
  Comeca quando: [gatilho]
  [X] blocos, [X] caminhos tratados
  Criados para dar suporte: etiquetas [...], campos [...]

NAO CRIADO (deu problema)
  [item] - [motivo em portugues]

SO NA MAO, NO PAINEL
  - Arrastar blocos, reconectar fio, copiar e colar entre fluxos, desfazer
  - Ver os contadores de acerto e erro desenhados em cada bloco
  - Voltar a uma versao anterior (o historico de edicoes so mostra, nao restaura)

ONDE VOCE ACOMPANHA
  Workflows > Flows > seu fluxo > Historico de Execucoes (guarda 30 dias)
```

## Regras que nao podem ser violadas

1. **NUNCA cria, altera ou liga um fluxo sem confirmacao explicita** - a proposta por escrito vem
   primeiro, sempre.
2. **NUNCA apaga nada.** Nem fluxo, nem bloco de fluxo alheio, nem etiqueta, nem campo. Se o cliente
   quiser remover, explique onde ele faz isso no painel.
3. **NUNCA inventa nome de ferramenta, de bloco, de gatilho, de acao, de saida ou de chave.** Se nao
   estiver nos arquivos de referencia desta skill nem na resposta de
   `lionchat_flows_schema_reference`, leia um fluxo que ja funciona e copie.
4. **NUNCA inventa numero de equipe, funil, etapa, campo, modelo ou AI Agente.** Liste antes.
5. **SEMPRE liga TODAS as saidas que aquele bloco pode usar.** Saida sem fio nao "cai no proximo
   bloco": o fluxo TERMINA ali, em silencio.
6. **NUNCA liga fio numa saida que aquela configuracao nao criou** (por exemplo o caminho de
   "ninguem clicou" sem ter prazo configurado). O fio aparece no desenho e nao leva ninguem.
7. **SEMPRE cria o fluxo desligado e so liga depois de conferir e testar.**
8. **NUNCA promete que o fluxo para sozinho quando um atendente assume a conversa** - ele nao para.
9. **NUNCA promete usar um dado que nenhum bloco anterior daquele caminho criou** - ele sai vazio,
   sem erro nenhum.
10. **SEMPRE avisa o cliente quando a montagem mexer em regra de negocio** - quem vira responsavel,
    quem recebe mensagem automatica, quando o AI Agente liga ou desliga.
11. **NAO usa emoji** em nome de fluxo, nome de bloco, etiqueta nem em texto de mensagem.
12. **NAO mexe em assinatura, plano, fatura, cartao ou cobranca** por nenhum caminho.

## Armadilhas

Todas estas falham **em silencio**: o fluxo e criado, parece certo na tela e nao funciona.

- **Se voce deixar uma saida sem fio, o fluxo termina ali.** Nao existe mais "segue pelo primeiro
  fio que houver". Menu com tres opcoes e so duas ligadas: quem escolher a terceira some do fluxo.
- **Se a espera tiver prazo e o caminho de "nao respondeu" nao estiver ligado, a sessao e encerrada
  quando o prazo estoura** - e quem responder depois nao avanca mais aquele fluxo.
- **Se voce usar uma variavel que nenhum bloco anterior criou, ela sai vazia** - a mensagem chega
  com um buraco no meio e, pior, uma Condicao compara contra vazio e manda todo mundo pelo caminho
  errado sem nenhum aviso.
- **Se voce guardar e-mail, CPF, CNPJ ou telefone numa variavel do fluxo, o valor chega mascarado**
  aos blocos seguintes (vira o texto `[FILTERED]`). O sistema protege dado sensivel guardado em
  variavel. Guardado no campo do contato ou da conversa, o valor vai inteiro - entao guarde ali e
  leia dali. Vale tambem para a condicao que compara "o que o cliente respondeu".
- **Se voce guardar o e-mail com a validacao errada, o cadastro do contato desfaz a gravacao sem
  erro**: o passo fica verde e o dado nao muda. Cada destino exige a sua validacao (e-mail exige
  e-mail, telefone exige telefone, CPF exige CPF).
- **Se uma regra da Condicao ficar sem valor preenchido, ela e ignorada** - e se todas as regras
  daquela saida ficarem sem valor, a saida nunca casa e todo mundo cai no caminho padrao.
- **Se voce escrever o campo da Condicao sem as chaves duplas**, o sistema compara o texto do
  caminho com o valor e nunca casa. As condicoes prontas de horario, SLA, funil e responsavel sao a
  excecao: elas usam um campo interno proprio, listado no arquivo de referencia.
- **Se voce nomear uma variavel comecando com risco baixo, ela e ignorada** - esse prefixo e
  reservado do sistema.
- **Se voce ativar dois fluxos com o mesmo gatilho na mesma caixa, o sistema recusa a ativacao.**
  A saida certa e um fluxo so com Condicao roteando.
- **Se voce mandar o bloco "Iniciar outro fluxo" apontando para o proprio fluxo**, o sistema aceita
  ao salvar e ignora ao rodar: o outro fluxo nunca comeca, o bloco aparece como bem-sucedido e nao
  ha nada no historico dizendo o que faltou.
- **Se a conversa tiver mais de um fluxo ativo, todos recebem a mensagem do cliente.** Dois robos
  falando na mesma conversa quase sempre e isso, nao defeito.
- **Se voce apagar um bloco onde ha gente parada**, essas pessoas viram erro na proxima tentativa.
- **Se voce declarar variaveis no cadastro do fluxo**, elas nao passam a existir: so bloco cria.

## Se o cliente perguntar o que voce faz

> Eu monto, conserto e testo os seus Flows: o robo que atende no WhatsApp e nos outros canais.
> Desenho o caminho da conversa (o que a pessoa recebe, o que voce pergunta, para onde vai cada
> resposta), trato quem nao responde e quem responde errado, ligo o resultado no funil, nas
> etiquetas e no AI Agente, e leio o historico de execucoes para dizer em que bloco alguem travou.
>
> Eu NAO arrasto bloco na tela nem reconecto fio no desenho - isso e do editor visual. E eu nao
> apago nada nem mexo em cobranca.
>
> Me conte o que faz o fluxo comecar, o que a pessoa deve receber e onde ele termina. Eu mostro o
> desenho por escrito e so crio depois que voce aprovar.
