# Diagnostico: "a automacao nao esta funcionando"

Indice:

1. Roteiro de diagnostico em 5 passos
2. Como ler o historico de execucoes
3. Sintoma que o cliente descreve, causa e conserto
4. Quem pode mexer em automacao e em macro
5. Limites do sistema que precisam ser ditos ao cliente
6. O que a base de conhecimento das ferramentas diz errado

---

## 1. Roteiro de diagnostico em 5 passos

1. **A regra esta ligada?** `lionchat_automation_rules_list`. Regra desligada nao e nem consultada.
   Se ela desligou sozinha, va para a secao 3 (auto-desligamento).
2. **Leia a regra inteira** com `lionchat_automation_rules_show` e confira, nesta ordem: o gatilho e o
   certo? A ULTIMA condicao esta com o conector vazio? As acoes usam as chaves certas (compare com
   `acoes.md`)? O gatilho e de card e a acao e de conversa?
3. **Peca o print da conversa ao cliente.** As pilulas de atividade (avisos cinzas no meio da
   conversa) mostram "Automacao: [nome] disparou" e, quando uma acao falha, uma linha
   "Automacao: [nome] — falha ao executar [acao]: [erro]". E o unico sinal que o cliente ve sem abrir
   nada.
4. **Abra o historico** com `lionchat_automation_rules_list_1` passando o numero da regra. Interprete
   pela secao 2. Lembre: **so as ultimas 48 horas** ficam guardadas.
5. **Provoque o gatilho de verdade** (mandar mensagem, mover card, por etiqueta) e repita o passo 4.
   Nao existe modo de ensaio nem botao de "rodar agora".

---

## 2. Como ler o historico de execucoes

| O que voce ve | O que significa | Onde olhar |
|---|---|---|
| Nenhuma execucao | O gatilho nunca chegou a ser avaliado | Gatilho errado, regra desligada, ou o evento nao aconteceu como o cliente acha |
| Execucao com "skipped" | As condicoes nao bateram | As condicoes: valor, operador, conector |
| Passo com "failed" | O parametro daquela acao esta errado | O formato de `action_params` daquela acao |
| Passo "delivery_failed" | A acao criou a mensagem, mas o canal recusou depois | O canal (WhatsApp fora do ar, numero invalido, janela de 24 horas fechada) |
| Tudo "success" e o cliente diz que nada aconteceu | Acao pulada em silencio | Card inexistente, agente fora da caixa, funcao desligada na conta, gatilho de card com acao de conversa |

Outros pontos:

- **"Sucesso" numa acao de envio significa apenas que a MENSAGEM FOI CRIADA**, nunca que o cliente
  recebeu. A recusa do canal chega depois, como um passo extra, e vira "failed" — mas so aparece se
  alguem voltar a olhar.
- No gatilho **Webhook** o campo "condicoes bateram" nao e mostrado: ali as condicoes nem sao
  avaliadas.
- Cada execucao guarda no maximo 100 passos.
- **Nao existe botao de reprocessar** uma execucao que falhou. Nem para automacao, nem em ferramenta
  nenhuma. Diferente dos eventos de integracao, que tem reenvio proprio.
- **Nao existe listagem das entregas do webhook disparado por uma AUTOMACAO.** As tentativas sao
  gravadas, mas nao ha como ve-las. Para MACRO existe: `lionchat_macros_deliveries_list` (so
  administrador, guarda 7 dias). Se o cliente precisa auditar o webhook, sugira montar por macro ou
  pela tela de Webhooks.

---

## 3. Sintoma que o cliente descreve, causa e conserto

### "Salvei e nao acontece nada"

- **Nome de gatilho que nao existe.** Ele nao e conferido ao salvar — a regra nasce muda. Confira
  contra a lista de 8 em `gatilhos-e-condicoes.md`. Sintoma: historico completamente vazio.
- **Conector sobrando na ultima condicao.** Deixe vazio na ultima. Ver `gatilhos-e-condicoes.md`.
- **Condicao que o motor de conversa nao reconhece** (funil ou etapa num gatilho de conversa,
  politica de SLA como condicao, cidade). Sem nenhum filtro reconhecido a regra nunca dispara.
- **Gatilho de card com acao de conversa.** As acoes fora das 10 sao descartadas.
- **Modelo do WhatsApp com as chaves do Flow** (`template_name` e companhia). Salva e nao envia nada.
- **"Conversa Criada" com filtro de conteudo**, num caso em que a conversa nasce sem mensagem do
  cliente (aberta pelo painel, por campanha, por integracao).
- **Prioridade "Nenhuma" como condicao.** Nunca e verdadeira.
- **Regra desligada** — inclusive desligada pelo proprio sistema (ver abaixo).

### "A automacao desligou sozinha"

Quando a regra usa uma chave de condicao que o motor nao sabe traduzir, ela falha na validacao. Na
**segunda** falha o sistema **desliga a regra** (`active` vira falso) e manda e-mail aos
administradores. Causas: atributo personalizado apagado depois de a regra ter sido criada; funil ou
etapa como condicao num gatilho de conversa; politica de SLA como condicao.

Conserto: corrija a condicao (editar as condicoes zera o contador) e religue a regra **no painel** —
ligar/desligar nao da pelas ferramentas. Se quiser descobrir quem mexeu, `lionchat_audit_logs_show`.

### "Dispara demais" / "mexeu em conversa que nao era"

- **Gatilho "Acao na conversa" criado por ferramenta**: nasce sem recorte e dispara em qualquer
  mudanca, inclusive em campos invisiveis como o carimbo de primeira resposta. Ajuste na tela.
- **Gatilho de card com condicao de conversa**: o motor de card **aprova** o que nao entende, entao a
  regra vale para todos os cards de todos os funis.
- **Gatilho "Mensagem Criada" sem o filtro de Tipo da Mensagem**: dispara tambem no que voce envia.
- **Gatilho Webhook**: as condicoes nao filtram nada. O recorte e no mapeamento de eventos da tela da
  integracao.
- **Etiqueta "diferente de"**: desde 24/08/2026 significa "nao tem essa etiqueta" e passou a alcancar
  tambem conversas sem etiqueta nenhuma.

### "Mandou para a conversa errada" / "reabriu uma conversa antiga"

Se foi uma **macro**: "Abrir conversa", "Pendenciar conversa" e a mudanca de status para
aberto/pendente vao para a conversa **viva** do contato naquela caixa, que pode nao ser a que estava
na tela. Resolver, adiar e silenciar ficam na de origem.

Se foi uma **automacao**: essas acoes so mudam de conversa quando a regra tem Caixa de Envio
preenchida (proximo item). Sem Caixa de Envio, elas agem na propria conversa que disparou a regra.

### "A mensagem foi para outra caixa"

A Caixa de Envio esta preenchida. Ela redireciona as acoes de mensagem para outra caixa, criando
conversa nova la. Para filtrar por caixa, a peca certa e a condicao "Caixa de Entrada".

### "Atribuiu o agente e a conversa ficou sem responsavel"

O agente nao e membro daquela caixa (ou nao e administrador, ou nao confirmou o convite, ou foi
removido da conta). A acao e pulada em silencio. Confira com `lionchat_inbox_members_show`.

### "O card nao foi criado / nao moveu"

- Sem `funnel_id` ou sem `funnel_stage`, a acao e pulada.
- Mover, anotar, atribuir, cronometro e Ganho/Perdido sao pulados quando nao existe card. Ponha
  "Criar Card Kanban" antes, na mesma regra.
- Card ja marcado como Ganho ou Perdido nao conta como existente: um novo card nasce.

### "A conversa sumiu do painel"

A acao "Adiar" da automacao e da macro adia **sem data**. So volta se alguem reabrir na mao ou o
cliente escrever.

### "O AI Agente parou de responder depois que usei a macro"

Macro que manda mensagem sai em nome de quem clicou, e o sistema entende isso como "um humano
assumiu": quando o AI Agente daquela conversa esta configurado para parar ao receber resposta
humana, ele fica calado por alguns minutos ou e desligado de vez (depende da configuracao dele). E o
efeito colateral da macro de encerramento. Mensagem de AUTOMACAO nao causa isso.

### "O texto saiu com um pedaco em branco"

Variavel que a automacao nao preenche sai vazia, em silencio. A automacao so conhece contato,
conversa, caixa, responsavel da conversa, conta e — so no lembrete da e-Clinica — agendamento. Ver
`acoes.md`, secao 6.

### "O ultimo Flow da cadeia nao disparou"

Uma acao de automacao pode gerar um evento que dispara um Flow, que dispara outra automacao. Ha um
teto de **5 encadeamentos entre motores**; depois disso a cadeia e cortada em silencio.

### "Mudei a regra pela ferramenta e nao mudou nada"

`lionchat_automation_rules_update` so aceita nome, descricao, condicoes e acoes. Ligar/desligar,
trocar a Caixa de Envio e trocar o gatilho **nao passam** — o campo e descartado sem erro. Isso e no
painel.

### "Duas regras brigando"

Todas as regras ativas daquele gatilho sao avaliadas, uma depois da outra, e **nao ha como escolher a
ordem** — nao existe campo de prioridade nem botao de reordenar. Uma regra que muda o status pode
invalidar a condicao da seguinte. O conserto e juntar as duas numa regra so, ou tornar as condicoes
mutuamente exclusivas — nunca contar com a ordem.

---

## 4. Quem pode mexer em automacao e em macro

**Automacao:** so administrador da conta, ou quem tiver um papel personalizado com a permissao de
gerenciar automacoes (`automation_manage`). Se o cliente diz "nao aparece Automacao no meu menu", e
isso. O caminho e criar ou ajustar o papel com `lionchat_custom_roles_list` /
`lionchat_custom_roles_create` / `lionchat_custom_roles_update` — sem precisar promover ninguem a
administrador. Repare que Flows, ao contrario, sao abertos a todo agente.

**Macro:** qualquer agente cria. Mas:

- agente comum so consegue gravar macro **pessoal**, mesmo pedindo global;
- macro pessoal de outra pessoa nao aparece nem executa para os demais;
- editar macro global exige ser o autor ou administrador.

E a resposta para "nao consigo editar a macro do meu colega".

---

## 5. Limites do sistema que precisam ser ditos ao cliente

- Historico de execucoes: **48 horas**, apagado por faxina diaria. Nao ha como aumentar. A consulta
  traz no maximo as **50** execucoes mais recentes, e cada execucao guarda no maximo 100 passos.
- Espera ("Aguardar"): teto de **5 minutos**, excedente cortado em silencio.
- Encadeamento entre motores: teto de **5**.
- Ordem das regras: nao da para escolher nem reordenar.
- Nao existe modo de ensaio, "rodar agora" nem reprocessar execucao que falhou.
- Nao existe condicao de horario de expediente.
- Macro nao tem chave liga/desliga.
- "Enviar Anexo": arquivo que ja esta publicado na internet entra por `lionchat_upload_create`
  (informando o endereco dele) e o identificador devolvido e o que a acao consome. Arquivo do
  computador do cliente so pela tela.

---

## 6. O que a base de conhecimento das ferramentas diz errado

Ao ler a descricao das ferramentas do LionChat, desconfie destes tres pontos — eles ja foram
conferidos no sistema e estao errados na descricao:

1. A ferramenta de **atualizar regra** diz aceitar gatilho, ligada/desligada e Caixa de Envio. **Nao
   aceita** — esses campos nao estao no formulario dela e sao descartados sem erro.
2. A descricao do campo **Caixa de Envio** na criacao diz "Restringir a uma caixa de entrada". Ela
   **nao restringe**: redireciona as acoes de mensagem.
3. Um dos documentos de apoio (o de fluxos de conversa) lista, entre os eventos "pra automation",
   `conversation_pending` e `first_reply_created`. **Esses dois nao valem como gatilho de
   automacao.** Sao 8 gatilhos, os da secao 1 de `gatilhos-e-condicoes.md`. E o nome do gatilho nao
   e conferido ao salvar: a regra e criada, aparece na lista e nunca dispara.
