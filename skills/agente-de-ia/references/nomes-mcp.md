# Nomes das ferramentas do conector nesta area

Os nomes desta area sao enganosos. Cenario, pergunta frequente e ferramenta de API vivem todos sob nomes
que comecam com "assistants", porque o nome foi derivado do primeiro pedaco do endereco. **Nao invente o
nome logico** - ele nao existe.

## O que NAO existe

- `lionchat_captain_scenarios_create` - nao existe. Cenario e `lionchat_captain_assistants_create_1`.
- `lionchat_captain_faqs_create` / `lionchat_captain_responses_create` - nao existem. Pergunta frequente e
  `lionchat_captain_assistants_create_2`.
- `lionchat_captain_custom_tools_create` - nao existe para criar. Ferramenta de API e
  `lionchat_captain_assistants_create_5` (mas o TESTE dela e `lionchat_captain_custom_tools_test`, esse sim
  com nome direto).

## Agente

| Nome | O que faz |
|---|---|
| `lionchat_captain_assistants_list` | lista os agentes |
| `lionchat_captain_assistants_show` | detalhe de um agente. **Use sempre depois de gravar, para conferir** |
| `lionchat_captain_assistants_create` | cria. Exige `name` e `description` |
| `lionchat_captain_assistants_update` | altera. Mistura parcial do `config`; `paused` vai no topo |
| `lionchat_captain_assistants_destroy` | apaga. **Nao use** |
| `lionchat_captain_assistants_tools` | catalogo de ferramentas com ligado/desligado por agente. Passe `assistant_id` |
| `lionchat_captain_assistants_playground` | testa o agente sem afetar cliente nenhum |

## Cenarios (enderecos aninhados no agente)

| Nome | O que faz |
|---|---|
| `lionchat_captain_assistants_list_3` | lista os cenarios de um agente |
| `lionchat_captain_assistants_create_1` | **cria cenario** |
| `lionchat_captain_assistants_show_1` | detalhe do cenario |
| `lionchat_captain_assistants_update_2` | altera o cenario |
| `lionchat_captain_assistants_destroy_1` | apaga o cenario |

## Perguntas frequentes

| Nome | O que faz |
|---|---|
| `lionchat_captain_assistants_list_4` | lista |
| `lionchat_captain_assistants_create_2` | **cria pergunta frequente** |
| `lionchat_captain_assistants_show_2` | detalhe |
| `lionchat_captain_assistants_update_3` | altera |
| `lionchat_captain_assistants_destroy_2` | apaga |
| `lionchat_captain_assistants_bulk_actions` | acao em massa sobre as perguntas frequentes. Pode APAGAR em lote - so com pedido explicito |

## Ferramentas de API

| Nome | O que faz |
|---|---|
| `lionchat_captain_assistants_list_7` | lista |
| `lionchat_captain_assistants_create_5` | **cria ferramenta de API** |
| `lionchat_captain_assistants_show_3` | detalhe |
| `lionchat_captain_assistants_update_4` | altera |
| `lionchat_captain_assistants_destroy_3` | apaga |
| `lionchat_captain_custom_tools_test` | testa o endereco externo antes de deixar a IA usar |

## Base de conhecimento

| Nome | O que faz |
|---|---|
| `lionchat_captain_documents_list` | lista. Aceita busca, status e modo enxuto (sem o texto inteiro) |
| `lionchat_captain_documents_create` | cria pagina. Exige `assistant_id` |
| `lionchat_captain_documents_show` | detalhe |
| `lionchat_captain_documents_update` | altera |
| `lionchat_captain_documents_destroy` | apaga |
| `lionchat_captain_documents_create_1` | **reprocessa** a pagina. O nome engana: nao e um segundo jeito de criar |

## Biblioteca de midia

| Nome | O que faz |
|---|---|
| `lionchat_captain_media_assets_list` | lista |
| `lionchat_captain_media_assets_create` | cadastra. Exige o identificador do arquivo |
| `lionchat_captain_media_assets_show` / `_update` / `_destroy` | detalhe, altera, apaga |
| `lionchat_captain_media_assets_reprocess` | manda reler o conteudo do arquivo |
| `lionchat_upload_create` | sobe um arquivo por endereco publico e devolve o identificador que o cadastro de midia pede |
| `lionchat_upload_validate` | testa um endereco de arquivo sem gravar nada (tipo, tamanho, se e pagina web) |

## Ferramentas de fluxo

| Nome | O que faz |
|---|---|
| `lionchat_flow_tools_create` | **cria**. Porta certa - NUNCA `lionchat_flows_create` |
| `lionchat_flow_tools_list` / `_show` / `_update` / `_destroy` / `_toggle` | lista, detalhe, altera, apaga, liga e desliga |
| `lionchat_flow_tools_run` | testa com parametros reais, sem passar pela IA. Use ANTES de vincular |
| `lionchat_flow_tools_assistants_list` | quais agentes usam a ferramenta. **Leia antes de vincular** |
| `lionchat_flow_tools_assistants_update` | vincula. **SUBSTITUI a lista inteira** - id ausente e desvinculado |
| `lionchat_flow_tools_executions_list` | historico de execucoes, para descobrir por que falhou |
| `lionchat_flow_tool_executions_show` | detalhe de uma execucao (repare: `flow_tool_executions`, no singular de "tool") |

## Ligar, acompanhar e diagnosticar

| Nome | O que faz |
|---|---|
| `lionchat_conversations_update` | liga a IA numa conversa (`captain_assistant_id` e `captain_reply_now`) |
| `lionchat_kanban_bulk_bulk_actions` | liga em massa (`type: "Conversation"`, `fields.captain_assistant_id`) |
| `lionchat_automation_rules_create` / `_update` | ligacao automatica (acao `assign_captain_assistant`) |
| `lionchat_conversations_captain_reasoning` | le o raciocinio guardado naquela conversa |
| `lionchat_conversations_captain_history_reset` | zera o que a IA enxerga na conversa, sem apagar nada |
| `lionchat_captain_supervisor_show` | placar comparativo dos agentes |
| `lionchat_captain_assistants_quality` | revisao de como o agente foi montado |
| `lionchat_captain_assistants_quality_apply` | aplica o conserto mecanico |
| `lionchat_captain_assistants_quality_rewrite` | pede proposta de reescrita com IA (nao grava) |
| `lionchat_captain_assistants_quality_apply_proposal` | grava a proposta aprovada |
| `lionchat_captain_assistants_quality_undo` | desfaz |
| `lionchat_captain_liquid_variables_list` | referencia rapida de variaveis (e o autocompletar das ferramentas de API, nao a lista completa da instrucao). Quem confere de verdade e a revisao de qualidade |
| `lionchat_captain_voices_list` | catalogo de vozes por provedor |

## Terreno (o que decide quais ferramentas vao existir)

`lionchat_labels_list` e `lionchat_labels_create` (etiqueta COM descricao), `lionchat_teams_list` e
`lionchat_teams_create` (time COM descricao), `lionchat_funnels_list`, `lionchat_booking_event_types_list`,
`lionchat_custom_attributes_list` e `lionchat_custom_attributes_create`, `lionchat_offers_list`,
`lionchat_inboxes_list`, `lionchat_account_show`.

## Cuidado: NAO e o Agente de IA

| Nome | O que e |
|---|---|
| `lionchat_copilot_settings_show` / `_update` | motor do Copiloto - **outro produto**: ajuda o ATENDENTE dentro do painel e nao fala com o cliente. Tem modelo, criatividade e comportamento proprios |
| `lionchat_copilot_prompts_*` | atalhos salvos do Copiloto |
| `lionchat_captain_assistants_list_1` / `_update_1` | tela "Configuracoes de IA" da conta. **Os valores dela nao sao lidos por ninguem hoje** - nao mande o cliente escolher o modelo ali |
| `lionchat_captain_assistants_create_6` ate `_create_10` | tarefas de apoio ao atendente (reescrever, resumir, sugerir resposta, sugerir etiqueta, sugerir acompanhamento). Nao sao configuracao do agente |
| `lionchat_captain_assistants_list_5` / `_list_6` / `_create_3` / `_create_4` | conversas do Copiloto |

## O que NAO tem cobertura pelo conector

Diga ao cliente que estes precisam ser feitos na tela:

- Cadastrar a chave da OpenAI da conta e ligar o recurso de IA (Integracoes).
- Subir o ARQUIVO da biblioteca de midia quando ele nao esta numa URL publica.
- Tocar a amostra de uma voz.
- Escolher um TIME por seletor no cenario - hoje o nome do time e texto escrito na instrucao.

E estes NAO existem em lugar nenhum hoje, nem na tela - nao prometa ao cliente:

- Fixar a AGENDA por cenario (o campo aparece no formulario, mas o servidor descarta ao salvar). Use
  `config.booking_event_type_ids` no agente inteiro.
- Cenario que dispara sozinho na ativacao ou na primeira mensagem do cliente.
- Envio garantido de arquivo por cenario.

## Se um nome nao existir no seu conector

Existem duas versoes do conector do LionChat e o conjunto de ferramentas nao e identico. Se um nome deste
documento nao existir na sua lista, **nao invente um parecido**: descreva a acao em palavras ao cliente e
pergunte. Errar o nome faz a IA concluir que a funcionalidade nao existe, quando ela existe.
