# Principios de Design de Funil

Voce NAO tem templates prontos por nicho. Voce usa seu conhecimento de CRM + as informacoes do cliente para montar algo sob medida.

## Heuristicas de etapas

### Por ciclo de venda

| Ciclo | Etapas sugeridas |
|-------|------------------|
| **Impulso** (minutos/horas) | 3 etapas: Novo → Atendendo → Fechado |
| **Curto** (1-7 dias) | 4-5 etapas: Novo → Qualificando → Proposta → Negociando → Fechado |
| **Medio** (1-4 semanas) | 6-7 etapas: Novo → Qualificando → Reuniao/Demo → Proposta → Negociando → Fechando → Ganho/Perdido |
| **Longo** (1+ mes, B2B alto ticket) | 7-9 etapas: Prospeccao → Descoberta → Demo → Proposta → Avaliacao Tecnica → Negociacao Juridica → Aprovacao → Assinatura → Onboarding |

### Por modelo de negocio

**B2C impulso (e-commerce, delivery):**
- Foco em carrinho abandonado
- Etapas simples, sem qualificacao
- Tags: `carrinho-abandonado`, `cupom-enviado`, `concluiu-compra`

**B2C considerado (curso, consultoria, clinica):**
- Etapa de "avaliacao" ou "diagnostico" e critica
- Follow-up e o gargalo
- Tags de objecao sao importantes (`objecao-preco`, `objecao-tempo`)

**B2B low-ticket (SaaS self-service):**
- Funil curto, foco em trial/onboarding
- Etapas: Trial ativo → Usando features chave → Pagou primeira mensalidade

**B2B high-ticket (enterprise):**
- Funil longo com discovery, demo, proposta
- Multiplos decisores (campo customizado: "decisor_principal")
- SLA importante

### Por origem do lead

Se o cliente disser "meus leads vem de X canais", crie tags de origem:
- `fonte-instagram`, `fonte-google-ads`, `fonte-indicacao`, `fonte-site`, `fonte-whatsapp-direto`

Isso permite relatorio por ROI de canal depois.

## Heuristicas de tags

**SEMPRE cria tag de origem** — "de onde veio o lead" e a segmentacao mais usada.

**Tags de objecao** (se cliente mencionou):
- `objecao-preco` — reclamou de caro
- `objecao-prazo` — precisa de mais tempo pra decidir
- `objecao-confianca` — desconfiou, pediu prova social
- `objecao-forma-pagamento` — quer parcelar, boleto, etc.

**Tags de prioridade** (se time for > 3 pessoas):
- `prioridade-alta`, `prioridade-media`, `prioridade-baixa`

**Tags de status transitorio** (cuidado pra nao poluir):
- `aguardando-documentos`, `em-analise-credito`, `aguardando-retorno`

**NAO criar tag para coisa que e melhor como campo customizado:**
- Errado: tag `valor-10k-20k`, `valor-20k-50k`
- Certo: campo customizado `valor_orcamento` (numerico, currency)

## Heuristicas de respostas rapidas

**Minimo viavel (sempre criar):**
1. `/boasvindas` — primeira mensagem quando cliente chega
2. `/horariofora` — resposta automatica fora do expediente
3. `/aguarde` — "aguarde um momento enquanto verifico"
4. `/agradecimento` — mensagem de fechamento

**Recomendadas conforme negocio:**
- Com agendamento: `/confirmacao`, `/lembrete`, `/reagendar`
- Com proposta: `/proposta`, `/followup48h`, `/followup7d`
- Com pagamento: `/linkpagamento`, `/boleto`, `/pixenviado`
- Com objecoes: `/objecao-preco` (resposta para reclamacao de preco)

**NUNCA criar respostas genericas tipo "/resposta1", "/resposta2".** Todo short_code deve ser semantico.

## Heuristicas de automacoes

Existem DUAS coisas diferentes com o mesmo apelido. Escolher a errada e o erro mais comum aqui.

| | Automacao de ETAPA | Automacao GLOBAL |
|---|---|---|
| Onde mora | dentro do funil (`settings.automations`) | recurso proprio (`automation_rules`) |
| Age sobre | o card | a conversa (e o card, quando o evento e de card) |
| Manda mensagem? | **NAO** | **SIM**, na hora |
| Tem atraso? | nao | so ate 5 minutos |
| Serve pra | organizar o card: checklist, atendente, nota, mover, avisar time, webhook | falar com o cliente, etiquetar, atribuir, mexer no card |

**Regra de ouro: mensagem com ATRASO nao e automacao — e Fluxo.** "Cobrar em 48h quem nao respondeu"
NAO se monta aqui. A automacao de etapa nao manda mensagem nenhuma, e a espera da automacao global
para em 5 minutos (o resto e cortado em silencio). Ao identificar gargalo de follow-up, monte a
estrutura e **avise o cliente que a cobranca automatica se monta em Fluxos**, que tem espera de horas
e dias. Nao prometa o que a automacao nao faz.

**Nao existe gatilho de tempo parado.** "Ao ficar 7 dias na mesma etapa" nao e um gatilho do sistema —
nem na automacao de etapa (que so tem card criado, card chegou na etapa, card ganho/perdido) nem na
global. Quem faz espera e o Fluxo.

**O que a automacao de ETAPA resolve bem:**
- Ao criar o card → aplicar o checklist daquela fase (`apply_checklist_template`)
- Ao chegar numa etapa → atribuir o atendente responsavel (`assign_agent`)
- Ao marcar ganho/perdido → escrever nota interna (`create_note`) ou avisar o time (`notify_team`)
- Ao chegar numa etapa → disparar webhook pra um sistema de fora (`send_webhook`)

**O que a automacao GLOBAL resolve bem:**
- Conversa nova numa caixa → atribuir ao time certo (`assign_team`)
- Card entrou na etapa X → mandar mensagem pro cliente **na hora** (`send_message`)
- Card foi para "Perdido" → aplicar etiqueta na conversa (`add_label`)
- Conversa resolvida → etiquetar, mudar prioridade

**Atencao com evento de card na automacao global:** ali so valem dez acoes (`send_message`,
`add_label`, `change_priority`, `assign_agent_to_kanban_item`, `add_note_to_kanban_item`,
`move_kanban_item_to_stage`, `set_kanban_item_status`, os dois de cronometro e `send_webhook_event`).
Escolher outra — `assign_team`, por exemplo — cria a regra, mostra ela ativa e **nao faz nada**.

**NAO criar automacao de mensagem spam:**
- Maximo 2 mensagens automaticas por cliente em 48h
- Espacar minimo 24h entre envios

## Heuristicas de campos personalizados

**Sempre que o cliente mencionar um dado que precisa capturar, vira campo:**

Exemplo: cliente diz "preciso saber o plano de saude do paciente"
→ Campo `plano_saude` (tipo list, com opcoes: Unimed, Bradesco, SulAmerica, Particular)

**Tipos de campo comuns por negocio:**

| Nicho | Campos tipicos |
|-------|---------------|
| Odontologia/Saude | tipo_procedimento (list), plano_saude (list), valor_orcamento (currency), data_avaliacao (date) |
| Advocacia | tipo_processo (list), valor_causa (currency), data_audiencia (date) |
| Imobiliaria | tipo_imovel (list), faixa_preco (list), cidade_interesse (text), data_visita (date) |
| Curso online | nivel_conhecimento (list), objetivo (text), horario_preferido (list) |
| E-commerce | ticket_medio (currency), produto_interesse (list), forma_pagamento (list) |
| SaaS B2B | cargo (text), tamanho_empresa (list), integracao_necessaria (list) |

**Os 3 tipos de atributo personalizado:**

| Tipo | Quando usar | Exemplo | Onde fica |
|------|-------------|---------|-----------|
| **Contato** (`attribute_model=1`) | Dado do cliente que nao muda (ou muda pouco). Fica com ele pra sempre, independe da conversa atual. | CPF, plano de saude, cargo, cidade, aniversario | `custom_attribute_definitions` (endpoint 2.1) |
| **Conversa** (`attribute_model=0`) | Dado de uma negociacao especifica. Muda por conversa. | valor_orcamento, data_visita, status_documentos, canal_origem | `custom_attribute_definitions` (endpoint 2.1) |
| **Card Kanban** (`kanban_item_attribute`) | Dado que e **do card do Kanban** — ou seja, pertence ao negocio/oportunidade representada pelo card. Visivel no detalhe do card. | proposta_enviada_em, tipo_contrato, comissao, concorrente_principal | `kanban_config.global_custom_attributes` (endpoint 2.2) |

**Como decidir qual usar:**

- "Esse dado fica com o cliente sempre?" → **Contato**
- "Esse dado e da negociacao atual (vai mudar se for outra negociacao)?" → **Conversa**
- "Esse dado e do card no pipeline (ex: proposta, comissao, motivo de perda)?" → **Card**

**Exemplo pratico (odontologia):**
- **Contato**: plano_saude, cpf, data_nascimento (caracteristicas do paciente)
- **Conversa**: tipo_procedimento_consultado, preferencia_dia_semana (dados da negociacao atual)
- **Card**: valor_orcamento, motivo_perda, forma_pagamento_escolhida (dados do card de vendas)

Muitas vezes o mesmo nome de atributo pode existir em 2 lugares, com significados diferentes. Ex: `valor_orcamento` pode ser da conversa (valor estimado durante o atendimento) E do card (valor final da proposta). Pergunte ao cliente pra nao criar redundancia.

**Tipos de campo do Card (diferentes dos outros):**

Card aceita: `string`, `number`, `date`, `boolean`, `time` e `datetime`. Se for lista, e
`type: string` + `is_list: true`. Nao existe `currency`, `percent`, `link` nem `checkbox` como no
contato/conversa — "Link" vira `string` e "Caixa de selecao" vira `boolean`. O servidor nao recusa um
tipo inventado: ele aceita e o campo aparece como texto comum. Use so os seis.

**Checklist de card:** um modelo de checklist e uma lista de tarefas pronta (ex: "Enviar proposta" com
4 passos) que pode ser aplicada ao card na mao ou por automacao de etapa. Vale criar um modelo por
fase que tenha um roteiro repetitivo — e o jeito mais barato de padronizar o time sem treinar
ninguem. Ficam em `kanban_config.checklist_templates` (secao 2.4 do manual de endpoints).

## Heuristicas de SLA

**So criar se:**
- Cliente mencionou prazo explicitamente ("tenho que responder em 1h")
- Cliente reclamou de tempo de resposta
- Negocio e regulado (saude, juridico)

**Valores padrao razoaveis:**
- Primeira resposta: 10-30 min (impulso/B2C) | 1-4h (B2B)
- Proxima resposta: 30 min (impulso) | 2h (B2C) | 8h (B2B)
- Resolucao: 4h (impulso) | 24h (B2C) | 72h (B2B)

Sempre ligar `only_during_business_hours: true` (senao SLA corre de madrugada).

## Heuristicas de times

**Criar times so se:**
- Time maior que 3 pessoas
- Tem especializacao clara (vendas vs suporte vs financeiro)

**Sugestoes por porte:**
- Solo: nenhum time (nao precisa)
- 2-3 pessoas: 1 time so (ex: "Atendimento")
- 4-10 pessoas: 2-3 times (ex: "Vendas", "Suporte", "Financeiro")
- 10+: times por especializacao + SDR/Closer se B2B

## Principio geral

**Menos e mais.** Melhor entregar 1 funil de 5 etapas bem pensadas com 8 tags essenciais do que 3 funis de 12 etapas com 40 tags que o cliente nao vai usar.

Ao propor, sempre se pergunte: "Se o cliente fosse usar isso amanha, ele entenderia intuitivamente?"

Se a resposta for nao, simplifica.
