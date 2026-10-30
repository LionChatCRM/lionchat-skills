# Enviar e acompanhar contratos

**Todo envio desta pagina fala com uma pessoa real.** Antes de `create`, `resend`, `approve_proofs` ou
`reject_proofs`, mostre ao usuario: quem recebe, o telefone ou e-mail, a caixa e o texto - e espere o
"pode enviar". Nenhuma dessas ferramentas pede confirmacao sozinha.

## Receita do envio na mao

1. `lionchat_signature_documents_list` -> o modelo.
2. `lionchat_signature_envelopes_limits` -> ha vaga? (`{used, limit, unlimited}`).
3. `lionchat_signature_documents_fill_fields` (`id` do modelo + `contact_id`) -> o que falta na ficha.
4. Mostre a proposta de envio e peca a confirmacao.
5. `lionchat_signature_envelopes_create`.
6. Leia a resposta: 201 com `delivery_failures[]` vazio = todos receberam. Depois acompanhe com `_show`.

## `lionchat_signature_envelopes_create`

| Parametro | O que e |
|---|---|
| `signature_document_id` | Modelo (obrigatorio). |
| `contact_id` | Ficha do titular (recomendado: preenche as variaveis e acha a conversa). |
| `conversation_id` | Conversa (numero que aparece na tela) por onde o link sai, quando ja se sabe. |
| `inbox_id` | Caixa de WhatsApp - obrigatorio quando a resposta anterior foi `422 escolher_caixa` (vem com `inboxes[]`). |
| `participants` | Quem assina (obrigatorio). Cada item: `{name, role, phone, email, delivery_channel, cpf, contact_id}`. |
| `collection` | `ready` (padrao) ou `form`. Outro valor = `422 coleta_invalida`. |
| `message` | Texto que vai junto do link (ate 1.000 caracteres; acima = `422 mensagem_longa`). |
| `contact_data` | So no `ready`: completa a ficha antes de gerar, `{variavel: valor}`. |

**Participantes:**
- `role`: `signer` (titular; pelo menos um, senao `sem_principal`) ou `witness` (testemunha). **Nunca mande
  `sender`**: o remetente nasce sozinho quando o modelo tem "eu tambem assino" (e o usuario do conector).
- `delivery_channel`: `whatsapp` (padrao, exige `phone` no formato +55DDDNUMERO) ou `email` (exige `email`).
  Participante sem nome ou sem destino e DESCARTADO em silencio; se nao sobrar nenhum, `sem_participantes`.
- `cpf` opcional; invalido = `422 cpf_invalido` (vai na linha de identidade da folha de assinaturas).
- Cada pessoa que recebe pelo WhatsApp tem a PROPRIA conversa (a testemunha recebe na conversa dela).
- O contrato e um so para todos os participantes; mandar o modelo para 5 clientes diferentes = 5 chamadas.

**Os dois jeitos de coletar os dados:**
- `ready` (contrato pronto): o PDF sai agora com a ficha. O que faltar pode ir em `contact_data`
  (`{"contact.cpf": "52998224725", "contact.marital_status": "casado"}`): grava na ficha E no contrato, tudo
  ou nada; erro de um campo = `422 dados_invalidos` com `field_errors`. Telefone nao se preenche por ai.
  Formato torto (nao objeto de textos, mais de 50 chaves, valor acima de 4 KB) = `422 contact_data_invalido`.
- `form` (formulario): exige `contact_id`. O link vai SO para o titular, que preenche os dados dele (em
  branco); o que ele digita atualiza a ficha e o contrato nasce com isso. Ate la o contrato fica
  `awaiting_data` e as outras pessoas recebem quando ele for gerado. O link do titular e o
  `participants[].public_url` dele.

**Escolha da caixa (sem `inbox_id`):** primeiro a caixa onde a pessoa ja conversa; depois a caixa onde ela
tem cadastro; por ultimo a unica caixa de WhatsApp da conta. Empate em qualquer degrau = `422
escolher_caixa` com a lista - pergunte ao usuario qual usar e repita com `inbox_id`. Com caixa definida no
modelo (`communication.inbox_id`), tudo sai por ela.

**Respostas:**
- 201: contrato com `participants[]` (o `public_url` de cada um), `events[]` e `delivery_failures[]` (quem
  NAO recebeu; os outros receberam).
- `422 entrega_falhou`: ninguem recebeu. O contrato fica rascunho (`draft`) e volta em `envelope`; corrija
  e use `resend`.
- Outros 422: ver `armadilhas-e-diagnostico.md`.

## O que a pessoa faz

1. Abre o link; no jeito `form`, preenche os dados antes.
2. Se o modelo pede documentos, manda as fotos ou PDFs na pagina, antes de assinar.
3. Confirma que e ela: codigo de 6 digitos enviado pelo mesmo canal do link (WhatsApp ou e-mail). O codigo
   vale 10 minutos, aceita 5 tentativas erradas e ate 3 reenvios; a caixa manda no maximo 50 codigos por dia.
   Quem recebeu pelo WhatsApp tambem pode confirmar mandando uma frase pronta (com `CT-XXXXXX`) para o
   WhatsApp do contrato - resolve a caixa Oficial com a janela de 24 h fechada. A IA, os fluxos e as
   automacoes nao respondem a essa mensagem.
4. Assina desenhando ou digitando o nome (conforme `signature_methods` do modelo). Tambem pode recusar.

## Estados do contrato

| Balde (`status` do `list`) | Estados | Significado |
|---|---|---|
| `aguardando` | `draft`, `sent`, `awaiting_data`, `awaiting_approval` | rascunho (nao entregue) / enviado e ainda nao aberto / esperando a pessoa preencher o formulario / todos assinaram e falta a equipe aprovar os documentos |
| `visualizado` | `viewed`, `partially_signed` | abriu / parte assinou |
| `assinado` | `signed_pending_seal`, `signed` | todos assinaram; gerando o documento final / pronto |
| `encerrado` | `refused`, `cancelled`, `expired` | recusado / cancelado / prazo venceu |
| `falta_eu` | - | contratos abertos em que o usuario do conector e o remetente e ainda nao assinou |

Participante: `pending` -> `viewed` -> `verified` (confirmou o codigo) -> `signed` ou `refused`.

`signed_pending_seal` = o PDF final esta sendo gerado, com novas tentativas automaticas por ate ~42 min.
So depois disso um `seal_failed` quer dizer que nao deu: oriente falar com o suporte.

## Acompanhar

- `lionchat_signature_envelopes_list`: filtros `signature_document_id`, `contact_id`, `conversation_id`,
  `company_id`, `status` (os baldes acima), `q` (nome, telefone ou e-mail de quem assina), `page` (25 por
  pagina). `meta.counts` traz a contagem por balde.
- `lionchat_signature_envelopes_show`: a LINHA DO TEMPO (`events[]`), que e o relatorio de evidencias em
  dados: `created`, `delivered` (link entregue a uma pessoa), `sent`, `opened`, `code_requested`,
  `code_verified` (`method: whatsapp` = confirmou pela frase), `signed` (com `ip`, `device`, `location`,
  `panel`, `signature_method`, `typed_name`), `data_filled`, `proof_sent`, `awaiting_approval`,
  `proofs_approved`, `proofs_rejected`, `all_signed`, `sealed` (com o hash `sealed_sha256`), `refused`,
  `cancelled`, `expired`, `delivery_failed`, `resent`, `copy_sent` / `copy_failed`, `reminded` (de contratos
  antigos). Tudo so acrescenta, nunca apaga.
- `delivery.failed[]` = quem nao recebeu e o motivo (inclui a recusa que a Meta manda DEPOIS, ex.
  `window_expired` na caixa Oficial fora da janela de 24 h; some sozinho se a mensagem acabar entregue).
  E-mail tenta de novo sozinho (1, 5 e 15 min) antes de virar `envio_falhou`; `email_recusado` aparece na
  hora.

## Reenviar, cancelar, assinar como remetente

- `lionchat_signature_envelopes_resend`: reenvia o link a quem ainda nao assinou nem recusou (com o
  formulario ainda nao preenchido, so ao titular). Com o contrato esperando aprovacao, manda de novo SO o
  pedido de documento a quem foi reprovado, com o motivo. Nao cria contrato nem gasta vaga. Resposta
  `{ok, resent, failures[]}` - leia `failures`. Contrato concluido = `422 ja_concluido`. Rascunho vira
  `sent` no primeiro link que sai. **Manda mensagem: confirme antes.**
- `lionchat_signature_envelopes_cancel`: so administrador. O link para de funcionar. Assinado, recusado ou
  expirado nao cancela (`422 ja_concluido`). Nao devolve a vaga. Confirme antes.
- `lionchat_signature_envelopes_sign_as_sender`: SO a assinatura do proprio usuario do conector, quando o
  modelo tem "eu tambem assino". Recebe `signature_image` (PNG em base64 com o prefixo
  `data:image/png;base64,...`, obrigatorio), opcional `location {latitude, longitude, accuracy}`,
  `remember_drawing`, `signature_method` (`drawn` ou `typed`) e `typed_name`. Recusas: `nao_e_sua`,
  `ja_assinado`, `sem_desenho`, `jeito_nao_permitido`. Prefira orientar o botao "Assinar agora" da tela;
  so use a ferramenta a pedido expresso do proprio usuario, e nunca invente o desenho dele.

## Documentos pedidos e a conferencia da equipe

1. `lionchat_signature_envelopes_proofs`: os pedidos (`requests`), por pessoa o que ela mandou (tipo,
   formato, quando, `ai_check` com `veredito` ok/duvida, `ai_kept`), a revisao (`pending` / `approved` /
   `rejected`, motivo, quem, quando) e o `last_proof_event_id`. **Os arquivos nao saem por ferramenta.**
2. Pergunte ao usuario se ele ja olhou os arquivos na tela (botao amarelo "Ver documentos" no contrato ou na
   aba Contratos da ficha). Sem isso, nao aprove.
3. `lionchat_signature_envelopes_approve_proofs` (`participant_id`, `seen_event_id` = o
   `last_proof_event_id` conferido). Com todos aprovados o contrato fecha, o PDF final e gerado e cada pessoa
   recebe a via - **manda mensagem**.
4. `lionchat_signature_envelopes_reject_proofs` (`participant_id`, `reason` obrigatorio, `seen_event_id`): a
   pessoa recebe no canal dela o pedido de documento novo com o motivo escrito - **confirme o texto antes**.
5. 422: `fora_do_momento` (o contrato nao esta esperando aprovacao), `sem_documentos`, `motivo_obrigatorio`,
   `documentos_mudaram` (a pessoa mandou outro depois do que foi visto: leia `_proofs` de novo).
6. `seal_blocked` em `_proofs` = um anexo nao entrou no documento final; o contrato fica parado em
   `signed_pending_seal` e o unico caminho e REPROVAR o documento daquela pessoa, com o motivo.

Quem aprova e reprova: administrador ou `signature_manage`. O remetente recebe aviso no sininho quando os
documentos esperam a equipe.

## A via assinada

Quando o documento final fica pronto, cada pessoa que assinou pelo link (testemunhas incluidas) recebe a sua
via em PDF pelo mesmo canal: anexada numa mensagem na conversa dela (WhatsApp) ou no e-mail (acima de 10 MB
vai sem anexo). Quem assinou no painel nao recebe. Estado de cada via em `delivery.copies[]`. Nao ha
ferramenta para reenviar a via nem para baixar o PDF: oriente baixar na tela do contrato.

## Limite do mes

- `limits` = `{used, limit, unlimited}`. Conta o contrato pela data do PRIMEIRO envio, no mes do fuso da
  conta. Rascunho que nunca foi entregue nao conta ate o primeiro link sair. Cancelado ou expirado continua
  contando.
- O do link para divulgar so conta depois que a pessoa confirma o codigo.
- O teto e ajustado pelo operador da plataforma: nao muda pelo conector. Estourou = `422 limite_do_mes`.
- Antes de enviar varios, compare a quantidade com `limit - used` e avise o usuario.

## Lembrete

Nao existe lembrete automatico do sistema. O lembrete e um fluxo do cliente: ver `no-fluxo.md`.
