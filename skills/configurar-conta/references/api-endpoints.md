# LionChat API — Endpoints para Configuracao

**Base URL:** `https://app.lionchat.com.br`

**Autenticacao:** header `api_access_token: SEU_TOKEN` em toda chamada.

**Content-Type:** `application/json` em POST/PATCH/PUT.

**IMPORTANTE — WRAP KEYS:** parte dos endpoints do LionChat espera o payload **embrulhado** em uma chave
com o nome do recurso (ex: `{"label": {...}}`); outros esperam o payload **plano**. Cada endpoint abaixo diz
qual e o caso. Errar isso NAO devolve 400 — devolve **422** com a mensagem
`param is missing or the value is empty: <recurso>`, ou, pior, cria o registro pela metade.

**Endpoints que EXIGEM wrap:** `labels`, `canned_responses`, `teams`, `funnels`, `sla_policies`,
`kanban_config`, `account_variables`.

**Endpoints PLANOS (sem wrap):** `accounts`, `custom_attribute_definitions`, `automation_rules`, `macros`,
`inboxes`.

> `automation_rules` e o caso mais perigoso: mandar embrulhado **falha**, porque o controller le
> `name`, `conditions` e `actions` do nivel de cima. Ver secao 8.

---

## 0. Validar credenciais (SEMPRE fazer primeiro)

```bash
curl -s --request GET \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id} \
  --header 'api_access_token: SEU_TOKEN'
```

| Resposta | O que significa | O que dizer ao cliente |
|----------|-----------------|------------------------|
| 200 | Token e conta OK | segue |
| 401 | Token invalido, expirado, ou o usuario nao tem permissao | "Confere o API token em Configuracoes > Perfil" |
| 404 | A conta nao existe **OU** seu usuario nao tem acesso a ela | "Confere o numero da conta na URL do painel — e se voce e mesmo membro dessa conta" |

**NAO existe 403 aqui.** O sistema propositalmente responde 404 para conta alheia (nao vaza a existencia de
contas de outros clientes). Entao 404 e ambiguo: pode ser numero errado **ou** falta de acesso — a mensagem
ao cliente precisa cobrir as duas hipoteses.

Para criar tudo que esta neste documento o usuario precisa ser **administrator** da conta.

---

## 1. Configuracoes gerais da conta

**Sem wrap key.**

```bash
curl -s --request PATCH \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{id} \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "Nome da Empresa",
    "locale": "pt_BR",
    "timezone": "America/Sao_Paulo",
    "auto_resolve_after": 10080
  }'
```

`auto_resolve_after`: minutos antes de resolver automaticamente conversa abandonada. 10080 = 7 dias.

---

## 2. Campos personalizados

O LionChat tem **3 tipos de atributos personalizados**, armazenados em 2 lugares diferentes:

| Tipo | Onde fica | Endpoint |
|------|-----------|----------|
| **Contato** (contact_attribute) | `custom_attribute_definitions` | POST `/custom_attribute_definitions` |
| **Conversa** (conversation_attribute) | `custom_attribute_definitions` | POST `/custom_attribute_definitions` |
| **Card do Kanban** (kanban_item_attribute) | `kanban_config.global_custom_attributes` | PUT `/kanban_config` |

### 2.1 Atributos de Contato e Conversa

**Sem wrap key.** (O Rails embrulha sozinho; se voce mandar embrulhado tambem funciona. Padronize no plano.)

`attribute_display_type`: `0`=text, `1`=number, `2`=currency, `3`=percent, `4`=link, `5`=date, `6`=list, `7`=checkbox.

`attribute_model`: `0`=conversation_attribute, `1`=contact_attribute (o valor `2`=account_attribute e reservado
pro sistema — nao usar).

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/custom_attribute_definitions \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "attribute_display_name": "Valor do Orcamento",
    "attribute_display_type": 2,
    "attribute_key": "valor_orcamento",
    "attribute_model": 0,
    "attribute_description": "Valor total proposto ao cliente"
  }'
```

Regras:
- `attribute_key` deve ser snake_case, sem acento, sem espaco (ha validacao no servidor: espaco e simbolo sao recusados).
- Para tipo `list` (6), adicionar `attribute_values: ["opcao1", "opcao2"]`.

### 2.2 Atributos de Card (Kanban)

**WRAP KEY OBRIGATORIO:** `{"kanban_config": {...}}`.

**ATENCAO — mecanismo diferente:** card attributes NAO usam `custom_attribute_definitions`. Sao armazenados
dentro de `kanban_config.global_custom_attributes` como um array JSONB. Cada item do array tem:

- `name` — nome de exibicao (string livre)
- `type` — ver tabela abaixo
- `is_list` — boolean, `true` se for lista de opcoes
- `list_values` — array de opcoes quando `is_list=true`, senao array vazio `[]`

**Mapa de tipos do que o cliente ve para o que se grava:**

| Tipo visual | type | is_list |
|-------------|------|---------|
| Texto | `string` | false |
| Numero | `number` | false |
| Link | `string` | false |
| Data | `date` | false |
| Hora | `time` | false |
| Data e hora | `datetime` | false |
| Lista | `string` | **true** (preencher list_values) |
| Caixa de selecao | `boolean` | false |

**Nao existem** `currency`, `percent`, `link` nem `checkbox` como valor de `type` — "Link" vira `string` e
"Caixa de selecao" vira `boolean`. O servidor **nao valida** o conteudo de `type` (so exige que `name` e `type`
estejam preenchidos), entao um tipo inventado e aceito e o campo aparece como texto comum na tela. Use so os
seis da tabela.

**Importante:** este endpoint **sobrescreve o array inteiro**. Para adicionar UM atributo novo, leia os
existentes primeiro, faca append no array e depois envie o PUT.

**Passo 1 — ler config atual:**

```bash
curl -s --request GET \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban_config \
  --header 'api_access_token: SEU_TOKEN'
```

Sempre responde **200**. Se a conta nunca teve configuracao de Kanban, o proprio GET cria uma com os padroes e
devolve ela — `global_custom_attributes` vem vazio (`[]`) ou ausente. **Nao existe 404 aqui**, e nao e preciso
POST: o PUT do passo 2 cria a configuracao se ela nao existir.

**Passo 2 — atualizar com o array completo (existentes + novos):**

```bash
curl -s --request PUT \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban_config \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "kanban_config": {
      "global_custom_attributes": [
        {
          "name": "Tipo de Procedimento",
          "type": "string",
          "is_list": true,
          "list_values": ["Implante", "Canal", "Limpeza", "Avaliacao"]
        },
        {
          "name": "Valor Orcamento",
          "type": "number",
          "is_list": false,
          "list_values": []
        },
        {
          "name": "Data da Consulta",
          "type": "date",
          "is_list": false,
          "list_values": []
        }
      ]
    }
  }'
```

### 2.3 Motivos de ganho e de perda

Ainda no `kanban_config`. Quando o vendedor marca um card como ganho ou perdido, o sistema pergunta o
MOTIVO — e quem define a lista de opcoes e voce. Sem lista cadastrada, a pergunta aparece sem opcao e o
relatorio de motivos fica vazio: o cliente sabe que perdeu 40 negocios e nao sabe por que.

Duas listas separadas, `win_reasons` e `loss_reasons`. Cada item: `{ "id": "<voce escolhe>", "title": "Texto que o vendedor ve" }`.

```bash
curl -s --request PUT \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban_config \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "kanban_config": {
      "win_reasons": [
        { "id": "preco_bom",    "title": "Preco/condicao" },
        { "id": "indicacao",    "title": "Veio por indicacao" },
        { "id": "urgencia",     "title": "Precisava com urgencia" }
      ],
      "loss_reasons": [
        { "id": "preco",        "title": "Achou caro" },
        { "id": "concorrente",  "title": "Fechou com concorrente" },
        { "id": "sem_retorno",  "title": "Sumiu, nao respondeu" },
        { "id": "sem_perfil",   "title": "Nao era o perfil" }
      ]
    }
  }'
```

**Mesma regra do array inteiro:** leia os motivos existentes antes e mande a lista completa de volta.

Escreva os motivos com as palavras que o cliente usou na entrevista — "achou caro" e melhor que
"objecao de preco". E prefira poucos: lista com 15 motivos ninguem preenche direito, e o relatorio
fica pulverizado.

### 2.4 Modelos de checklist (usados pela automacao de etapa)

Ainda no `kanban_config`, o campo `checklist_templates` guarda listas de tarefas prontas que podem ser
aplicadas a um card — na mao ou por automacao de etapa (acao `apply_checklist_template`, secao 7).

Cada modelo: `{ "id": "<voce escolhe>", "name": "Nome do checklist", "items": [{"text": "tarefa"}] }`.
O servidor exige `name` preenchido e `items` como array.

```bash
curl -s --request PUT \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban_config \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "kanban_config": {
      "checklist_templates": [
        {
          "id": "checklist_proposta",
          "name": "Enviar proposta",
          "items": [
            { "text": "Confirmar dados do paciente" },
            { "text": "Montar orcamento" },
            { "text": "Enviar PDF pelo WhatsApp" },
            { "text": "Agendar retorno em 48h" }
          ]
        }
      ]
    }
  }'
```

Guarde o `id` do modelo: e ele que vai em `action_config.template_id` da automacao de etapa.

**Mesma regra do array inteiro:** leia os `checklist_templates` existentes antes e mande a lista completa.

---

## 3. Tags (Labels)

**WRAP KEY OBRIGATORIO:** `{"label": {...}}`

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/labels \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "label": {
      "title": "fonte-instagram",
      "description": "Lead veio do Instagram",
      "color": "#E91E63",
      "show_on_sidebar": true
    }
  }'
```

Regras do `title` (validadas no servidor):
- E **forcado para minusculo** automaticamente.
- **Espaco NAO e aceito** — use hifen ou underscore.
- **Acento E aceito** (`promocao` com cedilha passa). Mesmo assim, prefira sem acento: fica mais facil de
  digitar no filtro e nas automacoes.
- Minimo de **2 caracteres**, e o primeiro tem que ser letra ou numero (nao pode comecar com hifen/underscore).
- Paleta sugerida (use cor diferente por categoria):
  - Fontes: `#1976D2`, `#1565C0`, `#0D47A1`
  - Objecoes: `#E53935`, `#C62828`, `#B71C1C`
  - Status positivo: `#43A047`, `#2E7D32`
  - Prioridade: `#F57C00`, `#E65100`
  - Neutro: `#616161`, `#424242`

**Listar antes de criar (evitar duplicata):**
```bash
curl -s --request GET \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/labels \
  --header 'api_access_token: SEU_TOKEN'
```

---

## 4. Respostas rapidas

**WRAP KEY OBRIGATORIO:** `{"canned_response": {...}}`

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/canned_responses \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "canned_response": {
      "short_code": "boasvindas",
      "content": "Ola {{contact.name}}! Obrigado pelo contato. Como posso ajudar?"
    }
  }'
```

Variaveis mais usadas no `content`:
- `{{contact.name}}`, `{{contact.first_name}}`, `{{contact.email}}`, `{{contact.phone}}`
- `{{agent.name}}`, `{{agent.first_name}}`
- `{{conversation.id}}`
- Atributo personalizado de contato: `{{contact.custom_attribute.CHAVE}}`
- Variavel da conta: `{{account.custom_attribute.CHAVE}}` (ver secao 12 — **o `custom_attribute` no meio e
  obrigatorio**)

Regras:
- `short_code` minusculo, sem espaco, sem acento. Se tiver 2 palavras, usar hifen: `follow-up-48h`.
- Minimo sugerido: `boasvindas`, `horariofora`, `aguarde`, `agradecimento`.

---

## 5. Times

**WRAP KEY OBRIGATORIO:** `{"team": {...}}`

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/teams \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "team": {
      "name": "Vendas",
      "description": "Time comercial",
      "allow_auto_assign": true
    }
  }'
```

`allow_auto_assign`: se `true`, o LionChat distribui conversas automaticamente entre agentes do time.

---

## 6. Funil (Kanban)

**WRAP KEY OBRIGATORIO:** `{"funnel": {...}}`

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/funnels \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "funnel": {
      "name": "Funil de Vendas",
      "description": "Processo comercial principal",
      "active": true,
      "stages": {
        "lead_novo": {
          "name": "Lead Novo",
          "color": "#3B82F6",
          "position": 1,
          "description": ""
        },
        "em_qualificacao": {
          "name": "Em Qualificacao",
          "color": "#F59E0B",
          "position": 2,
          "description": ""
        },
        "proposta_enviada": {
          "name": "Proposta Enviada",
          "color": "#8B5CF6",
          "position": 3,
          "description": ""
        },
        "fechado_ganho": {
          "name": "Fechado (Ganho)",
          "color": "#10B981",
          "position": 4,
          "description": ""
        }
      },
      "settings": {}
    }
  }'
```

Regras:
- As chaves das stages (ex: `lead_novo`) sao os identificadores internos da etapa. O painel gera essa chave
  fazendo slug do nome: minusculo, sem acento, tudo que nao e letra/numero vira `_`. Siga a mesma regra.
  **Guarde essas chaves** — sao elas que as automacoes de etapa referenciam.
- `name` da stage: o que o cliente ve na tela.
- Cores: use paleta coerente (azul no inicio, amarelo, roxo, verde/vermelho no fim).
- Sempre incluir etapa final "ganho" e, se fizer sentido, "perdido".
- Nenhum campo interno da etapa e obrigatorio alem do que esta acima.

**Retorno** contem `"id": N` — guarde o ID do funil.

### ATENCAO — `settings` e sobrescrito por inteiro

O campo `settings` do funil guarda, no mesmo lugar: `agents` (agentes do funil), `teams` (times),
`goals` (metas) e `automations` (automacoes de etapa, secao 7). Um PATCH que mande `settings` **substitui o
objeto inteiro** — o que voce nao mandar, some.

Portanto, para acrescentar automacoes a um funil que ja existe:

1. `GET /funnels/{id}` e leia o `settings` atual
2. acrescente/edite so a chave `automations`
3. `PATCH /funnels/{id}` com o `settings` **completo** de volta

Ao **criar** o funil (POST), o caminho e mais simples: como voce mesmo definiu as chaves das etapas, ja pode
mandar `settings.automations` junto no mesmo pedido.

---

## 7. Automacoes de etapa (dentro do funil)

**NAO existe endpoint proprio para automacao de etapa.** Nao use `/kanban/automations` nem
`/kanban_automations` — essas rotas foram removidas do sistema e hoje respondem 404.

Automacao de etapa vive **dentro do funil**, em `settings.automations` (array). Cada automacao:

```json
{
  "id": "automation_followup",
  "enabled": true,
  "trigger_type": "stage_moved",
  "trigger_value": "proposta_enviada",
  "action": "create_note",
  "action_config": { "note_text": "Proposta enviada — cobrar retorno" }
}
```

### Gatilhos (`trigger_type` + `trigger_value`)

| trigger_type | trigger_value | Dispara quando |
|--------------|---------------|----------------|
| `card_created` | `card_created` | um card novo entra no funil |
| `stage_moved` | a chave da etapa de **destino** (ex: `proposta_enviada`) | o card **chega** naquela etapa |
| `status_change` | `won` ou `lost` | o card e marcado como ganho ou perdido |

`enabled: false` desliga a automacao sem apagar.

### Acoes (`action` + `action_config`)

| action | action_config | O que faz |
|--------|---------------|-----------|
| `move_to_stage` | `{ "stage": "<chave da etapa>" }` | move o card para outra etapa |
| `assign_agent` | `{ "agent_id": N }` | atribui um atendente ao card |
| `create_note` | `{ "note_text": "texto" }` | escreve uma nota interna no card |
| `update_checklist` | `{ "checklist_updates": [...] }` | marca/desmarca itens do checklist do card |
| `notify_team` | `{ "message": "texto" }` | avisa o time |
| `duplicate_item` | `{ "funnel_id": N, "stage": "<chave>", "distribute_agents": true }` | cria uma copia do card em outro funil/etapa |
| `send_webhook` | `{ "webhook_url": "https://..." }` | dispara um webhook para um sistema externo |
| `apply_checklist_template` | `{ "template_id": "<id do modelo>" }` | aplica um checklist pronto ao card (secao 2.4) |

**Sao essas oito. Nao existem outras.** Em particular:

- **NAO existe `send_message`** — automacao de etapa **nao manda mensagem para o cliente**.
- **NAO existe `apply_label` / `add_note` / `assign_team`** — o mais proximo de nota e `create_note`.
- **NAO existe `delay_seconds`** — automacao de etapa e imediata, nao tem atraso.

Regras:
- Com o gatilho `stage_moved`, a acao `move_to_stage` e **ignorada** de proposito (evita loop infinito de
  card indo e voltando entre duas etapas).
- Uma automacao so vale se tiver `trigger_value` e `action` preenchidos. As acoes `move_to_stage`,
  `assign_agent`, `duplicate_item`, `send_webhook` e `apply_checklist_template` exigem seus campos de
  `action_config` — sem eles a automacao e descartada.

### Exemplo completo (criar funil ja com automacoes)

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/funnels \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "funnel": {
      "name": "Funil de Vendas",
      "active": true,
      "stages": {
        "lead_novo":        { "name": "Lead Novo",        "color": "#3B82F6", "position": 1 },
        "proposta_enviada": { "name": "Proposta Enviada", "color": "#8B5CF6", "position": 2 },
        "fechado_ganho":    { "name": "Fechado (Ganho)",  "color": "#10B981", "position": 3 }
      },
      "settings": {
        "automations": [
          {
            "id": "automation_checklist_proposta",
            "enabled": true,
            "trigger_type": "stage_moved",
            "trigger_value": "proposta_enviada",
            "action": "apply_checklist_template",
            "action_config": { "template_id": "checklist_proposta" }
          },
          {
            "id": "automation_nota_ganho",
            "enabled": true,
            "trigger_type": "status_change",
            "trigger_value": "won",
            "action": "create_note",
            "action_config": { "note_text": "Venda fechada — iniciar pos-venda" }
          }
        ]
      }
    }
  }'
```

### Como fazer follow-up com atraso (o que a automacao de etapa NAO faz)

"Mandar mensagem X horas depois que o card entrou na etapa" **nao e** automacao de etapa e **nao e** regra de
automacao global (a espera dela e limitada a 5 minutos — secao 8). Isso e trabalho do **FlowBuilder**:

1. o cliente cria um fluxo cujo gatilho de entrada e de card — `card_created`, `card_moved`, `card_won`,
   `card_lost` ou `card_attribute_changed`;
2. dentro do fluxo, um bloco de espera (horas/dias) e depois um bloco de mensagem.

Fluxo tem sessao propria e sustenta espera longa; automacao nao. **Nao prometa follow-up com atraso ao
cliente montando automacao — avise que essa parte se monta em Fluxos.**

---

## 8. Regras de automacao globais

**SEM WRAP KEY — payload PLANO.** Este endpoint le `name`, `event_name`, `conditions` e `actions` no nivel de
cima. Se voce mandar `{"automation_rule": {...}}`, o servidor nao enxerga nada e a criacao falha.

Eventos reais (`event_name`) — sao **oito**:

| event_name | Dispara quando |
|------------|----------------|
| `conversation_created` | nova conversa criada |
| `conversation_updated` | conversa foi alterada (aceita `action_types` pra afinar: etiqueta adicionada, status mudou, etc) |
| `conversation_opened` | conversa foi aberta |
| `conversation_resolved` | conversa foi resolvida |
| `message_created` | mensagem nova (recebida ou enviada) |
| `webhook` | chegou um evento no Webhook Universal |
| `kanban_item_created` | card novo criado no Kanban |
| `kanban_item_stage_changed` | card mudou de etapa |

**`conversation_status_changed` NAO existe** como evento de automacao (isso e evento de webhook de saida). O
equivalente e `conversation_updated` com `action_types: ["status_changed"]`.

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/automation_rules \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "Auto-atribuir ao time de vendas",
    "description": "Toda conversa nova da caixa 3 vai pro time comercial",
    "event_name": "conversation_created",
    "active": true,
    "conditions": [
      {
        "attribute_key": "inbox_id",
        "filter_operator": "equal_to",
        "values": [3],
        "query_operator": null
      }
    ],
    "actions": [
      {
        "action_name": "assign_team",
        "action_params": [2]
      }
    ]
  }'
```

`action_params` e **sempre array**, mesmo com um valor so.

Acoes disponiveis (`action_name`):

| Grupo | Acoes |
|-------|-------|
| Atendimento | `assign_agent`, `assign_team`, `assign_captain_assistant` (liga o agente de IA), `mute_conversation`, `snooze_conversation`, `resolve_conversation`, `open_conversation`, `pending_conversation`, `change_priority`, `add_sla` |
| Etiquetas | `add_label`, `remove_label` |
| Mensagens | `send_message`, `send_canned_response`, `send_whatsapp_template`, `send_attachment`, `add_private_note`, `send_email_to_team`, `send_email_transcript` |
| Kanban | `create_kanban_item`, `move_kanban_item_to_stage`, `assign_agent_to_kanban_item`, `add_note_to_kanban_item`, `set_kanban_item_status`, `start_kanban_item_timer`, `stop_kanban_item_timer` |
| Dados | `update_contact_attribute`, `update_conversation_attribute` |
| Outros | `send_webhook_event`, `wait` |

**`wait` tem teto de 5 minutos (300 segundos).** Serve pra dar respiro entre duas mensagens, nao pra
follow-up. Valor maior e cortado pra 300 em silencio. Follow-up de horas/dias = FlowBuilder (secao 7).

### ATENCAO — evento de CARD usa uma lista MENOR de acoes

Quando `event_name` e `kanban_item_created` ou `kanban_item_stage_changed`, a regra roda por um motor
diferente, que aceita **so estas dez acoes**:

`send_message`, `add_label`, `change_priority`, `assign_agent_to_kanban_item`,
`add_note_to_kanban_item`, `move_kanban_item_to_stage`, `set_kanban_item_status`,
`start_kanban_item_timer`, `stop_kanban_item_timer`, `send_webhook_event`.

Qualquer outra acao (`assign_team`, `send_canned_response`, `send_whatsapp_template`,
`add_private_note`, `assign_captain_assistant`, `update_contact_attribute`, `wait`...) e **ignorada em
silencio**: a regra e criada, aparece ativa na tela e simplesmente nao faz aquilo. Nao ha erro.

**A boa noticia:** `send_message` esta na lista. Entao "quando o card entrar na etapa X, mandar
mensagem para o cliente" **e possivel** — como regra de automacao GLOBAL com evento de card, e sem
atraso (na hora). O que nao existe e o atraso.

**A armadilha:** a mensagem vai para a conversa ligada ao card. Se o card tem conversas vinculadas e
**nenhuma** delas esta marcada para receber automacao, nada e enviado — em silencio. Card sem conversa
nenhuma tambem nao envia nada.

---

## 9. SLA

**WRAP KEY OBRIGATORIO:** `{"sla_policy": {...}}`

**O recurso SLA nasce DESLIGADO na conta.** Criar politica de SLA numa conta em que o recurso nao esta
liberado responde **403** (nao 404). O endpoint existe sempre — o que falta e a liberacao do recurso.

Teste antes de tentar criar:

```bash
curl -s -o /dev/null -w "%{http_code}" \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/sla_policies \
  --header 'api_access_token: SEU_TOKEN'
```

O GET responde 200 mesmo com o recurso desligado — quem barra e o POST. Entao: **tente criar; se vier 403,
pule o SLA** e avise no resumo final: "SLA nao criado porque o recurso nao esta liberado nessa conta — fale
com o suporte do LionChat para liberar".

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/sla_policies \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "sla_policy": {
      "name": "SLA Padrao",
      "description": "Prazos de atendimento padrao",
      "first_response_time_threshold": 600,
      "next_response_time_threshold": 1800,
      "resolution_time_threshold": 14400,
      "only_during_business_hours": true
    }
  }'
```

Valores em **segundos**:
- 600 = 10 min (primeira resposta rapida)
- 1800 = 30 min
- 14400 = 4 horas (resolucao)

---

## 10. Macros (acoes pre-configuradas)

**Sem wrap key.**

**ATENCAO:** `action_params` deve **sempre ser array**, mesmo que tenha so 1 valor.

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/macros \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "Escalar para Gerente",
    "visibility": "global",
    "actions": [
      {
        "action_name": "assign_team",
        "action_params": [1]
      },
      {
        "action_name": "add_label",
        "action_params": ["prioridade-alta"]
      },
      {
        "action_name": "send_message",
        "action_params": ["Sua solicitacao foi escalada para o gerente. Retornamos em ate 30 min."]
      }
    ]
  }'
```

`visibility`: `global` (todos agentes) ou `personal` (so quem criou).

A diferenca pra automacao: **macro nao dispara sozinha** — o atendente clica nela dentro da conversa.

---

## 11. Horario de trabalho (working hours)

Cada inbox tem seu horario. Precisa PATCH na inbox especifica:

```bash
# Listar inboxes
curl -s --request GET \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/inboxes \
  --header 'api_access_token: SEU_TOKEN'

# Atualizar horario de uma inbox (sem wrap key)
curl -s --request PATCH \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/inboxes/{inbox_id} \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "working_hours_enabled": true,
    "out_of_office_message": "Estamos fora do horario. Responderemos em breve.",
    "working_hours": [
      {"day_of_week": 1, "open_hour": 9, "open_minutes": 0, "close_hour": 18, "close_minutes": 0, "closed_all_day": false},
      {"day_of_week": 2, "open_hour": 9, "open_minutes": 0, "close_hour": 18, "close_minutes": 0, "closed_all_day": false},
      {"day_of_week": 3, "open_hour": 9, "open_minutes": 0, "close_hour": 18, "close_minutes": 0, "closed_all_day": false},
      {"day_of_week": 4, "open_hour": 9, "open_minutes": 0, "close_hour": 18, "close_minutes": 0, "closed_all_day": false},
      {"day_of_week": 5, "open_hour": 9, "open_minutes": 0, "close_hour": 18, "close_minutes": 0, "closed_all_day": false},
      {"day_of_week": 6, "closed_all_day": true},
      {"day_of_week": 0, "closed_all_day": true}
    ]
  }'
```

`day_of_week`: 0=domingo, 1=segunda, ..., 6=sabado.

---

## 12. Variaveis da conta

**WRAP KEY OBRIGATORIO:** `{"variable": {...}}`

Campos (nao aceita `name`/`key` como nomes diretos):
- `attribute_display_name` (nome de exibicao)
- `attribute_key` (chave snake_case)
- `attribute_description` (opcional)
- `attribute_display_type` (integer: 0=text, 1=number, etc — mesma tabela da secao 2.1)
- `value` (o valor em si)

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/account_variables \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "variable": {
      "attribute_display_name": "Telefone de Suporte",
      "attribute_key": "telefone_suporte",
      "attribute_description": "Numero principal do atendimento",
      "attribute_display_type": 0,
      "value": "(11) 9999-9999"
    }
  }'
```

**Como usar no texto:** `{{account.custom_attribute.telefone_suporte}}`

O trecho `custom_attribute` no meio e **obrigatorio**. Escrever `{{account.telefone_suporte}}` nao da erro —
a mensagem simplesmente sai com um **buraco no lugar do valor**, e ninguem percebe. Vale em resposta rapida,
campanha, automacao e fluxo.

---

## O que NAO e feito via API (avisar o cliente)

| O que | Por que |
|-------|---------|
| Conectar WhatsApp | Precisa escanear QR Code no celular |
| Conectar Instagram/Facebook | Fluxo OAuth (abre navegador) |
| Integracao Guru/Hotmart/Eduzz | Cliente precisa pegar token na outra plataforma |
| Google Calendar | OAuth |
| Upload de avatar/foto de perfil | Manual no painel |
| Convidar agentes novos | Envio de email — o cliente faz em Agentes > Convidar |
| Desenhar fluxos do FlowBuilder | Fora do escopo desta skill |

---

## Erros comuns

| Status | Causa | Acao |
|--------|-------|------|
| 401 | Token invalido/expirado, ou sem permissao | Para tudo. "Confere o API token em Configuracoes > Perfil" |
| 403 | Recurso nao liberado na conta (o caso conhecido e o SLA) | Pula o item e reporta no resumo final |
| 404 | Conta nao existe **ou** o usuario nao tem acesso a ela; ou endpoint que nao existe mais | Confere o numero da conta e o acesso. Se for `/kanban/automations`, veja a secao 7 |
| 422 | Validacao falhou **ou** falta/sobra de wrap key (`param is missing or the value is empty: X`) | Mostra a mensagem exata e corrige o formato do payload |
| 429 | Limite de requisicoes | Espera 5s e tenta de novo |
| 500 | Erro interno | Tenta 1 vez, se falhar de novo pula e reporta no final |

---

## Boas praticas de execucao

1. **Sempre em ordem de dependencia** — tags e modelos de checklist antes do funil; funil antes das automacoes
   que citam etapas.
2. **Validar account_id primeiro** — GET `/accounts/{id}` retorna 200 e mostra `name` da conta.
3. **Listar antes de criar** — pra evitar duplicata (ex: `GET /labels` antes de `POST /labels`).
4. **Ler antes de sobrescrever** — `kanban_config` e `funnel.settings` sao substituidos por inteiro. GET,
   junta, PUT/PATCH.
5. **Timeout curl** — use `--max-time 10` em toda chamada. Se falhar, tenta 1 vez de novo.
6. **Nao para tudo no primeiro erro** — registra erro, continua, reporta no final.
7. **Confirma IDs retornados** — ao criar funil, guarde o `id` e as chaves das etapas.
8. **Mostra progresso** — 1 linha curta por criacao: `Criando tag "fonte-instagram"... OK`.
9. **Silent** — use `curl -s` sempre pra nao poluir a saida com progress bar.
