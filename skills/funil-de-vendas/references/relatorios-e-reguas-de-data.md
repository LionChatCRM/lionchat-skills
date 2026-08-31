# Relatorios do funil e as reguas de data

## Indice

1. Por que dois relatorios do mesmo funil mostram numeros diferentes
2. As quatro reguas de data
3. Qual tela usa qual regua por padrao
4. Como fazer dois relatorios baterem
5. O relatorio agregado do funil
6. O relatorio de uma etapa
7. Contagem simples por etapa
8. Tempo do cartao e tempo por etapa
9. "Esta na etapa" e diferente de "entrou na etapa"
10. Ate onde o historico vai
11. Como responder "quanto eu vendi no mes"
12. Perguntas comuns e o caminho de cada uma

---

## 1. Por que dois relatorios do mesmo funil mostram numeros diferentes

Porque eles respondem perguntas diferentes, e cada um usa uma regua de data diferente POR PADRAO. Isso e intencional, nao defeito. Quando o cliente disser "os numeros nao batem", a primeira coisa a conferir e a regua de cada tela - nao o dado.

## 2. As quatro reguas de data

A regua decide qual data do cartao o periodo escolhido considera.

| Regua | Considera | Boa para |
|---|---|---|
| `any` | o cartao NASCEU no periodo **ou** FOI FECHADO no periodo | visao geral: e uma rede larga, feita para caber varias medidas de uma vez |
| `created` | quando o cartao nasceu | "quantos leads novos entraram em agosto" |
| `moved` | quando o cartao entrou na etapa em que ele esta agora | "quantos cartoes chegaram em Proposta este mes" |
| `closed` | quando o cartao virou Ganho ou Perdido | "quanto eu vendi no mes" |

## 3. Qual tela usa qual regua por padrao

| Tela | Padrao | Por que |
|---|---|---|
| relatorio agregado do funil (`lionchat_kanban_items_list_3`) | `any` | ele devolve varias medidas de uma vez e cada uma torna a filtrar por dentro. A rede de fora precisa ser larga o suficiente para conter as linhas de todas |
| relatorio de uma etapa (`lionchat_funnels_stage_report`) | `moved` | ele responde uma pergunta so: quantos cartoes ENTRARAM nesta etapa no periodo |

**Numero medido para dimensionar o efeito:** trocar a regua do relatorio agregado de `any` para `moved` num periodo de 30 dias fez sumir 28 de 162 negocios ganhos, o equivalente a R$ 161.094 - eram negocios fechados no mes cujo cartao tinha nascido antes e nao se moveu.

## 4. Como fazer dois relatorios baterem

As duas telas oferecem as MESMAS quatro reguas. Para comparar, escolha a mesma regua nas duas e o mesmo periodo. Nao existe um padrao unico possivel: qualquer padrao unico faria uma das duas telas mentir.

O relatorio agregado devolve tambem a regua que ele efetivamente aplicou - confira esse campo quando o numero surpreender, porque valor invalido cai no padrao em vez de dar erro.

## 5. O relatorio agregado do funil

`lionchat_kanban_items_list_3`. A resposta e um conjunto de medidas, nao uma lista de cartoes.

O que ele traz: receita ganha no periodo, quantidade e valor por etapa, previsao ponderada, melhores negocios, cartoes parados e as metas do funil.

Filtros: periodo, vendedores e time.

Cuidados:

- Para o periodo, **prefira a data no formato `AAAA-MM-DD`**: escrita assim, a data final cobre o DIA
  INTEIRO. O formato numerico (segundos) tambem e aceito, mas ali a data final vale ao pe da letra -
  meia-noite corta o ultimo dia fora. Valor que o sistema nao entende cai no padrao (ultimos 30 dias)
  em vez de dar erro.
- Em instalacao mais antiga, a receita ganha SUBCONTA: negocio ganho no periodo cujo cartao nasceu antes do periodo fica de fora. Se os numeros parecerem baixos numa conta antiga, avise que o valor pode estar subestimado em funil de ciclo longo.
- Para listar os cartoes em si, esta ferramenta nao serve: use o filtro.

## 6. O relatorio de uma etapa

`lionchat_funnels_stage_report`. Responde por UMA coluna: total de cartoes, faixa e soma de valores, ganhos e perdidos, distribuicao por prioridade e ranking de vendedores daquela etapa.

- **Exige a CHAVE da etapa, nunca o nome visivel.** Pegue em `lionchat_funnels_show`.
- Data invalida devolve erro, e nao a etapa inteira em silencio - isso e bom.
- No filtro por responsavel, o valor `-1` significa cartoes SEM responsavel.
- Os numeros sao calculados no servidor. Nunca some cartoes na sua cabeca a partir de uma lista parcial: e exatamente o defeito que essa ferramenta veio corrigir (a tela antiga somava so os cartoes ja carregados e escrevia "100 cartoes" numa etapa de 438).

## 7. Contagem simples por etapa

`lionchat_funnels_list_1` da a contagem e a soma de valor por etapa - e o que aparece no cabecalho de cada coluna. `lionchat_funnels_open_counts` da quantos cartoes ABERTOS cada funil tem, sem baixar cartao nenhum (ganhos e perdidos ficam de fora, e funil arquivado nao aparece).

## 8. Tempo do cartao e tempo por etapa

`lionchat_kanban_items_list_1` traz o tempo total de UM cartao. `lionchat_kanban_items_list_2` traz quanto tempo aquele cartao passou em cada etapa. Sao relatorios de um cartao so - nao servem para media do funil.

## 9. "Esta na etapa" e diferente de "entrou na etapa"

E a distincao que mais engana quem monta painel proprio:

- **Esta na etapa** conta os cartoes que estao naquela coluna AGORA. Cartao que passou e ja saiu nao aparece.
- **Entrou na etapa** le o historico de movimentos e conta quem passou por ali no periodo, tenha ficado ou nao.

Medido numa conta real em 30 dias: "esta na etapa" mostrava 55 e "entrou na etapa" mostrava 69 - 25% dos cartoes passaram e sairam, ficando invisiveis na primeira leitura.

Quando o cliente perguntar "quantos agendamentos eu fiz este mes", quase sempre a pergunta e a segunda. Quando ele perguntar "quantos estao esperando proposta", e a primeira.

O painel de relatorio personalizado tem blocos que leem o funil: cartoes por etapa (agora), entradas na etapa (no periodo), passagem entre etapas e origem do lead.

## 10. Ate onde o historico vai

O registro de movimentos entre etapas - a base da taxa de passagem e do "entrou na etapa" - **nasceu em 12 de julho de 2026 e guarda 365 dias**. Periodo que alcance datas anteriores a esses dois marcos devolve MENOS do que aconteceu, em silencio. Nao e defeito: e a fronteira do dado. Avise o cliente quando ele pedir comparacao com um periodo mais antigo.

Cartao ja excluido some da contagem de entradas, porque o vinculo do evento com o cartao e anulado quando o cartao morre.

## 11. Como responder "quanto eu vendi no mes"

O caminho seguro:

1. `lionchat_kanban_items_filter` com o funil, o estado `won` e a regua de fechamento (data de inicio e fim do desfecho).
2. Some o valor dos cartoes devolvidos.

**Nunca some pela data de atualizacao do cartao nem pela data de criacao** - o erro chega a semanas. A data de criacao responde "quando o lead entrou", nao "quando a venda fechou".

Um limite honesto: cartoes antigos e cartoes vindos de planilha podem nao ter a data do desfecho gravada. O sistema tem um remendo que usa a ultima alteracao nesses casos, mas ele e aproximado. Se o cliente cobrar precisao num periodo antigo, diga isso.

E lembre: **reabrir um cartao Ganho para "aberto" APAGA a data do ganho, quem marcou e a quem foi creditado.** Aquele ganho some dos relatorios. Marcar Ganho de novo um cartao que ja estava Ganho nao empurra a data para frente - o carimbo so e gravado quando o estado MUDA.

## 12. Perguntas comuns e o caminho de cada uma

| A pessoa pergunta | Caminho |
|---|---|
| "quanto vendi no mes" | filtro com estado ganho + regua de fechamento |
| "quantos leads novos entraram" | filtro com regua de criacao, ou o relatorio agregado |
| "onde meus leads travam" | contagem por etapa + "entrou na etapa" no mesmo periodo: onde entrou muito e ficou muito, e o gargalo |
| "quem vendeu mais" | relatorio agregado filtrando por vendedor, ou o ranking dentro do relatorio de uma etapa |
| "por que estou perdendo" | filtro com estado perdido, e leia os motivos de perda dos cartoes |
| "quanto tempo demora minha venda" | o relatorio agregado traz a media; para um cartao especifico, o tempo por etapa |
| "quais negocios estao parados" | a lista de cartoes parados dentro do relatorio agregado |
| "quanto tem em aberto" | contagem por etapa (a soma de valor de cada coluna) ou os cartoes abertos por funil |
| "quantos passaram por Proposta" | "entrou na etapa" no periodo, nunca "esta na etapa" |
