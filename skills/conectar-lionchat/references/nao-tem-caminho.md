# O que so o painel faz

Antes de dizer ao cliente que alguma coisa nao da, confira se esta aqui. Se estiver, a resposta
certa nao e "nao existe" — e "isso voce faz na tela, assim". E diga isso ANTES de comecar o
trabalho, nao no fim, para o cliente nao esperar por algo que nao vem.

Indice:

1. Conectar canais
2. Dinheiro
3. Arquivos e planilhas
4. Liberacao de acesso
5. Coisas que soam parecidas e nao existem
6. Como avisar isso no resumo final

---

## 1. Conectar canais

| O que o cliente pede | Por que nao da pela IA | O que dizer |
|---|---|---|
| Conectar o WhatsApp por QR Code | o codigo precisa ser lido com a camera do celular dele | "Quem le o QR Code com o celular e voce. Eu posso gerar um link de conexao para quem esta com o celular na mao abrir e ler, sem precisar entrar no painel — ou voce vai em Caixas de Entrada, abre a caixa e escaneia." |
| Conectar Instagram ou Facebook | a Meta exige o clique dele na tela de permissao, e nao ha ferramenta nenhuma pra isso | "A Meta exige que voce autorize clicando, e isso so existe na tela. Eu nao consigo clicar por voce." |
| Conectar Google Agenda, Google Contatos ou Conta Azul | quem autoriza e ele, na tela do Google / da Conta Azul | "Eu gero o link de autorizacao e te mando; quem abre e clica em Permitir e voce. Depois disso eu configuro o resto." |
| Conectar o WhatsApp oficial (Cadastro Incorporado da Meta) | a janela de cadastro da Meta e uma tela, com cliques | "O cadastro do numero oficial acontece numa janela da Meta que voce precisa preencher. Depois disso eu configuro o resto." |

O que voce CONSEGUE fazer nessa area:

- ligar ou religar a sessao do WhatsApp QR Code da caixa e ver a situacao dela;
- **gerar um LINK PUBLICO de conexao** (desde 28/08), que abre o QR numa pagina para quem esta com o
  celular na mao, sem acesso ao painel. Tres tipos: caixa QR Code
  (`lionchat_inboxes_waha_connect_link_create`), apoio da conexao hibrida de uma caixa oficial
  (`lionchat_inboxes_coex_backup_connect_link_create`) e linha de ligacao
  (`lionchat_inboxes_lioncalls_connect_link_create`). Devolvem o endereco e a validade. So administrador;
  caixa ja conectada volta 422. **O link conecta um canal de verdade:** peca o "sim" antes, mande so para
  quem vai ler o QR e revogue (`..._connect_link_revoke`) assim que conectar ou se o link vazar;
- buscar o codigo QR. **Ele volta como TEXTO, nao como imagem** — para o cliente escanear, esse
  texto precisa virar um desenho de QR, entao na pratica o caminho facil e ele abrir a caixa no
  painel. O codigo tambem vence em menos de um minuto;
- gerar o link de autorizacao do Google Agenda, do Google Contatos e da Conta Azul;
- ver a saude das caixas, listar e conferir modelos de WhatsApp, e configurar tudo o que vem depois
  que o canal ja esta conectado.

---

## 2. Dinheiro

**Nenhuma ferramenta de assinatura, plano, fatura, forma de pagamento, cartao, saldo ou recarga.**
Foram removidas de proposito e nao voltam — nem as de so leitura.

O que dizer:

> Eu nao mexo em plano, fatura, cartao nem saldo, por seguranca. Isso fica no painel, na area de
> assinatura da conta.

**Voce nao MEXE, mas consegue LER o essencial.** `lionchat_account_show` devolve o nome do plano, a
situacao da assinatura e, em `resource_limits`, quantos atendentes e quantas caixas de entrada a
conta ja usa e quantos o plano permite. Da pra dizer ao cliente "voce esta com 12 de 12 atendentes"
antes de tentar criar o decimo terceiro. O que nao existe e mexer: contratar, cancelar, trocar
cartao, ver fatura ou saldo.

---

## 3. Arquivos e planilhas

**Enviar arquivo DIRETO do computador nao existe na conexao por IA.** Arquivo que esta num LINK publico,
sim (ver "Os caminhos que existem" abaixo). Quatorze ferramentas tem campo de arquivo. Em **dez** delas o
arquivo e obrigatorio: essas aparecem na sua lista e sempre falham. Sao:

- importar contatos por planilha (tres delas: importar, conferir antes e a versao com mapeamento)
- importar cards do funil por planilha, e o preview dessa importacao
- anexar arquivo em card e anexar arquivo em nota de card
- subir arquivo de resposta pronta
- subir midia de cabecalho de modelo do WhatsApp

Nas outras quatro o arquivo e OPCIONAL e o resto da ferramenta funciona normalmente — so a imagem
nao sobe: foto de perfil da conta, foto de perfil do contato, `lionchat_upload_create` (que aceita
endereco de arquivo) e `lionchat_captain_documents_create` (que aceita endereco de pagina ou texto).

**Os caminhos que existem:**

- Muitos contatos de uma vez: `lionchat_contacts_bulk_create` — **so no conector remoto**, ate 1000
  por chamada. Cada campo personalizado usado precisa JA EXISTIR na conta, senao a importacao
  descarta em silencio. A importacao roda em segundo plano: confira depois buscando os contatos.
- Muitas empresas de uma vez: `lionchat_companies_bulk_create` — **so no conector remoto**, ate 1000 por
  chamada (`check_only: true` confere sem gravar); acompanhe com `lionchat_companies_import_status`.
- **Arquivo que ja esta na internet (desde 23/10 aceita foto, video, audio e documento — PDF, Word,
  Excel, CSV, ZIP):** `lionchat_upload_create` com `external_url` devolve `blob_id`. Para mandar o
  arquivo numa conversa, ponha esse `blob_id` no campo `attachments` de
  `lionchat_conversations_messages_create` (vale tambem em conversa de grupo do WhatsApp QR Code).
  Para a Biblioteca de Midia do agente de IA, o mesmo `blob_id` vai em `blob_signed_id`. Link que abre
  uma PAGINA (ex.: Google Drive `.../view`) e recusado — use o link de download direto. Teste antes com
  `lionchat_upload_validate` e confira o teto de cada tipo em `lionchat_upload_limits_show`.
- Base de conhecimento do agente de IA: da para criar por endereco de pagina ou por texto colado; o
  arquivo PDF e que nao sobe.
- Qualquer outra planilha: o cliente importa pela tela.

---

## 4. Liberacao de acesso

| O que o cliente pede | Onde se faz |
|---|---|
| Ligar o recurso de conexao por IA numa conta | so o suporte do LionChat ou o plano. Nao existe ferramenta |
| Liberar um usuario especifico | idem |
| Gerar ou trocar o Token de acesso | painel: Configuracoes do Perfil, bloco "Token de acesso" |
| Criar ou revogar um conector do modo remoto | painel: Configuracoes do Perfil, secao Conectores, "Conectores de IA" |

---

## 5. Coisas que soam parecidas e nao existem

### "Quero plugar o MEU sistema (ERP, financeiro, agenda) no agente de IA"

Isso **nao existe** hoje. A tela "Conectores de IA" e o caminho contrario: e uma IA de fora usando
as ferramentas do LionChat. Ligar um sistema do proprio cliente como fonte para o agente de IA dele
nunca foi construido.

**O substituto que existe hoje:** uma ferramenta de IA montada como fluxo, que chama o sistema do
cliente por endereco de internet. O agente de IA usa essa ferramenta quando precisa. Isso e assunto
da skill `agente-de-ia` e da skill `criar-fluxos`.

Como responder:

> Hoje nao da para plugar seu sistema direto como conector do agente de IA. O que da, e funciona
> bem, e montar uma ferramenta que a IA aciona quando precisa e que fala com seu sistema. Quer que
> eu monte?

### Vincular agenda por cenario no agente de IA

Da para restringir quais tipos de agendamento um agente de IA enxerga, mas nao por cenario. Isso se
faz na tela.

### Sessoes do servidor de WhatsApp por QR Code

Existia e foi **removido** por um problema de seguranca entre contas. Nao volta. O que sobrou e o
que age numa caixa especifica: ver situacao, buscar o codigo QR, importar historico.

---

## 6. Como avisar isso no resumo final

Sempre feche o trabalho com uma secao explicita. Nao deixe o cliente descobrir sozinho que faltou
alguma coisa:

```
So voce consegue fazer, na tela:

  1. Conectar o WhatsApp: Caixas de Entrada > [nome da caixa] > ler o QR Code com o celular
  2. Autorizar o Instagram: Caixas de Entrada > adicionar canal > Instagram > clicar em Permitir
  3. [outros itens que apareceram neste trabalho]

Depois que voce fizer isso, me chama que eu configuro o resto.
```
