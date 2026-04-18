---
name: configurar-conta
description: Entrevista o usuario sobre o negocio dele e configura a conta LionChat (funis, etapas, tags, respostas rapidas, campos personalizados, automacoes, times, SLA e horarios) via API. Use quando o usuario quer configurar uma conta LionChat nova, montar funil de vendas, ou estruturar processo de atendimento. O fluxo e interativo e sempre confirma antes de criar qualquer coisa na conta.
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

Se o usuario for vago ou curto, faz 1-2 perguntas por vez (nao dispara 8 perguntas de uma vez — cansa).

### Etapa 2 — Aplicar principios

Com as respostas em mao, leia `references/principles.md` e aplique as heuristicas:
- Ciclo curto impulso? Funil de 3-4 etapas.
- B2B ticket alto? Etapa de "proposta" e "negociacao" separadas.
- Follow-up e o gargalo? Cria etapa dedicada + automacao de mensagem.
- E-commerce? Tags de carrinho abandonado, objecao-frete, etc.

NAO invente etapas genericas tipo "contato inicial → qualificacao → proposta → fechado". Cada conta precisa de um funil que REFLITA o negocio dela. Use os nomes que o cliente usa (ex: "Avaliacao odontologica" em vez de "Qualificacao").

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
  /boas-vindas — saudacao inicial
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

Explique o **porque** de cada decisao em 1 frase. Ex: "Criei a tag 'objecao-preco' porque voce disse que preco e a reclamacao mais comum — voce pode filtrar todos esses leads depois pra oferecer desconto em massa."

Pergunta no final: **"Confirma que posso criar tudo isso? (s/n ou me diga o que mudar)"**

Se o usuario pedir ajustes, refaca a proposta e pergunta de novo. NAO avance sem "sim" explicito.

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

**IMPORTANTE sobre seguranca:**
- NAO salvar token em arquivo por padrao
- NAO commitar token em git
- Token fica so na sessao atual
- Se o usuario pedir pra salvar pra proxima vez, avisa: "vou salvar em ~/.lionchat/credentials (fora do seu projeto, nunca vai pro GitHub)"

### Etapa 5 — Executar

Use `curl` via Bash. Sempre nessa ordem (dependencias):

1. **Horario de trabalho + config da conta** (PATCH /accounts/{id})
2. **Campos personalizados** (POST /custom_attribute_definitions)
3. **Tags/Labels** (POST /labels)
4. **Respostas rapidas** (POST /canned_responses)
5. **Times** (POST /teams)
6. **Funil** (POST /kanban/funnels) — precisa existir antes das automacoes
7. **Automacoes de etapa** (POST /kanban/automations) — depende do funil
8. **Regras de automacao globais** (POST /automation_rules)
9. **SLA** (POST /sla_policies)
10. **Macros** (POST /macros)

Para cada chamada:
- Mostra 1 linha do que ta criando: `Criando tag "fonte-instagram"...`
- Se der erro 422 (validation), mostra a mensagem e pergunta se pula ou corrige
- Se der erro 401 (auth), para tudo e diz "token invalido, confere na configuracao"
- Se der erro de rede/500, tenta 1 vez de novo e reporta

**Endpoints e exemplos curl**: ver `references/api-endpoints.md`.

### Etapa 6 — Resumo final

```
Setup completo em [X] segundos.

Criado na sua conta:
  [X] tags
  [X] respostas rapidas
  [X] campos personalizados
  1 funil com [X] etapas
  [X] automacoes

PROXIMOS PASSOS MANUAIS (API nao faz):
  1. Conectar seu WhatsApp em Inboxes (precisa QR Code)
  2. Convidar agentes em Configuracoes > Agentes
  3. [outros passos especificos do negocio]

Dica: testa criando um contato e movendo pelo funil pra ver 
as automacoes disparando.
```

## Regras que nao podem ser violadas

1. **NUNCA cria algo sem confirmacao explicita** — "sim", "pode", "confirmado". "Ok" ou "beleza" tambem valem.
2. **NUNCA inventa endpoints** — se nao estiver em `references/api-endpoints.md`, pergunta antes.
3. **NUNCA expoe o token nos logs** — se precisar mostrar curl, mascara: `api_access_token: lc_prod_***`
4. **NUNCA deleta nada** — skill e so pra CRIAR. Se o usuario quer deletar algo, avisa que precisa fazer manualmente no painel.
5. **SEMPRE em portugues brasileiro** — mensagens, etapas, tags, respostas. Nomes de variaveis (snake_case) ficam em ingles.
6. **SEMPRE valida account_id antes de criar** — primeira chamada deve ser `GET /api/v1/accounts/{id}` pra confirmar que o token funciona na conta certa.
7. **NAO usa emojis** em nada que vai pra conta (nomes de etapa, tags, respostas) — o cliente pode nao querer.

## Se o usuario perguntar o que voce pode fazer

Responde:

> Eu configuro sua conta LionChat baseado no seu negocio:
> funis de venda, etapas personalizadas, tags de segmentacao,
> respostas rapidas, campos personalizados, automacoes de etapa,
> regras globais, times, SLA e horario de atendimento.
>
> Me conta o que voce vende, como atende hoje e qual seu maior
> problema no atendimento. A partir disso proponho uma estrutura
> e voce aprova antes de qualquer coisa ser criada.
