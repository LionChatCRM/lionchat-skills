# Modelo de contrato - tipos, variaveis e configuracao

O modelo e o contrato padrao. Ele NAO manda nada: so serve de base para cada envio. Mudar o modelo depois
nao muda contrato ja enviado (o PDF de cada um e fixo, e varias escolhas sao gravadas no contrato quando ele
nasce: documentos pedidos, conferencia por IA, jeitos de assinar).

## Os tres tipos (`kind`)

| Tipo | O que e | Pelo conector? |
|---|---|---|
| `text` | Escrito no editor do LionChat, com `{{variaveis}}`. O PDF e gerado por pessoa. | **Sim** - e o unico que `signature_documents_create` cria. |
| `file` | PDF pronto, igual para todo mundo (ate 10 MB). | Nao - upload pela tela. |
| `word` | Arquivo do Word (.docx) com campos; os campos sao trocados dentro do arquivo e ele vira PDF, identico ao do escritorio. O mapeamento campo -> variavel e feito na tela. | Nao - upload e mapeamento pela tela. |

O texto do `text` aceita markdown simples (**negrito**, listas, paragrafos). Os blocos de assinatura de
cada pessoa sao desenhados sozinhos no fim (a folha de assinaturas).

## Variaveis

Lista oficial: `lionchat_signature_documents_field_catalog` (com a conta, inclui os atributos
personalizados de CONTATO). Os fixos:

| Variavel | O que e |
|---|---|
| `{{contact.name}}`, `{{contact.first_name}}`, `{{contact.last_name}}` | Nome completo, primeiro nome, sobrenome |
| `{{contact.cpf}}`, `{{contact.cnpj}}`, `{{contact.rg}}`, `{{contact.passport}}` | Documentos |
| `{{contact.date_of_birth}}`, `{{contact.marital_status}}`, `{{contact.profession}}`, `{{contact.gender}}` | Qualificacao |
| `{{contact.email}}`, `{{contact.phone_number}}` | Contato |
| `{{contact.address.street}}` `.number` `.complement` `.neighborhood` `.city` `.state` `.cep` | Endereco |
| `{{contact.custom_attribute.<chave>}}` | Atributo personalizado de contato (exceto o tipo segredo e os protegidos) |
| `{{account.name}}` | Nome da empresa |
| `{{document.deadline}}` | Data limite para assinar (sai da validade do modelo; o sistema preenche) |
| `{{document.today}}` | Data em que o contrato e gerado (o sistema preenche) |
| `{{document.validity_days}}` | Quantos dias o contrato vale |

So use variavel do catalogo: outra coisa (ex.: `{{agent.name}}`) nao e preenchida pelo contrato.

**Trava:** no envio "contrato pronto", toda variavel do modelo precisa ter valor na ficha, senao o envio
volta `422 variaveis_sem_valor` com `missing` (as chaves) e `missing_labels` (os nomes de gente). Antes de
enviar, `lionchat_signature_documents_fill_fields` (com `contact_id`) devolve cada campo com `value` e
`missing`, e `blocked` = o que falta e NAO se preenche no proprio envio (telefone, dados da conta). Com algo em
`blocked`, corrija a ficha ou a conta antes.

## Configuracao do modelo (`roles_layout`)

E um objeto. No `update` ele e MESCLADO no primeiro nivel: mande so as chaves que mudam (as posicoes de
assinatura e o mapeamento do Word gravados pela tela ficam intactos). Dentro de `communication`, o que nao
vier fica o que estava guardado.

### Quem assina e prazo

| Chave | O que faz |
|---|---|
| `signers` | Quantos titulares assinam (numero inteiro; lido de 1 a 10; padrao 1). |
| `witnesses` | Quantas testemunhas (numero inteiro; lido de 0 a 10; padrao 0). |
| `sender_signs` | `true` = quem envia tambem assina ("eu tambem assino"), no PAINEL, sem link. |
| `validity_days` | Validade em dias contada do ENVIO (1 a 365; padrao 7). Passou = contrato `expired` (conferido de hora em hora). |
| `signature_methods` | Como a pessoa pode assinar: `["drawn","typed"]` (os dois, padrao), `["drawn"]` (so desenhar) ou `["typed"]` (so digitar o nome). Vazio ou outro valor = 422. |

**Formato:** `signers`, `witnesses` e `validity_days` sao NUMEROS puros (`{"signers": 2, "witnesses": 1,
"validity_days": 7}`); lista ou objeto ai e recusado. `sender_signs` e `true`/`false`.

**Ordem:** os links saem para todos ao mesmo tempo - nao ha assinatura em sequencia. A ordem dos
participantes so define a posicao de cada um no papel.

### Como o formulario pergunta (`field_prompts`)

Usado no jeito de envio "formulario" (a propria pessoa preenche os dados): `{"contact.cpf": {"label":
"Qual e o seu CPF?", "description": "Igual ao documento", "placeholder": "000.000.000-00"}}`. Texto ate
200 caracteres. Pergunta de variavel que o modelo nao usa e descartada. Sem pergunta, o formulario mostra o
nome do campo.

### Documentos pedidos antes de assinar (`proof_requests`)

Lista de itens `{key, label, accepts_pdf, attach_to_contract, from_witnesses, accepted_types, description}`.
Maximo 12 itens.

| `key` | O que pede |
|---|---|
| `id_document` | Documento com foto. Com `accepted_types` (`rg`, `cnh`, `cpf`; um ou mais) a pessoa manda QUALQUER UM dos aceitos. Sem a lista = os tres. |
| `selfie_with_document` | Selfie segurando o documento |
| `address_proof` | Comprovante de endereco (foto ou PDF) |
| `rg`, `cnh`, `cpf` | Cada um EXIGE aquele documento (os tres soltos = a pessoa manda os tres). |
| `custom_<6 a 12 letras/numeros minusculos>` | Documento criado por voce; `label` obrigatorio (1 a 60 caracteres). |

- **"RG ou CNH" = `{"key": "id_document", "accepted_types": ["rg", "cnh"]}`.** Nunca `rg` e `cnh` soltos.
- **A chave do documento criado e VOCE quem inventa** (ex.: `custom_oab2026`) e ela tem de ser mantida igual
  nas gravacoes seguintes: trocar a chave vira outro documento.
- `accepts_pdf` so vale no comprovante e nos criados (RG/CNH/CPF/selfie com PDF = 422).
- `attach_to_contract: true` = o arquivo entra como paginas no contrato assinado. Teto de 10 arquivos
  anexados por contrato (itens com "anexar" x pessoas que mandam); acima disso, 422 ao salvar ou
  `anexos_demais` ao enviar. A testemunha recebe a via dela SEM as paginas de anexo.
- `from_witnesses: true` = testemunha tambem manda (padrao: sim nos fixos, nao nos criados).
- `description` (so nos criados, ate 300 caracteres): explica a IA o que o documento e. Quem assina NAO ve.
  Em documento fixo = 422. Sugira quando o nome nao diz tudo.
- `required_proofs` e `require_id_photo` sao o formato antigo e continuam aceitos; prefira `proof_requests`.

**Com qualquer documento pedido, o contrato NAO fecha quando todos assinam**: fica "aguardando aprovacao"
ate a equipe aprovar (ver `enviar-e-acompanhar.md`). Avise o cliente disso na proposta.

`ai_document_check: true` liga a conferencia das fotos e dos PDFs de ate 5 paginas pela IA, usando a chave
da OpenAI DA CONTA (gasta credito dela; sem a chave nao confere). Arquivo em duvida pede outro; da 2a duvida
em diante a pessoa pode "Manter". Falha da IA = arquivo aceito. E um AVISO para a equipe olhar com
atencao, nao uma recusa.

### Caixa do modelo e mensagens (`communication`)

`{inbox_id, code_message, invite_message, copy_message, proofs_again_message, code_template,
invite_template, copy_template, proofs_again_template}`.

- `inbox_id` = caixa de WhatsApp da conta (Oficial ou QR Code). Com ela, TODA a comunicacao dos contratos
  do modelo sai por ali: codigo, link de quem assina e das testemunhas, via assinada, pedido de documento de
  novo e o link para divulgar. Sem caixa, cada contrato sai pela conversa da pessoa.
- **Caixa de QR Code:** os quatro textos (em branco = o texto padrao). `code_message` precisa ter
  `{{codigo}}`; `invite_message`, `copy_message` e `proofs_again_message` precisam ter `{{link}}`.
  `{{contrato}}` poe o nome do contrato; `{{motivo}}` poe o motivo da equipe no pedido de documento de novo.
  Sem a variavel obrigatoria = 422.
- **Caixa Oficial:** os quatro modelos do WhatsApp, cada um `{name, language}` de um modelo APROVADO com UMA
  variavel no corpo e sem foto, arquivo ou variavel no cabecalho; o do codigo e de Autenticacao. Modelo
  ainda em analise pode ser salvo, mas so vale quando a Meta aprovar (ate la sai o texto padrao). `null`
  tira. Para criar o modelo na Meta existe `lionchat_inboxes_whatsapp_templates_create` (codigo: categoria
  `AUTHENTICATION`) - confirme antes com o cliente: cada codigo enviado por ele e cobrado pela Meta.

### Link do contrato para divulgar (`public_link`)

`{enabled: true}` (usa a caixa de `communication`). Quem abre o endereco preenche um formulario (dados do
contrato, o WhatsApp dela e as testemunhas do modelo), recebe um codigo no WhatsApp e, confirmando, o
contrato dela nasce.

- So para modelo com UM titular e sem "eu tambem assino" (422 com `public_link` no texto), e so caixa de
  WhatsApp. Na Oficial o link fica indisponivel ate o modelo do codigo estar aprovado.
- A leitura devolve `{enabled, inbox_id, path}`; o endereco e o dominio da instalacao + `path`
  (`/contrato/<codigo>`). `regenerate: true` troca o codigo e o endereco antigo PARA de funcionar - pergunte
  antes. Salvar sem `regenerate`, ou desligar e religar, mantem o endereco.
- Contato que ja existe nao tem a ficha alterada. A mesma pessoa (mesmo WhatsApp e CPF) preenchendo de novo
  em 24 h ganha contrato novo e o anterior sem assinatura e cancelado.
- So conta no limite do mes depois que a pessoa confirma o codigo; quem abre e some nao gasta vaga.

## Editar, arquivar e apagar

- `lionchat_signature_documents_update`: nome, texto (so `text`), `tags` (substitui a lista; ate 20),
  `archived`, `roles_layout`.
- Arquivar (`archived: true`) e o caminho normal para tirar de uso: o modelo some da lista padrao.
  **Atencao:** modelo arquivado NAO e achado pelo bloco do fluxo, pela macro nem pela automacao (erro
  `modelo_nao_encontrado`).
- `lionchat_signature_documents_destroy`: so administrador. Os contratos ja enviados ficam (sem o nome do
  modelo). Prefira arquivar e confirme antes de apagar.
