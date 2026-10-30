# Analisar com IA - a IA sugere o que mudar nos cartoes

E o botao **Analisar com IA** do Kanban (topo do quadro e menu da coluna), desde 28/10/2026. A IA le os
cartoes ABERTOS e ADIADOS das etapas escolhidas de UM funil, junto com as conversas deles, e sugere: mudar de
etapa (inclusive para tras), levar para outro funil, por ou tirar etiqueta e marcar Ganho ou Perdido.

**Analisar nao muda nada. Aplicar muda e NAO TEM COMO DESFAZER**: fica como se uma pessoa tivesse mexido a mao,
e as automacoes do funil, os fluxos e as conversoes de anuncio disparam normalmente.

## Indice

1. Antes de oferecer: quem pode e o que precisa existir
2. As ferramentas
3. Passo a passo
4. Regras que mudam o que voce diz ao dono
5. "Por que a IA nao mudou nada?"

---

## 1. Antes de oferecer: quem pode e o que precisa existir

- **Quem pode**: administrador ou cargo com a permissao `kanban_ai_analyze` ("Analisar funil com IA"). Gerenciar
  o Kanban nao basta.
- **A conta precisa do recurso de plano `kanban_ai_analysis`** (aparece na lista de recursos de
  `lionchat_account_show`). Sem ele a resposta e `feature_disabled`.
- **Usa a chave da OpenAI DA CONTA** (`has_openai_key` no preparo; sem ela, `no_openai_key`). O custo e do
  dono. Costuma ficar abaixo de um centavo de dolar a cada dezenas de cartoes - mas mostre sempre o numero
  estimado antes.
- Funil que a pessoa nao ve = 404 `funnel_not_visible`.

## 2. As ferramentas

| Ferramenta | Para que |
|---|---|
| `lionchat_kanban_ai_analyses_setup` | o que a janela mostra, SEM criar nada: etapas com quantos cartoes cada uma tem, se o funil e as etapas tem descricao, quantos cartoes entram, custo e tempo estimados, funis de destino, etiquetas, se ha chave da OpenAI e a analise que ja esta viva no funil |
| `lionchat_kanban_ai_analyses_create` | cria a analise. Roda em segundo plano e NAO muda nenhum cartao |
| `lionchat_kanban_ai_analyses_show` | andamento: situacao, contadores e custo |
| `lionchat_kanban_ai_analyses_items` | a lista de sugestoes (so depois de pronta) |
| `lionchat_kanban_ai_analyses_apply` | aplica o que o dono escolheu. Pede `confirm: true`. **Sem desfazer** |
| `lionchat_kanban_ai_analyses_reanalyze` | analisa de novo UM cartao cuja linha travou |

## 3. Passo a passo

1. **Preparar**: `lionchat_kanban_ai_analyses_setup` com o `funnel_id` (e as `stages` escolhidas, para saber o
   total e o custo). Mostre as etapas e quantos cartoes cada uma tem. Marcar VARIAS etapas numa analise so evita
   ler a mesma conversa duas vezes.
2. **Perguntar ao dono**, uma ou duas coisas por vez:
   - o que a IA pode sugerir (`options`): mudar de etapa (`move_stage`), Ganho ou Perdido (`outcome`), levar
     para outro funil (`move_funnel` + `target_funnels`, cada funil com a chave da etapa de chegada), por
     etiquetas (`labels_add`) ou tirar etiquetas (`labels_remove`), pelo TITULO das etiquetas;
   - quais cartoes: todos, so os parados ha 7, 14 ou 30 dias (`stale_days`), ou so os sem resposta do cliente
     ha N dias (`no_reply_days`);
   - quantos (`limit`, de 1 a 1.000, padrao 200; os parados ha mais tempo primeiro).
   Mostre o custo e o tempo estimados para essa escolha (`estimated_cost_usd` e `estimated_minutes`). Se
   `already_read` vier maior que zero, diga que esses cartoes vao ser PULADOS (a IA ja os leu nos ultimos 7
   dias e o cliente nao escreveu depois); para ler de novo, `include_read: true` - e paga de novo.
3. **Criar** com `lionchat_kanban_ai_analyses_create` e acompanhar com `_show` a cada 5 a 10 segundos ate
   `ready` (ou `failed`, com o motivo em `error_code`).
4. **Mostrar as sugestoes** de `_items`, agrupadas por etapa: cartao, acao, forca ("Alta" = 70% ou mais,
   "Media" = 50 a 70%), motivo e o que a acao vai disparar (`hints`: conversao, regras do funil, fluxos, aviso
   de WhatsApp por QR Code).
5. **Confirmar com o dono** quais cartoes e quais acoes, quantas mudancas em quantos cartoes, e dizer com todas
   as letras que **nao tem como desfazer** e que automacoes, fluxos e conversoes vao disparar.
6. **Aplicar** com `lionchat_kanban_ai_analyses_apply`: `items` = `[{ "item_id": <id da linha>, "actions":
   [<ids das sugestoes escolhidas>] }]` e `confirm: true`. Os ids das acoes sao os das sugestoes daquela linha
   (`stage`, `funnel`, `outcome`, `label_add:...`, `label_remove:...`) - nunca invente: acao que nao foi
   sugerida volta em `ignored`.
7. **Conferir**: `_show` ate `finished` e `_items` de novo para ver o que foi feito em cada cartao.

## 4. Regras que mudam o que voce diz ao dono

- **Uma analise por vez por funil** (409 `funnel_busy`) e no maximo duas rodando na conta (`account_busy`).
  Criar uma nova **substitui a lista anterior** do mesmo funil que tiver etapa em comum.
- **A lista vale 24 horas.** Depois vence (`list_expired`) e e preciso analisar de novo.
- **O que vem marcado**: so as sugestoes de forca Alta de etapa, funil e Perdido. Ganho, forca Media e etiqueta
  vem desmarcados, com o motivo. Na duvida, proponha aplicar so as marcadas.
- **Linha travada** (`locked`: o cliente escreveu depois da analise, ou o cartao mudou): nao aplique. Use
  `_reanalyze` com o `item_id` e leia `_items` de novo depois de alguns segundos.
- **Linha que voltou** (`return_code`): `interrupted`, `card_busy` e `apply_failed` podem ser aplicadas de novo;
  `agent_no_access` quer dizer que o responsavel nao acessa o funil de destino - e preciso dar o acesso antes.
- **Sugestao melhora com descricao**: o preparo diz quais etapas e se o funil estao sem descricao. Escrever o
  que significa cada etapa ajuda a IA a acertar.

## 5. "Por que a IA nao mudou nada?"

A IA so sugere quando responde a mesma coisa com as opcoes em duas ordens diferentes e com pelo menos 50% de
certeza. Cartao sem conversa, ou com pouca conversa, costuma ficar como esta.

Para explicar caso a caso, chame `_items` com `no_change: true`: vem a lista dos cartoes lidos e deixados como
estao, cada um com o porque (`same` = confirmou o que ja esta; `doubt` = as duas ordens discordaram; `weak` =
abaixo de 50%; `other_funnel` = achou que o cartao e de outro funil; `not_chosen` = sugeriu um funil que nao
estava entre os destinos). Peca essa lista so quando for mostra-la.
