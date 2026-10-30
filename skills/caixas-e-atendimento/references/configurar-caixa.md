# Configurar a caixa: campos, valores e o que a ferramenta alcanca

Indice:
1. O que a ferramenta de atualizar a caixa alcanca (e o que ela descarta)
2. Nome da caixa
3. Saudacao automatica
4. Aviso de ausencia
5. Horario de atendimento: da CONTA, nao da caixa
6. Saudacao e aviso na mesma noite: a regra de exclusao
7. Conversa unica
8. Pesquisa de satisfacao
9. Ajustes especificos do Chat do Site
10. Ajustes especificos do WhatsApp por QR Code
11. Ajustes especificos do WhatsApp Oficial
12. Falhas de envio

---

## 1. O que a ferramenta de atualizar a caixa alcanca

`lionchat_inboxes_update` declara apenas estes campos:

| Campo | O que e na tela |
|---|---|
| `name` | Nome da caixa |
| `greeting_enabled` | Ativar saudacao do canal |
| `greeting_message` | Texto da saudacao |
| `enable_auto_assignment` | Habilitar atribuicao automatica |
| `auto_assignment_config` | O bloco da Distribuicao Automatica (ver `distribuicao.md`) |
| `out_of_office_enabled` | Interruptor do aviso de ausencia |
| `out_of_office_message` | Texto do aviso de ausencia |
| `channel.additional_attributes` | Interruptores do CANAL (vai DENTRO de `channel`) - ver abaixo |

**Campo que nao esta nessa lista nao faz parte do contrato da ferramenta.** Nada garante que ele
seja gravado, e a chamada volta com sucesso do mesmo jeito - nao use. Isso vale, entre outros, para:

- `lock_to_single_conversation` (conversa unica)
- `csat_survey_enabled` e `csat_config` (pesquisa de satisfacao)
- `portal_id` (central de ajuda vinculada)
- `timezone` e `working_hours_enabled` da caixa
- `sender_name_type` e `business_name` (assinatura do remetente)
- `allow_messages_after_resolved`, `enable_email_collect`

Tudo isso e feito no painel, na aba Configuracoes da propria caixa. Nao afirme que configurou nenhum
desses itens: proponha o valor, explique onde clicar e peca ao cliente que salve. Depois releia a
caixa com `lionchat_inboxes_show` e confirme.

**Interruptores do canal: `channel.additional_attributes`.** Eles vivem dentro do objeto do CANAL,
nao da caixa. Mandar `additional_attributes` solto, no nivel de cima, e descartado pelo servidor -
o nome certo e `channel.additional_attributes`. Regras:

- Na caixa de WhatsApp (QR Code e Oficial), Instagram, Facebook e TikTok a gravacao SOMA: so as
  chaves enviadas mudam. Na caixa de API o objeto e TROCADO inteiro.
- WhatsApp por QR Code: so estas chaves sao da tela - `accept_groups`, `reject_calls`,
  `reject_calls_message` (texto) e `mark_as_seen`. Booleano de verdade (true/false), nunca "false"
  em texto. O resto que aparece na leitura (sessao, importacao, proxy, migracao, conexao hibrida) e
  interno do sistema: NUNCA mande.
- WhatsApp Oficial: `mark_as_seen`.
- `accept_status`, `accept_channels` e `accept_broadcast` ficam SEMPRE desligados desde 09/10/2026:
  mandar true e desfeito ao salvar, sem erro.

Exemplo: `channel.additional_attributes: {"accept_groups": true, "reject_calls": false}`.

Uma observacao que salva tempo: dentro de `auto_assignment_config` qualquer chave passa pelo
conector, mas o servidor so aceita quatro (`assign_offline_agents`, `fair_distribution_limit`,
`fair_distribution_window`, `assignment_order`). Chave fora dessas quatro tambem some em silencio.

---

## 2. Nome da caixa

Texto livre e obrigatorio. Um detalhe que aparece so em caixa de E-MAIL: o nome tambem e usado como
nome do REMETENTE, e nessa hora alguns simbolos sao removidos automaticamente (barra invertida,
maior e menor que, arroba, aspas, exclamacao, cerquilha, cifrao, porcentagem, e comercial, asterisco,
mais, igual, interrogacao, acento circunflexo, crase, chaves, barra vertical, til, dois pontos e
ponto e virgula), pontuacao das pontas e cortada e espacos repetidos viram um so. Se o cliente
quiser que o destinatario veja exatamente um texto, use o campo de nome da empresa em vez de decorar
o nome da caixa.

---

## 3. Saudacao automatica

- Nasce DESLIGADA.
- Sai na PRIMEIRA mensagem do cliente, dentro do horario de atendimento.
- Exige as duas coisas: o interruptor ligado E o texto preenchido. Faltando um, nada sai.
- Aceita variaveis (nome do contato, nome da caixa, nome da conta). O nome do atendente sai em
  branco, porque nao ha atendente ainda.
- Nao sai em conversa de campanha.
- So mensagem DO CLIENTE dispara. Aviso interno do sistema (atribuicao automatica, mudanca de
  status) nao dispara mais.

Onde e feito: `lionchat_inboxes_update` alcanca os dois campos.

---

## 4. Aviso de ausencia

- Sai quando o cliente escreve FORA do horario de atendimento.
- Exige as duas coisas desde 17/09/2026: o interruptor `out_of_office_enabled` LIGADO e o texto
  `out_of_office_message` preenchido. O interruptor nasce desligado; as caixas que ja tinham texto
  nessa data foram ligadas na atualizacao. Texto em branco nao envia nada - nao existe texto padrao.
- No maximo UM por dia por conversa, contado no fuso do dono do horario.
- Nao sai se algum atendente respondeu nos ultimos 5 minutos - para nao interromper conversa viva no
  fim do expediente.
- So mensagem do cliente dispara.

Onde e feito: `lionchat_inboxes_update` com `out_of_office_enabled` e `out_of_office_message` juntos,
ou no painel, aba Configuracoes da caixa, logo abaixo da saudacao. Lembre que ele depende do horario
de atendimento da CONTA (secao 5), que continua sendo item de painel.

---

## 5. Horario de atendimento: da CONTA, nao da caixa

Esta e a confusao mais cara desta area.

- **Chat do Site** usa o horario da PROPRIA caixa (aba Horario de funcionamento).
- **Todas as outras caixas** (WhatsApp, Instagram, Facebook, Telegram, e-mail, SMS) usam o horario
  da CONTA, em Configuracoes > Conta, secao Horario Comercial.

Ou seja: cadastrar horario numa caixa de WhatsApp nao produz efeito nenhum - a aba nem aparece la.
Se a conta nao tem horario cadastrado e ligado, nenhuma caixa nao-widget consegue mandar aviso de
ausencia, e o SLA que deveria contar so no expediente passa a contar 24 horas por dia.

De fabrica, conta e caixa nascem com sabado e domingo fechados e de segunda a sexta das 09:00 as
17:00.

A pausa de almoco so existe no horario da CONTA, nunca no da caixa, e precisa ficar dentro do
expediente do dia.

**Nem o horario da conta nem o horario da caixa podem ser gravados pelas ferramentas.** Peca ao
cliente que faca no painel e passe o caminho: Configuracoes > Conta, secao Horario Comercial (para a empresa
inteira) ou aba Horario de funcionamento da caixa (so no Chat do Site). O que voce consegue gravar e
apenas o FUSO da conta, com `lionchat_account_settings_update` - e um fuso invalido e descartado em
silencio, entao releia a conta depois.

Caixa nova HERDA o fuso da conta. Por isso ajuste o fuso da conta ANTES de criar caixas.

---

## 6. Saudacao e aviso na mesma noite

Quem escreve fora do horario recebe SO o aviso de ausencia. A saudacao fica guardada e sai quando a
pessoa voltar a escrever dentro do horario. Cliente que espera as duas na mesma madrugada acha que a
saudacao quebrou - explique antes.

---

## 7. Conversa unica

"Bloquear para conversa unica": toda mensagem nova do mesmo contato cai na MESMA conversa em vez de
abrir um card novo.

- Caixa NOVA nasce com a conversa unica LIGADA, em qualquer canal, e o assistente de criacao do
  painel tambem propoe ligada.
- Caixa ANTIGA pode estar desligada: o padrao so mudou em marco de 2026 e as caixas criadas antes
  disso ficaram como estavam. Caixa desligada abre um card novo a cada mensagem e a operacao enche
  de conversas repetidas.
- Sempre confira o valor real com `lionchat_inboxes_show` antes de afirmar como esta.
- Nao e alcancavel pela ferramenta de atualizar a caixa: e item de painel.

---

## 8. Pesquisa de satisfacao

Depois que a conversa e finalizada, o sistema manda a pergunta DENTRO da propria conversa (sem link
externo) e registra a nota de 1 a 5.

| Ajuste | Valor |
|---|---|
| Ligar a pesquisa | Nasce desligada |
| Espera depois de finalizar | Padrao 300 segundos (5 minutos); a tela aceita ate cerca de 23 horas |
| Formato da nota | Numeros (padrao) ou carinhas |
| Pergunta, agradecimento e resposta invalida | Textos livres |
| Filtro por etiqueta | Enviar so para conversa que contem, ou que nao contem, certas etiquetas |

Regras que evitam reclamacao:

- **Deixar a pergunta em branco nao desliga a pesquisa.** O sistema manda um texto padrao em
  portugues. Para nao perguntar, desligue a pesquisa.
- **Em WhatsApp Oficial a pesquisa so sai se a janela de 24 horas ainda estiver aberta na hora do
  envio.** Conversa finalizada depois que o cliente sumiu simplesmente nao recebe. Quanto maior a
  espera configurada, maior o risco.
- **Lista de etiquetas vazia libera todo mundo.**
- **Nao ha modelo aprovado envolvido.** Existe uma ferramenta de modelo de pesquisa por caixa, mas
  ela e heranca do modo antigo: hoje a pesquisa e mensagem comum dentro da conversa. Nao gaste tempo
  com isso.

Onde e feito: no painel, aba CSAT da caixa. As ferramentas so LEEM os resultados
(`lionchat_csat_list`, `lionchat_csat_metrics`, `lionchat_csat_download`).

---

## 9. Ajustes especificos do Chat do Site

Todos no painel. Vale conhecer para propor:

- Endereco do site e cor do balao (obrigatorios na criacao).
- Titulo e subtitulo dentro do balao, e a expectativa de resposta mostrada ao visitante (em alguns
  minutos, em algumas horas, em um dia).
- Formulario antes de conversar (nome, e-mail, telefone, campos personalizados).
- Recursos do balao: anexos, seletor de carinhas, encerrar conversa, usar a imagem da caixa para o
  robo, permitir dentro de aplicativo.
- Dominios permitidos e validacao de identidade do visitante.
- Imagem propria no botao flutuante (PNG, WEBP, JPEG ou GIF ate 1 MB). Vazio usa o desenho padrao.
  Enviar imagem nova e so pela tela; VOLTAR ao desenho padrao tem ferramenta
  (`lionchat_inboxes_bubble_icon_destroy`, so em caixa de site - outro tipo devolve 422).
- Continuidade por e-mail: se o visitante sai, a resposta do atendente tambem vai por e-mail.
- Coleta de e-mail antes de conversar.

Desde 28/08/2026, o visitante so vira contato quando interage de verdade: mandar mensagem, mandar
arquivo, preencher o formulario, clicar numa mensagem proativa ou ser identificado pelo proprio site.
So abrir o site ou so abrir o balao nao cria contato.

---

## 10. Ajustes especificos do WhatsApp por QR Code

Interruptores da aba Configuracao: aceitar grupos, recusar ligacoes com texto de resposta e marcar
como visto. Voce grava pelo `channel.additional_attributes` (secao 1) ou o cliente no painel.
Detalhe de cada um em `canais.md`. "Aceitar canais" e "Aceitar lista de transmissao" nao existem mais.

Conexao e historico voce acompanha por ferramenta:

- `lionchat_inboxes_waha_status` diz o estado da sessao. WORKING e conectado; SCAN_QR_CODE precisa
  ler o codigo; STOPPED e FAILED sao problema.
- `lionchat_inboxes_waha_qrcode` devolve o TEXTO do codigo, nao uma imagem, e so funciona no estado
  SCAN_QR_CODE. O codigo expira em cerca de 20 a 60 segundos.
- `lionchat_inboxes_waha_connect_link_create` gera um LINK publico e temporario (2 horas) para quem
  esta com o celular ler o QR sem ter login no painel. So administrador. So um link vale por vez;
  caixa ja conectada devolve 422. `lionchat_inboxes_waha_connect_link_revoke` mata o link e derruba a
  sessao que ele abriu - revogue assim que a conexao terminar ou se o link vazar. Gerar o link e acao
  de escrita: confirme antes.
- **Queda avisa por e-mail.** Se a conexao cai e FICA fora (o sistema espera cerca de 30 minutos), os
  administradores da conta recebem um e-mail com o link de reconexao - um so por queda. Cobre tambem
  quem removeu o LionChat dos "Aparelhos conectados" no celular. Varias caixas caindo juntas e
  problema da plataforma e nao gera e-mail.
- **Numero adicionado a um grupo** com "Aceitar Grupos" ligado ganha a conversa do grupo na hora, com
  uma pilula dizendo como entrou.
- `lionchat_inboxes_waha_import_history` traz o historico do aparelho, de 1 a 90 dias. Uma rodada por
  vez. O texto entra em conversas de historico separadas, ja finalizadas e datadas no passado, sem
  acordar automacao nem agente de IA. As fotos e audios chegam depois, numa fila: bolha ambar
  significa "chegando", vermelha significa que o WhatsApp ja apagou o arquivo e ele nao volta.
- `lionchat_inboxes_waha_import_status` acompanha o andamento.

---

## 11. Ajustes especificos do WhatsApp Oficial

- O numero e o endereco de todas as conversas da caixa e nao e editavel depois.
- `lionchat_inboxes_health` mostra o painel de Saude da conta na Meta: se da para enviar, o motivo
  quando nao da, a nota de qualidade, a faixa de limite diario, quanto ja foi usado nas ultimas 24
  horas, as falhas de envio das ultimas 24 horas por motivo e (desde 06/10/2026, campo
  `charges_30d`) as mensagens ENTREGUES nos ultimos 30 dias separadas em cobradas e gratis, por tipo.
  Nao ha valor em reais: a Meta informa quantidade e tipo, nao preco. "Cobradas" sao so os modelos
  pagos que chegaram ao cliente; resposta comum dentro da janela nao custa.
- **Vigia do numero.** O sistema pergunta a Meta de tempos em tempos se o numero ainda esta na conta
  do WhatsApp do cliente e se a caixa continua vinculada. Se nao estiver: bolinha vermelha na caixa
  e um e-mail para os administradores (um por queda, depois de cerca de 30 minutos). O conserto e no
  painel: aba Configuracao, botao Reconfigurar.
- `lionchat_inboxes_capi_dataset_link` vincula a caixa ao conjunto de dados da Meta usado pelas
  conversoes de anuncio de WhatsApp (ver a skill `integracoes`). Hoje isso e feito sozinho; a
  ferramenta e o plano B. Acao de escrita: confirme antes.
- Modelos de mensagem aprovados: `lionchat_inboxes_sync_templates` sincroniza e as ferramentas de
  modelo listam, criam e alteram.
- Trazer o historico: `lionchat_inboxes_whatsapp_history_start`, `_status` e `_cancel`.
- Conexao hibrida (enviar fora das 24 horas): `lionchat_inboxes_coex_backup_connect`, `_status` e
  `_disconnect`. A leitura do QR e no celular; para quem nao tem login no painel ha o link
  `lionchat_inboxes_coex_backup_connect_link_create` / `_revoke` (mesmas regras do link da caixa QR).
  Regras em `canais.md`, secao 8.
- Ligacao pelo WhatsApp Oficial: `lionchat_inboxes_enable_whatsapp_calling` e
  `lionchat_inboxes_disable_whatsapp_calling`. Linha de ligacao LionCalls e Ligacao com IA: skill
  `agente-de-ligacao`.

Se a caixa foi criada a mao e voce precisar corrigir alguma credencial, o objeto de credenciais e
SUBSTITUIDO inteiro: leia o atual e reenvie tudo, senao a caixa para de receber em silencio. Na
duvida, mande fazer no painel.

---

## 12. Falhas de envio

Aba por caixa, so para administrador, disponivel em WhatsApp por QR Code, WhatsApp Oficial e
Facebook Messenger. NAO aparece no Instagram. Lista as mensagens que falharam nas ultimas 24 horas
agrupadas por motivo:

- falta de saldo ou pagamento
- erro comum, que vale reenviar
- janela de 24 horas fechada (precisa de modelo aprovado)
- erro definitivo (numero invalido, modelo recusado)
- mensagem de campanha, que nunca reenvia por ali
- envio parcial de varios anexos, que nao reenvia para nao duplicar no cliente

Ferramentas: `lionchat_inboxes_failed_messages_summary` para ver,
`lionchat_inboxes_failed_messages_bulk_retry` para reenviar em lote e
`lionchat_inboxes_failed_messages_bulk_cancel` para cancelar. Reenviar em lote entrega mensagem ao
cliente final: peca confirmacao explicita antes.
