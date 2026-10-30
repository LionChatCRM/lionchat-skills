# Publicar e ligar ao modelo

## Ciclo de vida

| status | O que significa | O que da para fazer |
|---|---|---|
| `draft` | Rascunho, so no LionChat (ou rascunho na Meta que nunca foi publicado) | editar, publicar, excluir |
| `publishing` | Publicacao em andamento | esperar; `_update` devolve 409 `busy` e outro `_publish` devolve `busy` |
| `published` | No ar na conta do WhatsApp do cliente | editar o rascunho e publicar de novo; aposentar. NAO da para excluir |
| `deprecated` | Aposentado na Meta, para sempre | so excluir daqui; as respostas antigas continuam nas conversas |

Outros campos do `_list`/`_show`:
- `meta_flow_id`: numero do formulario na Meta. E ele que vai no botao do modelo. Nasce na primeira publicacao.
- `meta_status`: o que a Meta disse por ultimo. `refresh=true` no `_list`/`_show` rele o estado na Meta (corrige
  publicacao que morreu no meio e formulario aposentado pelo painel da Meta).
- `unpublished_changes: true`: o rascunho mudou depois da ultima publicacao - o cliente ainda ve a versao antiga.
- `version_outdated: true` (no `_show`): publicado numa versao de arquivo que a Meta nao aceita mais. Publique de
  novo (nada mais muda).
- `last_error`: o motivo da ultima falha de publicacao.
- `inbox`: a caixa oficial usada para falar com a Meta. O formulario pertence a CONTA DO WHATSAPP, nao a caixa: se a
  caixa for excluida, outra caixa oficial da mesma conta do WhatsApp serve.

## Imagens

`lionchat_whatsapp_forms_images` com `id` do formulario e `url` = endereco http/https do ARQUIVO da imagem (nao a
pagina onde ela aparece). O servidor baixa, aceita so imagem ate 10 MB, reduz para ate 1200 px e 300 KB (JPEG/PNG) e
devolve `{blob_id, url, width, height}`. Use `{ "type": "image", "image": "<blob_id>", "alt_text": "..." }` no
`editor_data`. O formulario precisa existir antes (crie o rascunho primeiro). Erros em
`armadilhas-e-diagnostico.md`. Imagem tirada do rascunho deixa de ser usada; imagem citada que nao e deste
formulario barra o publicar (`image_missing`).

## Publicar

`lionchat_whatsapp_forms_publish` com o `id`. Ele:
1. confere o rascunho (texto acima do limite, pergunta sem rotulo, escolha sem opcao...). Se houver problema, volta
   422 com `problems` ANTES de qualquer chamada a Meta;
2. na primeira vez cria o formulario na Meta; nas seguintes troca o conteudo do MESMO formulario (e o nome, se
   mudou);
3. publica e grava uma foto da versao publicada.

O que saber antes:
- **Fica para sempre** na conta do WhatsApp do cliente. Peca o OK.
- **Nome unico na conta do WhatsApp**: nome repetido = `duplicate_name`.
- **Republicar depois de editar** mantem os modelos que usam o formulario funcionando, sem nova aprovacao.
- Mudanca no rascunho so chega ao cliente depois de publicar. O "Salvar em" vale na hora (nao precisa publicar).

## Previa oficial

`lionchat_whatsapp_forms_preview` devolve `{preview_url, expires_at}`: a previa da propria Meta, abre sem login e
vale 30 dias. `preview_url` nulo = o formulario ainda nao foi para a Meta (publique antes). Use para o cliente
aprovar o visual antes de mandar o modelo.

## Botao no modelo de mensagem

O formulario so chega ao cliente por um botao `FLOW` num modelo aprovado. Use
`lionchat_inboxes_whatsapp_templates_create` (skill `campanhas-e-modelos` para o resto do modelo):

```json
{ "inbox_id": 12, "name": "cadastro_paciente", "language": "pt_BR", "category": "UTILITY",
  "components": [
    { "type": "BODY", "text": "Ola! Para agilizar seu atendimento, preencha o cadastro abaixo." },
    { "type": "BUTTONS", "buttons": [
      { "type": "FLOW", "text": "Preencher cadastro", "flow_id": "1234567890123456", "flow_action": "navigate" } ] } ] }
```

- `flow_id` = `meta_flow_id` do formulario PUBLICADO, como TEXTO. Nunca o `id` nosso.
- No maximo 1 botao FLOW por modelo; texto do botao ate 25 caracteres.
- Use um formulario da MESMA conta do WhatsApp da caixa do modelo: e o que a tela oferece
  (`lionchat_whatsapp_forms_list` com o `inbox_id` da caixa do modelo, so os `published` com `meta_flow_id`).
- A Meta costuma aprovar em minutos e pode trocar a categoria para Marketing (custo por mensagem diferente - avise).
- Depois de criar, rode `lionchat_inboxes_sync_templates` (a criacao devolve so id/status; o conteudo aparece depois
  do sync) e acompanhe o status com `lionchat_inboxes_whatsapp_templates_list`.
- **Modelo ja aprovado nao ganha botao por edicao pelo conector**: `lionchat_inboxes_whatsapp_templates_update` so
  mexe no apontamento das variaveis. Para por o botao, crie um modelo NOVO. Trocar o formulario do botao de um
  modelo aprovado manda o modelo de volta para analise.

## Enviar

Envie o modelo como qualquer outro: na conversa, numa campanha, num bloco de modelo de fluxo ou por automacao. O
servidor monta sozinho a parte que a Meta exige para o botao de formulario (e marca de qual formulario veio) - nao
mande nada a mais para o botao. Por ser modelo, vale tambem fora da janela de 24 horas.
