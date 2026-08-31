# Acoes da automacao e da macro

Indice:

1. A regra de ouro dos parametros
2. As 31 acoes da automacao (formato exato)
3. O que a macro tem e o que ela nao tem
4. As 10 acoes que os gatilhos de card realmente executam
5. O menu de acoes MUDA conforme o gatilho e a caixa
6. Variaveis que a automacao preenche de verdade
7. A "Caixa de Envio" redireciona, nao filtra
8. Onde a acao entrega quando o contato tem varias conversas

---

## 1. A regra de ouro dos parametros

- `action_params` e **sempre uma lista**, mesmo quando tem so um item. O sistema le o primeiro item.
- O nome da acao (`action_name`) e conferido na hora de salvar: nome fora da lista devolve erro.
- **O conteudo dos parametros NAO e conferido por ninguem.** Chave errada, formato errado ou objeto
  solto no lugar de lista salva com sucesso e a acao vira um nada silencioso na hora de rodar.
- Por isso: se voce nao souber o formato de uma acao, **leia uma regra que ja funciona com
  `lionchat_automation_rules_show` e copie** — nunca invente nome de chave.
- As acoes rodam de cima para baixo. Se uma estourar, as seguintes continuam.

---

## 2. As 31 acoes da automacao (formato exato)

### Falar com o cliente

| Nome na tela | Nome tecnico | Parametros |
|---|---|---|
| Enviar Mensagem | `send_message` | `["texto"]` — aceita variaveis (secao 6). Sai como robo, sem nome de atendente |
| Adicionar uma Nota Privada | `add_private_note` | `["texto"]` — o cliente nao ve |
| Enviar Resposta Pronta | `send_canned_response` | `[numero_da_resposta_pronta]` — dispara a resposta inteira, com todos os blocos e intervalos dela |
| Enviar Template WhatsApp | `send_whatsapp_template` | `[{name, id, language, category, processed_params}]` — ver alerta abaixo |
| Enviar Anexo | `send_attachment` | `[identificador_do_arquivo]` — na hora de rodar, o arquivo precisa estar anexado a propria regra (ver abaixo) |
| Enviar uma transcricao por e-mail | `send_email_transcript` | `["a@b.com,c@d.com"]` — UMA string com virgulas |
| Enviar um e-mail para o Time | `send_email_to_team` | `[{"team_ids": [1,2], "message": "texto"}]` |
| Enviar evento de Webhook | `send_webhook_event` | `["https://..."]` |

**ALERTA — Enviar Template WhatsApp.** As chaves sao `name`, `id`, `language`, `category` e
`processed_params`. **NUNCA** use `template_name`, `template_id`, `template_language` ou
`template_category` — esse e o formato dos blocos de Flow e aqui e ignorado: a regra salva, aparece
na tela e NAO ENVIA NADA, sem erro e sem aviso. Incidente real: 14 horas sem enviar, 9 leads pagos
sem resposta.

- `name` e o nome EXATO do modelo aprovado na Meta e e por ele que o envio acontece.
- `language` tem que bater com o idioma aprovado (normalmente `pt_BR`).
- `processed_params` no formato `{"body": {"1": "{{contact.first_name}}"}}`.
- So funciona em caixa WhatsApp API Oficial. Confira o modelo antes com
  `lionchat_inboxes_whatsapp_templates_list` e exija status aprovado.

**Enviar transcricao por e-mail** depende de um interruptor da conta e do limite de e-mails. Se
estiver desligado, a acao nao faz nada e mesmo assim o passo aparece como sucesso no historico.

**Enviar Anexo — de onde vem o arquivo.** Arquivo que ja esta na internet: mande o endereco dele em
`lionchat_upload_create` e use, como parametro da acao, o identificador que ele devolve; ao salvar a
regra o arquivo fica preso a ela. Arquivo do computador do cliente e so pela tela. Reaproveitar um
arquivo ja anexado aquela regra tambem funciona (o parametro e o numero dele).

### Organizar a conversa

| Nome na tela | Nome tecnico | Parametros |
|---|---|---|
| Atribuir ao Agente | `assign_agent` | `[numero_do_agente]` ou `["nil"]` para tirar o responsavel |
| Atribuir um Time | `assign_team` | `[numero_da_equipe]` ou `["nil"]` / `[0]` para tirar |
| Atribuir AI Agente | `assign_captain_assistant` | `[numero_do_assistente]` ou `[{"assistant_id": 17, "proactive": true}]`. `proactive: false` = assume e espera o cliente falar. `["nil"]` desliga e marca que foi desligado de proposito |
| Adicionar uma Etiqueta | `add_label` | `["etiqueta1", "etiqueta2"]` — **NOMES**, nao numeros |
| Remover uma Etiqueta | `remove_label` | `["etiqueta1"]` |
| Alterar Prioridade | `change_priority` | `["low"]`, `["medium"]`, `["high"]`, `["urgent"]` ou `["nil"]` para limpar |
| Adicionar SLA | `add_sla` | `[numero_da_politica]` — so aplica se a conversa ainda nao tiver uma |
| Resolver Conversa | `resolve_conversation` | `[]` |
| Abrir conversa | `open_conversation` | `[]` |
| Pendenciar conversa | `pending_conversation` | `[]` |
| Adiar Conversa | `snooze_conversation` | `[]` — ver alerta |
| Silenciar Conversa | `mute_conversation` | `[]` |
| (nao aparece na tela) | `change_status` | `["open"]`, `["pending"]`, `["resolved"]` ou `["snoozed"]` |
| Alterar atributo do contato | `update_contact_attribute` | `[{"attribute_key": "chave", "value": "texto ou variavel"}]` |
| Alterar atributo da conversa | `update_conversation_attribute` | idem — mesclado, nao apaga os outros atributos |
| Aguardar | `wait` | `[segundos]` — ver alerta |

**ALERTA — Atribuir ao Agente.** O agente precisa ser **membro da caixa daquela conversa** (ou
administrador da conta) e ter a conta confirmada. Se nao for, a acao e pulada em silencio: a
conversa fica sem responsavel e o historico nao deixa isso claro. Confira antes com
`lionchat_inbox_members_show`. E o motivo mais comum de "atribuir agente nao funcionou".

**ALERTA — Adiar Conversa.** Na tela, adiar sempre pede uma data. Aqui **nao ha data**: a conversa
fica adiada por tempo indeterminado e so volta se alguem reabrir na mao ou o cliente escrever. E um
jeito facil de perder atendimento com uma automacao bem-intencionada.

**ALERTA — Aguardar.** A unidade e **segundos** e o teto e **300 (5 minutos)**; valor maior e
cortado sem avisar. Campo vazio, zero ou ausente **nao pausa**: a regra segue direto, tambem sem
erro. Como ultima acao da lista, nao faz nada. Na hora de retomar, o sistema confere se a regra ainda
existe, se ainda esta ligada e se a conversa ainda existe — se algo mudou, ele simplesmente nao
retoma. Espera de horas ou dias e Flow, nao automacao.

**Nota sobre "Alterar atributo da conversa":** ela nao aparece no menu do gatilho "Acao na conversa",
de proposito — gravar atributo de conversa dispara o proprio gatilho de novo, em cadeia.

### Mexer no card do funil

| Nome na tela | Nome tecnico | Parametros |
|---|---|---|
| Criar Card Kanban | `create_kanban_item` | `[{funnel_id, funnel_stage, allow_duplicates, edit_card_details, title, description, offer_ids, checklist_template_ids}]` |
| Mover Card Kanban para Etapa | `move_kanban_item_to_stage` | `[{funnel_id, funnel_stage}]` |
| Atribuir Agente ao Card Kanban | `assign_agent_to_kanban_item` | `[{funnel_id, agent_id, mode}]` — `mode`: `add` (padrao, soma), `replace`, `remove_all` |
| Adicionar Nota ao Card Kanban | `add_note_to_kanban_item` | `[{funnel_id, text}]` — a chave e `text`, nunca `note`. Aceita variaveis |
| Iniciar Timer do Card Kanban | `start_kanban_item_timer` | `[{funnel_id}]` |
| Parar Timer do Card Kanban | `stop_kanban_item_timer` | `[{funnel_id}]` |
| Kanban: Ganho ou Perda | `set_kanban_item_status` | `[{funnel_id, status}]` — `won`, `lost` ou `open` |

**Sobre Criar Card Kanban:**

- `funnel_id` e `funnel_stage` sao obrigatorios; sem eles a acao e pulada.
- Se ja existir card ABERTO daquela conversa no mesmo funil e `allow_duplicates` for falso, ele
  **move o card existente** para a etapa indicada em vez de criar outro. Card ja marcado como Ganho
  ou Perdido nao conta: nasce card novo.
- `title` e `description` so valem com `edit_card_details: true` e aceitam variaveis. Titulo vazio =
  nome do contato.
- `offer_ids` soma o valor das ofertas no card; cada item de `checklist_template_ids` vira um grupo
  proprio de tarefas.
- **Efeito colateral que precisa ser avisado ao cliente:** o card nasce ja com responsavel. O
  sistema tenta o responsavel da conversa e, se nao houver (ou ele nao tiver acesso ao funil), sorteia
  um agente do funil por rodizio. E se a conversa estiver SEM responsavel, o agente sorteado tambem
  vira o responsavel da conversa. Isso e atribuicao acontecendo sem ninguem ter pedido — pergunte se
  o funil tem rodizio configurado antes de montar "lead novo vira card".
- O card tambem herda a prioridade e os atributos personalizados da conversa.

**As demais acoes de card sao PULADAS se nao houver card** naquele funil, e o passo pode aparecer
como concluido: o unico rastro fica no registro do servidor. Sempre ponha "Criar Card Kanban" antes,
na mesma regra.

---

## 3. O que a macro tem e o que ela nao tem

A macro aceita **28** acoes: as mesmas de cima, com estas diferencas.

**A macro NAO tem:**
- Enviar Template WhatsApp
- Alterar atributo do contato
- Alterar atributo da conversa
- Aguardar

**So a macro tem:**
- `remove_assigned_team` — remover a equipe atribuida
- `assign_agent` com `["self"]` — atribui a conversa a **quem clicou**

**Diferencas de comportamento que precisam ser ditas ao cliente:**

1. **A mensagem da macro sai assinada pelo atendente que clicou** (o cliente ve o nome dele no
   WhatsApp). Na automacao sai como robo. Vale para mensagem, anexo e resposta pronta.
2. **Macro que manda mensagem pode DESLIGAR ou pausar o AI Agente daquela conversa.** O sistema
   entende aquilo como "um humano assumiu". Isso so acontece quando o AI Agente daquela conversa
   esta configurado para parar quando um humano responde; conforme a configuracao dele, ele fica
   calado por alguns minutos ou e desligado de vez. A mensagem da AUTOMACAO nao causa isso (ela e
   reconhecida como automatica). A macro classica de encerramento (resolver + etiqueta + mensagem de
   despedida) tem esse efeito colateral — se o cliente usa AI Agente, avise antes de montar.
3. **A macro roda em segundo plano.** A resposta "ok" da ferramenta de executar nao garante que as
   acoes deram certo — confira o resultado na conversa.
4. **Macro nao tem chave liga/desliga.** Ou ela existe (e aparece para quem tem visibilidade), ou e
   apagada.
5. **Visibilidade:** `personal` (so quem criou) ou `global` (a conta toda). Agente comum sempre grava
   pessoal, mesmo pedindo global — so administrador consegue criar macro global. Macro pessoal de
   outra pessoa nao aparece nem executa para os demais.
6. **Ao executar por ferramenta,** o campo pede o **numero da conversa que aparece no painel**, nao
   um identificador interno.
7. **Criar Card Kanban sem escolher o funil** salva na tela. Hoje a acao e pulada, mas a macro
   continua dizendo ao atendente "executada com sucesso" e o card nao existe.

---

## 4. As 10 acoes que os gatilhos de card realmente executam

Nos gatilhos **Card Kanban Criado** e **Card Kanban Movido** quem executa e um motor diferente. Ele
conhece apenas estas:

| Nome tecnico | O que faz nesse contexto | Parametros |
|---|---|---|
| `send_message` | Manda mensagem nas conversas vinculadas ao card que estejam marcadas para receber automacao | `["texto"]` |
| `add_label` | Poe etiqueta nas conversas vinculadas ao card | `["etiqueta"]` |
| `assign_agent_to_kanban_item` | Responsavel do card | `[numero]` ou `[{id}]` |
| `move_kanban_item_to_stage` | Move o card | `[{funnel_id, funnel_stage}]` |
| `add_note_to_kanban_item` | Nota no card | `["texto"]` — ver alerta |
| `start_kanban_item_timer` | Liga o cronometro | `[]` |
| `stop_kanban_item_timer` | Desliga o cronometro | `[]` |
| `set_kanban_item_status` | Ganho / Perdido / reabrir | `["won"]` ou `[{status: "won"}]` |
| `change_priority` | Prioridade do CARD (nao da conversa) | `["high"]` |
| `send_webhook_event` | Avisa sistema de fora com os dados do card | `["https://..."]` |

**Todas as outras 20 acoes sao ignoradas em silencio** — o menu da tela oferece, o motor descarta e
so fica um aviso no registro do servidor. Isso inclui: Enviar Template WhatsApp, Atribuir AI Agente,
Resolver Conversa, Atribuir ao Agente (da conversa), Criar Card Kanban, Alterar atributo, Aguardar,
Adicionar SLA, Enviar Resposta Pronta, Enviar Anexo, Nota Privada.

**ALERTA — Adicionar Nota ao Card no gatilho de card.** A tela grava `{funnel_id, text}`, mas esse
motor le o parametro cru e nao a chave `text`: a nota sai com o objeto inteiro dentro do texto. A
mesma acao, com o mesmo formato, se comporta diferente conforme o gatilho. Se precisar de nota nesse
gatilho, mande apenas o texto: `["texto da nota"]`.

**ALERTA — Atribuir Agente ao Card no gatilho de card.** Mesma armadilha. A tela grava
`{funnel_id, agent_id}`, mas esse motor procura a chave `id`. Com o formato da tela ele nao acha
ninguem e PULA a acao em silencio. Nesse gatilho, mande `[{"id": 12}]` (ou so o numero: `[12]`).

**Alem disso**, `send_message` e `add_label` so alcancam conversa: se o card tiver conversas
vinculadas, valem as que estiverem marcadas como "recebe automacao" (nenhuma marcada = nada sai);
se o card nao tiver nenhuma vinculacao, vale a conversa de origem dele.

---

## 5. O menu de acoes MUDA conforme o gatilho e a caixa

Nao existe um cardapio unico. O que a tela oferece depende de tres coisas:

1. **SLA ligado na conta.** Se a funcao de SLA nao estiver liberada, "Adicionar SLA" nem aparece —
   na automacao e na macro.
2. **Gatilho Webhook com Caixa de Envio de WhatsApp API Oficial:** "Enviar Mensagem" e "Enviar
   Anexo" **somem** (a Meta recusa texto livre fora da janela de 24 horas). Nesse caso, use modelo
   aprovado ou resposta pronta.
3. **Gatilho Webhook com Caixa de Envio de WhatsApp QR Code ou Twilio:** "Enviar Template WhatsApp"
   **some** (nao existe modelo aprovado nessas caixas) e o seletor de resposta pronta **esconde**
   toda resposta pronta que exija API Oficial — o sistema marca assim, sozinho, qualquer resposta
   pronta com botao ou com botao de link.
4. **Gatilhos de card:** as duas acoes de atributo somem.
5. **Gatilho "Acao na conversa":** "Alterar atributo da conversa" some (evita cadeia).

Isso importa na hora de propor: a automacao mais comum do produto e "pagamento aprovado, manda
mensagem". Em caixa oficial voce **nao** deve propor "Enviar Mensagem"; em caixa QR Code voce **nao**
deve propor modelo do WhatsApp.

---

## 6. Variaveis que a automacao preenche de verdade

A automacao so conhece estes conjuntos de variaveis:

- `{{contact.*}}` — dados do contato (nome, telefone, e-mail...)
- `{{conversation.*}}` — dados da conversa
- `{{inbox.*}}` — dados da caixa de entrada
- `{{agent.*}}` — o **responsavel da conversa** (nao quem criou a regra). Se a conversa nao tiver
  responsavel, sai vazio
- `{{account.*}}` — dados da conta
- `{{agendamento.*}}` — **so** quando a automacao foi disparada por um lembrete de agendamento da
  e-Clinica

**Variavel que a automacao nao conhece sai VAZIA, em silencio** — sem erro e sem aviso. Nao copie
variaveis de outra tela (Flow, campanha, modelo do WhatsApp): cada motor sabe coisas diferentes.

Onde as variaveis funcionam: nota privada, nota do card, titulo e descricao do card, valor de
atributo e no texto da mensagem.

**Se voce montar "Atribuir ao Agente" e logo depois uma mensagem que cita `{{agent.name}}`,** ponha
as duas na ordem certa e, se ainda sair vazia, use um "Aguardar" de poucos segundos entre elas — o
responsavel pode ainda nao ter sido carimbado.

---

## 7. A "Caixa de Envio" redireciona, nao filtra

O campo Caixa de Envio (`inbox_id` na regra) **nao filtra a regra por caixa**. Ele **redireciona as
acoes de mensagem** — mensagem, anexo, resposta pronta, modelo do WhatsApp e nota privada — para
outra caixa: usa a conversa aberta que o contato ja tiver la e, se nao houver nenhuma, cria uma nova
(que nasce como pendente). "Abrir conversa" e "Pendenciar conversa" seguem o mesmo desvio. As demais
acoes (etiqueta, prioridade, resolver, adiar, agente, equipe, card) continuam na conversa original.

- Para **filtrar por caixa**, use a CONDICAO "Caixa de Entrada".
- A tela so mostra o seletor no gatilho Webhook, mas o campo vale para todos os gatilhos quando
  gravado por fora.
- No gatilho Webhook ele e **obrigatorio**: sem ele a execucao falha.
- A descricao da ferramenta de criar regra diz "Restringir a uma caixa de entrada" — **esta errada**.
  O comportamento e o descrito acima.
- Trocar a Caixa de Envio de uma regra que ja existe **nao da pelas ferramentas**, so no painel.

---

## 8. Onde a acao entrega quando o contato tem varias conversas

**Na MACRO**, tres acoes **nao agem necessariamente na conversa aberta na tela**: "Abrir conversa",
"Pendenciar conversa" e `change_status` para aberto ou pendente. Elas vao para a conversa **viva** do
contato naquela caixa (a aberta; nao havendo nenhuma, a mais recente), que pode ser outra. Esse
desvio acontece dentro da MESMA caixa, sem nada configurado. E o que explica "a macro reabriu uma
conversa antiga".

"Resolver", "Adiar" e "Silenciar" continuam na conversa de origem.

**Na AUTOMACAO e diferente**: essas mesmas acoes seguem a regra da Caixa de Envio (secao 7). Sem
Caixa de Envio, elas agem na propria conversa que disparou a regra. Com Caixa de Envio preenchida,
vao para a conversa do contato NAQUELA caixa. Nao ha, na automacao, o desvio automatico para outra
conversa da mesma caixa.
