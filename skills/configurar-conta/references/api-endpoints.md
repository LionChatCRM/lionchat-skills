# LionChat API — Endpoints para Configuracao

**Base URL:** `https://app.lionchat.com.br`

**Autenticacao:** header `api_access_token: SEU_TOKEN` em toda chamada.

**Content-Type:** `application/json` em POST/PATCH/PUT.

**IMPORTANTE — WRAP KEYS:** A maioria dos endpoints do LionChat espera o payload **embrulhado** em uma chave com o nome do recurso. Ex: `labels` espera `{"label": {...}}`, `funnels` espera `{"funnel": {...}}`, e assim por diante. Isso está indicado em cada endpoint abaixo. **Se voce nao usar o wrap correto, a API retorna 400.**

---

## 0. Validar credenciais (SEMPRE fazer primeiro)

```bash
curl -s --request GET \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id} \
  --header 'api_access_token: SEU_TOKEN'
```

Se retornar 401 → token invalido. Se retornar 200 → pode continuar. Confirmar que o `role` do usuario (na resposta do `/profile`) e `administrator`.

---

## 1. Configuracoes gerais da conta

**Sem wrap key.** Campos `timezone` e `auto_resolve_after` vao pra `custom_attributes`/`settings` internamente.

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

**Sem wrap key.**

`attribute_display_type`: `0`=text, `1`=number, `2`=currency, `3`=percent, `4`=link, `5`=date, `6`=list, `7`=checkbox.

`attribute_model`: `0`=conversation_attribute, `1`=contact_attribute (o valor `2`=account_attribute e reservado pro sistema — nao usar).

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
- `attribute_key` deve ser snake_case, sem acento, sem espaco.
- Para tipo `list` (6), adicionar `attribute_values: ["opcao1", "opcao2"]`.

### 2.2 Atributos de Card (Kanban)

**WRAP KEY OBRIGATORIO:** `{"kanban_config": {...}}`.

**ATENCAO — mecanismo diferente:** card attributes NAO usam `custom_attribute_definitions`. Sao armazenados dentro de `kanban_config.global_custom_attributes` como um array JSONB. Cada item do array tem:

- `name` — nome de exibicao (string livre)
- `type` — `"string"`, `"number"`, `"date"` ou `"boolean"`
- `is_list` — boolean, `true` se for lista de opcoes
- `list_values` — array de opcoes quando `is_list=true`, senao array vazio `[]`

**Mapa de tipos do frontend para o backend:**

| Tipo visual | type | is_list |
|-------------|------|---------|
| Texto | `string` | false |
| Numero | `number` | false |
| Link | `string` | false |
| Data | `date` | false |
| Lista | `string` | **true** (preencher list_values) |
| Checkbox | `boolean` | false |

**Importante:** este endpoint **sobrescreve o array inteiro**. Para adicionar UM atributo novo, leia os existentes primeiro, faca append no array e depois envie o PUT.

**Passo 1 — ler config atual:**

```bash
curl -s --request GET \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban_config \
  --header 'api_access_token: SEU_TOKEN'
```

Resposta contem `global_custom_attributes: [...]` (pode ser vazio `[]` se nunca foi configurado).

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

**Se a conta nunca teve kanban_config**, o GET do passo 1 retorna 404. Nesse caso, use POST (em vez de PUT) para criar a config pela primeira vez:

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban_config \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "kanban_config": {
      "enabled": true,
      "config": { "title": "Kanban", "default_view": "kanban" },
      "global_custom_attributes": [...]
    }
  }'
```

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

Regras:
- `title` e **forcado para minusculo** automaticamente. So aceita letras, numeros, hifen e underscore.
- NAO pode ter espaco nem acento.
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

Variaveis disponiveis no `content`:
- `{{contact.name}}`, `{{contact.email}}`, `{{contact.phone}}`
- `{{agent.name}}`
- `{{conversation.id}}`

Regras:
- `short_code` minusculo, sem espaco, sem acento. Se tiver 2 palavras, usar hifen: `follow-up-48h` — esse formato funciona.
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
          "position": 1
        },
        "em_qualificacao": {
          "name": "Em Qualificacao",
          "color": "#F59E0B",
          "position": 2
        },
        "proposta_enviada": {
          "name": "Proposta Enviada",
          "color": "#8B5CF6",
          "position": 3
        },
        "fechado_ganho": {
          "name": "Fechado (Ganho)",
          "color": "#10B981",
          "position": 4
        }
      },
      "settings": {}
    }
  }'
```

Regras:
- Chaves das stages (ex: `lead_novo`): snake_case, sao os identificadores internos. Guarde-as para usar nas automacoes.
- `name` da stage: o que o cliente ve na tela.
- Cores: use paleta coerente (azul no inicio → amarelo → roxo → verde/vermelho no fim).
- Sempre incluir etapa final "ganho" e, se fizer sentido, "perdido".

**Retorno** contem `"id": N` — guarde o ID do funil para as automacoes.

---

## 7. Automacoes de etapa

**WRAP KEY OBRIGATORIO:** `{"kanban_automation": {...}}`

**Atencao — estrutura real:** o campo `trigger` e um JSONB livre. Gatilho de etapa fica DENTRO de `trigger`, nao como campos soltos.

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban/automations \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "kanban_automation": {
      "name": "Follow-up 48h apos proposta",
      "description": "Manda mensagem de followup 2 dias apos entrar na etapa",
      "active": true,
      "trigger": {
        "funnel_id": 7,
        "stage": "proposta_enviada",
        "event": "on_enter"
      },
      "conditions": [],
      "actions": [
        {
          "action_name": "send_message",
          "action_params": {
            "content": "Ola {{contact.name}}, conseguiu analisar a proposta? Estamos a disposicao!",
            "delay_seconds": 172800
          }
        }
      ]
    }
  }'
```

`trigger.event`: `on_enter` (quando card entra na etapa) ou `on_leave` (quando sai).

`action_name` disponiveis (verificar atualizacoes no painel):
- `send_message` — envia mensagem na conversa do card
- `assign_agent` — atribui agente (params: `{agent_id: N}`)
- `assign_team` — atribui time (params: `{team_id: N}`)
- `apply_label` — aplica tag (params: `{labels: ["tag1", "tag2"]}`)
- `add_note` — adiciona nota (params: `{content: "texto"}`)

`delay_seconds` em `send_message`: atraso antes do envio. 172800 = 48h.

---

## 8. Regras de automacao globais

**WRAP KEY OBRIGATORIO:** `{"automation_rule": {...}}` (nota: aqui e `automation_rule`, singular).

Triggers mais usados em `event_name`:
- `conversation_created` — nova conversa
- `message_created` — nova mensagem recebida
- `conversation_status_changed` — status mudou
- `conversation_updated` — conversa foi atualizada

```bash
curl -s --request POST \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/automation_rules \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "automation_rule": {
      "name": "Auto-atribuir ao time de vendas",
      "event_name": "conversation_created",
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
      ],
      "active": true
    }
  }'
```

---

## 9. SLA (Enterprise only)

**AVISO:** este endpoint requer licenca Enterprise. Em contas OSS retorna 404. Sempre verifique antes de criar SLA, fazendo um GET primeiro:

```bash
curl -s -o /dev/null -w "%{http_code}" \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/sla_policies \
  --header 'api_access_token: SEU_TOKEN'
```

Se retornar 404 → conta sem Enterprise, pular SLA. Se 200 → pode criar.

**WRAP KEY OBRIGATORIO:** `{"sla_policy": {...}}`

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

**Sem wrap key.** Campos sao recebidos direto em `params.permit`.

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

**Atencao — campos reais:** o endpoint NAO aceita `name/key/value` como nomes diretos. Os campos corretos sao:
- `attribute_display_name` (nome de exibicao)
- `attribute_key` (chave snake_case)
- `attribute_description` (opcional)
- `attribute_display_type` (integer: 0=text, 1=number, etc)
- `value` (o valor em si — enviado separadamente, e setado via `save_value` interno quando `params[:variable].key?(:value)`)

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

Uso nas respostas rapidas: `{{account.telefone_suporte}}`.

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

---

## Erros comuns

| Status | Causa | Acao |
|--------|-------|------|
| 400 | Wrap key faltando | Verifique se payload ta `{"label": {...}}` e nao `{...}` |
| 401 | Token invalido ou expirado | Para tudo, avisa: "confere o API token em Configuracoes > Perfil" |
| 403 | Token valido mas sem permissao na conta | "Seu usuario nao e administrator da conta {account_id}" |
| 404 (em endpoint que deveria existir) | SLA sem Enterprise, account_id errado | SLA: avisa e pula. account_id: confere URL |
| 422 | Validacao falhou | Mostra a mensagem exata do erro e pergunta se pula ou corrige |
| 429 | Rate limit | Espera 5s e tenta de novo |
| 500 | Erro interno | Tenta 1 vez, se falhar de novo pula e reporta no final |

---

## Boas praticas de execucao

1. **Sempre em ordem de dependencia** — tags antes de respostas, funil antes de automacoes de etapa.
2. **Validar account_id primeiro** — GET `/accounts/{id}` retorna 200 e mostra `name` da conta. Se 404, parar.
3. **Listar antes de criar** — pra evitar duplicata (ex: `GET /labels` antes de `POST /labels`).
4. **Timeout curl** — use `--max-time 10` em toda chamada. Se falhar, tenta 1 vez de novo.
5. **Nao para tudo no primeiro erro** — registra erro, continua, reporta no final.
6. **Confirma IDs retornados** — ao criar funil, guarde o `id` pra usar nas automacoes de etapa.
7. **Mostra progresso** — 1 linha curta por criacao: `Criando tag "fonte-instagram"... OK`.
8. **Silent** — use `curl -s` sempre pra nao poluir a saida com progress bar.
