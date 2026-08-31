# Catalogo de campos do funil, da etapa, do cartao e da oferta

## Indice

1. Funil (o quadro)
2. Etapa (a coluna)
3. Metas do funil
4. Cartao (o negocio)
5. O miolo do cartao (`item_details`)
6. Oferta (catalogo de produtos e servicos)
7. Configuracao do Kanban da conta
8. Checklist do cartao
9. Anotacao do cartao
10. Responsaveis do cartao
11. Conversas ligadas ao cartao
12. Tarefas da agenda ligadas ao cartao
13. Aviso para sistema externo (webhook do Kanban)
14. Quem ve o que (permissoes e visibilidade)

---

## 1. Funil (o quadro)

| Campo | Obrigatorio | Valores | Observacao |
|---|---|---|---|
| `name` | sim | texto | UNICO na conta - criar com nome repetido falha |
| `description` | nao | texto | explicacao livre do processo |
| `active` | nao | verdadeiro/falso (padrao verdadeiro) | falso = arquivado: some das abas mas mantem todos os cartoes |
| `stages` | sim | objeto, uma chave por etapa | funil SEM nenhuma etapa e recusado na criacao. Ver secao 2 |
| `settings.agents` | nao | lista de pessoas `[{id, name}]` | **lista vazia = funil ABERTO a todos** |
| `settings.teams` | nao | lista de times `[{id, name}]` | os membros sao resolvidos ao vivo: mexeu no time, reflete na hora |
| `settings.goals` | nao | lista de metas | ver secao 3 |
| `settings.automations` | nao | lista de automacoes | ver `references/automacoes-do-funil.md` |
| `position` | nao | numero | ordem das abas de funil na tela |

**O bloco `settings` inteiro e substituido a cada gravacao.** Gravar so uma parte apaga as outras, sem erro.

Time vinculado SEM membros conta como "ninguem" e deixa o funil aberto do mesmo jeito. Verifique a lista efetiva, nao a lista escrita.

Existe ainda um campo de atributos globais no proprio funil, herdado de uma versao antiga: **nenhuma tela do cartao le esse campo.** Os campos do cartao que aparecem vem da configuracao do Kanban da conta (secao 7).

## 2. Etapa (a coluna)

As etapas sao um objeto, e a **chave** de cada uma e o identificador real:

```
"stages": {
  "novo_lead":  { "name": "Novo Lead",  "color": "#3B82F6", "position": 1 },
  "proposta":   { "name": "Proposta",   "color": "#F59E0B", "position": 2 }
}
```

| Campo | Obrigatorio | Valores | Observacao |
|---|---|---|---|
| a chave | sim | texto curto, minusculo, sem acento e sem espaco | **e o identificador para sempre.** Automacao, filtro, relatorio e importacao usam ela |
| `name` | sim | texto | e o que aparece no topo da coluna. Renomear NAO mexe nos cartoes |
| `position` | sim | numero a partir de 1 | ordem das colunas da esquerda para a direita |
| `color` | nao | cor em hexadecimal, ex.: `#3B82F6` | faixa colorida da coluna |
| `description` | nao | texto | vira o subtitulo do relatorio da etapa |
| `message_templates` | nao | lista de modelos | modelos de mensagem da etapa - **nao existe ferramenta dedicada; escreva pela tela** |

Regras:

- **A chave nunca deve ser reaproveitada.** Etapa apagada deixa a chave queimada.
- **Renomear a etapa e seguro; trocar a chave nao.** Trocar a chave e o mesmo que apagar uma etapa e criar outra.
- **Apagar etapa com cartoes e recusado.** Mova os cartoes antes com `lionchat_funnels_create_2` (transferir cartoes entre etapas, aceita inclusive outro funil de destino).

## 3. Metas do funil

Cada meta e `{id, type, value, unit, period, currency, description}`.

| Tipo | Unidade | Responde |
|---|---|---|
| `conversion_rate` | `percentage` | taxa de conversao do funil |
| `average_value` | `currency` | ticket medio |
| `average_time` | `days` | tempo medio ate fechar |
| `total_conversions` | `count` | quantos negocios fechados |
| `total_revenue` | `currency` | quanto de receita |

Periodo: `monthly`, `quarterly` ou `yearly`. Metas com valor em branco sao descartadas no salvamento pela tela.

## 4. Cartao (o negocio)

| Campo | Obrigatorio | Valores | Observacao |
|---|---|---|---|
| `funnel_id` | sim | numero do funil | |
| `funnel_stage` | sim | **a chave** da etapa | etapa inexistente e redirecionada em silencio - confira a resposta |
| `position` | sim | numero | ordem dentro da coluna |
| `item_details.title` | sim | texto | e o nome do negocio no cartao |
| `conversation_display_id` | nao | numero da conversa | e por ela que o cartao sabe quem e o cliente - **o cartao nao guarda o contato** |
| `linked_conversations` | nao | lista de objetos `[{display_id: 123}]` | **nunca numeros soltos** - numero solto quebra a leitura do cartao |
| `assigned_agents` | nao | lista de pessoas | define tambem quem ENXERGA o cartao |
| `checklist` | nao | lista de tarefas | **fora do salvamento comum** - mexer so pelas ferramentas proprias |
| `activities` | nao | historico | escrito so pelo sistema, tambem fora do salvamento comum |
| `stage_entered_at` | nao | data e hora | quando o cartao entrou na etapa ATUAL; base do tempo na etapa |
| `timer_started_at` / `timer_duration` | nao | data e hora / segundos | cronometro de trabalho no cartao |

Dois avisos importantes sobre a conversa do cartao:

- **O sistema move sozinho o ponteiro da conversa principal** para a conversa mais recente daquela pessoa naquela caixa. Nao trate como fixo.
- **Num salvamento comum, mandar um numero de conversa cheio e DESCARTADO de proposito** - se nao fosse, o numero antigo sobrescreveria o novo. So o valor nulo passa, e ele significa "desvincular".

## 5. O miolo do cartao (`item_details`)

| Campo | Valores | Observacao |
|---|---|---|
| `title` | texto | obrigatorio |
| `description` | texto | |
| `status` | `open`, `won`, `lost` | ausente ou vazio conta como aberto |
| `priority` | `urgent`, `high`, `medium`, `low`, `none` | vira a bolinha colorida no cartao |
| `value` | numero | **recalculado como a soma das ofertas antes de salvar**. Escrever a mao so vale em cartao sem oferta |
| `currency` | `"BRL"` ou objeto com codigo, simbolo e formato | |
| `offers` | lista | e a FOTO congelada da oferta no momento em que ela entrou no cartao |
| `closed_offers` | lista | quais ofertas realmente fecharam quando o cartao virou Ganho |
| `win_reason` / `loss_reason` | texto vindo das listas da conta | a janela tambem deixa digitar um motivo avulso |
| `attributed_to` | numero da pessoa | a quem creditar o ganho quando o cartao tem dois ou mais responsaveis |
| `status_changed_at` / `status_changed_by` | data e hora / pessoa | carimbados SO quando o estado MUDA, e **apagados ao reabrir para aberto** |
| `deadline_at` | data e hora | prazo; muda de cor no cartao quando perto ou vencido |
| `scheduled_at` / `scheduling_type` | data e hora / texto | usados pelo filtro de data agendada |
| `custom_attributes` | **LISTA** `[{name, type, value}]` | e o que a aba Dados Adicionais mostra. Ver `references/onde-guardar-o-dado.md` |
| `notes` | lista | mexer pelas ferramentas proprias de anotacao |
| `duplicated_from_id` | numero do cartao de origem | marca que o cartao nasceu de uma duplicacao |
| `tracking_attributes` | so leitura | a origem do trafego (anuncio, campanha, endereco de origem), copiada da conversa quando o cartao nasce. **Nao da para escrever** e cartao duplicado mantem a foto do original |

## 6. Oferta (catalogo de produtos e servicos)

| Campo | Obrigatorio | Valores |
|---|---|---|
| `title` | sim | nome da oferta |
| `value` | sim | numero maior que zero. Mandando variacoes de preco, o sistema preenche sozinho com o MENOR delas |
| `currency` | **sim** | `BRL` (tambem aceita outras moedas). Nao tem valor padrao: sem ele a criacao e RECUSADA |
| `type` | **sim** | `product` ou `service`. Tambem nao tem padrao: sem ele a criacao e RECUSADA |
| `description` | nao | ate 1000 caracteres - **e o texto que o agente de IA le quando o cliente pergunta preco** |
| `prices` | nao | variacoes ou planos: `[{label, value, currency, description, checkout_url}]` |
| `links` | nao | links extras: `[{label, url}]` |
| `image` | nao | foto - so pela tela (aceita JPEG ou PNG, ate 5 MB) |

**Os QUATRO primeiros sao obrigatorios de verdade** (nome, valor, moeda e tipo). Faltando qualquer um, a
oferta nao e criada e volta recusa - nao existe preenchimento automatico de moeda nem de tipo. O valor e a
unica excecao: mandando `prices`, ele se preenche sozinho.

**Regra que nao muda: editar ou excluir a oferta NAO altera cartoes que ja a receberam.** A oferta dentro do cartao e uma foto congelada; o preco novo so vale para cartoes criados dali para frente. Isso e regra de negocio, nao defeito - para corrigir um cartao antigo e preciso trocar a oferta dentro daquele cartao.

E o mesmo catalogo que o agente de IA usa como lista de produtos: basta selecionar as ofertas nas configuracoes do assistente.

## 7. Configuracao do Kanban da conta

Vale para a conta inteira e para TODOS os funis. Leia com `lionchat_kanban_config_list` e grave com `lionchat_kanban_config_update`.

| Lista | Formato | Observacao |
|---|---|---|
| `win_reasons` | `[{id, title}]` | motivos de ganho |
| `loss_reasons` | `[{id, title}]` | motivos de perda |
| `checklist_templates` | `[{id, name, items: [{id, text}]}]` | o `name` vira o titulo do bloco no cartao. Renomear depois nao muda os cartoes ja aplicados |
| `global_custom_attributes` | `[{name, type, is_list, list_values: []}]` | os campos do cartao. Tipos da tela: `string`, `number`, `date`, `boolean` (a exibicao tambem trata `time`). **Nao existe o tipo "lista"** - lista e o modo `is_list` com as opcoes em `list_values` |
| `config` | preferencias gerais | esta e a unica parte que se MISTURA com o que ja existe; as listas acima sao substituidas inteiras |

Sobre as preferencias (`config`): as duas que valem de verdade sao o **titulo do modulo** (o nome que
aparece no menu lateral no lugar de "Kanban") e a **liberacao da visao em lista**. As outras chaves sao
gravadas e nao mudam nada na tela - **nao prometa comportamento a partir delas.** Em especial, nao existe
interruptor que faca o modelo de mensagem da etapa disparar sozinho.

## 8. Checklist do cartao

| Campo | Obrigatorio | Observacao |
|---|---|---|
| `text` | sim | o campo se chama `text`, nao "titulo" |
| `completed` | nao | verdadeiro ou falso; o campo se chama `completed`, nao "marcado" |
| `position` | nao | ordem na lista |
| `group_id` | nao | junta varios itens num bloco com barra de progresso propria; sem ele o item e avulso |
| `group_name` | nao | titulo do bloco; mande sempre junto com o `group_id` |

Para "aplicar um modelo" pelas ferramentas, crie item a item com o MESMO `group_id` e o mesmo `group_name`. Nao existe uma acao de aplicar modelo inteiro fora da automacao de etapa.

**Ler o checklist: use `lionchat_kanban_items_kanban_checklist_list`.** Nas outras leituras o campo volta como um RESUMO de contagem, sem nenhum texto de tarefa. E a soma dos blocos e MENOR que o total: tarefa avulsa conta no total geral e nao aparece na lista de blocos.

## 9. Anotacao do cartao

Texto puro, sem formatacao. Guarda autor e data. **Nao vai para o cliente** - e registro interno. Anotacao criada por automacao fica com autor "Sistema". Anexos de anotacao tem rota propria e, na pratica, sao enviados pela tela.

## 10. Responsaveis do cartao

Um cartao pode ter varios. Define tambem quem ENXERGA o cartao. A origem de cada atribuicao fica registrada: manual, herdada do dono da conversa, ou por automacao.

- A pessoa precisa ter acesso ao funil. Pelas ferramentas diretas, sem acesso da erro; pela automacao de etapa, e ignorado em silencio.
- **Cartao Ganho ou Perdido recusa acrescentar ou tirar responsavel.**
- Estar atribuido a um cartao libera abrir e responder AQUELA conversa mesmo sem ser membro da caixa. Nao libera a caixa inteira, e sai do cartao, perde o acesso.

## 11. Conversas ligadas ao cartao

Alem da conversa principal, o cartao aceita conversas adicionais (a pessoa falou no WhatsApp e tambem por e-mail). Cada conversa adicional tem um interruptor de "recebe automacao", que decide se ela recebe as mensagens automaticas do cartao.

Ferramentas: vincular, desvincular e ligar ou desligar o "recebe automacao" - ver `references/ferramentas-mcp.md`.

## 12. Tarefas da agenda ligadas ao cartao

O elo entre tarefa e cartao e a CONVERSA - nao existe tarefa presa diretamente ao cartao. Sem conversa ligada, o cartao nao tem como puxar tarefa nenhuma. A proxima tarefa aparece no cartao do quadro. Quem executa controla tambem quem VE a tarefa (administrador ve todas).

## 13. Aviso para sistema externo (webhook do Kanban)

| Campo | Observacao |
|---|---|
| `webhook_url` | endereco que recebe os avisos |
| `webhook_secret` | segredo para assinar o envio |
| `enabled` | sem endereco, o aviso nao e considerado ligado |
| `webhook_events` | lista fechada de acontecimentos |

Acontecimentos disponiveis: cartao criado, alterado, excluido, mudou de etapa, cartoes reordenados, responsavel atribuido ou removido, anotacao criada/alterada/excluida, item de checklist criado/alterado/excluido/marcado, cronometro iniciado ou parado, estado alterado, e as tres acoes em massa (movidos, atribuidos, prioridade alterada).

Existe um disparo de teste (`lionchat_kanban_config_create_1`) e um registro das tentativas de entrega dos ultimos 7 dias, so para administrador (`lionchat_kanban_deliveries_list`).

## 14. Quem ve o que (permissoes e visibilidade)

Permissoes de cargo, todas separadas:

| Permissao | Da o que |
|---|---|
| `funnel_manage` | criar, editar, excluir e arquivar funis e etapas |
| `kanban_view` | ve TODOS os cartoes da conta, mas so edita os proprios |
| `kanban_manage` | ve e edita qualquer cartao |
| `kanban_delete` | excluir cartao - **nao vem junto com o `kanban_manage`** |
| `kanban_export_import` | baixar e subir a planilha do funil - **tambem separada** |
| `offer_manage` | mexer no catalogo de ofertas |

Regra de visibilidade do CARTAO, em ordem:

1. Quem tem visao ampla (administrador, `kanban_view` ou `kanban_manage`) ve tudo.
2. O responsavel ve os cartoes dele.
3. Cartao SEM responsavel e visto por quem tem acesso ao funil.

Alem disso, um vendedor comum so ve cartao cuja conversa esteja numa caixa de entrada que ele acessa - com a excecao do cartao sem responsavel em funil acessivel.

**Efeito colateral que confunde:** quando alguem assume a conversa, o cartao ganha dono automaticamente e SOME da tela dos colegas. E o comportamento desejado, mas parece que o cartao desapareceu.
