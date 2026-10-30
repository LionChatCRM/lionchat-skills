# Relatorios: o que cada numero significa e como montar os seus

Este arquivo e o detalhe da metade RELATORIOS. Leia antes de montar ou explicar qualquer numero.

## Indice

1. Antes de comecar: fuso, permissao e recursos
2. As abas prontas e o que cada uma responde
3. O que cada numero da Visao geral significa
4. Sete armadilhas de leitura
5. O relatorio de Agendamentos
6. Relatorio personalizado: como montar
7. Catalogo dos blocos
8. Campos que todo bloco tem
9. O aviso diario no WhatsApp
10. Diagnostico: "o grupo nao recebeu"
11. Ferramentas do conector
12. Armadilhas
13. So da para fazer na mao, no painel

---

## 1. Antes de comecar: fuso, permissao e recursos

**Todo calculo e cortado no fuso da conta.** Hoje as ferramentas de relatorio (inclusive o bloco do relatorio personalizado) usam o fuso DA CONTA quando `timezone_offset` nao vem - e o certo para quase todos os casos. So mande o fuso para olhar de OUTRO fuso, e entao leia em `lionchat_account_show`, campo `timezone`, e converta para horas: America/Sao_Paulo e -3, Mato Grosso do Sul e -4, e existe cliente fora do Brasil. Em janela de UM dia (hoje ou ontem) um fuso errado e decisivo: tres horas jogam tudo o que aconteceu de madrugada para o dia errado. **Nunca chute -3.**

**Relatorio e area de administrador.** As abas e o relatorio personalizado aceitam tambem um perfil personalizado com a permissao de relatorios; **o aviso diario e SO de administrador de verdade, sem excecao**. Se a chamada for recusada por permissao, diga isso e pare - nao mande o cliente reconectar o conector.

**Abas que dependem de recurso liberado no plano** (confira a lista de recursos em `lionchat_account_show`): Kanban precisa de `kanban_board`; Origem dos Leads e Jornada do Lead precisam de `liontrack`; Formularios precisa de `lead_forms`; SLA precisa de `sla`; a aba CSAT so aparece com `csat_review_notes` ligado; os blocos da agenda da e-Clinica precisam de `eclinica_integration` (liga sozinho quando a conta cadastra a e-Clinica). Bloco de relatorio que depende de recurso nao liberado devolve um erro dizendo isso - nunca um grafico vazio.

**Periodo que ja terminou (19/10/2026):** cada quadro mostra o que aconteceu DENTRO do periodo, nao como as coisas estao hoje. O funil (abas e bloco de etapas) mostra onde cada card estava no ULTIMO dia do periodo; ganho conta no funil onde a venda aconteceu; "Etapa desconhecida" sao cards que mudaram depois e de quem nao ha historico daquela epoca (o historico de etapas comecou em julho de 2026). Periodo que vai ate hoje mostra o estado atual. Assim o relatorio de um mes fechado nao muda quando a equipe mexe nos cards depois.

**A aba CSAT vazia e a aba CSAT ausente sao problemas diferentes.** Ausente = recurso desligado. Vazia = a pesquisa de satisfacao nao esta ligada na CAIXA, ou nenhuma conversa foi RESOLVIDA (a pesquisa so dispara na resolucao). Ligar a pesquisa e escrever o texto dela e na configuracao da caixa.

---

## 2. As abas prontas e o que cada uma responde

Antes de montar relatorio novo, pergunte se uma destas ja responde.

| Aba | Responde | Cuidado |
|---|---|---|
| Visao geral | Panorama de operacao em 8 numeros | Ver secao 3 |
| Conversas | Detalhe das conversas com filtro de caixa, atendente, equipe e etiqueta | Conta pela data de CRIACAO da conversa; mostra MEDIA de tempo |
| Agentes / Equipes / Caixa de Entrada / Etiquetas | Comparar desempenho por pessoa, time, canal ou assunto | Na coluna de conversas por ATENDENTE vale o registro DATADO de atribuicao (28/09/2026): quem ficou com a conversa naquele periodo. Quem saiu da conta e trabalhou no periodo continua com linha propria. Por equipe e por caixa a conta segue pela conversa criada no periodo |
| CSAT | Satisfacao depois da conversa resolvida | Total, distribuicao por nota, comentarios |
| SLA | Cumprimento de prazo | So com o recurso liberado |
| Robos | Quanto a IA resolve sozinha e quanto passa para humano | |
| Agendamentos | Os numeros da agenda | Ver secao 5 |
| Kanban | Funil, produtividade e vendas | O filtro "Data considerada" muda MUITO o numero - e pergunta diferente, nao erro |
| Origem dos Leads | De onde vieram os contatos, em niveis: plataforma, tipo, campanha, conjunto, criativo | Precisa de LionTrack |
| Jornada do Lead | Por quais paginas do site a pessoa passou antes de virar conversa | Janela maxima de 30 dias; precisa de LionTrack |
| Formularios | Desempenho dos formularios publicos | |
| Personalizados | Os relatorios montados pelo proprio cliente | Ver secao 6 |
| Avisos | Os resumos diarios enviados no WhatsApp | Ver secao 9 |

Ha ainda os numeros em tempo real (carga do momento, que ignoram o seletor de periodo), o trafego por hora (horario de pico) e a retrospectiva do ano.

---

## 3. O que cada numero da Visao geral significa

| Cartao | O que conta de verdade |
|---|---|
| Conversas novas | CONVERSAS que nasceram no periodo |
| Conversas reabertas | Quantas vezes uma conversa foi reaberta. Mede retrabalho - ha conta com mais reaberturas do que conversas novas |
| Mensagens recebidas | MENSAGENS, nao conversas. Uma conversa pode ter dezenas |
| Aguardando agora | Foto de AGORA (abertas, pendentes e adiadas). **Ignora o periodo escolhido** |
| Tempo ate a 1a resposta | MEDIANA, nao media |
| Resolucoes no periodo | ACOES de resolver, nao conversas. Conversa criada antes conta; conversa reaberta e resolvida de novo conta outra vez |
| Satisfacao (CSAT) | PERCENTUAL de notas 4 e 5 sobre o total de respostas. **Nao e a nota media**. Mostra um traco quando ninguem respondeu |
| Tempo ate resolver | MEDIANA. Mede a idade da conversa, nao o tempo de trabalho da equipe |

**A taxa de resolucao em percentual foi REMOVIDA de proposito**: ela dividia acoes de resolucao por conversas nascidas na janela, universos diferentes, e passava de 100 por cento em contas reais (uma delas deu 678 por cento). Nao invente esse numero de volta.

---

## 4. Sete armadilhas de leitura

1. **Media e mediana nao sao o mesmo numero, e as telas usam as duas.** A Visao geral mostra a MEDIANA nos dois cartoes de tempo; o relatorio de Conversas mostra a MEDIA. Uma conversa antiga respondida hoje entra na media com toda a espera acumulada. Medido numa conta real: media de 164,6 HORAS contra mediana de 1,1 HORA no mesmo periodo. Ao responder "quanto tempo levamos para responder", use a mediana e diga que e a mediana.
2. **Resolucoes conta acoes, nao conversas.** Nunca compare com "conversas novas" para tirar percentual.
3. **O total de uma tabela nao e a soma das linhas.** E contagem propria por metrica. Quem saiu da conta mas trabalhou no periodo ganha linha propria (desde 26/09/2026), com o nome guardado - antes ele sumia das linhas e ficava so no total.
4. **Cem por cento de cumprimento de prazo pode ser ausencia de dado.** Conta sem nenhum prazo aplicado devolve 100. Olhe o campo de total antes de anunciar. Vale igual para satisfacao: o total separa "ninguem respondeu" de "zero legitimo".
5. **"Quantos estao na etapa agora" e "quantos entraram na etapa no periodo" sao perguntas diferentes.** Cartao que entrou e depois saiu SOME da primeira. Medido: 25 por cento de diferenca. Para "quantos aconteceram", use o bloco de entradas na etapa - com o limite de que esse historico so existe desde julho de 2026 e guarda 365 dias.
6. **Ligar a contagem so em horario de atendimento pode zerar tudo.** Se nenhuma caixa tem horario de atendimento configurado, os tempos voltam vazios. E evento gravado antes de julho de 2026 guarda um zero que puxa a media para baixo: periodo que atravessa essa data mistura os dois e nao e confiavel.
7. **Data tem formato.** As ferramentas de conversas, de satisfacao e de prazo aceitam hoje: numero em segundos (o mais seguro), em milissegundos, ISO 8601 (sem fuso vale a hora da conta) e data pura AAAA-MM-DD (o dia INTEIRO no fuso da conta). Texto ilegivel e RECUSADO com 400 e a explicacao do formato - nunca mais devolve zero com cara de sucesso. As ferramentas de Agendamentos e de Origem dos Leads aceitam data normal. Na duvida, mande segundos.

---

## 5. O relatorio de Agendamentos

Ferramenta: `lionchat_booking_reports`. Unidade = o agendamento.

- **Periodo padrao: 30 dias para TRAS e 30 para FRENTE** - agenda tem futuro.
- Devolve: total, pendentes, concluidos, cancelados, adiados, compareceu (`attended`), faltou (`no_show`) e a taxa de comparecimento; a linha do tempo; as quebras por servico, por profissional e por origem; e o bloco `treatments` dos programas de sessoes.
- **A situacao vem do compromisso na Agenda**, nao do registro da reserva. Isso ja esta resolvido dentro da ferramenta - por isso ela e o caminho certo para contar concluidos e cancelados. A consulta aparece no dia em que esta na Agenda: arrastou para outro dia, muda de dia no relatorio.
- **A taxa de comparecimento (desde 15/09/2026) e compareceu dividido por compareceu mais faltou**, pelas marcas de presenca. Periodo SEM nenhuma marca cai na regra antiga: concluidos dividido por concluidos mais cancelados. O campo `attendance_basis` diz qual valeu (`attendance` ou `status`) - diga isso ao dono. Os pendentes ficam FORA do denominador de proposito. **Nao recalcule com o total** - voce inventaria um numero pior que o real. A taxa nao muda quando se escolhe uma situacao no filtro.
- **Adiado aparece separado**, e nao somado aos pendentes: adiar e uma acao deliberada da equipe e some do radar se for misturada.
- A **origem do agendamento** vem do rastreamento da pagina publica. Agendamento marcado pela equipe ou pela IA nao tem origem - e por isso que parte deles aparece sem essa informacao.
- **Programas (`treatments`):** em andamento e atrasados (medidos no fim do periodo), comecaram, concluidos, encerrados, sessoes previstas, marcadas, aconteceram, faltaram e nao marcadas (previsao), com quebra por tipo. O filtro de servico vale; o de profissional e o de situacao nao.
- **Conta que nao usa Booking** (14/10/2026): `source=agenda` troca a fonte para os compromissos marcados a mao na Agenda (ligacao, reuniao, videochamada e tipos da conta; tarefa comum e evento do Google ficam fora). Nesse modo o tipo vai em `task_type`, a origem vem vazia e os programas vem nulos.

---

## 6. Relatorio personalizado: como montar

Fica em Relatorios, aba Personalizados. **So administrador, ou perfil personalizado com a permissao de relatorios.**

Tetos: **12 blocos por relatorio** e **50 relatorios por conta**.

O caminho seguro, em ordem:

1. Leia o fuso da conta (para conferir o corte do dia).
2. **Teste cada bloco antes de salvar** com `lionchat_custom_dashboards_preview_widget` (o fuso da conta vale sozinho; so mande `timezone_offset` para olhar outro fuso). Essa ferramenta calcula sem salvar nada e e a que responde pergunta de relatorio que nenhuma outra cobre.
3. Mostre o numero ao dono e confirme que e a pergunta dele.
4. Salve com `lionchat_custom_dashboards_create`, com `confirm: true` e um `title` em TODO bloco.
5. Para alterar: leia com `lionchat_custom_dashboards_list`, altere a lista e devolva-a COMPLETA. **A lista enviada SUBSTITUI a anterior** e nao ha historico. Essa listagem nao tem paginacao e pode vir CORTADA numa conta com muitos relatorios cheios; se vier o aviso de corte, NAO use - leia aquele relatorio sozinho com `lionchat_custom_dashboards_show`, que e o caminho seguro.

**O catalogo do servidor e fechado.** Tipo, campo ou valor fora dele e recusado com o motivo em texto. Mas **campo com nome errado e DESCARTADO EM SILENCIO**: o bloco salva, calcula, e o recorte que o cliente pediu simplesmente nao e aplicado. Se um valor "nao fez efeito", quase sempre e o nome do campo.

**Alguns tipos de bloco podem nao aparecer na ferramenta de testar**, dependendo da versao do conector, e mesmo assim serem aceitos ao salvar. Leia a lista de campos da propria ferramenta. Quando nao der para testar antes, salve e confira o numero com `lionchat_custom_dashboards_widget_data`.

Na tela existe ainda um assistente que monta o bloco a partir de uma frase em portugues - ele usa a chave de IA da conta. Sem chave, ele nao responde (mesma causa do aviso diario cair no texto padrao).

---

## 7. Catalogo dos blocos

| Tipo | Responde | Campos proprios | Desenho | Exige |
|---|---|---|---|---|
| `conversations_timeseries` | Como um numero anda dia a dia | `metric` (conversations, incoming_messages, outgoing_messages, resolutions, reopens, avg_first_response_time, avg_resolution_time), `bucket` (day, week, month), `scope_type` (agent, inbox, team) com `scope_id` | linha ou barra | - |
| `entity_ranking` | Comparar atendentes, caixas, equipes ou etiquetas | `dimension` (agent, inbox, team, label), `metric` (conversations, resolutions, avg_first_response_time, avg_resolution_time) | barra ou tabela | - |
| `csat_summary` | Satisfacao no periodo | so o periodo | cartao | - |
| `sla_summary` | Taxa geral de cumprimento de prazo | so o periodo | cartao | `sla` |
| `funnel_stages` | Quantos cartoes ESTAO em cada etapa | `funnel_id`, `date_basis` (created, moved, closed, any), `status` (all, open, won, lost), `measure` (count, value, both) | barra ou tabela | `kanban_board` |
| `stage_entries` | Quantos cartoes ENTRARAM na etapa no periodo | `funnel_id`, `measure` (count, value) | barra ou tabela | `kanban_board` |
| `stage_conversion` | Percentual de conversao entre etapas, calculado pelo servidor | `funnel_id`, a lista de etapas do numerador e a do denominador | cartao | `kanban_board` |
| `lead_origin` | De qual plataforma vieram os leads | so o periodo | barra ou pizza | `liontrack` |
| `agent_report` | Tabela de produtividade | `dimension` (agent, team, inbox), `columns` com as metricas, `scope_type` mais `scope_id` | tabela | - |
| `calls_report` | Tabela de ligacoes | `dimension` (agent, inbox), `scope_type` inbox OU team com `scope_id`, `direction_split` | tabela | - |
| `eclinica_no_show` | Faltas da agenda da e-Clinica | `dimension` (unit, operator, date), chavinhas `no_show_over_scheduled` e `only_first_visit` | tabela | `eclinica_integration` |
| `eclinica_conversion` | De quem foi atendido, quantos viraram tratamento | `dimension` (unit, operator), chavinha `count_retorno_as_consulta` | tabela | `eclinica_integration` |

Notas por tipo:

- **Os dois tempos vem em SEGUNDOS.** Converta antes de mostrar ao cliente.
- **`funnel_stages` vem na ordem das ETAPAS**, nunca ordenado por valor. E a leitura certa de um funil.
- **`measure` com quantidade E valor juntos so vale em tabela** - em grafico de barra e recusado, porque sao dois numeros por etapa.
- Combinacoes uteis do `funnel_stages`: `date_basis` fechado mais `status` ganho responde "quantos foram GANHOS no periodo".
- **`stage_conversion` e o unico percentual confiavel** para o aviso diario, porque quem calcula e o servidor. Cartao que passou por duas etapas do denominador conta UMA vez.
- **`agent_report`**: as colunas saem de uma lista fechada - leads, atendidos, resolvidas, abertas, pendentes, adiadas, contagem por etiqueta e conversao, e (so com dimensao `agent`) primeira resposta, tempo de resposta, ligacoes feitas, taxa de ligacao (sobre leads ou atendidos), ligacoes recebidas e tempo falado. A contagem por etiqueta exige dizer qual etiqueta; a conversao exige um denominador e uma lista de etiquetas como numerador. Leads por atendente = conversas ATRIBUIDAS a ele no periodo; quem saiu da conta e trabalhou no periodo continua com linha. "Resolvidas", "Abertas", "Pendentes" e "Adiadas" mostram como as conversas atendidas no periodo estao HOJE. Aqui `scope_type` tem sentido diferente: agente e equipe recortam QUEM aparece nas linhas; caixa recorta o que aquelas pessoas fizeram NAQUELA caixa. Caixa junto com dimensao equipe e recusado.
- **`calls_report`**: colunas padrao sao ligacoes, atendidas, nao atendidas, nao concluidas, tempo falado e media por ligacao (os tempos saem por extenso, tipo `1h07`). Ligacao sem atendente ganha uma linha propria. `direction_split: true` troca as colunas por Feitas x Recebidas (opt-in; a feita pelo aparelho do VTCall conta desde 11/09/2026, so com ramal cadastrado). `scope_type: team` (so com dimensao `agent`) mostra so as linhas de quem e da equipe; a linha "Sem responsavel" continua. O recorte por caixa depende de a origem da ligacao gravar a caixa - em algumas plataformas de telefonia isso nao acontece e o bloco esvazia.
- **`eclinica_no_show`** e **`eclinica_conversion`** leem a agenda da e-Clinica (por unidade, por operador ou por dia). As chavinhas sao regra de negocio do CLIENTE - pergunte: `no_show_over_scheduled` (falta sobre todos os agendados, em vez de sobre quem teve desfecho; nasce desligada), `only_first_visit` (so consulta inicial; nasce desligada) e `count_retorno_as_consulta` (retorno conta como consulta na base; nasce ligada). Consulta remarcada aparece no dia novo; falta ou atendimento de antes da remarcacao continua no dia em que aconteceu.
- **`funnel_stages` em periodo que ja terminou** mostra onde os cards estavam no ultimo dia do periodo (19/10/2026), nao a etapa de hoje.
- **`entity_ranking` nao tem recorte**: a dimensao ja e a comparacao.

---

## 8. Campos que todo bloco tem

| Campo | O que faz | Valores |
|---|---|---|
| `title` | O nome do bloco no cabecalho. E o que distingue quatro blocos de funil no mesmo relatorio | Texto ate 80 caracteres. **De um titulo a TODO bloco** |
| `time_range` | O periodo | hoje, ontem, ultimos 7 dias, ultimos 30 dias, este mes, mes passado, ou periodo escolhido com data de inicio e fim (todo tipo de bloco aceita, desde 19/08/2026) |
| Periodo escolhido | Janela sob medida | Exige as duas datas; **a janela nao pode passar de 92 dias** |
| `chart_type` | Como o bloco e desenhado | conforme o tipo; omitido usa o padrao |
| `timezone_offset` | O fuso em que o dia e cortado | horas. Opcional: ausente usa o fuso DA CONTA. So mande para olhar de outro fuso |
| Chavinhas por tipo | `direction_split` (ligacoes), `no_show_over_scheduled`, `only_first_visit`, `count_retorno_as_consulta` (e-Clinica) | true ou false; o tipo errado ignora |
| Largura e altura | O tamanho na grade | 1 a 4 colunas; altura normal ou alta |
| Seguir o periodo do relatorio | Se o bloco obedece o calendario do topo ou fica no periodo proprio | ligado por padrao |

**Truque util**: `lionchat_custom_dashboards_widget_data` recalcula um bloco JA SALVO em outra janela sem mexer no que o cliente ve. E assim que se monta um resumo de ontem a partir de um relatorio que o cliente deixou salvo em 30 dias.

---

## 9. O aviso diario no WhatsApp

Fica em Relatorios, aba Avisos. Manda todo dia, na hora escolhida, o resumo em TEXTO de um relatorio personalizado num grupo ou conversa de WhatsApp.

Regras do sistema:

- **Depende de um relatorio personalizado ja montado.** E ele que vira o texto.
- **A caixa PRECISA ser WhatsApp por QR Code.** A oficial e recusada (fora da janela de 24 horas ela so manda modelo aprovado).
- **A conversa de destino precisa ser da MESMA caixa.**
- **Teto de 10 avisos por conta.** So administrador.
- **A janela dos numeros e ONTEM**, no fuso da conta. Com "so dias uteis", o aviso de segunda-feira traz sexta, sabado e domingo.
- Hora de envio de 0 a 23, no fuso da CONTA. Todo dia ou so dias uteis.
- Tolerancia de 3 horas: se o disparo perder o horario, sai atrasado; passou disso, fica marcado como perdido e tenta amanha.
- **A mensagem entra na conversa como mensagem de SISTEMA.** Automacoes e fluxos ativos NAQUELA conversa reagem a ela. Ela nao conta como resposta de atendente e nao mexe em prazo. Avise o cliente antes de escolher um grupo que tenha automacao rodando.

O caminho seguro, sempre nesta ordem:

1. Crie **DESLIGADO** (`enabled: false`).
2. Rode `lionchat_report_alerts_preview` e leia o texto com `lionchat_report_alerts_preview_status`. A previa e assincrona: normalmente sai em segundos, pode passar de dois minutos, e o resultado fica guardado por 5 minutos. **O aviso nem precisa existir** para gerar previa - basta o relatorio.
3. Mostre o texto ao dono e peca aprovacao.
4. **So entao ligue** com o update.

**Criar ja ligado dentro da janela do dia dispara no proximo passo de hora** e o grupo do cliente recebe um texto que ninguem validou. Aconteceu de verdade.

**Sobre o texto escrito pela IA** (o campo de instrucoes, ate 2.000 caracteres):

- Sem instrucoes, ou sem chave de IA na conta, sai o texto padrao do sistema - fiel, bloco a bloco.
- **A conferencia de numeros REPROVA qualquer numero que nao exista no relatorio.** A IA nunca calcula: percentual, soma e media inventados fazem o aviso cair no texto padrao.
- **Nao peca para renomear ou reorganizar secoes.** Caso real: a IA trocou o rotulo de um numero e o grupo leu "novos leads" com o numero do bloco de atendentes.
- A instrucao que funciona: mantenha os titulos e todos os numeros exatamente como estao, nao renomeie secoes, nao calcule nada, inclua todos os blocos.
- Quer percentual no aviso? Ponha um bloco de conversao entre etapas no relatorio. Ai o numero vem pronto do servidor.

`lionchat_report_alerts_send_now` dispara o aviso AGORA como teste - e **ENVIA DE VERDADE** no grupo do cliente. Exige autorizacao nomeando o destino. Nao consome o envio automatico do dia, funciona mesmo com o aviso desligado, e tem trava de 60 segundos (nunca repita a chamada).

---

## 10. Diagnostico: "o grupo nao recebeu"

`lionchat_report_alerts_deliveries` guarda 90 dias, com o texto exato de cada envio e o motivo de cada falha.

| Estado | O que significa | O que fazer |
|---|---|---|
| enviado | Saiu do sistema | Confira o recibo do WhatsApp |
| confirmado | Entregue, com recibo | Nada |
| recusado | O canal recusou, e o motivo vem escrito | Leia o motivo: caixa desconectada, conversa apagada, grupo do qual o numero saiu |
| perdido | O horario passou da tolerancia de 3 horas | Costuma ser fila cheia ou atualizacao no horario do disparo; tenta sozinho amanha |
| erro | Falhou antes de montar a mensagem | Relatorio apagado, recurso desligado, chave de IA sem credito |

Confira tambem em `lionchat_report_alerts_show` a hora prevista do proximo envio e se o aviso esta mesmo ligado.

---

## 11. Ferramentas do conector

| Ferramenta | Para que serve |
|---|---|
| `lionchat_account_show` | O fuso e a lista de recursos. Primeiro passo sempre. |
| `lionchat_reports_summary` | Resumo geral: contagens e tempos em media E mediana, com comparacao automatica com o periodo anterior. Tempos em SEGUNDOS, datas em numero. |
| `lionchat_reports_list` a `_list_4` | Resumo por atendente, equipe, caixa, etiqueta e tipo de canal. |
| `lionchat_reports_list_5` | Serie no tempo de UMA metrica, com agrupamento e fuso. |
| `lionchat_reports_drilldown` | As CONVERSAS que formam UM ponto da serie (o mesmo que clicar na barra). Repita `metric`, `group_by`, `type`, `id`, `timezone_offset` e `business_hours` da serie e passe o `timestamp` do ponto em `bucket_timestamp`. `meta.hidden_count` = conversas que a pessoa nao pode abrir: diga isso, nao conclua que faltam. 25 por pagina. |
| `lionchat_reports_list_6` / `_list_14` | Resumo e detalhe do robo. |
| `lionchat_reports_list_7` a `_list_10`, `_list_12` | Exportar planilha de atendentes, caixas, etiquetas, equipes e resumo de conversas. |
| `lionchat_reports_list_11` | A lista de conversas do relatorio. |
| `lionchat_reports_list_13` | Trafego por hora - o horario de pico. |
| `lionchat_reports_list_15` | Retrospectiva do ano. |
| `lionchat_reports_list_16` / `_list_17` | Numeros em tempo real (nao obedecem o periodo). |
| `lionchat_csat_metrics` | Satisfacao agregada: total de respostas, distribuicao por nota e pesquisas enviadas. **A nota media e a taxa de resposta precisam ser calculadas por voce.** |
| `lionchat_csat_list` / `lionchat_csat_download` | Respostas individuais e exportacao. |
| `lionchat_sla_metrics` / `_list` / `_download` | Cumprimento de prazo. |
| `lionchat_booking_reports` | O relatorio de Agendamentos. |
| `lionchat_lead_origin_reports` | Origem dos leads, com abertura por plataforma, tipo, campanha, conjunto e criativo. |
| `lionchat_journey_funnel_reports` | Jornada no site. Janela maxima de 30 dias. |
| `lionchat_funnels_stage_report` | Numeros de UMA etapa do funil, com regua de data escolhida (em periodo passado, a situacao no fim do periodo). |
| `lionchat_kanban_items_shared_sales` | A lista "Vendas compartilhadas" do relatorio do Kanban: cards GANHOS no periodo com co-participante diferente do vendedor gravado, com `date_basis`, filtros de pessoa/time/etiqueta/motivo e paginacao por `nextCursor`. Quem nao enxerga parte do Kanban recebe essas vendas so como `hiddenCount`. |
| `lionchat_funnels_open_counts` / `lionchat_kanban_items_counts` | Contagens rapidas de cartoes. |
| `lionchat_campaigns_statistics` / `lionchat_campaigns_flow_report` | Resultado de campanhas. |
| `lionchat_lead_forms_stats` | Numeros de um formulario publico. |
| `lionchat_custom_dashboards_list` / `_show` | Os relatorios personalizados com os blocos dentro. E daqui que sai o numero de cada bloco. |
| `lionchat_custom_dashboards_preview_widget` | Monta e calcula um bloco NA HORA, sem salvar. |
| `lionchat_custom_dashboards_widget_data` | Calcula um bloco JA SALVO, e aceita trocar so a janela. |
| `lionchat_custom_dashboards_create` / `_update` / `_destroy` | Criar, alterar e apagar o relatorio. Exigem confirmacao. |
| `lionchat_report_alerts_list` / `_show` | Os avisos, com o proximo envio previsto. |
| `lionchat_report_alerts_create` / `_update` / `_destroy` | Criar, editar (inclusive ligar e desligar) e apagar. Exigem confirmacao. |
| `lionchat_report_alerts_preview` / `_preview_status` | A previa do texto, sem enviar nada. |
| `lionchat_report_alerts_send_now` | **ENVIA DE VERDADE** agora, como teste. |
| `lionchat_report_alerts_deliveries` | O historico de envios com o motivo de cada falha. |
| `lionchat_reporting_events_list` / `lionchat_conversations_reporting_events` | Os eventos crus que alimentam os relatorios - para investigar numero que nao bate. |

---

## 12. Armadilhas

### Salvar o relatorio mandando so o bloco novo

**Voce faz:** manda um bloco no update.
**Acontece:** todos os outros somem. Nao ha historico.
**Como sair:** leia a lista inteira, altere e devolva completa, preservando o numero de cada bloco que ja existia. Se a leitura veio cortada, nao grave.

### Chutar o fuso

**Voce faz:** manda `timezone_offset: -3` sem conferir a conta.
**Acontece:** numa conta em outro fuso o dia e cortado errado e o numero nao bate com a tela.
**Sinal:** cliente diz "aqui na tela da outro numero".
**Como sair:** omita o fuso (as ferramentas usam o da conta) ou leia o fuso da conta antes de mandar.

### Campo com nome errado

**Voce faz:** escreve o nome de um campo diferente do que aquele tipo aceita.
**Acontece:** e descartado calado. O bloco salva e calcula sem o recorte.
**Sinal:** "o filtro nao fez efeito".
**Como sair:** confira a lista de campos daquele tipo na secao 7.

### Usar "quantos estao na etapa" para responder "quantos aconteceram"

**Acontece:** cartao que entrou e saiu some. Medido: 25 por cento de diferenca.
**Como sair:** use o bloco de entradas na etapa, lembrando que o historico so existe desde julho de 2026.

### Somar as linhas de uma tabela

**Acontece:** nao fecha com o total, e nao e erro: o total e contagem propria por metrica.

### Anunciar 100 por cento de prazo cumprido

**Acontece:** conta sem nenhum prazo aplicado devolve 100. E ausencia de dado.
**Como sair:** olhe o campo de total antes de dizer qualquer coisa.

### Criar o aviso diario ligado

**Acontece:** dispara no proximo passo de hora e o grupo recebe texto nao validado.
**Como sair:** desligado, previa, aprovacao, ligar.

### Escolher caixa oficial para o aviso

**Acontece:** recusado, com a explicacao em texto.
**Como sair:** so caixa de WhatsApp por QR Code.

### Pedir para a IA do aviso calcular ou renomear

**Acontece:** a conferencia reprova e o aviso cai no texto padrao; renomear ja fez a IA trocar rotulo de numero.
**Como sair:** peca fidelidade; percentual so por bloco de conversao entre etapas.

### Comparar tempo de duas telas diferentes

**Acontece:** uma mostra mediana e a outra media. A diferenca chega a ser de 164 horas contra 1,1 hora.
**Como sair:** diga sempre qual das duas voce esta mostrando.

### Mandar data em formato ilegivel nas ferramentas de conversa, satisfacao e prazo

**Acontece:** a chamada e RECUSADA com 400 e a explicacao do formato. Data pura AAAA-MM-DD vale o dia inteiro no fuso da conta.
**Como sair:** segundos (o mais seguro), ISO 8601 ou AAAA-MM-DD.

### Ler a taxa de comparecimento sem olhar a base

**Acontece:** a taxa pode vir das marcas de presenca ou, sem nenhuma marca no periodo, de concluidos x cancelados.
**Como sair:** leia `attendance_basis` e diga ao dono qual regra valeu.

---

## 13. So da para fazer na mao, no painel

- Ligar a pesquisa de satisfacao numa caixa e escrever a mensagem dela.
- Usar o assistente da tela que monta bloco de relatorio conversando (ele precisa da chave de IA da conta).
- A aba Formularios: nao ha ferramenta do conector que devolva esse relatorio; existe so o numero do proprio formulario, que e outra coisa.
- Escolher com quem um relatorio personalizado e compartilhado: nao existe. Quem enxerga e quem tem permissao de relatorios na conta.
- Liberar um recurso do plano que esconde uma aba (Kanban, LionTrack, prazo, satisfacao, formularios, Agenda): quem libera e o suporte.
