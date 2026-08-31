# Gatilhos e condicoes da automacao

Indice:

1. Os 8 gatilhos que existem
2. Detalhe de cada gatilho
3. O recorte do gatilho "Acao na conversa"
4. Como se escreve uma condicao
5. O conector E/OU — a regra que mais mata automacao
6. Catalogo de condicoes e os operadores que cada uma aceita
7. Condicoes com atributo personalizado
8. Condicoes nos gatilhos de card (motor diferente)
9. O que NAO existe como condicao

---

## 1. Os 8 gatilhos que existem

Sao estes e mais nenhum.

**PERIGO:** o nome do gatilho **nao e conferido na hora de salvar**. Uma regra com um nome inventado
(ou copiado de outro lugar) e criada com sucesso, aparece na lista e **nunca dispara**, sem erro
nenhum. Nao existe outra pista alem do historico vazio. Escreva exatamente um destes oito.

| Nome na tela | Nome tecnico (o que vai na ferramenta) | Quando dispara |
|---|---|---|
| Conversa Criada | `conversation_created` | Nasceu uma conversa nova |
| Acao na conversa | `conversation_updated` | Mudou alguma coisa numa conversa que ja existia |
| Conversa Resolvida | `conversation_resolved` | A conversa foi marcada como resolvida |
| Conversa Aberta | `conversation_opened` | A conversa voltou para o status aberto |
| Mensagem Criada | `message_created` | Chegou ou saiu uma mensagem |
| Webhook | `webhook` | Um sistema de fora avisou (pagamento, lead, integracao) |
| Card Kanban Criado | `kanban_item_created` | Nasceu um card num funil |
| Card Kanban Movido | `kanban_item_stage_changed` | Um card mudou de etapa |

**Trocar o gatilho de uma regra que ja existe nao da pelas ferramentas** — so no painel, e a tela
zera as condicoes e as acoes ao trocar.

---

## 2. Detalhe de cada gatilho

### Conversa Criada
Serve para triagem de primeiro contato: atribuir equipe, criar card, ligar o AI Agente.

Condicoes que a tela oferece: qualquer conversa, conteudo, status, tipo de conversa, idioma do
navegador, assunto do e-mail, pais, telefone, link de origem, e-mail, caixa de entrada, idioma da
conversa, prioridade, etiquetas, agente atribuido, equipe, nome, identificador, empresa, mais os
atributos personalizados de contato e de conversa.

**Armadilha:** se a regra usar o filtro de CONTEUDO, ela nao e avaliada no momento em que a conversa
nasce (a mensagem ainda nao existe). Ela e avaliada quando chega a **primeira mensagem recebida**.
Conversa aberta pelo atendente, por campanha, por integracao ou pelas ferramentas — sem mensagem do
cliente — nunca dispara essa variacao. Sem filtro de conteudo, o gatilho pega todas normalmente.

### Acao na conversa
E o gatilho de "quando puserem a etiqueta X", "quando mudar de status", "quando atribuirem a
fulano".

So dispara quando muda um destes: equipe, agente responsavel, status, adiamento, atributo
personalizado, etiquetas, "aguardando desde", carimbo de primeira resposta, prioridade, AI Agente da
conversa, ou o idioma detectado.

Condicoes que a tela oferece: as mesmas de Conversa Criada, **menos** "qualquer conversa" e
"conteudo".

**Armadilha:** pela tela e obrigatorio escolher o recorte (secao 3). Pelas ferramentas esse campo
nao existe — a regra nasce **sem recorte**, disparando em qualquer mudanca, inclusive em campos que
nem tem recorte proprio (carimbo de primeira resposta, adiamento, "aguardando desde"). Uma regra que
devia rodar so ao adicionar etiqueta passa a rodar dezenas de vezes por conversa. Avise o cliente e
mande ele ajustar na tela.

### Conversa Resolvida
Pos-atendimento: pesquisa de satisfacao, card de Ganho, e-mail de resumo.

Condicoes que a tela oferece: qualquer conversa, tipo de conversa, idioma do navegador, e-mail,
assunto do e-mail, pais, link de origem, agente atribuido, telefone, equipe, caixa de entrada,
idioma da conversa, prioridade, etiquetas. A tela nao oferece atributo personalizado nem
nome/identificador/empresa aqui (o motor aceita, mas so por fora da tela).

### Conversa Aberta
"O cliente voltou a falar depois de encerrado."

As mesmas condicoes de Conversa Criada, **menos** "status".

O filtro de CONTEUDO aqui olha a ultima mensagem recebida ate o instante da reabertura. Se nunca
houve mensagem recebida, a regra e pulada.

### Mensagem Criada
E o gatilho de palavra-chave. **Dispara tanto para mensagem recebida quanto para mensagem enviada**
— use a condicao "Tipo da Mensagem" para separar, senao a regra tambem reage ao que voce responde.

Condicoes que a tela oferece: tipo da mensagem, tipo de conversa, conteudo, e-mail, caixa de
entrada, status, agente atribuido, equipe, prioridade, idioma da conversa, telefone, etiquetas,
nome, identificador, empresa, mais atributos personalizados.

Nao dispara para: pilulas de atividade (aqueles avisos cinzas no meio da conversa), respostas
automaticas de e-mail, nem mensagens geradas por outra automacao ou por um Flow.

### Webhook
Um sistema de fora avisou: gateway de pagamento, Webhook Universal, Meta Lead Ads, e-Clinica,
Topsend. Tambem e por aqui que entra o lembrete de agendamento da e-Clinica, e so nesse caso
existem as variaveis de agendamento no texto.

- **Exige a Caixa de Envio preenchida.** Sem ela a execucao falha.
- Em caixa WhatsApp, contato sem telefone e recusado.
- **AS CONDICOES NAO SAO AVALIADAS AQUI.** A tela nem mostra a secao de condicoes. Quem separa
  "compra aprovada" de "carrinho abandonado" e o mapeamento de eventos na tela da integracao, nao a
  automacao. Se voce gravar condicao por fora, ela e ignorada.

### Card Kanban Criado / Card Kanban Movido
Quem executa e um motor **diferente**, que so conhece 10 acoes e 3 condicoes. Ver secao 8 aqui e a
lista de acoes em `acoes.md`.

- Card Kanban Criado: unica condicao que a tela oferece e o funil (com a etapa embutida), e so com
  "igual a".
- Card Kanban Movido: funil (com a etapa de destino) e prioridade.

---

## 3. O recorte do gatilho "Acao na conversa"

Sao 9 opcoes. Na tela o campo se chama "Qual acao dispara a regra".

| Opcao na tela | Nome tecnico |
|---|---|
| Etiqueta adicionada | `label_added` |
| Etiqueta removida | `label_removed` |
| Status alterado | `status_changed` |
| Prioridade alterada | `priority_changed` |
| Agente atribuido | `agent_assigned` |
| Equipe atribuida | `team_assigned` |
| Atributo personalizado alterado | `custom_attribute_changed` |
| Idioma alterado | `language_changed` |
| Agente de IA atribuido | `ai_agent_assigned` |

Vazio significa "todas as mudancas". **As ferramentas nao tem esse campo** — quem cria por elas
sempre nasce vazio.

---

## 4. Como se escreve uma condicao

Cada condicao tem estas partes:

- `attribute_key` — que campo olhar (catalogo na secao 6)
- `filter_operator` — como comparar. Cada campo aceita so um subconjunto; operador fora do
  subconjunto **invalida a regra inteira** e, na segunda falha, o sistema a desliga sozinho.
- `values` — o valor, **sempre uma lista**, mesmo com um item so. So "preenchido" e "vazio"
  dispensam. Em "contem", cada item da lista e procurado separadamente e basta um bater — mas o
  texto **nao** e quebrado por virgula: `preco, orcamento` num item so vira uma frase unica. E se o
  cliente reabrir a regra na tela e salvar, a lista de varios itens vira uma frase unica com
  virgulas; por isso, para varias palavras, prefira **uma condicao por palavra ligadas por OU**.
  Em etiqueta, so o primeiro valor da lista e usado.
- `query_operator` — o conector com a condicao SEGUINTE (secao 5)
- `custom_attribute_type` — so quando o campo e atributo personalizado: `conversation_attribute` ou
  `contact_attribute`

Para "sempre, sem filtro", use uma unica condicao com `any_conversation` (na tela: "Qualquer
conversa"). Lista de condicoes vazia nunca e o certo.

Exemplo de uma condicao so:

```json
[{"attribute_key": "any_conversation", "filter_operator": "equal_to",
  "values": ["true"], "query_operator": null}]
```

Exemplo de duas condicoes ligadas por E:

```json
[{"attribute_key": "inbox_id", "filter_operator": "equal_to",
  "values": [12], "query_operator": "AND"},
 {"attribute_key": "content", "filter_operator": "contains",
  "values": ["orcamento"], "query_operator": null}]
```

---

## 5. O conector E/OU — a regra que mais mata automacao

O conector liga a condicao com a **seguinte**. Na ultima nao ha proxima, entao a ultima condicao tem
que ter o conector **vazio** (`null`). Condicao unica tambem vai vazia.

- O sistema confere se o valor do conector e "AND" ou "OU", mas **nao confere a posicao**.
- Conector sobrando na ultima condicao salva com sucesso e aparece certinho na tela.
- Isso ja nasceu morto 8 regras em 3 contas em julho de 2026, e uma conta inteira ficou com as 6
  regras de etiquetagem paradas.
- Hoje o motor limpa o conector solto antes de perguntar ao banco, mas a tela de edicao continua
  mostrando a regra estranha e **este e o formato correto**: nao deixe conector na ultima.
- O contrario tambem e erro: com duas ou mais condicoes, **so uma** pode ter o conector vazio. Duas
  vazias e recusado na hora de salvar.

---

## 6. Catalogo de condicoes e os operadores que cada uma aceita

Operadores em portugues: `equal_to` = igual a, `not_equal_to` = diferente de, `contains` = contem,
`does_not_contain` = nao contem, `is_present` = preenchido, `is_not_present` = vazio,
`is_greater_than` = maior que, `is_less_than` = menor que, `starts_with` = comeca com.

### Campos da conversa

| Campo | Nome tecnico | Operadores aceitos |
|---|---|---|
| Qualquer conversa | `any_conversation` | (dispensa — pega tudo) |
| Status | `status` | igual a, diferente de |
| Prioridade | `priority` | igual a, diferente de |
| Caixa de Entrada | `inbox_id` | igual a, diferente de |
| Idioma da conversa | `conversation_language` | igual a, diferente de |
| Idioma do navegador | `browser_language` | igual a, diferente de |
| Tipo de conversa (individual/grupo) | `chat_type` | igual a, diferente de |
| Agente atribuido | `assignee_id` | igual a, diferente de, preenchido, vazio |
| Equipe | `team_id` | igual a, diferente de, preenchido, vazio |
| Etiquetas | `labels` | igual a, diferente de, preenchido, vazio |
| Link de origem | `referer` | igual a, diferente de, contem, nao contem |
| Assunto do e-mail | `mail_subject` | igual a, diferente de, contem, nao contem |

### Campos da mensagem (so nos gatilhos que tem mensagem)

| Campo | Nome tecnico | Operadores aceitos |
|---|---|---|
| Conteudo | `content` | igual a, diferente de, contem, nao contem |
| Tipo da Mensagem | `message_type` | igual a, diferente de (valores: `incoming`, `outgoing`) |

### Campos do contato

| Campo | Nome tecnico | Operadores aceitos |
|---|---|---|
| E-mail | `email` | igual a, diferente de, contem, nao contem |
| Nome | `name` | igual a, diferente de, contem, nao contem |
| Identificador | `identifier` | igual a, diferente de, contem, nao contem |
| Empresa | `company` | igual a, diferente de, contem, nao contem |
| Pais | `country_code` | igual a, diferente de |
| Telefone | `phone_number` | igual a, diferente de, contem, nao contem, comeca com |

### Duas advertencias de comportamento

- **Prioridade "Nenhuma" nao funciona como condicao.** A pergunta ao banco vira "prioridade dentro de
  (nada)" e nunca e verdadeira. Para pegar quem nao tem prioridade, use "diferente de" com as
  prioridades que existem, ou simplesmente nao filtre por prioridade. Como ACAO, "Nenhuma" funciona
  normal (limpa a prioridade).
- **Etiqueta "diferente de" mudou de significado em 24/08/2026.** Hoje ela quer dizer "a conversa NAO
  TEM essa etiqueta" — e por isso passou a disparar TAMBEM em conversas sem etiqueta nenhuma. Antes
  ela errava dos dois lados. Regra antiga com essa condicao mudou de comportamento na pratica:
  reconfira com o cliente antes de mexer nela, e avise que a regra agora alcanca mais conversas.

---

## 7. Condicoes com atributo personalizado

O `attribute_key` e a chave do atributo. Ele precisa **existir na conta antes** — confira com
`lionchat_custom_attributes_list`. Se nao existir, a regra e recusada com uma mensagem dizendo que a
condicao nao e suportada.

E preciso informar tambem se o atributo e da conversa (`conversation_attribute`) ou do contato
(`contact_attribute`).

Operadores por tipo de atributo:

| Tipo do atributo | Operadores |
|---|---|
| Texto, Link | igual a, diferente de, contem, nao contem, preenchido, vazio |
| Numero, Moeda, Porcentagem, Data, Data e Hora, Hora | igual a, diferente de, preenchido, vazio, maior que, menor que |
| Lista, Sim/Nao | igual a, diferente de, preenchido, vazio |

**Perigo real:** se o atributo existia e foi apagado depois, a regra passa a falhar na validacao e,
na **segunda** falha, o sistema **desliga a regra sozinho** e manda e-mail aos administradores. O
cliente ve a chave desligada e nao entende por que. Editar as condicoes zera esse contador.

Vale o mesmo para chaves que a regra aceita gravar mas o motor nao sabe traduzir: **funil e etapa do
funil num gatilho de CONVERSA**, e a politica de SLA como condicao. Nao use nenhuma das tres.

---

## 8. Condicoes nos gatilhos de card (motor diferente)

Nos gatilhos Card Kanban Criado e Card Kanban Movido, o motor so entende tres campos:
`funnel_id` (funil), `funnel_stage` (etapa) e `priority` (prioridade). Operadores: igual a, diferente
de, preenchido, vazio.

**Condicao que ele nao entende e APROVADA, nao ignorada.** Isso e o oposto do motor de conversa. Na
pratica:

- Automacao de card com condicao de etiqueta ou status = a regra dispara em **todos** os cards de
  **todos** os funis. Sintoma: "a automacao dispara demais", "mexeu num card que nao era".
- Automacao de conversa com condicao que o motor nao reconhece = a regra **nunca** dispara. Sintoma:
  "salvei e nao acontece nada".

Sem nenhuma condicao, a automacao de card tambem dispara sempre.

---

## 9. O que NAO existe como condicao

- **Horario de atendimento / expediente / dia da semana.** Nao existe. "Ligar o AI Agente fora do
  horario" nao e montavel por automacao — o caminho e Flows (que tem condicao de hora) ou a mensagem
  de ausencia da caixa. Nao invente uma chave: campo que o sistema nao conhece e recusado na hora de
  salvar, com a mensagem de que a condicao nao e suportada.
- **Tempo desde a ultima mensagem / conversa parada ha X horas.** Nao existe. E Flow.
- **Quantidade de mensagens da conversa.** Nao existe.
- **Cidade do contato** existe no motor mas nao aparece na tela de automacao — nao use.
