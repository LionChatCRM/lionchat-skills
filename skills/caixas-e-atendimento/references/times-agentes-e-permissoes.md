# Times, agentes, cargos e disponibilidade

Indice:
1. Membros da caixa: agente x supervisor
2. Convidar e manter agentes
3. Convite ainda nao aceito: os vinculos ficam pendentes
4. Trocar um atendente por outro
5. Times
6. Cargos personalizados: os 20 poderes
7. Como as permissoes de conversa se somam
8. Cargos prontos
9. Grade de disponibilidade do agente
10. Limites do plano

---

## 1. Membros da caixa: agente x supervisor

A lista de membros decide QUEM enxerga as conversas daquela caixa. Quem nao esta na lista
simplesmente nao ve aquelas conversas, nem no Kanban. Administrador enxerga todas as caixas mesmo sem
estar na lista.

Duas listas na mesma operacao:

| Lista | O que significa |
|---|---|
| `user_ids` | Agentes: veem a caixa E entram na distribuicao automatica |
| `supervisor_ids` | Supervisores: veem TUDO da caixa e ficam FORA do sorteio |

**Use sempre `lionchat_inbox_members_update`.** A ferramenta de criar (`lionchat_inbox_members_create`)
tem descricao enganosa: apesar do texto dizer que substitui a lista e citar supervisores, na pratica
ela apenas ACRESCENTA quem falta e ignora a lista de supervisores - todo mundo entra como agente
normal. Quem substitui de verdade e entende supervisor e a de atualizar.

`lionchat_inbox_members_update` recebe a lista COMPLETA: quem nao estiver em `user_ids` nem em
`supervisor_ids` e REMOVIDO da caixa. Mandar "so o novo" tira todo mundo que ja estava.

Mande a lista inteira numa unica chamada, nunca um agente por vez em laco. Adicionar muita gente em
sequencia ja deixou o painel fora do ar: cada gravacao avisa todos os paineis logados da conta e
eles rebuscam a lista de caixas.

Para ler o que esta valendo: `lionchat_inbox_members_show`.

---

## 2. Convidar e manter agentes

`lionchat_agents_create` convida uma pessoa; `lionchat_agents_bulk_create` convida varias de uma vez
(so a lista de e-mails). Campos:

| Campo | Valores |
|---|---|
| `email` | obrigatorio; e para onde vai o convite |
| `name` | nome exibido |
| `role` | `agent` ou `administrator`. Administrador ve todas as caixas e todos os ajustes |
| `availability` | `online`, `busy` ou `offline`. Ocupado nao recebe automatico |
| `auto_offline` | privilegio "permanecer disponivel". Padrao ligado (a pessoa cai para offline sozinha quando fecha o painel). Desligar faz ela contar como disponivel 24 horas |
| `custom_role_id` | cargo personalizado; use junto com `role: agent` |
| `team_ids` | lista COMPLETA dos times |
| `inboxes` | lista COMPLETA das caixas, cada uma com o cargo: `[{inbox_id, supervisor}]` |

`lionchat_agents_update` altera. Cuidados:

- **`team_ids` e `inboxes` reconciliam**: o agente SAI de todo time e de toda caixa que nao estiver na
  lista enviada. Se voce so quer acrescentar, leia o que ele ja tem e reenvie tudo.
- **Se voce enviar `custom_role_id` vazio numa alteracao parcial, o cargo personalizado e APAGADO.**
  Omita o campo quando nao for para mexer nele.
- Omitir `team_ids` e `inboxes` por completo nao mexe em nada - e o comportamento seguro.

Outras ferramentas uteis: `lionchat_agents_list` (quem existe), `lionchat_agents_assignments` (a
carteira da pessoa), `lionchat_agents_assigned_resources` (o que ficaria orfao se ela sair) e
`lionchat_agents_resend_invitation`.

---

## 3. Convite ainda nao aceito: os vinculos ficam pendentes

Se voce convidar alguem ja informando times e caixas, e a pessoa ainda NAO confirmou o e-mail, nada e
gravado como membro: fica guardado como pendente e so passa a valer quando ela confirmar.

Consequencia pratica: `lionchat_inbox_members_show` NAO mostra essa pessoa, ela nao entra no rodizio,
e a conferencia final vai parecer que a operacao falhou. **Nao refaca a operacao.** Explique ao
cliente que a pessoa precisa aceitar o convite no e-mail e que os acessos entram sozinhos depois
disso.

Para provar isso ao cliente, use `lionchat_agents_assignments`: ela mostra os vinculos pendentes que
a lista de membros da caixa ainda nao enxerga.

---

## 4. Trocar um atendente por outro

`lionchat_agents_replace` cria um atendente NOVO, transfere para ele a carteira inteira de quem sai
(conversas, cards, times e caixas com o mesmo cargo, incluindo vinculos pendentes) e enfileira a
exclusao do antigo. Funciona mesmo com a conta no limite de vagas.

E IRREVERSIVEL e exige `confirm: true` na chamada. Nunca chame sem uma confirmacao explicita e
especifica do cliente, dizendo quem sai e quem entra.

---

## 5. Times

`lionchat_teams_create` e `lionchat_teams_update`. Campos e comportamento de distribuicao estao em
`distribuicao.md`.

Membros: `lionchat_team_members_update` com a lista COMPLETA de `user_ids`. Quem nao estiver na lista
sai do time. `lionchat_teams_list_2` le os membros atuais.

Antes de tirar alguem de um time, `lionchat_team_members_kanban_reassignment_preview` mostra quais
cards do Kanban ele perderia, e
`lionchat_team_members_remove_agents_with_reassignment` remove ja reatribuindo os cards. Use a previa
sempre - remover sem ela deixa cards sem dono.

Lembre da regra que mais confunde: estar no time NAO da acesso a caixa. A pessoa precisa estar nas
duas listas.

Marcar supervisor de time nao e possivel pelas ferramentas: e no painel.

---

## 6. Cargos personalizados: os 20 poderes

Dao poderes extras a um agente sem promove-lo a administrador. Na tela ficam em Configuracoes >
Funcoes Personalizadas. Dependem de um recurso de plano; se nao estiver liberado, a criacao e
recusada.

`lionchat_custom_roles_create` recebe nome, descricao e a lista de poderes. Nome invalido na lista
faz a criacao ser recusada. A lista fechada e:

| Poder | O que libera, em linguagem de negocio |
|---|---|
| `conversation_manage` | Ver e responder todas as conversas as quais tem acesso |
| `conversation_unassigned_manage` | Ver as proprias mais as que estao sem responsavel |
| `conversation_participating_manage` | Ver as proprias mais aquelas em que foi incluido |
| `conversation_team_manage` | Ver as proprias, aquelas em que participa e as do time dele |
| `conversation_delete` | Apagar conversa |
| `contact_manage` | Gerenciar contatos |
| `contact_export_import` | Exportar e importar contatos |
| `report_manage` | Ver e gerenciar relatorios |
| `knowledge_base_manage` | Gerenciar a central de ajuda |
| `kanban_view` | Ver o Kanban em modo acompanhamento (edita so os proprios cards) |
| `kanban_manage` | Gerenciar cards do Kanban |
| `kanban_delete` | Apagar cards do Kanban |
| `kanban_export_import` | Exportar e importar cards |
| `funnel_manage` | Gerenciar funis |
| `offer_manage` | Gerenciar ofertas |
| `campaign_manage` | Gerenciar campanhas |
| `automation_manage` | Gerenciar automacoes e fluxos |
| `marketing_integrations_manage` | Gerenciar integracoes de marketing |
| `captain_manage` | Gerenciar o agente de IA |
| `flowbuilder_manage` | Liberar as pecas sensiveis do fluxo (ferramenta de IA e registro de execucao). O editor visual de fluxo ja e aberto ao atendente |

O cargo personalizado e sempre aplicado junto com o papel de agente, e so limita DENTRO das caixas
das quais a pessoa ja e membro. Ele nao da acesso a caixa nenhuma.

---

## 7. Como as permissoes de conversa se somam

As quatro permissoes de conversa **SOMAM**, nao restringem uma a outra. Marcar "nao atribuidas" e "do
meu time" ao mesmo tempo entrega os dois conjuntos, nao a intersecao.

Nao marcar nenhuma das quatro deixa a pessoa praticamente cega: so enxerga as conversas em que ela
foi colocada como participante, as que estao num card do Kanban dela e as de caixa ja excluida. E a
regra do produto e simples: quem VE a conversa PODE responder.

---

## 8. Cargos prontos

A tela oferece quatro atalhos, que sao apenas conjuntos de poderes ja marcados: Vendedor, Gerente
Comercial, Operador de Marketing e Gestor de Automacoes. Use-os como ponto de partida quando o
cliente nao souber dizer exatamente o que quer liberar.

---

## 9. Grade de disponibilidade do agente

Faixas semanais de horario em que o agente esta disponivel. **Nao e o status online/offline** e nao
tem nada a ver com receber conversa: serve para a Agenda e para responder "quem esta livre as 14h".

- `lionchat_agent_availability_list` le a grade.
- `lionchat_agent_availability_update` substitui a grade INTEIRA (apaga tudo e recria com o que voce
  mandar) e grava SEMPRE para o usuario logado, nunca para outra pessoa. Cada faixa tem dia da semana
  (0 e domingo) e horario de inicio e fim.
- `lionchat_agent_availability_list_1` confere se alguem esta disponivel num horario.
- `lionchat_agent_availability_list_2` lista quem esta livre.

O fuso usado para interpretar esses horarios e o do proprio agente, no perfil dele.

---

## 10. Limites do plano

`lionchat_account_show` traz os limites da conta. Duas recusas aparecem como erro de limite (402) e
nao como erro de campo:

- Criar caixa acima do teto de caixas do plano.
- Convidar agente acima do teto de agentes do plano.

Nesses casos nao insista nem tente outro caminho: avise o cliente que o plano precisa ser ajustado e
siga com o resto da configuracao.

Alguns recursos desta area dependem de estarem ligados na conta:

- **SLA e cargos personalizados**: o proprio servidor recusa a criacao com 403 quando o recurso nao
  esta ligado. Pule o item e reporte no resumo final - nao e erro de configuracao, e o plano.
- **Modo Equilibrado e politica de capacidade**: o bloqueio e de TELA. Confirme com
  `lionchat_account_show` que o recurso esta ligado ANTES de propor: montar por ferramenta algo que
  o cliente nao consegue ver nem corrigir no painel e pior do que nao montar.
