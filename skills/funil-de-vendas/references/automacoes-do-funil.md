# Automacoes do funil e os outros caminhos de fazer algo acontecer sozinho

## Indice

1. Os cinco caminhos - qual usar para cada pedido
2. Automacao de etapa: os 3 gatilhos
3. Automacao de etapa: as 8 acoes
4. Formato exato de uma automacao de etapa
5. Como gravar sem apagar o que ja existe
6. Combinacoes proibidas e combinacoes que a tela esconde
7. O que a duplicacao copia e o que ela nao copia
8. Regra da conta com evento de Kanban
9. Fluxo (FlowBuilder) com gatilho de cartao
10. Macro: o botao de um clique para o atendente
11. Agente de IA mexendo no cartao
12. O que NAO existe (responda nao em vez de inventar)

---

## 1. Os cinco caminhos - qual usar para cada pedido

| O que a pessoa quer | Caminho certo | Fala com o cliente? |
|---|---|---|
| Mover o cartao, dar dono, escrever anotacao, aplicar checklist, copiar para outro funil, avisar um sistema externo | **automacao de etapa** (dentro do funil) | nao, nunca |
| Criar ou mover cartao quando chega mensagem, quando a conversa e resolvida, quando uma etiqueta e aplicada | **regra da conta** | pode, se voce somar uma acao de mensagem |
| Conversar com o cliente em varios passos, esperar resposta, esperar horas ou dias | **fluxo** | sim |
| Botao que o atendente clica dentro da conversa para o cartao andar | **macro** | so se a macro tiver acao de mensagem |
| O agente de IA cria ou move o cartao sozinho durante a conversa | **ferramentas de Kanban do agente de IA** | ja esta conversando |

**A regra que nao muda: automacao de etapa NUNCA manda mensagem para o cliente.** Nenhuma das oito acoes envia mensagem. A acao "avisar o time" apenas escreve uma linha de registro interno - ninguem recebe nada, nem por e-mail, nem no sininho.

## 2. Automacao de etapa: os 3 gatilhos

Sao exatamente tres. Qualquer outro nome e gravado sem erro e nunca roda.

| Gatilho | Quando dispara | O que vai no valor do gatilho |
|---|---|---|
| `card_created` | o cartao nasce, por qualquer caminho (a mao, regra, fluxo, IA, importacao) | a palavra `card_created` |
| `stage_moved` | o cartao CHEGA numa etapa | a **chave** da etapa de DESTINO |
| `status_change` | o cartao muda de estado | `won` ou `lost` |

Observacoes que evitam frustacao:

- **Nao existe gatilho de SAIDA de etapa.** Nao da para dizer "quando o cartao sair de Proposta".
- **Nao existe gatilho de exclusao de cartao.**
- `stage_moved` olha para onde o cartao CHEGOU, nunca de onde ele veio.
- `status_change` so dispara quando o estado MUDA de verdade. Marcar Ganho um cartao que ja estava Ganho nao dispara nada.

## 3. Automacao de etapa: as 8 acoes

| Acao | O que faz | O que precisa junto |
|---|---|---|
| `move_to_stage` | move o cartao para outra etapa do mesmo funil | `stage`: a chave da etapa |
| `assign_agent` | poe um responsavel no cartao | `agent_id`: o numero da pessoa |
| `create_note` | escreve uma anotacao interna no cartao (autor fica "Sistema") | `note_text` |
| `apply_checklist_template` | aplica um modelo de checklist como um bloco novo no cartao | `template_id`: o numero do modelo cadastrado na conta |
| `duplicate_item` | cria uma copia do cartao em outro funil e etapa | `funnel_id` e `stage`; opcional `distribute_agents` |
| `send_webhook` | manda os dados do cartao para um endereco do cliente | `webhook_url` |
| `notify_team` | **so escreve uma linha de registro interno - nao avisa ninguem** | `message` |
| `update_checklist` | marca ou renomeia tarefas de checklist que ja existem | `checklist_updates`; **a tela nao oferece esta acao** - trate como nao suportada |

A tela do funil oferece sete: as oito acima menos `update_checklist`.

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
- **`enabled`** - tem que nascer `true`. Ver a secao 6.
- **`trigger_value`** - a **chave** da etapa, nunca o nome visivel. Pegue as chaves em `lionchat_funnels_show`.
- **`action_config`** - o bloco de configuracao daquela acao. Faltando o campo obrigatorio, a automacao e descartada no proximo salvamento pela tela.

## 5. Como gravar sem apagar o que ja existe

**O bloco de configuracoes do funil e substituido inteiro.** Gravar so as automacoes apaga as pessoas, os times e as metas - e o funil que ficar com a lista de pessoas vazia vira ABERTO a todos, sem nenhum erro aparecer.

Sequencia obrigatoria:

1. `lionchat_funnels_show` - leia as configuracoes atuais.
2. Monte o bloco COMPLETO: pessoas + times + metas + automacoes (as que ja existiam mais as novas).
3. `lionchat_funnels_update` com o bloco inteiro.
4. `lionchat_funnels_show` de novo e confira campo a campo que tudo voltou.

O mesmo vale para as etapas: gravar a lista de etapas sem repetir uma delas EXCLUI essa etapa. Se a etapa tiver cartoes, o sistema recusa e diz que ela ainda tem cartoes; se estiver vazia, some calada e a chave dela nunca mais pode ser reaproveitada em automacao.

## 6. Combinacoes proibidas e combinacoes que a tela esconde

- **Automacao desligada e DESCARTADA.** Gravar com `enabled: false` "para ativar depois" faz a automacao sumir do banco no proximo salvamento do funil pela tela, sem aviso. Grave ligada ou nao grave.
- **Automacao sem valor de gatilho e descartada** pelo mesmo caminho.
- **Automacao com configuracao incompleta e descartada**: mover sem etapa, dar dono sem pessoa, duplicar sem funil ou sem etapa, avisar sistema externo sem endereco, aplicar checklist sem modelo.
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

## 8. Regra da conta com evento de Kanban

E o motor geral de regras da conta (`lionchat_automation_rules_create`). Ele serve para os dois lados.

**Eventos de Kanban que ele escuta - sao exatamente dois:**

- `kanban_item_created`
- `kanban_item_stage_changed`

**Nao existe** `kanban_item_moved`, nem evento de ganho, nem de perda no motor da conta. Ganho e perda so tem gatilho na automacao de etapa e no fluxo.

**Acoes que mexem no cartao** - disponiveis a partir de QUALQUER evento da conta (mensagem chegou, conversa criada, etiqueta aplicada, conversa resolvida):

`create_kanban_item`, `move_kanban_item_to_stage`, `assign_agent_to_kanban_item`, `add_note_to_kanban_item`, `start_kanban_item_timer`, `stop_kanban_item_timer`, `set_kanban_item_status`.

Detalhes que evitam surpresa:

- **Criar cartao e inteligente**: se ja existe um cartao ABERTO daquela conversa no mesmo funil, ele MOVE o existente em vez de criar outro (a menos que voce marque que duplicatas sao permitidas). Cartao ja Ganho ou Perdido nao conta como existente - a pessoa que volta ganha um cartao NOVO e o fechado fica parado como historico. Isso e proposital.
- **Mover sem cartao nenhum nao faz nada** e o passo aparece como bem-sucedido. Existe uma opcao de criar se nao existir - use-a quando o cartao pode ainda nao ter nascido.
- O titulo do cartao criado aceita variaveis, por exemplo o nome do contato.
- E aqui, somando a acao de enviar mensagem, que sai o aviso imediato para o cliente. Para esperar horas ou dias, e fluxo.

## 9. Fluxo (FlowBuilder) com gatilho de cartao

O fluxo e o unico caminho que conversa em varios passos, espera resposta e espera muito tempo.

**Gatilhos de cartao:** `card_created`, `card_moved`, `card_won`, `card_lost`, `card_attribute_changed`.

**Acoes de cartao dentro do fluxo:** criar cartao, mover de etapa, **marcar Ganho, marcar Perdido ou
reabrir**, dar responsavel, escrever anotacao, acrescentar checklist, **acrescentar oferta** e gravar
campo do cartao. E o unico caminho automatico, alem da macro e da regra da conta, que fecha o cartao.

**Desvios (condicoes):** existe cartao, cartao esta na etapa X, cartao ganho, cartao perdido. Cuidado: ganho e perdido olham o ESTADO do cartao, nunca a coluna em que ele esta.

Para disparar um fluxo a mao a partir do cartao existe `lionchat_kanban_items_start_flow`. **Ele faz o fluxo FALAR com o cliente - so use com autorizacao para aquele disparo especifico.** Tres condicoes precisam valer: o cartao tem conversa ligada; o fluxo esta ativo e e de conversa; e o fluxo tem pelo menos um gatilho de cartao. Se ja existir uma execucao daquele fluxo naquela conversa, o pedido e recusado.

## 10. Macro: o botao de um clique para o atendente

E o unico caminho MANUAL: o atendente clica a macro dentro da conversa e o cartao anda. E a resposta certa para "quero um botao para marcar como Proposta Enviada".

Acoes de cartao que a macro aceita: criar cartao, mover de etapa, dar responsavel, escrever anotacao, ligar e desligar o cronometro, e definir o estado (ganho, perdido, aberto).

Ferramentas: `lionchat_macros_list`, `lionchat_macros_create`.

## 11. Agente de IA mexendo no cartao

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

## 12. O que NAO existe (responda nao em vez de inventar)

| Pedido comum | Resposta honesta |
|---|---|
| "me avisa quando o cartao ficar 5 dias parado" | nao existe gatilho de cartao parado. O que existe e a medicao "cartoes parados" no relatorio do funil, e o aviso para sistema externo |
| "me notifica no sininho quando cair um cartao para mim" | nao existe notificacao de Kanban no sininho |
| "manda mensagem quando o cartao entrar na etapa" | nao pela automacao de etapa. Use regra da conta, fluxo, macro ou o agente de IA |
| "o modelo de mensagem da etapa dispara sozinho quando o cartao chega" | nao. O modelo de mensagem por etapa so sai por acao manual, no menu do cartao |
| "automacao quando o cartao sair da etapa" | nao existe. So chegada |
| "automacao quando o cartao for excluido" | nao existe |
| "apagar varios cartoes de uma vez" | nao existe. A tela apaga um a um |
| "duplicar o funil inteiro" | a TELA tem o botao (icone de copiar na lista de funis), mas as automacoes da copia ficam apontando para as chaves de etapa do funil ORIGINAL e nao disparam - refaca-as. Pelo conector nao ha ferramenta: e ler um funil e criar outro, o que e criacao e exige confirmacao |
