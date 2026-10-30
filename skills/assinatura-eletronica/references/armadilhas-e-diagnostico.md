# Armadilhas e diagnostico

Onde olhar: `lionchat_signature_envelopes_show` (linha do tempo e `delivery.failed[]`),
`lionchat_signature_envelopes_proofs` (documentos), e no fluxo `lionchat_flows_executions_list` +
`lionchat_flow_sessions_show` (o item do contrato traz o motivo em texto).

## Codigos de erro

| Codigo | Causa | O que fazer |
|---|---|---|
| `403 feature_disabled` / `funcionalidade_desligada` | Recurso de assinatura desligado na conta | Pedir para ligar (nao e pelo conector) |
| recusa de permissao | Usuario sem `signature_manage` (ou nao administrador para cancelar/apagar) | Usar um administrador ou dar o cargo com "Gerenciar Assinaturas" |
| `variaveis_sem_valor` (com `missing` e `missing_labels`) | Campo do modelo sem valor na ficha (ou dado que nao se preenche no envio, como telefone ou dado da conta) | `fill_fields`, completar a ficha ou mandar em `contact_data`; ou usar `collection: form` |
| `dados_invalidos` (com `field_errors`) | Valor de `contact_data` recusado (`invalid_cpf`, `invalid_option`, `email_em_uso`, `campo_desconhecido`...) | Corrigir o campo indicado; nada foi gravado |
| `contact_data_invalido` | `contact_data` fora do formato (nao objeto de textos, mais de 50 chaves, valor acima de 4 KB) | Mandar `{variavel: "valor"}` |
| `coleta_invalida` | `collection` diferente de `ready`/`form` | Corrigir |
| `mensagem_longa` | `message` acima de 1.000 caracteres | Encurtar |
| `sem_contato` | `form`, `contact_data` ou `fill_fields` sem `contact_id` (ou contato de outra conta) | Mandar o `contact_id` |
| `formulario_vazio` | `form` com modelo que nao pergunta nada (ex.: PDF pronto) | Usar `ready` |
| `sem_participantes` / `sem_principal` | Ninguem com nome e destino, ou nenhum `signer` | Conferir `participants` (telefone para WhatsApp, e-mail para e-mail) |
| `cpf_invalido` | CPF do participante invalido | Corrigir ou omitir |
| `escolher_caixa` (com `inboxes[]`) | A pessoa conversa em mais de uma caixa de WhatsApp | Perguntar ao usuario e repetir com `inbox_id` |
| `caixa_invalida` / `sem_caixa_whatsapp` | `inbox_id` que nao e WhatsApp da conta / conta sem caixa de WhatsApp | Escolher uma caixa valida, ou mandar por e-mail |
| `telefone_invalido` / `conversa_nao_criada` | Telefone que nao virou contato / conversa nao criada | Corrigir o telefone e tentar de novo |
| `limite_do_mes` (detalhe `usado/teto`) | Teto de contratos do mes atingido | Esperar o mes ou pedir mais (fora do conector) |
| `anexos_demais` | Documentos com "anexar ao contrato" x pessoas passam de 10 arquivos | Tirar pessoas ou desmarcar "anexar" no modelo |
| `campos_sem_escolha` (com `missing`) | Modelo de Word com campo sem variavel escolhida | Ajustar o mapeamento pela tela |
| `conversor_indisponivel` / `fila_cheia` | Servico que transforma o Word em PDF fora do ar / ocupado (uma conversao por vez na instalacao) | Repetir em instantes; o modelo esta bom. Nao afeta modelo de texto nem PDF pronto |
| `entrega_falhou` | Ninguem recebeu; o contrato ficou rascunho | Ler `detail` (motivos por pessoa), corrigir e `resend` |
| `ja_concluido` | Reenviar ou cancelar contrato ja assinado, recusado, expirado ou cancelado | Nada a fazer; mandar um contrato novo se for o caso |
| `nao_e_sua` / `ja_assinado` / `sem_desenho` / `jeito_nao_permitido` | `sign_as_sender` de quem nao e o remetente, ja assinado, sem imagem, ou jeito que o modelo nao permite | Conferir quem esta conectado e o `signature_methods` do modelo |
| `fora_do_momento` / `sem_documentos` / `motivo_obrigatorio` / `documentos_mudaram` | Aprovar/reprovar fora da espera, sem documento, sem motivo, ou a pessoa mandou outro depois do que foi visto | Reler `_proofs` e repetir com o `last_proof_event_id` novo |
| `modelo_nao_encontrado` (fluxo, macro, automacao) | Modelo apagado ou ARQUIVADO | Desarquivar ou trocar o modelo no bloco |
| `conversa_sem_contato` / `contato_sem_destino` (fluxo) | Conversa sem contato / ficha sem telefone e sem e-mail | Completar a ficha |
| `erro_inesperado` | Falha interna | Tentar de novo; persistindo, suporte |

Motivos por pessoa em `delivery.failed[]` / `failures[]`: `sem_destino`, `email_invalido`, `sem_conversa`,
`window_expired` (caixa Oficial fora da janela de 24 h), `envio_falhou` (e-mail nao saiu depois das novas
tentativas), `email_recusado`.

## Armadilhas

- **Enviar e real.** Testar com o telefone de um cliente do cliente manda o contrato a ele. Use contato de
  teste com o telefone do proprio usuario, e so com autorizacao.
- **Cancelar nao devolve a vaga** e um contrato de teste tambem conta no mes.
- **Caixa Oficial com a janela de 24 h fechada:** o texto do link pode ser recusado depois pela Meta
  (`window_expired` em `delivery.failed`). Configure os modelos aprovados em `communication`
  (`invite_template`, `code_template`...) ou use caixa de QR Code. Para o codigo, a pessoa tambem pode
  confirmar mandando a frase pelo WhatsApp.
- **Modelo do WhatsApp em analise** pode ser salvo no modelo do contrato, mas so vale quando a Meta aprovar;
  ate la sai o texto padrao, e o link para divulgar fica indisponivel na caixa Oficial.
- **"RG ou CNH"** e `id_document` com `accepted_types: ["rg","cnh"]`. Com `rg` e `cnh` soltos a pessoa
  precisa mandar os DOIS.
- **Documento pedido = o contrato nao fecha sozinho**: fica esperando a aprovacao da equipe. Avise o
  cliente, senao ele acha que travou. A condicao `all_signed` do fluxo e falsa nessa espera.
- **Chave do documento criado (`custom_...`) muda = outro documento.** Mantenha a mesma em cada gravacao.
- **Mudar o modelo nao muda contrato ja enviado.** Para o cliente receber a versao nova, cancelar e enviar
  de novo (gasta outra vaga) - confirme antes.
- **Modelo arquivado some do fluxo, da macro e da automacao.**
- **Fluxo so manda ao titular.** Contrato com testemunha ou dois titulares e envio na mao.
- **Formato do `roles_layout`:** `signers`, `witnesses` e `validity_days` sao numeros puros; o resto e
  objeto ou lista conforme a tabela. Formato errado = recusado.
- **`regenerate: true` no link para divulgar quebra o endereco antigo** (quem ja divulgou perde o link).
- **Lembrete:** nao existe campo de lembrete no modelo. E fluxo (`no-fluxo.md`).
- **Arquivos (PDFs e documentos de identidade) so pela tela.** Nao prometa enviar, baixar ou mostrar.
- **Nao e a assinatura do plano.** Fatura, cartao e cobranca do LionChat ficam fora do conector.

## Sintomas sem codigo de erro

- **"O cliente diz que nao recebeu":** `_show` -> tem `delivered` para ele? Ha item em `delivery.failed`?
  Telefone certo no participante? Caixa Oficial fora da janela? Depois, `resend` (com confirmacao).
- **"Todos assinaram e nao fechou":** `status` `awaiting_approval` = falta aprovar os documentos
  (`_proofs`). `signed_pending_seal` = gerando o PDF final (espere ate ~42 min); com `seal_blocked` em
  `_proofs`, reprove o anexo daquela pessoa.
- **"A testemunha nao recebeu lembrete":** o lembrete precisa do gatilho `signature_delivered` (por pessoa).
  Com `signature_sent` ele cai so na conversa do contrato. Quem recebe so por e-mail nao dispara fluxo.
- **"O lembrete cobrou contrato cancelado":** a condicao tem so `person_not_signed`; falta a segunda regra
  `pending` na mesma saida (`logic: 'and'`).
- **"O fluxo nao dispara com o contrato que ele mesmo mandou":** de proposito para `signature_sent` e
  `signature_delivered` (evita laco). Os outros gatilhos valem.
- **"Saiu com dado errado":** o contrato le a ficha no momento do envio. Corrigir a ficha e enviar outro
  contrato; o ja enviado nao muda.
- **"A conferencia por IA nao fez nada":** a conta precisa da chave da OpenAI; PDF com mais de 5 paginas
  nao e conferido; falha da IA aceita o arquivo.
