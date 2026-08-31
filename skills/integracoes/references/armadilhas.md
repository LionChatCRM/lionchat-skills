# Armadilhas: o que falha em silencio

Todas as linhas abaixo tem a mesma forma: a coisa e criada, parece certa na tela, ninguem recebe erro
- e nao funciona. Sao as que mais custam, porque o cliente so descobre quando alguem reclama que nao
recebeu mensagem.

---

## Montagem

**Se voce criar a integracao sem alvo mapeado**, o evento chega, aparece como recebido no historico,
o contato e criado ou atualizado, e nada mais acontece. Nenhum erro em lugar nenhum. E o "configurei
e nao funciona" numero um. *Saida:* grave o mapeamento evento para alvo.

**Se a automacao alvo estiver sem caixa de entrada**, todo evento morre. A frase exata - "Automacao
sem caixa de envio configurada" - so aparece no historico de execucao da automacao
(`lionchat_automation_rules_list_1`), nunca no historico da integracao. *Saida:* grave `inbox_id` na
automacao.

**Se voce escolher a caixa no cadastro do checkout achando que resolveu**, nao resolve: esse campo e
gravado e nunca lido. A caixa sai da automacao ou do fluxo. *Saida:* ignore o campo e cuide do alvo.

**Se a automacao alvo estiver desativada**, ela e pulada em silencio - por seguranca, e proposital.
*Saida:* `active: true`.

**Se voce criar a integracao antes do alvo**, a pessoa fica com o endereco ja cadastrado no sistema
de fora e nada acontecendo por dias. *Saida:* alvo primeiro, sempre.

---

## Fluxo como alvo

**Se o fluxo nao tiver o gatilho "Webhook recebido" gravado no bloco Inicio**, ele nunca dispara. A
chave exata e `webhook_received`. Hoje o sistema corrige sozinho quem gravou so `webhook`, mas escreva
o nome certo mesmo assim - nenhum outro nome de gatilho tem essa correcao.

**Se o fluxo estiver desligado**, nao dispara e nao avisa.

**Se o fluxo for do tipo ferramenta de IA** (e nao fluxo de conversa), ele e recusado por seguranca -
integracao de fora nao pode acionar ferramenta da IA.

**Se o fluxo nao tiver caixa de entrada vinculada**, nao dispara.

**Se ja existir uma sessao daquele fluxo aberta naquela conversa**, o evento novo e DESCARTADO. E a
armadilha mais traicoeira da lista: uma sessao presa (esperando resposta sem tempo limite, ou pausada
por um atendente) faz a integracao parar de funcionar **so para aquele cliente, para sempre**. A
segunda compra do mesmo comprador nao dispara nada. *Saida:* confira em
`lionchat_flows_executions_list` se ha sessao aberta antiga naquela conversa.

**Se o contato nao tiver telefone e a caixa for de WhatsApp**, para ali.

---

## Nomes e mapeamento

**Se voce montar o mapeamento da Greenn com os codigos numericos que a tela mostra**, ele nunca casa:
o sistema recebe os eventos da Greenn com NOME (`sale_paid`, `contract_canceled`,
`checkout_abandoned`). Webhook chega, e gravado como recebido, e nada dispara. *Saida:* use os nomes
de `catalogo-de-eventos.md`.

**Se voce escrever o evento da Eduzz com sublinhado** (`myeduzz_invoice_paid`), nao casa - o nome tem
ponto (`myeduzz.invoice_paid`).

**Se voce marcar os eventos na lista do assistente achando que isso filtra**, nao filtra: essa lista
e so memoria da tela. Evento marcado e nao mapeado nao faz nada; evento mapeado e nao marcado dispara
do mesmo jeito. Desmarcar um evento nao o desliga.

**Se voce gravar uma chave que inventou no mapeamento** (por exemplo "mandar_mensagem"), ela e aceita,
entra no banco, volta na resposta da API - e nunca e lida. As unicas chaves lidas sao
`automation_id`, `flow_id` e `first_purchase_only`. O cliente acha que configurou algo.

**Se voce gravar "so a primeira compra" fora dos seis checkouts que a suportam**, ela e aceita,
aparece na resposta e nao faz nada. Ela so funciona onde o evento traz o numero da cobranca: Hotmart,
Guru, Kiwify, Ticto, Monetizze e Eduzz. Na **Greenn**, na **Omie**, na **Conta Azul** e no **Webhook
Universal** e decoracao - a renovacao dispara do mesmo jeito e o cliente recebe as boas-vindas de novo.

**Se voce inverter o sentido do de-para de campos**, tudo fica vazio sem erro e o lead morre com "Lead
sem telefone nem CPF nos campos mapeados". O sentido e `{caminho do que chegou: campo daqui}`.

**Se voce mapear para uma chave de conversa reservada** (a pior e `imported_from`), ela e recusada e
o cliente ve "o atributo nao salvou" sem explicacao. A lista das 14 esta em `mapeamento-de-campos.md`.

**Se voce mapear atributo de CONVERSA sem ter automacao ou fluxo no alvo**, o valor e descartado: sem
alvo nao nasce conversa. Acontece sempre no "puxar leads antigos" do Meta Lead Ads.

**Se voce atualizar um mapeamento mandando so a chave nova**, o resto some. Leia, junte e mande o
objeto inteiro. (A unica excecao e a configuracao de eventos de conversao por etapa do funil, que
junta sozinha.)

---

## Condicoes de automacao

**Se voce montar condicoes numa automacao de integracao** (evento "webhook") - por exemplo "so
dispara se a etiqueta for VIP" -, a tela deixa salvar e o motor NAO executa. A automacao roda em
TODO evento mapeado.

*Saida:* quem filtra e o mapeamento (um evento por automacao diferente) ou um fluxo, que tem bloco de
condicao de verdade. Diga isso ao cliente com essas palavras: "nesse tipo de automacao as condicoes
nao valem".

---

## Dados que se atropelam

**Se o cliente usar dois checkouts**, a compra mais recente sobrescreve a anterior no mesmo contato:
os 7 gravam nos MESMOS 19 campos, sem separacao por plataforma. Uma mensagem que cita o nome do
produto pode citar o produto da outra plataforma. So `payment_source` diz de onde veio. (Omie, Conta
Azul e e-Clinica tem prefixo proprio e nao se atropelam.)

**Se dois lembretes da e-Clinica caem no MESMO agendamento com o mesmo numero de dias de
antecedencia**, eles disputam a mesma linha e o paciente recebe so um - o ultimo gravado vence. Isso
vale mesmo que os filtros sejam diferentes, desde que os dois casem com aquele agendamento. Dois
lembretes de dias diferentes convivem sem problema.

---

## Conversao para anuncio

**Se voce testar arrastando o cartao para tras no funil**, nao dispara nada. So movimento para frente
conta.

**Se voce ligar so o Google Analytics achando que e "so medicao"**, e o Pixel e o Google Ads estiverem
ligados, os tres disparam juntos: um movimento de cartao vira ate tres envios.

**Se voce conferir a conexao do Google Analytics pela tela**, ela parece certa mesmo com a credencial
errada - o Google responde sucesso de qualquer jeito e o historico diz "enviado". So o teste de
conexao revela.

**Se o cartao nao tiver contato vinculado**, nao ha conversao.

**Se voce ligar a etapa sem escolher o nome do evento**, ela e ignorada.

**Se voce gravar uma chave fora da lista** no bloco da etapa, ela e descartada em silencio. Aceitas:
`enabled`, `name`, `is_standard`, `value_strategy`, `value_fixed`, `currency`.

---

## Meta Lead Ads

**Se o formulario estiver ativo, mapeado, e o historico de eventos estiver VAZIO**, provavelmente a
captura de leads de anuncio nao esta liberada nesta instalacao. O lead e descartado antes de virar
registro, entao nao ha nada para diagnosticar. Peca ao suporte para conferir.

**Se o formulario estiver desativado**, o lead chega e e jogado fora. Ele se perde.

**Se voce puxar os leads antigos esperando que as boas-vindas saiam**, nao saem. Isso cria contato e
so - de proposito, para nao mandar mensagem retroativa para gente de semanas atras.

**Se voce conectar sem escolher as paginas**, entram TODAS as que o Facebook liberar (ja aconteceu de
serem dezenas). Remover exige a remocao em lote; uma a uma estoura o tempo do pedido.

---

## Webhook Universal

**Se o conteudo enviado passar de 1 MB**, e recusado com um erro que o cliente nao entende. Peca ao
sistema de fora que mande so os campos que interessam.

**Se o sistema de fora disser "entreguei"**, isso NAO prova que o evento existe aqui: o endereco
responde sucesso de proposito mesmo quando falha ao gravar, para a plataforma de fora nao ficar
tentando de novo. Confira sempre no historico.

**Se voce mapear posicao de lista** (`itens.0.nome`), lembre que e posicional: se a ordem mudar la, o
campo muda aqui.

---

## Historico e prova

**Se voce ler o historico muito depois do teste**, o evento pode ter sumido: nos 7 checkouts, no
Webhook Universal, na Omie e na Conta Azul sao os 50 mais recentes, sem paginacao (a e-Clinica e o
Meta Lead Ads sao as excecoes: os dois paginam). Numa conta movimentada isso passa em minutos, e voce
vai concluir "nao chegou" quando chegou. Leia logo depois do disparo.

**Se voce parar no "processado" do historico da integracao**, voce parou cedo demais. "Processado"
significa "o evento foi tratado", nao "a mensagem saiu". O motivo real esta no historico de execucao
da automacao ou do fluxo.

**Se voce reprocessar um evento de checkout sem a conferencia previa**, pode reenviar boas-vindas para
quem ja recebeu. A conferencia (`retry_preflight`) responde se ja existe outro evento igual
processado. Ela existe so nos 7 checkouts - no Webhook Universal, na Omie, na Conta Azul e na
e-Clinica o reprocessamento e cego.

---

## Webhook de saida

**Se o endereco falhar 10 vezes seguidas**, o envio fica 1 hora bloqueado e o que acontece nessa hora
quase nao deixa registro. Passada a hora, o proximo envio serve de teste e o bloqueio se desfaz
sozinho se o destino voltar. *Saida imediata:* trocar o endereco zera o bloqueio na hora.

**Se voce inventar um nome de aviso fora da lista fechada**, o cadastro inteiro e recusado. A lista
esta em `ferramentas-mcp.md`.

---

## Painel de Aplicativos

**Se o site de fora recusar ser exibido dentro de outra pagina**, o aplicativo aparece em branco
dentro do LionChat. Nao ha o que consertar aqui: e uma regra do site de fora. Confira com a pessoa
antes de montar.
