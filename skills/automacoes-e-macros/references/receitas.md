# Receitas prontas

Indice:

1. Triagem de primeiro contato
2. Etiquetar por palavra-chave
3. Lead novo vira card no funil
4. Aviso de urgencia para a equipe
5. Ligar o AI Agente numa caixa
6. Fechamento: conversa resolvida vira Ganho no funil
7. Quando puserem a etiqueta X
8. Venda aprovada chegou de fora (gatilho Webhook)
9. Card chegou na etapa final
10. Macro de encerramento
11. Macro de "assumir e priorizar"

Antes de usar qualquer receita: **levante os numeros e nomes reais da conta** (etiquetas, equipes,
caixas, funis, etapas, respostas prontas, AI Agente) — os valores abaixo sao ilustrativos. E lembre
que a ultima condicao vai sempre com o conector vazio.

---

## 1. Triagem de primeiro contato

**Quando usar:** o cliente quer que toda conversa nova ja caia com a equipe certa e marcada.

**Perguntar antes:** vale para todas as caixas ou so uma? Qual equipe? Alguma etiqueta?

```
Nome:    Triagem de primeiro contato
Gatilho: conversation_created
Condicoes:
  [{"attribute_key":"inbox_id","filter_operator":"equal_to",
    "values":[12],"query_operator":null}]
Acoes:
  [{"action_name":"assign_team","action_params":[3]},
   {"action_name":"add_label","action_params":["novo-contato"]}]
```

**Conferir:** a etiqueta existe com esse nome exato? A equipe existe?

---

## 2. Etiquetar por palavra-chave

**Quando usar:** "quando o cliente falar em orcamento, marca a conversa".

**Perguntar antes:** quais palavras? Vale so para o que o cliente escreve (quase sempre sim)?

```
Nome:    Interesse em orcamento
Gatilho: message_created
Condicoes:
  [{"attribute_key":"message_type","filter_operator":"equal_to",
    "values":["incoming"],"query_operator":"AND"},
   {"attribute_key":"content","filter_operator":"contains",
    "values":["orcamento"],"query_operator":null}]
Acoes:
  [{"action_name":"add_label","action_params":["interesse-orcamento"]}]
```

**Cuidados:**
- Sem a condicao de Tipo da Mensagem, a regra tambem dispara no que os atendentes escrevem.
- "Contem" nao quebra por virgula: `orcamento, preco` num valor so seria procurado como uma frase
  unica. Para varias palavras, **uma condicao por palavra ligadas por OU** — e nao varios valores na
  mesma condicao: eles funcionam no motor, mas viram uma frase unica se o cliente reabrir a regra na
  tela e salvar.
- Cada palavra e procurada como texto, sem acento nao vira com acento: pergunte ao cliente as
  variacoes que ele quer pegar.

---

## 3. Lead novo vira card no funil

**Quando usar:** "todo contato novo tem que aparecer no meu funil".

**Perguntar antes:** qual funil, qual etapa, o card ja nasce com oferta ou checklist? **E avise:**
o card nasce com responsavel — o da conversa ou, se nao houver, um agente sorteado por rodizio do
funil; e nesse caso o sorteado tambem vira responsavel da CONVERSA.

```
Nome:    Lead novo entra no funil
Gatilho: conversation_created
Condicoes:
  [{"attribute_key":"any_conversation","filter_operator":"equal_to",
    "values":["true"],"query_operator":null}]
Acoes:
  [{"action_name":"create_kanban_item",
    "action_params":[{"funnel_id":4,"funnel_stage":"novo_lead",
                      "allow_duplicates":false}]}]
```

**Conferir:** a chave da etapa (`funnel_stage`) vem da leitura do funil, nao do nome que aparece na
tela.

---

## 4. Aviso de urgencia para a equipe

**Quando usar:** "se alguem falar em cancelar, quero saber na hora".

```
Nome:    Alerta de cancelamento
Gatilho: message_created
Condicoes:
  [{"attribute_key":"message_type","filter_operator":"equal_to",
    "values":["incoming"],"query_operator":"AND"},
   {"attribute_key":"content","filter_operator":"contains",
    "values":["cancelar"],"query_operator":null}]
Acoes:
  [{"action_name":"change_priority","action_params":["urgent"]},
   {"action_name":"add_label","action_params":["risco-cancelamento"]},
   {"action_name":"send_email_to_team",
    "action_params":[{"team_ids":[3],
                      "message":"Cliente falou em cancelar. Olhar agora."}]},
   {"action_name":"add_private_note",
    "action_params":["Sinal de cancelamento detectado automaticamente."]}]
```

---

## 5. Ligar o AI Agente numa caixa

**Quando usar:** "quero que a IA atenda o primeiro contato dessa caixa".

**Perguntar antes:** ela fala na hora ou so assume e espera o cliente?

```
Nome:    IA atende o primeiro contato
Gatilho: conversation_created
Condicoes:
  [{"attribute_key":"inbox_id","filter_operator":"equal_to",
    "values":[12],"query_operator":null}]
Acoes:
  [{"action_name":"assign_captain_assistant",
    "action_params":[{"assistant_id":17,"proactive":true}]}]
```

`proactive: false` faz o AI Agente assumir a conversa e esperar o cliente falar.
`["nil"]` no lugar do objeto desliga o AI Agente e marca que foi desligado de proposito.

**Nao tente** condicionar isso a horario de expediente: nao existe condicao de horario em automacao.
Para "so fora do expediente", o caminho e Flows ou a mensagem de ausencia da caixa.

---

## 6. Fechamento: conversa resolvida vira Ganho no funil

```
Nome:    Encerrou, marcar Ganho
Gatilho: conversation_resolved
Condicoes:
  [{"attribute_key":"labels","filter_operator":"equal_to",
    "values":["venda-fechada"],"query_operator":null}]
Acoes:
  [{"action_name":"set_kanban_item_status",
    "action_params":[{"funnel_id":4,"status":"won"}]}]
```

**Cuidado:** se nao existir card daquela conversa naquele funil, a acao e pulada em silencio. Se o
fluxo do cliente nao garante que o card ja exista, ponha "Criar Card Kanban" antes, na mesma regra.

---

## 7. Quando puserem a etiqueta X

```
Nome:    Etiqueta VIP vai para o time premium
Gatilho: conversation_updated
Condicoes:
  [{"attribute_key":"labels","filter_operator":"equal_to",
    "values":["vip"],"query_operator":null}]
Acoes:
  [{"action_name":"assign_team","action_params":[5]},
   {"action_name":"change_priority","action_params":["high"]}]
```

**AVISO OBRIGATORIO AO CLIENTE:** criada por fora da tela, essa regra nasce **sem recorte** — ela vai
disparar em qualquer mudanca da conversa (troca de status, de responsavel, de prioridade, ate em
carimbos invisiveis), nao so quando a etiqueta for adicionada. Diga ao cliente que ele precisa abrir
a regra no painel, em Configuracoes > Automacao, e marcar em "Qual acao dispara a regra" apenas
"Etiqueta adicionada".

---

## 8. Venda aprovada chegou de fora (gatilho Webhook)

**Quando usar:** gateway de pagamento, Meta Lead Ads, Webhook Universal, e-Clinica.

**Perguntar antes (obrigatorio):** por qual caixa a mensagem sai? Ela e WhatsApp API Oficial ou QR
Code? Isso muda a acao.

**Duas verdades sobre este gatilho:**
- A Caixa de Envio e obrigatoria.
- **As condicoes nao filtram nada.** Quem separa "compra aprovada" de "carrinho abandonado" e o
  mapeamento de eventos na tela da integracao. Se o cliente quer duas mensagens diferentes, sao duas
  regras e dois mapeamentos.

**Em caixa WhatsApp API Oficial** (texto livre e recusado fora da janela de 24 horas):

```
Nome:    Compra aprovada - boas-vindas
Gatilho: webhook
Caixa de Envio: 12   (definida na criacao; depois so muda no painel)
Condicoes:
  [{"attribute_key":"any_conversation","filter_operator":"equal_to",
    "values":["true"],"query_operator":null}]
Acoes:
  [{"action_name":"send_whatsapp_template",
    "action_params":[{"name":"boas_vindas_compra","id":"123456789",
                      "language":"pt_BR","category":"UTILITY",
                      "processed_params":{"body":{"1":"{{contact.first_name}}"}}}]},
   {"action_name":"add_label","action_params":["compra-aprovada"]}]
```

Confira o modelo antes com `lionchat_inboxes_whatsapp_templates_list`: nome exato, idioma e status
aprovado. **Nunca** escreva `template_name`, `template_id` ou `template_language` — a regra salva e
nao envia nada.

**Em caixa WhatsApp QR Code**, troque a primeira acao por:

```
  {"action_name":"send_message",
   "action_params":["Oi {{contact.first_name}}, sua compra foi aprovada!"]}
```

Ali nao existe modelo aprovado. E resposta pronta com botao tambem nao vale nessas caixas.

---

## 9. Card chegou na etapa final

**Quando usar:** "quando o card entrar em Fechado, avise meu sistema".

```
Nome:    Card fechado avisa o ERP
Gatilho: kanban_item_stage_changed
Condicoes:
  [{"attribute_key":"funnel_id","filter_operator":"equal_to",
    "values":[4],"funnel_stage":"fechado","query_operator":null}]
Acoes:
  [{"action_name":"send_webhook_event","action_params":["https://..."]},
   {"action_name":"add_label","action_params":["cliente-fechado"]}]
```

**Cuidados deste gatilho:**
- So estas 10 acoes rodam: mensagem, etiqueta, responsavel do card, mover card, nota no card,
  cronometro, Ganho/Perdido, prioridade do card e webhook. Qualquer outra e descartada em silencio.
- **Nunca** ponha condicao de conversa (etiqueta, status) aqui: o que esse motor nao entende ele
  **aprova**, e a regra passa a valer para todos os cards de todos os funis.
- "Adicionar Nota ao Card" nesse gatilho precisa do texto puro (`["texto"]`), nao do objeto com
  `funnel_id` e `text`. E "Atribuir Agente ao Card" precisa de `[{"id": 12}]` ou `[12]` — com o
  formato da tela (`agent_id`) esse motor nao acha ninguem e pula em silencio.
- Mensagem e etiqueta so alcancam conversa: se o card tem conversas vinculadas, valem as marcadas
  para receber automacao; se nao tem vinculacao nenhuma, vale a conversa de origem do card.

---

## 10. Macro de encerramento

**Quando usar:** o atendente quer fechar o atendimento com um clique.

**AVISO OBRIGATORIO AO CLIENTE:** a mensagem sai assinada por quem clicou (o cliente ve o nome dele
no WhatsApp) e, se o AI Agente daquela conversa estiver configurado para parar quando um humano
responde, ele **e pausado ou desligado ali**. Se a conta usa AI Agente, confirme se e isso mesmo que
ele quer.

```
Nome:        Encerrar atendimento
Visibilidade: global
Acoes:
  [{"action_name":"send_message",
    "action_params":["Obrigado pelo contato! Qualquer coisa e so chamar."]},
   {"action_name":"add_label","action_params":["atendida"]},
   {"action_name":"resolve_conversation","action_params":[]}]
```

Se o cliente quiser tambem marcar o card como Ganho, some
`{"action_name":"set_kanban_item_status","action_params":[{"funnel_id":4,"status":"won"}]}` — e
lembre que sem card a acao e pulada, mas a tela ainda diz "executada com sucesso".

---

## 11. Macro de "assumir e priorizar"

**Quando usar:** o atendente quer pegar a conversa para si com um clique.

```
Nome:        Assumir esta conversa
Visibilidade: global
Acoes:
  [{"action_name":"assign_agent","action_params":["self"]},
   {"action_name":"change_priority","action_params":["high"]},
   {"action_name":"add_private_note",
    "action_params":["Assumida pelo atendente."]}]
```

`"self"` (atribuir a quem clicou) **so existe na macro** — nao tente usar em automacao.
Esta macro nao manda mensagem para o cliente, entao **nao** mexe no AI Agente.

**Cuidado:** "Atribuir ao Agente" so vale se quem clicou for membro daquela caixa (ou administrador
da conta). Se nao for, a acao e pulada em silencio e a conversa continua sem responsavel.
