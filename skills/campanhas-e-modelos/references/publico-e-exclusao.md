# Publico do disparo, exclusao e estimativa

Indice:

1. Como o publico e montado
2. Os 7 criterios, um a um
3. Os 2 modos de combinacao
4. Os 7 operadores do filtro de atributo
5. A exclusao (nao enviar para)
6. Estimar antes de criar
7. O teto diario da Meta (so WhatsApp Oficial)
8. Lista de planilha (o caminho que existe de verdade)
9. O que e descartado em silencio — a lista completa
10. O que acontece com quem entra no publico

---

## 1. Como o publico e montado

O publico e uma LISTA DE CRITERIOS. Cada item da lista e um criterio de um tipo. Dentro do mesmo
tipo vale uniao: quem atender a qualquer um dos itens daquele tipo entra. Entre tipos diferentes,
quem manda e o modo de combinacao (`sum` ou `all`).

O resultado final e uma lista de contatos. Depois disso a exclusao e descontada — excluir SEMPRE
ganha de incluir.

Formato geral de cada item:

```json
{ "type": "Label", "id": 12 }
```

---

## 2. Os 7 criterios, um a um

### `Label` — etiqueta de CONVERSA

```json
{ "type": "Label", "id": 12 }
```

Pega os contatos que tem alguma conversa marcada com aquela etiqueta. E a etiqueta que fica na
conversa, nao a que fica colada na ficha da pessoa. Os identificadores vem de `lionchat_labels_list`.

**Este e o unico criterio que funciona em campanha de SMS.**

### `ContactLabel` — etiqueta de CONTATO

```json
{ "type": "ContactLabel", "id": 12 }
```

Pega os contatos que tem aquela etiqueta colada na propria ficha. Mesma lista de etiquetas da conta,
mesmos identificadores. Se o identificador estiver errado, o criterio resolve ZERO contatos — e no
modo "quem estiver em todos" isso zera o publico inteiro.

### `Funnel` — etapa do funil (Kanban)

```json
{ "type": "Funnel", "id": 3, "stages": ["proposta", "negociacao"],
  "include_won": false, "include_lost": false }
```

Pega os contatos que tem cartao naquele funil, nas etapas escolhidas. Regras:

- **So 1 funil por campanha.** Se voce mandar dois, so o primeiro e lido.
- **`stages` e OBRIGATORIO** e leva os identificadores curtos das etapas (nao o nome que aparece na
  tela). Sem esse campo o criterio INTEIRO e ignorado em silencio.
- `include_won` e `include_lost` decidem se cartoes marcados como Ganho e Perdido entram. Os dois
  nascem falsos.
- Cartao com 2 ou mais conversas so entra se a conversa estiver com "receber automacoes" ligado.

Os identificadores de funil e de etapa vem de `lionchat_funnels_list`.

### `ConversationAttribute` — atributo personalizado da CONVERSA

```json
{ "type": "ConversationAttribute", "key": "origem_do_lead",
  "operator": "eq", "value": "instagram" }
```

### `ContactAttribute` — atributo personalizado da FICHA do contato

```json
{ "type": "ContactAttribute", "key": "plano", "operator": "contains", "value": "premium" }
```

### `CardAttribute` — atributo personalizado do CARTAO do funil

```json
{ "type": "CardAttribute", "key": "valor_orcamento", "operator": "present" }
```

Nos tres, `key` e a chave do atributo (nao o nome de exibicao). As chaves vem de
`lionchat_custom_attributes_list`. Respeita "receber automacoes" igual ao funil.

### `AgentTeam` — atendente responsavel ou time

```json
{ "type": "AgentTeam", "assignee_ids": [7, 9], "team_ids": [2] }
```

Pega os contatos cuja conversa esta com aquele atendente OU com aquele time, independente do status
da conversa. Pelo menos uma das duas listas precisa estar preenchida, senao o criterio e ignorado.
Os identificadores vem de `lionchat_agents_list` e `lionchat_teams_list`.

---

## 3. Os 2 modos de combinacao

| Valor | O que faz |
|---|---|
| `sum` (padrao) | Quem estiver em QUALQUER criterio entra. Uniao. |
| `all` | Quem estiver em TODOS os criterios preenchidos. Intersecao. |

**Atencao ao lugar do campo**: na estimativa ele vai solto no pedido, como `audience_mode`. Na
criacao da campanha ele vai DENTRO de `trigger_rules`:

```json
{ "trigger_rules": { "audience_mode": "all" } }
```

Mandar `audience_mode` solto na criacao nao da erro — simplesmente nao vale, e o disparo sai no modo
padrao (uniao). Em modo `all` isso pode multiplicar o tamanho do disparo.

---

## 4. Os 7 operadores do filtro de atributo

Valem nos tres criterios de atributo e tambem dentro da exclusao.

| Operador | Significa | Leva `value`? | Detalhe |
|---|---|---|---|
| `eq` | e exatamente igual a | sim | Comparacao EXATA. "Ativo" nao casa com "ativo" |
| `neq` | e diferente de | sim | INCLUI quem nunca preencheu o campo |
| `contains` | contem o texto | sim | Nao diferencia maiuscula de minuscula |
| `not_contains` | nao contem o texto | sim | INCLUI quem nunca preencheu o campo |
| `starts_with` | comeca com o texto | sim | Nao diferencia maiuscula de minuscula |
| `present` | esta preenchido | NAO | Qualquer valor serve |
| `blank` | esta vazio ou nao existe | NAO | |

Regras que doem quando se erra:

- **Operador desconhecido vira `eq` sem aviso.** Vocabulario de outra tela ("igual a", "esta
  preenchido" com os nomes do filtro de conversa) NAO existe aqui e cai em comparacao exata.
- **`neq` e `not_contains` incluem quem nao tem o atributo.** Isso e decisao de produto, nao
  defeito: "placa nao contem ABC" tem que pegar tambem quem nunca cadastrou placa. Pode dobrar o
  tamanho do disparo sem voce perceber — sempre estime e mostre o numero.
- Nos tres operadores de texto, `%` e `_` digitados valem como texto normal, nunca como curinga.
- **Criterio sem `key` e descartado.** Criterio com operador que exige valor e sem `value` tambem.
  Nos dois casos, em silencio.

---

## 5. A exclusao (nao enviar para)

Mesmo formato do publico, mesmos 7 tipos, mesmos operadores. Dentro da exclusao a combinacao e
SEMPRE uniao — o modo escolhido para o publico nao vale ali.

Onde ela vai:

- Na estimativa, solta no pedido, como `exclusion`.
- Na criacao da campanha, dentro de `trigger_rules`:

```json
{ "trigger_rules": { "audience_mode": "sum",
                     "exclusion": [{ "type": "ContactLabel", "id": 44 }] } }
```

**A receita mais util do produto**: no disparo de hoje, coloque uma etiqueta em quem receber (nas
acoes pos-envio). No disparo da semana que vem, exclua essa etiqueta. E assim que ninguem leva a
mesma mensagem duas vezes.

Cuidado em caixa oficial: a etiqueta de quem recebeu so e aplicada quando a Meta CONFIRMA a
entrega. Quem falhou ou foi barrado nao ganha a marca — de proposito, para o proximo disparo
alcancar essas pessoas.

---

## 6. Estimar antes de criar

Ferramenta: `lionchat_campaigns_estimate_audience`.

Mande os quatro juntos: `audience`, `audience_mode`, `exclusion` e `inbox_id`. Os tres primeiros
precisam ser IDENTICOS aos que vao para a criacao — estimar com um conjunto e criar com outro
entrega um publico diferente do numero que voce mostrou.

A resposta traz:

- `estimated_count` — quantas pessoas COM TELEFONE atendem aos criterios. Nao existe campo `count`.
- `limit_info` — o teto do dia da Meta: `tier`, `cap`, `used_24h`, `remaining`, `will_send_now` e
  `batches_preview` (o cronograma dia a dia do restante).

Duas leituras erradas que precisam ser evitadas:

- **Sem `inbox_id`, `tier`, `cap`, `used_24h` e `remaining` voltam NULOS**, `batches_preview` volta
  vazio e `will_send_now` vira a lista inteira. O silencio parece "nao ha limite" e voce promete
  5.000 disparos num numero que so fala com 250 pessoas novas por dia.
- **Nulo nao significa "sem limite".** Em caixa QR Code e SMS esses campos voltam nulos sempre,
  porque nao existe teto da Meta ali — o que existe e risco de bloqueio do numero.

A estimativa conta so quem tem TELEFONE na ficha. O disparo real e um pouco mais generoso: quem nao
tem telefone mas ja conversou naquela caixa tambem entra. Ou seja, a estimativa pode ficar um pouco
abaixo do disparo — nunca acima.

---

## 7. O teto diario da Meta (so WhatsApp Oficial)

A Meta libera uma faixa de contatos NOVOS por dia para cada numero:

| Faixa | Pessoas novas por dia |
|---|---|
| `TIER_250` | 250 |
| `TIER_1K` | 1.000 |
| `TIER_2K` | 2.000 |
| `TIER_10K` | 10.000 |
| `TIER_100K` | 100.000 |
| `TIER_UNLIMITED` | sem teto |

Quando a Meta nao responde, responde erro ou devolve uma faixa desconhecida, o sistema segue SEM
teto e manda para todo mundo — nunca trava um disparo por falha da Meta.

O "quanto ja foi usado hoje" e uma conta NOSSA: contatos distintos que receberam mensagem de
campanha aceita ou entregue nas ultimas 24 horas naquela caixa. Modelo enviado fora de campanha nao
entra nessa conta, entao o numero pode ser otimista.

Quando a lista e maior que o que sobra hoje, ha duas saidas e o cliente precisa escolher:

| Escolha | O que grava | O que acontece |
|---|---|---|
| Mandar ate o limite e parar | nada (e o padrao) | Quem ficou de fora nao recebe nunca |
| Dividir em lotes diarios | `trigger_rules.over_limit_mode = "batches"` | O excedente vira uma fila; um lote por dia ate acabar, reconsultando o teto a cada dia |

No modo de lotes existe ainda uma reposicao automatica: quando um lote termina e ainda sobra folga
no dia, o sistema manda a proxima leva sozinho. E por isso que o cliente pode ver o disparo
continuar depois de aparentemente ter batido o limite. Isso NAO acontece no modo "ate o limite e
parar".

---

## 8. Lista de planilha (o caminho que existe de verdade)

Subir CSV ou Excel na criacao da campanha existe no painel, depende de uma chave que nasce
desligada na conta e de permissao dupla — e o conector NAO envia arquivo. Nao prometa isso.

O caminho equivalente, que funciona sempre:

1. O cliente importa os contatos no painel (Contatos, botao de importar), aplicando uma etiqueta so
   daquela lista, por exemplo `lista-blackfriday-2026`.
2. Voce pega o identificador dela com `lionchat_labels_list`.
3. Voce monta o publico com `{ "type": "ContactLabel", "id": N }`.

Vantagem colateral: a etiqueta fica na ficha e serve de exclusao nos disparos seguintes.

Ao orientar a importacao, avise: **o telefone precisa vir com o codigo do pais** (por exemplo
`5511988887777`; o sinal de mais e opcional). Linha sem codigo de pais vira erro e o contato nao e
criado.

---

## 9. O que e descartado em silencio — a lista completa

Confira estes pontos antes de mostrar qualquer numero ao cliente:

| Situacao | Consequencia |
|---|---|
| Publico vazio | Campanha criada, nao manda para ninguem, e nao aceita mais edicao |
| `Funnel` sem `stages` | O criterio inteiro some da conta |
| Criterio de atributo sem `key` | O criterio some |
| Criterio de atributo sem `value` (fora de `present` e `blank`) | O criterio some |
| `AgentTeam` com as duas listas vazias | O criterio some |
| Operador escrito com outro vocabulario | Vira comparacao exata |
| `audience_mode` mandado solto na criacao | Ignorado; vale o padrao (uniao) |
| Publico de SMS com qualquer criterio que nao seja `Label` | Resolve zero pessoas |

---

## 10. O que acontece com quem entra no publico

Vale dizer isso ao cliente ANTES de ele confirmar, porque muda a rotina do time:

- **O disparo abre (ou reaproveita) uma conversa por pessoa.** Uma lista de 800 pessoas vira ate 800
  conversas na caixa.
- Em caixa oficial as conversas sao criadas antes do envio. Contato sem telefone e sem historico
  naquela caixa (por exemplo alguem que so falou por Instagram) e pulado.
- Na Campanha de Fluxo, conversa ja encerrada e REABERTA em vez de virar uma nova.
- As acoes pos-envio (etiqueta, atendente, prioridade, atributo) sao aplicadas por pessoa:
  - em caixa oficial, so quando a Meta confirma a entrega;
  - em caixa QR Code, ao fim de cada rodada de envio.
