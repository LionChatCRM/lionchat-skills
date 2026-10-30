# Armadilhas e diagnostico

## Erros ao SALVAR (`_create` / `_update`)

| Resposta | Causa | O que fazer |
|---|---|---|
| 422 `inbox_must_be_official` (`error.kind: no_channel`) | caixa que nao e do WhatsApp Oficial, ou de outra conta. No `_update`, trocar `inbox_id` para caixa de OUTRA conta do WhatsApp tambem da isso | usar uma caixa oficial da conta (da mesma conta do WhatsApp) |
| 422 `{errors: [...]}` | FORMATO errado: tipo desconhecido, numero fora da faixa ou como texto, data fora de "AAAA-MM-DD", valor de lista desconhecido, `category` fora da lista, telas fora de 1-10, mais de 40 campos ou 5 aceites numa tela, opcoes no formato errado | corrigir o formato e mandar de novo |
| 422 `save_targets: ...` | destino invalido: atributo inexistente ou protegido, codigo que nao e pergunta, pergunta de foto/arquivo, telefone | ver `respostas-e-fluxo.md` |
| 409 `busy` | publicacao em andamento | esperar e tentar de novo |
| 401 | quem esta conectado nao e administrador | pedir a um administrador |

Limite de caracteres da Meta NAO barra o salvar (da para salvar pela metade). Ele so aparece ao publicar.

## Erros ao PUBLICAR

422 `{problems: [{code, screen, field, limit}], error, form}`. `screen` e `field` comecam em 0 (tela 1 = 0).

| code | O que corrigir |
|---|---|
| `no_questions` | o formulario nao tem nenhuma pergunta |
| `empty_label` / `empty_text` | pergunta sem texto / conteudo sem texto |
| `too_long` (`limit`) | texto acima do limite do tipo (pergunta, ajuda, fim do intervalo, link, texto alternativo) |
| `title_too_long` / `button_too_long` | titulo de tela acima de 30 / botao acima de 35 |
| `description_too_long` | descricao de foto/arquivo/calendario acima de 300 |
| `no_options` / `empty_option` / `option_too_long` / `option_description_too_long` | escolha sem opcoes / opcao vazia / acima de 30 / descricao acima de 300 |
| `selection_invalid` | minimo/maximo de marcacoes impossivel |
| `date_range_invalid` | data minima depois da maxima |
| `empty_end_label` / `range_helper_incomplete` / `calendar_title_too_long` | intervalo sem rotulo do fim / ajuda numa ponta so / titulo do calendario acima de 80 |
| `invalid_url` / `invalid_read_more_url` | link ou "Ler mais" sem http/https |
| `no_image` / `image_missing` | imagem sem arquivo / arquivo que nao e deste formulario (suba de novo por `_images`) |
| `too_many_links` / `too_many_images` / `too_many_pickers` | mais de 2 links / 3 imagens / 1 foto-arquivo na tela |
| `too_many_files` | soma de arquivos acima de 10 no formulario |

`error.kind`:

| kind | Causa | O que fazer |
|---|---|---|
| `no_channel` | nenhuma caixa oficial com credencial dessa conta do WhatsApp | conferir a caixa oficial e a conexao com a Meta |
| `busy` | ja ha uma publicacao em andamento | esperar; publicacao parada ha mais de 2 minutos e corrigida por `_show` com `refresh=true` |
| `duplicate_name` | nome ja usado nessa conta do WhatsApp | trocar o `name` e publicar de novo |
| `validation` | a Meta recusou o arquivo; `message` diz onde | corrigir o ponto indicado |
| `meta` / `network` | erro da Meta ou de rede | tentar de novo em alguns minutos |
| `too_large` | arquivo do formulario grande demais (imagens demais) | tirar imagens |
| `unexpected` | erro interno | tentar de novo; persistindo, suporte |

## Erros de IMAGEM (`_images`, 422 `{error}`)

| error | Causa |
|---|---|
| `fetch_failed` | link inacessivel, pagina da web em vez do arquivo da imagem, ou endereco interno |
| `invalid_image` | o arquivo nao e imagem |
| `file_too_large` | acima de 10 MB |
| `too_large` | nao coube em 300 KB mesmo reduzida |

## Outros erros

- `_deprecate` em formulario nunca publicado: 422 `not_published`.
- `_destroy` em formulario publicado: 422 `deprecate_first` (so rascunho ou aposentado se excluem).
- `_preview` com `preview_url: null`: ainda nao foi para a Meta - publique antes.

## Sintomas sem mensagem de erro

- **O cliente nao ve o botao / o botao nao abre:** modelo ainda em analise, recusado, ou botao apontando para o `id`
  nosso em vez do `meta_flow_id`. Confira o modelo e o `flow_id`. Formulario aposentado tambem derruba o botao.
- **Editei e o cliente ainda ve o formulario antigo:** `unpublished_changes: true` - falta publicar de novo.
- **A resposta chegou mas nao gravou na ficha:** sem `save_targets` para aquela pergunta; atributo de lista com
  resposta que nao casa com nenhuma opcao; valor que nao cabe no tipo (data, numero); e-mail que ja e de outra ficha;
  resposta em branco (nao apaga). Confira `_show` -> `save_targets` e o historico do atributo na ficha.
- **O fluxo nao disparou:** gatilho com o `meta_flow_id` em vez do `id` nosso; fluxo inativo; caixa do fluxo diferente
  da caixa da conversa; resposta ilegivel (`whatsapp_form.unreadable`). Confira `lionchat_flows_executions_list`.
- **A automacao "Mensagem criada" leu o atributo vazio:** ela roda antes do "Salvar em". Use fluxo.
- **`version_outdated: true`:** publicar de novo resolve.
- **Formulario sumiu da lista do botao do modelo:** a tela so oferece formulario PUBLICADO da mesma conta do WhatsApp
  da caixa.

## Armadilhas

- **Aposentar e irreversivel** e para todo modelo que abre o formulario. Para trocar perguntas, edite e publique de
  novo - nao aposente.
- **`editor_data` substitui o rascunho inteiro.** Mandar so a tela nova apaga as outras. Sempre `_show` antes.
- **Codigos:** campo novo sem `code`; codigos existentes mantidos. Codigo ja publicado nunca volta, entao apagar e
  recriar uma pergunta muda o codigo dela - e o `save_targets` e as variaveis `{{trigger.form.f_<n>}}` precisam ser
  atualizados.
- **Rotulo de pergunta de texto tem so 20 caracteres** (lista suspensa tambem). Explique no `helper`.
- **Data so fixa.** Nao existe "a partir de hoje".
- **Foto acima de 5 MB nao chega**, mesmo com `max_size_mb` maior.
- **Nome unico na conta do WhatsApp**, inclusive contra formularios feitos fora do LionChat no painel da Meta.
- **Modelo aprovado nao ganha botao por edicao** pelo conector: crie um modelo novo.
- **Nao confundir com Formulario publico** (`lead_forms`): aquele e pagina com link; este so abre dentro do WhatsApp,
  por modelo, em caixa oficial.
