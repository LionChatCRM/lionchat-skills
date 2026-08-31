# O catalogo de ferramentas: como achar a certa

Indice:

1. Como as ferramentas se chamam
2. O sufixo numerico e a armadilha mais cara do catalogo
3. As 76 areas, com o slug e a contagem
4. As ferramentas que nao vem do catalogo (embutidas no conector)
5. Os 12 documentos de referencia que voce pode pedir
6. Os 9 roteiros prontos
7. Ensaios: como ver o resultado sem criar nada
8. As 17 ferramentas que exigem o OK do cliente
9. O que a resposta esconde de voce

---

## 1. Como as ferramentas se chamam

O padrao e `lionchat_<area>_<acao>`. As acoes mais comuns sao: `list` (listar), `show` (ver um),
`create` (criar), `update` (alterar), `destroy` (excluir), `filter` (filtrar com criterio),
`search` (buscar por texto). Existem outras (`toggle`, `move`, `execute`, `retry`, `import`...),
entao a lista acima nao esgota o catalogo.

Sao 850 ferramentas hoje. **Antes de dizer que uma funcao nao existe, pesquise pelo prefixo da
area.** Alguns clientes de IA (o ChatGPT em especial) nao carregam o catalogo inteiro de uma vez.

Cada ferramenta traz na ficha: um titulo em portugues, o que ela devolve, casos de uso e as
ferramentas irmas. **Leia o titulo, sempre.**

---

## 2. O sufixo numerico e a armadilha mais cara do catalogo

Existem 246 ferramentas com sufixo numerico (`_1`, `_2`, ate `_19`). Elas **nao sao variantes** da
ferramenta base. Sao operacoes completamente diferentes que por acaso mexem no mesmo recurso.

Exemplos reais, todos da familia "contatos":

| Ferramenta | O que voce acha que faz | O que ela FAZ de verdade |
|---|---|---|
| `lionchat_contacts_create` | cria contato | cria contato (essa esta certa) |
| `lionchat_contacts_create_3` | cria contato | mexe nas **etiquetas** do contato — e **SUBSTITUI a lista inteira**, mesmo o titulo dela dizendo "Adicionar Labels" |
| `lionchat_contacts_create_4` | cria contato | **mescla dois contatos** — nao tem volta |
| `lionchat_contacts_create_5` | cria contato | **exporta a base** e manda por e-mail |
| `lionchat_conversations_unread` | lista as nao lidas | **marca a conversa como nao lida** |

Regra: **nunca deduza pelo nome; leia a ficha inteira — titulo E descricao.** Se a ficha nao
estiver clara, nao chame. O caso das etiquetas de contato mostra por que o titulo sozinho nao
basta: ele diz "Adicionar Labels" e a descricao diz "Substitui os labels do contato pelo array
enviado". Quem parou no titulo apagou as outras etiquetas da pessoa.

Outros nomes que enganam:

- `lionchat_account_list` **nao lista contas da plataforma** — devolve o perfil do usuario logado
  (com as contas dele dentro). E o unico jeito, no conector local, de descobrir quais empresas o
  token alcanca.
- `lionchat_conversations_search` funciona, mas e o caminho antigo e lento: ele procura dentro do
  texto das mensagens e leva de 15 a 22 segundos (medido em producao). Para achar a conversa de uma
  pessoa ou empresa, use `lionchat_search_list` ou `lionchat_search_search`.
- **A familia `lionchat_search_*` sao BUSCAS, apesar do sufixo `_list`.** Aquele `_list` e
  historico. A regra "nao baixe tudo com `_list`" vale para `lionchat_<recurso>_list`, nao para
  essas.
- Para listar as conversas nao lidas, use a listagem de conversas com o tipo "nao lidas"; para
  contar, `lionchat_conversations_meta`.

---

## 3. As 76 areas, com o slug e a contagem

A coluna **Slug** e o valor aceito no filtro do conector LOCAL (`LIONCHAT_CATEGORIES`).
A coluna **Nome da area** e o valor aceito no filtro do conector REMOTO (`?categories=`), em
minusculo. Os dois nao sao iguais — errar deixa o cliente sem nenhuma ferramenta daquela area, sem
erro nenhum.

| Slug (local) | Nome da area (remoto) | Ferramentas |
|---|---|---|
| account | Conta | 4 |
| account_variables | Variaveis da Conta | 5 |
| agendamento | Agendamento | 1 |
| agent_availability | Disponibilidade | 4 |
| agents | Agentes | 10 |
| announcements | Anuncios | 5 |
| assignment_policies | Politicas de Atribuicao | 11 |
| auditoria | Auditoria | 1 |
| automation_rules | Regras de Automacao | 7 |
| biblioteca_de_midia | Biblioteca de Midia | 6 |
| booking_event_types | Tipos de Evento | 8 |
| campanhas | Campanhas | 21 |
| campanhas_sms_voz | Campanhas SMS/Voz | 5 |
| canned_responses | Respostas Rapidas | 6 |
| capacity_policies | Politicas de Capacidade | 11 |
| captain_assistants | Assistentes | 46 |
| captain_documents | Base de Conhecimento | 6 |
| chat_interno | Chat Interno | 11 |
| companies | Empresas | 6 |
| contacts | Contatos | 37 |
| conversations | Conversas | 39 |
| copilot_prompts | Prompts Salvos | 4 |
| csat | CSAT | 4 |
| csat_template | CSAT Template | 2 |
| custom_attributes | Atributos Personalizados | 5 |
| custom_filters | Filtros Personalizados | 5 |
| custom_roles | Roles Personalizados | 5 |
| dashboard_apps | Dashboard Apps | 5 |
| documentos_do_contato | Documentos do Contato | 6 |
| ecommerce_webhooks | Integracoes E-commerce | 63 |
| ferramentas_da_ia | Ferramentas da IA | 11 |
| flow_sessions | Sessoes | 3 |
| flows | Flows | 21 |
| formularios | Formularios | 12 |
| funnels | Funis | 14 |
| google_calendar | Google Calendar | 14 |
| ia_captain | IA / Captain | 2 |
| inbox_members | Membros de Inbox | 4 |
| inbox_migration | Migracao de Inbox | 3 |
| inboxes | Caixas de Entrada | 21 |
| integracoes | Integracoes | 86 |
| kanban_agents | Agentes (Kanban) | 3 |
| kanban_bulk | Operacoes em Massa | 4 |
| kanban_checklist | Checklist | 6 |
| kanban_config | Config Kanban | 6 |
| kanban_items | Itens | 28 |
| kanban_notes | Notas | 4 |
| kanban_v2 | Kanban V2 | 5 |
| labels | Labels | 5 |
| ligacoes_whatsapp | Ligacoes WhatsApp | 7 |
| liontrack | LionTrack | 8 |
| macros | Macros | 7 |
| messages | Mensagens | 12 |
| meta_lead | Meta Lead Ads | 11 |
| notification_settings | Config de Notificacao | 2 |
| notifications | Notificacoes | 8 |
| offers | Ofertas | 6 |
| portals | Portais | 20 |
| public_booking | Publica | 5 |
| public_contacts | Contatos (Cliente) | 3 |
| public_conversations | Conversas (Cliente) | 5 |
| public_csat | Pesquisa CSAT | 1 |
| public_messages | Mensagens (Cliente) | 3 |
| reports | Relatorios | 38 |
| scheduled_messages | Mensagens Agendadas | 10 |
| search | Busca | 6 |
| sla | SLA | 8 |
| support_access | Acesso de Suporte | 3 |
| tasks | Agenda / Tarefas | 16 |
| teams | Times | 11 |
| upload | Upload | 3 |
| voip | VoIP | 1 |
| voip_calls | Chamadas | 26 |
| waha_groups | Grupos WhatsApp | 14 |
| webhooks | Webhooks | 6 |
| whatsapp_templates | Templates WhatsApp | 9 |

**Cuidado com a contagem no conector local:** `lionchat_list_categories` conta o catalogo inteiro,
inclusive as ferramentas que aquela sessao nao carregou. As 6 de agendamento e pesquisa publicos
(`public_booking`, `public_csat`) sao desligadas de fabrica mas continuam contadas. Se a contagem
disser que existe e a ferramenta nao aparecer, e isso.

---

## 4. As ferramentas que nao vem do catalogo

| Ferramenta | Onde existe | Para que serve |
|---|---|---|
| `lionchat_ping` | so no LOCAL | prova a conexao e devolve o perfil. Primeira chamada de qualquer trabalho |
| `lionchat_list_categories` | so no LOCAL | lista as areas e quantas ferramentas cada uma tem, sem chamar a plataforma |
| `lionchat_current_account` | so no REMOTO | em qual conta a sessao esta agora |
| `lionchat_list_my_accounts` | so no REMOTO | as contas do usuario, com papel e situacao |
| `lionchat_switch_account` | so no REMOTO | troca a conta da sessao inteira; a escolha fica gravada |
| `lionchat_contacts_bulk_create` | so no REMOTO | cria ate 1000 contatos numa chamada. Cada campo personalizado usado precisa JA EXISTIR na conta, senao e descartado |
| `lionchat_flows_schema_reference` | nos DOIS | o manual completo do desenho de fluxo. Nao gasta chamada. **Obrigatorio antes de criar ou alterar fluxo** |
| `lionchat_search_tools` | REMOTO compacto | procura no catalogo por linguagem natural, com ou sem acento |
| `lionchat_describe_tool` | REMOTO compacto | a ficha completa de uma ferramenta |
| `lionchat_execute_tool` | REMOTO compacto | executa qualquer ferramenta pelo nome |

---

## 5. Os 12 documentos de referencia que voce pode pedir

Sao documentos longos que ficam guardados no conector e so ocupam espaco quando voce os pede. Peca
pelo endereco. Nem todo cliente de IA le sozinho — peca explicitamente.

| Endereco | Quando ler |
|---|---|
| `lionchat://docs/flowbuilder-design-guide` | **obrigatorio** antes de criar ou alterar fluxo |
| `lionchat://docs/formularios-publicos` | **obrigatorio** antes de criar formulario de captacao |
| `lionchat://docs/filtros-e-relatorios` | antes de montar qualquer filtro ou relatorio |
| `lionchat://docs/flowbuilder-patterns` | 10 fluxos prontos para adaptar |
| `lionchat://docs/kanban-deep-dive` | funil, etapas, cards, esteira, permissoes |
| `lionchat://docs/conversation-flows` | ciclo de vida da conversa: saudacao, atribuicao, IA, resolucao |
| `lionchat://docs/reports-guide` | como ler cada relatorio, unidades, CSAT, SLA |
| `lionchat://docs/best-practices` | ordem das operacoes, IA a fundo, campanhas, modelos |
| `lionchat://docs/api-conventions` | paginacao, filtros, datas, erros, limites |
| `lionchat://docs/data-model` | como as coisas se ligam entre si |
| `lionchat://docs/glossary` | termos, situacoes e opcoes de cada campo |
| `lionchat://docs/troubleshooting` | erros e cenarios de produto |

---

## 6. Os 9 roteiros prontos

O cliente dispara pela barra de comando da ferramenta de IA dele. Se ele pedir alguma dessas coisas
com outras palavras, voce pode simplesmente executar o mesmo trabalho.

| Roteiro | O que responde |
|---|---|
| `productivity_report` | produtividade da equipe no periodo |
| `stuck_leads` | cards parados ha N dias na mesma etapa |
| `weekly_recap` | resumo executivo da semana |
| `customer_health` | retrato completo de um cliente: conversas, cards e satisfacao |
| `inactive_contacts` | quem sumiu, para reativar |
| `team_load_balance` | como a carga esta distribuida entre os atendentes |
| `quality_audit` | auditoria por amostra de conversas resolvidas |
| `whatsapp_template_usage` | quais modelos aprovados sao usados e quais estao parados |
| `create_flow_brainstorm` | perguntas para descobrir o que o cliente quer antes de desenhar o fluxo |

---

## 7. Ensaios: como ver o resultado sem criar nada

Use SEMPRE que existir. E o que separa "eu acho que vai funcionar" de "eu te mostro antes".

| Ferramenta | O que ela mostra sem criar nada |
|---|---|
| `lionchat_campaigns_estimate_audience` | quantos contatos com telefone caem no criterio da campanha. Mesmo motor do disparo. Chame sempre e mostre o numero ao cliente |
| `lionchat_flows_check_conflicts` | se os gatilhos e caixas do fluxo colidiriam com outro fluxo ativo. Ativar com conflito e recusado |
| `lionchat_lead_forms_test_run` | roda o rascunho do formulario no motor de verdade e desfaz tudo: nao cria contato, nao cria conversa, nao dispara nada |
| `lionchat_lead_forms_check_embed` | se um site aceita ser exibido dentro do formulario |
| `lionchat_captain_assistants_playground` | conversa de teste com o agente de IA |
| `lionchat_captain_assistants_quality` | revisao de como o agente foi montado. So leitura |
| `lionchat_flow_tools_run` | testa uma ferramenta da IA antes de vincular ao agente |
| `lionchat_flows_test_ai_node` | testa o bloco de IA de dentro de um fluxo, sem enviar nem gravar nada na conversa. A IA roda de verdade, entao tem custo de uso |
| `lionchat_captain_custom_tools_test` | testa uma ferramenta personalizada do agente. A chamada ao sistema do cliente acontece DE VERDADE |
| `lionchat_custom_dashboards_preview_widget` | calcula um bloco de relatorio na hora e devolve os numeros sem salvar |
| `lionchat_report_alerts_preview` e `lionchat_report_alerts_preview_status` | como o aviso vai ficar antes de arma-lo. E o unico jeito de mostrar o resultado antes, porque criar o aviso exige o OK explicito |
| `lionchat_upload_validate` e `lionchat_upload_limits_show` | testa uma URL de midia (tipo, tamanho, seguranca) e mostra os limites por tipo de arquivo |
| `lionchat_integrations_waha_check_phone` | pergunta ao WhatsApp se um numero existe, por uma caixa QR Code conectada. Em numero brasileiro devolve a forma certa, com ou sem o nono digito |
| `lionchat_meta_lead_validate_token` | se a chave dos anuncios da Meta ainda vale |
| `lionchat_meta_pixel_integrations_test_event` | dispara um evento de teste para a Meta. **Este NAO e ensaio limpo**: o evento vai de verdade e, se a Meta aceitar, a integracao e LIGADA automaticamente. Peca o OK antes |
| `lionchat_ga4_integrations_test_connection` e as irmas `_test_connection` | testa a conexao de GA4, TopSend, Omie e VTCall (telefonia). Conta Azul nao tem teste: o que existe la e `lionchat_conta_azul_integrations_sync_now`, que sincroniza de verdade |
| `lionchat_agents_assigned_resources` | o que esta pendurado num atendente: conversas, cards do funil, tarefas, mensagens agendadas, campanhas, automacoes e funis que citam ele. Leia antes de propor tirar ou trocar alguem |

**Nao existe ensaio de planilha.** As tres ferramentas de importacao por arquivo
(`lionchat_contacts_import`, `lionchat_contacts_import_validate`, `lionchat_kanban_items_import`)
exigem um arquivo, e enviar arquivo nao existe na conexao por IA: elas aparecem na lista e sempre
falham. Para criar muitos contatos, use `lionchat_contacts_bulk_create` (so no conector remoto, ate
1000 por chamada) ou mande o cliente importar pela tela.

---

## 8. As 17 ferramentas que exigem o OK do cliente

Estas RECUSAM a primeira chamada de proposito. A recusa nao e defeito: e o pedido de autorizacao.
Descreva o efeito exato ao cliente, espere o "sim" e reenvie a MESMA chamada com `confirm:true`.

- `lionchat_custom_dashboards_create`, `_update`, `_destroy` — painel de relatorio personalizado
- `lionchat_report_alerts_create`, `_update`, `_destroy`, `_send_now` — aviso de relatorio por e-mail
- `lionchat_agents_replace` — substituir um atendente por outro em tudo que estava com ele
- `lionchat_campaigns_reprogram_dispatch` e `lionchat_campaigns_remove_dispatch_entry` — mexer no
  cronograma de uma campanha ja programada
- `lionchat_kanban_items_meta_capi_fire`, `_ga4_mp_fire`, `_google_ads_fire` — disparar conversao
  manualmente para Meta, Google Analytics e Google Ads
- `lionchat_google_contacts_disconnect` — desconectar o Google Contatos
- `lionchat_greenn_webhooks_destroy` — excluir uma integracao de pagamento
- `lionchat_meta_lead_bulk_destroy` — excluir varios formularios de anuncio de uma vez
- `lionchat_captain_assistants_quality_apply_proposal` — aplicar no agente as mudancas que a
  revisao de qualidade sugeriu

**Esta lista nao substitui a sua regra de confirmacao.** Ela e o piso do sistema, nao o teto do seu
comportamento: voce confirma com o cliente antes de QUALQUER criacao, alteracao, exclusao ou envio
de mensagem, esteja a ferramenta nesta lista ou nao.

---

## 9. O que a resposta esconde de voce

**Segredo nunca volta.** Chave de integracao, senha, token de acesso e afins voltam sempre
censurados, e nao ha como pedir a versao completa. Se o cliente quer conferir uma chave, ele ve no
painel. Nao insista, nao entre em laco.

**Campo pesado vem podado.** Modelos de WhatsApp, horario de atendimento, historico de importacao e
fotos de perfil vem substituidos por um marcador de texto para caber no limite de tamanho. **Nao
conclua "a conta nao tem horario de atendimento" porque veio o marcador** — significa que foi
podado, nao que esta vazio. Para pedir a resposta completa existe a opcao `full_response: true`,
disponivel em cerca de 28 ferramentas principais (conversas, contatos, campanhas, caixas, cards,
busca, atendentes, times, conta). O caminho melhor costuma ser chamar a ferramenta dedicada: para
os modelos, `lionchat_inboxes_whatsapp_templates_list`.

**Lista muito grande vem cortada.** Passando de 80 mil caracteres, o conector prefere devolver
menos itens INTEIROS a cortar um registro no meio, e avisa no fim quantos de quantos ele mostrou.
Se voce viu esse aviso, o trabalho nao acabou: continue paginando.
