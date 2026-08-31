# Armadilhas: o sintoma que o cliente relata e a causa real

Indice:
1. "O lead nao chega pra ninguem"
2. "O atendente nao ve a conversa"
3. "A mensagem automatica nao sai"
4. "Eu salvei e nao mudou nada"
5. "Nao consigo responder o cliente"
6. "A pesquisa de satisfacao nao chega"
7. "O SLA nao esta medindo"
8. "Sumiu gente da caixa / do time"
9. "Desliguei a distribuicao e continua distribuindo"
10. "A aba nao aparece"
11. Armadilhas de conversa duplicada e de grupo

Todas falham EM SILENCIO. Nao ha erro na tela, o registro parece certo e o comportamento e outro.

---

## 1. "O lead nao chega pra ninguem"

| Causa | Como confirmar | Conserto |
|---|---|---|
| A equipe toda foi marcada como SUPERVISOR da caixa | `lionchat_inbox_members_show` mostra todo mundo como supervisor | Passar as pessoas para `user_ids` e deixar so quem realmente supervisiona em `supervisor_ids` |
| Ninguem tem presenca real (painel fechado) e "distribuir para offline" esta desligado | O cliente confirma que a equipe responde pelo celular | Ligar "distribuir tambem para agentes offline" no nivel que manda (politica se houver, senao caixa) |
| A pessoa esta no TIME mas nao e membro da CAIXA | Comparar `lionchat_teams_list_2` com `lionchat_inbox_members_show` | Acrescentar a pessoa a caixa tambem |
| A distribuicao automatica foi desligada na caixa | `lionchat_inboxes_show` | Religar, se for o desejado |
| Politica de atribuicao vinculada e desativada | `lionchat_inboxes_assignment_policies_list` e depois `lionchat_assignment_policies_show` | Reativar a politica |
| Todo mundo bateu o teto da politica de capacidade | `lionchat_capacity_policies_show` e `_list_1` | Rever o teto por caixa |
| Passou menos de um minuto | - | Nao e defeito: a rede de seguranca da fila roda a cada minuto |

---

## 2. "O atendente nao ve a conversa"

- **Nao e membro da caixa.** Ser membro do time nao basta. Administrador ve tudo mesmo sem estar na
  lista, o que confunde quem testa com a propria conta de admin.
- **O convite ainda nao foi aceito.** Times e caixas informados no convite ficam pendentes e nao
  aparecem em `lionchat_inbox_members_show`. So passam a valer quando a pessoa confirmar o e-mail.
  Nao refaca a operacao.
- **Cargo personalizado sem nenhuma permissao de conversa.** Sem nenhuma das quatro, a pessoa so
  enxerga conversa em que foi colocada como participante ou que esta num card do Kanban dela. E as
  quatro SOMAM: marcar duas entrega os dois conjuntos, nao a intersecao.

---

## 3. "A mensagem automatica nao sai"

- **Horario cadastrado na caixa errada.** So o Chat do Site tem horario proprio. WhatsApp,
  Instagram, Facebook, Telegram e e-mail leem o horario da CONTA. O cliente configura na caixa, testa
  de madrugada, nao recebe nada e conclui que o sistema esta quebrado.
- **Horario da conta nao esta cadastrado ou esta desligado.** Sem ele, nenhuma caixa nao-widget
  consegue mandar aviso de ausencia.
- **Texto do aviso em branco.** Campo vazio nao envia nada; nao existe texto padrao.
- **Saudacao com o interruptor ligado e texto vazio, ou o contrario.** Precisa dos dois.
- **O cliente esperava a saudacao E o aviso na mesma madrugada.** Fora do horario sai SO o aviso; a
  saudacao fica guardada para quando ele voltar dentro do horario.
- **Ja saiu um aviso hoje naquela conversa.** No maximo um por dia por conversa.
- **Um atendente respondeu nos ultimos 5 minutos.** O aviso e segurado para nao interromper conversa
  viva no fim do expediente.
- **O campo do aviso foi mandado pela ferramenta de atualizar a caixa.** Ele nao esta declarado e foi
  descartado em silencio: o valor nunca chegou ao banco.

---

## 4. "Eu salvei e nao mudou nada"

Esta e a familia mais perigosa.

- **Campo nao declarado na ferramenta.** A ferramenta de atualizar a caixa so aceita nome, saudacao
  (ligar e texto), ligar a distribuicao, o bloco de distribuicao e atributos extras. Qualquer outra
  chave e apagada antes de sair, a resposta volta com sucesso e o banco continua igual. Ver a lista
  em `configurar-caixa.md`.
- **Bloco de distribuicao substituido inteiro.** Mandar so uma chave apaga as outras. Leia, junte,
  mande completo.
- **Chave desconhecida dentro do bloco de distribuicao.** So quatro sao aceitas pelo servidor; as
  demais somem.
- **Conferencia feita pela resposta da chamada.** Salvar e conferir na mesma resposta esconde defeito
  de leitura. A conferencia honesta e: salvar, RELER e comparar.
- **Fuso invalido na conta.** E descartado em silencio. Releia a conta depois de gravar.

---

## 5. "Nao consigo responder o cliente"

- **Janela fechada.** WhatsApp Oficial 24 horas, Instagram e Facebook 24 horas, TikTok 48 horas.
  Fora dela so sai modelo aprovado. Mandar modelo NAO reabre a janela: so a mensagem do cliente
  reabre.
- **Conversa de historico importado.** Ela nunca aceita resposta livre, mesmo que a fala seja
  recente: aquelas mensagens nunca passaram pela conexao oficial.
- **Conversas migradas de QR Code para Oficial.** Todas tem mais de 24 horas, entao nenhuma aceita
  texto livre na caixa nova.
- **A caixa foi excluida.** A conversa fica em modo somente leitura com o aviso "A caixa de entrada
  foi removida". Use a ferramenta que religa a conversa a outra caixa.

---

## 6. "A pesquisa de satisfacao nao chega"

- **Janela fechada na hora do envio.** Em WhatsApp Oficial a pesquisa so sai se a conversa ainda
  puder receber mensagem naquele momento. Conversa finalizada depois que o cliente sumiu nao recebe,
  e quanto maior a espera configurada, maior o risco.
- **Filtro por etiqueta.** Se ha etiquetas na regra, so quem se encaixa recebe. Lista vazia libera
  todo mundo.
- **Pesquisa desligada** na caixa - lembre que ela nasce desligada.
- **Texto em branco NAO impede o envio**: sai um texto padrao em portugues. Para nao perguntar,
  desligue a pesquisa.

---

## 7. "O SLA nao esta medindo"

- **A politica nao foi aplicada.** Politica sozinha nao mede nada: falta a regra de automacao, macro
  ou bloco de fluxo com a acao "Adicionar SLA".
- **Prazo informado em minutos.** Os campos sao em SEGUNDOS. "30" e trinta segundos.
- **"Contar so no horario de atendimento" ligado sem horario cadastrado na conta.** O prazo passa a
  correr 24 horas por dia, que e MAIS APERTADO. O cliente ve estouros que nao esperava.
- **Cronometro da primeira resposta preso a atribuicao.** Se esse ajuste estiver ligado no painel e
  ninguem assumir a conversa, o relogio nao conta.
- **Conversas antigas.** O SLA nao e aplicado retroativamente.
- **Tentativa de trocar a politica de uma conversa.** E recusado: uma conversa aceita UMA politica e
  ela nao sai mais.

---

## 8. "Sumiu gente da caixa / do time"

- **Lista parcial.** Membros de caixa, membros de time e os vinculos do agente sao listas COMPLETAS:
  quem nao estiver na lista enviada e REMOVIDO. Mandar "so o novo" tira todo mundo que ja estava.
- **Cargo personalizado apagado.** Enviar o campo de cargo vazio numa alteracao parcial apaga o cargo
  da pessoa. Omita o campo quando nao for para mexer nele.
- **Uso da ferramenta errada de membros da caixa.** A de criar so acrescenta e ignora supervisores;
  quem substitui de verdade e a de atualizar.

---

## 9. "Desliguei a distribuicao e continua distribuindo"

Quando a conversa esta atribuida a um TIME, quem autoriza a entrega e a configuracao do proprio time,
nao a da caixa nem a da politica. E proposital. Se o cliente quer parar tudo, desligue tambem a
distribuicao automatica do time.

Outro caso da mesma familia: com politica vinculada, o ajuste que sobrou na caixa vira configuracao
fantasma - some da tela e o comportamento passa a ser o da politica. Nunca configure os dois.

---

## 10. "A aba nao aparece"

- **Horario de funcionamento**: so em Chat do Site.
- **Grupos, Conexao, Configuracao e Importar Historico do WhatsApp**: so em WhatsApp por QR Code.
  Migracao: WhatsApp por QR Code e Canal da API.
- **Saude da conta**: so em WhatsApp Oficial.
- **Falhas de envio**: so para administrador, e so em WhatsApp (os dois tipos) e Facebook Messenger.
  Nao aparece no Instagram.
- **Ligacao (LionCalls)**: caixa por QR Code ou Oficial, com o recurso de voz liberado na conta E
  vaga de caixa de voz disponivel. Se a conta atingiu o limite de caixas de voz, a aba simplesmente
  nao aparece na caixa nova - e limite, nao defeito.
- **Politica de capacidade**: depende de dois recursos de plano; sem eles a tela nao mostra a
  secao. **Modo Equilibrado**: depende de um recurso de plano e a opcao aparece bloqueada na tela.
- **SLA e cargos personalizados**: dependem de recurso de plano, e ai a recusa e do servidor: a
  tentativa de criar volta com 403.

---

## 11. Armadilhas de conversa duplicada e de grupo

- **Conversa unica desligada em caixa ANTIGA.** Caixa nova ja nasce com a conversa unica ligada, mas
  as criadas antes de marco de 2026 ficaram como estavam: cada mensagem nova abre um card novo e a
  operacao enche de conversas repetidas. Confirme o valor real antes de afirmar como esta.
- **"Aceitar Grupos" desligado.** Mensagem de grupo e descartada em silencio. Sem erro, sem conversa.
- **Ligar aceitar grupos, canais ou lista de transmissao REINICIA a sessao** do WhatsApp por alguns
  segundos. Avise antes. O interruptor de recusar ligacoes nao reinicia.
- **Grupo em caixa Oficial.** Nao existe. Ao migrar conversas de QR Code para Oficial, os grupos
  ficam para tras; se a caixa antiga for apagada, essas conversas se perdem.
- **Campanha em canal que nao aceita.** So Website, SMS, Twilio SMS, WhatsApp (os dois tipos) e
  Canal da API fazem campanha. Criar a caixa de Instagram planejando disparo em massa e trabalho
  perdido.
- **Adicionar muita gente na caixa em laco.** Ja deixou o painel fora do ar. Mande a lista completa
  numa unica chamada.
