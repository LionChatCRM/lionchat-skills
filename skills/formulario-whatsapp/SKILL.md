---
name: formulario-whatsapp
description: Monta, publica, liga a um modelo de mensagem e corrige o Formulario do WhatsApp do LionChat pelo conector - o formulario NATIVO da Meta (WhatsApp Flows) que o cliente preenche sem sair do WhatsApp, aberto por um botao de modelo, so em caixa do WhatsApp Oficial. Cobre telas e campos, "Salvar em" (gravar a resposta na ficha), imagens, previa oficial, publicar, aposentar, o botao FLOW no modelo e o gatilho de fluxo "Formulario do WhatsApp respondido". Use quando o usuario disser "quero um formulario dentro do WhatsApp", "o cliente preenche o cadastro pelo WhatsApp", "botao que abre um formulario", "pesquisa dentro do WhatsApp", "o formulario nao publica", "a resposta do formulario nao gravou na ficha", "o fluxo nao continuou depois do formulario", ou pedir WhatsApp Flows. NAO e o Formulario publico de captacao (pagina com link). Sempre pergunta antes e so cria ou publica depois de confirmacao explicita.
---

# Formulario do WhatsApp do LionChat

O Formulario do WhatsApp e o formulario nativo da Meta (WhatsApp Flows). O cliente toca num botao de um modelo de
mensagem, preenche as telas DENTRO do WhatsApp e as respostas voltam para a conversa como uma mensagem
"Formulario respondido", pergunta por pergunta. Ele e o segundo tipo do menu Formularios.

Sao TRES pecas que precisam existir juntas, e esquecer uma e o erro mais comum:

| Peca | Ferramentas | O que faz |
|---|---|---|
| **Formulario** | `lionchat_whatsapp_forms_*` | As telas e perguntas. Nasce rascunho; so existe na Meta depois de publicar. |
| **Modelo com botao FLOW** | `lionchat_inboxes_whatsapp_templates_create` | O botao que abre o formulario. Sem modelo aprovado, nenhum cliente consegue abrir. |
| **O que acontece com a resposta** | `save_targets` do formulario e/ou fluxo com gatilho `whatsapp_form_answered` | Grava na ficha, segue o atendimento. Sem isso a resposta so fica na conversa. |

Regras que nunca mudam:

- **NAO confundir** com o Formulario publico de captacao (`lionchat_lead_forms_*`, pagina com link proprio) nem com o
  formulario de anuncio da Meta (`lionchat_meta_lead_*`). Se o cliente quer um link para por no Instagram ou no site,
  e o Formulario publico, nao este.
- **So caixa do WhatsApp Oficial (API da Meta).** Caixa de QR Code nao tem formulario nem modelo.
- **Publicar e para sempre na conta do WhatsApp do cliente.** Depois de publicado so da para editar e publicar de
  novo, ou aposentar (irreversivel). Nunca publique para "testar" sem o cliente autorizar.
- **Montar, publicar, previa, imagem, aposentar e excluir = so administrador da conta.** Listar e ver = qualquer
  pessoa. Recusa por permissao: pare e avise.
- Se `lionchat_whatsapp_forms_list` nao estiver entre as suas ferramentas, PESQUISE pelo nome antes de dizer que
  nao existe.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 - Conferir os pre-requisitos

Use `lionchat_inboxes_list`, `lionchat_whatsapp_forms_list` e `lionchat_custom_attributes_list`.

1. **Uma caixa do WhatsApp Oficial na conta.** Criar com caixa que nao e oficial devolve 422
   `inbox_must_be_official`. Se a conta so tem QR Code, PARE: este recurso nao existe para ela.
2. **Formularios que ja existem** (`_list`, com `inbox_id` para filtrar pela conta do WhatsApp daquela caixa). O nome
   precisa ser UNICO na conta do WhatsApp - a Meta recusa nome repetido ao publicar. Se ja existe um parecido,
   ajuste-o em vez de criar outro.
3. **Os atributos para o "Salvar em".** Atributo personalizado precisa existir na conta antes (`custom.<chave>` do
   contato ou `conversation_custom.<chave>` da conversa). Se faltar, crie primeiro (skill `organizar-contatos`), com
   OK do cliente.
4. **Quem esta conectado e administrador?** Sem isso, so da para ler.

### Etapa 2 - Entender (1 ou 2 perguntas por vez)

1. Para que e o formulario? (cadastro, pesquisa de satisfacao, pedido de orcamento, agendamento, envio de
   documento) - isso define a `category`.
2. Quais perguntas, em que ordem, e quais sao obrigatorias? Pergunta de texto aceita so 20 caracteres no rotulo:
   combine com o cliente um rotulo curto e explique o resto no texto de ajuda (80).
3. Precisa receber FOTO ou ARQUIVO do cliente? (uma por tela, ate 10 arquivos no formulario inteiro)
4. Onde cada resposta deve ficar gravada? (nome, e-mail, CPF, endereco, atributo do contato ou da conversa)
5. Como o formulario chega ao cliente? (modelo enviado na conversa, numa campanha, num fluxo ou por automacao)
6. O que deve acontecer depois que o cliente responde? (so ficar na conversa, seguir um fluxo, avisar a equipe)

### Etapa 3 - Propor e confirmar

Mostre ao cliente, ANTES de gravar:

```
FORMULARIO "[nome]" - caixa [nome da caixa oficial] - categoria [category]
  Tela 1 "[titulo, ate 30]"
    - [tipo] "[pergunta]" (obrigatoria?) -> Salvar em: [destino ou "so na conversa"]
  Tela 2 ...
  Botoes: "[next_label]" (telas do meio) / "[submit_label]" (ultima tela, envia)

MODELO "[nome do modelo]" - categoria [UTILITY/MARKETING] - botao "[texto, ate 25]" que abre o formulario
DEPOIS DA RESPOSTA: [fluxo com gatilho "Formulario do WhatsApp respondido" que faz X / nada]
```

Avise claramente: publicar leva o formulario para a conta do WhatsApp dele PARA SEMPRE, e o modelo vai para analise
da Meta (que pode trocar a categoria para Marketing). So grave depois de um "sim" explicito.

### Etapa 4 - Montar

1. `lionchat_whatsapp_forms_create` com `name`, `inbox_id` e o `editor_data` (formato em
   `references/estrutura-e-campos.md`). Nada vai para a Meta ainda.
2. Imagens, se houver: `lionchat_whatsapp_forms_images` com o LINK da imagem -> use o `blob_id` no campo `image`.
3. Ajustes: SEMPRE `lionchat_whatsapp_forms_show` antes e `lionchat_whatsapp_forms_update` com o `editor_data`
   COMPLETO (ele substitui o rascunho inteiro). Mantenha os `code` que o servidor deu.
4. "Salvar em": `_update` com `save_targets` por codigo de pergunta (`{"f_1": "contact.name"}`). Vale na hora,
   sem publicar. Ver `references/respostas-e-fluxo.md`.
5. **Peca o OK de novo** e publique: `lionchat_whatsapp_forms_publish`. Se voltar 422, corrija pelos `problems`
   (`references/armadilhas-e-diagnostico.md`) e publique de novo.
6. Botao no modelo: `lionchat_inboxes_whatsapp_templates_create` com o botao `FLOW` e o `meta_flow_id`. Depois
   `lionchat_inboxes_sync_templates`. Ver `references/publicar-e-ligar-ao-modelo.md` e a skill
   `campanhas-e-modelos`.
7. Depois da resposta, se pedido: fluxo com gatilho `whatsapp_form_answered` (skill `criar-fluxos`).

### Etapa 5 - Testar (sempre antes de mandar para clientes)

1. `lionchat_whatsapp_forms_preview` -> mande o `preview_url` ao cliente (abre sem login, vale 30 dias). E a previa
   OFICIAL da Meta, so existe depois da primeira publicacao.
2. Confira o status do modelo (`lionchat_inboxes_whatsapp_templates_list`) ate ficar aprovado.
3. Com autorizacao, envie o modelo para um numero de teste DO CLIENTE (nunca para cliente final), preencha e confira
   na conversa: a mensagem "Formulario respondido" com as respostas
   (`lionchat_conversations_messages_list`), a ficha do contato (`lionchat_contacts_show`) com o "Salvar em" gravado,
   e o fluxo, se houver (`lionchat_flows_executions_list`).

### Etapa 6 - Relatar

Diga o que foi criado (formulario com `id` e `meta_flow_id`, modelo, fluxo), o status de cada um, o que foi
testado e o que falta (aprovacao do modelo, teste real). Nunca diga que "esta funcionando" sem uma resposta de
verdade chegando na conversa.

## Regras que nao podem ser violadas

1. **NUNCA publique, aposente ou exclua sem confirmacao explicita.** Publicar e permanente na conta do WhatsApp;
   aposentar (`_deprecate`) e IRREVERSIVEL e derruba todo modelo que abre o formulario. O conector nao pede
   confirmacao sozinho para essas ferramentas - quem pergunta e voce.
2. **NUNCA mande `editor_data` parcial.** Ele substitui o rascunho inteiro: leia com `_show`, altere e devolva tudo.
3. **NUNCA invente `code` em campo novo** - omita; o servidor da `f_<n>`. Codigo ja publicado nunca volta a ser usado.
4. **NUNCA use o `id` nosso no botao do modelo.** O botao leva o `meta_flow_id`; o gatilho do fluxo leva o `id` nosso
   (como texto).
5. **NUNCA aponte "Salvar em" para telefone, nem para pergunta de foto/arquivo** (o servidor recusa).
6. **SEMPRE em portugues do Brasil**, sem emoji, usando os nomes da tela: Formularios, Formulario do WhatsApp,
   Publicar, Previa oficial, Aposentar, Salvar em, Modelos de Mensagens.

## Referencias

- `references/estrutura-e-campos.md` - o `editor_data`: telas, tipos de campo, ajustes e todos os limites da Meta.
- `references/publicar-e-ligar-ao-modelo.md` - rascunho x publicado x aposentado, previa, imagens, publicar de novo,
  botao FLOW no modelo e como enviar.
- `references/respostas-e-fluxo.md` - como a resposta chega, "Salvar em", gatilho de fluxo e variaveis, IA.
- `references/armadilhas-e-diagnostico.md` - cada erro de salvar/publicar, sintomas sem erro e o que fazer.
