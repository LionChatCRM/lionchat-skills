# Registrar vendas: Importar vendas, lancamento retroativo e co-participante

Tres recursos para a mesma dor: o relatorio de vendas so fica certo se cada ganho estiver no cartao certo,
com o valor certo e NO MES em que a venda aconteceu. Leia antes de mexer em qualquer ganho que nao seja
"fechou agora e estou marcando agora".

## Indice

1. Qual dos tres usar
2. Importar vendas (a planilha do financeiro)
3. Formato de cada linha
4. O que acontece com cada linha
5. O que dizer ao dono antes de aplicar
6. Lancamento retroativo (um ganho com data passada)
7. Co-participante da venda (o SDR)

---

## 1. Qual dos tres usar

| A pessoa diz | Caminho |
|---|---|
| "o vendedor esqueceu de marcar as vendas", "tenho a planilha do financeiro", "o valor do cartao nao bate com o que o cliente pagou" | **Importar vendas** (secao 2) |
| "essa venda fechou dia 5 do mes passado, marca ela" (um ou poucos cartoes) | **lancamento retroativo** (secao 6) |
| "o SDR que trouxe o lead tambem tem que aparecer" | **co-participante** (secao 7) |

## 2. Importar vendas (a planilha do financeiro)

Confere uma planilha de vendas contra os cartoes de UM funil e, depois do sim do dono, grava o valor e marca
o GANHO com a DATA DA VENDA - nao com a data de hoje. E isso que faz o faturamento do relatorio contar a venda
no mes certo.

| Ferramenta | Para que |
|---|---|
| `lionchat_kanban_sales_reconciliation_preview` | CONFERE e NAO GRAVA NADA. Devolve, linha a linha, o que aconteceria. **Sempre primeiro** |
| `lionchat_kanban_sales_reconciliation_apply` | aplica em segundo plano, com a MESMA regra da conferencia. Devolve o codigo da rodada |
| `lionchat_kanban_sales_reconciliation_status` | acompanha ate `completed` ou `failed`. O registro vive 7 dias |

Os tres pedem `funnel_id` (o funil onde os cartoes vivem), `inbox_id` (a caixa usada para criar contato e
conversa de quem ainda nao tem cartao) e `rows` (as linhas).

- **O conector nao sobe arquivo**: as linhas vao em `rows`, uma lista de objetos. Se o dono anexar a planilha
  na conversa, leia o arquivo e monte as linhas voce mesmo - nao peca para ele converter.
- Quem prefere a tela usa Kanban > botao azul **+** > **Importar vendas**, que le o arquivo. O mesmo menu tem
  **Baixar modelo de vendas**, a planilha de exemplo.
- **Quem pode**: administrador ou cargo com a permissao de exportar e importar do Kanban
  (`kanban_export_import`). Gerenciar o Kanban NAO basta.

## 3. Formato de cada linha

| Campo | Obrigatorio | Como mandar |
|---|---|---|
| `phone` | sim | Com o codigo do pais: `5511999887766`. Sem o 55 a linha e recusada (nao e completado). Celular antigo sem o nono digito e encontrado do mesmo jeito |
| `value` | sim, menos quando vier `products` | `1500`, `"1500,00"` ou `"R$ 1.500,00"` |
| `sale_date` | sim | `"01/08/2026"` ou `"2026-08-01"`: o dia em que a venda ACONTECEU. Sem data a linha e recusada - nunca vira "hoje" |
| `name` | nao | nome do cliente (vira o titulo do cartao novo) |
| `email` | nao | |
| `agent` | nao | nome ou e-mail do vendedor. So entra em cartao SEM responsavel - nunca troca quem ja esta la |
| `order_id` | nao | numero do pedido, guardado no cartao |
| `co_participant` | nao | nome ou e-mail de quem ajudou na venda (secao 7) |
| `products` | nao | TEXTO no formato `"Plano Pro:1200; Instalacao:300"`, ate 20 por linha. Lista ou objeto e erro da linha |

Sobre os produtos (desde 30/10/2026): o nome e casado com o catalogo de ofertas da conta (sem diferenca de
maiuscula e acento); o que nao esta no catalogo entra SO naquele cartao, como produto avulso, e a conferencia
avisa. Com `products`, os produtos do cartao sao TROCADOS pelos da planilha e o valor vira a SOMA - se `value`
tambem vier e nao bater, valem os produtos (com aviso). Sem `products`, cartao com um produto recebe o valor
nesse produto e cartao com dois ou mais produtos NAO muda de valor (com aviso).

## 4. O que acontece com cada linha

A regra e por PESSOA dentro do funil:

| Situacao no funil | O que acontece | Na conferencia |
|---|---|---|
| tem cartao ABERTO (o mais antigo, se houver varios) | grava valor e produtos e marca Ganho com a data da venda | `completa` |
| tem cartao GANHO com o mesmo valor e a mesma data | e a mesma venda: nada muda | `ja_ganho` |
| tem cartao GANHO com outro valor ou outra data | **cartao ganho NUNCA e editado**: e uma venda nova e nasce OUTRO cartao ja ganho (so com `criar_sem_card: true`) | `nova_venda` |
| so tem cartao PERDIDO | nao e tocado - vai para conferencia humana | `conferir` |
| nao tem cartao nenhum | cria um cartao ja ganho (so com `criar_sem_card: true`; `stage` escolhe a etapa onde ele nasce, padrao a primeira) | `cria` |
| telefone sem DDI, valor ilegivel, data no futuro, produtos em formato errado | erro da linha, com o motivo | `erro` |

No andamento (`_status`): `sem_mudanca` e a linha ja registrada; `criados` inclui a venda nova de quem ja
tinha cartao ganho; `pulados` e a linha que precisaria criar cartao com `criar_sem_card` desligado.

**Rodar a mesma planilha duas vezes e seguro**: a segunda rodada da "ja registrada" e nada e gravado de novo.

## 5. O que dizer ao dono antes de aplicar

Mostre o resumo da conferencia (quantas completam, quantas ja estavam, quantas viram venda nova, quantas
criam cartao, quantas vao para conferencia e quantas deram erro) e diga, se valer:

- **`criar_sem_card: true` abre uma CONVERSA nova por cartao criado** (inclusive na venda nova de quem ja
  ganhou), e conversa nova acorda o que a conta tiver em "Conversa criada" (automacao, fluxo, notificacao).
  Numa planilha grande isso e muito barulho - avise antes de ligar.
- **As automacoes de Ganho do funil rodam normalmente** (por exemplo, duplicar o cartao para o pos-venda). A
  conferencia diz quantas vao rodar e quantos cartoes de pos-venda vao nascer.
- **Conversao de anuncio**: a planilha nao dispara conversao por conta propria; quem dispara e o caminho
  normal do ganho, e so nos funis com o evento de Ganho configurado. Rodar de novo nao reenvia (o
  identificador do evento de ganho e estavel).

Termine com a pergunta: **"Confirma que posso aplicar essa planilha? (s/n)"**. So com um sim explicito.

Se o dono depois disser "a venda entrou no mes errado", confira a data em `sale_date` da linha: e ela que
manda.

## 6. Lancamento retroativo (um ganho com data passada)

Para o caso do dia a dia: o negocio fechou semanas atras e so agora alguem registra. Sem isso, o relatorio
conta a venda no mes em que o registro foi feito.

Pelo conector e a chave `retroativo_em` dentro de `item_details`, no `lionchat_kanban_items_update`:

```
{ "item_details": { "status": "won", "win_reason": "ganho-indicacao", "reason": "Indicacao",
                    "retroativo_em": "2026-08-05" } }
```

Na tela, ao marcar Ganho ou Perdido aparece a chavinha **"Lancamento retroativo"** com um campo de data.

| Regra | Detalhe |
|---|---|
| Quando vale | SO junto de um Ganho ou Perdido que MUDA o estado. Cartao que ja esta Ganho ignora a data |
| O que faz | decide a data do fechamento que TODOS os relatorios contam |
| O que nao faz | nao fica gravada no cartao, e nao muda a data das conversoes de anuncio (elas saem com a data de hoje) |
| Data futura ou ilegivel | recusada com 422 |
| Data anterior a criacao do cartao | permitida |
| Onde funciona | so no `lionchat_kanban_items_update`. No criar cartao a chave e descartada - venda antiga ja fechada e o "Importar vendas" |

**Corrigir uma data lancada errado**: marcar Ganho de novo por cima nao muda nada. Mande `status: "open"`
(apaga o carimbo do ganho, o co-participante e quem foi creditado) e depois o Ganho ou Perdido com o
`retroativo_em` certo. Avise o dono desse efeito antes.

## 7. Co-participante da venda (o SDR)

No ganho, alem do vendedor (`attributed_to`, "Atribuir ganho a"), da para marcar UM co-participante - por
exemplo o SDR que trouxe a venda. A venda conta INTEIRA para o vendedor; o co-participante ganha numeros
proprios no relatorio (ver `relatorios-e-reguas-de-data.md`, secao 13). Ser co-participante NAO da acesso ao
cartao nem poe a pessoa como responsavel.

Como gravar:

- Junto com o ganho: `lionchat_kanban_items_create_2` com `status: "won"` e `co_attributed_to: <id da pessoa>`.
- Num cartao JA ganho: `lionchat_kanban_items_update` com `item_details.co_attributed_to`. Para tirar, `null`.
- Pela planilha: a coluna `co_participant` do "Importar vendas" (nome ou e-mail).
- Mande SO o numero da pessoa. O nome o proprio cartao preenche - nunca envie.

Recusas (422, com a mensagem): valor que nao e numero (objeto ou lista tambem), pessoa que nao e membro da
conta, o proprio vendedor do cartao, pessoa sem acesso ao funil.

O que mais saber:

- Reabrir ou marcar como Perdido APAGA o co-participante (ele so existe em cartao Ganho).
- **"Marcar como ganho" por fluxo, automacao ou macro NAO marca co-participante.** Ele entra depois, pela tela,
  pela edicao do cartao ou pela planilha.
- Duplicar o cartao nao copia o co-participante.
- Sair da conta nao transforma a venda compartilhada em venda sozinho: vale o que foi gravado no cartao.
