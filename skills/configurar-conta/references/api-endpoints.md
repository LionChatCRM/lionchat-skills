# LionChat API — Endpoints para Configuracao

**Base URL:** `https://app.lionchat.com.br`

**Autenticacao:** header `api_access_token: SEU_TOKEN` em toda chamada.

**Content-Type:** `application/json` em POST/PATCH/PUT.

---

## 0. Validar credenciais (SEMPRE fazer primeiro)

```bash
curl --request GET \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id} \
  --header 'api_access_token: SEU_TOKEN'
```

Se retornar 401 → token invalido. Se retornar 200 → pode continuar.

---

## 1. Configuracoes gerais da conta

```bash
curl --request PATCH \
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

`attribute_display_type`: `0`=text, `1`=number, `2`=currency, `3`=percent, `4`=link, `5`=date, `6`=list, `7`=checkbox.

`attribute_model`: `0`=contact_attribute, `1`=conversation_attribute.

```bash
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/custom_attribute_definitions \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "attribute_display_name": "Valor do Orcamento",
    "attribute_display_type": 2,
    "attribute_key": "valor_orcamento",
    "attribute_model": 1,
    "attribute_description": "Valor total proposto ao cliente"
  }'
```

Regras:
- `attribute_key` deve ser snake_case, sem acento, sem espaco.
- Para tipo `list` (6), adicionar `attribute_values: ["opcao1", "opcao2"]`.

---

## 3. Tags (Labels)

```bash
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/labels \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "title": "fonte-instagram",
    "description": "Lead veio do Instagram",
    "color": "#E91E63",
    "show_on_sidebar": true
  }'
```

Regras:
- `title` deve ser kebab-case (minusculo, hifen), sem acento, sem espaco.
- Use cores diferentes por categoria (ex: azul = fonte, vermelho = objecao, verde = positivo).
- Paleta sugerida:
  - Fontes: `#1976D2`, `#1565C0`, `#0D47A1`
  - Objecoes: `#E53935`, `#C62828`, `#B71C1C`
  - Status positivo: `#43A047`, `#2E7D32`
  - Prioridade: `#F57C00`, `#E65100`
  - Neutro: `#616161`, `#424242`

---

## 4. Respostas rapidas

```bash
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/canned_responses \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "short_code": "boasvindas",
    "content": "Ola {{contact.name}}! Obrigado pelo contato. Como posso ajudar?"
  }'
```

Variaveis disponiveis no `content`:
- `{{contact.name}}`, `{{contact.email}}`, `{{contact.phone}}`
- `{{agent.name}}`
- `{{conversation.id}}`

Regras:
- `short_code` minusculo, sem espaco, sem acento. Se tiver 2 palavras, usar hifen: `follow-up-48h`.
- Sempre sugerir pelo menos: boas-vindas, horario-fora, aguarde-um-momento, agradecimento-final.

---

## 5. Times

```bash
# Criar time
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/teams \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "Vendas",
    "description": "Time comercial",
    "allow_auto_assign": true
  }'
```

`allow_auto_assign`: se `true`, o LionChat distribui conversas automaticamente entre agentes do time.

---

## 6. Funil (Kanban)

```bash
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/funnels \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
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
  }'
```

Regras:
- Chaves das stages (ex: `lead_novo`): snake_case, sao os identificadores internos. Use-as depois nas automacoes.
- `name` da stage: o que o cliente ve na tela.
- Cores: use uma paleta coerente (ex: azul no inicio → amarelo no meio → roxo negociando → verde fechado / vermelho perdido).
- Sempre inclua uma etapa final "ganho" e uma "perdido" se fizer sentido.

Retorno contem `"id": N` — guarde o ID do funil para as automacoes.

---

## 7. Automacoes de etapa

```bash
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/kanban/automations \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "funnel_id": 7,
    "stage": "proposta_enviada",
    "trigger": "on_enter",
    "actions": [
      {
        "action_name": "send_message",
        "action_params": {
          "content": "Ola {{contact.name}}, envie sua proposta. Estamos a disposicao para tirar duvidas!",
          "delay_seconds": 172800
        }
      }
    ],
    "active": true
  }'
```

`trigger`: `on_enter` (quando card entra na etapa) ou `on_leave` (quando sai).

`action_name` disponiveis:
- `send_message` — envia mensagem na conversa do card
- `assign_agent` — atribui agente (params: `{agent_id: N}`)
- `assign_team` — atribui time (params: `{team_id: N}`)
- `apply_label` — aplica tag (params: `{labels: ["tag1", "tag2"]}`)
- `add_note` — adiciona nota (params: `{content: "texto"}`)

`delay_seconds` em `send_message`: atraso antes do envio. 172800 = 48h.

---

## 8. Regras de automacao globais

Triggers mais usados:
- `conversation_created` — nova conversa
- `message_created` — nova mensagem recebida
- `conversation_status_changed` — status mudou
- `conversation_updated` — conversa foi atualizada

```bash
curl --request POST \
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

## 9. SLA

```bash
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/sla_policies \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "SLA Padrao",
    "description": "Prazos de atendimento padrao",
    "first_response_time_threshold": 600,
    "next_response_time_threshold": 1800,
    "resolution_time_threshold": 14400,
    "only_during_business_hours": true
  }'
```

Valores em **segundos**:
- 600 = 10 min (primeira resposta rapida)
- 1800 = 30 min
- 14400 = 4 horas (resolucao)

---

## 10. Macros (acoes pre-configuradas)

```bash
curl --request POST \
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
        "action_params": "Sua solicitacao foi escalada para o gerente. Retornamos em ate 30 min."
      }
    ]
  }'
```

`visibility`: `global` (todos agentes) ou `personal` (so quem criou).

---

## 11. Horario de trabalho (working hours)

Cada inbox tem seu horario. Ao criar inbox nova o cliente define. Se quiser configurar horario padrao da conta:

```bash
# Listar inboxes
curl --request GET \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/inboxes \
  --header 'api_access_token: SEU_TOKEN'

# Atualizar horario de uma inbox
curl --request PATCH \
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

```bash
curl --request POST \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id}/account_variables \
  --header 'api_access_token: SEU_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "Telefone Suporte",
    "key": "telefone_suporte",
    "value": "(11) 9999-9999"
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
| 401 | Token invalido ou expirado | Para tudo, avisa: "confere o API token em Configuracoes > Perfil" |
| 403 | Token valido mas sem permissao na conta | "Seu usuario nao e administrator da conta {account_id}" |
| 404 | account_id errado | "Confere o numero da conta na URL do painel" |
| 422 | Validacao falhou | Mostra a mensagem exata do erro e pergunta se pula ou corrige |
| 429 | Rate limit | Espera 5s e tenta de novo |
| 500 | Erro interno | Tenta 1 vez, se falhar de novo pula e reporta no final |

---

## Boas praticas de execucao

1. **Sempre em ordem de dependencia** — tags antes de respostas (respostas referenciam tags), funil antes de automacoes de etapa.
2. **Nao para tudo no primeiro erro** — registra erro, continua, reporta no final.
3. **Nao cria duplicata** — antes de criar tag "fonte-instagram", chama `GET /labels` e verifica se ja existe.
4. **Confirma IDs** — ao criar funil, guarde o `id` retornado pra usar nas automacoes.
5. **Mostra progresso** — 1 linha curta por criacao: `Criando tag "fonte-instagram"... OK`.
