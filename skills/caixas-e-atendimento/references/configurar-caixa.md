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
| `additional_attributes` | Atributos extras do canal |

**Campo que nao esta nessa lista e APAGADO antes de sair.** A chamada volta com sucesso, o servidor
responde 200 e o banco continua igual. Isso vale, entre outros, para:

- `out_of_office_message` (aviso de ausencia)
- `lock_to_single_conversation` (conversa unica)
- `csat_survey_enabled` e `csat_config` (pesquisa de satisfacao)
- `portal_id` (central de ajuda vinculada)
- `timezone` e `working_hours_enabled` da caixa
- `sender_name_type` e `business_name` (assinatura do remetente)
- `allow_messages_after_resolved`, `enable_email_collect`

Tudo isso e feito no painel, na aba Configuracoes da propria caixa. Nao afirme que configurou nenhum
desses itens: proponha o valor, explique onde clicar e peca ao cliente que salve. Depois releia a
caixa com `lionchat_inboxes_show` e confirme.

**`additional_attributes` chega no corpo no nivel de cima**, e os interruptores do canal (aceitar
grupos, recusar ligacoes, marcar como visto) vivem dentro do objeto do CANAL, nao da caixa. Trate
esses interruptores como item de painel: aba Configuracao da caixa por QR Code.

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
- Campo em branco significa que nada e enviado. Nao existe texto padrao.
- No maximo UM por dia por conversa, contado no fuso do dono do horario.
- Nao sai se algum atendente respondeu nos ultimos 5 minutos - para nao interromper conversa viva no
  fim do expediente.
- So mensagem do cliente dispara.

Onde e feito: no painel, aba Configuracoes da caixa, logo abaixo da saudacao. A ferramenta de
atualizar a caixa NAO alcanca esse campo.

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
- Continuidade por e-mail: se o visitante sai, a resposta do atendente tambem vai por e-mail.
- Coleta de e-mail antes de conversar.

Desde 28/08/2026, o visitante so vira contato quando interage de verdade: mandar mensagem, mandar
arquivo, preencher o formulario, clicar numa mensagem proativa ou ser identificado pelo proprio site.
So abrir o site ou so abrir o balao nao cria contato.

---

## 10. Ajustes especificos do WhatsApp por QR Code

Na aba Configuracao da caixa (painel): aceitar grupos, aceitar canais, aceitar lista de transmissao,
recusar ligacoes com texto de resposta e marcar como visto. Detalhe de cada um em `canais.md`.

Conexao e historico voce acompanha por ferramenta:

- `lionchat_inboxes_waha_status` diz o estado da sessao. WORKING e conectado; SCAN_QR_CODE precisa
  ler o codigo; STOPPED e FAILED sao problema.
- `lionchat_inboxes_waha_qrcode` devolve o TEXTO do codigo, nao uma imagem, e so funciona no estado
  SCAN_QR_CODE. O codigo expira em cerca de 20 a 60 segundos.
- `lionchat_inboxes_waha_import_history` traz o historico do aparelho, de 1 a 90 dias. Uma rodada por
  vez. O texto entra em conversas de historico separadas, ja finalizadas e datadas no passado, sem
  acordar automacao nem agente de IA. As fotos e audios chegam depois, numa fila: bolha ambar
  significa "chegando", vermelha significa que o WhatsApp ja apagou o arquivo e ele nao volta.
- `lionchat_inboxes_waha_import_status` acompanha o andamento.

---

## 11. Ajustes especificos do WhatsApp Oficial

- O numero e o endereco de todas as conversas da caixa e nao e editavel depois.
- `lionchat_inboxes_health` mostra o painel de Saude da conta na Meta: se da para enviar, o motivo
  quando nao da, a nota de qualidade, a faixa de limite diario e quanto ja foi usado nas ultimas 24
  horas.
- Modelos de mensagem aprovados: `lionchat_inboxes_sync_templates` sincroniza e as ferramentas de
  modelo listam, criam e alteram.
- Trazer o historico: `lionchat_inboxes_whatsapp_history_start`, `_status` e `_cancel`.
- Conexao hibrida (enviar fora das 24 horas): `lionchat_inboxes_coex_backup_connect`, `_status` e
  `_disconnect`. A leitura do QR e no celular.
- Ligacao pelo WhatsApp Oficial: `lionchat_inboxes_enable_whatsapp_calling` e
  `lionchat_inboxes_disable_whatsapp_calling`.

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
