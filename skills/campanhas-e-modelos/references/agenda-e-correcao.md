# Acompanhar o disparo e corrigir o que der errado

Indice:

1. Como saber se o disparo terminou
2. O placar de entrega (campanha de mensagem)
3. Os 7 motivos de falha, e o que fazer em cada um
4. Reenviar as falhas
5. A agenda dia a dia
6. As 5 operacoes de agenda
7. Parar x remarcar
8. Perguntas que o cliente faz e a resposta certa

---

## 1. Como saber se o disparo terminou

**O selo "Concluida" NAO quer dizer que terminou de enviar.** A campanha vira concluida no instante
em que o disparo COMECA, para nenhum outro trabalhador pega-la de novo. O cliente pode ver
"Concluida" com mil pessoas ainda na fila.

Onde olhar de verdade:

| Tipo de campanha | Onde olhar |
|---|---|
| Mensagem, saindo de uma vez | `lionchat_campaigns_statistics` — o total ja e o que saiu |
| Mensagem em lotes diarios | `lionchat_campaigns_show`, o que o sistema guardou sobre a fila de lotes |
| Campanha de Fluxo | `lionchat_campaigns_flow_report` — `pending` e quem ainda falta |
| QR Code com teto por dia | `lionchat_campaigns_dispatch_days` — o dia a dia |

Efeito colateral do selo: **campanha concluida nao aceita edicao**. Na pratica so da para editar
campanha AGENDADA que ainda nao chegou a hora. Para repetir um disparo, use "Reusar configuracoes"
no menu de tres pontinhos do card, que abre uma campanha nova ja preenchida (o agendamento vem
sempre em branco, e uma lista de planilha nao volta).

---

## 2. O placar de entrega (campanha de mensagem)

`lionchat_campaigns_statistics` devolve:

| Campo | Nome na tela | O que e |
|---|---|---|
| `total` | | Todas as mensagens criadas, inclusive as que falharam |
| `sent` | sem confirmacao | O canal aceitou, mas ainda nao confirmou a entrega |
| `delivered` | entregues | Chegou no aparelho |
| `read` | lidas | Foi lida |
| `failed` | falhas | Nao chegou |
| `delivery_rate` | Entrega | `(entregues + lidas) / total` |
| `read_rate` | Leitura | `lidas / total` |
| `failure_breakdown` | Por que falharam | A quebra por motivo |

Os quatro baldes sao EXCLUSIVOS e somam o total: quem LEU saiu do balde "entregues". Por isso a taxa
de entrega soma entregues mais lidas — mensagem lida foi obrigatoriamente entregue.

Acima de 5.000 falhas a quebra por motivo passa a ser por amostra; o total de falhas continua exato.

---

## 3. Os 7 motivos de falha, e o que fazer em cada um

| Motivo | Nome na tela | O que aconteceu | O que fazer |
|---|---|---|---|
| `permanent` | Numero invalido ou modelo rejeitado | Numero que a Meta nao alcanca, ou modelo recusado | Reenviar nao resolve. Conferir o telefone na ficha |
| `limit` | Limite de marketing da Meta | A Meta limita quantas mensagens de marketing cada PESSOA recebe | Esperar 24 horas |
| `spam_limit` | Limite de spam (Meta) | A Meta barrou o numero por spam | Reduzir o volume e melhorar a lista. Nao insistir |
| `account_locked` | Conta bloqueada pela Meta | A Meta trancou a conta comercial | So se resolve no painel da Meta |
| `billing` | Saldo/pagamento da Meta | Cobranca pendente | Regularizar e reenviar |
| `window_expired` | Janela de 24h vencida | Tentou texto livre fora da janela | Mandar como modelo |
| `transient` | Erro comum | Temporario (instabilidade, tempo esgotado) | Reenviar costuma resolver |

**Aviso que precisa ser dito sempre**: "numero invalido" NAO autoriza voce a dizer
ao cliente que aquela pessoa nao tem WhatsApp. A Meta usa o mesmo aviso para "nao tem WhatsApp" e
para "a pessoa nao esta recebendo" (bloqueou o numero, aparelho desligado). Ja houve atendente
encerrando a conversa de um paciente que existia. Diga "a Meta nao conseguiu entregar" e sugira
conferir o telefone.

---

## 4. Reenviar as falhas

Ferramenta: `lionchat_campaigns_resend_failures`. **Manda mensagem de verdade: confirme antes.**

- So funciona em campanha de caixa WhatsApp OFICIAL. Qualquer outro tipo e recusado.
- O numero devolvido e a contagem BRUTA de falhas. O que sai de verdade e menor, porque o sistema
  aplica sozinho: uma tentativa por pessoa, quem ja recebeu por outro caminho fica de fora, numero
  marcado como invalido nunca e reenviado, falha por limite so volta depois de 24 horas, e o teto
  da Meta continua valendo.
- O reenvio reusa o MESMO balao na conversa; nao cria mensagem repetida.

**Nunca ofereca reenviar campanha pelo botao do balao nem pelo painel de falhas da caixa**
(`lionchat_inboxes_failed_messages_bulk_retry`). Mensagem de campanha e excluida dali de proposito:
tres baloes de falha viravam tres cliques e tres envios reais ao mesmo cliente.

O painel de falhas da caixa (`lionchat_inboxes_failed_messages_summary`) continua util para
DIAGNOSTICO — ele mostra a saude geral daquele numero, com as falhas de campanha num grupo proprio.

---

## 5. A agenda dia a dia

Existe em Campanha de Fluxo e em campanha de QR Code criada COM teto por dia. Campanha de mensagem
comum e campanha do site nao tem agenda (a chamada e recusada).

`lionchat_campaigns_dispatch_days` traz, para cada dia, no fuso da conta:

- quantas pessoas estavam planejadas, quantas ja dispararam, quantas estao pendentes, quantas foram
  puladas (com os motivos), canceladas e falhadas;
- a janela de horario do dia;
- as MENSAGENS que sairam naquele dia;
- o teto por dia, a previsao de termino e o historico das edicoes da agenda.

**Duas contagens que nunca somam e nunca devem ser confundidas:**

- **disparos** = quantas PESSOAS o fluxo comecou naquele dia;
- **mensagens** = quantas MENSAGENS sairam naquele dia.

Um fluxo com bloco de espera cria mensagem dias depois do disparo. Explique isso antes de o cliente
achar que os numeros estao errados.

`lionchat_campaigns_dispatch_day_entries` traz QUEM sai num dia especifico: contato, telefone,
minuto, situacao e motivo do pulo. Devolve no maximo 100 linhas por vez, com o total real ao lado. E
aqui que se descobre o identificador de uma pessoa para remove-la.

---

## 6. As 5 operacoes de agenda

| Ferramenta | O que faz | Destrutivo? |
|---|---|---|
| `lionchat_campaigns_skip_dispatch_day` | Pula o dia. As pessoas vao para o FIM da fila | nao |
| `lionchat_campaigns_move_dispatch_day` | Leva as pendentes de um dia para outro dia | nao |
| `lionchat_campaigns_shift_dispatch_from` | Empurra um dia E todos os seguintes pelo mesmo numero de dias | nao |
| `lionchat_campaigns_reprogram_dispatch` | Refaz a agenda so do que falta, com teto, inicio e intervalo novos | nao |
| `lionchat_campaigns_remove_dispatch_entry` | Tira UMA pessoa da campanha | SIM |

Regras que precisam ser ditas ao cliente:

- **Pular NAO cancela ninguem.** As pessoas vao para o fim da fila e a campanha termina mais tarde.
  Quem espera que "pular" reduza o alcance vai se surpreender.
- **Mover recusa** quando o dia de destino nao cabe no teto. Nesse caso o certo e empurrar dali.
- **Empurrar dali e a operacao certa para "pula o domingo"** quando mover um dia so nao cabe.
- **Reprogramar nunca toca em quem ja recebeu.** Parametros que voce nao mandar mantem o valor
  atual.
- **Remover e a UNICA operacao que cancela alguem de verdade.** A linha vira cancelada e nao some,
  para o dono saber que a pessoa ficou de fora. Nao redistribui nada.

Datas de agenda vao no formato ano-mes-dia, no fuso da conta.

---

## 7. Parar x remarcar

Quatro ferramentas, duas destrutivas e duas nao. **Ofereca sempre a nao destrutiva primeiro.**

| Ferramenta | Vale para | O que faz | Desfazer? |
|---|---|---|---|
| `lionchat_campaigns_reschedule_flow` | Campanha de Fluxo | Empurra a fila para outro horario, preservando o espacamento | nao precisa |
| `lionchat_campaigns_stop_flow` | Campanha de Fluxo | Cancela quem ainda nao foi disparado | NAO TEM |
| `lionchat_campaigns_reschedule_batch` | Oficial em lotes diarios | Muda a hora do proximo lote (adia ou antecipa) | nao precisa |
| `lionchat_campaigns_stop_batches` | Oficial em lotes diarios | Esvazia a fila de lotes que faltam | NAO TEM |

Antes de chamar qualquer uma das duas destrutivas:

1. Pegue quantas pessoas ainda estao na fila (`lionchat_campaigns_flow_report`, campo `pending`,
   ou a agenda).
2. Diga esse numero ao cliente com todas as letras: "isso deixa N pessoas de fora, e nao tem como
   voltar atras — para retomar seria preciso criar a campanha do zero".
3. Espere o sim.

O que ja saiu, as conversas criadas e os numeros do relatorio ficam intactos nos dois casos: so a
fila e esvaziada.

Data e hora nas duas de remarcar precisam ir COM fuso e no futuro (por exemplo
`2026-09-02T09:00:00-03:00`). Sem fuso a hora e lida no fuso do servidor e o disparo sai na hora
errada — normalmente 3 horas fora, o que num disparo de madrugada vira reclamacao de cliente.

---

## 8. Perguntas que o cliente faz e a resposta certa

**"Ja terminou?"** — Olhe a agenda ou o relatorio, nunca o selo. O selo vira "Concluida" quando o
disparo comeca.

**"Por que fulano nao recebeu?"** — Em campanha de mensagem, procure a pessoa no placar de falhas e
leia o motivo. Em Campanha de Fluxo, use a lista de pessoas do dia e leia o motivo do pulo.

**"Por que esta tao devagar?"** — Em QR Code, e o intervalo entre pessoas somado ao teto por dia,
de proposito. Em caixa oficial, pode ser o freio automatico de velocidade, o teto diario da Meta, ou
o disjuntor de spam, que pausa o disparo por 30 minutos e volta sozinho.

**"Parou sozinha?"** — Em caixa oficial, provavelmente o disjuntor de spam. Ele volta sozinho.

**"Continuou saindo depois de bater o limite"** — No modo de lotes diarios, quando uma leva termina e
ainda ha folga no dia, o sistema manda a proxima leva sozinho, reconsultando o teto da Meta ao vivo.
No modo "ate o limite e para" isso nao acontece.

**"Criei a campanha e nao saiu nada"** — Confira, nesta ordem: publico vazio; caixa oficial com a
funcionalidade de campanha desligada na conta; caixa por QR Code desconectada; horario agendado que
passou ha mais de 3 dias sem o disparo ser pego (nesse caso ela nunca mais sai e o certo e criar
outra).

**"Nao consigo editar a campanha"** — Ela ja disparou. O caminho e "Reusar configuracoes".

**"Quero pausar"** — Nao existe pausar disparo em massa. O liga e desliga so vale para a campanha do
chat do site. O que existe e remarcar, pular dia, mover dia ou parar (destrutivo).
