# Ferramentas do conector, por integracao

**Por que este arquivo existe:** as ferramentas de integracao tem sufixo numerico (`_create_4`,
`_list_11`, `_show_2`) que vem da ordem em que foram geradas. Se uma ferramenta nova entrar antes na
lista, os numeros seguintes mudam. E o Webhook Universal se chama `lionchat_ecommerce_webhooks_*`,
nome que nao tem nada a ver com ele.

**Regra:** confira sempre o CAMINHO da API, nunca so o nome. Se o nome que voce tem em maos nao bate
com o caminho desta tabela, procure na sua lista de ferramentas a que tem o caminho certo. Se nao
achar, diga que a ferramenta nao esta disponivel - nao chute um numero vizinho.

---

## 0. Sempre primeiro

| Ferramenta | Caminho | Para que serve |
|---|---|---|
| `lionchat_integrations_apps_list` | GET `/integrations/apps` | Catalogo das integracoes DESTA conta. A lista ja vem filtrada pelo que a conta pode usar: integracao ausente = nao disponivel. `enabled` diz se ja esta configurada. |
| `lionchat_integrations_apps_show` | GET `/integrations/apps/{id}` | Uma integracao so. Ids: `webhook`, `dashboard_apps`, `openai`, `groq`, `elevenlabs`, `linear`, `notion`, `slack`, `google_translate`, `dyte`, `shopify`, `conta_azul`, `omie`, `eclinica`, `guru`, `hotmart`, `kiwify`, `eduzz`, `monetizze`, `greenn`, `ticto`, `custom_webhooks`, `meta_lead`, `topsend`. |

---

## 1. Webhook Universal (a integracao "qualquer sistema")

Nome da tela: **Webhook Universal**. Caminho da API: `custom_webhook_integrations`.

| Ferramenta | Caminho |
|---|---|
| `lionchat_ecommerce_webhooks_list` | GET `/custom_webhook_integrations` |
| `lionchat_ecommerce_webhooks_create` | POST `/custom_webhook_integrations` |
| `lionchat_ecommerce_webhooks_show` | GET `/custom_webhook_integrations/{id}` |
| `lionchat_ecommerce_webhooks_update` | PUT `/custom_webhook_integrations/{id}` |
| `lionchat_ecommerce_webhooks_destroy` | DELETE `/custom_webhook_integrations/{id}` |
| `lionchat_ecommerce_webhooks_list_1` | GET `/custom_webhook_integrations/{id}/events` (historico) |
| `lionchat_ecommerce_webhooks_create_1` | POST `.../events/{event_id}/retry` (reprocessar um evento) |

Campos aceitos no cadastro: `name` (obrigatorio), `active`, `event_field`, `field_mapping`,
`event_automation_mapping`, `sample_payload`, `known_event_values`. Tudo dentro de
`custom_webhook_integration`.

**NUNCA mande `flow_id` no cadastro.** Ele nao e um campo normal: mandado na CRIACAO, faz nascer o
webhook EMBUTIDO do editor de fluxo - outra coisa, que nao aparece na tela Webhook Universal e ja
nasce com o mapeamento pronto. Na atualizacao ele e ignorado. Para apontar o Webhook Universal a um
fluxo, use `flow_id` DENTRO de `event_automation_mapping`.

**O que a leitura devolve e que ninguem espera:** `sample_payload` volta com o **ultimo webhook
recebido de verdade** (o exemplo congelado do cadastro so entra como reserva), e `source_fields` ja
vem com a lista achatada de caminhos prontos para mapear, no formato `{path, value}`. Use
`source_fields` - nao ache o caminho na mao.

`webhook_url` vem na resposta. Nao tem conferencia previa de duplicata: reprocessar aqui e cego.

---

## 2. Os sete checkouts

Todos tem o mesmo desenho: cadastro, historico e reprocessamento. Campos comuns: `name`,
`active`, `events` (decoracao), `event_automation_mapping`, `inbox_id` (**gravado e nunca lido**) e o
token de conferencia proprio de cada plataforma.

| Plataforma | Caminho da API | Criar | Listar | Ver | Atualizar | Apagar | Historico | Reprocessar | Conferencia previa |
|---|---|---|---|---|---|---|---|---|---|
| Guru | `/guru_webhooks` | `..._create_2` | `..._list_2` | `..._show_1` | `..._update_1` | `..._destroy_1` | `..._list_3` | `..._create_3` | `..._list_4` |
| Hotmart | `/hotmart_webhooks` | `..._create_4` | `..._list_5` | `..._show_2` | `..._update_2` | `..._destroy_2` | `..._list_6` | `..._create_5` | `..._list_7` |
| Kiwify | `/kiwify_webhooks` | `..._create_6` | `..._list_8` | `..._show_3` | `..._update_3` | `..._destroy_3` | `..._list_9` | `..._create_7` | `..._list_10` |
| Ticto | `/ticto_webhooks` | `..._create_8` | `..._list_11` | `..._show_4` | `..._update_4` | `..._destroy_4` | `..._list_12` | `..._create_9` | `..._list_13` |
| Eduzz | `/eduzz_webhooks` | `..._create_10` | `..._list_14` | `..._show_5` | `..._update_5` | `..._destroy_5` | `..._list_15` | `..._create_11` | `..._list_16` |
| Monetizze | `/monetizze_webhooks` | `..._create_12` | `..._list_17` | `..._show_6` | `..._update_6` | `..._destroy_6` | `..._list_18` | `..._create_13` | `..._list_19` |

O prefixo dos seis acima e `lionchat_ecommerce_webhooks`.

A **Greenn** tem nomes legiveis, sem numero:

| Ferramenta | Caminho |
|---|---|
| `lionchat_greenn_webhooks_list` / `_create` / `_show` / `_update` / `_destroy` | `/greenn_webhooks` |
| `lionchat_greenn_webhooks_events` | GET `.../events` |
| `lionchat_greenn_webhooks_retry_event` | POST `.../events/{event_id}/retry` |
| `lionchat_greenn_webhooks_retry_preflight` | GET `.../events/{event_id}/retry_preflight` |

Particularidades:

- **Guru:** exige `webhook_type` (`transaction`, `subscription` ou `eticket`). Um webhook por tipo,
  com endereco proprio para cada.
- **Token de conferencia:** Hotmart usa `hottok`; Kiwify, `kiwify_token`; Ticto, `ticto_token`;
  Greenn, `greenn_token`; Guru, `api_token`.
- **Conferencia previa antes de reprocessar** (`retry_preflight`): devolve `has_duplicate` e
  `duplicate_processed_at`, procurando outro evento com a mesma transacao ja processado. Chame
  SEMPRE antes de reprocessar - e o que impede reenviar boas-vindas para quem ja recebeu.
- O corpo vai dentro do embrulho da plataforma (`greenn_webhook`, `hotmart_webhook`, e assim por
  diante). O conector cuida disso.

---

## 3. Meta Lead Ads (leads de anuncio do Facebook e Instagram)

| Ferramenta | Caminho | Observacao |
|---|---|---|
| `lionchat_meta_lead_list` | GET `/integrations/meta_lead` | Paginas conectadas |
| `lionchat_meta_lead_create` | POST `/integrations/meta_lead` | **Exige o acesso que so o login do Facebook no navegador produz.** Aceita `page_ids` para escolher as paginas - sem isso entram TODAS. |
| `lionchat_meta_lead_destroy` | DELETE `/integrations/meta_lead/{id}` | Uma pagina |
| `lionchat_meta_lead_bulk_destroy` | POST `/integrations/meta_lead/bulk_destroy` | Varias paginas de uma vez, em segundo plano |
| `lionchat_meta_lead_sync_forms` | POST `/integrations/meta_lead/{id}/sync_forms` | Busca os formularios da pagina |
| `lionchat_meta_lead_show` | GET `/integrations/meta_lead_forms/{id}` | Um formulario, com o exemplo e o mapeamento |
| `lionchat_meta_lead_update` | PUT `/integrations/meta_lead_forms/{id}` | So `field_mapping` e `event_automation_mapping`, dentro de `form` |
| `lionchat_meta_lead_create_2` / `_create_3` | POST `.../activate` / `.../deactivate` | Liga e desliga o formulario |
| `lionchat_meta_lead_list_1` | GET `.../events` | Historico de leads recebidos - **este pagina** (25 por pagina) |
| `lionchat_meta_lead_create_4` | POST `.../backfill` | Puxa leads antigos - **cria contato e NAO dispara nada** |
| `lionchat_meta_lead_refresh_sample` | POST `.../refresh_sample` | Atualiza o exemplo usado para montar o de-para |
| `lionchat_meta_lead_ads_token` / `_validate_token` | POST `/integrations/meta_lead/ads_token` / `/validate_token` | Acesso extra que traz o nome da campanha e do anuncio, e conferencia dele |

**Nao existe `webhook_url` aqui.** Nao tem endereco para colar em lugar nenhum.

O mapeamento evento para alvo do formulario aceita `{"automation_id": N, "flow_id": M}` no topo, ou
sob a chave `__all__`. Formulario desligado recebe o lead e joga fora.

---

## 4. Omie ERP

| Ferramenta | Caminho |
|---|---|
| `lionchat_omie_integrations_list` / `_show` | GET `/omie_integrations` |
| `lionchat_omie_integrations_create` | POST `/omie_integrations` - `app_key` + `app_secret`. A integracao nasce `pending` e a conexao e conferida em SEGUNDO PLANO: releia com `..._show` (ou chame `..._test_connection`) antes de dizer que conectou |
| `lionchat_omie_integrations_update` | PATCH - `enabled_categories`, `event_mapping`, `omie_company_name` |
| `lionchat_omie_integrations_test_connection` | POST `.../test_connection` |
| `lionchat_omie_integrations_toggle_category` | POST `.../toggle_category` |
| `lionchat_omie_integrations_events` / `_retry_event` | GET `.../events` / POST `.../events/{event_id}/retry` |
| `lionchat_omie_integrations_disconnect` / `_destroy` | POST `.../disconnect` / DELETE |

`webhook_url` vem na resposta e leva o numero da conta. O mapeamento e agrupado por categoria:
`{"recebimentos": {"pagamento_confirmado": {"automation_id": 12}}}`. Teto de 7 categorias e 30
eventos por categoria. **"So a primeira compra" nao funciona aqui.**

---

## 5. Conta Azul

| Ferramenta | Caminho |
|---|---|
| `lionchat_conta_azul_integrations_create` | POST `/conta_azul_integrations` - nasce pendente |
| `lionchat_conta_azul_integrations_connect` | POST `.../connect` - devolve o link de autorizacao para a pessoa abrir no navegador (vale 15 minutos) |
| `lionchat_conta_azul_integrations_update` | PATCH - `event_mapping` |
| `lionchat_conta_azul_integrations_sync_now` | POST `.../sync_now` - forca a varredura (o normal e de 5 em 5 minutos) |
| `lionchat_conta_azul_integrations_events` / `_retry_event` | historico e reprocessamento |
| `lionchat_conta_azul_integrations_disconnect` / `_destroy` | desconectar e apagar |

**Nao tem `webhook_url`**: aqui nao ha webhook, o LionChat e quem consulta a Conta Azul.

---

## 6. e-Clinica

**Pelo conector e SO LEITURA.** Token, unidades, mapeamento de evento e as reguas de lembrete sao
tela (ver `so-no-painel.md`).

| Ferramenta | Caminho |
|---|---|
| `lionchat_eclinica_integrations_list` / `_show` | a integracao da conta, com situacao, mapeamento, lembretes, `webhook_url` e as UNIDADES (cada uma com endereco proprio) |
| `lionchat_eclinica_integrations_events` / `_retry_event` | historico (este pagina) e reprocessamento |
| `lionchat_eclinica_reminder_history_list` / `_reprocess` | historico dos lembretes disparados e reenvio de um deles |

---

## 7. Conversao para plataforma de anuncio

| Ferramenta | Caminho | Observacao |
|---|---|---|
| `lionchat_meta_pixel_integrations_create` / `_update` / `_list` / `_destroy` | `/meta_pixel_integrations` | `pixel_id` + `access_token`. **Nasce desligado** e liga sozinho depois de um teste com sucesso |
| `lionchat_meta_pixel_integrations_test_event` | POST `.../test_event` | Manda um evento de teste |
| `lionchat_meta_pixel_integrations_reset_test_event_window` | POST | Reabre a janela de teste |
| `lionchat_ga4_integrations_create` / `_update` / `_list` / `_destroy` | `/ga4_integrations` | `measurement_id` + `api_secret`. Nasce ativa |
| `lionchat_ga4_integrations_test_connection` | POST `.../test_connection` | **Unica forma de saber se a credencial presta** |
| `lionchat_google_ads_integrations_list` / `_show` / `_update` / `_destroy` | `/google_ads_integrations` | **Nao tem criar** - a conexao e autorizacao no navegador |
| `lionchat_google_ads_integrations_list_customers` | GET | As contas de anuncio disponiveis |
| `lionchat_google_ads_integrations_list_conversion_actions` | GET | As acoes de conversao para montar o de-para |
| `lionchat_funnels_meta_events_config` | PATCH `/funnels/{id}/meta_events_config` | Qual evento sai em cada ETAPA. **Junta com o que ja existe** - mande so o que muda |
| `lionchat_funnels_meta_capi_events_list` / `_show` / `_retry` | historico do Pixel, por funil |
| `lionchat_funnels_ga4_events_list` / `_show` / `_retry` | historico do Analytics |
| `lionchat_funnels_google_ads_conversions_list` / `_show` / `_retry` | historico do Google Ads |
| `lionchat_kanban_items_meta_capi_fire` / `_ga4_mp_fire` / `_google_ads_fire` | POST no cartao | Dispara a conversao a mao, para testar sem mover cartao |

---

## 8. Google Calendar e Google Contatos

| Ferramenta | Caminho |
|---|---|
| `lionchat_google_calendar_list` | GET `/google_calendar/status` |
| `lionchat_google_calendar_list_1` | GET `/google_calendar/connect` - devolve o link para autorizar no navegador |
| `lionchat_google_calendar_list_2` | GET `/google_calendar/calendars` |
| `lionchat_google_calendar_create` | POST `/google_calendar/select_calendar` |
| `lionchat_google_calendar_create_1` | POST `/google_calendar/toggle_sync` |
| `lionchat_google_calendar_create_2` | POST `/google_calendar/sync_now` |
| `lionchat_google_calendar_list_4` / `_update` | agendas compartilhadas: listar e gravar a escolha |
| `lionchat_google_calendar_reactivate` / `_destroy` | reativar e desconectar |
| `lionchat_google_contacts_status` / `_connect` / `_toggle_sync` / `_backfill` / `_disconnect` | `/google_contacts/...` |

Uma conexao por conta, so administrador. Desconectar apaga as escolhas de agendas compartilhadas e
elas nao voltam ao reconectar.

---

## 9. Avisar sistema de fora, apps e credenciais

| Ferramenta | Caminho | Observacao |
|---|---|---|
| `lionchat_webhooks_list` / `_create` / `_update` / `_destroy` | `/webhooks` | Webhook de saida. `url` (unico por conta), `subscriptions` (lista fechada), `inbox_id` opcional |
| `lionchat_webhooks_deliveries_list` / `_show` | `/webhooks/{id}/deliveries` | Cada tentativa, com situacao, erro e tempo. So leitura |
| `lionchat_dashboard_apps_list` / `_create` / `_show` / `_update` / `_destroy` | `/dashboard_apps` | Painel de Aplicativos. `content` e uma lista de `{url, type: "frame"}`, so `http` ou `https`, pelo menos um |
| `lionchat_integrations_hooks_create` / `_update` / `_show` / `_destroy` | `/integrations/hooks` | Apps por chave: `openai`, `groq`, `elevenlabs`, `google_translate`, `dyte`. **So a chave da OpenAI e conferida antes de salvar** (recusada se o proprio provedor disser que e invalida; se ele estiver fora do ar, salva assim mesmo). As outras quatro salvam sem conferencia nenhuma |
| `lionchat_integrations_slack_list_all_channels` / `_update` / `_destroy` | `/integrations/slack` | Escolher o canal depois de conectado (`reference_id`) |
| `lionchat_integrations_notion_search_pages` / `_import_pages` / `_destroy` | `/integrations/notion` | Trazer paginas para a base de conhecimento da IA |
| `lionchat_integrations_linear_teams` / `_create_issue` / `_link_issue` / `_unlink_issue` / `_search_issue` / `_linked_issues` / `_destroy` | `/integrations/linear` | Tarefas a partir da conversa |
| `lionchat_integrations_shopify_orders` / `_destroy` | `/integrations/shopify` | Ver os pedidos do cliente |
| `lionchat_topsend_campaigns_list` / `_show` / `_create` / `_destroy` | `/topsend_campaigns` | SMS e voz. `test_connection` confere a chave |
| `lionchat_account_variables_list` / `_create` / `_update` / `_show` / `_destroy` | `/account_variables` | Cofre de credenciais da conta. Tipo `secret` guarda criptografado |

**Webhook de saida - avisos aceitos** (lista fechada; nome fora dela derruba o cadastro):
`conversation_created`, `conversation_updated`, `conversation_status_changed`,
`conversation_typing_on`, `conversation_typing_off`, `contact_created`, `contact_updated`,
`message_created`, `message_updated`, `webwidget_triggered`, `inbox_created`, `inbox_updated`.

**TopSend:** `campaign_type` e `SMSSHORT`, `SMSFLASH`, `TVOZ` ou `URA`. Os numeros vao com 13 digitos
sem o sinal de mais (55 + DDD + numero). Voz e URA exigem audio ja na TopSend (`audio_id`) ou um
endereco publico (`audio_url`) - subir arquivo pelo conector nao da. A hora do disparo vai em
`scheduled_at`; ela e repassada a TopSend, que segura e dispara na hora marcada (sem `scheduled_at`,
sai agora). **A chave da TopSend e cadastrada na tela**, nao pelo conector.

---

## 10. Ferramentas de prova (a parte que mais falta)

| Ferramenta | Caminho | Para que serve |
|---|---|---|
| `lionchat_automation_rules_list_1` | GET `/automation_rules/{id}/executions` | Historico de execucao da automacao, ultimas 48 horas. **E aqui que aparece o motivo em texto** de a mensagem nao ter saido |
| `lionchat_flows_executions_list` | GET `/flows/{id}/executions` | Historico de sessoes do fluxo, com filtro por situacao e data |
| `lionchat_contacts_form_entries_list` / `_show` | GET `/contacts/{contact_id}/form_entries` | Por pessoa: tudo que ela preencheu ou que entrou por ela - formulario da conta, formulario de anuncio do Meta, Webhook Universal e webhook embutido de fluxo |
| `lionchat_integrations_waha_check_phone` | GET `/integrations/waha/check_phone` | Confere se um numero tem WhatsApp, usando a caixa de QR Code da conta |
