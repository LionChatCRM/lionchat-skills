# Mapa das ferramentas: o que cada uma faz e o que ela NAO faz

Indice:
1. Conta e limites
2. Ler a caixa
3. Alterar a caixa
4. Membros da caixa
5. Distribuicao: politicas
6. Capacidade
7. Times
8. Agentes e cargos
9. Disponibilidade do agente
10. SLA
11. Pesquisa de satisfacao
12. WhatsApp por QR Code
13. WhatsApp Oficial
14. Conversas orfas e caixas contactaveis
15. Relatorios para conferir a divisao
16. Ferramentas que NAO existem

Nomes escritos exatamente como devem ser chamados. Se algo que voce precisa nao estiver aqui,
pergunte ao cliente em vez de tentar um nome parecido.

---

## 1. Conta e limites

| Ferramenta | Para que |
|---|---|
| `lionchat_account_show` | Nome, fuso, plano, limites e recursos ligados. Leia SEMPRE antes de criar qualquer coisa |
| `lionchat_account_settings_update` | Altera nome, idioma, e-mail de suporte, dominio e FUSO da conta. Fuso invalido e descartado em silencio: releia depois |
| `lionchat_account_list` | Perfil do usuario com todas as contas as quais ele tem acesso |

**NAO faz:** gravar o horario de atendimento da conta nem a pausa de almoco.

---

## 2. Ler a caixa

| Ferramenta | Para que |
|---|---|
| `lionchat_inboxes_list` | Todas as caixas da conta |
| `lionchat_inboxes_show` | Uma caixa em detalhe: canal, saudacao, ausencia, conversa unica, pesquisa de satisfacao, bloco de distribuicao, se ha politica vinculada, fuso |
| `lionchat_inboxes_campaigns_list` | As campanhas ligadas aquela caixa. Consulte antes de qualquer conversa sobre excluir a caixa |

---

## 3. Alterar a caixa

| Ferramenta | Para que |
|---|---|
| `lionchat_inboxes_update` | Nome, saudacao (ligar e texto), ligar/desligar a distribuicao automatica e o bloco de distribuicao |

**NAO faz:** aviso de ausencia, conversa unica, pesquisa de satisfacao, central de ajuda vinculada,
fuso da caixa, assinatura do remetente, interruptores de grupo/canal/transmissao. Todos esses sao
descartados em silencio - a resposta volta com sucesso e nada muda. Ver `configurar-caixa.md`.

Outras ferramentas de caixa que existem e podem ser uteis: `lionchat_inboxes_register_webhook`
(reassinar o webhook do canal).

---

## 4. Membros da caixa

| Ferramenta | Para que |
|---|---|
| `lionchat_inbox_members_show` | Quem sao os membros e quem e supervisor |
| `lionchat_inbox_members_update` | **A que voce usa.** Define a lista COMPLETA, separando `user_ids` de `supervisor_ids` |
| `lionchat_inbox_members_create` | Apenas ACRESCENTA quem falta e IGNORA supervisores, apesar da descricao dizer o contrario. Evite |
| `lionchat_inbox_members_destroy` | Remove membros. Nao use: voce nao apaga nada |
| `lionchat_inboxes_list_1` | Quem aparece na lista de escolher responsavel de UMA caixa: membros MAIS todos os administradores da conta. NAO e a lista de quem entra no sorteio automatico |
| `lionchat_assignable_agents_list` | O mesmo, considerando VARIAS caixas ao mesmo tempo (quem e membro de todas elas, mais os administradores) |

---

## 5. Distribuicao: politicas

| Ferramenta | Para que |
|---|---|
| `lionchat_assignment_policies_list` / `_show` | Politicas de atribuicao que existem |
| `lionchat_assignment_policies_create` / `_update` | Criar e alterar a politica |
| `lionchat_inboxes_assignment_policies_list` | Ver a politica pelo lado da CAIXA |
| `lionchat_inboxes_assignment_policies_create` | Vincular a politica a caixa |
| `lionchat_assignment_policies_list_1` | Ver as caixas de uma politica, pelo lado da politica |
| `lionchat_assignment_policies_create_1` | Vincular caixa pelo lado da politica |

As de excluir (`_destroy`, `_destroy_1`) existem, mas voce nao apaga nada.

---

## 6. Capacidade

| Ferramenta | Para que |
|---|---|
| `lionchat_capacity_policies_list` / `_show` | Politicas de capacidade que existem |
| `lionchat_capacity_policies_create` | Cria a politica (nome, descricao, regras de excecao) |
| `lionchat_capacity_policies_create_2` | Define o limite de conversas POR CAIXA |
| `lionchat_capacity_policies_update_1` | Altera esse limite |
| `lionchat_capacity_policies_create_1` | Vincula UM agente a politica |
| `lionchat_capacity_policies_list_1` | Lista os agentes vinculados |

Os tres passos de criacao sao obrigatorios juntos. Ver `distribuicao.md`.

---

## 7. Times

| Ferramenta | Para que |
|---|---|
| `lionchat_teams_list` / `lionchat_teams_show` | Times que existem |
| `lionchat_teams_create` / `lionchat_teams_update` | Criar e alterar: nome, distribuir sozinho, so online ou tambem offline, ordem e freio |
| `lionchat_teams_list_2` | Membros do time |
| `lionchat_team_members_update` | Define a lista COMPLETA de membros |
| `lionchat_team_members_create` | Acrescenta membros |
| `lionchat_team_members_kanban_reassignment_preview` | Previa: quais cards a pessoa perderia ao sair |
| `lionchat_team_members_remove_agents_with_reassignment` | Remove ja reatribuindo os cards |

**NAO faz:** marcar supervisor de TIME. O campo nao esta declarado nas ferramentas de membros de
time; isso e no painel.

---

## 8. Agentes e cargos

| Ferramenta | Para que |
|---|---|
| `lionchat_agents_list` | Quem existe na conta |
| `lionchat_agents_create` | Convida uma pessoa, ja com papel, times e caixas |
| `lionchat_agents_bulk_create` | Convida varias de uma vez (so os e-mails) |
| `lionchat_agents_update` | Altera papel, disponibilidade, privilegio de permanecer disponivel, cargo, times e caixas |
| `lionchat_agents_resend_invitation` | Reenvia o convite |
| `lionchat_agents_assignments` | Times e caixas em que a pessoa esta, com o papel em cada uma. Mostra tambem os vinculos que ficaram PENDENTES de um convite ainda nao aceito - e a forma de conferir isso |
| `lionchat_agents_assigned_resources` | O que ficaria orfao se ela sair (conversas, times, caixas) |
| `lionchat_agents_replace` | Substitui uma pessoa por outra e enfileira a exclusao da antiga. IRREVERSIVEL e exige `confirm: true` |
| `lionchat_custom_roles_list` / `_show` | Cargos personalizados que existem |
| `lionchat_custom_roles_create` / `_update` | Criar e alterar cargo com a lista de poderes |

---

## 9. Disponibilidade do agente

| Ferramenta | Para que |
|---|---|
| `lionchat_agent_availability_list` | Le a grade semanal |
| `lionchat_agent_availability_update` | Substitui a grade INTEIRA e sempre para o usuario logado |
| `lionchat_agent_availability_list_1` | Confere se alguem esta disponivel num horario |
| `lionchat_agent_availability_list_2` | Lista quem esta livre |

Isso e a agenda de disponibilidade, nao o status online/offline e nao decide quem recebe conversa.

---

## 10. SLA

| Ferramenta | Para que |
|---|---|
| `lionchat_sla_list` / `lionchat_sla_show` | Politicas que existem |
| `lionchat_sla_create` / `lionchat_sla_update` | Criar e alterar. Prazos em SEGUNDOS |
| `lionchat_sla_list_1` | Conversas que estouraram |
| `lionchat_sla_metrics` / `lionchat_sla_download` | Numeros e exportacao |
| `lionchat_automation_rules_create` | A regra com a acao "Adicionar SLA", que e o que APLICA a politica |

**NAO faz:** o ajuste de comecar o cronometro na atribuicao (o campo nao esta declarado). Painel.

---

## 11. Pesquisa de satisfacao

| Ferramenta | Para que |
|---|---|
| `lionchat_csat_list` | Respostas recebidas |
| `lionchat_csat_metrics` | Numeros no periodo |
| `lionchat_csat_download` | Exportacao |
| `lionchat_csat_update` | Grava a anotacao interna do gestor sobre uma avaliacao. NAO muda a nota que o cliente deu |

**NAO faz:** ligar a pesquisa na caixa nem configurar os textos. Painel, aba CSAT da caixa. As
ferramentas de modelo de pesquisa (`lionchat_inboxes_csat_template_*`) sao heranca do modo antigo:
hoje a pesquisa e mensagem comum e nao usa modelo. Nao gaste tempo com elas.

---

## 12. WhatsApp por QR Code

| Ferramenta | Para que |
|---|---|
| `lionchat_inboxes_waha_status` | Estado da sessao. WORKING conectado, SCAN_QR_CODE precisa ler, STOPPED e FAILED sao problema |
| `lionchat_inboxes_waha_qrcode` | Devolve o TEXTO do QR (nao uma imagem). So funciona no estado SCAN_QR_CODE. Expira em segundos |
| `lionchat_inboxes_waha_connect` | Inicia a sessao |
| `lionchat_inboxes_waha_import_history` | Traz o historico do aparelho, de 1 a 90 dias. Uma rodada por vez |
| `lionchat_inboxes_waha_import_status` | Andamento da importacao |
| `lionchat_inboxes_waha_groups_list` / `_show` / `_list_1` / `_list_2` | Grupos, participantes e convite |
| `lionchat_inboxes_waha_groups_create` e irmas | Colocar e tirar participante, promover e rebaixar administrador, revogar convite, sair do grupo |
| `lionchat_inboxes_waha_groups_update` e irmas | Renomear, descricao, ajustes e foto do grupo |
| `lionchat_integrations_waha_check_phone` | Confere se um numero tem WhatsApp |
| `lionchat_inboxes_inbox_migration_list` | Previa da migracao de conversas |
| `lionchat_inboxes_inbox_migration_execute` | Executa a migracao. Destrutivo: previa primeiro, confirmacao depois |
| `lionchat_inboxes_inbox_migration_list_1` | Andamento da migracao |

**NAO faz:** ler o QR Code. Isso e a pessoa, no celular.

---

## 13. WhatsApp Oficial

| Ferramenta | Para que |
|---|---|
| `lionchat_inboxes_health` | Painel de Saude da conta na Meta |
| `lionchat_inboxes_sync_templates` | Sincroniza os modelos aprovados |
| `lionchat_inboxes_whatsapp_templates_list` e irmas | Listar, criar e alterar modelos |
| `lionchat_inboxes_whatsapp_history_start` / `_status` / `_cancel` | Importar historico |
| `lionchat_inboxes_coex_backup_connect` / `_status` / `_disconnect` | Conexao hibrida |
| `lionchat_inboxes_enable_whatsapp_calling` / `_disable_whatsapp_calling` | Ligacao pelo WhatsApp Oficial |
| `lionchat_inboxes_failed_messages_summary` | Falhas de envio agrupadas por motivo |
| `lionchat_inboxes_failed_messages_bulk_retry` / `_bulk_cancel` | Reenviar ou cancelar em lote. Reenviar entrega ao cliente final: confirme antes |

**NAO faz:** conectar o numero pelo botao da Meta (Cadastro Incorporado). Isso e no painel, com
login na Meta.

---

## 14. Conversas orfas e caixas contactaveis

| Ferramenta | Para que |
|---|---|
| `lionchat_contacts_list_3` | Por quais caixas da para falar com aquele contato |
| `lionchat_conversations_create_9` | Religa uma conversa orfa (de caixa excluida) a outra caixa. Recusa quando a caixa de WhatsApp de destino ja tem conversa ativa com o contato |

---

## 15. Relatorios para conferir a divisao

| Ferramenta | Para que |
|---|---|
| `lionchat_reports_list_2` | Resumo por CAIXA no periodo |
| `lionchat_reports_list_1` | Resumo por TIME no periodo (nao traz o nome do time: cruze com a lista de times) |
| `lionchat_reports_summary` | Resumo geral |

Use depois de montar, para mostrar ao cliente se a distribuicao ficou equilibrada de verdade.

---

## 16. Ferramentas que NAO existem

Nao invente nome parecido. Estas operacoes NAO tem ferramenta:

- **Criar caixa de entrada.** Nem excluir.
- **Gerar o link publico de conexao por QR** (a pagina que deixa quem esta com o celular ler o
  codigo sem ter login no painel). Existe no produto, dura 2 horas, so um link fica vivo por vez e
  pode ser revogado - mas so pelo painel.
- **Gravar o horario de atendimento**, nem o da conta nem o da caixa, nem a pausa de almoco.
- **Marcar supervisor de TIME.**
- **Ligar a pesquisa de satisfacao ou escrever os textos dela.**
- **Gravar aviso de ausencia, conversa unica e os interruptores de grupo/canal/transmissao.**
- **Construtor do Widget e formulario pre-chat do Chat do Site.**
- **CRIAR um grupo de WhatsApp.** Da para mexer em grupo que ja existe, nao para criar um. Criar e
  feito por um bloco de Fluxo.
- **Ligar ou desligar recursos de plano.** Isso e do time do LionChat, nao da conta.
