# Respostas, "Salvar em" e fluxo

## Como a resposta chega

Quando o cliente envia o formulario, entra uma mensagem do CLIENTE na conversa dele, com o texto fixo
"Formulario respondido" e `content_type: form`. Leia por `lionchat_conversations_messages_list`:

- `content_attributes.items`: `[{name: 'f_1', label: 'Seu nome'}]` (a pergunta, como estava na versao publicada que
  o cliente viu);
- `content_attributes.submitted_values`: `[{name: 'f_1', value: 'Maria'}]`;
- `content_attributes.whatsapp_form`: `form_id`, `publication_number`, `flow_token` e `raw` (as respostas cruas).

Os valores ja vem legiveis: opcao pelo nome, escolha multipla separada por virgula, data DD/MM/AAAA, intervalo
"DD/MM/AAAA a DD/MM/AAAA", aceite Sim/Nao, foto/arquivo "N arquivos: nomes". Os arquivos chegam como ANEXOS da mesma
mensagem; arquivo acima do limite de recebimento do WhatsApp Oficial fica de fora (a resposta diz quantos vieram).

Casos especiais:
- Nenhuma resposta preenchida: "Formulario respondido (nenhuma resposta preenchida)".
- Resposta ilegivel: "Formulario recebido (nao foi possivel ler as respostas)" com `whatsapp_form.unreadable = true`.
  Nesse caso nao roda o "Salvar em" nem o gatilho "Formulario do WhatsApp respondido"; a mensagem chega aos fluxos
  como mensagem comum.

A resposta e mensagem do cliente: reabre conversa encerrada, nao e confundida com nota da pesquisa de satisfacao e a
IA de atendimento le as respostas (pergunta e resposta) junto da mensagem.

## "Salvar em" (`save_targets`)

Grava cada resposta num campo, quando o cliente envia. Vai no `lionchat_whatsapp_forms_update`, por codigo de
pergunta:

```json
{ "id": 5, "save_targets": { "f_1": "contact.name", "f_2": "contact.email",
                              "f_3": "contact.cadastral.cpf", "f_4": "custom.cidade_interesse",
                              "f_5": "conversation_custom.servico_desejado" } }
```

Destinos aceitos:
- `contact.name`, `contact.email`
- `contact.cadastral.<cpf|rg|cnpj|passport|profession|marital_status|gender|date_of_birth>`
- `contact.address.<cep|street|number|complement|neighborhood|city|state|country>`
- `custom.<chave>` - atributo personalizado do CONTATO
- `conversation_custom.<chave>` - atributo personalizado da CONVERSA

Regras:
- O atributo precisa existir na conta (`lionchat_custom_attributes_list`) e nao ser protegido do sistema. Destino
  invalido = 422 no `_update`.
- **Telefone nunca** (quem responde ja esta identificado pelo numero do WhatsApp).
- **Foto e arquivo nao tem destino** (viram anexo da mensagem); codigo de pergunta de foto/arquivo = 422.
- O codigo tem de existir no rascunho atual ou na ultima publicacao. Pergunta tirada do rascunho mantem o destino
  ate publicar de novo (a versao no ar ainda pergunta).
- Mudar o "Salvar em" NAO precisa publicar de novo e nao mexe na Meta.
- Ao gravar: dado novo vence o antigo; resposta em branco nao apaga; mesmo valor nao regrava; e-mail que ja e de
  outra ficha da conta e pulado e o resto grava; valor que nao cabe no tipo do atributo nao grava (data, numero,
  sim/nao; em atributo de LISTA a resposta tem de casar com uma opcao - maiuscula/minuscula nao importam, acento
  importa). No historico do atributo aparece "Formulario do WhatsApp - <nome>".
- Prefira sempre `save_targets`. Se algum campo do `editor_data` mandado no mesmo pedido trouxer `save_to` (o jeito
  da tela, por posicao), o `save_targets` daquele pedido e ignorado. Nao misture os dois.

## Fluxo depois da resposta

Ordem garantida, depois que a mensagem e gravada:
1. o "Salvar em" grava;
2. a mensagem chega aos fluxos - um fluxo parado em "Aguardar resposta" (por exemplo, o que mandou o modelo) segue,
   e o gatilho de mensagem recebida dispara;
3. dispara o gatilho "Formulario do WhatsApp respondido".

Assim o fluxo ja le o dado gravado. Cada passo roda uma vez por mensagem, mesmo se o trabalho repetir.
Limite: a AUTOMACAO "Mensagem criada" recebe a mensagem na hora, ANTES do "Salvar em" - para usar o dado gravado,
prefira fluxo.

### Gatilho `whatsapp_form_answered`

No bloco Inicio do fluxo (skill `criar-fluxos`):

```json
{ "key": "whatsapp_form_answered", "config": { "whatsapp_form_id": "5" } }
```

- `whatsapp_form_id` = o `id` NOSSO do formulario, como TEXTO (nao o `meta_flow_id`). Vazio = qualquer Formulario do
  WhatsApp (inclusive um feito direto no painel da Meta). Lista tambem e aceita.
- Roda na conversa da resposta, respeitando a caixa do fluxo como qualquer gatilho de conversa.
- Dois fluxos ativos com esse gatilho para o mesmo formulario (ou "qualquer" num deles) podem ser recusados por
  conflito de gatilho (`flow_trigger_conflict`).

Variaveis do gatilho:
- `{{trigger.form.name}}`, `{{trigger.form.id}}`;
- `{{trigger.form.f_<n>}}` - a resposta de cada pergunta, ja em texto. Pegue os codigos com
  `lionchat_whatsapp_forms_list` com `with_questions=true` (`questions: [{code, label}]`).

Receitas:
- **Cadastro que vira card:** gatilho do formulario -> acao criar card no funil com
  `{{trigger.form.f_1}}` no titulo -> mensagem de agradecimento.
- **Triagem:** gatilho -> condicao sobre `{{trigger.form.f_3}}` (a opcao escolhida) -> transferir para o time certo.
- **Agendamento pedido:** gatilho -> nota interna para a equipe com o resumo das respostas.
