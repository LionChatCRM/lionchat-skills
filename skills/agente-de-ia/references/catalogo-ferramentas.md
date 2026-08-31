# Catalogo de ferramentas do Agente de IA

O que a IA sabe fazer de fabrica, o que cada coisa exige para EXISTIR, e o que acontece quando voce promete
no texto algo que a ferramenta nao carregou.

## Indice

1. Como ler este catalogo
2. Grupo 1 - sempre ativas
3. Grupo 2 - condicionais (so existem se o recurso existir na conta)
4. Grupo 3 - escondidas (a IA usa, o cliente nao liga nem desliga)
5. As cinco que NUNCA podem ser citadas num cenario
6. Ferramentas proprias: de API e de fluxo
7. Como desligar uma ferramenta de um agente
8. A pegadinha do "salvou mas nao carrega"

---

## 1. Como ler este catalogo

Toda ferramenta tem um identificador em minusculas (ex.: `add_label_to_conversation`). Esse identificador e
o que voce escreve dentro de um cenario, na forma `[Rotulo em portugues](tool://identificador)`.

Tres grupos, com regras diferentes:

| Grupo | Existe quando | Aparece na tela Ferramentas | Pode ser citada em cenario |
|-------|---------------|-----------------------------|----------------------------|
| Sempre ativas | sempre | sim | sim |
| Condicionais | so se o recurso existir na conta | sim (a maioria) | sim, mas veja a secao 8 |
| Escondidas | sozinhas, por conteudo ou por acao | nao | nao - o save falha ou nao adianta |

Antes de escrever qualquer cenario, rode `lionchat_captain_assistants_tools` passando o `assistant_id`.
Ela devolve o catalogo com um sinal de ligado/desligado por agente. **Ela e a fonte da verdade para o que
pode ser CITADO e para o que esta desligado - nao para o que vai carregar de verdade** (secao 8).

---

## 2. Grupo 1 - sempre ativas

Todo agente ja nasce com estas. Nao dependem de nada da conta.

| Identificador | O que faz | Quando a IA usa |
|---|---|---|
| `handoff` | Transfere para um atendente humano. Desliga a IA daquela conversa, abre a conversa e deixa uma nota interna com o motivo. | Cliente pede humano, ou o assunto sai do papel dela |
| `resolve_conversation` | Encerra a conversa. | So depois do cliente confirmar que esta satisfeito. **Cuidado**: encerrar dispara a pesquisa de satisfacao daquela caixa, se ela estiver configurada |
| `add_label_to_conversation` | Coloca etiqueta na conversa | Classificar assunto, interesse, objecao |
| `remove_label_from_conversation` | Tira etiqueta | Corrigir classificacao |
| `assign_agent` | Poe um atendente como responsavel, por nome ou e-mail. A IA continua respondendo. | Direcionar sem parar o atendimento |
| `set_conversation_pending` | Muda o status da conversa (pendente / aberta) | Marcar que algo ficou em espera |
| `update_contact` | Grava na ficha: nome, e-mail, telefone e dados cadastrais brasileiros (CPF, CNPJ, RG, passaporte, nascimento, genero, estado civil, profissao, endereco). **Nunca sobrescreve telefone ou e-mail ja preenchido**; dado sensivel exige o cliente confirmar antes | Cliente informa um dado dele |
| `update_attribute` | Grava campo personalizado e descobre sozinha se ele e de Contato, Conversa ou Card. Valida o tipo (lista so aceita valor cadastrado, hora exige HH:MM) | Guardar a resposta de uma qualificacao |
| `update_priority` | Muda a prioridade da conversa | Cliente irritado, caso urgente |
| `add_contact_note` | Anota no perfil do contato | Registrar contexto que vale para sempre |
| `update_contact_note` | ATUALIZA a anotacao existente em vez de acumular | Preferencia do cliente mudou |
| `add_private_note` | Recado interno na conversa, invisivel ao cliente | Passar contexto para quem vai assumir |
| `schedule_message` | Agenda uma mensagem futura na conversa, assinada pela IA | "Te lembro amanha as 9h" |
| `cancel_scheduled_message` | Cancela mensagem que ELA agendou. Nunca apaga agendamento feito por humano | Cliente remarcou ou desistiu |
| `request_team_input` | Pede ajuda da equipe por nota interna com mencao | **E a alternativa a inventar.** O comportamento padrao obriga usar isto quando ela nao sabe |
| `calculator` | Conta exata: totais, descontos, porcentagens, parcelas | Sempre que houver numero. Impede erro de aritmetica |

---

## 3. Grupo 2 - condicionais

**So carregam se o recurso existir.** Esta e a parte mais importante do catalogo: sem o recurso, a
ferramenta nao existe para aquele agente, e um texto que ensina o caminho dela faz a IA prometer o que nao
consegue executar.

| Identificadores | O que faz | EXIGE na conta |
|---|---|---|
| `list_available_funnels`, `create_kanban_item`, `move_kanban_item`, `add_kanban_note` | Criar e mover card no funil, e anotar nele. Funil e etapa aceitam nome parcial; se dois candidatos batem, ela nao adivinha - devolve a lista | pelo menos 1 funil |
| `view_booking_option`, `create_booking`, `reschedule_appointment` | Consultar horarios livres, marcar, remarcar e cancelar. `view_booking_option` e a UNICA fonte de horario | pelo menos 1 agenda configurada. **Sem agenda a IA nao agenda de jeito nenhum** - o modo avulso foi removido |
| `view_agenda` | Ver a agenda de um atendente num dia | existe, mas **quando a conta tem agenda ela responde "use a agenda"** e nao mostra nada. Nao monte cenario contando com ela |
| `check_agent_availability` | Diz quem esta trabalhando num horario, pelo expediente geral do atendente | expediente de atendente cadastrado. **Se a conta tem agenda, ela recusa responder** e manda usar a agenda - a agenda manda acima do expediente geral |
| `list_media_assets`, `send_media_asset`, `lookup_media_content` | Listar, enviar e reler o conteudo de um arquivo da biblioteca | ids liberados em `config.media_asset_ids` do agente. So cadastrar nao basta |
| `list_teams`, `assign_team`, `remove_team` | Rotear a conversa para um time, lendo a DESCRICAO do time. **So agem enquanto ninguem for responsavel** - nao arrancam conversa das maos de um atendente | pelo menos 1 time cadastrado |
| `list_assistants`, `transfer_to_agent` | Descobrir e transferir para outro Agente de IA, ou se desativar. Maximo 3 transferencias seguidas | pelo menos 2 agentes na conta |
| `get_products` | Consultar catalogo, planos e precos | ids em `config.offer_ids` do agente |
| `list_whatsapp_templates` | Descobrir os modelos aprovados e usaveis na caixa daquela conversa | caixa WhatsApp Oficial com pelo menos 1 modelo aprovado que sirva para a IA. Nao ha liberacao na mao: o sistema descarta sozinho o modelo com variavel sem mapeamento, cabecalho de midia ou botao dinamico |
| `get_flow_result` | Puxar o resultado completo de um fluxo ou consulta externa que ja rodou naquela conversa, em vez de rodar de novo | ter ferramenta de fluxo ou de API cadastrada |

---

## 4. Grupo 3 - escondidas

O cliente nao liga nem desliga. Elas seguem o conteudo ou a acao correspondente.

| Identificador | Carrega quando | Observacao |
|---|---|---|
| `faq_lookup` | ha base de conhecimento ou pergunta frequente aprovada | O comportamento padrao obriga a cascata: busca na base PRIMEIRO |
| `search_articles` | ha artigos publicados na Central de Ajuda | so entra se a busca na base voltou vazia |
| `list_labels` | sempre | le a DESCRICAO da etiqueta - e o que faz a IA etiquetar certo sem nome chumbado no cenario |
| `list_media_assets` | ha arquivo liberado E ela pode enviar | some se `send_media_asset` estiver desligada |
| `list_available_funnels` | ha funil E ela pode criar ou mover card | idem |
| `list_teams` | ha time E ela pode mover de time | de proposito: ler a lista sem poder agir faria ela prometer roteamento |
| `schedule_self_callback` | sempre | a IA agenda a propria volta para cumprir uma promessa ("ja te retorno"). Espera de 10 a 50 segundos, no maximo 1 pendente por conversa |

Alem da busca acionada, a base de conhecimento tambem e consultada por semelhanca em TODA mensagem, sem a
IA precisar pedir. Por isso base bem escrita vale mais que instrucao grande.

---

## 5. As cinco que NUNCA podem ser citadas num cenario

Estas cinco existem e rodam, mas **nao estao no catalogo que valida a mencao**. Escrever
`[Ver quem esta livre](tool://check_agent_availability)` dentro de um cenario faz o salvamento voltar erro
dizendo que o texto cita ferramenta invalida:

- `check_agent_availability`
- `get_flow_result`
- `lookup_media_content`
- `list_labels`
- `schedule_self_callback`

Elas carregam sozinhas quando fazem sentido. **Nao cite nenhuma delas.** Se precisar que a IA consulte
disponibilidade ou releia um arquivo, escreva o passo em portugues, sem a mencao.

---

## 6. Ferramentas proprias

### De API (consultar um sistema do cliente)

Deixa a IA chamar uma API externa (status do pedido, estoque, ficha no ERP) e usar a resposta.

- Criar: `lionchat_captain_assistants_create_5`. Testar antes: `lionchat_captain_custom_tools_test`.
- Campos obrigatorios: `title` (o nome vira o identificador `custom_<nome>`, que e como voce cita a
  ferramenta num cenario), `description` (e o que a IA le para decidir QUANDO chamar), `endpoint_url`,
  `http_method`.
- `param_schema`: lista de `{name, type, description, required}`. O tipo so aceita `string`, `integer`,
  `number` ou `boolean` - qualquer outra coisa e recusada no cadastro.
- `auth_type`: `none`, `bearer`, `basic` ou `api_key`, com as credenciais em `auth_config`.
- `request_template` e `response_template` aceitam variaveis - confira os nomes com
  `lionchat_captain_liquid_variables_list` antes.
- **Ela e da CONTA, nao do agente**: toda ferramenta de API ligada entra em TODOS os agentes
  automaticamente. Para tirar de um agente so, ponha `custom_<id>` em `config.disabled_tools` dele.
- Enderecos internos sao bloqueados por seguranca.

### De fluxo (tarefa de varias etapas)

Um fluxo desenhado no editor de fluxos que vira uma funcao da IA: ela aciona, o fluxo roda na hora e
devolve um resultado estruturado.

- Criar pela porta PROPRIA: `lionchat_flow_tools_create`. **Nunca por `lionchat_flows_create`** - o tipo do
  fluxo nao pode ser mudado depois, e um fluxo criado pela porta errada nunca vira ferramenta.
- Antes de montar o desenho, leia o esquema com `lionchat_flows_schema_reference` (se o seu conector nao
  tiver essa ferramenta, peca o formato ao cliente em vez de chutar).
- `tool_name`: minusculas, comeca com letra, so letras, numeros e sublinhado, ate 50. Unico na conta e nao
  pode colidir com ferramenta nativa nem de API.
- `tool_description`: ate 500 letras dizendo QUANDO acionar. Seja especifico.
- Sempre desenhe um bloco de Fim: e ele que define o que volta para a IA. Sem nenhum, a IA nao recebe
  resultado. Blocos que ESPERAM resposta sao proibidos (o salvamento recusa) e nao pode ter caixa de
  entrada vinculada (tambem recusado).
- Teste com `lionchat_flow_tools_run` ANTES de vincular.
- Vincular: `lionchat_flow_tools_assistants_update`. **Esse endereco SUBSTITUI a lista inteira** - leia a
  lista atual com `lionchat_flow_tools_assistants_list` e mande a lista completa, senao voce desvincula a
  ferramenta de todos os outros agentes em silencio.
- Para citar num cenario, o identificador e `flow_<id do fluxo>` (o id que `lionchat_flow_tools_list`
  devolve). **A listagem de ferramentas do conector nao mostra as de fluxo** - nao adianta procurar la.
- Deu errado? `lionchat_flow_tools_executions_list` mostra o historico de execucoes.

---

## 7. Como desligar uma ferramenta de um agente

`config.disabled_tools` no agente, um array com os identificadores:

- nativa: o identificador cru (`resolve_conversation`)
- de API: `custom_<numero do registro da ferramenta>` (o id que a listagem de ferramentas de API devolve,
  nao o nome). Atencao: o identificador para CITAR num cenario e outro - e o que aparece em
  `lionchat_captain_assistants_tools`, no formato `custom_<nome>`
- de fluxo: `flow_<id do fluxo>`

Precisa ser array. Lembre que `config` faz mistura parcial no update (mandar so `disabled_tools` preserva o
resto), **mas array dentro de `config` e substituido inteiro** - mandar `disabled_tools` de novo apaga o
que estava la antes. Leia o valor atual primeiro.

Casos comuns de desligar:

- `resolve_conversation` - quando o cliente nao quer que a IA encerre sozinha (e quando encerrar dispararia
  a pesquisa de satisfacao sem querer).
- `assign_agent` ou `assign_team` - quando a distribuicao e feita por automacao e a IA nao deve mexer.
- `schedule_message` - quando o cliente nao quer mensagem futura assinada pela IA.

---

## 8. A pegadinha do "salvou mas nao carrega"

Esta e a armadilha silenciosa mais cara do catalogo.

A validacao do cenario confere a mencao contra a lista de ferramentas do CATALOGO (as nativas do sistema,
mais as de API e as de fluxo vinculadas). Ela **nao** confere se o recurso da conta existe. Entao:

- Escrever `[Agendar](tool://create_booking)` numa conta SEM agenda **salva com sucesso**.
- Na hora do atendimento, a ferramenta nao carrega.
- A IA le o passo a passo mandando agendar, nao tem como agendar, e escreve que agendou.

O mesmo vale para card do funil sem funil, envio de arquivo sem arquivo liberado, roteamento por time sem
time e catalogo sem produto liberado.

**Regra pratica:** antes de escrever um cenario que use ferramenta condicional, confirme que o recurso
existe (`lionchat_funnels_list`, `lionchat_booking_event_types_list`, `lionchat_teams_list`,
`lionchat_offers_list`, `lionchat_captain_media_assets_list`) e que o agente tem os ids liberados. Se nao
existir, ou voce cria o recurso antes, ou voce reescreve o cenario sem aquela promessa.
