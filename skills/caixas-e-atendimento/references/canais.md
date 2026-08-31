# Canais: o que da e o que nao da em cada um

Indice:
1. Tabela dos tipos de caixa
2. WhatsApp Oficial x WhatsApp por QR Code - a decisao que nao tem volta
3. Janela de conversa (ate quando da para responder com texto livre)
4. Grupos, listas de transmissao e canais do WhatsApp
5. Campanha (disparo em massa): quais canais aceitam
6. Fluxo: as restricoes de canal que recusam o salvar
7. Abas que aparecem em cada tipo de caixa
8. Conexao hibrida (WhatsApp de apoio na caixa Oficial)
9. Migrar conversas de uma caixa para outra
10. Excluir uma caixa: o que acontece de verdade

---

## 1. Tabela dos tipos de caixa

Na tela de criacao (Configuracoes > Caixas de Entrada > Adicionar) sao oferecidos NOVE tipos:

| Tipo na tela | Para que serve |
|---|---|
| Website | Balao de chat ao vivo no site do cliente |
| WhatsApp | Numero na API oficial da Meta (conta comercial verificada) |
| WhatsApp QR Code | Numero conectado lendo um QR Code, igual ao WhatsApp Web |
| E-mail | Conta de e-mail para receber e responder mensagens |
| Telegram | Um bot do Telegram |
| Facebook Messenger | Uma pagina do Facebook |
| Instagram | Mensagens diretas de uma conta do Instagram |
| TikTok | Mensagens diretas de uma conta do TikTok |
| Canal da API | Canal generico para integracao propria |

Nao prometa SMS, LINE nem Voz (Twilio) como opcao dessa tela: eles nao aparecem na lista. Se o
cliente pedir um desses, oriente-o a falar com o suporte antes de qualquer promessa.

A caixa e um envelope comum a todos: nome, saudacao, aviso de ausencia, conversa unica, distribuicao,
pesquisa de satisfacao. O canal e um registro separado pendurado nela. Por isso muitos ajustes valem
para qualquer canal e alguns poucos so existem em um.

---

## 2. WhatsApp Oficial x WhatsApp por QR Code

Esta e a decisao mais importante da area, porque troca de canal depois exige migrar conversas e
reconectar tudo.

| | WhatsApp Oficial | WhatsApp por QR Code |
|---|---|---|
| Como conecta | Botao da Meta no painel (login na Meta) ou os 4 dados na mao | Le um QR Code com o celular |
| Precisa de conta comercial verificada | Sim | Nao |
| Selo de conta verificada | Sim | Nao |
| Janela de 24 horas | Sim | Nao |
| Disparo em massa com modelo aprovado | Sim | Nao ha modelo, mas ha campanha |
| Grupos do WhatsApp | Nao existe | Sim, e so aqui |
| Lista de transmissao e canais do WhatsApp | Nao existe | Sim |
| Trazer o historico antigo do celular | Existe pelo caminho de coexistencia | Sim |
| Painel de Saude da conta na Meta | Sim | Nao |
| Recusar ligacao recebida no WhatsApp | Nao | Sim |

Regra pratica: se o cliente citar GRUPO, lista de transmissao ou "quero trazer as conversas antigas
do meu celular", a resposta e QR Code e nao ha alternativa. Se ele citar selo azul, verificacao de
empresa ou disparo com modelo aprovado, e Oficial e a janela de 24 horas vem junto.

---

## 3. Janela de conversa

A janela conta a partir da ultima mensagem que o CLIENTE mandou. Quando ela fecha, a tela bloqueia o
envio de texto livre.

| Canal | Janela |
|---|---|
| WhatsApp Oficial | 24 horas |
| Facebook Messenger e Instagram | 24 horas (7 dias se a instalacao tiver o modo de agente humano ligado) |
| TikTok | 48 horas |
| Twilio com WhatsApp | 24 horas |
| WhatsApp por QR Code, Chat do Site, E-mail, Telegram, SMS, LINE | Sem janela |
| Canal da API | Definida pelo proprio cliente, em horas |

Duas consequencias que confundem o cliente:

- **Conversa trazida por importacao de historico nunca aceita resposta livre.** Aquelas mensagens
  nunca passaram pela conexao oficial, entao para a Meta o contato nunca escreveu. A janela fica
  fechada mesmo que a fala do cliente seja de hoje de manha.
- **Mandar modelo aprovado NAO reabre a janela.** So a mensagem do cliente reabre.

---

## 4. Grupos, listas de transmissao e canais do WhatsApp

So existem na caixa por QR Code, e quase todos tem um interruptor proprio na aba de configuracao
daquela caixa:

| Interruptor | Nasce | O que faz |
|---|---|---|
| Aceitar Grupos | desligado | Grupo do WhatsApp vira uma conversa no painel |
| Aceitar Canais | desligado | Mensagens de canais do WhatsApp |
| Aceitar Lista de Transmissao | desligado | Mensagens de lista de transmissao |
| Status / stories | sempre desligado | NAO aparece na tela: foi escondido e o servidor forca desligado por consumo de memoria. Gravar ligado nao adianta |
| Recusar ligacoes | desligado | Recusa chamada de voz e video recebida, com uma resposta de texto opcional |
| Marcar como visto | ligado | Deixa o check azul no celular do contato quando o atendente abre a conversa |

Com "Aceitar Grupos" desligado, mensagem de grupo e DESCARTADA em silencio: nenhum erro, nenhuma
conversa. Ligar ou desligar Aceitar Grupos, Canais ou Lista de Transmissao REINICIA a sessao do
WhatsApp por alguns segundos. O interruptor de recusar ligacoes nao reinicia.

Grupo no LionChat vira um contato de verdade, com nome real do grupo, foto e o nome de quem falou em
cada mensagem. Ha ferramentas para listar grupos, colocar e tirar participante, promover e rebaixar
administrador, renomear, mudar descricao, trocar foto, pegar e revogar o link de convite e sair do
grupo - todas so na caixa por QR Code. CRIAR um grupo NAO tem ferramenta: isso e feito por um bloco
de Fluxo, nao por aqui.

---

## 5. Campanha (disparo em massa)

Campanha so existe nestes tipos de caixa: Website, SMS, Twilio SMS, WhatsApp Oficial, WhatsApp por
QR Code e Canal da API. Instagram, Facebook, Telegram, E-mail e Voz NAO fazem campanha - a criacao e
recusada com "tipo de caixa nao suportado".

Isso e criterio de escolha de canal: nao adianta criar a caixa de Instagram planejando disparar em
massa por ela.

---

## 6. Fluxo: restricoes de canal

Se o cliente pretende usar Fluxos, tres regras decidem o canal antes de tudo, porque elas RECUSAM o
salvar do fluxo:

- Um fluxo so aceita caixas do MESMO canal. Nao da para um fluxo unico atender WhatsApp e Instagram.
- Fluxo em caixa de WhatsApp Oficial aceita UMA caixa apenas. Quem monta quatro numeros oficiais nao
  consegue um fluxo unico para os quatro.
- Fluxo de grupo e gatilho de entrada/saida de participante so funcionam em caixa por QR Code.

---

## 7. Abas que aparecem em cada tipo de caixa

Sempre presentes: Configuracoes e Agentes (a lista de quem atende).

| Aba | Onde aparece |
|---|---|
| CSAT (pesquisa de satisfacao) | Todos, menos Voz |
| Horario de funcionamento | So Chat do Site |
| Formulario pre-chat e Construtor de Widget | So Chat do Site |
| Conexao, Configuracao e Importar Historico do WhatsApp | So WhatsApp por QR Code |
| Saude da conta, Historico | So WhatsApp Oficial |
| Falhas de envio | WhatsApp por QR Code, WhatsApp Oficial e Facebook Messenger, e so para administrador. NAO aparece no Instagram |
| Migracao | WhatsApp por QR Code e Canal da API |
| Ligacao (LionCalls) | Caixa por QR Code ou WhatsApp Oficial, com o recurso de voz liberado na conta e vaga de caixa de voz disponivel |
| Campanhas | So nos canais que aceitam campanha |

Se a aba Ligacao nao aparecer numa caixa nova, o motivo mais provavel nao e defeito: a conta atingiu
o limite de caixas de voz do plano.

---

## 8. Conexao hibrida (WhatsApp de apoio na caixa Oficial)

Pendura uma sessao de WhatsApp por QR Code numa caixa OFICIAL, so para ENVIAR quando a janela de 24
horas ja fechou, sem precisar de modelo aprovado.

- Exige caixa Oficial em coexistencia e o recurso liberado para a conta.
- O numero precisa ser o MESMO da caixa. O sistema confere e desfaz a conexao na hora se for
  diferente.
- A conexao tambem e por leitura de QR Code no celular.

---

## 9. Migrar conversas de uma caixa para outra

Existe na caixa por QR Code (aba Migracao) e move conversas para outra caixa, de QR Code ou Oficial.
Sempre rode a previa antes. Duas coisas que o cliente precisa ouvir ANTES:

- Grupos, listas de transmissao e canais NAO existem no WhatsApp Oficial: ficam para tras na caixa
  antiga. Se ele apagar a caixa velha depois de migrar, essas conversas se perdem.
- As conversas migradas tem mais de 24 horas: na caixa Oficial nenhuma delas aceita resposta livre,
  so modelo aprovado.

---

## 10. Excluir uma caixa

E a acao mais perigosa da area. Voce nunca faz isso - so explica. E irreversivel, so administrador
faz, e a confirmacao exige digitar o nome da caixa.

O que acontece:

- As conversas e mensagens NAO sao apagadas. Elas ficam em modo somente leitura, com o aviso "A
  caixa de entrada foi removida" no topo, e um botao para falar com aquele contato por outra caixa.
- As CAMPANHAS ligadas aquela caixa sao removidas junto. Consulte antes com a ferramenta que lista
  as campanhas da caixa.
- Em caixa por QR Code o WhatsApp do cliente e DESCONECTADO do celular e a sessao e apagada do
  servidor. Ele precisa ler o QR Code de novo para reconectar.

Se o cliente ja apagou e quer trazer as conversas orfas de volta para outra caixa, existe uma
ferramenta que religa a conversa a outra caixa - ela recusa quando a caixa de destino ja tem uma
conversa ativa com o mesmo contato.
