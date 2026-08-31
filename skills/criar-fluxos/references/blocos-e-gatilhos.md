# Catalogo de blocos, gatilhos, acoes e saidas

Fonte da verdade em tempo real: chame `lionchat_flows_schema_reference` e leia
`lionchat://docs/flowbuilder-design-guide`. Este arquivo e o resumo estavel, com o que costuma
falhar em silencio. Onde os dois divergirem, vale o que o conector devolver.

## Indice

1. Como um fluxo e guardado
2. Bloco Inicio e o catalogo de gatilhos
3. Bloco Enviar mensagem
4. Bloco Aguardar resposta
5. Bloco Condicao
6. Bloco Acoes
7. Bloco Espera
8. Bloco Randomizador
9. Bloco Requisicao (chamar sistema de fora)
10. Bloco IA
11. Bloco Gestao de Grupos
12. Bloco Definir variavel
13. Bloco Fim (so em ferramenta da IA)
14. Bloco Nota adesiva
15. Os fios: tabela de saidas por bloco
16. Condicoes de saida do fluxo
17. Regras por canal

---

## 1. Como um fluxo e guardado

O desenho vai no campo `flow_data`, com tres partes:

- `nodes` - a lista de blocos. Cada bloco precisa de `id` unico, `type`, `position` com x e y, e
  `data` com a configuracao daquele tipo.
- `edges` - a lista de fios. Cada fio precisa de `id`, `source` (bloco de onde sai), `target`
  (bloco para onde vai) e `sourceHandle` (o NOME da saida de onde ele sai).
- `exit_conditions` - as regras que tiram a pessoa do fluxo (secao 16). E irma de nodes e edges,
  nao e um bloco.

Layout: bloco Inicio em x 50, y 300; some 320 no x entre blocos em sequencia; separe ramos irmaos
por 150 a 180 no y. **Dois blocos na mesma posicao empilham** e o cliente abre o fluxo e nao
entende nada. Teto do desenho inteiro: 2 MB.

Exatamente **um** bloco Inicio por fluxo. Fio que volta para o proprio bloco e recusado ao salvar.

---

## 2. Bloco Inicio (`start`) e o catalogo de gatilhos

Nao faz nada sozinho: guarda a lista de eventos que fazem o fluxo comecar.

A lista vai em **`data.items`**, e cada item e `{ key, config }`. Escreva SEMPRE nesse formato. Ha
tambem o formato antigo `data.triggers` (usado por integracoes velhas): hoje o proprio sistema o
converte para `data.items` ao salvar, mas gatilho escrito com nome errado DENTRO de `data.items` nao
e convertido nem corrigido.

| Gatilho (`key`) | Quando dispara | `config` |
|---|---|---|
| `conversation_created` | Nasce uma conversa nova | `event_filter: { match_mode, keywords: [] }` |
| `message_received` | Qualquer mensagem do cliente com as palavras escolhidas | `keywords: []` (min 3 letras cada), `match_type: contains ou exact` |
| `message_sent` | Sai uma mensagem (atendente, celular ou a IA). Nota interna nao conta | `keywords: []` opcional, `match_type` |
| `conversation_resolved` | A conversa e encerrada | sem config |
| `conversation_reopened` | Conversa encerrada volta a abrir | `event_filter: { match_mode, keywords }` |
| `team_changed` | Muda a equipe responsavel | `team_ids: []` (textos; vazio = qualquer) |
| `assignee_changed` | Muda o atendente responsavel | `agent_ids: []` (textos) |
| `label_added` / `label_removed` | Etiqueta posta ou tirada da conversa | `label_names: []` - use `label_names`, nunca `label` |
| `sla_missed` | Um prazo de SLA estoura | `sla_policy_id`, `sla_type: any/frt/nrt/rt` |
| `card_created` / `card_moved` | Card criado ou movido de etapa no funil | `funnel_ids: ["37"]` (TEXTO), `funnel_stages: ["37:chave_interna"]` |
| `card_won` / `card_lost` | Card marcado como ganho ou perdido | `funnel_ids: []` (textos) |
| `conversation_attribute_changed` / `card_attribute_changed` | Um campo personalizado da conversa ou do card vira o valor combinado | `logic`, `rules: [{ attr_key, operator, value }]` |
| `contact_attribute_changed` | Campo personalizado do CONTATO muda | `logic`, `rules: [{ attrSource: contact, attr_key, operator, value }]` |
| `date_trigger` | Uma data guardada na ficha chega (aniversario, exame, vencimento) | `attr_key`, `offset_direction`, `offset_days`, `repeat_yearly`, `send_time`, e outros |
| `webhook_received` | Chega evento de integracao (pagamento, formulario, agenda, lead) | `integration_id` quando for webhook proprio do fluxo |
| `campaign_trigger` | Nao e evento: e a AUTORIZACAO para o fluxo aparecer na Campanha de Fluxo | sem config |
| `page_track` | O contato visitou uma pagina ou disparou um evento no site | `track_type`, `urls`, `event_names`, `cooldown_hours` |
| `lead_form_completed` / `lead_form_milestone` / `lead_form_abandoned` | Formulario publico preenchido, marco atingido, abandonado | conforme o formulario |
| `booking_created` / `booking_cancelled` / `booking_rescheduled` / `booking_completed` | Agendamento da Agenda do LionChat | `booking_event_type_ids: []`, `agent_ids: []`, `create_conversation` |
| `group_participant_joined` / `group_participant_left` | Alguem entrou ou saiu de um grupo de WhatsApp | so em fluxo de caixa WhatsApp por QR Code |

Escreva o nome exato da coluna `key`. **Nao existe** o gatilho `cron` (para data, use
`date_trigger`), e `webhook` sozinho nao vale: o certo e `webhook_received`. Chave desconhecida em
`data.items` = fluxo ativo que nunca dispara, sem erro nenhum.

Numeros de funil e de etapa vao como TEXTO. A etapa e a chave interna, no formato
`"37:chave_interna_da_etapa"`, nunca o nome que aparece na tela.

**Iniciar o fluxo a mao** pela lateral da conversa nao precisa de gatilho nenhum: a barra aceita
qualquer fluxo ativo do tipo conversa. Pela ferramenta e
`lionchat_conversations_flow_sessions_create`.

Saida do Inicio: `success`.

---

## 3. Bloco Enviar mensagem (`send_message`)

Manda um ou varios baloes em sequencia. A lista vai em **`data.messageItems`**. A chave `items`
ainda e lida como reserva para fluxo velho, mas escreva sempre em `messageItems`: e a unica que a
tela grava, e `messageItems` presente porem VAZIO deixa o bloco mudo.

| Tipo de item | Para que serve | Campos |
|---|---|---|
| `text` | Balao de texto, aceita variaveis | `content` |
| `text` com botoes | Vira menu clicavel; **para o fluxo e espera** | `buttons_enabled`, `buttons: [{title, value}]`, `buttons_timeout`, `buttons_timeout_unit` (minutes, hours, days), `buttons_timeout_action` (advance ou remind), `buttons_reminder_text` |
| `template` | Modelo aprovado da Meta - obrigatorio para falar primeiro ou fora da janela de 24h | `template_id`, `template_params`, `template_language`, `template_buttons` |
| `canned_response` | Manda uma resposta pronta ja cadastrada | `canned_response_id` |
| `delay` | Pausa entre um balao e outro | `duration_seconds` (0 a 30) |
| `user_input` | Para no meio do bloco e guarda a proxima resposta numa variavel | `variable` |
| `image`, `video`, `audio`, `file`, `document`, `url_media` | Foto, video, audio, documento ou midia por link | `attachment_url` / `file_url` / `url`, `blob_signed_id`, `caption` |

Mencao em grupo (so em conversa de grupo): `mentionMode` com `sender` (marca quem falou), `all`
(marca todo mundo) ou `custom` com `mentionNumbers`. Teto de 50 pessoas, que e trava contra
banimento do numero.

Saidas: `success` (so quando NAO ha botoes), `error`, uma `button_<valor>` por botao,
`no_response` (o cliente digitou texto livre em vez de clicar - na tela aparece como "Outros"),
`no_reply_timeout` (ninguem clicou dentro do prazo, so existe com prazo configurado) e
`window_closed` (janela de 24h fechada, so em caixa WhatsApp Oficial).

`no_response` e `no_reply_timeout` sao coisas DIFERENTES: um e "respondeu outra coisa", o outro e
"sumiu". Ligue os dois.

No modo `remind`, o sistema reenvia os mesmos botoes uma vez, re-arma a espera e so depois segue o
caminho de tempo esgotado. Em caixa Oficial, se a janela de 24h fechou, ele desiste e avanca.

---

## 4. Bloco Aguardar resposta (`wait_response`)

Para o fluxo e espera. **Ele nao pergunta nada sozinho** - ponha um bloco de mensagem antes.

| Campo | Para que serve |
|---|---|
| `validation` | O formato aceito: `any`, `options`, `varied_options`, `regex`, `email`, `phone`, `number`, `cpf`, `cnpj`, `cpf_cnpj`, `date`, `rg`, `profession` |
| `acceptedOptions` | As opcoes fixas quando `validation` e `options`. Cada uma vira uma saida |
| `optionGroups` | Quando a mesma resposta vem de varios jeitos: `[{ id, terms: [], matchType }]`. `matchType` `equals` evita que o termo "1" case dentro de "12" |
| `regexPattern` | Padrao livre quando `validation` e `regex` |
| `invalidMessage` | O que o cliente le quando responde fora do formato. Aceita variaveis |
| `maxRetries` | Quantas vezes ele pode errar. Vazio ou 0 vira 3 |
| `waitTime` + `waitUnit` | Prazo de silencio. Numero INTEIRO + `seconds`, `minutes` (padrao) ou `hours`. **Campo ausente = espera para sempre** |
| `saveTo` | Onde guardar (secao seguinte) |
| `saveVariable` | Nome da variavel quando `saveTo` e `variable` |
| `saveAttrKey` | Qual campo personalizado receber, quando `saveTo` e `contact_attr` ou `conversation_attr` |
| `groupInputsSeconds` | Junta resposta picada em varios baloes antes de validar. 0 = desligado; a tela oferece 10, 15, 20, 25, 30 e 40 |

**Onde guardar (`saveTo`) e a validacao que cada destino exige.** Fora do par certo, o cadastro do
contato desfaz a gravacao em silencio: o passo fica verde e o dado nao muda.

| `saveTo` | Exige `validation` |
|---|---|
| `variable` | qualquer |
| `contact_name` | `any` serve |
| `contact_email` | `email` |
| `contact_phone` | `phone` |
| `contact_cpf` | `cpf` |
| `contact_cnpj` | `cnpj` |
| `contact_document` | `cpf_cnpj` |
| `contact_birthdate` | `date` |
| `contact_rg` | `rg` |
| `contact_profession` | `profession` |
| `contact_attr` / `conversation_attr` | qualquer, mas exige `saveAttrKey` |
| `""` | nao salva - o fluxo avanca do mesmo jeito |

**Nao existem** os destinos `attribute` nem `contact_attribute`. Escrever qualquer outra coisa
descarta a resposta sem avisar.

CPF, CNPJ, RG e data de nascimento **ja preenchidos nao sao sobrescritos** pelo fluxo.

Se o campo personalizado escolhido tiver um tipo (numero, data, lista, moeda), a resposta e
conferida tambem contra o tipo, mesmo com a validacao do bloco em "qualquer resposta" - e o cliente
acha que e defeito.

Saidas: `success` (quando nao ha opcoes), uma `option_<valor>` por opcao ou por grupo, `timeout`
(ficou em silencio) e `retries_exhausted` (errou o formato demais). Sem fio no `timeout`, a sessao
e ENCERRADA quando o prazo estoura. `retries_exhausted` sem fio proprio cai no fio do `timeout`;
sem nenhum dos dois, encerra.

---

## 5. Bloco Condicao (`condition`)

Avalia as condicoes **em ordem** e para na primeira que bate. Cada condicao vira uma saida
`cond_0`, `cond_1`, e assim por diante, **pela posicao na lista** (nunca use o id da condicao como
nome do fio). Sempre existe a saida `default`.

Cada condicao pode ter uma regra so, ou agrupar ate 10 regras com E / OU (`logic: and` ou `or`).

**Campo (`field`)**: normalmente vai entre chaves duplas, por exemplo
`{{contact.custom_attribute.plano}}`. Sem as chaves, o sistema compara o texto do caminho com o
valor e nunca casa. **Atributo e sempre no singular**: `custom_attribute`, nunca
`custom_attributes`.

**Excecao**: as condicoes prontas do catalogo usam um campo interno proprio, que comeca com risco
baixo, e ali quem decide e o operador. Pares corretos - escreva SEMPRE os dois juntos, senao a
regra roda certa e aparece errada na tela do cliente:

| Condicao | `field` | `operator` |
|---|---|---|
| Tem atendente / nao tem / nao e o atendente X | `_agent_check` | `conversation_has_agent`, `conversation_no_agent`, `conversation_not_agent` |
| Tem AI Agente / nao tem / nao e o assistente X | `_ai_agent_check` | `conversation_has_ai_agent`, `conversation_no_ai_agent`, `conversation_not_ai_agent` |
| Esta na equipe X | `{{conversation.team_id}}` | `equal` |
| Sem equipe | `_team_check` | `conversation_no_team` |
| Status da conversa | `{{conversation.status}}` | `equal` (nasce com o valor `open`) |
| Da para responder agora | `_can_reply` | `can_reply`, `can_reply_closed` |
| Conversa tem / nao tem etiqueta | `{{conversation.label}}` | `conversation_has_label`, `conversation_no_label` |
| Contato tem / nao tem etiqueta | `{{contact.label}}` | `contact_has_label`, `contact_no_label` |
| Situacao do SLA | `_sla_check` | `sla_check` (valores `frt_breached`, `frt_ok`, `nrt_breached`, `nrt_ok`, `rt_breached`, `rt_ok`, `has_sla`, `no_sla`) |
| Existe card no funil | `_kanban_check` | `kanban_exists` |
| Card esta na etapa X | `_kanban_stage` | `kanban_in_stage` |
| Card ganho / perdido | `_kanban_status` | `kanban_won`, `kanban_lost` |
| Horario comercial | `_business_hours` | `business_hours`, `outside_business_hours` |
| Visitou pagina / disparou evento no site | `_pagetrack` | `pagetrack_visited`, `pagetrack_event` |
| **O que o cliente respondeu** | `{{last_response}}` | `equal_any` e as variacoes com `_any` |
| **O que nos respondemos por ultimo** | `{{last_agent_response}}` | `equal_any` e as variacoes com `_any` |

As duas ultimas sao as mais usadas no dia a dia ("se o cliente escreveu X, vai por aqui").

Operadores de texto: `equal`, `not_equal`, `contains`, `not_contains`, `starts_with`, `ends_with`,
`is_empty`, `is_not_empty`, `has_length`, `regex`, `is_number`, `is_letter`, `is_email`,
`is_phone`. De numero e data: `greater_than`, `less_than`, `number_range` (formato "min-max").
Contra lista (usam `values`): `equal_any`, `not_equal_any`, `contains_any`, `not_contains_any`,
`starts_with_any`, `ends_with_any`.

**Regra sem valor preenchido e PULADA em silencio.** Se todas as regras de uma saida forem puladas,
aquela saida nunca casa e todo mundo cai no `default`. Os unicos operadores que dispensam valor sao
os de presenca (`is_empty`, `is_not_empty`), os de formato (`is_number`, `is_letter`, `is_email`,
`is_phone`), os de atendente e AI Agente, `conversation_no_team`, `can_reply`, `can_reply_closed`,
`business_hours`, `outside_business_hours`, `kanban_exists`, `kanban_won` e `kanban_lost`.

Ao montar pela ferramenta, mande tambem `valueType: "variable"` nas regras comuns - sem ele a saida
abre vazia no editor e a regra some se alguem salvar pela tela.

Horario comercial usa `days` (0 = domingo ate 6 = sabado), `start_hour`, `end_hour` e `timezone`
(padrao America/Sao_Paulo). Fuso escrito errado cai no padrao sem avisar.

**Nao ponha um bloco de Condicao logo depois de um Aguardar resposta com opcoes** - o proprio
Aguardar resposta ja tem uma saida por opcao.

---

## 6. Bloco Acoes (`action`)

Executa uma ou varias acoes de uma vez, sem falar com o cliente. A lista vai em `data.items`, e
cada item e `{ key, config }` - **e `config`, nunca `params`**.

Saida: so `success`. **Nao existe saida de erro**: acao que falha vira aviso no historico e o fluxo
continua. Fio de erro ligado ali e fio fantasma.

### Conversa

| Chave | O que faz | `config` |
|---|---|---|
| `assign_agent` | Define o atendente responsavel | `agent_id` (aceita variavel; o texto `nil` REMOVE o responsavel) |
| `distribute_agents` | Rodizio simples entre atendentes | `agents: [{agent_id}]`, `dist_id` |
| `assign_team` | Define a equipe responsavel | `team_id` (aceita variavel) |
| `change_status` | Abre, encerra ou adia | `status: open, resolved, snoozed ou pending`; com `snoozed`, informe tambem `snooze_option` (quando acorda) |
| `change_priority` | Prioridade | `priority: low, medium, high, urgent` ou `none` para tirar |
| `add_sla` | Aplica politica de prazo (nao troca se ja houver uma) | `sla_policy_id` |
| `mute_conversation` | Silencia a conversa | vazio |
| `mark_unread` | Deixa como nao lida para o time (use apos transferir) | vazio |
| `add_private_note` | Recado interno; o cliente nao ve. Aceita variaveis e mencao | `text` |
| `send_email_transcript` | Manda a conversa por e-mail | `email` |
| `add_conversation_label` / `remove_conversation_label` | Etiqueta na CONVERSA | `labels: ["slug"]` |
| `add_label` / `remove_label` | Etiqueta no CONTATO | `labels: ["slug"]` |

### Campos e ficha

| Chave | O que faz | `config` |
|---|---|---|
| `update_contact_attribute` / `update_conversation_attribute` / `update_card_attribute` | Grava um campo | `attr_key`, `attr_value`, `attr_op: set, add, sub, mul, div` |
| `update_attribute` | Mesma coisa, com a fonte explicita | `attr_source: contact, conversation ou card`, `attr_key`, `attr_value` |

Campos nativos do contato aceitos em `attr_key`: `contact.name`, `contact.email`, `contact.cpf`,
`contact.cnpj`, `contact.rg`, `contact.passport`, `contact.date_of_birth`, `contact.gender`,
`contact.marital_status`, `contact.profession` e o endereco, com estes nomes exatos:
`contact.address.cep`, `.street` (rua), `.number`, `.complement`, `.neighborhood` (bairro),
`.city`, `.state` (UF) e `.country`. **Telefone e etiqueta nao sao aceitos ali**, e CPF, CNPJ, RG,
passaporte, nascimento, genero e e-mail so preenchem campo VAZIO.

O valor e convertido pelo tipo do campo (sim/nao, decimal, data no formato brasileiro). Se nao
reconhecer, grava o original e deixa aviso amarelo no historico em vez de falhar.

### Funil (Kanban)

| Chave | O que faz | `config` |
|---|---|---|
| `create_kanban_item` | Cria card | `funnel_id`, `funnel_stage` (obrigatorios), `title`, `description`, `allow_duplicates` |
| `move_kanban_stage` | Move o card (funil e etapa sao o DESTINO). Sem card, cria | `funnel_id`, `funnel_stage` |
| `set_won` / `set_lost` / `set_open` | Marca ganho, perdido ou reabre | vazio, mais `funnel_id` se preciso |
| `assign_agent_card` | Responsavel do card | `agent_id`, `mode: add (padrao), replace ou remove_all` |
| `add_card_note` | Anotacao no card | **`text`** - com `content` a acao e pulada em silencio |
| `add_card_checklist` | Aplica um modelo de checklist | `template_id` |
| `add_card_offer` | Adiciona produto ou servico e recalcula o total | `offer_id`, `use_custom_value`, `custom_value` |

Nas acoes de card, `card_source` diz de qual card se trata: `funnel` (padrao, procura pelo funil)
ou `trigger` (o card que iniciou o fluxo).

### Sistema

| Chave | O que faz | `config` |
|---|---|---|
| `assign_captain` | Liga o AI Agente na conversa | `assistant_id` e `proactive` (padrao LIGADO: com ele ligado, a IA fala sozinha em ~2 segundos; desligado, ela assume e espera o cliente) |
| `deactivate_captain` | Tira a IA da conversa | vazio |
| `send_webhook` | Avisa um sistema de fora (so dispara, nao le resposta) | `url`, `method` (GET, POST padrao, PUT, DELETE), `headers`, `body` |
| `start_flow` | Inicia OUTRO fluxo | `flow_id` - apontar para o proprio fluxo e aceito ao salvar e ignorado ao rodar |
| `deactivate_flow` | **PAUSA todos os fluxos daquela conversa**, nao so o atual. E pausa, nao encerramento | vazio |

O interruptor "Responder na hora" (`proactive`) **nao controla a cobranca automatica** da IA depois:
sao coisas diferentes. Avise o cliente que, com ele ligado, a IA comeca a falar quase na hora.

---

## 7. Bloco Espera (`wait`)

| Campo | Valores |
|---|---|
| `waitMode` | `duration` (padrao), `date`, `weekday` |
| `waitDuration` + `waitUnit` | numero + `seconds`, `minutes` (padrao), `hours`, `days` |
| `waitDate` + `waitTime` + `waitTimezone` | "AAAA-MM-DD", "HH:MM", fuso (padrao America/Sao_Paulo) |
| `waitDateMode` / `waitTimeMode` | `fixed` (padrao) ou `variable`. Em `variable` o campo carrega a variavel |
| `waitWeekday` + `waitWeekdayTime` + `waitWeekdayTimeMode` | 0 = domingo ate 6 = sabado, "HH:MM", `fixed` ou `variable` |

Mandar variavel sem marcar o modo como `variable` nao funciona e nao avisa. Data so aceita campo do
tipo Data; hora so do tipo Hora - o tipo "Data e Hora" nao serve.

Saidas: `success` e `error` (a variavel nao virou data ou hora valida).

Em caixa WhatsApp Oficial, esperar 24 horas ou mais fecha a janela: o proximo bloco de mensagem
precisa ser modelo aprovado.

---

## 8. Bloco Randomizador (`randomizer`)

**Divide na proporcao exata, nao sorteia**: 50/50 alterna A, B, A, B; 70/30 da 7 de cada 10
intercalados. Dois modos:

- `mode: branches` (padrao) - teste A/B. `data.branches: [{ id: "A", label, percent: 50 }]`. O campo
  e **`percent`**; `weight` e formato antigo e abre com as porcentagens em branco na tela. Cada
  ramificacao vira uma saida com o proprio id (`A`, `B`, `C`).
- `mode: distribute_agents` - escolhe o atendente da vez e ja atribui a conversa.
  `data.agents: [{ agent_id, percent }]` somando 100. Saida unica: `success`.

Atendente que foi removido da caixa e pulado em silencio.

---

## 9. Bloco Requisicao (`api`)

Chama um sistema de fora e guarda a resposta numa variavel.

| Campo | Para que serve |
|---|---|
| `apiUrl` | O endereco. Obrigatorio. Enderecos internos sao bloqueados por seguranca |
| `apiMethod` | GET (padrao), POST, PUT, PATCH, DELETE |
| `apiHeaders` | `[{ key, value }]`, aceitam variaveis |
| `apiBody` / `apiBodyMode` / `apiBodyFields` | O corpo enviado |
| `apiQueryParams` | Parametros na URL |
| `apiAuthType` + `apiAuthToken` | `bearer`, `basic` ou `api_key` |
| `apiTimeout` | Segundos. Padrao 10, teto 25 |
| `apiResponseVar` | Nome da variavel com a resposta. Padrao `api_response` |

**Nomes crus nao funcionam**: `url`, `method`, `headers`, `body` sao ignorados e o bloco manda um
GET vazio ou erra "URL nao configurada".

Para ler a resposta: `{{api_response.campo}}` - **nao existe** `.payload` no meio. O codigo HTTP
fica em `{{api_response_status}}`. Para token guardado na conta:
`{{account.custom_attribute.NOME}}` - variavel marcada como segredo so resolve DENTRO deste bloco.
**Nao existe** `{{env.X}}`.

Saidas: `success` (resposta 2xx ou 3xx) e `error` (4xx, 5xx, tempo esgotado ou URL invalida).
**Sem fio no `error`, a pessoa para ali com a execucao em vermelho e nada retenta.**

Para experimentar a chamada antes de montar o resto, existe a ferramenta de testar requisicao
(`lionchat_flows_create_3`, que apesar do nome bate na URL e devolve a resposta).

---

## 10. Bloco IA (`ai`)

| Campo | Para que serve |
|---|---|
| `aiMode` | `generate` (padrao), `intent`, `sentiment`, `extract`, `custom` |
| `aiAssistantId` | Qual AI Agente da conta usar. **Obrigatorio** em generate, intent, sentiment e extract - sem ele o bloco sai pela saida de erro |
| `aiPrompt` | A instrucao daquele bloco. Aceita variaveis |
| `aiContextMessages` | Quantas mensagens recentes a IA enxerga: 0, 1, 3, 5 (padrao), 10, 25, 50, 75, 100. A chave e `aiContextMessages` - `contextMessages` e descartada em silencio |
| `aiResponseVar` | Onde guardar a resposta. Padrao `ai_response` |
| `aiIntents` | As intencoes possiveis: `[{ name: "compra" }]`, array de OBJETOS. Tambem grava `{{ai_intent}}` |
| `aiExtractParams` | Os dados a extrair: `[{ name, description }]`. Cada um vira uma variavel |
| `aiInputText` | O texto que a IA deve processar. **Obrigatorio no modo `custom`** (vazio sai pela saida de erro); ali o `aiPrompt` vira a instrucao e os dois sao juntados |

Saidas: `success` e `error` nos modos generate, custom e extract. No modo `intent`, uma
`intent_<nome>` por intencao (com o nome ORIGINAL, acento e tudo), mais `no_intent` e `error`. No
modo `sentiment`: `positive`, `negative`, `neutral` e `error`.

No modo `custom`, se a IA sinalizar que quer passar para um humano ou encerrar a conversa, o bloco
sai pelo `error` - use esse caminho para transferir para atendimento humano.

Da para experimentar o bloco com uma conversa real, sem enviar nada, com
`lionchat_flows_test_ai_node`. A IA roda de verdade (tem custo).

---

## 11. Bloco Gestao de Grupos (`update_group`)

Uma operacao por bloco, em grupo de WhatsApp. Exige que a conta tenha caixa WhatsApp por QR Code.

`groupOperation` aceita: `create`, `find_by_id`, `find_by_name`, `update_subject`,
`update_description`, `update_picture`, `settings`, `leave`, `list_participants`,
`add_participants`, `remove_participants`, `promote_admin`, `demote_admin`, `get_invite`,
`revoke_invite`, `send_invite`, `send_message`.

- `groupInboxId` - por qual numero o bloco fala. **Obrigatorio** sempre que as caixas do fluxo nao
  forem TODAS de WhatsApp por QR Code (fluxo sem caixa nenhuma tambem exige). Quando o fluxo ja
  roda inteiro em QR Code, ele herda o numero da conversa e o campo e opcional. O numero escolhido
  precisa ser uma caixa de WhatsApp por QR Code DA CONTA - o salvar recusa qualquer outra.
- `groupTargetId` - qual grupo. Obrigatorio sempre que a conversa em que o bloco roda nao for a do
  proprio grupo; so `create` e `find_by_name` dispensam alvo.
- `groupResponseVar` - nome da variavel com o resultado (padrao `grupo`). Depois:
  `{{grupo.id}}`, `{{grupo.participants_count}}`, `{{grupo.invite_link}}`, `{{grupo.not_added}}`.
- Criacao: `groupName`, `groupParticipants`, `groupInitialAdmins`, `groupAttributes`.
- Permissoes (`settings`): `infoAdminOnly`, `messagesAdminOnly` e `membersCanAddNewMember`.
  **Atencao: a terceira e invertida** - `true` quer dizer que TODOS podem adicionar, enquanto nas
  duas primeiras `true` quer dizer SO admin. Confundir libera o grupo achando que esta trancando.
- Convite automatico (`groupInviteOnFailure`) nasce desligado e exige `groupInviteMessage`. Cada
  convite entregue CRIA contato e conversa no painel e acorda tudo que escuta conversa nova.

Teto de **20 participantes por execucao** - e trava contra banimento do numero, nao limite tecnico.

Saidas: `success`, `error` e, nas operacoes de participante e no `create`, `partial` (rodou, mas
nem todo mundo entrou ou saiu). **Sem fio no `partial`, quem ficou de fora passa em silencio e o
bloco reporta sucesso.**

---

## 12. Bloco Definir variavel (`set_variable`)

`data.variables: [{ name, value }]`. O valor aceita variaveis e formulas.

Nome vazio ou comecando com risco baixo e **ignorado** - esse prefixo e reservado do sistema.

---

## 13. Bloco Fim (`end`) - so em ferramenta da IA

Define o que a ferramenta devolve para o AI Agente. E terminal, sem saidas.

- `mode`: `template` (padrao - texto que a IA trata como instrucao a cumprir no turno) ou `auto`
  (dado estruturado). **Nao existe o modo `structured`.**
- `template`: o texto devolvido; usa os parametros coletados e as variaveis do fluxo.
- `include_log`: so vale no modo `auto`.

Varios blocos Fim sao permitidos (um proprio para o caminho de erro, por exemplo). Vale o Fim
ALCANCADO.

**Em fluxo de conversa esse bloco e recusado ao salvar** - la o ramo simplesmente acaba no ultimo
bloco.

---

## 14. Bloco Nota adesiva (`note`)

Bilhete colorido no desenho, so para documentar. **Nao executa e nao tem ponto de ligacao** - nunca
ligue um fio nele.

Campos: `title` e `body` (usar `content` grava a nota EM BRANCO), `color` (yellow padrao, teal,
blue, violet, pink, orange, slate), `width` e `height` (minimo 200 por 80).

---

## 15. Os fios: tabela de saidas por bloco

Todo fio precisa de `sourceHandle` com o nome de uma saida que aquele bloco REALMENTE cria naquela
configuracao. Nome inventado (`c1`, `next`, `out`, `option1` sem o risco baixo) vira fio fantasma:
aparece no desenho e nao leva ninguem.

| Bloco | Saidas |
|---|---|
| Inicio | `success` |
| Enviar mensagem | `success` (so sem botoes), `error`, `button_<valor>`, `no_response`, `no_reply_timeout`, `window_closed` |
| Aguardar resposta | `success`, `option_<valor>`, `timeout`, `retries_exhausted` |
| Condicao | `cond_0`, `cond_1`, ... e `default` |
| Acoes | `success` (so isso) |
| Espera | `success`, `error` |
| Randomizador | o id de cada ramificacao (`A`, `B`, `C`) ou `success` no modo de distribuir atendentes |
| Requisicao | `success`, `error` |
| IA | `success` e `error`; no modo intencao, `intent_<nome>`, `no_intent` e `error`; no modo sentimento, `positive`, `negative`, `neutral` e `error` |
| Gestao de Grupos | `success`, `error`, `partial` |
| Fim, Nota adesiva | nenhuma |

**Saidas condicionais** (so existem sob a configuracao certa): `no_reply_timeout` exige prazo nos
botoes; `window_closed` so existe em caixa WhatsApp Oficial; `button_<valor>` so se o botao existe;
`option_<valor>` so se a opcao existe; `cond_N` some quando a condicao sai da lista;
`intent_<nome>` so se a intencao existe; `partial` so nas operacoes de participante e no criar
grupo.

Duas pontes que ainda valem: `option_X`, `button_X` e `no_response` sem fio proprio caem no
`success` SE ele existir; e o `partial` do bloco de grupo sem fio cai no `success`. Fora disso,
**saida sem fio termina o fluxo**.

---

## 16. Condicoes de saida do fluxo (`exit_conditions`)

Regras do fluxo inteiro, conferidas antes de cada bloco. Se qualquer uma bater, a pessoa sai na
hora e a execucao fica marcada como concluida.

| Tipo | Quando tira a pessoa |
|---|---|
| `label_added` / `label_removed` | O contato ganha ou perde uma etiqueta (`value` = nome da etiqueta) |
| `conversation_resolved` | A conversa e encerrada |
| `agent_assigned` / `team_assigned` | Um atendente ou equipe assume (`value` opcional; vazio = qualquer) |
| `kanban_won` / `kanban_lost` | O card vira ganho ou perdido (`value` = funil, opcional) |
| `attribute_condition` | Um campo vira determinado valor: `logic`, `rules: [{ attr_key, attrSource, operator, value }]` |

E a unica forma de o fluxo parar quando um atendente assume a conversa - ele **nao para sozinho**.

---

## 17. Regras por canal

- **Todas as caixas do fluxo precisam ser do mesmo canal.**
- **WhatsApp Oficial aceita UMA caixa por fluxo** (o modelo aprovado pertence a conta de WhatsApp
  Business, nao ao numero). Fluxo antigo com varias caixas continua salvavel; a regra so e cobrada
  quando as caixas mudam.
- **Janela de 24 horas: so no WhatsApp Oficial.** Texto livre, resposta pronta e anexo conferem a
  janela; modelo aprovado passa sempre. Fechada, sai por `window_closed` se houver fio; sem fio,
  cai no caminho de erro. No WhatsApp por QR Code nao ha janela.
- **Gatilhos de entrou/saiu do grupo**: so em fluxo cujas caixas sao todas WhatsApp por QR Code.
- **Bloco de Gestao de Grupos**: roda em fluxo de qualquer canal, desde que a conta tenha uma caixa
  WhatsApp por QR Code. Conta sem nenhuma: o bloco nem aparece.
- **Fluxo de grupo nao tem** os gatilhos e condicoes de navegacao no site nem o gatilho de data
  (grupo nao tem um contato unico).
- Quando muita gente sai de uma espera ao mesmo tempo, o sistema manda UM POR VEZ e o intervalo
  muda por canal: WhatsApp por QR Code, 40 segundos entre cada (risco de banimento e real);
  WhatsApp Oficial, 5 segundos; os demais canais, 2 segundos.
