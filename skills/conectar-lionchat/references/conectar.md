# Como o cliente liga a IA na conta LionChat

Indice:

1. Descobrir por qual conector voce esta ligado
2. Os tres portoes de liberacao (o que barra o cliente antes de tudo)
3. Conector REMOTO (sem instalar nada)
4. Conector LOCAL (programa no computador)
5. Trocar de conta
6. "Conectei e nao funciona" — o roteiro de socorro
7. Onde isso fica registrado

---

## 1. Descobrir por qual conector voce esta ligado

Nao pergunte ao cliente. Olhe a sua propria lista de ferramentas.

| Se voce tem esta ferramenta | Voce esta no | O que muda |
|---|---|---|
| `lionchat_ping` ou `lionchat_list_categories` | conector LOCAL | a conta se troca por um campo em cada chamada |
| `lionchat_current_account` ou `lionchat_switch_account` | conector REMOTO | a conta e da sessao inteira |
| `lionchat_search_tools` (com `lionchat_describe_tool` e `lionchat_execute_tool`) | REMOTO em modo compacto | voce nao ve o catalogo inteiro: pesquisa, le a ficha e executa pelo nome |
| nenhuma que comece com `lionchat_` | nada conectado | siga a secao 3 ou 4 |

Ferramenta que existe nos DOIS: `lionchat_flows_schema_reference` (o manual do desenho de fluxo;
nao gasta chamada, pode chamar a vontade).

Ferramenta que existe SO no remoto, alem das tres de conta: `lionchat_contacts_bulk_create` (cria
ate 1000 contatos numa chamada; no local ela nao existe).

---

## 2. Os tres portoes de liberacao

A regra e UMA e vale nos tres lugares:

> Passa quem tem a marca individual de acesso no proprio usuario. Sem ela, precisa das DUAS coisas:
> o recurso de conexao por IA ligado NA CONTA **e** ser administrador daquela conta.

| Portao | Onde bate | O que o cliente ve |
|---|---|---|
| 1 — nas chamadas | toda vez que voce chama uma ferramenta | erro de acesso com a mensagem "O acesso via MCP nao esta habilitado para esta conta. Peca ao administrador ou fale com o suporte para liberar." |
| 2 — no Autorizar | na tela de autorizacao do conector remoto | a mesma mensagem, na tela, e a autorizacao nao completa |
| 3 — no painel | ao tentar criar um conector novo | faixa amarela e o botao "Adicionar conector" desligado |

**O estado de fabrica e DESLIGADO.** Conta nova (e conta de plataforma com marca propria) chega ao
cliente sem esse recurso: o rollout so ligou nas contas que ja existiam. Ou seja, para um cliente
novo esse erro e o comportamento ESPERADO, nao um defeito de configuracao dele. Fale assim:

> O acesso por IA e liberado por conta e ainda nao esta ligado na sua. Isso nao e erro de
> configuracao sua. Se voce e administrador, fale com o suporte do LionChat para ativar no seu
> plano; se voce nao e administrador, peca a um administrador da conta.

Depois de liberado, **nao precisa reconectar nada** — a IA volta a funcionar sozinha.

---

## 3. Conector REMOTO (sem instalar nada)

Para quem usa a IA no navegador ou no celular: Claude.ai, ChatGPT, Gemini, Cursor, n8n.

**O que o cliente precisa ter em maos:** so o login do LionChat.

**Passo a passo:**

1. **ANTES de tudo, abrir no painel a conta certa.** Isso e o que mais da errado. A conta julgada
   na hora de autorizar e a conta que estiver ABERTA no painel naquele momento — quem tem varias
   empresas e autoriza estando na errada leva recusa mesmo sendo administrador de outra, ou pior:
   conecta e depois toda chamada e bloqueada. Essa conta tambem e onde a sessao nasce.
2. Colar o endereco `https://mcp.lionchat.com.br` na ferramenta de IA, no lugar de conector
   personalizado.
3. Fazer login no LionChat e clicar em Autorizar.

**Se a ferramenta de IA pedir Client ID e Client Secret** (algumas pedem, o Claude.ai padrao nao),
o cliente gera antes no painel:

> Configuracoes do Perfil > secao "Conectores" > "Conectores de IA" > botao "Adicionar conector"

Ali ele da um nome ao conector ("Claude", "ChatGPT") e recebe tres campos: URL do conector, Client
ID e Client Secret. **O Client Secret aparece uma unica vez** — perdeu, cria outro conector. Ha um
botao "Copiar tudo (JSON)".

Nessa mesma tela ele ve a lista de conectores, quando cada um foi usado pela ultima vez, e o botao
"Revogar" por linha. Revogar continua liberado mesmo para quem perdeu o direito de criar.

**Para ChatGPT, acrescente `?mode=compact` no fim do endereco.** Sem isso o ChatGPT nao carrega o
catalogo e passa a responder que a funcao "nao existe". No modo compacto voce ganha tres
ferramentas de balcao (pesquisar no catalogo, ver a ficha de uma ferramenta, executar pelo nome)
mais um enxoval de cerca de 27 ferramentas do dia a dia. O modo compacto ignora qualquer filtro por
area.

**Filtro por area no remoto** (opcional): `?categories=` aceita o nome da area **em portugues,
minusculo**, como aparece na coluna "Nome da area" de `references/catalogo.md` — por exemplo
`?categories=conversas,caixas de entrada`. Nao use o slug do conector local aqui: nao casa nada e o
cliente fica sem nenhuma ferramenta daquela area.

**Freio de frequencia:** o servidor remoto aceita 120 chamadas por minuto por origem. Trabalho
volumoso (paginar centenas de contatos, varrer conversas, acao em massa em lotes) estoura isso e
recebe recusa no meio. Se acontecer, espere um minuto e continue de onde parou — nao recomece.

---

## 4. Conector LOCAL (programa no computador)

Para quem usa Claude Code, Cursor, Windsurf, Continue.dev, n8n rodando na propria maquina.

**O que o cliente precisa ter em maos, nesta ordem:**

1. **O Token de acesso.** Painel > Configuracoes do Perfil > bloco "Token de acesso" > botao
   "Copiar". Esse token e do USUARIO, nao da conta: o mesmo token vale em qualquer empresa onde ele
   e membro. Ha um botao "Reiniciar" ao lado que gera outro e **invalida o anterior** — se o cliente
   apertar por engano, todos os lugares que usavam o token param.
2. **O numero da conta.** Esta na URL do painel, depois de `/accounts/`:
   `app.lionchat.com.br/app/accounts/NUMERO/dashboard`.
3. Ter o Node.js versao 18 ou mais nova instalado.

**O comando** (Claude Code):

```
claude mcp add lionchat -e LIONCHAT_API_TOKEN=o_token -e LIONCHAT_ACCOUNT_ID=o_numero -- npx @lionchat/mcp-server
```

**Ajustes opcionais**, todos como `-e NOME=valor` no mesmo comando:

| Ajuste | Para que serve |
|---|---|
| `LIONCHAT_BASE_URL` | apontar para outra instalacao (plataforma com marca propria ou servidor do proprio cliente). Padrao: `https://app.lionchat.com.br` |
| `LIONCHAT_CATEGORIES` | carregar so algumas areas, para a IA nao gastar memoria com o catalogo inteiro. Lista separada por virgula, usando os SLUGS de `references/catalogo.md` (ex.: `contacts,conversations,kanban_items`) |
| `LIONCHAT_INCLUDE_PUBLIC_API` | liga 6 ferramentas de agendamento e pesquisa de satisfacao publicos, desligadas de fabrica |
| `LIONCHAT_MCP_MAX_RESPONSE` | teto de tamanho da resposta antes do corte. Padrao 80 mil caracteres, minimo 10 mil |

**Aviso sobre seguranca do token:** ele vale como a senha do usuario para a API. Nunca mostre o
token inteiro em texto na conversa — mascare (`lc_prod_***`). Nunca escreva o token num arquivo do
projeto do cliente nem sugira commitar em repositorio.

---

## 5. Trocar de conta

**No conector remoto:** a conta e da SESSAO.

1. `lionchat_current_account` — diz onde voce esta agora. Chame antes de qualquer escrita.
2. `lionchat_list_my_accounts` — lista as contas do usuario com papel e situacao.
3. `lionchat_switch_account` — troca. Ele confere se o usuario e mesmo membro antes de trocar, e a
   escolha fica gravada: a proxima sessao ja nasce nessa conta.

O campo `account_id` **nao existe** nas ferramentas do remoto, de proposito — a conta vem do login.

**No conector local:** nao existe ferramenta de trocar conta.

- Para descobrir quais contas o usuario tem: `lionchat_account_list`. **O nome engana** — ele
  devolve o perfil do usuario, e as contas vem dentro, cada uma com o nome, a situacao e o PAPEL do
  usuario nela. E onde voce confere se ele e administrador antes de tentar qualquer coisa. E o mesmo
  lugar que o `lionchat_ping` bate.
- Para falar com outra conta: passe `account_id` na propria chamada. Ele existe como campo opcional
  em toda ferramenta. Ausente, usa o numero da configuracao.

**Skill que passa `account_id` em toda chamada funciona no local e e IGNORADA no remoto.** Skill que
chama `lionchat_switch_account` nao funciona no local. Trate os dois casos.

---

## 6. "Conectei e nao funciona" — o roteiro de socorro

Percorra nesta ordem e pare no primeiro que explicar o sintoma.

1. **Nao aparece NENHUMA ferramenta do LionChat.** Ou a conexao nao foi concluida, ou o cliente esta
   no ChatGPT sem o `?mode=compact`, ou (no local) o filtro por area usou um valor que nao casa
   nada. Confira o filtro contra os slugs de `references/catalogo.md`: `campaigns` nao casa nada, o
   slug e `campanhas`; `integrations` nao casa nada, e `integracoes`.
2. **Aparecem ferramentas, mas toda chamada e recusada por acesso.** Portao 1: ou a conta nao tem o
   recurso, ou o usuario nao e administrador dela. Ver secao 2. Nao insista — o resultado nao muda.
3. **Recusou na hora de Autorizar.** Portao 2: quase sempre e a conta errada aberta no painel. Peca
   para o cliente abrir a empresa certa no painel e refazer a autorizacao.
4. **"Conecta e desconecta" no ChatGPT.** A causa mais comum e ter MAIS DE UM conector do LionChat
   no mesmo usuario: o ChatGPT mantem um vivo e abandona os outros, que expiram e aparecem como
   desconectados. Regra: um conector so por usuario, e trocar de conta pelo proprio chat com
   `lionchat_switch_account`, nunca criando outro conector.
5. **Continua caindo depois de arrumar isso.** Se o cliente criou o conector ha muito tempo, pode
   ser preciso apagar e criar o app de novo uma vez — algumas ferramentas de IA guardam os dados da
   conexao no momento da criacao e nao releem.
6. **"Conecta e desconecta" a cada poucos minutos no Claude Code ligado ao remoto.** Era um defeito
   do servidor remoto, corrigido. Se ainda acontece, o cliente esta numa instalacao com versao
   antiga do conector remoto — nao e a skill nem a configuracao dele.
7. **"Essa funcao nao existe".** Antes de afirmar isso, PESQUISE. Ferramentas seguem o padrao
   `lionchat_<area>_<acao>`: procure por `lionchat_conversations_`, `lionchat_contacts_`,
   `lionchat_kanban_items_`. Ja aconteceu de uma sessao afirmar que nao havia funcao de listar
   conversas e achar dezenas na primeira pesquisa.

---

## 7. Onde isso fica registrado

O LionChat mantem um historico de Auditoria e, quando a mudanca que voce fez entra nesse historico,
ela sai carimbada com a origem da conexao (conector local ou remoto, e a versao). O cliente ve em
Configuracoes > Auditoria, e voce le com `lionchat_audit_logs_show`.

Duas ressalvas, para nao prometer demais:

- **O historico nao cobre tudo.** Ele registra o que a plataforma ja auditava antes de existir
  conexao por IA: caixas de entrada, membros de caixa, times e membros de time, atendentes e o
  papel deles, automacoes, webhooks, configuracao da conta e exclusao de conversa. Criar um
  contato, um fluxo ou uma campanha, por exemplo, **nao** gera linha ali.
- **A tela de Auditoria e um recurso a parte, que nem toda conta tem.** Sem ele a lista vem vazia
  mesmo que a mudanca tenha acontecido. Se o cliente perguntar "quem mexeu nisso?" e a lista vier
  vazia, essa e a explicacao mais provavel — nao afirme que ninguem mexeu.
