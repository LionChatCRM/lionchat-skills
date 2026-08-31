# Tipos de campo e o que da para filtrar

Leia antes de escolher o tipo de qualquer campo personalizado e antes de montar filtro ou segmento.

## Indice

1. A regra de ouro do tipo
2. Os 11 tipos e o que cada um aceita comparar
3. Como criar o campo (o que a ferramenta pede)
4. Tudo que da para filtrar num contato
5. Como se escreve uma condicao de filtro
6. Datas: os atalhos e o intervalo
7. Filtrar contato pelo que aconteceu na conversa ou no card
8. Segmento salvo: o que ele e e o que ele nao e
9. Onde o filtro de contato tambem e usado

---

## 1. A regra de ouro do tipo

O tipo do campo decide quais comparacoes existem para ele no filtro. **Oferecer uma comparacao que o
tipo nao aceita nao devolve "nenhum resultado": derruba a consulta inteira com erro.**

Isso ja aconteceu em producao mais de uma vez — dez campos de contato dos tipos Moeda, Porcentagem e
Hora apareciam na lista e quebravam ao aplicar o filtro. Nao e teoria.

Existe uma diferenca importante entre campo nativo e campo personalizado:

- **Campo nativo** (nome, telefone, cidade, etiqueta, criado em...): se voce mandar uma comparacao
  fora da lista dele, o sistema RECUSA com mensagem clara. Barulhento, mas seguro.
- **Campo personalizado**: nao ha essa porta. A comparacao entra direto na consulta e, se o tipo nao
  suportar, o filtro estoura. Por isso a tabela abaixo e obrigatoria.

Consequencia pratica: **escolha o tipo pensando na pergunta que o cliente vai querer fazer depois**,
nao no formato do dado. Se ele vai perguntar "quem tem valor acima de 5 mil", precisa ser Numero ou
Moeda. Se ele vai perguntar "quem tem a palavra X escrita ali", precisa ser Texto.

## 2. Os 11 tipos e o que cada um aceita comparar

| Tipo (nome na tela) | Guarda | Comparacoes que existem |
|---|---|---|
| Texto | qualquer texto | igual, diferente, contem, nao contem, preenchido, nao preenchido |
| Link | um endereco | igual, diferente, contem, nao contem, preenchido, nao preenchido |
| Lista | uma opcao de uma lista fixa que voce define | igual, diferente, preenchido, nao preenchido |
| Caixa de selecao | sim ou nao | igual, diferente, preenchido, nao preenchido |
| Numero | numero | igual, diferente, preenchido, nao preenchido, maior que, menor que |
| Moeda | numero (o simbolo e so exibicao) | igual, diferente, preenchido, nao preenchido, maior que, menor que |
| Porcentagem | numero | igual, diferente, preenchido, nao preenchido, maior que, menor que |
| Data | uma data | igual, diferente, preenchido, nao preenchido, maior que, menor que |
| Data e Hora | data com horario | igual, diferente, preenchido, nao preenchido, maior que, menor que |
| Hora (24h) | horario do dia, HH:MM | igual, diferente, preenchido, nao preenchido, maior que, menor que |
| Confidencial | valor secreto, guardado criptografado | NENHUMA que funcione — filtrar por ele quebra a consulta |

Notas que mudam decisao:

- **"Contem" so existe em Texto e Link.** Se o cliente disser "quero achar quem tem *parte* disso",
  o campo tem que ser Texto. Lista nao aceita "contem".
- **"Maior que" e "menor que" so existem onde ha ordem**: numero, moeda, porcentagem, data, data e
  hora, e hora. Em Hora a comparacao e cronologica, entao "maior que 12:00" traz a tarde.
- **"Esta preenchido" e "nao esta preenchido" funcionam em todos os tipos** e sao a forma mais util
  de achar cadastro incompleto ("quem ainda nao tem o CPF").
- **Data e Hora e comparado com precisao de DIA** no filtro, por decisao de produto. Nao espere
  recorte por hora exata.
- **Nunca monte filtro em campo Confidencial.** A tela ate oferece "igual a" e "diferente de" para
  ele, mas o servidor nao sabe comparar esse tipo e a consulta quebra. O valor tambem nao aparece no
  seletor de variaveis de mensagem: ele so e usado dentro de uma chamada de API no Fluxo. Nao
  proponha Confidencial para nada que o cliente queira ver na tela, usar em texto ou filtrar.

## 3. Como criar o campo (o que a ferramenta pede)

A ferramenta de criar campo personalizado exige tres coisas e aceita mais algumas:

| Campo | Obrigatorio | O que colocar |
|---|---|---|
| `attribute_display_name` | sim | O nome que a equipe ve. Pode ter acento e espaco: "Endereco da unidade" |
| `attribute_key` | sim | A chave tecnica. **Minusculas, sem acento e sem espaco** — use `_`. Ex.: `endereco_unidade` |
| `attribute_display_type` | sim | Um de: `text`, `number`, `currency`, `percent`, `link`, `date`, `list`, `checkbox`, `time`, `datetime`, `secret` |
| `attribute_model` | nao, mas SEMPRE mande | `contact_attribute` (fica na ficha) ou `conversation_attribute` (fica no atendimento). **O padrao e conversa** — sem mandar, o campo do contato nasce no lugar errado |
| `attribute_values` | so no tipo Lista | As opcoes fixas, ex.: `["Basico", "Premium"]` |
| `attribute_timezone` | so no tipo Hora | Fuso, padrao America/Sao_Paulo |
| `attribute_description` | nao | Explica ao time para que serve o campo. Aparece na ficha e no seletor de variaveis |

Regras que valem a pena saber antes de criar:

- **Criar, alterar e excluir campo exige perfil de administrador.** Se o cliente nao for, avise.
- **A chave e forcada para minusculas e sem acento na criacao.** "Endereco Tatuape" com acento vira a
  chave sem acento; escrever a chave com acento na mensagem devolve branco.
- **Espaco na chave e recusado.** Escreva com `_`.
- **A chave nao muda de verdade depois.** Renomear a chave de um campo que ja tem dado gravado NAO
  move o dado: os contatos ficam com o valor preso na chave antiga. Se precisar mudar, crie um campo
  novo e refaca o preenchimento.
- **Nome que colide com campo nativo e recusado.** A lista esta em
  `campos-nativos-e-etiquetas.md`.
- **A tela de criacao so oferece Texto, Numero, Link, Data, Hora, Data e Hora, Lista e Caixa de
  selecao.** Moeda, Porcentagem e Confidencial funcionam por aqui mas nao aparecem naquele seletor —
  o cliente vai editar como texto se abrir a tela. So proponha esses tres se ele entender isso.
- **Card do Kanban e outro lugar.** Esta ferramenta so cria campo de contato e de conversa. Campo de
  card mora na configuracao do Kanban e usa outro vocabulario de tipos. Cuidado com um detalhe: na
  TELA o terceiro item do seletor "Aplica-se a" e Card; nas ferramentas, o numero 2 significa
  Variavel da Conta. Nao copie o numero da tela.

## 4. Tudo que da para filtrar num contato

Esta e a lista fechada de chaves nativas. **Qualquer outra chave e interpretada como campo
personalizado do contato.**

| Chave | O que e | Comparacoes aceitas |
|---|---|---|
| `id` | o numero da ficha | igual, diferente |
| `name` | nome | igual, diferente, contem, nao contem |
| `email` | e-mail | igual, diferente, contem, nao contem |
| `phone_number` | telefone | igual, diferente, contem, nao contem, **comeca com** |
| `identifier` | codigo em outro sistema | igual, diferente, contem, nao contem |
| `city` | cidade | igual, diferente, contem, nao contem |
| `country_code` | pais | igual, diferente |
| `company` | empresa | igual, diferente, contem, nao contem |
| `cpf` | CPF | igual, diferente, contem, nao contem, preenchido, nao preenchido |
| `cnpj` | CNPJ | igual, diferente, contem, nao contem, preenchido, nao preenchido |
| `rg` | RG | igual, diferente, contem, nao contem, preenchido, nao preenchido |
| `profession` | profissao | igual, diferente, contem, nao contem |
| `labels` | etiquetas do CONTATO | igual, diferente, preenchido, nao preenchido |
| `blocked` | contato bloqueado | igual, diferente (valores `true` / `false`) |
| `created_at` | data de cadastro | maior que, menor que, hoje, ontem, nos ultimos N dias, N dias atras, mais de N dias atras |
| `last_activity_at` | ultima atividade | as mesmas de `created_at` |
| `conversation_status` | situacao de alguma conversa dele | igual, diferente |
| `conversation_labels` | etiqueta de alguma conversa dele | igual, diferente, preenchido, nao preenchido |
| `conversation_inbox_id` | caixa de entrada de alguma conversa dele | igual, diferente |
| `card_funnel_id` | funil de algum card dele | igual, diferente |
| `card_stage` | etapa do card | igual, diferente |
| `card_priority` | prioridade do card | igual, diferente |
| `card_status` | situacao do card | igual, diferente |

Observacoes:

- **`comeca com` existe so no telefone.** E o jeito de filtrar por DDD.
- **A busca de texto ignora acento e maiuscula** nas duas direcoes: quem digita sem acento acha o
  contato cadastrado com acento, e o contrario tambem.
- **`labels` aqui e a etiqueta do CONTATO**, nao a da conversa. As duas sao marcacoes diferentes.
- Os campos de origem e de rastreamento (primeira origem, ultima origem, UTM, cliques) sao campos
  personalizados de sistema: nao aparecem na lista de campos por padrao, mas **funcionam no filtro**.
  Peca a listagem incluindo os campos de sistema para descobrir as chaves.

## 5. Como se escreve uma condicao de filtro

A ferramenta de filtrar contatos recebe uma lista de condicoes. Cada condicao tem:

| Parte | O que e |
|---|---|
| `attribute_key` | a chave da tabela acima, ou a chave de um campo personalizado seu |
| `filter_operator` | `equal_to`, `not_equal_to`, `contains`, `does_not_contain`, `is_present`, `is_not_present`, `is_greater_than`, `is_less_than`, `starts_with`, `today`, `yesterday`, `last_days`, `days_ago`, `days_before` |
| `values` | lista de valores. Varios valores na mesma condicao funcionam como "ou" entre eles |
| `query_operator` | `and` ou `or` — liga esta condicao com a PROXIMA. Na ultima, deixe vazio |
| `custom_attribute_type` | so quando a chave for campo de CONVERSA e voce estiver filtrando contatos: `conversation_attribute` |

Coisas que quebram e nao sao obvias:

- **O "e" e o "ou" sao encadeados, sem parenteses.** Nao ha como montar "(A ou B) e C". Se o cliente
  pedir isso, quebre em dois recortes ou explique a limitacao.
- **`is_present` e `is_not_present` nao levam valor.** `today` e `yesterday` tambem nao. Todas as
  outras comparacoes EXIGEM valor; sem ele a condicao e recusada.
- **A resposta vem paginada, 15 por vez.** Para percorrer o filtro inteiro, avance as paginas ate
  acabar. Nao afirme um total ao cliente olhando so a primeira pagina.

## 6. Datas: os atalhos e o intervalo

Quando o cliente falar "hoje", "ontem", "nos ultimos 7 dias", **use os atalhos**, nao data fixa: eles
resolvem no fuso da conta e nao erram a virada do dia.

| O cliente diz | Use |
|---|---|
| "hoje" | `today` (sem valor) |
| "ontem" | `yesterday` (sem valor) |
| "nos ultimos 7 dias" | `last_days` com valor `7` |
| "exatamente 7 dias atras" | `days_ago` com valor `7` |
| "mais de 30 dias atras" | `days_before` com valor `30` |
| "depois de 01/07" | `is_greater_than` com a data |
| "antes de 10/07" | `is_less_than` com a data |

**Nao existe "entre duas datas".** Sao duas condicoes ligadas por `and`, e **nenhuma das duas pontas
entra no resultado**. Para incluir os dias 01 e 10, peca de 30/06 a 11/07. O numero de dias tem teto
de 998.

## 7. Filtrar contato pelo que aconteceu na conversa ou no card

Este e o pedido mais comum do cliente leigo e quase sempre e resolvido sem criar campo nenhum:

- "quero a lista de quem esta na etapa Negociacao" -> `card_stage`
- "quem tem conversa aberta na caixa do WhatsApp" -> `conversation_status` mais `conversation_inbox_id`
- "quem tem a etiqueta orcamento em alguma conversa" -> `conversation_labels`
- "quem preencheu o campo X em algum atendimento" -> a chave do campo de conversa, com
  `custom_attribute_type` igual a `conversation_attribute`

Uma coisa a saber sobre a negacao: `not_equal_to`, `does_not_contain` e `is_not_present` nessas
chaves significam **"nenhuma conversa dele casa"**, e por isso incluem tambem quem nao tem conversa
nenhuma. Avise o cliente, porque o resultado costuma vir maior do que ele imagina.

## 8. Segmento salvo: o que ele e e o que ele nao e

Um segmento e um conjunto de condicoes guardado com um nome. Ele e **dinamico**: mostra sempre quem
atende as condicoes AGORA, nao uma lista congelada.

Para criar, use a ferramenta de filtros salvos com o tipo de filtro `contact` e a mesma lista de
condicoes da secao 5.

**Cuidado com o embrulho, que e diferente do filtro avulso.** No filtro de contatos as condicoes vao
soltas, dentro de `payload`. No segmento salvo elas vao **dentro de `query`, e ainda dentro de
`payload`**: `query` recebe `{ "payload": [ ...as condicoes... ] }`. Mandar a lista direta em `query`
NAO da erro — o segmento e criado, aparece na barra lateral com o nome certo e nao filtra nada. Ao
terminar, abra o segmento e confira que ele mostra gente.

**Duas coisas que o cliente precisa ouvir de voce:**

1. **O segmento e privado de quem criou.** O colega da equipe nao ve. A documentacao antiga diz o
   contrario; o comportamento real e privado. Diga isso ao criar, senao alguem vai concluir que o
   sistema perdeu o dado.
2. **Segmento NAO serve como publico de campanha.** O publico da campanha entende tag da conversa,
   tag do contato, atributo de conversa, atributo de contato, atributo de card, funil e atendente ou
   time — **nunca um segmento salvo**. Se o cliente quer disparar para um recorte, o recorte precisa
   ser remontado dentro da campanha, ou virar uma etiqueta de contato aplicada em massa.

## 9. Onde o filtro de contato tambem e usado

A mesma linguagem de condicoes aparece em outros lugares, e por isso o tipo do campo importa alem da
tela de Contatos:

- na **acao em massa** de contatos, quando se escolhe "todos os contatos deste filtro" em vez de uma
  lista de fichas;
- na **exportacao** de contatos, que respeita as condicoes enviadas;
- no **segmento salvo**, que e essa mesma consulta com um nome.

Ou seja: um campo com o tipo errado nao quebra so o filtro — quebra tambem a exportacao e a acao em
massa que dependem dele.
