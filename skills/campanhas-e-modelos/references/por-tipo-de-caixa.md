# O que muda em cada tipo de campanha

Indice:

1. Tabela comparativa
2. WhatsApp Oficial
3. WhatsApp por QR Code
4. Campanha de Fluxo
5. SMS
6. Chat do site
7. SMS, torpedo de voz e URA pelo fornecedor externo
8. As travas anti-bloqueio, uma a uma
9. Variaveis no texto da mensagem

---

## 1. Tabela comparativa

| | WhatsApp Oficial | WhatsApp QR Code | Campanha de Fluxo | SMS | Chat do site |
|---|---|---|---|---|---|
| Como e a caixa | `Channel::Whatsapp` | `Channel::Waha` ou `Channel::Api` | qualquer uma das duas de WhatsApp | `Channel::Sms` ou `Channel::TwilioSms` | `Channel::WebWidget` |
| Precisa de modelo aprovado | SIM | Nao | So em caixa oficial | Nao | Nao |
| Janela de 24 horas | SIM | Nao existe | Herda a da caixa | Nao existe | Nao existe |
| Teto diario da Meta | SIM | Nao existe | So em caixa oficial | Nao | Nao |
| Criterios de publico que funcionam | os 7 | os 7 | os 7 | so etiqueta de CONVERSA | nao tem publico |
| Intervalo entre pessoas | ritmo automatico | 20 a 40 segundos (ajustavel) | 20 a 40 segundos (ajustavel) | sem intervalo | nao se aplica |
| Maximo por dia | o que a Meta libera | 500 por padrao | 500, ou o teto da Meta em caixa oficial | nao tem | nao se aplica |
| Variacoes de texto | nao | SIM | o texto sai dos blocos do fluxo | nao | nao |
| Acoes depois de enviar | SIM, na confirmacao de entrega | SIM, ao fim de cada rodada | os blocos do fluxo fazem | nao | nao |
| Placar de entrega | SIM | SIM | SIM (as mensagens do fluxo contam) | nao cria mensagem, entao nao tem | nao se aplica |
| Reenviar falhas | SIM | nao | nao | nao | nao |
| Agenda dia a dia editavel | NAO | SIM, quando ha teto por dia | SIM | nao | nao |
| Anexo no disparo | so o cabecalho do modelo | NAO | o que os blocos do fluxo mandarem | NAO | nao |
| Risco principal | teto da Meta e recusa por modelo | numero ser bloqueado pelo WhatsApp | os dois, conforme a caixa | custo por mensagem | nenhum |

Excecao a conferir: uma caixa Twilio configurada para WhatsApp aparece com o tipo de canal de SMS
mas se comporta como WhatsApp. Se a caixa for Twilio, confirme com o cliente se ela e de SMS ou de
WhatsApp antes de montar o publico.

---

## 2. WhatsApp Oficial

Como criar, resumido:

```json
{
  "title": "Aviso Black Friday",
  "inbox_id": 12,
  "message": "espelho do corpo do modelo",
  "audience": [{ "type": "ContactLabel", "id": 44 }],
  "trigger_rules": { "audience_mode": "sum", "over_limit_mode": "batches" },
  "template_params": {
    "name": "aviso_black_friday", "language": "pt_BR", "category": "MARKETING",
    "processed_params": { "body": { "1": "{{contact.name}}" } },
    "bulk_actions": { "labels": [7], "contact_labels": [44], "assignee_id": 3 }
  }
}
```

Pontos que so existem aqui:

- **A funcionalidade de campanha de WhatsApp oficial nasce DESLIGADA na conta.** Com ela desligada
  nada sai e nao aparece erro nenhum: a campanha fica ATIVA para sempre, sendo retentada por 3 dias,
  e depois some. O cliente ve "campanha ativa que nunca saiu". Ligar e trabalho do suporte — nao ha
  ferramenta para isso. Se voce nao conseguir confirmar que esta ligada, avise antes de disparar.
- O campo `message` continua obrigatorio, mas e so um espelho: quem manda de verdade e o modelo.
- Existe um freio de velocidade automatico, por numero, que DESACELERA sozinho quando a Meta
  reclama de velocidade e volta ao normal depois de um tempo em silencio. O cliente nao configura.
- **Nao existe agenda dia a dia aqui.** Mesmo no modo de lotes diarios, as ferramentas de agenda
  (`dispatch_days`, pular, mover, empurrar, reprogramar, remover) recusam a chamada. Para acompanhar
  os lotes use `lionchat_campaigns_show`; para mexer neles, `reschedule_batch` e `stop_batches`.
- Existe um disjuntor de spam: se a Meta acusar spam muitas vezes na mesma campanha, o disparo PAUSA
  sozinho por 30 minutos e volta sozinho, dobrando a pausa a cada nova ocorrencia ate 4 horas.
  Ninguem e cancelado — os envios voltam a fluir.
- Numero que a Meta ja marcou como "nao existe no WhatsApp" e pulado nos disparos seguintes, para
  nao queimar vaga do teto diario. Entrega confirmada depois limpa essa marca.
- As acoes pos-envio (etiqueta, atendente, prioridade, atributo) so sao aplicadas quando a Meta
  CONFIRMA a entrega. Podem levar de segundos a minutos. Quem falhou nao recebe a marca, de
  proposito, para nao furar a exclusao do proximo disparo.

---

## 3. WhatsApp por QR Code

```json
{
  "title": "Reativacao de clientes",
  "inbox_id": 30,
  "message": "Oi {{contact.name}}, tudo bem? ...",
  "audience": [{ "type": "Label", "id": 5 }],
  "trigger_rules": { "audience_mode": "sum" },
  "template_params": {
    "delay_min": 25, "delay_max": 60, "daily_cap": 300,
    "variations": ["Oi {{contact.name}}! ...", "Ola {{contact.name}}, ..."],
    "bulk_actions": { "contact_labels": [44] }
  }
}
```

Pontos que so existem aqui:

- Texto livre, sem modelo e sem janela — mas com risco real de bloqueio do numero.
- **Se `daily_cap` NAO for enviado, o sistema assume 500 por dia** e monta uma agenda de varios
  dias. Uma lista de 3.000 pessoas vira 6 dias de disparo, e o cliente costuma achar que travou.
  Para mandar tudo no mesmo dia, o campo precisa ir explicitamente vazio ou zero.
- As variacoes alternam com a mensagem principal por rodizio (posicao na fila), nao por sorteio.
- Nao existe reenvio de falhas.
- Nao existe anexo: so texto.
- **A agenda dia a dia so existe na caixa conectada por QR Code (`Channel::Waha`).** Numa caixa
  `Channel::Api` a campanha sai do mesmo jeito, mas as ferramentas de agenda recusam a chamada.
- No painel existe uma caixa de marcar de responsabilidade que o conector nao exige. Como ela nao
  aparece por aqui, **faca esse aviso por escrito**: disparo em massa por QR Code esta sujeito aos
  termos da Meta e pode levar ao bloqueio do numero.

---

## 4. Campanha de Fluxo

Em vez de mandar uma mensagem, comeca um fluxo para cada pessoa da lista, cadenciado.

```json
{
  "title": "Qualificacao de leads da feira",
  "inbox_id": 12,
  "flow_id": 44,
  "audience": [{ "type": "ContactLabel", "id": 51 }],
  "trigger_rules": { "audience_mode": "sum" },
  "template_params": { "delay_min": 30, "delay_max": 90, "daily_cap": 200 }
}
```

- **Nao mande `message`** — o texto sai dos blocos do fluxo.
- O fluxo precisa: existir, ser da mesma conta, ser fluxo de conversa, estar ATIVO, ter o gatilho
  Campanha ligado no bloco Inicio e estar vinculado a MESMA caixa da campanha. Se qualquer regra
  falhar, a criacao e recusada dizendo qual. Ache os fluxos elegiveis com `lionchat_flows_list`
  passando `with_campaign_trigger: true` e o `inbox_id`.
- **Em caixa oficial, o primeiro bloco de mensagem de CADA CAMINHO do fluxo precisa ser um modelo
  aprovado.** Com sorteio ou condicao logo no inicio, TODAS as variacoes contam — nao basta a
  primeira. A recusa nomeia ate 5 blocos culpados.
- O teto por dia conta PESSOAS, nao mensagens. **Se o fluxo manda 4 mensagens por pessoa, o volume
  real e 4 vezes o que o teto sugere.** Diga isso ao cliente ao sugerir o numero.
- Os dois freios automaticos da caixa oficial valem aqui tambem: o disjuntor de spam pausa a
  campanha de fluxo inteira, e o freio de velocidade segura a mensagem que sai de um bloco de fluxo.
  Mesmo assim, como cada pessoa pode receber varias mensagens, o teto por dia merece folga.
- Quem ja esta com esse MESMO fluxo rodando e pulado (para nao receber a sequencia em dobro).
  Conversa aberta com atendente e sessao de OUTRO fluxo nao pulam.
- Conversa encerrada e REABERTA em vez de virar uma nova.

Os 10 motivos de pulo que aparecem no relatorio, e o que dizer ao cliente:

| Motivo | O que aconteceu |
|---|---|
| `sessao_ja_ativa` | A pessoa ja estava nesse mesmo fluxo |
| `contato_sem_telefone` | A ficha nao tem telefone |
| `template_necessario` | Caixa oficial e algum caminho do fluxo comeca sem modelo aprovado |
| `caixa_desvinculada` | O fluxo deixou de estar ligado a caixa da campanha |
| `flow_removido` | O fluxo foi apagado |
| `flow_inativo` | O fluxo foi desativado |
| `gatilho_removido` | Tiraram o gatilho Campanha do bloco Inicio |
| `conversa_falhou` | Nao deu para abrir a conversa daquela pessoa |
| `sessao_duplicada` | Duas tentativas ao mesmo tempo; so uma valeu |
| `erro_interno` | Falha inesperada naquela pessoa; as outras seguem |

---

## 5. SMS

**So a etiqueta de CONVERSA funciona no publico.** Montar publico de SMS por etiqueta de contato,
funil, atributo ou atendente resolve zero pessoas e nada avisa. A exclusao, essa sim, funciona
normalmente.

O disparo de SMS nao cria mensagem no sistema, entao **nao existe placar de entrega nem reenvio de
falhas** para ele. Tambem nao existem acoes pos-envio nem modo de combinacao.

---

## 6. Chat do site

E o unico tipo que NAO e disparo. E uma mensagem que a bolha do chat mostra sozinha quando o
visitante entra numa pagina.

```json
{ "title": "Ajuda na pagina de precos", "inbox_id": 4,
  "message": "Posso ajudar a escolher o plano?",
  "trigger_rules": { "url": "https://site.com/precos", "time_on_page": 20 },
  "enabled": true, "trigger_only_during_business_hours": true }
```

- O endereco precisa comecar com `http://` ou `https://`.
- **Este e o unico tipo em que o liga e desliga funciona de verdade**, e tambem o unico em que
  "so no horario comercial" funciona. Nos disparos em massa esses dois campos existem mas nao fazem
  nada.
- E o unico tipo que aceita edicao depois de criado, porque nunca fica concluido.

---

## 7. SMS, torpedo de voz e URA pelo fornecedor externo

Familia separada, por um fornecedor externo, com ferramentas proprias
(`lionchat_topsend_campaigns_create` e `lionchat_topsend_campaigns_list`). Exige a integracao
configurada e perfil de administrador. Nao tem edicao: so criar e apagar.

| Formato | O que e |
|---|---|
| `SMSSHORT` | SMS comum, ate 160 caracteres |
| `SMSFLASH` | SMS que aparece na tela do celular sem ir para a caixa de mensagens |
| `TVOZ` | Torpedo de voz: toca um audio |
| `URA` | Toca o audio e capta a tecla que a pessoa digitar |

- Torpedo de voz e URA exigem audio, por identificador ja existente no fornecedor ou por endereco de
  MP3 externo. O conector nao sobe arquivo.
- Na URA, cada tecla pode acionar uma automacao.
- Os destinatarios vem por criterios de publico ou por uma lista de numeros informada na mao.

---

## 8. As travas anti-bloqueio, uma a uma

| Trava | Onde vale | Configuravel? | O que faz |
|---|---|---|---|
| Intervalo aleatorio entre envios | QR Code e Fluxo | sim (`delay_min`, `delay_max`, em segundos) | Faz o disparo parecer humano. Padrao 20 a 40 |
| Variacoes de texto | QR Code | sim (`variations`) | Evita mandar texto identico centenas de vezes seguidas |
| Maximo por dia | QR Code e Fluxo | sim (`daily_cap`) | Espalha o disparo por varios dias |
| Freio de velocidade | Oficial (mensagem e fluxo) | nao | Limita mensagens por segundo e desacelera sozinho |
| Disjuntor de spam | Oficial (mensagem e fluxo) | nao | Pausa a campanha sozinho quando a Meta acusa spam, e volta sozinho |
| Pulo de quem nao tem WhatsApp | Oficial | nao | Nao queima vaga do teto diario com numero invalido |
| Teto diario da Meta | Oficial | nao (a Meta define) | Ver `references/publico-e-exclusao.md` |

**O maximo por dia do QR Code NAO e garantia de seguranca.** Numero de QR Code nao tem limite
publicado pela Meta; 500 e um padrao escolhido pela casa. Diga o que a funcao FAZ (espalha o disparo
por varios dias), nunca que ela protege contra bloqueio.

Recomendacoes praticas para QR Code, se o cliente pedir um numero:

- Numero novo ou pouco usado: comece bem abaixo (algo entre 50 e 150 por dia) e intervalo maior.
- Lista fria (gente que nunca falou com a empresa): e o cenario de maior risco. Diga isso.
- Lista de clientes que ja conversam com a empresa: risco muito menor.

---

## 9. Variaveis no texto da mensagem

Valem no texto livre do QR Code e do SMS, e nos valores das variaveis do modelo oficial.

| Variavel | O que traz |
|---|---|
| `{{contact.name}}` | Nome do contato |
| `{{contact.first_name}}` | Primeiro nome |
| `{{contact.phone_number}}` | Telefone |
| `{{contact.email}}` | E-mail |
| `{{contact.custom_attribute.chave}}` | Atributo personalizado da ficha |
| `{{agent.name}}` | Nome do atendente remetente, se houver |
| `{{inbox.name}}` | Nome da caixa |
| `{{account.name}}` | Nome da empresa |
| `{{account.custom_attribute.chave}}` | Variavel da conta |

**A variavel da conta exige a forma COMPLETA**: `{{account.custom_attribute.slogan}}`. Nao existe
atalho `{{slogan}}` — sem o pedaco do meio a variavel sai VAZIA, sem erro. As chaves disponiveis vem
de `lionchat_account_variables_list`.

Variavel escrita errada nao da erro: some do texto. Sempre releia o texto proposto procurando
buracos antes de mostrar ao cliente.
