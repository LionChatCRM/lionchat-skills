# SLA: prazo prometido de atendimento

Indice:
1. O que o SLA faz
2. Os campos, todos em SEGUNDOS
3. O que cada relogio mede
4. Contar so no horario de atendimento
5. A politica sozinha nao mede nada
6. Uma conversa aceita UMA politica, para sempre
7. Como conferir o resultado
8. Receita completa

---

## 1. O que o SLA faz

E o cronometro da promessa de atendimento. Ele mede tres coisas: quanto tempo levou a primeira
resposta, quanto tempo levam as respostas seguintes e quanto tempo levou para resolver. Quando um
prazo estoura, a conversa e marcada e os participantes e administradores sao avisados.

O SLA depende de um recurso de plano. Se ele nao estiver liberado na conta, a criacao e recusada
(403). Nesse caso avise o cliente e siga - nao e erro de configuracao.

---

## 2. Os campos, todos em SEGUNDOS

`lionchat_sla_create` e `lionchat_sla_update`:

| Campo | O que e | Observacao |
|---|---|---|
| `name` | nome da politica | obrigatorio |
| `description` | descricao | opcional |
| `first_response_time_threshold` | prazo da PRIMEIRA resposta | SEGUNDOS; vazio significa nao medir |
| `next_response_time_threshold` | prazo das respostas SEGUINTES | SEGUNDOS |
| `resolution_time_threshold` | prazo para RESOLVER | SEGUNDOS |
| `only_during_business_hours` | contar so no horario de atendimento | nasce desligado |
| `resolution_check_delay` | quanto tempo a conversa precisa ficar resolvida antes de o resultado ser selado | SEGUNDOS; fecha a brecha do "resolve e reabre" |

**Todos os prazos sao em SEGUNDOS.** Escrever 30 pensando em 30 minutos cria um prazo de 30 segundos:
o SLA estoura em toda conversa e a conta enche de aviso de estouro. Conversao de bolso:

| O cliente disse | Voce escreve |
|---|---|
| 5 minutos | 300 |
| 15 minutos | 900 |
| 30 minutos | 1800 |
| 1 hora | 3600 |
| 4 horas | 14400 |
| 8 horas | 28800 |
| 24 horas | 86400 |
| 48 horas | 172800 |

Sempre repita o prazo em linguagem humana na proposta ("primeira resposta em 15 minutos") e ponha o
numero em segundos entre parenteses, para o cliente conferir.

Existe um ajuste adicional no produto que faz o cronometro da primeira resposta comecar quando alguem
ASSUME a conversa, em vez de quando ela nasce. Esse campo NAO esta disponivel nas ferramentas: se o
cliente pedir isso, mande ajustar no painel, em Configuracoes > SLA.

---

## 3. O que cada relogio mede

- **Primeira resposta**: do inicio da conversa (ou da atribuicao, se o ajuste de painel citado acima
  estiver ligado) ate a primeira resposta da equipe. Se o cronometro depender da atribuicao e
  ninguem assumir, ele nao conta.
- **Proxima resposta**: comeca a contar de quando o cliente voltou a esperar. So entra em cena depois
  que houve a primeira resposta.
- **Resolucao**: do inicio ate a conversa ser finalizada.

O resultado de cada conversa fica em um destes estados: em andamento, cumprido, estourado, ou em
andamento ja com algum estouro.

---

## 4. Contar so no horario de atendimento

Ligado, o cronometro conta apenas os segundos dentro do expediente. Quem manda no expediente:

- Chat do Site: o horario da PROPRIA caixa.
- Todos os outros canais: o horario da CONTA.

**Se o dono do horario nao estiver com o horario LIGADO, o sistema cai no 24 horas por dia** - que e
MAIS APERTADO que o expediente. O prazo passa a correr de madrugada e no fim de semana, e a unica
pista disso e uma linha de registro interno. O mesmo acontece se a grade estiver com todos os dias
fechados.

Antes de ligar esse ajuste, confirme com `lionchat_account_show` que o horario da conta esta ligado:
o campo que responde isso e `working_hours_enabled`. A grade dos sete dias vem OMITIDA na resposta
para economizar espaco - se precisar ver as faixas, peca ao cliente que confira no painel.

---

## 5. A politica sozinha nao mede nada

Criar a politica NAO faz nenhuma conversa passar a ser medida. Ela precisa ser APLICADA. Formas de
aplicar:

- Uma **regra de automacao** com a acao "Adicionar SLA" - o caminho normal.
  `lionchat_automation_rules_create`, tipicamente no gatilho de conversa criada, filtrando pela
  caixa.
- Uma **macro** com a mesma acao, quando a aplicacao e manual pelo atendente.
- Um **bloco de fluxo** com a acao de adicionar SLA.

Sempre crie a politica E a regra que a aplica, na mesma sessao. Politica sem regra e o erro mais
comum desta area: o cliente ve a politica na tela, acha que esta funcionando e descobre semanas
depois que nenhum numero foi medido.

---

## 6. Uma conversa aceita UMA politica, para sempre

Depois que uma politica e aplicada a uma conversa, ela **nao pode ser removida nem trocada**. A
tentativa e recusada.

Diga isso ao cliente ANTES de criar: se ele tem duvida sobre os prazos, e melhor comecar com uma
politica so, aplicada a uma caixa, e observar por alguns dias.

---

## 7. Como conferir o resultado

- `lionchat_sla_list` e `lionchat_sla_show`: as politicas que existem.
- `lionchat_sla_list_1`: as conversas que ESTOURARAM o prazo.
- `lionchat_sla_metrics`: os numeros no periodo.
- `lionchat_sla_download`: exportacao.

Na tela, o cliente encontra isso em Relatorios > SLA.

---

## 8. Receita completa

1. Confirme com `lionchat_account_show` que o recurso de SLA esta liberado.
2. Se o cliente quiser contar so no expediente, confirme em `lionchat_account_show` que
   `working_hours_enabled` esta ligado. Se nao estiver, peca para cadastrar e ligar no painel ANTES.
3. Converta os prazos para segundos e mostre a conversao na proposta.
4. Peca confirmacao.
5. `lionchat_sla_create`.
6. `lionchat_automation_rules_create` com a acao "Adicionar SLA" apontando para a politica criada,
   no gatilho de conversa criada e filtrando pela caixa certa.
7. Releia com `lionchat_sla_show` e explique ao cliente que os numeros comecam a aparecer nas
   conversas NOVAS - as antigas nao sao medidas retroativamente.
