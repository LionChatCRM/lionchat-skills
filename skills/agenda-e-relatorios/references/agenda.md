# Agenda: o cliente marca hora sozinho

Este arquivo e o detalhe da metade AGENDA. Leia antes de montar ou consertar qualquer agendamento.

## Indice

1. Antes de comecar: o que precisa estar ligado
2. As pecas e como se encaixam
3. Catalogo de campos do servico
4. Dias e horarios de atendimento
5. Confirmacao no WhatsApp
6. Lembretes
7. Variaveis dos textos: duas listas DIFERENTES
8. Como o horario livre e calculado
9. Roteiro de "nao aparece horario nenhum"
10. Os links publicos e o codigo para o site
11. Google Agenda
12. A IA marcando pela conversa
13. Fluxos que comecam num agendamento
14. Os oito campos gravados na ficha do contato
15. A situacao do compromisso: reserva contra Agenda
16. Ferramentas do conector
17. Armadilhas
18. So da para fazer na mao, no painel

---

## 1. Antes de comecar: o que precisa estar ligado

**O recurso da Agenda unificada nasce DESLIGADO.** Sem ele, o item Agenda nao aparece no menu lateral e a tela nao abre - inclusive em conta de whitelabel. Como a aba Booking (onde se cadastra o servico, a disponibilidade, a confirmacao e os lembretes) mora dentro dessa tela, criar o servico pelo conector numa conta sem o recurso deixa o cliente sem onde terminar a configuracao. Confira em `lionchat_account_show` e, se faltar, avise antes de qualquer coisa: quem libera e o suporte.

**Liberar minha agenda.** Cada atendente tem um interruptor proprio que diz se os outros podem ver e marcar na agenda dele. Sem ele ligado, atendente comum nao enxerga o servico nem os horarios de outro profissional (administrador enxerga todos). E a explicacao de "sumiu o servico da fulana da minha lista" e "nao consigo marcar para ela" em clinica com varios profissionais.

**Google Agenda** so e conectado por administrador, e e uma conexao por conta LionChat. Sem ela nao existe link de sala para servico de videochamada.

---

## 2. As pecas e como se encaixam

| Peca | Nome na tela | O que e |
|---|---|---|
| Servico | Tipo de Evento | O modelo de compromisso que o cliente marca. Tem dono, duracao, regras e link proprio. |
| Dias e horarios | Disponibilidade | Os blocos de "atendo neste dia, desta hora ate esta hora" daquele servico. |
| Confirmacao | Confirmacao WhatsApp | A mensagem que sai na hora em que a pessoa marca. |
| Lembretes | Follow Ups | Mensagens programadas em relacao ao horario marcado (vespera, 10 minutos antes, dia seguinte). |
| Compromisso | o cartao na Agenda | O que a equipe ve e move no calendario. E dele que sai a situacao real. |

Cada marcacao cria, de uma vez: um contato (ou reaproveita um existente), o compromisso na agenda do profissional e o registro da reserva, com links para o cliente cancelar ou remarcar sozinho.

**Os tres caminhos de marcar usam o MESMO motor**: a pagina publica, a IA dentro da conversa e a marcacao feita pela equipe no painel. Uma trava no banco impede dois agendamentos no mesmo horario do mesmo profissional.

---

## 3. Catalogo de campos do servico

A coluna "Pelo conector" diz o que a ferramenta de criar costuma aceitar hoje. **Leia sempre a lista de campos da propria ferramenta antes de montar a chamada** - o conector evolui.

| Campo na tela | O que faz | Valores aceitos | Pelo conector |
|---|---|---|---|
| Titulo | O nome que o cliente ve na pagina e no cartao da agenda | Texto, ate 100 caracteres. Obrigatorio | sim |
| Agente responsavel | De quem e a agenda. Os horarios sao calculados contra os compromissos DESTE profissional | Numero do atendente. So administrador aponta para outro | sim |
| Duracao | Quanto dura o compromisso | SO 15, 30, 45, 60, 90 ou 120 minutos. Qualquer outro numero e recusado | sim |
| Tipo | Reuniao (presencial) ou Call / Meet (videochamada, que gera link de sala) | `meeting` ou `video_call` | sim |
| Endereco do agendamento | O pedaco editavel do link publico | Minusculas, numeros e hifen, 3 a 60, nunca comecando ou terminando com hifen. Vazio na criacao sorteia um | sim |
| Fuso horario do local de trabalho | Em que fuso os horarios sao montados e conferidos | Nome de fuso, por exemplo America/Sao_Paulo. Vazio herda do atendente, depois da conta | sim |
| Ativo | Pausa o servico sem apagar. Pausado some da pagina publica | ligado ou desligado | sim |
| Cor | A cor do compromisso no calendario e nas fatias do relatorio | Cor em formato hexadecimal. Vazio herda a cor do atendente | sim |
| Descricao | Explicacao abaixo do titulo na pagina publica | Texto livre | NAO |
| Antecedencia minima | Impede marcar "para daqui a 10 minutos" | Horas, numero inteiro. Vazio = sem antecedencia | NAO |
| Antecedencia maxima | Ate quantos dias no futuro se pode marcar | Dias, inteiro a partir de 1. Vazio = sem teto | NAO |
| Intervalo entre eventos | Respiro obrigatorio ANTES e DEPOIS de cada compromisso ja marcado | Minutos, MULTIPLO DE 5. Padrao 0 | NAO |
| Espacamento dos horarios | De quanto em quanto tempo os horarios aparecem na lista | 5 a 120, multiplo de 5. A tela oferece 15, 30, de hora em hora e "igual a duracao mais o intervalo" | NAO |
| Maximo de agendamentos por dia | Quando a cota zera, o dia FECHA por inteiro | Inteiro a partir de 1. Vazio = sem teto | NAO |
| Datas de inicio e termino | A janela de calendario em que esse servico existe | Duas datas; o fim tem que ser depois do inicio | NAO |
| Pedir e-mail | Se a pagina publica pede e-mail | ligado ou desligado | NAO |
| Pedir observacao | Se a pagina publica pede um motivo | ligado ou desligado | NAO |
| Enviar confirmacao por WhatsApp | Liga a mensagem automatica na hora da marcacao | ligado ou desligado. Nao pode ser desligado se ja existem lembretes | NAO |
| Caixa de fallback | Qual caixa envia a confirmacao e os lembretes | Uma caixa da mesma conta | NAO |
| Modelo de mensagem | O texto simples da confirmacao | Texto com variaveis | sim |
| Lembretes | As mensagens programadas do servico | Ate 5, cada um com antes/depois e texto | sim, mas so com a confirmacao JA ligada no painel |

**O texto da confirmacao e os lembretes VOCE consegue escrever pelo conector; LIGAR a confirmacao,
nao.** O interruptor "Enviar confirmacao por WhatsApp" e a escolha da caixa nao existem no conector.
E como lembrete sem confirmacao ligada e recusado pelo sistema, na pratica: numa conta em que a
confirmacao ainda nao foi ligada no painel, mandar lembrete junto com o servico faz a criacao
INTEIRA falhar. Ordem certa: criar o servico, o cliente liga a confirmacao e escolhe a caixa no
painel, e so entao voce grava texto e lembretes.

**Intervalo e espacamento sao coisas diferentes e o cliente confunde sempre.** O intervalo espaca a agenda de verdade: 15 minutos num compromisso das 10h as 11h bloqueia das 9h45 as 11h15. O espacamento so decide o que aparece na lista: de 15 em 15 mostra 9h, 9h15, 9h30, e nao encurta nem alonga nada.

**Intervalo e espacamento PRECISAM ser multiplos de 5.** Nao e capricho: a gravacao do agendamento so aceita horario em multiplo de 5. Com 7 minutos de intervalo numa reuniao de 45, a lista publica oferece 09:07 e 10:14 - e a gravacao recusa TODOS. O cliente clica no botao que a propria pagina desenhou e ouve que o horario nao esta disponivel.

**Trocar o endereco do link derruba o link antigo.** Inclusive dentro de mensagens ja enviadas, no codigo colado no site e no anuncio pago. Os agendamentos ja feitos, a IA e os links de cancelar e remarcar continuam funcionando, porque trabalham por numero e nao pelo endereco.

**Apagar um servico e recusado** se houver agendamento futuro confirmado - o sistema diz quantos sao. Cancele-os ou pause o servico. Os agendamentos passados de um servico apagado ficam no relatorio sob o rotulo de tipo excluido.

---

## 4. Dias e horarios de atendimento

Cada bloco tem tres coisas: o dia da semana (0 = domingo ate 6 = sabado), a hora de inicio e a hora de fim, as duas no formato HH:MM.

Regras:

- Varios blocos por dia sao permitidos (manha e tarde, por exemplo).
- **Dois blocos do mesmo dia nao podem se sobrepor.** A recusa compara com os blocos JA SALVOS - dois blocos sobrepostos enviados no MESMO cadastro passam e a agenda fica com horarios repetidos. Confira o que voce esta mandando antes de mandar.
- O **inicio** precisa terminar em multiplo de 5. O fim nao precisa.
- O fim e ate onde o compromisso precisa CABER INTEIRO: com bloco das 9h as 12h e duracao de 60 minutos, o ultimo horario oferecido e 11h.
- **Sem nenhum bloco, a pagina publica nunca oferece horario nenhum.** Este e o erro mais comum de conta nova.

**Isto NAO e o horario de trabalho do atendente.** Sao duas coisas independentes: existe uma disponibilidade geral do atendente (usada na Agenda e por outras ferramentas da IA) e existe a disponibilidade DESTE servico, que e a unica que o agendamento usa. Quem preenche so o horario de trabalho geral e reclama que "nao aparece horario nenhum" caiu exatamente aqui.

**Pelo conector**, os blocos so entram no momento da CRIACAO do servico. Editar os dias e horarios de um servico que ja existe e no painel.

---

## 5. Confirmacao no WhatsApp

E a mensagem que sai no instante em que a pessoa marca. Liga-se na secao Confirmacao WhatsApp do servico, no painel.

- Sem ela ligada **nao existe lembrete** - o sistema recusa criar.
- Com ela ligada e uma caixa escolhida, **cada pessoa que marcar e nao tiver conversa naquela caixa ganha uma conversa nova, aberta**. Isso muda a rotina da equipe e acorda automacao, fluxo, card de funil, prazo e notificacao. Avise antes de ligar.
- Sem caixa escolhida, o sistema tenta a conversa mais recente do contato em qualquer caixa (mesmo encerrada). Se nao houver nenhuma, a confirmacao nao sai e nao fica registro.
- A caixa escolhida MANDA SEMPRE: o sistema nao desvia para uma conversa aberta de outra caixa.
- A confirmacao pode ser um texto simples ou uma sequencia de blocos com texto, audio, imagem, arquivo e modelo aprovado do WhatsApp oficial.

---

## 6. Lembretes

Na tela aparecem como **Follow Ups**, dentro do servico.

- **No maximo 5 por servico.**
- Cada um diz se sai ANTES ou DEPOIS do horario marcado, e de quanto: dias, horas e minutos. A soma nao pode ser zero - o minimo e 1 minuto.
- Precisa ter texto ou pelo menos um bloco de conteudo.
- **Todos herdam a caixa da confirmacao.** Nao da para escolher caixa por lembrete.
- Cada lembrete dispara UMA vez por agendamento. Nao existe repeticao.
- Mudou um lembrete e ja existem agendamentos marcados? A tela pergunta o que fazer com as mensagens ja na fila. Pelo conector, a ferramenta de aplicar mudanca de lembrete faz o mesmo em tres modos: reagendar as pendentes, cancelar as pendentes, ou agendar para quem ja estava marcado e ainda nao tinha lembrete.

**Pelo conector** da para criar e editar lembretes (na criacao do servico e depois, pela ferramenta
de atualizar), desde que a confirmacao ja esteja ligada. Duas regras da propria ferramenta: para
EDITAR um lembrete que ja existe e obrigatorio mandar o numero dele - sem isso nasce outro; e o
texto precisa ir nos DOIS lugares (no campo de texto e no bloco de texto), porque a tela le um e o
envio usa o outro, e gravar so um deixa a tela e a mensagem entregue diferentes.

Usos que funcionam bem: lembrete na vespera com a data e o dia da semana; link da sala 10 minutos antes; pedido de retorno no dia seguinte.

---

## 7. Variaveis dos textos: duas listas DIFERENTES

**A confirmacao em TEXTO SIMPLES e o lembrete tem renderizadores separados.** Variavel escrita no texto errado sai LITERAL na mensagem do cliente, com as chaves e tudo.

Funcionam SEMPRE, nos dois:

`{{nome}}` `{{email}}` `{{telefone}}` `{{descricao}}` `{{data}}` `{{horario}}` `{{dia_semana}}` `{{duracao}}` `{{titulo}}` `{{tipo_evento}}` `{{agente}}`

Estas cinco funcionam sempre no lembrete, e na confirmacao SO quando ela foi montada no editor de
blocos do painel:

`{{meet_link}}` `{{link_cancelar}}` `{{link_remarcar}}` `{{horas_ate_evento}}` `{{horas_desde_evento}}`

Existem apelidos antigos em ingles que continuam funcionando no lembrete por compatibilidade. **Nao use em texto novo.**

Consequencia pratica: **a confirmacao que VOCE grava pelo conector e sempre a de texto simples** -
nela o link de cancelar, o link de remarcar e o link da sala saem LITERAIS. Se o cliente quer
oferecer o cancelamento ja na confirmacao, o caminho e ele montar a confirmacao no painel, ou usar
um lembrete de poucos minutos depois.

---

## 8. Como o horario livre e calculado

O sistema parte dos blocos do dia e vai tirando:

1. **O que ja esta ocupado na agenda daquele profissional.** Conta TUDO: compromisso de qualquer outro servico, tarefa criada na mao e evento vindo do Google Agenda. Compromisso sem duracao definida ocupa 30 minutos. Concluido ou cancelado libera o horario.
2. **O intervalo antes e depois** de cada um desses compromissos. E por isso que somem os horarios vizinhos de um compromisso, e nao so o dele.
3. **A antecedencia minima**, contada a partir de agora.
4. **A antecedencia maxima** e a janela de datas de inicio e termino do servico.
5. **O teto por dia**: quando a cota do dia fecha, o dia inteiro para de oferecer horario.

E o compromisso precisa CABER INTEIRO dentro do bloco.

**Os horarios mudam com a duracao.** Pedir a lista com 60 minutos e gravar com 30 (ou o contrario) da erro: a gravacao refaz a conta e recusa o horario que a lista tinha oferecido. Peca sempre a lista com a mesma duracao que vai usar.

---

## 9. Roteiro de "nao aparece horario nenhum"

Confira nesta ordem, com `lionchat_booking_event_types_show` e `lionchat_booking_event_types_slots` numa data real:

1. **A agenda do profissional esta cheia naquele dia?** E a causa numero um do dia a dia. Olhe o calendario dele antes de mexer em qualquer configuracao.
2. **O servico esta ativo?** Pausado nao aparece na pagina publica e responde "nao encontrado" na marcacao pelo painel.
3. **Existe bloco de horario naquele dia da semana?** Sem bloco, nunca ha horario.
4. **A data esta dentro da janela de inicio e termino?** E dentro da antecedencia maxima em dias?
5. **A antecedencia minima ja passou?** Testar "hoje daqui a pouco" com 24 horas de antecedencia sempre volta vazio.
6. **O teto do dia estourou?**
7. **A duracao pedida cabe no bloco?** 60 minutos num bloco de 9h as 9h30 nao cabe.
8. **O intervalo esta grande demais?** Intervalo de 60 minutos numa agenda cheia apaga quase tudo.
9. **Voce esta olhando o fuso certo?** Os horarios saem no fuso do servico, nunca no do navegador de quem pergunta.

---

## 10. Os links publicos e o codigo para o site

Sao tres formatos, e todos vem PRONTOS na resposta das ferramentas. **Nunca monte a URL na mao**: cada instalacao tem o proprio endereco, e escrever o endereco de outra manda o cliente para o painel errado.

| Link | O que mostra | Onde vem |
|---|---|---|
| Do servico | Vai direto na escolha de dia e hora daquele servico | Campo `public_url` do tipo de evento |
| Do profissional | Lista os servicos ATIVOS daquele atendente | Campo `agent_profile_url` |
| Da equipe | Todos os atendentes com servicos ativos, agrupados | Pagina da equipe da conta |

O **codigo para colar a agenda no site** (botao Embed, na aba Booking) e gerado na tela, por escopo: equipe, profissional ou servico. Nao existe pelo conector.

Os links de cancelar e remarcar que o cliente recebe tem protecao contra abuso e cada uso invalida o anterior - por isso o cliente que tenta reusar um link velho recebe erro.

---

## 11. Google Agenda

O que a conexao faz: espelha os compromissos no Google Agenda do profissional, manda convite por e-mail para o cliente (quem manda e o Google, nao o LionChat) e gera o link do Meet nos servicos do tipo Call / Meet.

- **So administrador conecta**, pelo navegador, e e uma conexao por conta LionChat.
- Com varios profissionais, cada um precisa de um sub-calendario do Google atribuido a ele (calendario compartilhado). **Sem isso o compromisso daquele profissional nao vai para o Google** e o convite e o link de sala nao saem.
- Da para trazer calendarios so-leitura de fora: os compromissos deles aparecem na Agenda e OCUPAM horario no calculo.
- Se a autorizacao for revogada, a sincronizacao desliga e aparece um aviso de reconexao na tela.
- Pelo conector da para ver o estado, escolher qual calendario recebe os compromissos, ligar e desligar a sincronizacao, mandar sincronizar agora, ajustar os calendarios so-leitura e os compartilhados. **Conectar, nao.**

**Servico de videochamada sem Google conectado vira reuniao sem sala.** E o e-mail do cliente e obrigatorio nesse tipo mesmo com "pedir e-mail" desligado - sem ele a reserva e recusada e o cliente ve um erro generico.

---

## 12. A IA marcando pela conversa

O agente de IA consulta os horarios, responde "as 13h50 cabe?" e marca, usando exatamente o mesmo motor da pagina publica. Precisa de servicos ATIVOS.

- Para limitar quais servicos ela pode oferecer: `lionchat_captain_assistants_update` com `config.booking_event_type_ids`. **Lista vazia significa todos os servicos ativos do atendente da conversa** - em clinica com varios profissionais isso costuma ser errado. Restrinja.
- Quando o assistente agenda por essa agenda, ele passa a RECUSAR responder horario pela ferramenta de agenda geral do atendente, e diz que a agenda de agendamento e a autoridade. E de proposito: as duas listas de horario nao sao a mesma coisa.
- Peca o e-mail do contato antes de marcar servico de videochamada.

---

## 13. Fluxos que comecam num agendamento

O bloco Inicio de um fluxo aceita quatro gatilhos de agenda: agendamento criado, cancelado, remarcado e concluido.

- **A fonte dos eventos e o compromisso na Agenda**, nao o registro da reserva. Por isso concluir, cancelar, adiar ou mudar a hora pela tela da Agenda dispara o fluxo.
- **Filtro vazio significa TODOS.** Sem escolher servico e profissional, o fluxo dispara para a agenda inteira da conta.
- Dois fluxos ativos com o mesmo gatilho e filtros que se cruzam disparam os dois no mesmo agendamento - o sistema aponta isso como conflito.
- A chave **criar conversa** nasce desligada. Ligada, cada agendamento passa a abrir (ou reabrir) uma conversa na primeira caixa do fluxo, e isso acorda automacao, outros fluxos, o aviso para sistemas de fora, card de funil, prazo, notificacao e a tela ao vivo. Nunca ligue sem avisar.
- Dentro do fluxo existem variaveis do agendamento (numero, servico, situacao, tipo, duracao, profissional, observacao, data, hora, dia da semana, links de cancelar e remarcar e link da sala). No remarcado vem tambem a data e a hora ANTERIORES. O link da sala nao existe no gatilho de criado.
- Agendamento marcado na ficha de um vinculado SEM telefone (a consulta do filho) nao chega a abrir
  conversa em caixa de WhatsApp, e o gatilho fica mudo. Se aquela ficha ja tiver conversa nas caixas
  do fluxo, o gatilho dispara normalmente.

---

## 14. Os oito campos gravados na ficha do contato

Quando o PRIMEIRO servico da conta e criado, oito campos protegidos nascem sozinhos na ficha do contato e passam a ser preenchidos a cada agendamento: titulo, data, hora, data e hora completa, duracao, tipo, profissional e observacao. Eles viram variavel em mensagem, campanha, automacao e fluxo. A data e a hora tem tipo proprio (data e hora, e nao texto solto).

Eles NUNCA sobrescrevem um campo que o cliente ja tinha criado com a mesma chave.

**Duas armadilhas que chegam ao cliente final:**

- **Guardam so o ULTIMO agendamento daquela pessoa.** Quem tem duas consultas marcadas recebe a data da segunda numa mensagem sobre a primeira.
- **Nao sao limpos no cancelamento.** Quem cancelou continua com a data preenchida e pode receber "sua consulta e dia X".

Use esses campos em mensagem imediata. Para cobranca ou lembrete de uma consulta especifica, prefira os lembretes do proprio servico, que trabalham em cima daquele agendamento.

---

## 15. A situacao do compromisso: reserva contra Agenda

Existem dois registros com o campo situacao, e eles NAO andam juntos:

- O **registro da reserva** so muda quando o cancelamento parte dos links do cliente.
- O **compromisso na Agenda** e o que a equipe move: concluir, cancelar, adiar, mudar a hora.

Quem conclui ou cancela pela tela da Agenda - o caminho normal da equipe - deixa a reserva parada em "confirmado". Foi medido em producao: 61 reservas todas confirmadas contra 25 compromissos concluidos e 7 cancelados.

Consequencias praticas:

- **Para contar concluidos e cancelados, use o relatorio de Agendamentos**, que ja le a situacao certa. Ler a reserva direto da zero concluidos e zero cancelados.
- **Cancelar pela tela da Agenda NAO desarma os lembretes.** O paciente e lembrado de uma consulta cancelada.
- **Remarcar ou adiar pela tela da Agenda muda so o compromisso.** O lembrete continua no relogio antigo e o texto anuncia a hora ERRADA.

Diga isso ao cliente quando ele montar lembretes: para cancelar de verdade, o caminho seguro e o link de cancelar que o proprio cliente final recebe.

---

## 16. Ferramentas do conector

| Ferramenta | Para que serve |
|---|---|
| `lionchat_booking_event_types_list` | Lista os servicos, com os links publicos, os dias e horarios e os lembretes. E daqui que saem os numeros. |
| `lionchat_booking_event_types_show` | Um servico com todo o detalhe. E a unica forma de LER pelo conector as regras que o create nao deixa escrever. |
| `lionchat_booking_event_types_create` | Cria o servico. Os dias e horarios so entram AQUI. Aceita tambem o texto da confirmacao e os lembretes. |
| `lionchat_booking_event_types_update` | Edita o servico, o texto da confirmacao e os lembretes. Nao edita dias e horarios, nem liga a confirmacao. |
| `lionchat_booking_event_types_destroy` | Apaga o servico. Recusa se houver agendamento futuro confirmado. **Voce nao usa: skill nao apaga.** |
| `lionchat_booking_event_types_slots` | Os horarios livres de uma data, para a equipe marcar. Aceita duracao sob medida. Servico pausado responde "nao encontrado". |
| `lionchat_booking_event_types_list_1` | Conta os lembretes ainda pendentes de um servico. |
| `lionchat_booking_event_types_list_2` | Lista as mensagens agendadas de UM agendamento. |
| `lionchat_booking_event_types_create_1` | Aplica mudanca de lembrete no que ja esta marcado: reagendar, cancelar as pendentes ou agendar para quem ja estava marcado. |
| `lionchat_public_booking_show_1` | O que o cliente final ve de um servico. |
| `lionchat_public_booking_list_1` | Os horarios livres do lado do CLIENTE. Nao aceita duracao sob medida, de proposito. |
| `lionchat_public_booking_create` | **MARCA DE VERDADE.** Unico caminho do conector que cria agendamento. So com autorizacao para aquela marcacao. |
| `lionchat_public_booking_list` / `_show` | A pagina publica da equipe e o perfil de um atendente. |
| `lionchat_booking_reports` | O relatorio de Agendamentos. Detalhe em `references/relatorios.md`. |
| `lionchat_google_calendar_list` | O estado da conexao com o Google. |
| `lionchat_google_calendar_create` | Escolhe qual calendario recebe os compromissos. |
| `lionchat_google_calendar_list_2` / `_list_4` / `_update` / `_create_3` | Calendarios disponiveis, compartilhados por atendente, gravar os compartilhados e os so-leitura. |
| `lionchat_google_calendar_create_1` / `_create_2` / `_reactivate` / `_destroy` | Ligar e desligar a sincronizacao, sincronizar agora, religar e desconectar. |
| `lionchat_google_calendar_list_3` | Quais atendentes a pessoa conectada consegue enxergar na agenda. |
| `lionchat_google_calendar_create_4` | INVERTE o interruptor "liberar minha agenda" **de quem esta conectado**, e so dele. E um inversor, nao um liga: chamar duas vezes volta ao estado anterior, e chamar sem saber o estado atual pode ESCONDER uma agenda que estava visivel. Nao existe caminho para liberar a agenda de outra pessoa - ela mesma faz no proprio perfil. |
| `lionchat_tasks_list` / `_show` / `_update` / `_toggle` / `_snooze` | Ler e mudar a situacao dos compromissos na Agenda: concluir, reabrir, adiar. E aqui que a situacao real mora. |
| `lionchat_captain_assistants_update` | Limita os servicos que a IA pode oferecer. |
| `lionchat_agents_list` | Os numeros dos atendentes. |
| `lionchat_custom_attributes_list` | Confere que os oito campos de agendamento existem na conta. |

**NUNCA use a ferramenta de criar tarefa da Agenda para marcar um agendamento.** Nasce um compromisso solto: sem confirmacao, sem lembrete, sem link de cancelar e sem entrar no relatorio de Agendamentos.

---

## 17. Armadilhas

Cada item: o que voce faz, o que acontece, o sinal, como sair.

### Criar o servico e dizer que esta pronto

**Voce faz:** cria o servico pelo conector e anuncia agenda pronta.
**Acontece:** o cliente fica sem antecedencia minima, sem teto por dia, sem intervalo e com a confirmacao DESLIGADA (o conector nao liga, nem escolhe a caixa) - e nem os dias e horarios ele consegue editar pelo conector depois.
**Sinal:** o cliente volta dizendo que alguem marcou para daqui a 5 minutos, ou que ninguem recebeu aviso.
**Como sair:** ao terminar, liste item por item o que falta e o caminho na tela.

### Intervalo ou espacamento fora do multiplo de 5

**Voce faz:** grava 7 minutos de intervalo.
**Acontece:** a pagina oferece horarios que a gravacao recusa.
**Sinal:** "cliquei no horario e disse que nao esta disponivel".
**Como sair:** so multiplos de 5, nos dois campos.

### Dois blocos de horario sobrepostos no mesmo cadastro

**Voce faz:** manda 09:00-12:00 e 11:00-18:00 na mesma criacao.
**Acontece:** passa. A recusa de sobreposicao so compara com blocos ja salvos.
**Sinal:** horarios repetidos ou estranhos na lista publica.
**Como sair:** confira a sua propria lista antes de mandar.

### Variavel do lembrete escrita na confirmacao

**Voce faz:** poe o link de cancelar no texto da confirmacao.
**Acontece:** sai literal na mensagem do cliente.
**Sinal:** o cliente recebe as chaves na mensagem.
**Como sair:** use as duas listas da secao 7.

### Lembrete antes da confirmacao

**Voce faz:** tenta criar lembrete com a confirmacao desligada.
**Acontece:** o sistema recusa dizendo para ativar a confirmacao antes. E depois, com lembretes criados, desligar a confirmacao tambem e recusado.
**Como sair:** ligue a confirmacao e escolha a caixa primeiro.

### Videochamada sem Google

**Voce faz:** cria servico do tipo Call / Meet sem a conexao.
**Acontece:** o compromisso nasce sem sala.
**Como sair:** conecte o Google antes, e garanta um calendario para cada profissional.

### Filtro vazio no gatilho de fluxo

**Voce faz:** deixa servico e profissional em branco achando que vazio e nenhum.
**Acontece:** vazio e TODOS - o fluxo dispara para a agenda inteira.
**Como sair:** preencha os dois filtros sempre.

### Contar concluidos pelo registro da reserva

**Voce faz:** le a situacao da reserva para responder quantos compareceram.
**Acontece:** da zero concluidos e zero cancelados.
**Como sair:** use o relatorio de Agendamentos.

### Pedir os horarios com uma duracao e gravar com outra

**Voce faz:** lista com 60 minutos, grava com 30.
**Acontece:** a gravacao recusa o horario que a lista ofereceu.
**Como sair:** mesma duracao nos dois passos.

### Trocar o endereco do link de um servico divulgado

**Voce faz:** "so arrumando o nome do link".
**Acontece:** todo link ja enviado para de funcionar.
**Como sair:** nao troque sem avisar; se trocar, o cliente precisa reenviar o link novo.

---

## 18. So da para fazer na mao, no painel

- Editar os dias e horarios de um servico que ja existe.
- Antecedencia minima, antecedencia maxima, intervalo entre eventos, espacamento dos horarios, maximo por dia e a janela de datas.
- LIGAR a confirmacao no WhatsApp, escolher a caixa, escolher se sai por caixa QR Code ou oficial, e montar o conteudo dela em blocos (audio, imagem, arquivo, modelo aprovado). O texto SIMPLES da confirmacao o conector escreve.
- Apagar um lembrete: o conector ate aceita, mas esta skill NAO apaga nada - mande o cliente apagar na tela. Criar e editar lembrete o conector faz.
- Pendurar audio, imagem, arquivo ou modelo aprovado num lembrete: o conector documenta so o bloco de TEXTO. Midia precisa do arquivo ja carregado - faca na tela.
- A descricao do servico, pedir e-mail e pedir observacao.
- Conectar o Google Agenda (e a janela de autorizacao no navegador).
- Pegar o codigo para colar a agenda no site.
- Liberar a agenda de OUTRO atendente para os colegas: cada pessoa faz no proprio perfil. Pelo conector so da para inverter o interruptor de quem esta conectado.
- Escolher caixa diferente por lembrete: nao existe, todos herdam a da confirmacao.
- Repetir um lembrete: nao existe, cada um dispara uma vez por agendamento.
