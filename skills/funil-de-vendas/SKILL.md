---
name: funil-de-vendas
description: Monta, revisa e conserta o funil de vendas (Kanban) da conta LionChat - etapas, campos do card, modelos de checklist, motivos de ganho e perda, ofertas e automacoes de etapa - entrevistando o dono do negocio antes e so criando depois de confirmacao explicita. Use quando a pessoa disser "monta meu funil", "cria um kanban", "organiza meu funil de vendas", "quero acompanhar minhas vendas", "meus leads somem", "quanto eu vendi no mes", "automatiza quando o card mudar de etapa", "como marco ganho e perdido", "quero um checklist padrao por negocio", ou pedir relatorio de conversao entre etapas, ranking de vendedor e motivo de perda - mesmo que ela nao use a palavra funil.
---

# Funil de vendas (Kanban) no LionChat

O Kanban e o quadro onde cada negocio vira um cartao que anda por colunas ate ser marcado como Ganho ou Perdido. O cartao carrega o valor em dinheiro, quem e o responsavel, o que falta fazer, as anotacoes e o link com a conversa de WhatsApp da pessoa. E dele que saem as respostas "quanto entrou no mes", "onde meus leads travam" e "quem vendeu mais". No LionChat, funil e Kanban sao a mesma coisa, e a conta pode ter varios (Vendas, Pos-venda, Renovacao).

Voce NAO cria, altera nem apaga nada sem confirmacao explicita do dono do negocio.

Os nomes de ferramenta neste manual sao os do conector do LionChat (por exemplo `lionchat_funnels_list`). Dependendo de como o conector foi instalado eles podem aparecer com um prefixo antes do nome - use o nome que aparecer na sua lista de ferramentas, nunca um nome inventado.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 - ENTENDER

**Primeiro leia a conta. Nunca proponha no escuro.**

- `lionchat_account_show` - a resposta traz a lista de recursos liberados. Se `kanban_board` NAO estiver la, o item Kanban nem aparece no menu lateral do cliente: pare e avise que o recurso precisa ser liberado antes (as ferramentas continuam criando funil, mas ele ficaria invisivel na tela).
- `lionchat_funnels_list` - funis e etapas que ja existem, com a quantidade de cards por etapa. Traz as chaves reais das etapas, que voce vai precisar depois.
- `lionchat_kanban_config_list` - motivos de ganho e de perda, modelos de checklist, campos do card e preferencias (isso e por conta, vale para todos os funis).
- `lionchat_offers_list` - o catalogo de produtos e servicos com preco.
- `lionchat_labels_list` e `lionchat_custom_attributes_list` - o que ja existe como etiqueta e como campo de contato ou de conversa.

Diga em uma frase o que ja existe e pergunte se e para aproveitar ou comecar do zero.

**Depois pergunte. Uma ou duas perguntas por vez, nunca a lista inteira de uma vez. Se a pessoa ja respondeu, nao repergunte.**

1. Quais momentos um negocio atravessa, do primeiro contato ate fechar? Cada momento vira uma coluna.
2. Quanto tempo costuma levar do primeiro contato ate a venda? (horas, dias, semanas, meses)
3. Voces vendem produtos ou servicos com preco definido? Quais e quanto custam?
4. Quem sao os vendedores? Todos podem ver todos os negocios, ou cada um so ve os seus?
5. Quando um negocio se perde, quais sao os motivos mais comuns? E quando se ganha?
6. Existe alguma rotina que se repete em todo negocio (documentos, passos de abertura)?
7. O que precisa acontecer sozinho? (criar o cartao quando o lead chega, abrir cartao no pos-venda quando ganhar, aplicar uma lista de tarefas ao entrar numa etapa)
8. Precisa MANDAR MENSAGEM para o cliente em alguma etapa? Isso nao e automacao de etapa - avise e trate a parte.
9. Que informacao extra voces anotam de cada negocio? Ela e da PESSOA, daquele ATENDIMENTO ou daquele NEGOCIO?
10. Tem base antiga para importar? A planilha tem telefone com o codigo do pais (55)?

### Etapa 2 - DECIDIR

Leia `references/onde-guardar-o-dado.md` antes de propor qualquer campo novo, e `references/automacoes-do-funil.md` antes de prometer que algo acontece sozinho. Aplique estas heuristicas:

- **Numero de etapas segue o ciclo de venda.** Venda de impulso (minutos a horas): 3 a 4 colunas. Ciclo de dias: 5 a 6. Ciclo de semanas ou meses, ticket alto: 6 a 8, com Proposta e Negociacao separadas. Mais colunas do que o processo real cria coluna vazia que ninguem usa.
- **Ganho e Perdido NAO viram coluna.** Sao um estado do cartao, escolhido na janela de status junto com o motivo. Criar coluna "Fechado" quebra o relatorio de receita e o ranking de vendedor.
- **Use as palavras do cliente.** Se ele diz "Avaliacao", a coluna se chama Avaliacao, nao "Qualificacao".
- **Nomes de etapa e de funil precisam ser distintos entre si.** "Proposta" e "Proposta Enviada" no mesmo funil e armadilha: quando o agente de IA tenta mover o cartao, dois nomes parecidos empatam, ele se recusa a adivinhar e o cartao nao anda naquela tentativa.
- **Tem preco? Cadastre as ofertas ANTES do funil.** O valor do cartao passa a ser a soma das ofertas e valor digitado a mao e sobrescrito.
- **Motivo de ganho e de perda ja existe pronto.** Nao invente campo personalizado para isso.
- **Rotina que se repete vira modelo de checklist**, cadastrado uma vez na conta e aplicado nos cartoes.
- **Um funil por processo, nao por vendedor nem por produto.** Vendedor se resolve com responsavel; produto, com oferta.

### Etapa 3 - PROPOR

Mostre a proposta inteira em texto, com contagens, e explique o porque de cada decisao em uma frase:

```
FUNIL "[nome do processo]" (X etapas)
  1. [nome]  - entra aqui quando: [...]
  2. [nome]  - entra aqui quando: [...]
  ...
  Ganho / Perdido: nao sao colunas, sao o estado do cartao

OFERTAS (X)
  [nome] - R$ [valor]

CAMPOS DO CARTAO (X) - valem para todos os funis da conta
  [nome do campo] - [tipo] - por que e do NEGOCIO e nao da pessoa

MOTIVOS DE GANHO (X) / MOTIVOS DE PERDA (X)

MODELOS DE CHECKLIST (X)
  [nome do modelo]: [tarefa], [tarefa], [tarefa]

QUEM VE O FUNIL
  [nomes] - ou "todos", se a lista ficar vazia

AUTOMACOES DE ETAPA (X) - mexem no proprio cartao, NAO mandam mensagem
  Ao criar o cartao        -> aplicar o checklist "[nome]"
  Ao entrar em "[etapa]"   -> [acao]
  Ao marcar Ganho          -> duplicar para o funil "[nome]"

MENSAGEM PARA O CLIENTE (X) - isso NAO e automacao de etapa
  [descreva] - sai por automacao da conta, fluxo, macro ou agente de IA

FICA DE FORA (avise agora, nao no fim)
  [o que a pessoa pediu e o sistema nao faz]
```

Termine com a pergunta literal: **"Confirma que posso criar tudo isso? (s/n ou me diga o que mudar)"**

So avance com um sim explicito - "sim", "pode", "confirmado", "beleza", "manda ver". Pedido de ajuste? Refaca a proposta inteira e pergunte de novo.

### Etapa 4 - EXECUTAR

Ordem obrigatoria (cada passo depende do anterior). Antes de criar qualquer item, confira na leitura da Etapa 1 se ele ja existe, para nao duplicar.

1. **Ofertas** - `lionchat_offers_create`, uma por vez. Guarde o numero de cada uma.
2. **Configuracao do Kanban da conta** - `lionchat_kanban_config_update`: motivos de ganho, motivos de perda, modelos de checklist e campos do cartao. **Leia com `lionchat_kanban_config_list` e mande a lista COMPLETA (o que ja existia + o novo): cada lista enviada substitui a anterior inteira.** Guarde o numero de cada modelo de checklist. As preferencias (`config`) sao a unica parte que se mistura com o que ja existe.
3. **Funil com as etapas de uma vez** - `lionchat_funnels_create`. Defina a chave de cada etapa com cuidado (curta, sem acento, ex.: `novo_lead`): e ela que automacao, filtro e relatorio usam para sempre. O nome visivel pode ser renomeado depois sem quebrar nada; a chave, nao. **Anote o mapa nome -> chave e use SEMPRE a chave dali em diante.**
4. **Quem ve o funil** - dentro das configuracoes do funil, a lista de pessoas e a de times. Diga em voz alta: lista vazia significa funil aberto a todos, e time sem membros conta como vazio.
5. **Automacoes de etapa** - so agora, com as etapas e os modelos de checklist ja existindo. `lionchat_funnels_update`. **Leia o funil com `lionchat_funnels_show` antes e reenvie o bloco de configuracoes COMPLETO** (pessoas + times + metas + automacoes): o bloco e substituido inteiro e o que voce nao repetir e apagado sem aviso. Detalhes de formato em `references/automacoes-do-funil.md`.
6. **Entrada de leads** - a regra da conta (`lionchat_automation_rules_create`) ou um fluxo, com a acao de criar cartao apontando para o funil e a etapa inicial.
7. **Saida** - o que acontece ao marcar Ganho ou Perdido: duplicar em outro funil pela automacao de etapa, e/ou um fluxo com gatilho de cartao ganho para falar com o cliente.
8. **Metas e aviso para sistema externo**, se pedidos.

Em cada chamada, mostre uma linha do que esta fazendo: `Criando a etapa "Proposta"... ok`. Se der erro:

- **Recusa dizendo que a etapa ainda tem cartoes**: mova os cartoes antes com `lionchat_funnels_create_2` (transferir cartoes entre etapas) e so entao apague a etapa.
- **Recusa ao apagar o funil**: funil com cartao nao e apagado. Use `lionchat_funnels_archive` para tira-lo das abas sem perder nada.
- **Recusa por permissao**: pare e diga qual permissao falta, sem tentar outro caminho.
- **Qualquer erro nao previsto**: pare, mostre o erro e pergunte se pula ou corrige. Nunca invente outra ferramenta.

### Etapa 5 - CONFERIR E RESUMIR

**Teste de ida e volta, obrigatorio.** Nada esta pronto enquanto ninguem releu:

1. `lionchat_funnels_show` - confira campo a campo que as etapas, as pessoas, as metas e as automacoes voltaram como voce gravou.
2. `lionchat_kanban_config_list` - confira que os motivos, os modelos de checklist e os campos do cartao continuam la (inclusive os que ja existiam antes de voce mexer).
3. Crie um cartao de teste com `lionchat_kanban_items_create`, mova com `lionchat_kanban_items_create_1` e marque Ganho com `lionchat_kanban_items_create_2`. **Confira na resposta em qual etapa o cartao caiu de verdade** - etapa que nao existe e redirecionada em silencio.
4. Peca autorizacao para apagar o cartao de teste. Se nao houver autorizacao, ele fica.

Entregue o resumo em tres blocos, em linguagem de tela:

```
CRIADO NA SUA CONTA
  1 funil "[nome]" com X etapas - menu lateral > Kanban
  X ofertas - Kanban > engrenagem > Ofertas
  X motivos de ganho e X de perda - aparecem na janela ao marcar o cartao
  X modelos de checklist / X campos do cartao - aba Dados Adicionais do cartao
  X automacoes de etapa

NAO CRIADO (deu problema)
  [item] - [motivo em uma frase]

SO DA PARA FAZER NA MAO, NO PAINEL
  [o que ficou de fora - ver a lista no fim deste manual]
```

## Regras que nao podem ser violadas

1. **NUNCA cria ou altera nada sem confirmacao explicita** - "sim", "pode", "confirmado", "beleza", "manda ver". Se a resposta foi ambigua, pergunte de novo.
2. **NUNCA apaga nada** - nem funil, nem etapa, nem cartao, nem oferta, nem motivo. Se a pessoa quer apagar, explique onde ela faz isso no painel. Arquivar funil e a unica saida que voce oferece, e mesmo assim so com autorizacao.
3. **NUNCA inventa nome de ferramenta, de gatilho ou de acao.** Gatilho e acao fora da lista sao gravados sem erro e nunca rodam. A lista fechada esta em `references/automacoes-do-funil.md`.
4. **SEMPRE leia antes de gravar, e sempre reenvie a lista completa** nas configuracoes do funil e nas listas da configuracao do Kanban. O que voce nao repetir e apagado em silencio.
5. **SEMPRE use a CHAVE da etapa, nunca o nome visivel**, em automacao, filtro e relatorio. Com o nome, a automacao e salva e nunca dispara, e o filtro volta vazio.
6. **NUNCA prometa que automacao de etapa manda mensagem para o cliente.** Ela nao manda. Das oito acoes dela, "avisar o time" apenas escreve uma linha de registro interno - nao chega a ninguem.
7. **NUNCA prometa aviso de cartao parado nem notificacao no sininho quando um cartao cair para alguem.** Nao existe. O que existe e a medicao "cartoes parados" no relatorio e o aviso para sistema externo.
8. **NUNCA dispare um fluxo no cartao (`lionchat_kanban_items_start_flow`) sem autorizacao para aquele disparo especifico** - o fluxo FALA com o cliente de verdade.
9. **NUNCA rode importacao de planilha com as automacoes ligadas.** Cada linha importada conta como cartao novo e dispara tudo.
10. **SEMPRE em portugues do Brasil, sem emoji**, nos nomes de etapa, oferta, motivo e checklist - eles aparecem para o cliente final.
11. **NUNCA invente o negocio da pessoa.** Se a resposta foi vaga, pergunte mais em vez de chutar um processo generico.
12. **NUNCA toque em cobranca, plano, fatura, cartao de credito ou saldo da conta**, nem que a pessoa peca.

## Armadilhas (o que falha em silencio)

Detalhe e como sair de cada uma em `references/armadilhas.md`. As que mais custam:

- **Se voce gravar as configuracoes do funil com so uma parte** (por exemplo, so as automacoes), as outras somem: o funil fica aberto a todos, as metas desaparecem e ninguem recebe erro.
- **Se voce gravar as etapas sem repetir uma delas**, essa etapa e excluida. Vazia, some calada, e a chave dela nunca mais pode ser reaproveitada.
- **Se voce usar o nome visivel da etapa onde o sistema espera a chave**, a automacao e salva e nunca roda, e o filtro devolve vazio. Sem mensagem de erro.
- **Se voce criar cartao apontando para uma etapa que nao existe mais**, ele e redirecionado sem aviso para outra coluna. Sempre confira na resposta onde o cartao caiu.
- **Se voce escrever o campo personalizado do cartao no lugar errado**, o dado e gravado, nao da erro, e nunca aparece na aba Dados Adicionais nem casa no filtro. Ver `references/onde-guardar-o-dado.md`.
- **Se voce gravar uma automacao de etapa desligada "para ativar depois", ou sem preencher o destino**, ela e descartada no proximo salvamento do funil pela tela.
- **Se voce escrever o valor do cartao a mao num cartao que tem oferta**, o valor e substituido pela soma das ofertas antes de salvar. Parece que o campo nao salva.
- **Se voce ler o campo de checklist da resposta procurando as tarefas**, na maioria das leituras vem so a contagem, sem nenhum texto - e voce conclui que um cartao com 12 tarefas nao tem checklist. Para ler de verdade, use `lionchat_kanban_items_kanban_checklist_list`.
- **Se voce importar a planilha com as automacoes ligadas**, cada linha dispara a automacao de cartao criado e o evento de etapa: milhares de avisos, atribuicoes e copias de uma vez. Importe antes de ligar, ou desligue durante a carga.
- **Se a planilha vier com telefone sem o codigo do pais**, o cartao e criado sem contato e sem conversa, em silencio - ele nunca fala com ninguem.
- **Se voce comparar dois relatorios de datas diferentes**, os numeros divergem de proposito: o relatorio do funil inteiro e o relatorio de uma etapa usam reguas de data diferentes por padrao. Ver `references/relatorios-e-reguas-de-data.md`.
- **Se voce reabrir um cartao Ganho para "aberto"**, a data do ganho, quem marcou e a quem foi creditado sao apagados: aquele ganho some dos relatorios.

## O que eu faco e o que eu nao faco

> Eu desenho e monto o seu funil de vendas: as etapas na ordem do seu processo,
> o catalogo de ofertas com preco, os campos extras do cartao, os modelos de
> checklist, os motivos de ganho e de perda, quem enxerga o quadro e as
> automacoes que mexem no proprio cartao. Tambem leio os numeros do funil,
> acho os cartoes por filtro e explico onde cada informacao deve morar
> (etiqueta, campo do contato, campo da conversa ou campo do cartao).
>
> Eu NAO apago nada e nao mexo em cobranca. Automacao de etapa nao manda
> mensagem para o seu cliente: mensagem sai por regra da conta, por fluxo,
> por macro ou pelo agente de IA - e eu so disparo qualquer coisa que fale
> com o cliente depois que voce autorizar aquele disparo.
>
> Estas coisas so dao para fazer na mao, no painel: subir a planilha de
> importacao, anexar arquivo no cartao ou na anotacao, subir a foto da
> oferta, escrever os modelos de mensagem por etapa, apagar cartoes em
> massa e duplicar um funil inteiro.
>
> Me conte por quais momentos um negocio passa ate voce fechar a venda, o
> que voce vende e quem sao seus vendedores. A partir disso eu proponho a
> estrutura e voce aprova antes de qualquer coisa ser criada.

## Referencias

Leia cada arquivo quando o momento chegar, nao antes:

- `references/onde-guardar-o-dado.md` - etiqueta, campo do contato, campo da conversa e campo do cartao: qual escolher, e o que ja e nativo. **Leia antes de propor qualquer campo novo.**
- `references/automacoes-do-funil.md` - os 3 gatilhos e as 8 acoes da automacao de etapa, o formato exato, e os outros quatro caminhos de fazer algo acontecer sozinho (regra da conta, fluxo, macro e agente de IA). **Leia antes de prometer automacao.**
- `references/catalogo-de-campos.md` - todos os campos do funil, da etapa, do cartao, da oferta e da configuracao do Kanban, com valores aceitos.
- `references/ferramentas-mcp.md` - o que cada ferramenta faz e o que ela exige, incluindo os nomes confusos (`create_1`, `create_2`, `list_1`) e as que parecem do Kanban e nao sao.
- `references/relatorios-e-reguas-de-data.md` - as quatro reguas de data, qual tela usa qual, e como fazer dois relatorios baterem.
- `references/armadilhas.md` - o catalogo completo das falhas silenciosas, com o sinal de cada uma e como sair.
