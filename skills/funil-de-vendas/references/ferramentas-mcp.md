# Ferramentas do funil de vendas

Os nomes abaixo sao os do conector do LionChat. Dependendo de como o conector foi instalado eles podem aparecer com um prefixo antes do nome - **use sempre o nome que aparecer na sua lista de ferramentas, nunca um nome inventado.** Se o que voce precisa nao estiver nesta lista, pergunte antes de tentar.

## Indice

1. Ler a conta antes de mexer
2. Funil
3. Configuracao do Kanban da conta
4. Ofertas
5. Cartao: criar, ler, mover, fechar
6. Cartao: buscar e filtrar
7. Cartao: checklist, anotacoes, responsaveis, anexos
8. Cartao: conversas ligadas, tarefas, fluxo
9. Acoes em massa
10. Relatorios
11. Importacao e exportacao
12. Envio de conversao para anuncio
13. Nomes que enganam
14. O que so da para fazer na tela

---

## 1. Ler a conta antes de mexer

| Ferramenta | Para que |
|---|---|
| `lionchat_account_show` | traz a lista de recursos liberados. Sem `kanban_board`, o item Kanban nem aparece no menu lateral do cliente |
| `lionchat_funnels_list` | os funis visiveis, com a quantidade de cartoes por etapa. **Sempre o primeiro passo** - e o que traz as chaves reais das etapas. Nao e paginado |
| `lionchat_kanban_config_list` | motivos de ganho e perda, modelos de checklist, campos do cartao, preferencias e o aviso para sistema externo. Cria o registro sozinho se ainda nao existir |
| `lionchat_offers_list` | catalogo de produtos e servicos |
| `lionchat_labels_list` | etiquetas da conta |
| `lionchat_custom_attributes_list` | campos de CONTATO e de CONVERSA (nao os do cartao) |

## 2. Funil

| Ferramenta | Para que | Cuidado |
|---|---|---|
| `lionchat_funnels_show` | um funil com etapas, pessoas, times, metas e automacoes | funil que a pessoa nao pode ver responde erro de nao encontrado, nao lista vazia |
| `lionchat_funnels_create` | cria o funil ja com as etapas e as configuracoes | o nome e unico na conta |
| `lionchat_funnels_update` | renomeia, muda etapas e grava pessoas, times, metas e automacoes | **as etapas e as configuracoes sao SUBSTITUIDAS por inteiro** |
| `lionchat_funnels_archive` / `lionchat_funnels_unarchive` | tira e devolve o funil das abas ativas sem perder cartao | e o caminho certo para funil antigo |
| `lionchat_funnels_destroy` | exclui o funil | so funciona com zero cartoes. **Voce nao apaga nada sem autorizacao** |
| `lionchat_funnels_create_2` | transferir cartoes de uma etapa para outra (aceita outro funil de destino) | e o passo obrigatorio ANTES de apagar uma etapa com cartoes |
| `lionchat_funnels_create_1` | transferir os cartoes de um vendedor para outro dentro do funil | so administrador |
| `lionchat_funnels_reorder` | ordem das abas de funil | |
| `lionchat_funnels_open_counts` | quantos cartoes ABERTOS cada funil tem, sem baixar cartao nenhum | so funis ativos e visiveis |
| `lionchat_funnels_list_1` | contagem e soma de valor por etapa (o cabecalho de cada coluna) | |
| `lionchat_funnels_stage_report` | os numeros de UMA etapa, calculados no servidor | exige a **chave** da etapa |

## 3. Configuracao do Kanban da conta

| Ferramenta | Para que |
|---|---|
| `lionchat_kanban_config_list` | ler tudo antes de gravar |
| `lionchat_kanban_config_update` | gravar motivos de ganho e perda, modelos de checklist, campos do cartao, preferencias e o aviso externo |
| `lionchat_kanban_config_create_1` | disparar um envio de teste para o endereco externo |
| `lionchat_kanban_deliveries_list` | registro das tentativas de entrega dos ultimos 7 dias (so administrador) |

**Cada LISTA enviada substitui a anterior inteira.** Leia, junte o novo ao que ja existia, e mande completo. So o bloco de preferencias se mistura com o que ja estava la.

## 4. Ofertas

`lionchat_offers_list`, `lionchat_offers_create`, `lionchat_offers_show`, `lionchat_offers_update`, `lionchat_offers_destroy`, `lionchat_offers_search`.

Editar ou excluir a oferta **nao muda os cartoes que ja a receberam**. A foto de dentro do cartao fica congelada.

## 5. Cartao: criar, ler, mover, fechar

| Ferramenta | Para que | Obrigatorios |
|---|---|---|
| `lionchat_kanban_items_create` | criar cartao | funil, chave da etapa, posicao e titulo |
| `lionchat_kanban_items_show` | um cartao com historico e funil | traz os selos `can_edit`, `can_move`, `can_delete`, `can_assign` - **confira antes de tentar mexer** |
| `lionchat_kanban_items_update` | editar o cartao | o miolo do cartao se MISTURA com o que ja existe; checklist e historico NAO passam por aqui; numero de conversa cheio e descartado |
| `lionchat_kanban_items_create_1` | mover dentro do MESMO funil | cartao e chave da etapa |
| `lionchat_kanban_items_move` | mover entre funis | responsavel sem acesso ao funil de destino sai do cartao (a resposta diz quem saiu); se o cartao tem responsavel e NENHUM deles tem acesso, o pedido e recusado - administrador passa mesmo assim |
| `lionchat_kanban_items_create_2` | marcar Ganho, Perdido ou reabrir | `won`, `lost` ou `open`, com a pessoa creditada opcional |
| `lionchat_kanban_items_reorder` | gravar a ordem dos cartoes dentro da coluna | |
| `lionchat_kanban_items_destroy` | excluir o cartao | **voce nao apaga nada sem autorizacao** |
| `lionchat_kanban_items_counts` | contagens do cartao | |

## 6. Cartao: buscar e filtrar

| Ferramenta | Para que | Limite |
|---|---|---|
| `lionchat_kanban_items_list` | cartoes de um funil, paginado | **e a unica leitura em que o checklist vem COMPLETO** |
| `lionchat_kanban_items_filter` | o filtro rico da tela: etapas, prioridades, estado, responsavel, faixa de valor, tarefa, canal, etiqueta, oferta, campos personalizados de cartao, contato e conversa, e cinco faixas de data (criacao, entrada na etapa, ultima alteracao, desfecho e agendamento) | **sem paginacao**, teto de 5000 cartoes, ordem fixa (do mais novo para o mais velho). Nao tem filtro por time |
| `lionchat_kanban_items_search` | busca por texto (titulo, descricao, nome ou e-mail do cliente) | |
| `lionchat_search_kanban_items` | busca por texto ou pelo numero do cartao, incluindo campos personalizados e a conversa ligada | ignora campo do tipo secreto |
| `lionchat_contacts_kanban_items_list` | todos os cartoes de UMA pessoa | **e a unica forma de achar cartao a partir do contato** - o cartao nao guarda o contato |

No filtro, as etapas entram pela **chave**, nunca pelo nome. Para "quantos ganhos eu dei no mes passado", use a regua de fechamento junto com o estado ganho.

## 7. Cartao: checklist, anotacoes, responsaveis, anexos

| Ferramenta | Para que |
|---|---|
| `lionchat_kanban_items_kanban_checklist_list` | **a unica leitura confiavel das tarefas** - nas outras o campo volta como resumo de contagem |
| `lionchat_kanban_items_kanban_checklist_create` | criar item; com o mesmo `group_id` e `group_name` em varios itens voce monta um bloco |
| `lionchat_kanban_items_kanban_checklist_create_1` | marcar e desmarcar um item |
| `lionchat_kanban_items_kanban_checklist_update` / `_destroy` / `_group_destroy` | editar item, remover item, remover um bloco inteiro |
| `lionchat_kanban_items_kanban_notes_create` / `_list` / `_update` / `_destroy` | anotacoes internas do cartao |
| `lionchat_kanban_items_kanban_agents_create` / `_destroy` / `_list` | atribuir, remover e listar responsaveis. **Cartao Ganho ou Perdido recusa mexer nos responsaveis** |
| `lionchat_kanban_v2_list_1` / `lionchat_kanban_v2_attachments` / `lionchat_kanban_v2_destroy_1` | listar, enviar e apagar anexos do cartao |
| `lionchat_kanban_v2_create_1` / `lionchat_kanban_v2_destroy_2` | anexos de ANOTACAO (rota separada da dos anexos do cartao) |
| `lionchat_upload_create` | subir um arquivo e receber o endereco dele |

Anexo e envio de arquivo: na pratica, o caminho confortavel e a tela.

## 8. Cartao: conversas ligadas, tarefas, fluxo

| Ferramenta | Para que |
|---|---|
| `lionchat_kanban_items_create_5` | vincular uma conversa ao cartao |
| `lionchat_kanban_items_destroy_1` | desvincular uma conversa |
| `lionchat_kanban_items_create_4` | ligar ou desligar o "recebe automacao" de uma conversa vinculada |
| `lionchat_kanban_items_tasks_list` | tarefas da agenda ligadas ao cartao |
| `lionchat_kanban_items_tasks_list_1` | a proxima tarefa |
| `lionchat_kanban_items_scheduled_messages_list` | mensagens agendadas das conversas do cartao |
| `lionchat_kanban_items_start_flow` | **dispara um fluxo que FALA com o cliente. So com autorizacao para aquele disparo** |

## 9. Acoes em massa

| Ferramenta | Para que |
|---|---|
| `lionchat_kanban_bulk_create` | mover varios cartoes para uma etapa |
| `lionchat_kanban_bulk_create_1` | atribuir um responsavel a varios cartoes |
| `lionchat_kanban_bulk_create_2` | definir a prioridade de varios cartoes |

**Nenhuma delas aceita etiqueta**, e **nao existe exclusao em massa de cartoes**.

## 10. Relatorios

| Ferramenta | Responde |
|---|---|
| `lionchat_kanban_items_list_3` | o relatorio agregado do funil: receita ganha no periodo, cartoes e valor por etapa, previsao ponderada, melhores negocios, cartoes parados e metas. Filtra por vendedor e por time |
| `lionchat_funnels_stage_report` | os numeros de UMA etapa: total, faixa de valores, ganhos e perdidos, distribuicao por prioridade e ranking de vendedores |
| `lionchat_funnels_list_1` | contagem e soma por etapa (o cabecalho da coluna) |
| `lionchat_kanban_items_list_1` | quanto tempo o cartao levou no total |
| `lionchat_kanban_items_list_2` | quanto tempo o cartao passou em cada etapa |

As reguas de data e como fazer dois relatorios baterem estao em `references/relatorios-e-reguas-de-data.md`.

## 11. Importacao e exportacao

| Ferramenta | Para que | Exige |
|---|---|---|
| `lionchat_kanban_items_create_3` | previa da planilha (colunas e amostra) | perfil administrador ou a permissao de exportar e importar |
| `lionchat_kanban_items_import` | importacao em massa, roda em segundo plano | a mesma permissao; teto de 10.000 linhas |
| `lionchat_kanban_items_import_status` | acompanhar o progresso | |
| `lionchat_kanban_items_export` | baixar a planilha do funil | a mesma permissao; a resposta e o arquivo, nao um resultado normal |

**A importacao exige enviar um arquivo de verdade.** Na pratica, o conector nao anexa arquivo: oriente a pessoa a importar pela tela (que ainda converte planilha do Excel para o formato certo antes de enviar).

Tres regras que evitam estrago:

- **So SEIS colunas sao aproveitadas**: titulo, descricao, nome do cliente, e-mail, telefone e etapa (a etapa pode vir pela chave ou pelo nome visivel). Valor, prioridade e estado NAO entram - todo cartao nasce aberto e sem valor. O responsavel so entra por rodizio ou por um mapeamento montado na propria tela de importacao.
- **Telefone sem o codigo do pais (55) e considerado invalido** e o cartao nasce sem contato e sem conversa, em silencio.
- **A importacao dispara a automacao de cartao criado e o evento de etapa em CADA linha.** Importe antes de ligar as automacoes, ou desligue-as durante a carga.

## 12. Envio de conversao para anuncio

Existe uma area do funil que manda a conversao para Meta, Google Analytics e Google Ads, com historico, reenvio e disparo manual pelo cartao. Da para disparar por ETAPA, por GANHO e por PERDIDO - tres gatilhos diferentes.

O campo que mais confunde e a estrategia de valor: mandar o valor do proprio negocio, mandar um valor fixo, ou nao mandar valor. Sem valor, o anuncio otimiza sem faturamento.

Ferramentas: `lionchat_funnels_meta_events_config` para a configuracao; `lionchat_funnels_meta_capi_events_list` / `_show` / `_retry` e as equivalentes de Analytics e Ads para o historico; `lionchat_kanban_items_meta_capi_fire`, `_ga4_mp_fire` e `_google_ads_fire` para o disparo manual a partir de um cartao; `lionchat_kanban_items_recapture_attribution` para reler a origem.

**Isso e area de investimento em anuncio: nao mexa sem pedido claro.** E lembre que a importacao dispara o evento de etapa em cada linha - com essa area ligada, importar base antiga manda milhares de conversoes falsas.

## 13. Nomes que enganam

| Nome | O que ele realmente e |
|---|---|
| `lionchat_kanban_bulk_bulk_actions` | **NAO e do Kanban.** E acao em massa de CONVERSAS e CONTATOS |
| `lionchat_custom_attributes_create` | cria campo de CONTATO ou de CONVERSA. **Nao serve para campo do cartao** - esse e a configuracao do Kanban |
| `lionchat_funnels_create_1` | nao cria funil: transfere os cartoes de um vendedor para outro |
| `lionchat_funnels_create_2` | nao cria funil: transfere cartoes entre etapas |
| `lionchat_kanban_items_create_1` | nao cria cartao: move dentro do mesmo funil |
| `lionchat_kanban_items_create_2` | nao cria cartao: muda o estado (ganho, perdido, aberto) |
| `lionchat_kanban_items_create_3` | nao cria cartao: e a previa da importacao |
| `lionchat_kanban_items_create_4` / `_create_5` | nao criam cartao: mexem nas conversas ligadas |
| `lionchat_kanban_items_destroy_1` | nao apaga cartao: desvincula uma conversa |
| `lionchat_kanban_items_list_1` / `_list_2` | nao listam cartoes: sao relatorios de tempo de UM cartao |
| `lionchat_kanban_items_list_3` | nao lista cartoes: e o relatorio agregado do funil |
| `lionchat_journey_funnel_reports` | e outro recurso (jornada), nao o funil de vendas |

## 14. O que so da para fazer na tela

Diga isso no resumo final, nao no meio do trabalho:

- Subir a planilha de importacao.
- Anexar arquivo no cartao ou na anotacao, e subir a foto da oferta.
- Escrever os modelos de mensagem por etapa (nao ha ferramenta dedicada; a unica alternativa seria regravar todas as etapas de uma vez, o que arrisca apagar etapa).
- Baixar o modelo de planilha de importacao.
- Apagar cartoes em massa (a tela apaga um a um).
- Duplicar o funil inteiro (icone de copiar na lista de funis) - lembrando que as automacoes da copia
  ficam apontando para as chaves de etapa do original e precisam ser refeitas.
- Duplicar um CARTAO nao da em lugar nenhum, nem na tela: o unico jeito e a automacao de etapa com a
  acao de duplicar.
- A preferencia de esconder itens ja concluidos no checklist (e de cada pessoa, nao da conta).
- A largura das colunas e a ordenacao salva de cada coluna (ficam no navegador de cada pessoa).
