# A mecanica do fluxo e como descobrir onde ele travou

O que faz o desenho funcionar de verdade: a memoria do robo (variaveis), a ficha de cada pessoa
dentro do fluxo (sessao), a espera, a regra dos fios, o teste, o historico e as versoes. **A maior
parte dos defeitos de fluxo em producao nao e bloco errado - e fio que faltou ligar e variavel que
sai vazia em silencio.**

## Indice

1. Variaveis: a memoria do robo
2. O dado mascarado - a armadilha mais cara
3. A sessao: a pessoa dentro do fluxo
4. Espera, tempo esgotado e tentativas
5. A regra dos fios
6. Trava de gatilho duplicado
7. Testar antes de ligar
8. Historico de execucoes: como ler
9. Arvore de diagnostico por sintoma
10. Versoes do fluxo
11. Campanha de Fluxo
12. Fluxo como ferramenta da IA
13. Limites e tetos
14. Ferramentas do conector, por tarefa
15. O que so da na tela

---

## 1. Variaveis: a memoria do robo

Uma variavel guarda um valor durante UMA conversa. Escrevendo `{{nome}}` num texto, o valor entra
ali. **A memoria morre quando a execucao termina** - variavel nao e campo de ficha.

### Quem CRIA variavel (so estes cinco)

| Fonte | Nome da variavel |
|---|---|
| Bloco Definir variavel | cada `name` da lista |
| Aguardar resposta guardando em variavel | o `saveVariable` |
| Bloco IA nos modos gerar e personalizado | `aiResponseVar` (padrao `ai_response`) |
| Bloco IA no modo intencao / sentimento / extrator | `ai_intent`, `ai_sentiment`, uma por dado extraido |
| Bloco Requisicao | `apiResponseVar` (padrao `api_response`), mais `api_response_status` |

**A lista de variaveis no cadastro do fluxo nao cria nada.** Ela e aceita, e devolvida pela API e
nenhum pedaco do motor le. Variavel so nasce de bloco.

### Variaveis prontas (nao precisa criar)

Contato: `contact.id`, `contact.name`, `contact.first_name`, `contact.last_name`, `contact.email`,
`contact.phone`, `contact.cpf`, `contact.cnpj`, `contact.rg`, `contact.date_of_birth`,
`contact.gender`, `contact.marital_status`, `contact.profession`, `contact.address.*` e
`contact.custom_attribute.CHAVE`.

Conversa: `conversation.id` (o numero que aparece na tela), `conversation.status`,
`conversation.team_id` e `conversation.custom_attribute.CHAVE`. Atendente: `agent.name`,
`agent.email`, `agent.id`. Caixa: `inbox.id`, `inbox.name`. Conta:
`account.custom_attribute.CHAVE`.

Ultimas falas: `{{last_response}}` (o que o cliente respondeu por ultimo) e
`{{last_agent_response}}` (a ultima mensagem publica que saiu para o cliente - nota interna nao
conta).

Quem disparou: `{{trigger.type}}` (o codigo do evento - compare SEMPRE por ele), `{{trigger.name}}`
(o rotulo legivel - use para ESCREVER) e `{{trigger.activated_at}}`, mais dados conforme o evento.

Quando o fluxo e disparado por webhook, o pacote inteiro recebido vira variavel:
`{{webhook.cliente.nome}}`, `{{webhook.itens.0.titulo}}`. Pacote acima de 256 KB nao vira variavel,
e em fluxo disparado por outro gatilho isso resolve vazio.

### O silencio

Variavel que nao existe vira **texto vazio**. Nao da erro, nao pinta de vermelho, nao aparece no
historico. Onde doi mais:

- Numa Condicao: a comparacao passa a ser contra vazio e o fluxo toma o RAMO ERRADO sem sinal.
- Num bloco de Requisicao: o parametro sai vazio e o sistema de fora responde erro.
- Numa mensagem: o cliente le a frase furada.

Escritas que sempre resolvem vazio: `custom_attributes` no plural (o certo e `custom_attribute`),
`{{env.X}}` (o certo e `{{account.custom_attribute.X}}`) e `{{api_response.payload.campo}}` (o
corpo fica direto sob a variavel: `{{api_response.campo}}`).

Existe uma lista fechada de apelidos que o sistema traduz sozinho:
`conversation.assignee.name` e `conversation.assignee.email` viram `agent.name` e `agent.email`;
`conversation.assignee_id` e `conversation.agent_id` viram `agent.id`;
`conversation.display_id` vira `conversation.id`; `contact.name.split.first` vira
`contact.first_name`; `conversation.team` vira o nome da equipe. **Qualquer outra composicao
continua caindo no silencio** - nao invente escrita nova, e nao mande o cliente reescrever fluxo
que ja esta funcionando com um desses apelidos.

Nome de variavel comecando com risco baixo e **ignorado**: esse prefixo guarda o estado interno da
execucao. No bloco Definir variavel o item e pulado; em `apiResponseVar` e `aiResponseVar` cai no
nome padrao.

Formulas: qualquer `{{x | filtro}}` sai do caminho simples e vai para o motor de formulas. Erro de
sintaxe devolve o texto original em vez de apagar. Existe um assistente que escreve a formula
(`lionchat_flows_generate_expression`).

---

## 2. O dado mascarado - a armadilha mais cara

**A resposta do cliente guardada em VARIAVEL passa por um filtro de dado sensivel antes de ser
gravada.** O trecho vira o texto `[FILTERED]` quando casa com um destes padroes:

- e-mail;
- CNPJ;
- 11 digitos seguidos (o padrao de CPF) - **isso inclui celular com DDD**;
- 13 a 19 digitos seguidos (o padrao de cartao) - **isso inclui telefone com codigo do pais**.

Ou seja: o fluxo pergunta o e-mail, guarda em variavel, manda para o bloco de Requisicao, e sai
`[FILTERED]` no lugar do dado - sem erro nenhum, com a variavel existindo e tendo valor. Uma
Condicao que compare `{{last_response}}` com um e-mail tambem nunca casa.

Vale para a variavel escolhida em `saveVariable` **e** para `{{last_response}}`. O que vai para o
campo do contato ou da conversa (`saveTo`) NAO passa por esse filtro - la o valor e gravado inteiro.

**O caminho certo**: guarde em campo do contato ou da conversa (`saveTo` com `contact_email`,
`contact_phone`, `contact_cpf`, `contact_attr`, `conversation_attr`), que grava o valor de verdade,
e leia depois com `{{contact.email}}`, `{{contact.phone}}` ou
`{{contact.custom_attribute.CHAVE}}`. Isso vale duplamente: o dado fica na ficha, aparece no
painel, serve de filtro de campanha e entra em relatorio.

Nao ha interruptor para desligar o mascaramento.

---

## 3. A sessao: a pessoa dentro do fluxo

Cada pessoa que entra ganha uma ficha de execucao: em que bloco esta, o que ja respondeu, o que deu
certo e o que deu errado.

| Estado | Quer dizer |
|---|---|
| `active` | andando |
| `waiting_input` | parada esperando a pessoa responder |
| `paused` | alguem pausou, ou o fluxo foi desligado |
| `completed` | terminou, ou saiu por uma condicao de saida |
| `error` | um bloco estourou |

**Uma execucao por pessoa por fluxo.** Tentar iniciar de novo enquanto a anterior esta viva devolve
"fluxo ja ativo para esta conversa".

**Mas a mesma conversa pode estar dentro de VARIOS fluxos ao mesmo tempo**, e uma unica mensagem do
cliente e entregue a todos eles. Depois disso, o sistema ainda tenta casar os gatilhos dos outros
fluxos ativos da caixa. E por isso que "dois robos responderam" quase nunca e defeito: sao dois
fluxos. A trava de gatilho duplicado nao resolve isso (ela so barra gatilho IGUAL). A solucao e um
fluxo so com Condicao roteando.

**O fluxo NAO para quando um atendente humano assume a conversa.** A resposta do atendente nao faz
o fluxo avancar (so mensagem que CHEGA do cliente avanca a espera), mas tambem nao encerra nada:
assim que o cliente responder, o robo volta a falar por cima do atendimento. Os unicos jeitos de
parar sao: condicao de saida (atendente assumiu, conversa encerrada), pausa manual na lateral da
conversa, ou o botao Parar do historico.

Controle manual (`lionchat_flow_sessions_update` com `action_type`): `pause`, `resume` e `remove`.
**`remove` nao e pausa**: encerra de vez e escreve um aviso na conversa dizendo quem parou.

Desligar o fluxo pausa todo mundo que esta dentro. Religando, so voltam os que o SISTEMA pausou -
quem um atendente pausou a mao continua pausado.

Apagar um bloco onde ha gente parada transforma essas execucoes em erro na proxima tentativa. **Por
ferramenta nao ha aviso**: chame `lionchat_flows_live_sessions` antes.

Editar fluxo no ar nao afeta quem ja esta dentro ate a proxima transicao - dai em diante, a versao
nova. Alguem pode atravessar uma edicao pela metade; o historico daquela execucao acende o aviso de
que o fluxo mudou.

---

## 4. Espera, tempo esgotado e tentativas

**Prazo**: `waitTime` (numero inteiro, nunca texto) + `waitUnit`. Campo ausente = espera para
sempre. Padrao recomendado: 60 minutos.

**Sem fio no caminho de tempo esgotado, a execucao e ENCERRADA quando o prazo estoura.** Quem
responder depois nao avanca mais aquele fluxo: a mensagem cai na conversa como comum e pode
disparar o gatilho de novo, criando uma execucao do zero. Quem quer "esperar ate responder" precisa
de prazo longo E o fio do tempo esgotado ligado num caminho explicito.

**Tentativas esgotadas e diferente de tempo esgotado.** `maxRetries` (padrao 3; vazio ou 0 vira 3)
conta erro de FORMATO. Cada erro reenvia a `invalidMessage` (que aceita variaveis) e mantem a
espera. Sem fio proprio, `retries_exhausted` cai no fio do `timeout`; sem nenhum dos dois, encerra.

**Campo numerico esvaziado na tela grava texto vazio, nao nulo** - e o sistema trata texto vazio
como o valor que a tela exibe. Ao montar por ferramenta, mande numero inteiro (`waitTime: 60`),
nunca texto ("60") nem vazio.

**Menu de botoes e uma segunda forma de espera**, com regras proprias: prazo em minutos, horas ou
DIAS, e um modo de lembrete que reenvia os mesmos botoes uma vez antes de seguir. E cuidado com os
nomes: `no_response` quer dizer "respondeu OUTRA coisa" (na tela: Outros); quem some de vez sai por
`no_reply_timeout`. Sao caminhos diferentes.

**Mensagem picada**: `groupInputsSeconds` junta os baloes por alguns segundos antes de validar.
Cada balao novo reinicia o relogio.

**Sinal de execucao presa**: no historico, o MESMO bloco aparece com estado "esperando resposta"
duas ou mais vezes. Causa legitima hoje: espera sem prazo nenhum - e enquanto ela existe, aquele
fluxo nao re-dispara para aquela conversa.

---

## 5. A regra dos fios

Todo fio precisa de `sourceHandle`, e **fio com nome so e seguido por quem casa com o nome**. Isso
mudou em agosto de 2026 e e a regra mais importante do produto hoje.

O que isso significa na pratica:

- Resposta "A" num menu com so a saida do "B" ligada **nao** e entregue no caminho do B - a pessoa
  some do fluxo.
- Envio bem-sucedido **nao** desce pelo fio de "janela de 24h fechada".
- Quem nao respondeu **nao** cai no caminho do botao ligado.
- **Sem fio elegivel, o fluxo TERMINA ali.** Nao existe "cai no proximo bloco".

O unico ultimo recurso e uma ligacao SEM nome de saida, que so existe em fluxo antigo. As duas
pontes explicitas que continuam valendo: `option_X`, `button_X` e `no_response` sem fio proprio
caem no `success` se ele existir; e o `partial` do bloco de grupo cai no `success`.

Saidas que **nunca** podem ser aproveitadas por outro caminho nenhum: `timeout`,
`no_reply_timeout` e `error`. Sem fio proprio, o fluxo termina ali (no caso do erro, em vermelho).
`retries_exhausted` tem uma unica reserva: sem fio proprio, ele segue o fio do `timeout`.

Bloco que da erro **sem fio no `error` mata a pessoa ali** - a execucao vira erro e nada retenta.
Isso fecha o raciocinio: "parou no meio" pode ser fio faltando (fim silencioso, historico sem
vermelho) OU erro sem fio de erro (historico vermelho).

Mexeu na configuracao de um bloco, revise os fios dele: a saida condicional pode ter deixado de
existir e o fio virou fantasma.

---

## 6. Trava de gatilho duplicado

Dois fluxos ATIVOS com o mesmo gatilho, na mesma caixa e no mesmo modo: a ativacao e recusada.
Confira antes com `lionchat_flows_check_conflicts` (manda `inbox_ids` e `triggers` no mesmo formato
salvo no fluxo; nao salva nada). A resposta traz qual fluxo, qual gatilho e qual caixa.

**So colidem gatilhos do MESMO tipo.** Card criado com card criado (e card movido com card
movido) quando os funis e etapas se sobrepoem; mensagem recebida com mensagem recebida por
palavra-chave em comum (**lista vazia significa "pega tudo" e colide com tudo**); etiqueta por
etiqueta em comum; conversa encerrada sempre colide.

Isentos (podem coexistir de proposito): webhook, inicio manual e campanha. Fluxo desligado nao gera
conflito.

Os tres gatilhos de formulario publico e os quatro de agendamento tem uma regua PROPRIA: colidem
por CONTA, nao por caixa - dois fluxos ativos ouvindo o mesmo formulario (ou o mesmo tipo de
agendamento) colidem mesmo estando em caixas diferentes, ou sem caixa nenhuma.

**Nao fique reativando.** Explique o conflito e ofereca desligar o outro fluxo ou unir os dois num
fluxo so, com gatilho amplo e um bloco de Condicao roteando.

---

## 7. Testar antes de ligar

**Fluxo de conversa nao tem modo de ensaio.** Teste iniciando o fluxo numa conversa de teste com
`lionchat_conversations_flow_sessions_create` e leia o resultado no historico.

**Ferramenta da IA tem ensaio.** `lionchat_flow_tools_run` passando o numero de uma conversa vira
SEMPRE simulacao:

- E fingido: envio de mensagem (texto, modelo, midia, resposta pronta), o bloco Acoes inteiro, a
  atribuicao do randomizador e a pausa entre baloes (nao dorme de verdade). O resultado mostra
  "enviaria: ..." em cada um.
- **Roda DE VERDADE**: o bloco de Requisicao (a chamada bate no sistema do cliente mesmo) e o bloco
  de IA (custo real). O webhook do bloco Acoes e fingido.
- A execucao ganha o selo de simulacao no historico.

Outros testes: `lionchat_flows_test_ai_node` roda so o bloco de IA com uma conversa real, sem
gravar nem enviar (a IA roda de verdade). `lionchat_flows_create_3` bate na URL do bloco de
Requisicao e devolve a resposta. `lionchat_flows_pin_test_result` congela a ultima resposta da
Requisicao para os blocos seguintes terem dado com que trabalhar (vale 30 dias, ate 50 blocos por
fluxo). `lionchat_flows_create_4` mostra como uma formula vai renderizar.

---

## 8. Historico de execucoes: como ler

`lionchat_flows_executions_list` lista quem entrou; `lionchat_flow_sessions_show` abre uma execucao
passo a passo. **So administrador enxerga** - ou quem tem a permissao de gerenciar fluxos no
cargo personalizado (o fluxo em si qualquer atendente ve).

Filtros da lista: `status` (use exatamente `active`, `waiting_input`, `paused`, `completed` ou
`error` - **valor fora dessa lista e ignorado em silencio e a lista volta sem filtro**), `from` e
`to` (janela maxima de 90 dias), `q` (nome do contato ou numero da conversa), `page`, `per_page`.

A lista mostra estado, inicio, duracao, rotulo do gatilho, numero da conversa, nome do contato,
bloco atual e o motivo resumido do erro.

O detalhe traz o passo a passo com o estado e o erro de cada bloco, a foto das variaveis no fim, os
blocos visitados e as setas percorridas. Marca `node_removed` quando o bloco foi apagado depois. Se
o registro passar de 100 passos, avisa que foi cortado.

**O passo Inicio mostra o que disparou** aquela execucao: contato, conversa, etiqueta, responsavel,
card com etapa anterior e nova, politica de SLA, grupo, formulario, anuncio com as respostas,
produto do pagamento, dados da agenda. **Cuidado: isso e area so de registro** - esses fatos NAO
existem como variavel e uma Condicao que conte com eles resolve vazio.

**Recusa depois do envio**: quando o canal aceita e recusa depois (o WhatsApp respondendo por
webhook de estado), o passo daquele bloco e reescrito como erro com o motivo real. E por isso que
um bloco pode aparecer verde no comeco e vermelho depois.

**Dado mascarado no historico**: variavel cujo nome lembra token, senha, segredo, CPF, CNPJ,
telefone ou e-mail aparece como FILTERED, e valores sao cortados em 500 caracteres.

**Retencao: 30 dias.** Uma faxina diaria apaga as execucoes concluidas e com erro mais velhas que
isso. A tela oferece janela de ate 90 dias, mas nao ha nada la atras para achar.

---

## 9. Arvore de diagnostico por sintoma

**"Ninguem entra no fluxo"**
1. O fluxo esta ligado? (`lionchat_flows_list`)
2. A caixa certa esta vinculada?
3. O gatilho existe e esta em `data.items`? (`lionchat_flows_show`) Gatilho com nome que o motor
   nao conhece nunca dispara e nao da erro.
4. Ha outro fluxo ativo com o mesmo gatilho na mesma caixa? (`lionchat_flows_check_conflicts`)
5. O fluxo esta no fim de uma corrente (automacao que chama fluxo que chama fluxo)? A corrente e
   cortada no quinto salto, em silencio.
6. E fluxo de grupo tentando disparar em conversa individual, ou o contrario?

**"O fluxo para no meio, sem erro nenhum"**
Quase sempre e saida sem fio. Abra a execucao (`lionchat_flow_sessions_show`), veja qual foi o
ultimo bloco e liste as saidas que aquela configuracao cria. A que a pessoa usou nao tinha fio.

**"O fluxo para no meio, com vermelho no historico"**
Bloco que estourou sem fio no `error` (Requisicao, IA, Espera com variavel invalida). Leia o motivo
no passo vermelho e ligue o caminho de erro.

**"A mensagem chegou com um buraco no meio"**
Variavel que nenhum bloco anterior daquele caminho criou, ou escrita errada. Confira na foto das
variaveis da execucao se o nome aparece.

**"O dado do cliente virou a palavra FILTERED"**
Voce guardou e-mail, CPF, CNPJ ou telefone numa variavel. Guarde em campo do contato (secao 2).

**"Todo mundo cai no caminho padrao da Condicao"**
Campo escrito sem as chaves duplas; ou `custom_attributes` no plural; ou regra sem valor preenchido
(regra sem valor e pulada, e se todas forem puladas a saida nunca casa); ou a variavel comparada
esta vazia.

**"O cliente respondeu e recebeu a mensagem de quem nao respondeu"**
Era o comportamento antigo, hoje corrigido. Se aparecer de novo, confira se a saida certa tem fio -
sem ela, hoje o fluxo termina em vez de ir pelo caminho errado.

**"Fica travado esperando resposta para sempre"**
O MESMO bloco aparece duas vezes com estado de espera no historico. Espera sem prazo configurado;
ponha prazo e ligue o caminho de tempo esgotado.

**"O balao ficou verde e o cliente nao recebeu"**
O canal aceitou e recusou depois. O passo daquele bloco e reescrito com o motivo real - leia ali.
Em caixa Oficial, quase sempre e janela de 24 horas fechada ou modelo nao aprovado.

**"Dois robos responderam a mesma pessoa"**
Dois fluxos ativos na mesma caixa com gatilhos diferentes. Junte num fluxo so com Condicao.

**"O robo continuou falando depois que assumi a conversa"**
Comportamento esperado. Monte uma condicao de saida.

---

## 10. Versoes do fluxo

Toda vez que alguem salva, o sistema guarda uma foto do desenho e o resumo do que mudou. Serve para
auditar quem mexeu e para o historico desenhar a execucao sobre o desenho DA EPOCA.

Salvar sem mudanca de conteudo nao cria versao, e mover bloco no desenho nao conta como mudanca.
Alem do desenho, renomear o fluxo, mudar a descricao, ligar/desligar e mudar o modo tambem viram
versao. Retencao: 90 dias. Quem ve o fluxo ve o historico de edicoes.

**Nao existe botao de restaurar versao.** `lionchat_flow_versions_list` e
`lionchat_flow_versions_show` so leem. Voltar atras e pegar a foto da versao e mandar de volta como
desenho novo com `lionchat_flows_update`.

---

## 11. Campanha de Fluxo

Em vez de mandar UMA mensagem para uma lista, inicia o FLUXO INTEIRO para cada pessoa, cadenciado.

Exige: fluxo ATIVO, vinculado a caixa da campanha, **com o gatilho `campaign_trigger` no bloco
Inicio**. Sem ele o fluxo nem aparece na lista de escolha da campanha e a criacao e recusada. Para
listar os elegiveis: `lionchat_flows_list` com `with_campaign_trigger` e `inbox_id`.

Criar: `lionchat_campaigns_create` passando `flow_id`. O campo de mensagem fica vazio de proposito
(quem manda texto sao os blocos). A cadencia vai em `template_params`: `delay_min`, `delay_max` e
`daily_cap`. Vale em WhatsApp Oficial e WhatsApp por QR Code.

**Em caixa Oficial, o primeiro bloco de mensagem de CADA caminho precisa ser modelo aprovado** - o
conferidor olha todos os ramos que saem do Inicio e recusa a campanha nomeando os blocos sem
modelo.

Quem e pulado: so quem JA esta nesse mesmo fluxo. Conversa aberta com atendente nao pula. Execucao
de OUTRO fluxo nao pula. Conversa encerrada e reaberta em vez de criar outra. Motivos de pulo que
aparecem no relatorio: fluxo removido, fluxo inativo, gatilho removido, caixa desvinculada, ja
tinha execucao ativa, contato sem telefone, modelo necessario, conversa falhou, execucao duplicada,
erro interno.

Acompanhar: `lionchat_campaigns_flow_report`. Parar: `lionchat_campaigns_stop_flow` (cancela so
quem esta pendente). Adiar: `lionchat_campaigns_reschedule_flow` (desloca preservando o espacamento).

**Aviso obrigatorio ao cliente**: o teto por dia conta PESSOAS, nao mensagens. Se o fluxo manda 4
mensagens por pessoa, o ritmo real contra o WhatsApp e quatro vezes o configurado - e em caixa por
QR Code isso e risco de banimento do numero. Os freios de ritmo comuns nao cobrem mensagem de fluxo.

---

## 12. Fluxo como ferramenta da IA

Em vez de responder a mensagens, o fluxo vira uma ferramenta que o AI Agente chama sozinho
("consultar o status do pedido", "calcular o frete"), recebe parametros da IA, roda na hora e
devolve um dado para a IA compor a resposta.

- Crie com `lionchat_flow_tools_create`, **nunca** com `lionchat_flows_create`.
- `tool_name`: minusculas, comeca com letra, so letras, numeros e risco baixo, ate 50 caracteres,
  unico na conta. `tool_description`: ate 500 caracteres - **e o texto que a IA le para decidir
  quando chamar, a peca mais importante do desenho.**
- **Nao pode ter caixa de entrada vinculada.** A validacao recusa.
- Blocos permitidos: Inicio, Fim, Requisicao, Condicao, Definir variavel, IA, Nota adesiva,
  Randomizador, Acoes e Enviar mensagem. **Nao existem** Aguardar resposta, Espera e Gestao de
  Grupos. No bloco Acoes nao existem enviar webhook nem iniciar outro fluxo.
- O bloco Fim e o que define o retorno. Sem nenhum Fim alcancado, a ferramenta devolve VAZIO para
  a IA - e ela responde ao cliente sem o dado que foi buscar.
- Vincule ao assistente com `lionchat_flow_tools_assistants_update`. **Sem vincular, a IA nao
  conhece a ferramenta.**
- O parametro "perguntar sempre ao cliente" (`always_ask`) obriga a IA a confirmar aquele dado com
  a pessoa mesmo que ja exista valor salvo. Nasce desligado.
- Tetos: 50 blocos por execucao, 3 chamadas de IA por execucao e tempo de resposta padrao de 20
  segundos (45 quando o fluxo tem bloco de Requisicao).
- O bloco de IA aqui roda sem persona, sem base de conhecimento e sem ferramentas - para nao haver
  recursao infinita.
- Historico proprio: `lionchat_flow_tools_executions_list` e `lionchat_flow_tool_executions_show`.
  Retencao de 30 dias.

---

## 13. Limites e tetos

| Teto | Valor | O que acontece |
|---|---|---|
| Blocos por rodada | 50 | A rodada PARA no bloco atual. Fluxo que da volta em si mesmo bate nisso |
| Saltos entre motores (fluxo, automacao, fluxo) | 5 | A corrente e cortada EM SILENCIO no quinto |
| Tamanho do desenho | 2 MB | O salvar recusa |
| Passos guardados por execucao | 100 | O historico avisa que foi cortado |
| Participantes por execucao no bloco de grupo | 20 | Trava contra banimento |
| Mencoes por mensagem de grupo | 50 | Trava contra banimento |
| Tempo da chamada no bloco Requisicao | 25 segundos | Sai pela saida de erro |
| Retencao do historico de execucoes | 30 dias | Some |
| Retencao das versoes do desenho | 90 dias | O historico avisa que o fluxo mudou em vez de desenhar a epoca |

Estrutura minima para salvar: `flow_data` com `nodes` e `edges` (mesmo `edges` vazio), todo bloco
com `id` e `type`, exatamente um bloco Inicio, nenhum fio voltando para o proprio bloco.

---

## 14. Ferramentas do conector, por tarefa

**Consultar o formato antes de montar**
`lionchat_flows_schema_reference` (sem parametros; sempre disponivel) e os recursos
`lionchat://docs/flowbuilder-design-guide` e `lionchat://docs/flowbuilder-patterns`.

**Montar**
`lionchat_flows_list` (lista leve: nome, tipo, modo, ativo, caixas, tags, tipos de gatilho - **nao
traz o desenho**), `lionchat_flows_show` (o desenho completo, mais contadores por bloco),
`lionchat_flows_create`, `lionchat_flows_update` (**substitui o desenho inteiro**),
`lionchat_flows_toggle`, `lionchat_flows_destroy`, `lionchat_flows_create_1` (duplicar - a copia
nasce desativada e SEM caixa nenhuma, de proposito).

**Conferir antes de mexer**
`lionchat_flows_check_conflicts`, `lionchat_flows_live_sessions`.

**Testar**
`lionchat_conversations_flow_sessions_create`, `lionchat_flows_test_ai_node`,
`lionchat_flows_create_3` (testar a requisicao), `lionchat_flows_create_4` (prever a formula),
`lionchat_flows_generate_expression`, `lionchat_flows_pin_test_result`.

**Diagnosticar**
`lionchat_flows_executions_list`, `lionchat_flow_sessions_show`, `lionchat_flow_sessions_update`,
`lionchat_conversations_flow_sessions_list`.

**Versionar**
`lionchat_flow_versions_list`, `lionchat_flow_versions_show`.

**Gatilho de data**
`lionchat_flows_scheduled_firings_list` (os disparos futuros: quando, para quem, por qual campo) e
`lionchat_flows_scheduled_firings_cancel`.

**Campanha de fluxo**
`lionchat_campaigns_create` com `flow_id`, `lionchat_campaigns_flow_report`,
`lionchat_campaigns_stop_flow`, `lionchat_campaigns_reschedule_flow`.

**Ferramenta da IA**
`lionchat_flow_tools_create`, `_update`, `_list`, `_show`, `_toggle`, `_destroy`,
`lionchat_flow_tools_run`, `lionchat_flow_tools_assistants_list` e `_assistants_update`,
`lionchat_flow_tools_executions_list`, `lionchat_flow_tool_executions_show`.

**Levantar ids antes de montar**
`lionchat_inboxes_list`, `lionchat_teams_list`, `lionchat_agents_list`, `lionchat_labels_list`,
`lionchat_funnels_list`, `lionchat_custom_attributes_list`, `lionchat_captain_assistants_list`,
`lionchat_sla_list`, `lionchat_offers_list`, `lionchat_kanban_config_list`,
`lionchat_inboxes_whatsapp_templates_list`, `lionchat_account_variables_list`,
`lionchat_upload_create` (subir o arquivo antes de usa-lo num balao de midia).

**Disparar de outro lugar**
`lionchat_kanban_items_start_flow` (a partir de um card do funil).

---

## 15. O que so da na tela

- Arrastar bloco, puxar fio de uma saida ate outro bloco, pegar a ponta da linha e mover a ligacao,
  apagar a ligacao pelo botao de lixeira que aparece sobre a linha, desfazer com Ctrl+Z.
- Copiar e colar blocos entre fluxos (Ctrl+C e Ctrl+V, que passa pela area de transferencia do
  sistema e funciona ate entre maquinas) e exportar/importar o fluxo em arquivo.
- Ver os contadores de acerto e erro desenhados em cada bloco, e o menu de ver registros por bloco.
- O aviso ao salvar dizendo quantas pessoas estao paradas num bloco que voce vai apagar (por
  ferramenta esse aviso nao existe - chame `lionchat_flows_live_sessions` por conta propria).
- Escolher o modelo aprovado do WhatsApp com previa. **O bloco guarda uma FOTO do modelo (corpo e
  botoes)**: se o modelo mudar na Meta, so ao REABRIR o bloco na tela o sistema atualiza - e ao
  atualizar, o fio do botao antigo e removido e precisa ser religado. Botao renomeado sem isso faz
  todo mundo cair em "outra resposta".
- Anexar arquivo direto no bloco de mensagem (por ferramenta, suba antes com
  `lionchat_upload_create`).
- Ligar o recurso de ferramenta da IA na conta - isso e do administrador da plataforma, nao da
  conta do cliente.
