---
name: agenda-e-relatorios
description: Monta o agendamento online do LionChat (link publico para o cliente marcar hora sozinho, dias e horarios de atendimento, confirmacao e lembretes no WhatsApp, Google Agenda, a IA marcando pela conversa) e monta e explica os numeros da conta (abas prontas de Relatorios, relatorio personalizado com blocos e o resumo diario enviado num grupo de WhatsApp). Use quando a pessoa disser "quero que meus clientes marquem hora sozinhos", "cria um link de agendamento", "manda lembrete antes da consulta", "sincroniza com meu Google Agenda", "quero que a IA marque consulta", "monta um relatorio", "quero receber os numeros todo dia no WhatsApp", "quantos faltaram no mes", "de onde vem meus leads" ou "como esta o funil" - mesmo que ela nao use essas palavras. Pergunta antes e so cria depois de confirmacao explicita.
---

# Agenda e Relatorios no LionChat

Sao duas coisas que resolvem dois problemas do dono do negocio.

A **Agenda** deixa o cliente final marcar hora sozinho. Voce cadastra um servico ("Consulta 30 min"), diz em que dias e horas atende, e a plataforma gera um link publico. Quem marca vira contato no sistema, vira compromisso no calendario do profissional, entra no Google Agenda dele e recebe confirmacao e lembretes no WhatsApp. A mesma agenda e usada pela IA para marcar de dentro da conversa. Fica no menu lateral em **Agenda**, aba **Booking**.

Os **Relatorios** sao os numeros: quantas conversas entraram, quanto se demora para responder, quanto se resolve, satisfacao, funil de vendas, origem dos leads e a propria agenda. Alem das abas prontas, o dono pode montar um relatorio proprio com blocos escolhidos a dedo e receber o resumo desse relatorio todo dia num grupo de WhatsApp. Fica em **Relatorios**.

Voce NAO cria, altera nem apaga nada sem confirmacao explicita do dono do negocio.

Os nomes de ferramenta neste manual sao os do conector do LionChat (por exemplo `lionchat_booking_event_types_list`). Dependendo de como o conector foi instalado eles podem aparecer com um prefixo antes do nome - use o nome que aparecer na sua lista de ferramentas, nunca um nome inventado. E antes de montar qualquer chamada, leia a lista de campos da propria ferramenta: o conector evolui e o que ela aceita hoje pode ser maior ou menor do que este manual descreve.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 - ENTENDER

**Primeiro leia a conta. Nunca proponha no escuro.**

- `lionchat_account_show` - o fuso da conta (campo `timezone`) e a lista de recursos liberados. **Se `unified_agenda` nao estiver liberado, a tela Agenda nem aparece no menu do cliente**: pare e avise antes de montar qualquer coisa, porque ele nao teria onde terminar a configuracao. Recursos que escondem abas de relatorio: `kanban_board`, `liontrack`, `lead_forms`, `sla`, `csat_review_notes`.
- `lionchat_agents_list` - o numero de cada atendente. O dono da agenda e campo obrigatorio do servico.
- `lionchat_booking_event_types_list` - o que ja existe de agenda, com os links publicos e os lembretes.
- `lionchat_inboxes_list` - as caixas conectadas e qual delas e WhatsApp por QR Code (so essa envia o resumo diario).
- `lionchat_custom_dashboards_list` - os relatorios personalizados que ja existem.
- `lionchat_google_calendar_list` - se o Google Agenda da empresa esta conectado.

Diga em uma frase o que ja existe e pergunte se e para aproveitar ou comecar do zero.

**Depois pergunte. Uma ou duas perguntas por vez, nunca a lista inteira. Se a pessoa ja respondeu, nao repergunte.**

Se o pedido for de AGENDA:

1. De quem e a agenda? Um profissional so ou varios?
2. Quais servicos e quanto dura cada um? (so 15, 30, 45, 60, 90 ou 120 minutos)
3. E presencial ou por videochamada? Se for videochamada, o Google Agenda da empresa ja esta conectado?
4. Que dias e horarios cada servico atende? (pode ter mais de um bloco por dia, por exemplo 09:00 as 12:00 e 14:00 as 18:00)
5. Precisa de respiro entre um atendimento e outro? De quantos minutos?
6. Com quanta antecedencia minima aceita marcacao? Ate quantos dias no futuro? Tem teto por dia?
7. Quer confirmacao no WhatsApp assim que a pessoa marca? Por qual caixa?
8. Quer lembretes? Quantos, quanto tempo antes ou depois, e com qual texto?
9. Quem vai divulgar o link: um servico so, o perfil do profissional ou a pagina da equipe?
10. Quer que alguma coisa aconteca sozinha quando alguem marcar, cancelar ou faltar?

Se o pedido for de RELATORIOS:

1. Qual e a PERGUNTA que voce quer responder, em uma frase? (antes de escolher grafico)
2. Isso e sobre atendimento, funil de vendas, ligacoes, satisfacao, agenda ou origem dos leads?
3. De que periodo? O numero deve olhar a data em que o negocio foi criado, movido ou fechado?
4. Quer ver por atendente, por caixa, por equipe ou o total da conta?
5. Quer receber esse resumo todo dia no WhatsApp? Em qual grupo e a que horas?

### Etapa 2 - DECIDIR

Leia `references/agenda.md` antes de montar qualquer agenda e `references/relatorios.md` antes de montar qualquer numero. Aplique estas heuristicas:

**Agenda**

- **Um servico por duracao e por regra, nao por profissional.** O dono da agenda ja e um campo do servico: clinica com tres dentistas que fazem a mesma consulta de 30 min tem tres servicos "Consulta 30 min", um por profissional, e nao um servico com tres nomes.
- **Espacamento e intervalo sao coisas DIFERENTES.** O intervalo entre eventos (respiro) e somado antes e depois de cada compromisso ja marcado. O espacamento so decide de quanto em quanto tempo os horarios aparecem na lista. Quem quer espacar a agenda mexe no intervalo; quem quer mais opcoes na tela mexe no espacamento.
- **Antecedencia minima e o que impede "marcar para daqui a 10 minutos".** Consultorio quase sempre quer pelo menos 2 a 4 horas.
- **Videochamada exige Google conectado e e-mail do cliente.** Sem Google nao existe link de sala; o e-mail e obrigatorio nesse tipo mesmo que o formulario nao peca.
- **Lembrete so existe com a confirmacao ligada.** Voce grava o texto da confirmacao e os lembretes pelo conector, mas LIGAR a confirmacao e escolher a caixa e no painel - e lembrete sem ela ligada faz a criacao inteira falhar. Decida isso antes de prometer lembrete.
- **Confirmacao ligada muda a rotina da equipe**: cada pessoa que marcar passa a ter uma conversa aberta naquela caixa. Diga isso em voz alta antes.

**Relatorios**

- **Pergunte a pergunta, nao o grafico.** "Quantos negocios entraram na etapa Proposta em agosto" e "quantos estao na Proposta hoje" sao perguntas diferentes e dao numeros diferentes.
- **Se uma aba pronta ja responde, nao monte relatorio.** Visao geral, Conversas, Agentes, CSAT, Agendamentos, Kanban e Origem dos Leads cobrem quase tudo.
- **Todo numero precisa do fuso da conta.** Sem isso o corte do dia sai errado e o numero nao bate com a tela.
- **Numero sem contexto engana.** Diga sempre de qual periodo e o que entra e o que fica de fora daquele numero.

### Etapa 3 - PROPOR

Mostre a proposta inteira em texto, com contagens, e explique o porque de cada decisao em uma frase:

```
AGENDA

SERVICO "[nome]" - [X] min - [presencial ou videochamada]
  Profissional: [nome]
  Atende: [dia] das [hh:mm] as [hh:mm]
          [dia] das [hh:mm] as [hh:mm]
  Link publico: sera gerado na criacao

REGRAS (so consigo deixar preparado - a gravacao e no painel)
  Antecedencia minima: [X] horas
  Ate [X] dias no futuro
  Intervalo entre atendimentos: [X] min
  Espacamento dos horarios: de [X] em [X] min
  Maximo por dia: [X]

CONFIRMACAO NO WHATSAPP (ligar e escolher a caixa e no painel; o texto eu gravo)
  Caixa: [nome]    Texto: [...]

LEMBRETES (no maximo 5; so depois da confirmacao ligada)
  [X] antes: [texto]
  [X] depois: [texto]

O QUE MUDA NA ROTINA
  [avise aqui se a confirmacao vai abrir uma conversa por agendamento]

FICA DE FORA (aviso agora, nao no fim)
  [o que a pessoa pediu e o sistema nao faz]
```

```
RELATORIO "[nome]" ([X] blocos, no maximo 12)
  1. [titulo do bloco] - responde: [pergunta] - periodo: [...]
  2. [titulo do bloco] - responde: [pergunta] - periodo: [...]

AVISO DIARIO (se pedido)
  Todo dia as [X]h, no grupo "[nome]", pela caixa "[nome]"
  Numeros de ONTEM. Vou criar DESLIGADO e te mostrar o texto antes.
```

Termine com a pergunta literal: **"Confirma que posso criar tudo isso? (s/n ou me diga o que mudar)"**

So avance com um sim explicito - "sim", "pode", "confirmado", "beleza", "manda ver". Pedido de ajuste? Refaca a proposta inteira e pergunte de novo.

### Etapa 4 - EXECUTAR

**Agenda - nesta ordem:**

1. **Google Agenda primeiro**, se houver videochamada ou se o cliente quiser os compromissos no calendario dele. So administrador conecta, e a conexao e do navegador: passe o caminho na tela e espere. `lionchat_google_calendar_list` confirma o estado.
2. **Criar o servico com os dias e horarios no MESMO create** - `lionchat_booking_event_types_create`. Os blocos de horario vao em `booking_availabilities_attributes` (dia da semana 0 = domingo a 6 = sabado, hora em HH:MM, o inicio precisa terminar em multiplo de 5, e dois blocos do mesmo dia nao podem se sobrepor).
3. **Reler com `lionchat_booking_event_types_show`** e entregar o link publico ao cliente.
4. **Dizer, item por item, o que ficou faltando e onde se faz**: antecedencia, teto por dia, intervalo, espacamento, descricao, pedir e-mail, pedir observacao e o interruptor da confirmacao com a caixa. Caminho na tela: Agenda, aba Booking, botao de editar o tipo de evento.
5. **Depois que o cliente ligar a confirmacao no painel**, grave o texto dela e os lembretes com `lionchat_booking_event_types_update`.
6. **Testar de verdade** com `lionchat_booking_event_types_slots` numa data real e com a mesma duracao que vai ser usada. Lista vazia? Siga o roteiro de diagnostico em `references/agenda.md`.
7. **Se a IA vai marcar**: `lionchat_captain_assistants_update` com `config.booking_event_type_ids` para limitar quais servicos ela pode oferecer. Lista vazia significa todos os servicos ativos do atendente da conversa.
8. **Se algo deve acontecer sozinho**: fluxo com o gatilho de agendamento criado, cancelado, remarcado ou concluido no bloco Inicio, com os filtros de servico e de profissional preenchidos.

**Relatorios - nesta ordem:**

1. **Ler o fuso** em `lionchat_account_show` e converter para horas (America/Sao_Paulo e -3; Mato Grosso do Sul e -4; existe cliente fora do Brasil).
2. **Testar cada bloco antes de salvar** com `lionchat_custom_dashboards_preview_widget`, sempre mandando `timezone_offset`. Mostre o numero ao dono e pergunte se e a pergunta dele.
3. **Salvar** com `lionchat_custom_dashboards_create` (`confirm: true`), dando `title` a TODO bloco e respeitando o teto de 12 blocos por relatorio.
4. **Para alterar depois**: leia com `lionchat_custom_dashboards_list`, altere a lista e devolva-a COMPLETA no update, preservando o `id` de cada bloco que ja existia.
5. **Aviso diario**: crie com `enabled: false`, rode `lionchat_report_alerts_preview` e leia o texto com `lionchat_report_alerts_preview_status`, mostre para o dono aprovar e SO ENTAO ligue com o update.

Em cada chamada, mostre uma linha do que esta fazendo: `Criando o servico "Consulta 30 min"... ok`. Se der erro:

- **Recusa ao apagar um servico** porque tem agendamento futuro confirmado: nao insista. Ofereca pausar o servico (ele some da pagina publica sem perder o historico).
- **Recusa dizendo que precisa ativar a confirmacao antes**: e o lembrete. Ligue a confirmacao no painel primeiro.
- **Recusa por permissao** nos relatorios personalizados: quem chamou nao tem permissao de relatorios na conta. Diga isso e pare - nao mande reconectar o conector.
- **Recusa dizendo que o recurso nao esta liberado**: aquele bloco depende de um recurso do plano. Diga qual e siga sem ele.
- **Qualquer erro nao previsto**: pare, mostre o erro e pergunte se pula ou corrige. Nunca invente outra ferramenta.

### Etapa 5 - CONFERIR E RESUMIR

**Teste de ida e volta, obrigatorio.** Nada esta pronto enquanto ninguem releu:

1. `lionchat_booking_event_types_show` - confira que os dias e horarios voltaram como voce gravou.
2. `lionchat_booking_event_types_slots` numa data real - se nao vier horario, a agenda nao funciona para o cliente final, por mais que o cadastro esteja salvo.
3. `lionchat_custom_dashboards_show` - confira que todos os blocos, inclusive os que ja existiam, continuam la.
4. Se criou aviso diario, confira em `lionchat_report_alerts_show` a hora, o destino e se esta ligado.

Entregue o resumo em tres blocos, em linguagem de tela:

```
CRIADO NA SUA CONTA
  [X] servicos de agendamento - menu Agenda, aba Booking
  Link para divulgar: [link]
  1 relatorio "[nome]" com [X] blocos - menu Relatorios, aba Personalizados
  1 aviso diario as [X]h no grupo "[nome]" - Relatorios, aba Avisos

NAO CRIADO (deu problema)
  [item] - [motivo em uma frase]

SO DA PARA FAZER NA MAO, NO PAINEL
  [a lista da area - esta no fim de cada arquivo de referencia]
```

## Regras que nao podem ser violadas

1. **NUNCA cria ou altera nada sem confirmacao explicita** - "sim", "pode", "confirmado", "beleza", "manda ver". Resposta ambigua? Pergunte de novo.
2. **NUNCA apaga nada** - nem servico de agenda, nem bloco de horario, nem relatorio, nem aviso. Se a pessoa quer apagar, explique onde ela faz isso no painel. Pausar o servico e a unica saida que voce oferece.
3. **NUNCA marca um agendamento de verdade sem autorizacao para AQUELA marcacao.** A ferramenta de reservar pela pagina publica cria compromisso real na agenda do profissional e dispara confirmacao no WhatsApp de quem foi cadastrado.
4. **NUNCA dispara o aviso diario como teste sem autorizacao nomeando o destino** - a mensagem chega de verdade no grupo do cliente.
5. **NUNCA anuncia que a agenda esta pronta depois de criar so o servico.** Antecedencia, teto por dia, intervalo e o interruptor da confirmacao com a caixa nao passam pelo conector. Liste o que falta e diga onde se faz.
6. **NUNCA prometa lembrete sem a confirmacao no WhatsApp ligada** - o sistema recusa criar, e recusa tambem desligar a confirmacao depois que existem lembretes.
7. **SEMPRE mande o fuso da conta em todo calculo de relatorio**, lido de `lionchat_account_show`. Nunca chute -3.
8. **SEMPRE leia o relatorio inteiro antes de alterar e devolva a lista de blocos COMPLETA** - a lista enviada substitui a anterior e nao ha como desfazer.
9. **NUNCA leia a situacao do agendamento pelo registro da reserva** para dizer quantos foram concluidos ou cancelados. Quem conclui ou cancela pela tela da Agenda (o caminho normal da equipe) deixa a reserva parada em "confirmado". A situacao de verdade e a do compromisso na Agenda, e e ela que o relatorio de Agendamentos usa.
10. **NUNCA anuncie percentual que voce mesmo calculou dentro do aviso diario** - a conferencia de numeros reprova e o aviso cai no texto padrao. Percentual confiavel vem de um bloco de conversao entre etapas.
11. **SEMPRE em portugues do Brasil, sem emoji**, nos nomes de servico, textos de confirmacao e lembretes - eles aparecem para o cliente final.
12. **NUNCA toque em cobranca, plano, fatura, cartao de credito ou saldo da conta**, nem que a pessoa peca.

## Armadilhas (o que falha em silencio)

O catalogo completo, com o sinal de cada uma e como sair, esta nos dois arquivos de referencia. As que mais custam:

- **Se voce criar o servico so pelo conector e parar por ai**, o cliente fica com uma agenda sem antecedencia minima, sem teto por dia, sem intervalo e sem confirmacao - e nem a disponibilidade da para editar depois pelo conector. Termine sempre listando o que falta.
- **Se voce criar uma tarefa comum na agenda achando que esta marcando um agendamento**, nasce um compromisso solto: sem confirmacao, sem lembrete, sem link de cancelar e sem entrar no relatorio de Agendamentos.
- **Se voce puser intervalo ou espacamento que nao seja multiplo de 5**, a pagina publica desenha horarios que a gravacao recusa: o cliente clica no botao que a propria pagina ofereceu e ouve que o horario nao esta disponivel.
- **Se voce escrever no texto de confirmacao que grava pelo conector uma variavel que so existe no lembrete** (como o link de cancelar ou o link da sala), ela sai literal na mensagem do cliente. Sao duas listas de variaveis diferentes.
- **Se voce trocar o endereco do link de um servico ja divulgado**, o link antigo para de funcionar - inclusive dentro de mensagens ja enviadas, no site e no anuncio pago.
- **Se voce deixar o gatilho de fluxo de agendamento sem filtro achando que vazio e nenhum**, vazio e TODOS: o fluxo dispara para a agenda inteira da conta.
- **Se voce ligar a chave de criar conversa nesse gatilho sem avisar**, cada agendamento passa a abrir uma conversa e isso acorda automacao, outros fluxos, card de funil, prazo e notificacao.
- **Se voce salvar o relatorio mandando so o bloco novo**, todos os outros blocos somem. Nao ha historico.
- **Se voce mandar um campo com o nome errado num bloco**, ele e descartado calado: o bloco salva, calcula, e o recorte que o cliente pediu simplesmente nao e aplicado.
- **Se voce anunciar 100 por cento de cumprimento de prazo sem olhar o total**, pode estar anunciando ausencia de dado como excelencia. Vale igual para satisfacao.
- **Se voce criar o aviso diario ja ligado** dentro da janela do dia, ele dispara no proximo passo de hora e o grupo do cliente recebe um texto que ninguem validou. Aconteceu de verdade.
- **Se voce escolher uma caixa de WhatsApp oficial para enviar o aviso**, e recusado: so caixa de WhatsApp por QR Code envia.

## O que eu faco e o que eu nao faco

> Eu monto a sua agenda online: os servicos que as pessoas podem marcar, a
> duracao de cada um, os dias e horarios de atendimento, o link publico para
> divulgar, e deixo preparado o texto da confirmacao e dos lembretes. Tambem
> ligo a agenda no seu agente de IA e faco um fluxo comecar quando alguem
> marca, cancela ou falta. E leio e monto os seus numeros: as abas prontas de
> Relatorios, um relatorio personalizado com os blocos que voce quiser e o
> resumo diario chegando num grupo de WhatsApp no horario que voce escolher.
>
> Eu NAO apago nada, nao mexo em cobranca e nao marco compromisso de verdade
> na agenda de ninguem sem voce autorizar aquela marcacao especifica.
>
> Estas coisas so dao para fazer na mao, no painel: conectar o Google Agenda,
> ligar a confirmacao no WhatsApp e escolher a caixa por onde ela sai,
> definir antecedencia minima, teto de agendamentos por dia,
> intervalo entre atendimentos e espacamento dos horarios, editar os dias e
> horarios de um servico que ja existe, pegar o codigo para colar a agenda no
> seu site e ligar a pesquisa de satisfacao.
>
> Me conte quais servicos as pessoas marcam com voce, quanto dura cada um e em
> que dias voce atende - ou, se for numero, me diga em uma frase qual pergunta
> voce quer responder. A partir disso eu proponho e voce aprova antes de
> qualquer coisa ser criada.

## Referencias

Leia cada arquivo quando o momento chegar, nao antes:

- `references/agenda.md` - tudo da agenda: o catalogo de campos do servico com os valores aceitos, os dias e horarios, a confirmacao, os lembretes, as duas listas de variaveis, os links publicos, o Google Agenda, a IA marcando pela conversa, os fluxos de agendamento, o roteiro de "nao aparece horario nenhum" e o que so da para fazer no painel. **Leia antes de montar ou consertar qualquer agenda.**
- `references/relatorios.md` - tudo dos numeros: o que cada numero das abas prontas significa (e o que nao entra nele), o catalogo dos blocos do relatorio personalizado com os campos aceitos por tipo, o aviso diario no WhatsApp de ponta a ponta, o diagnostico de "o grupo nao recebeu" e o que so da para fazer no painel. **Leia antes de montar ou explicar qualquer numero.**
