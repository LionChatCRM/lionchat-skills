---
name: configurar-conta
description: Entrevista o usuario sobre o negocio dele e configura a conta LionChat (funis, etapas, tags, respostas rapidas, campos personalizados, automacoes, times, SLA e horarios) pelo conector do LionChat (as ferramentas `lionchat_*`). Use quando o usuario quer configurar uma conta LionChat nova, montar funil de vendas, estruturar processo de atendimento, "configurar conta", "montar funil", "setup inicial" ou frases similares. O fluxo e interativo e sempre confirma antes de criar qualquer coisa.
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
8b. **Por que voce ganha e por que voce perde** — os motivos mais comuns dos dois lados. Viram a lista
    que o vendedor escolhe ao fechar o card, e o relatorio de "por que estamos perdendo".
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

AUTOMACOES DE ETAPA (X) — rodam no card do funil, NAO mandam mensagem
  Ao criar card → aplicar checklist "Enviar proposta"
  Ao mover para "Fechado (Ganho)" → escrever nota interna

AUTOMACOES GLOBAIS (X) — rodam na conversa
  Conversa nova na caixa X → atribuir ao time "Vendas"
  Conversa resolvida → aplicar etiqueta "atendida"

MENSAGEM COM ATRASO (ex: follow-up 48h) — NAO e automacao, e Fluxo.
  Avisar que essa parte se monta em Fluxos, fora desta skill.

MOTIVOS DE GANHO E PERDA (X + X)
  Ganho: preco bom, indicacao, urgencia
  Perda: achou caro, fechou com concorrente, sumiu
  (aparecem na janela quando o vendedor marca o card, e viram relatorio)

CAMPOS PERSONALIZADOS (X total)
  Contato (dado persistente do cliente):
    - plano_saude, cpf, data_nascimento
  Conversa (dado da negociacao atual):
    - tipo_procedimento_consultado, preferencia_dia
  Card/Kanban (dado do card de venda):
    - valor_orcamento, motivo_perda, forma_pagamento

TIMES (X)
  [se fizer sentido pro tamanho]

SLA (se ciclo curto ou se cliente mencionou prazo)
  Primeira resposta em X min, resolucao em Y h
```

Explique o **porque** de cada decisao em 1 frase. Ex: "Criei a tag 'objecao-preco' porque voce disse que preco e a reclamacao mais comum — voce pode filtrar esses leads depois pra oferecer desconto em massa."

Pergunta no final: **"Confirma que posso criar tudo isso? (s/n ou me diga o que mudar)"**

Se o usuario pedir ajustes, refaca a proposta e pergunta de novo. NAO avance sem "sim" explicito (ou equivalente: "pode", "beleza", "confirmado", "manda ver").

### Etapa 4 — Conferir a conexao e a conta

Apos confirmacao, rode a rotina de pre-voo da skill `conectar-lionchat` (ela e a base de todas as skills):

1. **Provar a conexao.** Conector remoto: `lionchat_current_account`. Conector local: `lionchat_ping`.
   Nenhuma ferramenta `lionchat_` na sua lista = o cliente ainda nao conectou: siga a `conectar-lionchat`
   para ligar a conta antes de qualquer coisa.
2. **Dizer em texto o NOME da conta** em que voce vai mexer e esperar o "sim". Conta errada: no remoto,
   `lionchat_list_my_accounts` e `lionchat_switch_account`; no local, o campo `account_id` na chamada.
3. **Ler o terreno:** `lionchat_account_show` (fuso, plano e `resource_limits` - quantos atendentes e
   caixas cabem), `lionchat_inboxes_list`, `lionchat_agents_list`, `lionchat_teams_list`.

Para criar tudo desta skill o usuario precisa ser **administrador** da conta (ou ter um cargo que libere
cada area).

**Sem conector (caminho antigo, so se o cliente nao puder conectar):** da para usar chamada direta com o
Token de acesso, como em `references/api-endpoints.md` (secao 0). Nesse caminho o token NUNCA aparece inteiro
em texto (mascare: `api_access_token: lc_prod_***`), nao e salvo em arquivo do projeto e nao vai para o git.
Prefira sempre o conector: ele aplica as mesmas permissoes da tela, cuida do embrulho do corpo e registra a
origem no historico de Auditoria.

### Etapa 5 — Executar

Use as ferramentas do conector. Sempre nessa ordem (dependencias). O corpo de cada uma (campos e
exemplos) esta em `references/api-endpoints.md`, na secao indicada:

1. **Configuracoes gerais da conta** - `lionchat_account_settings_update` (fuso, idioma, nome; secao 1).
   **NAO use `lionchat_account_update`**: apesar do nome, ele grava o PERFIL do usuario logado. Fuso
   invalido e descartado em silencio: releia com `lionchat_account_show`. Em conta que ja usa a integracao
   e-Clinica, trocar o fuso RECALCULA os lembretes pendentes para sair no mesmo horario de relogio no fuso
   novo - avise o cliente antes.
2. **Atributos personalizados de contato, conversa e empresa** - `lionchat_custom_attributes_create`
   (`attribute_model`: `0` conversa, `1` contato, `3` empresa; secao 2.1). Se o cliente quiser os campos em
   pastas, `lionchat_custom_attribute_categories_create` e `lionchat_custom_attribute_categories_assign`
   (secao 2.5).
3. **Motivos de ganho/perda + atributos de card + modelos de checklist** - `lionchat_kanban_config_list`
   para ler e `lionchat_kanban_config_update` com as listas COMPLETAS (secoes 2.2 a 2.4). A leitura sempre
   responde (cria a configuracao sozinha se nao existir). O que voce nao mandar, some. **Guarde o `id` de
   cada modelo de checklist** - vai ser usado pelas automacoes de etapa.
4. **Tags/Labels** - `lionchat_labels_create` (secao 3).
5. **Respostas rapidas** - `lionchat_canned_responses_create` (secao 4).
6. **Times** - `lionchat_teams_create` (secao 5).
7. **Funil** - `lionchat_funnels_create` (secao 6). PRECISA existir antes das automacoes. **Guarde o ID e as
   chaves das etapas.**
8. **Automacoes de etapa** - **NAO tem ferramenta propria.** Vao DENTRO do funil, em `settings.automations`
   (secao 7). O jeito limpo e mandar junto na criacao do passo 7. Em funil que ja existe:
   `lionchat_funnels_show`, junte ao `settings` atual e `lionchat_funnels_update` com o `settings` inteiro.
9. **Regras de automacao globais** - `lionchat_automation_rules_create` (secao 8). Leia a regra do conector
   para automacao: a ULTIMA condicao precisa ter o conector de ligacao (`query_operator`) nulo.
10. **SLA** - `lionchat_sla_create` (secao 9). **Pode ser recusado se o recurso SLA nao estiver liberado na
    conta** (ele nasce desligado) - se falhar, pula e avisa no final.
11. **Macros** - `lionchat_macros_create` (secao 10). `action_params` sempre array.
12. **Variaveis da conta** - `lionchat_account_variables_create` (secao 12). No texto usa-se
    `{{account.custom_attribute.CHAVE}}` - com o `custom_attribute` no meio, senao sai em branco.
13. **Convidar atendentes**, se o cliente pediu e passou os e-mails - `lionchat_agents_create` (um, com nome,
    papel, times e caixas) ou `lionchat_agents_bulk_create` (varios so por e-mail). Cada pessoa recebe um
    convite por e-mail. Confira antes em `resource_limits` se o plano comporta: estourar o limite e recusado.

**Antes de criar cada item, listar pra evitar duplicata:**
- Tags: `lionchat_labels_list` e comparar por `title`
- Respostas: `lionchat_canned_responses_list` e comparar por `short_code`
- Funis: `lionchat_funnels_list` e comparar por `name`
- Atributos: `lionchat_custom_attributes_list` e comparar por `attribute_key`
- Atendentes: `lionchat_agents_list` e comparar por e-mail

Para cada chamada:
- Mostra 1 linha do que esta criando: `Criando tag "fonte-instagram"... OK`
- Se der `422` com `param is missing or the value is empty: X` → campo com nome errado (com o conector, o
  campo desconhecido some e o corpo fica vazio). Confere os nomes na ficha da ferramenta.
- Se der `422` normal → mostra mensagem do backend e pergunta se pula ou corrige.
- Se der `401` com mensagem de token ("Invalid Access Token") → para tudo: a conexao perdeu a validade.
- Se der `401` com "You are not authorized to do this action" → **nao e token**: o usuario nao tem
  permissao para AQUELA area. Pula o item e reporta no final (precisa de administrador).
- Se der `403` em SLA → pula (recurso nao liberado na conta) e reporta no final.
- Se der `404` → o item ou a rota nao existe. Confere contra `references/api-endpoints.md` antes de insistir.
- Se der `500` → tenta 1 vez de novo. Se falhar de novo, pula e reporta.
- Se a ferramenta recusar pedindo `confirm:true` → nao e erro: descreve o efeito, espera o "sim" e reenvia a
  MESMA chamada com `confirm:true`.

**Campos e exemplos de corpo**: ver `references/api-endpoints.md`.

### Etapa 6 — Resumo final

```
Setup completo em [X] segundos.

Criado na sua conta:
  [X] campos personalizados
  [X] motivos de ganho e [X] de perda
  [X] tags
  [X] respostas rapidas
  [X] times
  1 funil com [X] etapas
  [X] automacoes de etapa
  [X] regras globais
  [X] macros
  [X] variaveis da conta
  [X] atendentes convidados (o convite chega por e-mail)

NAO CRIADOS (problemas):
  - SLA: o recurso nao esta liberado nessa conta (se aplicavel) — fale com o suporte
  - [outros erros que aconteceram]

PROXIMOS PASSOS MANUAIS (o conector nao faz):
  1. Conectar seu WhatsApp: quem le o QR Code e voce, com o celular
     (eu posso gerar um link de conexao para quem esta com o celular)
  2. Os atendentes convidados precisam aceitar o convite do e-mail
  3. Se usar Hotmart/Guru/Kiwify, configurar webhook em Integracoes
  4. Se voce quer follow-up automatico com atraso (ex: cobrar em 48h),
     isso se monta em Fluxos — nao e automacao
  5. [outros passos especificos do negocio]

Dica: testa criando um contato e movendo pelo funil pra ver
as automacoes disparando.
```

## Regras que nao podem ser violadas

1. **NUNCA cria algo sem confirmacao explicita** — "sim", "pode", "confirmado", "beleza", "manda ver". Se o usuario nao foi claro, pergunta de novo.
2. **NUNCA inventa ferramenta nem endpoint** — se nao estiver na sua lista de ferramentas nem em `references/api-endpoints.md`, pesquise pelo padrao `lionchat_<area>_<acao>` e, se nao achar, pergunte.
3. **NUNCA expoe o token** — no caminho sem conector, se precisar mostrar uma chamada, mascara: `api_access_token: lc_prod_***`.
4. **NUNCA deleta nada** — skill e so pra CRIAR. Se o usuario quer deletar algo, avisa que precisa fazer manualmente no painel.
5. **Embrulho do corpo:** com o conector, mande os campos como a ficha da ferramenta mostra - ele embrulha sozinho onde precisa. So no caminho sem conector o embrulho e seu: COM wrap `labels`, `canned_responses`, `teams`, `funnels`, `sla_policies`, `kanban_config`, `account_variables`; SEM wrap `accounts`, `custom_attribute_definitions`, `automation_rules`, `macros`, `inboxes`. **`automation_rules` embrulhado FALHA.**
6. **SEMPRE prova a conexao e diz o nome da conta antes de executar** — `lionchat_current_account` (remoto) ou `lionchat_ping` (local) antes de qualquer criacao.
7. **SEMPRE em portugues brasileiro** — mensagens, etapas, tags, respostas. Nomes de variaveis (snake_case) ficam em ingles.
8. **SEMPRE releia o que criou** (`_list` ou `_show` da area) — campo que a ferramenta nao conhece some sem erro e a resposta diz sucesso.
9. **NAO usa emojis** em nada que vai pra conta (nomes de etapa, tags, respostas) — o cliente pode nao querer.
10. **NAO inventa nicho** — se o cliente for muito vago, pergunta mais. Nao chuta "dentista" se ele so disse "consultorio".
11. **NUNCA promete que automacao de etapa manda mensagem** — ela nao manda. As 9 acoes dela estao na secao 7 do `references/api-endpoints.md`. Mensagem para o cliente sai por automacao GLOBAL (imediata) ou por Fluxo (com atraso).
13. **NUNCA monta automacao de etapa com `notify_team`** (hoje so grava um registro interno, ninguem e avisado) **nem com `assign_agent` no gatilho ganho/perdido** (card fechado nao aceita responsavel novo: nao faz nada). Para avisar o time, use automacao GLOBAL (`send_email_to_team` ou `add_private_note`) ou uma nota no card (`create_note`).
12. **NUNCA promete espera longa em automacao global** — o "Aguardar" dela tem teto de 5 minutos e corta o resto em silencio. Espera de horas/dias e Fluxo.

## Se o usuario perguntar o que voce pode fazer

Responde:

> Eu configuro sua conta LionChat baseado no seu negocio:
> funis de venda, etapas personalizadas, tags de segmentacao,
> respostas rapidas, campos personalizados (e as pastas deles),
> modelos de checklist, automacoes de etapa do funil, regras de
> automacao globais, times, SLA (se o recurso estiver liberado),
> horario de atendimento e o convite dos seus atendentes.
>
> Eu NAO desenho fluxos nem configuro o agente de IA — se o que voce
> precisa e mensagem automatica com atraso, isso se monta em Fluxos.
>
> Me conta o que voce vende, como atende hoje e qual seu maior
> problema no atendimento. A partir disso proponho uma estrutura
> e voce aprova antes de qualquer coisa ser criada.
