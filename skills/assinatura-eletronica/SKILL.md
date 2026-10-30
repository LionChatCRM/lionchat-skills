---
name: assinatura-eletronica
description: Monta, envia e acompanha a assinatura eletronica de contratos do LionChat pelo conector - modelo de contrato com variaveis da ficha, quem assina (titular, testemunha, "eu tambem assino"), validade, documentos pedidos (RG, CNH, selfie, comprovante) e a conferencia pela equipe, caixa e mensagens do codigo, link para divulgar, envio pelo fluxo, macro ou automacao, lembrete por fluxo, reenvio, cancelamento e limite do mes. Use quando o usuario disser "quero mandar contrato para o cliente assinar", "cria um modelo de contrato", "o cliente nao recebeu o link do contrato", "o contrato nao saiu", "quero pedir RG e comprovante antes de assinar", "aprova os documentos", "quero lembrar quem nao assinou", "o fluxo nao manda o contrato", ou pedir assinatura digital/eletronica de documento. NAO e a assinatura do PLANO (cobranca). Sempre pergunta antes e so cria ou envia depois de confirmacao explicita.
---

# Assinatura eletronica de contratos no LionChat

O cliente final assina um contrato pelo celular: recebe um link (no WhatsApp ou por e-mail), confirma que
e ele com um codigo de 6 digitos, desenha ou digita a assinatura, e o LionChat gera o PDF final com a
prova (IP, aparelho, localizacao, hash e relatorio de evidencias). Na tela isso fica no menu
**Assinaturas**, na aba **Contratos** da ficha do contato e em pilulas na conversa.

**Nao confundir com a "assinatura" do PLANO** (fatura, cartao, cobranca do LionChat): essa area fica FORA
do conector e se resolve no painel. Aqui e so contrato que o CLIENTE DO CLIENTE assina.

Sao tres camadas, e confundir as duas primeiras e o erro mais comum:

| Camada | O que e | Ferramentas |
|---|---|---|
| **Modelo** | O contrato padrao, escrito UMA vez com `{{variaveis}}` da ficha. Nao manda nada sozinho. | `lionchat_signature_documents_*` (list, show, create, update, destroy, field_catalog, fill_fields) |
| **Contrato** | CADA envio a uma pessoa: PDF proprio com os dados dela, prazo, linha do tempo. Mandar o mesmo modelo para 5 clientes = 5 contratos. | `lionchat_signature_envelopes_*` (create, list, show, resend, cancel, sign_as_sender, limits, proofs, approve_proofs, reject_proofs) |
| **Participante** | Quem assina DENTRO de um contrato: titular (`signer`, pode ser mais de um), testemunha (`witness`, opcional) e o remetente (`sender`, quando o modelo tem "eu tambem assino" - assina no painel, sem link). Cada um tem link e codigo proprios. | dentro do contrato (`participants[]`) |

## Regras que nunca mudam

- **Enviar contrato manda mensagem para pessoa REAL.** `signature_envelopes_create`, `_resend`,
  `_approve_proofs` e `_reject_proofs` entregam WhatsApp ou e-mail ao cliente na hora. Nenhuma dessas
  ferramentas tem trava de confirmacao no conector - **a trava e voce**: mostre a quem vai, por qual
  caixa e com que texto, e so chame depois de um "sim, pode enviar" explicito. Autorizacao para MONTAR o
  modelo nao e autorizacao para ENVIAR.
- **O conector so cria modelo ESCRITO** (`kind: text`). Modelo de PDF pronto ou de Word e subido pela tela.
  Pre-visualizar o modelo e baixar os PDFs (original, assinado, evidencias) tambem so pela tela - nao
  prometa "vou gerar a previa" nem "vou baixar o assinado".
- **Campo do modelo sem valor na ficha BLOQUEIA o envio** (`422 variaveis_sem_valor`, com a lista).
  Confira com `lionchat_signature_documents_fill_fields` antes de enviar.
- **Cada contrato enviado gasta 1 vaga do limite do mes, e cancelar nao devolve.** Veja
  `lionchat_signature_envelopes_limits` antes de qualquer envio em quantidade.
- **Documento de identidade e selfie nunca saem por ferramenta.** Para olhar, a pessoa usa a tela (botao
  amarelo "Ver documentos"). Nunca aprove documento que ninguem olhou.
- **Lembrete NAO e campo do modelo.** O antigo `reminder_days` foi removido (14/09/2026). Quem lembra e um
  fluxo do cliente (ver `references/no-fluxo.md`). Nunca sugira `reminder_days`.
- **Nunca assine no lugar do cliente.** `sign_as_sender` serve SO para a assinatura do proprio usuario do
  conector, quando o modelo tem "eu tambem assino".
- Se uma ferramenta `lionchat_signature_*` nao estiver visivel, PESQUISE pelo nome antes de dizer que nao
  existe.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 - Conferir os pre-requisitos

Use `lionchat_signature_documents_list`, `lionchat_signature_envelopes_limits` e `lionchat_inboxes_list`.

1. **O recurso "Assinatura de documentos" ligado na conta** (`document_signing`). Desligado, toda ferramenta
   da area volta `403 feature_disabled`. Isso NAO se liga pelo conector (e ligado por conta no Super Admin):
   se faltar, PARE e avise o cliente.
2. **Permissao de quem esta conectado:** administrador, ou cargo personalizado com "Gerenciar Assinaturas"
   (`signature_manage`). Sem ela da para LISTAR e LER modelos, mas nao criar, editar, enviar, acompanhar
   nem aprovar documentos. Cancelar contrato e apagar modelo: so administrador.
3. **Vagas do mes:** `limits` devolve `{used, limit, unlimited}`.
4. **Uma caixa de WhatsApp** (Oficial ou QR Code) para o link sair pelo WhatsApp. Sem nenhuma, so por e-mail.
5. **Modelos que ja existem:** se o contrato ja existe como modelo, ajuste-o; nao crie outro igual.
6. Se for pedir conferencia dos documentos por IA: a conta precisa da chave da OpenAI cadastrada
   (Integracoes). Nao se cadastra pelo conector.

### Etapa 2 - Entender (1 ou 2 perguntas por vez)

1. Que contrato e esse e quem assina? (so o cliente; dois socios; com testemunha; voce tambem assina?)
2. O texto ja existe? E Word ou PDF do escritorio (sobe pela tela) ou podemos escrever aqui?
3. Que dados do cliente entram no contrato? (nome, CPF, endereco, estado civil, profissao, campo proprio)
4. Esses dados ja estao na ficha, ou o proprio cliente deve preencher num formulario antes?
5. Quantos dias o cliente tem para assinar? (padrao 7)
6. Precisa pedir documento antes de assinar (RG ou CNH, selfie com o documento, comprovante de
   endereco, outro)? Alguem da equipe vai conferir?
7. Por qual caixa saem o link e o codigo? Se for WhatsApp Oficial: ja existem os modelos aprovados na Meta?
8. Como o contrato sai: na mao (por voce, agora), por um fluxo, por uma automacao, por um link para
   divulgar? Quer lembrete para quem nao assinou?

### Etapa 3 - Propor e confirmar

Mostre ANTES de gravar: o texto do modelo com as variaveis destacadas, a configuracao (quem assina,
validade, documentos pedidos, caixa e mensagens, jeitos de assinar) e, se houver, o desenho do fluxo.
So grave depois de um "sim" explicito. Para ENVIAR, peca uma segunda confirmacao, com a lista de quem
recebe, o telefone ou e-mail de cada um, a caixa e a mensagem.

### Etapa 4 - Montar

1. Variaveis disponiveis: `lionchat_signature_documents_field_catalog`.
2. Modelo: `lionchat_signature_documents_create` (ou `_update` no existente). Campos e configuracao em
   `references/modelos-e-campos.md`.
3. Fluxo, automacao ou macro que envia, quando pedido: `references/no-fluxo.md` e a skill `criar-fluxos`.

### Etapa 5 - Testar (com autorizacao)

1. Peca autorizacao para um envio de teste. Ele gasta 1 vaga do mes e manda mensagem de verdade.
2. Mande para um contato de TESTE com o telefone do proprio cliente (nunca para um cliente dele sem pedir):
   `lionchat_signature_documents_fill_fields` -> `lionchat_signature_envelopes_create`.
3. Leia a resposta: `delivery_failures[]` vazio? Depois `lionchat_signature_envelopes_show` (linha do tempo:
   `delivered`, `opened`, `code_verified`, `signed`, `sealed`).
4. Peca ao cliente para abrir o link e conferir o PDF com os dados certos. A previa e o PDF so pela tela.
5. Fluxo: teste num contato de teste e confira a execucao (`lionchat_flows_executions_list`,
   `lionchat_flow_sessions_show`); o bloco mostra o motivo em texto quando da erro.

### Etapa 6 - Relatar

Diga o que foi criado (com ids), o que foi enviado e para quem, o que foi testado e o que ficou para o
cliente conferir na tela. Nunca diga "esta funcionando" sem um envio de teste lido na linha do tempo.

## Operar no dia a dia (sempre com confirmacao antes de qualquer acao que fala com o cliente)

- Acompanhar: `lionchat_signature_envelopes_list` (filtros `status`, `contact_id`, `q`) e `_show`.
- Nao chegou: `_resend` (nao cria contrato nem gasta vaga).
- Errou o envio: `_cancel` (so administrador; nao devolve a vaga).
- Documentos pedidos: `_proofs`, depois `_approve_proofs` ou `_reject_proofs`.
- Falta a sua assinatura: `status=falta_eu` e `_sign_as_sender` (ou o botao "Assinar agora" na tela).
Detalhe em `references/enviar-e-acompanhar.md`.

## Referencias

- `references/modelos-e-campos.md` - tipos de modelo, variaveis, quem assina, validade, documentos pedidos,
  caixa e mensagens, jeitos de assinar, link para divulgar.
- `references/enviar-e-acompanhar.md` - os dois jeitos de enviar, canais e codigo, estados, linha do tempo,
  reenvio, cancelamento, aprovacao de documentos, via assinada, limite do mes.
- `references/no-fluxo.md` - bloco "Enviar documento para assinatura", macro e automacao, os 12 gatilhos,
  variaveis `{{contrato.*}}`, condicao "Situacao do contrato" e a receita do lembrete.
- `references/armadilhas-e-diagnostico.md` - cada codigo de erro e o que fazer; sintomas sem erro.
