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

## Heuristicas de automacoes de etapa

**Sempre que identificar gargalo de follow-up:**
- Automacao no `on_enter` da etapa de follow-up com `delay_seconds` configurado
- Ex: ao entrar em "Orcamento enviado", enviar mensagem 48h depois

**Atribuicao automatica:**
- Se time for maior que 1 pessoa, automacao ao criar card aplica `assign_team`
- Se tem SDR + Closer, automacao ao entrar em "Qualificado" muda de time

**Etiquetagem automatica:**
- Ao ficar 7 dias em uma etapa sem mover → aplicar tag `reativacao`
- Ao entrar em "Perdido" → aplicar tag `lead-perdido-motivo-X`

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

**Diferenca contato vs conversa:**
- **Contato** (model 0): dado do cliente (CPF, plano, cargo, cidade). Fica com ele sempre.
- **Conversa** (model 1): dado de uma negociacao especifica (valor_orcamento, data_visita, status_documentos). Muda por conversa.

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
