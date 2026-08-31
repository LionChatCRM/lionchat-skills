# Base de conhecimento, perguntas frequentes e biblioteca de midia

Este e o material que faz a IA parar de inventar. Sem ele, ela so sabe o que esta escrito na instrucao - e
o que nao estiver escrito, ela preenche.

## Indice

1. Base de conhecimento: como montar
2. O que acontece quando voce salva um documento
3. Perguntas frequentes
4. Biblioteca de midia: cadastrar
5. Biblioteca de midia: liberar no agente
6. Ofertas e agendas: a mesma logica de liberar
7. Checklist do material

---

## 1. Base de conhecimento: como montar

Ferramenta: `lionchat_captain_documents_create`. Campos:

| Campo | Para que serve |
|---|---|
| `assistant_id` | de qual agente e esta pagina. **Documento e POR AGENTE, nao da conta** - dois agentes precisam de duas copias |
| `name` | titulo da pagina. E o que aparece ao apontar a pagina dentro de um cenario |
| `content` | o texto direto (ate 200.000 letras) |
| `external_link` | endereco de uma pagina da web para ler e indexar, no lugar do texto colado. Unico por agente |
| `pdf_file` | arquivo para extrair o texto (PDF, DOCX, XLSX/CSV, MD) - envio de arquivo, nao texto |
| `description` | descricao curta da pagina, ate 500 letras |

**Uma pagina por assunto.** Nao jogue o negocio inteiro numa pagina so: a busca funciona por semelhanca e
uma pagina gigante dilui o assunto certo.

Divisao que costuma funcionar:

- Precos e formas de pagamento
- Enderecos, horarios de funcionamento e como chegar
- Catalogo de produtos ou servicos, um por um
- Politica de troca, cancelamento e garantia
- Perguntas frequentes de verdade (as que a equipe mais responde)
- Regras de agendamento (antecedencia, tolerancia de atraso, o que levar)

**Escreva com o dado dentro.** "Consulte os valores com a recepcao" nao serve para nada. A base existe para
ter os numeros: valor, endereco completo, horario, prazo.

Para apontar uma pagina especifica dentro de um cenario, use `[@Nome da pagina](document://ID)` DENTRO do
texto da instrucao daquele cenario. Isso faz a busca priorizar aquela pagina, mas nao impede a IA de achar
outras. Teto de 3 paginas por cenario.

---

## 2. O que acontece quando voce salva um documento

1. Se algum pedaco da pagina ficar grande demais para a IA ler inteiro, o texto e reorganizado sozinho em
   blocos com titulo. Isso tem rede de seguranca: se o resultado ficar vazio, sem titulo, encolher mais de
   15% ou **mudar qualquer numero** (CEP, telefone, preco), o texto ORIGINAL e mantido. Pagina muito grande
   (acima de 60 mil letras) nao passa por essa reorganizacao.
2. O texto e quebrado em pedacos e indexado para a busca por semelhanca.
3. O documento fica com um status. **So documento disponivel serve a IA** - em processamento nao vale.
   Confira com `lionchat_captain_documents_list` (ela aceita modo enxuto, sem trazer o texto inteiro) ou
   `lionchat_captain_documents_show`.

**Sem chave da OpenAI cadastrada na conta, a indexacao e pulada** e o documento nunca serve para nada.
Confira a chave antes de cadastrar material.

Se o material mudou na origem (pagina da web) ou algo deu errado, reprocesse com
`lionchat_captain_documents_create_1` - o nome engana, mas e o reprocessar, nao um segundo jeito de criar.

---

## 3. Perguntas frequentes

Ferramenta: `lionchat_captain_assistants_create_2`. Campos: `assistant_id`, `question`, `answer` e
`status` (`approved` entra em uso, `pending` fica aguardando revisao humana).

E um par pergunta-resposta que a IA copia quase palavra por palavra quando casa. Bom para:

- respostas que precisam sair sempre iguais (politica, texto juridico, condicao comercial);
- perguntas que a equipe responde 20 vezes por dia.

Escreva a `question` do jeito que o CLIENTE faria, nao do jeito que a empresa fala:

- ruim: "Politica de reembolso"
- bom: "posso pedir meu dinheiro de volta?"

**Armadilha:** uma resposta que so NOMEIA algo sem trazer o dado faz a IA copiar o vazio. "A unidade mais
proxima e a de Sorocaba" nao ajuda ninguem: escreva o endereco completo.

Duas formas de a base crescer sozinha, se o cliente quiser:

- `config.feature_faq`: quando a conversa e resolvida, a IA gera perguntas e respostas daquela conversa
  para a base. Bom para base crescer com o que o time ja respondeu; exige alguem revisando o que entra.
- `config.feature_memory`: quando a conversa e resolvida, a IA extrai informacoes do cliente (preferencias,
  contexto) e grava como anotacao no contato, para lembrar na proxima conversa.

Ha uma ferramenta de acao em massa sobre as perguntas frequentes
(`lionchat_captain_assistants_bulk_actions`, pedindo tipo, lista de ids e a operacao). **Como ela pode
apagar em lote, nao use sem pedido explicito do cliente.**

---

## 4. Biblioteca de midia: cadastrar

Arquivos que a IA pode ENVIAR sozinha: catalogo em PDF, foto do produto, video institucional, audio. O
sistema tambem LE o conteudo do arquivo (descreve a imagem, transcreve o audio) e usa isso para a IA saber
o que mandou e responder perguntas sobre ele.

Ferramenta: `lionchat_captain_media_assets_create`. Campos:

| Campo | Onde vai | Para que serve |
|---|---|---|
| `blob_signed_id` | na raiz do corpo, FORA de `media_asset` | o arquivo em si (secao seguinte) |
| `media_asset.title` | ate 120 letras | nome do arquivo na biblioteca |
| `media_asset.description` | ate 2000 letras, obrigatorio | **QUANDO a IA deve enviar.** E o unico texto que ela le para decidir a hora certa |
| `media_asset.media_kind` | `image`, `video`, `audio` ou `document` | precisa casar com o tipo real, senao e recusado |
| `media_asset.skip_when` | opcional | quando ela NAO deve enviar. Evita mandar o catalogo inteiro para quem quer um produto so |
| `media_asset.enabled` | padrao ligado | liga e desliga sem apagar |
| `media_asset.extracted_content` | opcional | deixe em branco: o sistema le o arquivo sozinho. Preenchendo a mao, o texto passa a ser tratado como revisado por humano e a leitura automatica para de sobrescrever |

Limites de formato e tamanho seguem o que o WhatsApp aceita: imagem JPG/PNG/WEBP ate 5MB; video MP4/3GP
ate 16MB; audio MP3/AAC/M4A/AMR/OGG/OPUS ate 16MB; documento PDF/DOC/DOCX/XLS/XLSX/PPT/PPTX/TXT ate 50MB.
O limite que vale e sempre o MENOR entre o do WhatsApp e o configurado na plataforma - por isso o teto de
documento fica em 50MB, e nao nos 100MB do WhatsApp.

### Como conseguir o `blob_signed_id`

O arquivo em si nao sobe pelo cadastro de midia. Dois caminhos:

- **Se o arquivo ja esta numa URL publica** (site do cliente, link de armazenamento): use
  `lionchat_upload_create` mandando `external_url`. Ela baixa o arquivo com protecao contra endereco
  interno e devolve `blob_id` - esse valor e exatamente o que o cadastro de midia espera em
  `blob_signed_id`. Vale testar o link antes com `lionchat_upload_validate`, que diz se o tipo e o tamanho
  passam (e avisa quando o link e uma pagina web em vez do arquivo).
- **Se o arquivo esta so no computador do cliente**: nao ha como subir pelo conector. Peca para ele
  cadastrar pela tela Biblioteca de Midia e depois voce so libera os ids no agente.

Se o primeiro caminho falhar por qualquer motivo, caia no segundo em vez de insistir.

Depois de cadastrar, `lionchat_captain_media_assets_reprocess` manda reler o conteudo do arquivo.

---

## 5. Biblioteca de midia: liberar no agente

**Cadastrar NAO basta.** O arquivo aparece na tela da biblioteca, e a IA nao enxerga nem consegue enviar
enquanto o id nao estiver em `config.media_asset_ids` do agente. As tres ferramentas de midia so carregam
com essa lista preenchida.

```
lionchat_captain_assistants_update
  config.media_asset_ids: [12, 13, 14]
```

**Mande a lista COMPLETA** - array dentro de `config` e substituido inteiro, entao mandar so o id novo
apaga os outros. Leia o valor atual com `lionchat_captain_assistants_show` antes.

Ja houve um incidente por causa disso ao contrario: o texto do agente ensinava o caminho de enviar um
video, com a biblioteca vazia. A IA prometeu o video ao cliente e nao tinha o que enviar.

---

## 6. Ofertas e agendas: a mesma logica

Mesma armadilha, outros dois campos do agente:

- `config.offer_ids` - quais produtos ou planos ele pode consultar. Sem nenhum, a ferramenta de catalogo
  nem carrega. Liste com `lionchat_offers_list`.
- `config.booking_event_type_ids` - quais agendas ele pode oferecer. Liste com
  `lionchat_booking_event_types_list`. Preenchendo esse campo, ele passa a agendar SOMENTE por essas
  agendas. Detalhe que confunde: a ferramenta de "quem esta livre agora" recusa responder em QUALQUER
  agente que possa agendar por agenda - basta a conta ter uma agenda, mesmo sem esse campo preenchido.
  Isso e proposital: a agenda manda acima do expediente geral.

---

## 7. Checklist do material

Antes de dizer que a base esta pronta:

- [ ] Existe pagina com PRECO e com ENDERECO, se o negocio tem os dois.
- [ ] Nenhuma resposta manda "consulte a recepcao" ou "fale com um atendente" no lugar do dado.
- [ ] Todos os documentos estao disponiveis, nenhum em processamento nem com erro.
- [ ] As perguntas frequentes estao escritas do jeito que o cliente pergunta.
- [ ] Todo arquivo cadastrado na biblioteca tem uma descricao dizendo QUANDO enviar.
- [ ] Todo arquivo que a IA deve poder enviar esta liberado em `config.media_asset_ids`.
- [ ] Se o agente vai agendar, existe agenda e ela esta liberada.
- [ ] Se o agente vai falar de produto, existe oferta e ela esta liberada.
