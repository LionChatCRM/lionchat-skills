# Estrutura e campos - o `editor_data`

O rascunho do formulario e o `editor_data`. Ele vai no `lionchat_whatsapp_forms_create` (opcional: sem ele nasce uma
tela vazia) e no `lionchat_whatsapp_forms_update`, onde SUBSTITUI o rascunho inteiro. Sempre leia com
`lionchat_whatsapp_forms_show` antes de editar e devolva tudo, inclusive os `code` e o `seq` que vieram.

O `_show` traz tambem `limits` com TODOS os limites da Meta por tipo (fonte unica no servidor). Na duvida sobre um
numero, confira ali em vez de confiar na memoria.

## Formato

```json
{ "category": "SURVEY", "next_label": "Continuar", "submit_label": "Enviar",
  "screens": [
    { "title": "Seus dados",
      "fields": [
        { "type": "heading", "text": "Conte um pouco sobre voce" },
        { "type": "short_text", "label": "Seu nome", "required": true, "max_chars": 60 },
        { "type": "email", "label": "Seu e-mail", "helper": "Para enviarmos a confirmacao" },
        { "type": "single_choice", "label": "Como nos achou?",
          "options": [{ "label": "Instagram" }, { "label": "Indicacao", "description": "Amigo ou parente" }] }
      ] },
    { "title": "Documentos",
      "fields": [
        { "type": "photo", "label": "Foto do documento", "max_files": 2, "source": "camera_gallery" },
        { "type": "opt_in", "label": "Aceito os termos", "required": true,
          "read_more_url": "https://exemplo.com/termos" }
      ] } ] }
```

- `category`: SIGN_UP, SIGN_IN, APPOINTMENT_BOOKING, LEAD_GENERATION, CONTACT_US, CUSTOMER_SUPPORT, SURVEY, OTHER
  (padrao OTHER).
- `next_label` (botao das telas do meio, padrao "Continuar") e `submit_label` (botao da ULTIMA tela, padrao
  "Enviar"): ate 35 caracteres. So a ultima tela envia; as outras levam para a seguinte.
- `title` da tela: ate 30. O cliente ve as telas na ordem da lista; o modelo sempre abre a primeira.
- Campo NOVO vai sem `code`: o servidor da `f_<n>` (pergunta) e `o_<n>` (opcao). Mantenha os codigos existentes ao
  devolver o rascunho: e por eles que o "Salvar em", as variaveis do fluxo e as respostas antigas se ligam.
- `required: true` = obrigatoria (em foto/arquivo vira "pelo menos 1").
- `helper`: texto de ajuda da pergunta, ate 80 - e onde se explica o que nao cabe no rotulo.

## Tipos de pergunta (rotulo maximo entre parenteses)

| type | O que e | Ajustes opcionais |
|---|---|---|
| `short_text` (20) | texto curto | `max_chars` (NUMERO 1-4000) |
| `email` (20) | e-mail | `max_chars` |
| `phone` (20) | telefone | `max_chars` |
| `number` (20) | numero | `max_chars` |
| `long_text` (20) | texto longo | `max_chars` |
| `date` (40) | uma data | `min_date`, `max_date` ("AAAA-MM-DD") |
| `calendar` (40) | calendario: uma data ou intervalo | `mode` single\|range, `end_label`, `end_helper`, `title` (80), `description` (300), `days` (Mon..Sun), `min_date`, `max_date` |
| `single_choice` (30) | escolha unica | `options` (ate 200; acima de 20 vira lista suspensa e o rotulo cai para 20) |
| `multiple_choice` (30) | escolha multipla | `options` (ate 20), `min_selected`, `max_selected` |
| `opt_in` (120) | aceite (caixinha) | `read_more_url` (mostra "Ler mais" que abre o link) |
| `photo` (80) | foto | `description` (300), `max_files` 1-10, `max_size_mb` 1-25, `source` camera_gallery\|camera\|gallery |
| `document` (80) | arquivo | `description` (300), `max_files` 1-10, `max_size_mb` 1-25, `file_kinds` |

- Opcao: `{ "label": "...", "description": "..." }` - rotulo ate 30, descricao ate 300.
- Escolha multipla: `max_selected` nunca acima do numero de opcoes; `min_selected` nunca acima do maximo.
- Data/calendario: so datas FIXAS. Nao existe "a partir de hoje": se o cliente quer isso, avise que a data minima
  precisa ser atualizada e o formulario publicado de novo.
- Calendario em intervalo: `end_label` (rotulo do fim) e obrigatorio; a ajuda (`helper`/`end_helper`) vai nas DUAS
  pontas ou em nenhuma.
- Foto: sai SEMPRE com no maximo 5 MB por arquivo, mesmo com `max_size_mb` maior (e o limite de recebimento do
  WhatsApp Oficial). Arquivo: `file_kinds` entre pdf, word, spreadsheet, presentation, image, text, compressed,
  video.
- Ajuste que nao pertence ao tipo e descartado. `max_chars`, `max_files` etc. tem de ser NUMERO, nao texto.

## Conteudo (nao e pergunta, nao tem resposta)

| type | Campos | Limite |
|---|---|---|
| `heading` / `subheading` | `text` | 80 |
| `paragraph` / `caption` | `text` | 4096 |
| `link` | `text` + `url` (http/https) | texto 25 |
| `image` | `image` = `blob_id` de `lionchat_whatsapp_forms_images`, `alt_text` | texto alternativo 100 |

## Tetos do formulario

- 1 a 10 telas; ate 40 campos e 5 aceites por tela.
- Por tela: 1 foto OU arquivo, 3 imagens, 2 links.
- Formulario inteiro: ate 10 arquivos (soma dos `max_files`).
- Pelo menos UMA pergunta (formulario so de conteudo nao publica).

## Dicas de montagem

- Rotulo curto + ajuda: "Seu CPF" no rotulo, "So os numeros, sem pontos" no `helper`.
- Uma ideia por tela; telas curtas sao mais preenchidas. A primeira tela deve deixar claro para que serve.
- Pergunta que vai para um atributo de lista: use `single_choice` com as MESMAS opcoes do atributo, senao o valor
  nao cabe e nao grava (ver `respostas-e-fluxo.md`).
