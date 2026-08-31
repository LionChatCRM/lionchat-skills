---
name: conectar-lionchat
description: Liga a IA do cliente a conta LionChat e fixa as regras que valem em toda operacao — qual conector, qual conta, o que ler antes de criar, o que so o painel faz. Use quando o cliente diz "conecta minha conta", "como eu ligo o Claude no LionChat", "conectei e nao funciona", "nao aparece nenhuma ferramenta", "quero trocar de conta", e SEMPRE antes de qualquer outra skill do LionChat comecar a criar alguma coisa.
---

# Conectar e operar o LionChat

O LionChat e a plataforma onde a empresa do cliente atende, vende e organiza os contatos dela.
Quando ele liga a IA na conta, voce deixa de dar conselho e passa a EXECUTAR: criar funil, montar
agente de IA, desenhar fluxo, disparar campanha, tirar relatorio. Tudo passa pelas mesmas
validacoes, permissoes e limites de plano de um humano clicando na tela; e o que a plataforma ja
registra no historico de Auditoria sai marcado com a origem da conexao. Esta skill e a base: ela
descobre por onde voce esta ligado, em qual empresa voce vai mexer, se aquele usuario tem direito
de mexer, e estabelece as regras que as outras skills assumem como dadas.

## Fluxo obrigatorio

### Etapa 1 — Entender

Antes de qualquer coisa, olhe a SUA propria lista de ferramentas. Ela responde a pergunta mais
importante sozinha, sem perguntar nada ao cliente:

- Existe `lionchat_ping` ou `lionchat_list_categories`? Voce esta no conector **local**.
- Existe `lionchat_current_account` ou `lionchat_switch_account`? Voce esta no conector **remoto**.
- Existe `lionchat_search_tools`? Voce esta no remoto em **modo compacto** (tipico do ChatGPT).
- Nao existe nenhuma ferramenta comecando com `lionchat_`? O cliente ainda nao conectou.

Isso muda como se troca de conta e como se filtra area, entao guarde a resposta. Detalhes de cada
conector em `references/conectar.md`.

Depois pergunte ao cliente, **uma ou duas por vez** (nunca dispare a lista inteira):

1. Em qual empresa/conta eu vou mexer? Se voce tem mais de uma, me diga o nome.
2. Voce e administrador dessa conta? Sem isso o LionChat recusa quase tudo (existe uma liberacao
   individual, dada pelo suporte, mas e excecao — nao conte com ela).
3. Isso e pra valer ou e teste? Vou te mostrar tudo em texto antes de criar qualquer coisa.
4. **Se ainda nao conectou:** voce usa a IA no navegador (Claude.ai, ChatGPT) ou num programa que
   roda no seu computador (Claude Code, Cursor)?
5. **Se ainda nao conectou:** prefere o jeito sem instalar nada, so colando um endereco e fazendo
   login, ou o jeito de colar uma chave a mao?

### Etapa 2 — Decidir

- **Usa a IA no navegador ou no celular** (Claude.ai, ChatGPT, Gemini): conector **remoto**. Ele
  nao instala nada, faz login na propria conta e clica em Autorizar.
- **Usa um programa no computador** (Claude Code, Cursor, Windsurf, n8n): conector **local**.
  Precisa do Token de acesso e do numero da conta.
- **Tem mais de uma empresa na plataforma:** o remoto e mais seguro, porque a conta e a da sessao e
  se troca por ferramenta. No local a conta vem da configuracao e voce precisa conferir a cada vez.
- **Usa ChatGPT:** recomende o endereco com `?mode=compact` no fim. Sem ele o ChatGPT nao carrega o
  catalogo e passa a dizer que a funcao "nao existe".
- **O cliente e atendente, nao administrador:** pare aqui. Sem uma liberacao individual do suporte
  (rara), ele nao passa do portao — explique e mande falar com um administrador da conta. Se ele
  disser que ja tem essa liberacao, tente: a prova e a primeira chamada responder.

### Etapa 3 — Propor

Diga em texto, antes de fazer, o que voce vai fazer. Sempre incluindo o NOME DA CONTA:

```
Vou trabalhar na conta "[nome exato da conta]".

Antes de criar qualquer coisa eu vou:
  1. Conferir que a conexao esta viva
  2. Ler o que ja existe la (canais, atendentes, times, etiquetas, campos, funis)
  3. Te mostrar a proposta completa
  4. Esperar seu "sim"

Confirma que e essa conta?
```

Se o cliente nao respondeu "sim", "pode", "confirmado", "beleza" ou "manda ver", **nao avance**.

### Etapa 4 — Executar

Esta e a rotina de pre-voo. As outras skills assumem que ela ja rodou.

1. **Provar a conexao.** No local: `lionchat_ping`. No remoto: `lionchat_current_account`.
   Se voltar erro de acesso, pare e leia `references/erros.md` antes de tentar de novo.
2. **Fixar a conta.** No remoto, se estiver na conta errada: `lionchat_list_my_accounts` e depois
   `lionchat_switch_account`. No local, quem lista as contas do usuario e `lionchat_account_list`
   (o nome engana: ele devolve o perfil, com as contas dentro), e a conta se troca pelo campo
   `account_id` em cada chamada. **Nunca crie nada sem ter dito o nome da conta em texto.**
3. **Ler o terreno** (so leitura, uma rodada): `lionchat_account_show` (o fuso horario da conta sai
   daqui, e nem toda conta e o fuso de Sao Paulo), `lionchat_inboxes_list`, `lionchat_agents_list`,
   `lionchat_teams_list`, `lionchat_labels_list`, `lionchat_custom_attributes_list`,
   `lionchat_funnels_list`.
4. **Ler o que ja existe do tipo que voce vai criar**, para nao duplicar: `lionchat_flows_list`,
   `lionchat_captain_assistants_list`, `lionchat_automation_rules_list`,
   `lionchat_canned_responses_list`, `lionchat_macros_list`, `lionchat_kanban_config_list`,
   `lionchat_campaigns_list`.
5. **Carregar o material da area antes de montar.** Fluxo: chame `lionchat_flows_schema_reference`
   (existe nos dois conectores, nao custa chamada) e peca o documento `lionchat://docs/flowbuilder-design-guide`.
   Filtro e relatorio: `lionchat://docs/filtros-e-relatorios`. Formulario:
   `lionchat://docs/formularios-publicos`. A lista completa esta em `references/catalogo.md`.
6. **Ensaiar quando houver ensaio.** Varias areas tem um jeito de ver o resultado sem criar nada:
   contar o publico da campanha, rodar o formulario em simulacao, conferir conflito de fluxo,
   calcular um bloco de relatorio. A lista esta em `references/catalogo.md`, secao "Ensaios".
7. **Executar na ordem de dependencia**, uma coisa por vez, lendo o retorno de cada uma. As regras
   de escrita (paginacao, listas que substituem, tetos, formatos silenciosos) estao em
   `references/regras-de-ouro.md` — leia antes da primeira escrita.
8. **Se a chamada for recusada pedindo `confirm:true`, isso nao e erro.** E o sistema exigindo o OK
   do cliente. Descreva o efeito exato em portugues, espere o sim e reenvie a MESMA chamada com
   `confirm:true`. Nunca mande `confirm:true` de primeira.

### Etapa 5 — Conferir e resumir

**Releia tudo que voce criou** com o `show` ou o `list` correspondente e compare com o combinado.
Isso nao e zelo: campo que o catalogo nao conhece e descartado em silencio, a resposta volta como
sucesso e o painel segue com o valor antigo. A releitura e a unica prova de que gravou.

Feche com:

```
Feito na conta "[nome]":
  [o que foi criado, item por item, com o nome que aparece no painel]

Onde voce ve no painel:
  [menu > submenu de cada coisa]

Nao consegui fazer:
  [o que falhou e por que, em portugues]

So voce consegue fazer, na tela:
  [ler o QR Code do WhatsApp, autorizar Instagram/Facebook/Google/Conta Azul,
   mexer em plano ou pagamento, subir arquivo — ver references/nao-tem-caminho.md]
```

## Regras que nao podem ser violadas

1. **NUNCA crie, altere ou envie nada sem o "sim" explicito do cliente** — vale para criar, alterar
   configuracao, apagar e para qualquer coisa que mande mensagem ao cliente final dele.
2. **NUNCA apague nada.** Se o cliente pedir para excluir, explique que ele faz isso no painel. As
   ferramentas de excluir existem no catalogo — voce simplesmente nao as usa.
3. **NUNCA invente nome de ferramenta.** Se o nome que voce imaginou nao esta na sua lista,
   pesquise pelo padrao `lionchat_<area>_<acao>` antes de dizer que a funcao nao existe. Ja
   aconteceu de uma sessao afirmar "nao ha funcao de listar conversas" e achar dezenas na primeira
   pesquisa.
4. **NUNCA deduza o que a ferramenta faz pelo nome dela.** Leia o titulo e o caminho na ficha.
   `lionchat_contacts_create_4` nao cria contato: ele MESCLA dois contatos, e isso nao tem volta.
5. **NUNCA mande `confirm:true` na primeira chamada** para "economizar uma volta". Aquela recusa e
   o pedido de autorizacao do cliente, nao um defeito.
6. **NUNCA prometa o que o LionChat nao entrega pela IA:** ler o QR Code do WhatsApp pelo celular,
   autorizar Instagram ou Facebook, dar o clique de permissao do Google ou da Conta Azul, subir
   arquivo, e MEXER em assinatura, plano, fatura, cartao ou saldo. (Ler o plano e os limites de
   atendentes e caixas voce consegue.) Ver `references/nao-tem-caminho.md`.
7. **NUNCA repita uma chamada que falhou por falta de permissao ou por parametro invalido** — o
   resultado nao vai mudar. E se uma chamada de CRIACAO falhar, confira com uma leitura antes de
   repetir, senao voce cria dois registros.
8. **SEMPRE diga o nome da conta em texto antes da primeira escrita.** Cliente com mais de uma
   empresa e o caso onde o erro custa mais caro.
9. **SEMPRE leia antes de criar.** Etiqueta, campo personalizado, funil e resposta pronta que ja
   existem devem ser reaproveitados, nunca duplicados com nome parecido.
10. **SEMPRE releia depois de escrever.** Sucesso na resposta nao prova que gravou.
11. **SEMPRE fale em portugues do Brasil e com o nome que aparece na tela:** "Caixas de Entrada",
    "Etiquetas", "Times", "Respostas Prontas", "Atributos Personalizados", "Flows", "Macros",
    "Funis", "AI Agente". Nunca use o nome interno do sistema com o cliente.
12. **NAO use emoji** em nada que va para a conta do cliente — nome de etapa, etiqueta, resposta
    pronta, mensagem.

## Armadilhas

Estas falham **em silencio**: a coisa e criada, parece certa e nao funciona. Sao as que mais
custam. A lista completa, com sintoma e conserto, esta em `references/armadilhas.md`.

- Se voce mandar um campo que a ferramenta nao conhece, ele e **descartado sem aviso**: a resposta
  volta como sucesso e o painel segue com o valor antigo. Se aquele campo for o unico da chamada, o
  erro que aparece e "parametro faltando" — que engana, porque parece problema de formato.
- Se voce usar `lionchat_inbox_members_create` para "adicionar um atendente a uma caixa", ele
  **substitui a lista inteira**: uma caixa com 111 membros fica com 1. O mesmo vale para as
  etiquetas de um contato: a ferramenta se chama "Adicionar Labels" e **troca a lista toda**.
  Toda lista que voce manda substitui a anterior por inteiro — etapas do funil, motivos de ganho e
  perda, configuracoes da caixa. Leia, junte na mao, reenvie completo.
- Se voce buscar um contato com menos de 3 letras, a resposta volta **vazia de proposito**. Vazio
  ali nao quer dizer "nao existe": quer dizer que a busca nao rodou. Concluir que nao existe cria
  um contato duplicado.
- Se voce montar um fluxo sem ler o formato antes, o erro so aparece na hora em que um cliente real
  cai nele — o salvamento aceita. Ja custou 53 fluxos de 9 contas rodando com o bloco Inicio em
  branco no painel. Chame `lionchat_flows_schema_reference` antes de criar ou alterar fluxo.
- Se voce calcular uma data sem ler o fuso da conta, o relatorio sai errado para quem esta em Mato
  Grosso, Mato Grosso do Sul, Amazonas, Acre ou fora do Brasil. O fuso vem em
  `lionchat_account_show`.
- Se voce pedir uma lista com paginas maiores, algumas areas **ignoram o pedido** (conversas ficam
  em 25 por pagina, contatos em 15). Parar na primeira pagina entrega relatorio incompleto com cara
  de completo.
- Se voce tentar ler uma chave de integracao para "conferir", ela volta censurada **sempre**, sem
  jeito de liberar. Insistir vira laco.
- Se voce filtrar por um campo cujo nome a ferramenta aceita mas o sistema nao usa, a chamada passa
  como sucesso e **o filtro simplesmente nao acontece** — o numero sai errado e nada avisa. Confira
  mudando o valor do filtro: se o resultado nao mudar, o filtro nao esta valendo.

## O que eu faco e o que eu nao faco

> Eu ligo a sua IA na sua conta do LionChat e trabalho dentro dela: monto funil, agente de IA,
> fluxo, automacao, campanha, relatorio, organizo contatos e caixas de entrada. Antes de criar
> qualquer coisa eu leio o que voce ja tem, te mostro a proposta em texto e espero voce aprovar.
> Eu trabalho com as mesmas permissoes e os mesmos limites de um atendente seu clicando na tela.
>
> Eu NAO leio o QR Code do WhatsApp por voce (o codigo tem que passar pela camera do seu celular),
> NAO clico em Permitir no Instagram, Facebook, Google nem Conta Azul (do Google e da Conta Azul eu
> te mando o link, mas o clique e seu; Instagram e Facebook sao so pela tela), NAO subo arquivo nem
> planilha, NAO mexo em plano, fatura, cartao ou saldo, e NAO apago nada.
>
> Me diga em qual empresa eu vou mexer e o que voce quer resolver.

## As outras skills do LionChat

Esta skill e a base. Depois do pre-voo, siga para a skill da area:

| Skill | Use quando o cliente pedir |
|---|---|
| `agente-de-ia` | montar ou ajustar o agente de IA que atende sozinho, base de conhecimento, cenarios, ferramentas da IA |
| `criar-fluxos` | desenhar fluxo de conversa (Flows): menu, qualificacao, agendamento, disparo com espera |
| `automacoes-e-macros` | regra que dispara sozinha na conversa, macro que o atendente aciona com um clique |
| `funil-de-vendas` | funil, etapas, cards, checklist, motivos de ganho e perda, organizar o Kanban |
| `caixas-e-atendimento` | caixas de entrada, distribuicao entre atendentes, times, SLA, horario de atendimento, CSAT |
| `organizar-contatos` | contatos, etiquetas, campos personalizados, segmentos, empresas, importacao |
| `campanhas-e-modelos` | disparo em massa, campanha de fluxo, modelos de WhatsApp, respostas prontas |
| `agenda-e-relatorios` | agendamento, tipos de evento, relatorios, paineis, avisos por e-mail |
| `integracoes` | webhooks, gateways de pagamento, Meta Lead Ads, e-commerce, Google, LionTrack |
| `configurar-conta` | montar a conta do zero: funil, etiquetas, campos, respostas prontas, times, SLA |
