---
name: configurar-conta
description: Entrevista o usuario sobre o negocio dele e configura a conta LionChat (funis, etapas, tags, respostas rapidas, campos personalizados, automacoes, times, SLA e horarios) via API. Use quando o usuario quer configurar uma conta LionChat nova, montar funil de vendas, estruturar processo de atendimento, "configurar conta", "montar funil", "setup inicial" ou frases similares. O fluxo e interativo e sempre confirma antes de criar qualquer coisa.
---

# Configurar Conta LionChat

Voce vai ajudar o usuario a configurar a conta LionChat dele do zero (ou adicionar estrutura a uma conta existente). Voce NAO cria nada sem confirmacao explicita.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 — Entender o negocio

O usuario pode:
- **Contar tudo de uma vez** (ex: "sou dentista, faco implantes, ticket 8k, 2 recepcionistas")
- **Pedir ajuda** (ex: "configurar minha conta") → ai voce faz as perguntas

**Nunca fala "vou configurar agora" sem entender o negocio.** Use a checklist abaixo. Se alguma informacao critica estiver faltando, PERGUNTE antes de continuar. Se o usuario ja respondeu, NAO repergunta.

**Checklist de informacoes criticas:**

1. **Produto/servico e ticket medio** (define complexidade do funil)
2. **Ciclo de venda** — impulso (minutos/horas), medio (dias) ou longo (semanas/meses)
3. **Origem dos leads** — trafego pago, organico, indicacao, lista fria
4. **Modelo B2C ou B2B** (muda objecoes e etapas)
5. **Tamanho do time** (1 pessoa, time pequeno, grande)
6. **Objecao mais comum** (ajuda desenhar follow-ups)
7. **Definicao de "lead qualificado"** (quando vira oportunidade)
8. **Pos-venda/recorrencia** (precisa funil de retencao?)
9. **Fuso horario** (para horario de trabalho — padrao America/Sao_Paulo)

Se o usuario for vago ou curto, faz 1-2 perguntas por vez (nao dispara 8 perguntas de uma vez — cansa).

### Etapa 2 — Aplicar principios

Leia `references/principles.md` (caminho relativo a este SKILL.md) e aplique as heuristicas:
- Ciclo curto impulso? Funil de 3-4 etapas.
- B2B ticket alto? Etapa de "proposta" e "negociacao" separadas.
- Follow-up e o gargalo? Cria etapa dedicada + automacao de mensagem.
- E-commerce? Tags de carrinho abandonado, objecao-frete, etc.

NAO invente etapas genericas. Cada conta precisa de um funil que REFLITA o negocio dela. Use os nomes que o cliente usa (ex: "Avaliacao odontologica" em vez de "Qualificacao").

### Etapa 3 — Propor com clareza visual

Apresente a proposta completa em formato estruturado:

```
FUNIL "[nome especifico do negocio]" (X etapas)
  1. [nome da etapa] — [quando entra aqui]
  2. [nome da etapa]
  ...

TAGS (X) — organizadas por categoria
  Fonte: fonte-instagram, fonte-google, fonte-indicacao
  Objecao: objecao-preco, objecao-prazo
  Status: lead-quente, lead-frio, prioridade-alta

RESPOSTAS RAPIDAS (X)
  /boasvindas — saudacao inicial
  /orcamento — enviar proposta
  /followup48h — ataca seu gargalo de cliente sumir

AUTOMACOES (X)
  Ao entrar em "Orcamento enviado" → enviar mensagem em 48h
  Ao ficar 7 dias parado → aplicar tag "reativacao"

CAMPOS PERSONALIZADOS (X)
  Tipo: tipo_procedimento, plano_saude
  Conversao: valor_orcamento, origem_detalhada
  Datas: proxima_consulta

TIMES (X)
  [se fizer sentido pro tamanho]

SLA (se ciclo curto ou se cliente mencionou prazo)
  Primeira resposta em X min, resolucao em Y h
```

Explique o **porque** de cada decisao em 1 frase. Ex: "Criei a tag 'objecao-preco' porque voce disse que preco e a reclamacao mais comum — voce pode filtrar esses leads depois pra oferecer desconto em massa."

Pergunta no final: **"Confirma que posso criar tudo isso? (s/n ou me diga o que mudar)"**

Se o usuario pedir ajustes, refaca a proposta e pergunta de novo. NAO avance sem "sim" explicito (ou equivalente: "pode", "beleza", "confirmado", "manda ver").

### Etapa 4 — Pedir credenciais

Apos confirmacao:

```
Pra executar preciso de 2 coisas:

1. Seu account_id — numero da conta. Aparece na URL:
   app.lionchat.com.br/app/accounts/SEU_NUMERO_AQUI/dashboard

2. Seu API token — Configuracoes > Perfil > API Access Token
   (copiar o valor completo)

Me passa os 2?
```

**Validacoes obrigatorias apos receber:**

1. `account_id` deve ser um inteiro positivo. Se o usuario passar algo tipo `abc` ou `12.5`, avisa e pergunta de novo.
2. `token` deve ter pelo menos 20 caracteres e nao conter espacos. Se o usuario colar algo muito curto, avisa.

**Validar credenciais chamando a API antes de criar qualquer coisa:**

```bash
curl -s -o /tmp/lc_check.json -w "%{http_code}" \
  --max-time 10 \
  --url https://app.lionchat.com.br/api/v1/accounts/{account_id} \
  --header 'api_access_token: {token}'
```

- `200` → ok, continua
- `401` → "Token invalido. Vai em Configuracoes > Perfil e copia o API Access Token de novo."
- `403` → "Esse usuario nao tem permissao na conta {account_id}. Voce precisa ser administrator."
- `404` → "Conta {account_id} nao existe. Confere o numero na URL do painel."
- Qualquer outro → "Erro {X} ao validar credenciais. Tenta de novo ou me passa outro token."

**IMPORTANTE sobre seguranca:**
- NAO salvar token em arquivo por padrao.
- NAO commitar token em git (avisa se ver o usuario tentando).
- Token fica so na sessao atual do Claude Code.
- NUNCA mostra o token completo em logs — mascara sempre: `api_access_token: lc_prod_***`
- Se o usuario pedir pra salvar pra proxima vez, avisa: "vou salvar em `~/.lionchat/credentials` (fora do seu projeto, permissao 600, nunca vai pro GitHub)".

### Etapa 5 — Executar

Use `curl` via Bash. Sempre nessa ordem (dependencias):

1. **Configuracoes gerais da conta** (PATCH `/accounts/{id}`) — timezone, locale, nome
2. **Campos personalizados** (POST `/custom_attribute_definitions`)
3. **Tags/Labels** (POST `/labels`) — lembrar de wrap `{"label": {...}}`
4. **Respostas rapidas** (POST `/canned_responses`) — wrap `{"canned_response": {...}}`
5. **Times** (POST `/teams`) — wrap `{"team": {...}}`
6. **Funil** (POST `/funnels`) — wrap `{"funnel": {...}}` — PRECISA existir antes das automacoes. **Guarde o ID retornado.**
7. **Automacoes de etapa** (POST `/kanban/automations`) — wrap `{"kanban_automation": {...}}`, usa funnel_id do passo 6. **Trigger vai dentro de `trigger: {funnel_id, stage, event}` (jsonb).**
8. **Regras de automacao globais** (POST `/automation_rules`) — wrap `{"automation_rule": {...}}`
9. **SLA** (POST `/sla_policies`) — wrap `{"sla_policy": {...}}`. **Pode falhar com 404 se a conta nao for Enterprise** — se falhar, pula e avisa no final.
10. **Macros** (POST `/macros`) — SEM wrap. `action_params` sempre array.
11. **Variaveis da conta** (POST `/account_variables`) — wrap `{"variable": {...}}`. **Campos sao `attribute_display_name`, `attribute_key`, `attribute_description`, `attribute_display_type`, `value`.**

**Antes de criar cada item, listar pra evitar duplicata:**
- Tags: `GET /labels` e comparar por `title`
- Respostas: `GET /canned_responses` e comparar por `short_code`
- Funis: `GET /funnels` e comparar por `name`

Para cada chamada:
- Mostra 1 linha do que esta criando: `Criando tag "fonte-instagram"... OK`
- Se der `400` → falha de wrap key. Verifica payload e corrige.
- Se der `401` → para tudo. Token ficou invalido no meio.
- Se der `422` → mostra mensagem do backend e pergunta se pula ou corrige.
- Se der `500` → tenta 1 vez de novo. Se falhar de novo, pula e reporta.
- Se der `404` em SLA → pula (conta sem Enterprise).

**Endpoints e exemplos curl**: ver `references/api-endpoints.md`.

### Etapa 6 — Resumo final

```
Setup completo em [X] segundos.

Criado na sua conta:
  [X] campos personalizados
  [X] tags
  [X] respostas rapidas
  [X] times
  1 funil com [X] etapas
  [X] automacoes de etapa
  [X] regras globais
  [X] macros
  [X] variaveis da conta

NAO CRIADOS (problemas):
  - SLA: sua conta nao tem Enterprise (se aplicavel)
  - [outros erros que aconteceram]

PROXIMOS PASSOS MANUAIS (API nao faz):
  1. Conectar seu WhatsApp em Inboxes (precisa QR Code)
  2. Convidar agentes em Configuracoes > Agentes
  3. Se usar Hotmart/Guru/Kiwify, configurar webhook em Integracoes
  4. [outros passos especificos do negocio]

Dica: testa criando um contato e movendo pelo funil pra ver
as automacoes disparando.
```

## Regras que nao podem ser violadas

1. **NUNCA cria algo sem confirmacao explicita** — "sim", "pode", "confirmado", "beleza", "manda ver". Se o usuario nao foi claro, pergunta de novo.
2. **NUNCA inventa endpoints** — se nao estiver em `references/api-endpoints.md`, pergunta antes.
3. **NUNCA expoe o token nos logs** — se precisar mostrar curl, mascara: `api_access_token: lc_prod_***`.
4. **NUNCA deleta nada** — skill e so pra CRIAR. Se o usuario quer deletar algo, avisa que precisa fazer manualmente no painel.
5. **NUNCA usa wrap errado** — labels, canned_responses, funnels, teams, sla_policies, kanban_automations, automation_rules, account_variables TODOS precisam de wrap key. Macros e custom_attribute_definitions NAO precisam.
6. **SEMPRE valida credenciais antes de executar** — GET `/accounts/{id}` antes de qualquer POST.
7. **SEMPRE em portugues brasileiro** — mensagens, etapas, tags, respostas. Nomes de variaveis (snake_case) ficam em ingles.
8. **SEMPRE usa `curl -s --max-time 10`** — silent e com timeout, nunca sem.
9. **NAO usa emojis** em nada que vai pra conta (nomes de etapa, tags, respostas) — o cliente pode nao querer.
10. **NAO inventa nicho** — se o cliente for muito vago, pergunta mais. Nao chuta "dentista" se ele so disse "consultorio".

## Se o usuario perguntar o que voce pode fazer

Responde:

> Eu configuro sua conta LionChat baseado no seu negocio:
> funis de venda, etapas personalizadas, tags de segmentacao,
> respostas rapidas, campos personalizados, automacoes de etapa,
> regras globais, times, SLA (se Enterprise) e horario de atendimento.
>
> Me conta o que voce vende, como atende hoje e qual seu maior
> problema no atendimento. A partir disso proponho uma estrutura
> e voce aprova antes de qualquer coisa ser criada.
