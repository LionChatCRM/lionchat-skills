# Regras que valem em toda operacao

Este documento existe porque o conector nem sempre entrega estas regras sozinho: **no conector
local elas nao chegam ate voce.** Nao delegue — leia daqui.

Indice:

1. Antes de negar, pesquise
2. Ler antes de escrever, reler depois de escrever
3. Confirmar antes de agir
4. Paginacao: o tamanho da pagina nem sempre obedece
5. Acao em massa: teto de 300
6. Fuso horario: nunca chute
7. Regras de formato que falham em silencio
8. Listas substituem, nunca misturam
9. Campo desconhecido some sem aviso
10. Embrulho do corpo: quase sempre tanto faz, com uma excecao
11. Prefira o campo nativo ao campo personalizado
12. O que nao adianta repetir
13. Freio de frequencia no conector remoto

---

## 1. Antes de negar, pesquise

Nunca diga "nao existe funcao para isso" sem ter pesquisado o catalogo pelo padrao
`lionchat_<area>_<acao>`. Alguns clientes de IA nao carregam as 850 ferramentas de uma vez. Ja
aconteceu em producao: uma sessao afirmou que nao havia funcao de listar conversas e achou dezenas
na primeira pesquisa.

Se depois de pesquisar continuar sem achar, consulte `nao-tem-caminho.md` antes de dizer que nao da:
pode ser uma das coisas que so o painel faz, e ai a resposta certa e outra.

---

## 2. Ler antes de escrever, reler depois de escrever

**Antes:** liste o que ja existe do tipo que voce vai criar. Etiqueta, campo personalizado, funil,
resposta pronta e time duplicados com nome parecido sao o estrago mais comum de uma IA apressada, e
a skill nao apaga nada — o cliente fica com a bagunca.

**Depois:** releia com o `show` ou o `list` correspondente e compare com o combinado. Nao e zelo: e
a unica prova de que gravou (ver item 9).

---

## 3. Confirmar antes de agir

Confirme com o cliente, em portugues e em linguagem de tela, antes de:

- criar qualquer coisa
- alterar qualquer configuracao
- excluir qualquer coisa (e mesmo assim, prefira nao excluir)
- enviar qualquer mensagem ao cliente final dele
- executar um pedido ambiguo

Contam como "sim": sim, pode, confirmado, beleza, manda ver, isso. Nao conta: silencio, "acho que
sim", "e isso mesmo?".

17 ferramentas tem trava propria no sistema e recusam a primeira chamada de proposito, pedindo
`confirm:true`. A lista esta em `catalogo.md`, secao 8. **Aquilo e o piso, nao o teto**: sua regra
de confirmacao vale para tudo, esteja a ferramenta na lista ou nao.

---

## 4. Paginacao: o tamanho da pagina nem sempre obedece

O padrao e 25 itens por pagina, com teto de 100. **Mas algumas areas tem tamanho FIXO e ignoram o
pedido:**

- **Conversas: 25 por pagina, fixo.**
- **Contatos: 15 por pagina, fixo.**

Nesses casos pedir paginas maiores nao muda nada. O jeito e avancar de pagina em pagina. Parar na
primeira e entregar relatorio incompleto com cara de completo — e ninguem percebe.

Se a resposta trouxer o aviso de que a lista foi enxugada (mostrando N de M itens), o trabalho nao
acabou: continue.

---

## 5. Acao em massa: teto de 300

Acao em massa aceita no maximo 300 identificadores por chamada. Acima disso e recusada. Vale
tambem para os cards do funil. Divida em lotes de 300.

Excecao: acao em massa de CONTATOS com a opcao de selecionar tudo — ali o servidor refaz o filtro
por conta propria e processa em segundo plano.

---

## 6. Fuso horario: nunca chute

A conta tem fuso proprio e **nem toda conta e o fuso de Sao Paulo**. Mato Grosso, Mato Grosso do
Sul, Amazonas e Acre sao outro fuso, e ha clientes fora do Brasil.

Leia o campo de fuso horario em `lionchat_account_show` e use ELE em todo filtro de data, todo
relatorio e todo horario que voce escrever. Datas vao no formato completo, com o fuso junto.

---

## 7. Regras de formato que falham em silencio

| Campo | Regra | Errado | Certo |
|---|---|---|---|
| titulo de etiqueta | sem espaco; use hifen ou sublinhado | `Lead Emive` | `lead-emive` |
| telefone | formato internacional, sem espaco nem parenteses | `(11) 99999-9999` | `+5511999999999` |
| chave de etapa do funil | minusculo, palavras ligadas por sublinhado | `Em Negociacao` | `em_negociacao` |
| motivos de ganho e perda | lista de objetos com identificador e titulo, nunca texto solto | `["Preco"]` | `[{"id":"wr-1","title":"Preco competitivo"}]` |
| conversas vinculadas | lista de objetos com o numero da conversa, nunca numeros soltos | `[123]` | `[{"display_id":123}]` |

---

## 8. Listas substituem, nunca misturam

Quando voce manda uma lista, ela **substitui a anterior por inteiro**. Nao ha mistura. Isso vale
para etapas do funil, motivos de ganho e perda, conversas vinculadas, membros de caixa,
configuracao de distribuicao automatica e configuracao do canal.

Consequencias reais:

- `lionchat_inbox_members_create` **nao adiciona um membro: define a lista completa.** Chamar com um
  atendente numa caixa que tem 111 deixa a caixa com 1. Para um atendente NOVO, use
  `lionchat_agents_create` ja com os times e as caixas dele (esse e aditivo). Para um atendente que
  ja existe, LEIA a lista atual, junte o novo e mande a lista COMPLETA.
- `lionchat_team_members_create` **e aditivo** — o contrato dos dois e diferente de proposito. Nao
  suponha que o de time se comporta como o de caixa.
- Na configuracao do canal de uma caixa de WhatsApp oficial, mandar um campo apaga os outros em
  silencio, inclusive a chave que faz a caixa receber mensagem. **A caixa para de receber.** Leia,
  junte na mao, reenvie inteiro.

Regra unica: **leia com o `show`, junte na mao, reenvie completo.**

---

## 9. Campo desconhecido some sem aviso

Se voce mandar um campo que a ferramenta nao conhece, ele e descartado. A resposta volta como
sucesso, voce diz "pronto, configurei" e o painel segue com o valor antigo.

Dois sintomas:

- **O comum:** sucesso, e nada mudou. So a releitura pega.
- **O que engana:** se o campo inventado for o UNICO que voce mandou, o corpo da chamada fica vazio e
  o erro que volta e "parametro faltando". Parece problema de formato, mas e campo inexistente.

Existe uma variante ainda mais traicoeira, em FILTRO: o campo esta declarado na ferramenta, a
chamada passa, e o filtro **simplesmente nao acontece**. O numero sai errado e nada avisa. Como
descobrir: **mude o valor do filtro e veja se o resultado muda.** Se nao mudar, o filtro nao esta
valendo — nao entregue aquele numero ao cliente.

---

## 10. Embrulho do corpo: quase sempre tanto faz, com uma excecao

Quase todos os caminhos aceitam os campos direto ou embrulhados numa chave com o nome do recurso —
o conector resolve isso sozinho. Voce nao precisa se preocupar.

**A excecao:** criar ou alterar CONTATO. Ali, embrulhar faz os campos sumirem e o contato nasce com
o nome vazio, sem erro nenhum. Mande os campos na raiz.

(Se voce leu em algum lugar que "sem o embrulho a API devolve erro", isso e falso e vale para o
caminho antigo por chamada direta, nao para as ferramentas.)

---

## 11. Prefira o campo nativo ao campo personalizado

Antes de criar um campo personalizado, veja se ja existe um campo nativo para aquilo:

| Voce quer | Ja existe como | Nao crie campo personalizado |
|---|---|---|
| motivos de ganho e perda do funil | configuracao do Kanban | sim |
| checklist reaproveitavel | modelos de checklist na configuracao do Kanban | sim |
| marcacao transversal em contato ou conversa | Etiqueta | sim |
| ao ganhar, mandar o card para outro funil | Automacoes do proprio funil (gatilho "Ganho", acao "Duplicar Item para Funil") | sim |

Campo personalizado criado a mais nunca some sozinho e polui a ficha do cliente para sempre.

---

## 12. O que nao adianta repetir

- **Recusa por acesso ou permissao:** permanente. Repetir nao muda. Explique ao cliente e pare.
- **Recusa por parametro invalido:** permanente. Corrija o parametro, nao repita igual.
- **Chamada de CRIACAO que falhou:** nao repita direto. Confira antes com uma leitura — se ela
  criou e falhou depois, repetir cria dois registros.

Detalhes de cada erro em `erros.md`.

---

## 13. Freio de frequencia no conector remoto

O conector remoto aceita 120 chamadas por minuto. Trabalho volumoso (paginar centenas de contatos,
varrer conversas, acao em massa em lotes) estoura isso e recebe recusa no meio do caminho.

Se acontecer: espere um minuto e **continue de onde parou** — nao recomece do zero, e nao repita
chamadas de escrita que ja podem ter passado sem voce ver a resposta.

O conector local nao tem esse freio.
