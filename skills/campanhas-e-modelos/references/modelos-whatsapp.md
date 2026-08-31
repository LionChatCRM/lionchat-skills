# Modelo de mensagem do WhatsApp (o texto aprovado pela Meta)

Indice:

1. Para que serve e quando e obrigatorio
2. A regra da janela de 24 horas
3. Quem pode mexer e onde os modelos vivem
4. As 3 categorias
5. Os estados de um modelo
6. As partes do modelo: cabecalho, corpo, rodape, botoes
7. As combinacoes de botao que a Meta aceita
8. Variaveis: escrever, exemplificar e apontar de onde vem o valor
9. Mudar o modelo sem mandar para a Meta de novo
10. Arquivo padrao do cabecalho
11. O que a Meta mais recusa, e a saida de cada caso
12. Usar o modelo num disparo

---

## 1. Para que serve e quando e obrigatorio

Modelo e um texto que a Meta analisou e liberou. Ele e a UNICA forma de comecar conversa no WhatsApp
Oficial quando o cliente nao escreveu nas ultimas 24 horas. Consequencias praticas:

- Todo disparo em massa por caixa oficial gira em torno de um modelo aprovado.
- Em caixa por QR Code nao existe modelo nenhum: o texto vai livre.
- Modelo criado hoje nao dispara hoje. A analise da Meta leva de minutos a dias, e nao ha como
  acelerar nem consultar o motivo detalhado da recusa por aqui.

---

## 2. A regra da janela de 24 horas

| Situacao | O que da para mandar |
|---|---|
| O cliente escreveu ha menos de 24 horas | Qualquer coisa: texto livre, foto, audio |
| Passou de 24 horas | So modelo aprovado. Texto livre e recusado pela Meta |

Detalhes que confundem quase todo mundo:

- **Mandar um modelo NAO reabre a janela.** So a resposta DO CLIENTE reabre.
- Cartao de ligacao e recado interno nao contam como fala do cliente.
- **Conversa cujo historico foi importado do celular fica com a janela FECHADA**, mesmo com a fala
  do cliente visivel na tela: aquelas mensagens nunca passaram pela conexao oficial, e a Meta
  recusa. Nesse caso, a saida e um modelo.
- Em caixa por QR Code nao existe janela nenhuma. Isso NAO quer dizer que nao ha risco: ali o risco
  e o numero ser bloqueado pelo WhatsApp.

---

## 3. Quem pode mexer e onde os modelos vivem

- Criar, editar e excluir modelo exige perfil de ADMINISTRADOR.
- Os modelos moram na conta comercial do WhatsApp, entao sao compartilhados por todos os numeros
  daquela conta comercial.
- Teto de 250 modelos por conta. A listagem devolve o total e o teto.
- Depois de criar ou editar, rode `lionchat_inboxes_sync_templates`: a criacao devolve so o
  identificador e a situacao, e o conteudo do modelo so aparece na tela depois desse sincronismo.

---

## 4. As 3 categorias

| Categoria | Nome na tela | Para que serve | Efeito |
|---|---|---|---|
| `MARKETING` | Marketing | Promocao, novidade, oferta, convite | Mais caro e mais rigoroso na analise |
| `UTILITY` | Utilidade | Confirmacao, lembrete, atualizacao de pedido | Mais barato, aprova mais facil |
| `AUTHENTICATION` | Autenticacao | Codigo de verificacao | Regras proprias; so aceita o botao de codigo |

Existe um campo que deixa a Meta reclassificar o modelo sozinha em vez de recusar. Deixe ligado —
e o padrao da tela e evita recusa boba.

Regra de bolso: se o texto vende, e Marketing. Se o texto informa algo que a pessoa ja contratou ou
agendou, e Utilidade. Escrever "aproveite" ou "desconto" num modelo de Utilidade e o jeito mais
rapido de ser recusado.

---

## 5. Os estados de um modelo

| Estado | Nome na tela | O que significa |
|---|---|---|
| `APPROVED` | Aprovado | Pode usar |
| `PENDING` | Pendente | A Meta ainda esta analisando |
| `REJECTED` | Rejeitado | A Meta recusou |
| `PAUSED` | Pausado | A Meta pausou por qualidade ruim |
| `DISABLED` | Desativado | A Meta desabilitou de vez |

Duas consequencias que respondem perguntas reais do cliente:

- **"Meu modelo estava aprovado e parou de funcionar"**: provavelmente ele foi PAUSADO. O envio so
  encontra modelo APROVADO, entao um modelo pausado passa a falhar na Meta.
- **Editar so e possivel em modelo Aprovado ou Rejeitado.** Modelo Pendente, Pausado ou Desativado
  nao abre para edicao — nao mande o cliente "editar e reenviar" nesses casos.

---

## 6. As partes do modelo

Um modelo e feito de componentes. So o corpo e obrigatorio.

### Cabecalho (opcional)

| Formato | O que e |
|---|---|
| Texto | Um titulo de uma linha |
| Foto | JPG ou PNG, ate 5 MB |
| Video | MP4 ou 3GP, ate 16 MB |
| Documento | PDF, ate 100 MB |
| Localizacao | Um mapa fixo (endereco da loja ou clinica) |

O arquivo enviado na criacao serve so para a Meta olhar e aprovar o formato. Depois, cada envio pode
levar um arquivo diferente — ou o arquivo padrao do modelo (secao 10).

No cabecalho de localizacao as coordenadas nao vao para a Meta: ficam guardadas no LionChat e
acompanham cada mensagem.

### Corpo (obrigatorio)

- Ate 1024 caracteres.
- As variaveis se escrevem como `{{1}}`, `{{2}}` e assim por diante.
- **O limite conta o texto FINAL, com os valores das variaveis ja substituidos.** Um modelo que
  cabe na tela de criacao pode estourar na hora do envio e ser recusado. A tela reserva 50
  caracteres por variavel justamente por isso. Caso real: 1.025 caracteres, recusado.

### Rodape (opcional)

Ate 60 caracteres. A tela de criacao nao oferece variavel no rodape.

### Botoes (opcional)

Ver a secao seguinte — as combinacoes sao regra da Meta, nao do LionChat.

---

## 7. As combinacoes de botao que a Meta aceita

| Tipo | O que faz | Teto |
|---|---|---|
| Resposta rapida | O cliente clica e o texto do botao volta como mensagem dele | 3 |
| Link (URL) | Abre um endereco | 2, somando com telefone e chamada |
| Telefone | Liga para um numero informado | 2, somando com link e chamada |
| Chamada | Liga para o proprio numero do WhatsApp da empresa | 2, somando com link e telefone |
| Copiar codigo | Copia um cupom ou voucher | 1 |
| Codigo de verificacao | So em modelo de Autenticacao | 1 e exclusivo |

Regras de convivencia:

- Resposta rapida NAO se mistura com link, telefone ou chamada. Ou um grupo, ou o outro.
- Copiar codigo convive com os outros, menos com o de verificacao.
- O botao de codigo de verificacao e exclusivo: com ele, nenhum outro botao pode existir. E ele so
  existe na categoria Autenticacao.
- Botao de copiar codigo SEM exemplo e recusado pela Meta.

**Link do proprio WhatsApp em botao e PROIBIDO pela Meta** (convite de grupo ou link direto de
conversa). Nao adianta insistir — ja houve cliente tentando tres vezes achando que era instabilidade
da plataforma. As duas saidas:

1. Apagar o botao e escrever o link no TEXTO do corpo. Ele continua clicavel no celular.
2. Apontar o botao para um endereco proprio que redirecione para o WhatsApp.

**Link que muda por pessoa** (por exemplo um link de rastreio): o endereco do botao pode terminar em
`{{1}}`. Regras da Meta: no maximo UMA variavel, exatamente `{{1}}`, sempre NO FINAL, com o inicio
do endereco fixo. E obrigatorio mandar um exemplo do endereco completo, senao a Meta recusa na hora.

---

## 8. Variaveis: escrever, exemplificar e apontar de onde vem o valor

Sao tres coisas diferentes e o cliente costuma confundir:

1. **Escrever a variavel no texto** — `{{1}}` dentro do corpo.
2. **Dar um exemplo para a Meta** — obrigatorio na criacao, so serve para a analise.
3. **Apontar de onde vem o valor de verdade** — e o que faz o nome da pessoa aparecer no envio.

Duas regras da Meta que reprovam modelo:

- **Variavel nao pode abrir nem fechar o texto.** E a Meta ignora espaco e pontuacao ao redor:
  terminar com `{{1}}?` tambem e recusado.
- **O texto precisa ter conteudo real proporcional as variaveis.** Corpo que e quase so variavel e
  recusado.

### Apontar de onde vem o valor

Ferramenta: `lionchat_inboxes_whatsapp_templates_update`.

O mapa e por posicao. Para cada posicao, uma origem e um campo:

| Origem | Campos possiveis |
|---|---|
| `contact` (a ficha da pessoa) | `name`, `name.split.first` (primeiro nome), `phone_number`, `email`, ou `custom_attribute.<chave>` |
| `conversation` (o atendimento) | `display_id`, `assignee.name`, `team.name`, `custom_attribute.<chave>` |
| `account` (a empresa) | `name`, `custom_attribute.<chave>` |
| `custom` | um texto fixo, igual para todo mundo |

Exemplo:

```json
{ "1": { "source": "contact", "field": "name.split.first", "label": "Primeiro nome" },
  "2": { "source": "account", "field": "name", "label": "Empresa" } }
```

Tres coisas que precisam ser ditas ao cliente:

- **O mapa substitui tudo.** Posicao que voce nao mandar e REMOVIDA; mandar um mapa vazio limpa
  todos os apontamentos. Sempre releia o modelo e reenvie o conjunto completo.
- **Variavel sem apontamento nao falha o envio**: sai um PONTO no lugar dela, para todo mundo. Na
  lista de modelos aparece um aviso "Falta preencher variavel". Ja aconteceu com 65 pessoas de uma
  vez, que leram uma frase terminando em ponto no meio.
- Modelo de Autenticacao NAO aceita apontamento (o codigo nao pode virar nome de contato).

---

## 9. Mudar o modelo sem mandar para a Meta de novo

Esta e a distincao mais cara desta area:

| Acao | Passa pela Meta? | O modelo continua aprovado? |
|---|---|---|
| Editar o texto, o cabecalho ou os botoes | SIM | Volta para a fila de analise |
| Apontar de onde vem cada variavel | NAO | Sim |
| Definir, trocar ou remover o arquivo padrao do cabecalho | NAO | Sim |

Confundir os dois faz o cliente perder o modelo por dias sem necessidade. Enquanto a nova versao e
analisada, a versao anterior continua valendo — mas nao da para editar de novo nesse periodo.

---

## 10. Arquivo padrao do cabecalho

Ferramenta: `lionchat_inboxes_whatsapp_templates_header_media`.

Guarda um arquivo como padrao do modelo, para nao precisar escolher um a cada envio.

```json
{ "header_media": { "url": "...", "type": "image", "filename": "cardapio.png" } }
```

- `type` e `image`, `video` ou `document`, e precisa bater com o que o modelo espera.
- Mandar o pedido sem `header_media` REMOVE o arquivo padrao.
- Se o endereco for de arquivo nosso, ele precisa ser um arquivo de modelo DESTA CONTA (arquivo de
  outra conta, ou anexo de conversa, e recusado). O caminho seguro e pegar o endereco em
  `lionchat_inboxes_whatsapp_templates_media_library` da propria caixa.
- Endereco de fora funciona, mas se aquele site cair, o envio para de funcionar.
- Quem escolher um arquivo na hora do envio sempre ganha do padrao.

**O conector NAO sobe arquivo novo.** Se o cliente quer usar uma foto que nunca foi para o sistema,
ele sobe no painel e depois voce a reaproveita pela biblioteca de midia.

---

## 11. O que a Meta mais recusa, e a saida de cada caso

| O que aconteceu | Saida |
|---|---|
| Link do WhatsApp num botao | Apagar o botao e por o link no corpo, ou usar um endereco proprio que redireciona |
| Nome de modelo excluido ha pouco tempo | A Meta trava o nome por cerca de 30 dias e devolve uma mensagem enganosa (fala em semanas ou em "menos de 1 minuto"). Use um nome DIFERENTE, ou mantenha o nome e escolha a categoria Marketing |
| Ja existe modelo com esse nome | Use outro nome, ou edite o que existe |
| Falta exemplo de variavel | Preencha um exemplo em cada variavel |
| Variavel no comeco ou no fim do texto | Reescreva com texto real nas pontas |
| Texto acima do limite | Encurte, lembrando que o limite conta o valor final das variaveis |
| Categoria invalida | Escolha Marketing, Utilidade ou Autenticacao |

Regras de nome de modelo: so letras minusculas, numeros e sublinhado, comecando por letra. A tela
converte enquanto se digita (maiuscula vira minuscula, espaco vira sublinhado, acento sai).

Idiomas aceitos na tela: `pt_BR`, `en`, `en_US`, `es`, `fr`, `de`, `it`, `ja`, `ko`, `zh_CN`, `ar`,
`hi`. Mesmo texto em outro idioma e outro modelo.

**Excluir modelo apaga todas as versoes de idioma daquele nome** e trava o nome por cerca de 30
dias. Nunca proponha excluir para "recriar do jeito certo".

---

## 12. Usar o modelo num disparo

No `lionchat_campaigns_create` de caixa oficial, o modelo escolhido vai em `template_params`. O
modelo e encontrado pelo par NOME + IDIOMA, e so entre os APROVADOS:

```json
{
  "template_params": {
    "name": "aviso_black_friday",
    "language": "pt_BR",
    "category": "MARKETING",
    "processed_params": {
      "body": { "1": "{{contact.name}}", "2": "30%" }
    }
  }
}
```

- `processed_params.body` leva um valor por posicao. **Preencha TODAS as posicoes** — posicao vazia
  vira um ponto na mensagem de todo mundo.
- Os valores aceitam variavel de contato (por exemplo `{{contact.name}}`), resolvida por pessoa.
- Se o modelo tem cabecalho de arquivo e voce nao mandar nada, entra o arquivo padrao do modelo.
- Se o par nome e idioma nao encontrar nenhum modelo aprovado (por exemplo porque a Meta pausou o
  modelo), o envio E TENTADO e a Meta RECUSA — as falhas aparecem no placar. Isso e diferente de
  "nao saiu nada".
- Se `template_params` vier inteiramente vazio, ai sim NINGUEM recebe, sem erro visivel.
