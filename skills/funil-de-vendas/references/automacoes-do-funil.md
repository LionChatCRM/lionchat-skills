# Automacoes do funil e os outros caminhos de fazer algo acontecer sozinho

## Indice

1. Os cinco caminhos - qual usar para cada pedido
2. Automacao de etapa: os 3 gatilhos
3. Automacao de etapa: as 9 acoes
4. Formato exato de uma automacao de etapa
5. Como gravar sem apagar o que ja existe
6. Combinacoes proibidas e combinacoes que a tela esconde
7. O que a duplicacao copia e o que ela nao copia
8. Tarefa automatica do cartao ("Criar tarefa")
9. Cartao adiado ("Adiar ate")
10. Regra da conta com evento de Kanban
11. Fluxo (FlowBuilder) com gatilho de cartao, inclusive o de cartao parado
12. Macro: o botao de um clique para o atendente
13. Agente de IA mexendo no cartao
14. O que NAO existe (responda nao em vez de inventar)

---

## 1. Os cinco caminhos - qual usar para cada pedido

| O que a pessoa quer | Caminho certo | Fala com o cliente? |
|---|---|---|
| Mover o cartao, dar dono, escrever anotacao, aplicar checklist, copiar para outro funil, avisar um sistema externo, criar uma tarefa na agenda | **automacao de etapa** (dentro do funil) | nao, nunca |
| Avisar ou agir quando o cartao ficar parado X tempo numa etapa ou no funil | **fluxo** com gatilho "Card aberto na etapa" / "Card aberto no funil" (secao 11) | pode |
| Criar ou mover cartao quando chega mensagem, quando a conversa e resolvida, quando uma etiqueta e aplicada | **regra da conta** | pode, se voce somar uma acao de mensagem |
| Conversar com o cliente em varios passos, esperar resposta, esperar horas ou dias | **fluxo** | sim |
| Botao que o atendente clica dentro da conversa para o cartao andar | **macro** | so se a macro tiver acao de mensagem |
| O agente de IA cria ou move o cartao sozinho durante a conversa | **ferramentas de Kanban do agente de IA** | ja esta conversando |

**A regra que nao muda: automacao de etapa NUNCA manda mensagem para o cliente.** Nenhuma das nove acoes envia mensagem. A acao "avisar o time" apenas escreve uma linha de registro interno - ninguem recebe nada, nem por e-mail, nem no sininho.

## 2. Automacao de etapa: os 3 gatilhos

Sao exatamente tres. Qualquer outro nome e gravado sem erro e nunca roda.

| Gatilho | Quando dispara | O que vai no valor do gatilho |
|---|---|---|
| `card_created` | o cartao nasce, por qualquer caminho (a mao, regra, fluxo, IA, importacao) | a palavra `card_created` |
| `stage_moved` | o cartao CHEGA numa etapa | a **chave** da etapa de DESTINO |
| `status_change` | o cartao muda de estado | `won`, `lost` ou `snoozed` (adiado, desde 11/10/2026). **Nao existe `open`** aqui |

Observacoes que evitam frustacao:

- **Nao existe gatilho de SAIDA de etapa.** Nao da para dizer "quando o cartao sair de Proposta".
- **Nao existe gatilho de exclusao de cartao.**
- `stage_moved` olha para onde o cartao CHEGOU, nunca de onde ele veio.
- `status_change` so dispara quando o estado MUDA de verdade. Marcar Ganho um cartao que ja estava Ganho nao dispara nada. Com `snoozed`, dispara na passagem de aberto para adiado; trocar a data de um cartao que ja estava adiado nao dispara de novo.
- **Nao existe gatilho de cartao parado aqui.** Ele existe no FLUXO (secao 11).

## 3. Automacao de etapa: as 9 acoes

| Acao | O que faz | O que precisa junto |
|---|---|---|
| `move_to_stage` | move o cartao para outra etapa do mesmo funil | `stage`: a chave da etapa |
| `assign_agent` | poe um responsavel no cartao | `agent_id`: o numero da pessoa |
| `create_note` | escreve uma anotacao interna no cartao (autor fica "Sistema") | `note_text` |
| `apply_checklist_template` | aplica um modelo de checklist como um bloco novo no cartao | `template_id`: o numero do modelo cadastrado na conta |
| `duplicate_item` | cria uma copia do cartao em outro funil e etapa | `funnel_id` e `stage`; opcional `distribute_agents` |
| `send_webhook` | manda os dados do cartao para um endereco do cliente | `webhook_url` |
| `notify_team` | **so escreve uma linha de registro interno - nao avisa ninguem**. Nao grave regra nova com ela | `message` |
| `update_checklist` | marca ou renomeia tarefas de checklist que ja existem | `checklist_updates`; **a tela nao oferece esta acao** - trate como nao suportada |
| `create_task` | cria uma tarefa da Agenda PRESA ao cartao (20/10/2026). Ver a secao 8 | `title` e `responsible`; exige a Agenda (`unified_agenda`) ligada na conta |

A tela do funil oferece sete: as nove acima menos `update_checklist` e `notify_team` (que saiu da tela em 22/10/2026; regra antiga com ela aparece com um aviso amarelo). E desde 22/10 a tela tambem nao oferece "dar dono" com os gatilhos Ganho e Perdido - ver a secao 6.

## 4. Formato exato de uma automacao de etapa

As automacoes moram dentro das configuracoes do funil, na lista `automations`. Cada uma e assim:

```
{
  "id": "automation_1756500000000",
  "enabled": true,
  "trigger_type": "stage_moved",
  "trigger_value": "proposta_enviada",
  "action": "apply_checklist_template",
  "action_config": { "template_id": "tpl-proposta" }
}
```

- **`id`** - a tela usa o formato `automation_` seguido do horario em milissegundos. Mantenha algo unico e estavel; e por ele que a automacao e editada depois.
- **`enabled`** - grave `true`. Desligada ela fica guardada mas nao roda (ver a secao 6).
- **`trigger_value`** - a **chave** da etapa, nunca o nome visivel. Pegue as chaves em `lionchat_funnels_show`.
- **`action_config`** - o bloco de configuracao daquela acao. Faltando o campo obrigatorio, a automacao e descartada no proximo salvamento pela tela.
- **A ordem da lista e a ordem de execucao.** Duas regras do mesmo acontecimento (por exemplo duas em `won`) rodam de cima para baixo. Para mudar a ordem, reenvie a lista inteira na ordem nova - a configuracao de cada regra nao muda. No painel a pessoa segura e arrasta cada regra.

## 5. Como gravar sem apagar o que ja existe

**O bloco de configuracoes do funil e substituido inteiro.** Gravar so as automacoes apaga as pessoas, os times e as metas - e o funil que ficar com a lista de pessoas vazia vira ABERTO a todos, sem nenhum erro aparecer.

Sequencia obrigatoria:

1. `lionchat_funnels_show` - leia as configuracoes atuais.
2. Monte o bloco COMPLETO: pessoas + times + metas + automacoes (as que ja existiam mais as novas).
3. `lionchat_funnels_update` com o bloco inteiro.
4. `lionchat_funnels_show` de novo e confira campo a campo que tudo voltou.

O mesmo vale para as etapas: gravar a lista de etapas sem repetir uma delas EXCLUI essa etapa. Se a etapa tiver cartoes, o sistema recusa e diz que ela ainda tem cartoes; se estiver vazia, some calada e a chave dela nunca mais pode ser reaproveitada em automacao.

## 6. Combinacoes proibidas e combinacoes que a tela esconde

- **Automacao desligada fica guardada e nao roda.** Ate outubro de 2026 a tela apagava a regra desligada no proximo salvamento; hoje ela fica, desligada. Mesmo assim, so grave desligada se o dono pedir: uma regra parada esquecida confunde quem le o funil depois.
- **Automacao sem valor de gatilho e descartada** no proximo salvamento do funil pela tela.
- **Automacao com configuracao incompleta e descartada** pelo mesmo caminho: mover sem etapa, dar dono sem pessoa, duplicar sem funil ou sem etapa, avisar sistema externo sem endereco, aplicar checklist sem modelo, criar tarefa sem titulo ou sem responsavel.
- **"Criar tarefa" torta e recusada na hora**: o servidor confere cada regra `create_task` nova ou alterada e devolve 422 dizendo a posicao da regra e o campo a consertar. As outras regras do funil nao sao afetadas.
- **`assign_agent` com gatilho Ganho ou Perdido nao faz nada**: a regra roda com o cartao ja fechado, e cartao fechado nao aceita responsavel novo. O servidor aceita gravar, mas a regra fica morta. Nao grave.
- **`stage_moved` + `move_to_stage` nao funciona**: o proprio motor bloqueia mover de novo logo depois de mover, para nao criar um vaivem infinito entre duas colunas. A tela nem oferece a combinacao.
- **`card_created` + `duplicate_item`**: a tela esconde (cartao recem-nascido duplicando a si mesmo). O motor tem protecao contra o laco infinito, mas evite.
- **Dar dono a quem nao tem acesso ao funil e ignorado em silencio** - so fica um aviso no registro tecnico. Confira antes que a pessoa esta na lista de acesso do funil.
- **Cartao Ganho ou Perdido recusa qualquer mudanca de responsavel.** Os responsaveis ficam congelados.
- **Aplicar o mesmo modelo de checklist duas vezes cria DOIS blocos iguais** no cartao - nao ha verificacao de repeticao. Cuidado com automacao numa etapa por onde o cartao passa mais de uma vez.

## 7. O que a duplicacao copia e o que ela nao copia

E a acao mais usada no fechamento ("ganhou em Vendas, abre em Pos-venda"):

| Copia | Nao copia |
|---|---|
| valor, ofertas, campos personalizados do cartao, prazo, a foto de origem do trafego | anotacoes, checklist, responsaveis, historico |

Alem disso:

- O cartao novo nasce **aberto**, e o titulo ganha o sufixo " (copia)".
- O cartao novo nasce **sem responsavel**, a menos que voce ligue `distribute_agents` - ai ele entra no rodizio de responsaveis do funil de DESTINO.
- **Os dois cartoes ficam com a MESMA conversa.** A partir dai aquela conversa tem dois cartoes: o agente de IA passa a exigir qual dos dois quando for gravar um campo, e a ferramenta de mover pega o mais recente.

## 8. Tarefa automatica do cartao ("Criar tarefa")

A acao `create_task` cria uma tarefa COMUM da Agenda, presa ao cartao (ela aparece na aba Tarefas do cartao e na agenda de quem for o responsavel). E a resposta para "quando o lead entrar em Proposta, o vendedor tem que ligar em 2 dias uteis".

```
{
  "id": "automation_1761000000000",
  "enabled": true,
  "trigger_type": "stage_moved",
  "trigger_value": "proposta",
  "action": "create_task",
  "action_config": {
    "title": "Ligar para {{trigger.kanban.title}}",
    "task_type": "call",
    "due": { "amount": 2, "unit": "days", "at": "10:00", "business_days": true },
    "responsible": { "mode": "card_owner", "fallback_user_id": 7 }
  }
}
```

Regras:

- **So existe com a Agenda ligada na conta** (`unified_agenda` nos recursos da conta).
- `title` e obrigatorio e aceita os dados do cartao (`{{trigger.kanban.title}}`, `.funnel_name`, `.stage_name`).
- `task_type`: um dos tipos de fabrica ou um tipo ativo da conta. `agenda_id` vazio = a agenda padrao. `priority`: `none`, `low`, `medium`, `high`, `urgent`.
- **Prazo (`due`)**: em horas (`unit: "hours"`, de 1 a 720) ou em dias (`unit: "days"`, de 0 a 365). So em dias valem o horario (`at`, "HH:MM") e `business_days: true` (segunda a sexta, sem feriado de dia inteiro da agenda). O fuso e o da agenda. Prazo que ja passou nasce atrasado.
- **Duracao**: `duration_minutes` de 1 a 1440; sem ele vale o tempo do tipo (15, 30 ou 60 minutos). Com `occupies_agenda: true` a tarefa bloqueia o horario na agenda.
- **Responsavel (`responsible.mode`)**: `card_owner` (o primeiro responsavel do cartao; senao o da conversa; senao a pessoa de reserva em `fallback_user_id`, obrigatoria), `fixed` (uma pessoa, `user_id`), `funnel_rotation` (rodizio de quem acessa o funil) ou `team_rotation` (rodizio de um time, `team_id`).
- **Uma tarefa por acontecimento**: repetir o mesmo gatilho do mesmo acontecimento nao duplica.
- **Cartao importado por planilha nao cria tarefa.** Se uma pessoa mexer no cartao depois, ai cria.
- Excluir etapa com `lionchat_funnels_remove_stage` NAO dispara esta acao (nem nenhuma outra).
- Ao EXCLUIR o cartao, a tarefa presa a ele so e apagada se for simples (sem conversa, contato, agendamento, anexo). Tarefa automatica de cartao COM conversa grava a conversa, entao sobrevive e fica na conversa.

## 9. Cartao adiado ("Adiar ate")

Desde 11/10/2026 o cartao tem mais uma situacao: **Adiado** - "este negocio so volta a importar no dia X". Ele SAI da coluna e, na data marcada, abre sozinho e volta (numa rodada de ate cerca de 5 minutos depois da hora).

- **E situacao do CARTAO, nao da conversa.** Mensagem do cliente NAO abre o cartao.
- Gravado, o estado do cartao continua `open`. Quem diz que esta adiado e `effective_status: "snoozed"` com `snoozed_until`. Leia esses dois, nunca so o estado.
- So cartao ABERTO pode ser adiado, e a data precisa estar pelo menos 1 hora a frente.
- Mover, editar, anotar, criar tarefa ou trocar responsavel NAO encerra o adiamento. Marcar Ganho ou Perdido, abrir de proposito ou chegar a data encerram.
- Ferramentas: `lionchat_kanban_items_snooze` (adiar; adiar de novo so troca a data) e `lionchat_kanban_items_unsnooze` (abrir agora). **Abrir agora dispara o fluxo "Card aberto" na hora** - se esse fluxo manda mensagem, o cliente recebe ja. Confirme antes.
- Gravar `status: "open"` no `lionchat_kanban_items_update` NAO abre um cartao adiado. Para abrir, `unsnooze` ou `lionchat_kanban_items_create_2` com `open`.
- Na automacao de etapa, o gatilho e `status_change` com `snoozed`. Na regra da conta e na macro, a acao `snooze_kanban_item` (`amount` e `unit` em `hours` ou `days`), que FALHA visivel quando nao da para adiar. No fluxo, a acao de adiar, os gatilhos "Card adiado" e "Card aberto" e a condicao "Card adiado".
- No quadro o adiado so aparece com o botao **Adiados** ligado. Se o cliente diz que o cartao "sumiu", confira isso primeiro.
- **Laco de cadencia e permitido**: um fluxo "Card aberto" que manda mensagem e adia de novo roda ate alguem parar (no maximo uma volta por hora por cartao). Avise o dono antes de montar.

## 10. Regra da conta com evento de Kanban

E o motor geral de regras da conta (`lionchat_automation_rules_create`). Ele serve para os dois lados.

**Eventos de Kanban que ele escuta - sao exatamente dois:**

- `kanban_item_created`
- `kanban_item_stage_changed`

**Nao existe** `kanban_item_moved`, nem evento de ganho, nem de perda no motor da conta. Ganho e perda so tem gatilho na automacao de etapa e no fluxo.

**Acoes que mexem no cartao** - disponiveis a partir de QUALQUER evento da conta (mensagem chegou, conversa criada, etiqueta aplicada, conversa resolvida):

`create_kanban_item`, `move_kanban_item_to_stage`, `assign_agent_to_kanban_item`, `add_note_to_kanban_item`, `start_kanban_item_timer`, `stop_kanban_item_timer`, `set_kanban_item_status`, `snooze_kanban_item` (adiar, secao 9).

Detalhes que evitam surpresa:

- **Criar cartao e inteligente**: se ja existe um cartao ABERTO (ou adiado) daquela conversa no mesmo funil, ele MOVE o existente em vez de criar outro (a menos que voce marque que duplicatas sao permitidas). Cartao ja Ganho ou Perdido nao conta como existente - a pessoa que volta ganha um cartao NOVO e o fechado fica parado como historico. Isso e proposital.
- **Criar cartao NAO sorteia responsavel.** O cartao novo recebe o responsavel da CONVERSA; quem distribui e a caixa de entrada. Se a caixa atribuir depois, o cartao sem dono adota a pessoa.
- **Mover sem cartao nenhum nao faz nada** e o passo aparece como bem-sucedido. Existe uma opcao de criar se nao existir - use-a quando o cartao pode ainda nao ter nascido.
- O titulo do cartao criado aceita variaveis, por exemplo o nome do contato.
- E aqui, somando a acao de enviar mensagem, que sai o aviso imediato para o cliente. Para esperar horas ou dias, e fluxo.

## 11. Fluxo (FlowBuilder) com gatilho de cartao

O fluxo e o unico caminho que conversa em varios passos, espera resposta e espera muito tempo.

**Gatilhos de cartao:** `card_created`, `card_moved`, `card_won`, `card_lost`, `card_opened` ("Card aberto", inclusive quando o adiamento termina), `card_snoozed` ("Card adiado"), `card_attribute_changed` e os dois de PRAZO abaixo.

**Cartao parado (desde 28/10/2026)** - os gatilhos de prazo `card_open_in_stage` ("Card aberto na etapa") e `card_open_in_funnel` ("Card aberto no funil"):

- Disparam o fluxo UMA vez quando o cartao cruza um prazo: "na etapa" conta desde que o cartao ENTROU na etapa atual (sai e volta = conta do zero); "no funil" conta a idade do cartao desde que nasceu.
- Prazo de 15 minutos a 365 dias (`duration_value` + `duration_unit` em `minutes`, `hours` ou `days`). Funis e etapas escolhidos (`funnel_ids`, e `funnel_stages` no formato `"<funil>:<chave da etapa>"`); lista vazia = qualquer um.
- Conta cartao aberto ou adiado; ganho e perdido nunca. Cartao que ja tinha passado do prazo quando o fluxo foi ligado NAO dispara.
- A conferencia roda de 5 em 5 minutos (o aviso pode chegar ate uns 5 minutos depois do prazo). Cartao sem conversa nao dispara.
- E a resposta para "me avisa quando o lead ficar 1 hora parado em Novo Lead": fluxo com esse gatilho e, por exemplo, uma mensagem num grupo de WhatsApp da equipe. O desenho do fluxo e da skill de criar fluxos.

**Acoes de cartao dentro do fluxo:** criar cartao, mover de etapa, **marcar Ganho, marcar Perdido ou
reabrir**, **adiar**, dar responsavel, escrever anotacao, acrescentar checklist, **acrescentar oferta**,
criar tarefa na agenda e gravar campo do cartao. E o unico caminho automatico, alem da macro e da regra
da conta, que fecha o cartao. Nenhuma acao automatica marca co-participante.

**Desvios (condicoes):** existe cartao, cartao esta na etapa X, cartao ganho, cartao perdido, cartao adiado. Cuidado: ganho e perdido olham o ESTADO do cartao, nunca a coluna em que ele esta. Em fluxo disparado por cartao, as condicoes de cartao olham para o cartao QUE DISPAROU.

Para disparar um fluxo a mao a partir do cartao existe `lionchat_kanban_items_start_flow`. **Ele faz o fluxo FALAR com o cliente - so use com autorizacao para aquele disparo especifico.** Tres condicoes precisam valer: o cartao tem conversa ligada; o fluxo esta ativo e e de conversa; e o fluxo tem pelo menos um gatilho de cartao. Se ja existir uma execucao daquele fluxo naquela conversa, o pedido e recusado.

## 12. Macro: o botao de um clique para o atendente

E o unico caminho MANUAL: o atendente clica a macro dentro da conversa e o cartao anda. E a resposta certa para "quero um botao para marcar como Proposta Enviada".

Acoes de cartao que a macro aceita: criar cartao, mover de etapa, dar responsavel, escrever anotacao, ligar e desligar o cronometro, definir o estado (ganho, perdido, aberto), adiar, gravar campo do cartao, acrescentar checklist, acrescentar oferta e enviar conversao de anuncio (esta ultima so quem gerencia integracoes de marketing pode salvar).

Ferramentas: `lionchat_macros_list`, `lionchat_macros_create`.

## 13. Agente de IA mexendo no cartao

O agente de IA tem ferramentas proprias de Kanban, que precisam estar LIGADAS no assistente: criar cartao, mover cartao e escrever anotacao no cartao (mais uma ferramenta interna de listar funis, que o sistema liga sozinho). Gravar campo do cartao e feito pela ferramenta geral de atributos.

**Elas so aparecem se a conta tiver pelo menos um funil.** Monte o funil primeiro, depois ligue as ferramentas no assistente.

Regras que decidem se isso funciona ou nao:

- **A IA fala pelo NOME do funil e da etapa** (a chave tambem e aceita como alternativa). O sistema
  tenta o nome exato e, se nao achar, tenta por parte do nome. **Se duas etapas casarem, ele se recusa a
  adivinhar**: nao move nada e devolve a lista de etapas para a IA tentar de novo com um nome mais
  especifico. Nao e silencioso, mas gasta uma rodada e a IA pode desistir no meio - por isso nomes
  distintos importam: "Proposta" e "Proposta Enviada" no mesmo funil e armadilha.
- **Funil arquivado some para a IA.**
- **Mover pega o cartao mais recente daquela conversa, de qualquer funil.** Com dois cartoes, ela pode mover o errado.
- **Gravar campo do cartao exige cartao ligado a conversa**: sem nenhum, da erro; com dois ou mais, ela pergunta qual.
- **Criar cartao nao duplica**: reaproveita o cartao aberto daquela conversa naquele funil, e ignora o que ja esta Ganho ou Perdido.

O catalogo de ofertas do funil e o MESMO que o agente de IA usa quando o cliente pergunta preco - basta selecionar as ofertas nas configuracoes do assistente. Nao cadastre produto duas vezes.

## 14. O que NAO existe (responda nao em vez de inventar)

| Pedido comum | Resposta honesta |
|---|---|
| "me avisa quando o cartao ficar 5 dias parado" | EXISTE, mas pelo FLUXO: gatilho "Card aberto na etapa" (ou "no funil") com 5 dias (secao 11). Pela automacao de etapa, nao |
| "me notifica no sininho quando cair um cartao para mim" | nao existe notificacao de Kanban no sininho |
| "manda mensagem quando o cartao entrar na etapa" | nao pela automacao de etapa. Use regra da conta, fluxo, macro ou o agente de IA |
| "o modelo de mensagem da etapa dispara sozinho quando o cartao chega" | nao. O modelo de mensagem por etapa so sai por acao manual, no menu do cartao |
| "automacao quando o cartao sair da etapa" | nao existe. So chegada |
| "automacao quando o cartao for excluido" | nao existe |
| "apagar varios cartoes de uma vez" | so na TELA, selecionando os cartoes no quadro (roda em segundo plano). Pelo conector nao ha ferramenta - e voce nao apaga nada |
| "duplicar o funil inteiro" | a TELA tem o botao (icone de copiar na lista de funis), mas as automacoes da copia ficam apontando para as chaves de etapa do funil ORIGINAL e nao disparam - refaca-as. Pelo conector nao ha ferramenta: e ler um funil e criar outro, o que e criacao e exige confirmacao |
