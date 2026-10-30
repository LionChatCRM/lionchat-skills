# Contrato pelo fluxo, pela macro e pela automacao

Para o formato completo de fluxo (nos, arestas, regras de condicao, salvar e testar), use a skill
`criar-fluxos`. Aqui fica so o que e proprio do contrato. Um fluxo que envia contrato manda mensagem a
cliente REAL assim que for ligado: monte desligado, teste num contato de teste e so ligue com autorizacao.

## A acao "Enviar documento para assinatura" (`send_signature_document`)

Mesma peca no bloco Acoes do fluxo, na macro e na automacao. Ela manda o modelo para a PESSOA DA CONVERSA:
o titular sai da ficha do contato (nome e CPF da ficha; WhatsApp se a ficha tem telefone, senao e-mail) e o
link sai na propria conversa.

| Chave (fluxo) | O que faz |
|---|---|
| `document_id` | Modelo (id de `lionchat_signature_documents_list`). Obrigatorio. |
| `inbox_id` | Caixa de WhatsApp (opcional). |
| `collection` | `ready` (padrao: contrato pronto com a ficha) ou `form` (a pessoa preenche os dados pelo link antes). |
| `message` | Texto junto do link; aceita `{{variaveis}}` do fluxo. |
| `send_link` | `false` = so GERA o contrato, nao manda nada; o link vira `{{contrato.link}}` para um bloco de Mensagem ou de Modelo do WhatsApp logo depois. |

Na macro e na automacao a acao recebe `[numero_do_modelo]` (ou `{document_id, inbox_id}` pela API) e usa o
padrao: contrato pronto, link na conversa. As chaves `collection`, `message` e `send_link` sao so do fluxo.

Limites que o cliente precisa saber:
- **So o titular.** Testemunhas e um segundo titular nao entram por aqui; para isso, envio na mao
  (`signature_envelopes_create`). No fluxo nao existe usuario, entao o "eu tambem assino" nao cria o
  participante do remetente.
- **Variavel sem valor na ficha = o item fica com ERRO** e o motivo aparece no historico do fluxo (ex.:
  `variaveis_sem_valor: CPF`), nunca verde por engano. Com `collection: form`, o que o formulario pergunta
  nao barra.
- **Modelo arquivado nao e achado** (`modelo_nao_encontrado`).
- Cada contrato gerado gasta 1 vaga do mes, mesmo com `send_link: false`.
- Se o fluxo e retomado no mesmo bloco, o contrato ja criado e reaproveitado (nao cria um segundo).
- Depois do bloco ficam `{{contrato.*}}` (as mesmas variaveis dos gatilhos) + `{{contrato.link}}` (link do
  titular), e a condicao "Situacao do contrato" passa a falar DESTE contrato.
- Exige o recurso de assinatura ligado na conta.

No **Formulario publico** existe o bloco "Gerar contrato" (`generate_contract`: `document_id`, `send_link`),
que gera o contrato pelas respostas mapeadas para a ficha; com `send_link: false` o link vai por
`{{contrato.<id_do_bloco>.link}}` num bloco de mensagem. Detalhes no guia de formularios publicos.

## Os 12 gatilhos de contrato (Inicio do fluxo)

| Chave | Quando dispara |
|---|---|
| `signature_sent` | O contrato foi enviado (cai na conversa do CONTRATO, uma vez) |
| `signature_delivered` | O link chegou a UMA pessoa - uma vez por participante, na conversa DELA; tambem no Reenviar |
| `signature_viewed` | A pessoa abriu o link pela 1a vez |
| `signature_signed_signer` | O titular assinou |
| `signature_signed_witness` | Uma testemunha assinou |
| `signature_signed_other` | O remetente ou outro papel assinou |
| `signature_all_signed` | Todos assinaram e o documento final foi gerado |
| `signature_refused` | Alguem recusou |
| `signature_expired` | O prazo terminou sem todas as assinaturas |
| `signature_awaiting_approval` | Os documentos pedidos esperam a conferencia da equipe |
| `signature_proofs_approved` | A equipe aprovou os documentos de uma pessoa |
| `signature_proofs_rejected` | A equipe reprovou os documentos de uma pessoa |

Item: `{key: 'signature_signed_signer', config: {document_ids: []}}` - ids de MODELO como TEXTO; vazio =
qualquer modelo.

Regras:
- So fluxo INDIVIDUAL, ativo, ligado a caixa da conversa (ou a nenhuma caixa).
- A sessao nasce na conversa da PESSOA do acontecimento. Quem recebe so por e-mail (sem conversa) NAO
  dispara nada.
- Sessao ativa do mesmo fluxo na conversa nao e atropelada (o disparo e pulado). Repeticao em 24 h nao
  duplica.
- O fluxo que CRIOU o contrato nao e disparado pelos proprios `signature_sent` e `signature_delivered`
  (evita contrato em laco). Os gatilhos da pessoa (abriu, assinou, recusou) valem no mesmo fluxo.
- Na reprovacao, a mensagem "mande de novo" ja sai sozinha pela secao Comunicacao do modelo: se o fluxo
  tambem escrever, a pessoa recebe as duas.

## Variaveis `{{contrato.*}}`

`contrato.id`, `.evento` (sent, opened, signed, sealed, refused, expired, awaiting_approval,
proofs_approved, proofs_rejected), `.titulo`, `.modelo`, `.modelo_id`, `.status`, `.enviado_em`, `.prazo`,
`.validade_dias`, `.assinado_por_todos` (sim/nao), `.conversa_id`, `.documentos_situacao` (sem_documentos,
pendentes, esperando_aprovacao, aprovados, reprovados), `.pessoas_com_documento_reprovado`, `.participantes`
(lista de {nome, papel, status, assinou_em}).

Quem agiu: `contrato.assinante.nome`, `.papel` (Assinante/Testemunha/Remetente), `.papel_chave`
(signer/witness/sender), `.email`, `.telefone`, `.cpf`, `.status`, `.assinou_em`, `.abriu_em`, `.link`,
`.ip`, `.dispositivo`, `.localizacao`, `.foto_documento`, `.pelo_painel`, `.documentos_pedidos`,
`.documentos_enviados`, `.documentos_situacao` (sem_documentos, faltando, enviados, esperando_aprovacao,
aprovados, reprovados), `.motivo_da_reprovacao`, `.documento_reenviado`, `.aviso_da_ia`.

Sao uma FOTO do momento do disparo. Para saber a situacao atual depois de uma espera, use a condicao abaixo.

## Condicao "Situacao do contrato"

Regra: `field: '_signature_status'`, `operator: 'signature_status'`, `valueType: 'signature_status'`, e
`value` um destes: `person_signed`, `person_not_signed`, `all_signed`, `pending`, `refused`, `expired`,
`cancelled`.

- Ela RELE o contrato no banco na hora: um fluxo que espera 2 dias responde certo mesmo se a pessoa assinou
  no meio.
- Fala do contrato que comecou o fluxo, ou do contrato criado pelo bloco do proprio fluxo. Sem contrato,
  responde FALSO.
- `pending` inclui o formulario ainda nao preenchido. `all_signed` e falso enquanto os documentos esperam a
  aprovacao.
- Recusa da PESSOA ja conta como `refused`, antes de o contrato inteiro mudar.

## Receita: lembrete para quem nao assinou

1. Inicio: gatilho `signature_delivered` (um disparo por pessoa, na conversa dela), com os modelos desejados.
2. Bloco Espera: `waitDuration` + `waitUnit: days` (ex.: 2 dias).
3. Bloco Condicao com UMA saida, `logic: 'and'`, e DUAS regras: `person_not_signed` E `pending`.
   Sem a segunda, contrato cancelado, vencido ou recusado durante a espera ainda cobraria.
4. Na saida verdadeira: a mensagem de lembrete com `{{contrato.assinante.link}}` (o link da pessoa que
   recebeu; `{{contrato.link}}` so existe depois do bloco que cria o contrato).
5. Caixa Oficial: esperar 24 h ou mais fecha a janela; o lembrete precisa ser um modelo aprovado do
   WhatsApp, nao texto livre.
6. Opcional: outra espera + condicao para um segundo lembrete, sempre antes do prazo (`{{contrato.prazo}}`).

Mostre o desenho ao cliente e so salve e ligue com confirmacao: cada lembrete e mensagem real.

## Outras receitas comuns

- **Mover o card quando assinar:** gatilho `signature_all_signed` + acao de mover o card para a etapa
  combinada (skill `funil-de-vendas` / `criar-fluxos`).
- **Avisar a equipe que ha documento para conferir:** gatilho `signature_awaiting_approval` + nota interna
  ou atribuicao (o remetente ja recebe aviso no sininho).
- **Contrato depois de fechar a venda:** gatilho do card ganho + acao `send_signature_document` (confira que
  a ficha tem todos os campos do modelo, ou use `collection: form`).
