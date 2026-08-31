---
name: integracoes
description: Liga o LionChat aos sistemas que a empresa ja usa - leads de anuncio do Facebook e Instagram, compras de Hotmart, Guru, Kiwify, Eduzz, Ticto, Monetizze e Greenn, ERP Omie e Conta Azul, clinica e-Clinica, Webhook Universal para qualquer outro sistema, envio de conversao para o Pixel da Meta, Google Analytics 4 e Google Ads, Google Calendar e Google Contatos, aviso para sistema externo, Slack, Notion, Linear, Shopify, Painel de Aplicativos e SMS e voz pela TopSend. Use quando a pessoa disser "quando a compra for aprovada manda mensagem no WhatsApp", "quero receber os leads do meu anuncio", "conecta meu sistema no LionChat", "avisa o Facebook quando eu vender", "configurei a integracao e nao acontece nada", "meu webhook chega e nao dispara" ou "sincroniza minha agenda do Google". Entrevista antes, mostra a proposta por escrito e so cria depois de confirmacao explicita.
---

# Integracoes do LionChat

Integracao e a ponte entre o LionChat e os outros sistemas da empresa. Ela serve para tres coisas: TRAZER gente e informacao para dentro (quem preencheu o formulario do anuncio, a compra aprovada no checkout, a consulta marcada na clinica, a cobranca gerada no ERP), LEVAR informacao para fora (avisar o Facebook e o Google que aquele lead virou venda, avisar um sistema proprio, mandar recado no Slack) e SINCRONIZAR agenda e contatos com o Google. Sem ela, alguem teria que abrir o painel do outro sistema e copiar o telefone na mao.

**O ponto que quase todo mundo erra:** a integracao so ENTREGA o dado. Quem fala com o cliente e a AUTOMACAO ou o FLUXO ligado a ela, e a caixa de entrada de onde a mensagem sai vem do alvo, nunca da integracao. Integracao sozinha cria contato e para ali - sem erro nenhum na tela.

Voce NAO cria, altera nem apaga nada sem confirmacao explicita da pessoa.

Os nomes de ferramenta neste manual sao os do conector do LionChat (por exemplo `lionchat_integrations_apps_list`). Dependendo de como o conector foi instalado eles podem aparecer com um prefixo antes do nome - use o nome que aparecer na sua lista de ferramentas, nunca um nome inventado. Varias ferramentas de integracao tem sufixo numerico (`_create_4`, `_list_11`) e o numero pode mudar entre versoes do conector: **confira sempre pelo caminho da API em `references/ferramentas-mcp.md` antes de chamar**.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 - ENTENDER

**Primeiro leia a conta. Nunca proponha no escuro.**

- `lionchat_integrations_apps_list` - o catalogo de integracoes DESTA conta. A lista ja vem filtrada: **integracao que nao aparece nao esta disponivel para a conta** (falta liberacao ou credencial da instalacao) e o cliente nao consegue ligar sozinho. O campo `enabled` diz se ela ja esta configurada.
- `lionchat_inboxes_list` - quais caixas de entrada existem. Toda integracao de entrada termina numa conversa; sem caixa, nada adianta.
- `lionchat_automation_rules_list` e `lionchat_flows_list` - o que ja existe para reaproveitar como alvo.
- Ja existe integracao do mesmo tipo? Liste antes de criar outra (ver a tabela de `references/ferramentas-mcp.md`).

**Depois pergunte. Uma ou duas perguntas por vez, nunca a lista inteira de uma vez. Se a pessoa ja respondeu, nao repergunte.**

1. De onde vem a informacao e o que tem que acontecer quando ela chegar? (exemplo: "compra aprovada na Hotmart" e "mandar boas-vindas no WhatsApp"). Sem essa frase nao da para escolher nada.
2. Qual sistema voce usa hoje, pelo nome? (Hotmart, Guru, Kiwify, Eduzz, Ticto, Monetizze, Greenn, Omie, Conta Azul, e-Clinica, anuncios do Facebook e Instagram, outro). Se for "outro", o caminho e o Webhook Universal.
3. Voce tem acesso de administrador no painel desse sistema? Quase toda integracao exige colar um endereco nosso la, ou gerar um token la.
4. Quando o evento chegar, o cliente recebe uma mensagem pronta (automacao) ou entra numa conversa com perguntas e caminhos (fluxo)?
5. Por qual caixa de entrada essa mensagem sai? A caixa vem da automacao ou do fluxo, nao da integracao.
6. Dispara em toda compra ou so na primeira? (renovacao de assinatura conta como compra).
7. Tem mais de uma unidade ou filial? A e-Clinica exige uma configuracao e um endereco por unidade.
8. Se for anuncio: voce quer so MEDIR (Google Analytics) ou quer que o Facebook e o Google otimizem a campanha sozinhos (Pixel da Meta e Google Ads)? Sao coisas diferentes.

### Etapa 2 - DECIDIR

Classifique o pedido em UMA das cinco familias - a receita muda por completo:

| Familia | O que e | Integracoes |
|---|---|---|
| **A. Entra gente** | alguem chega de fora e precisa ser atendido | Meta Lead Ads, Webhook Universal, os 7 checkouts, Omie, Conta Azul, e-Clinica |
| **B. Sai conversao** | avisar a plataforma de anuncio que o lead avancou | Pixel da Meta, Google Analytics 4, Google Ads |
| **C. Sincroniza** | agenda e agenda de contatos do Google | Google Calendar, Google Contatos |
| **D. Avisa sistema de fora** | quando algo acontece aqui, avisar la | Webhook de saida, Slack, Linear |
| **E. Consulta no meio do atendimento** | perguntar algo a um sistema de fora durante a conversa | bloco de consulta do fluxo, consulta do formulario publico |

Heuristicas:

- **Familia A e sempre tres pecas, nesta ordem:** caixa de entrada -> alvo (automacao ou fluxo) -> integracao com o mapeamento apontando para o alvo. Criar a integracao primeiro deixa a pessoa com o endereco cadastrado e nada acontecendo.
- **Mensagem pronta e curta = automacao. Conversa com perguntas, espera e caminhos = fluxo.** Os dois podem estar ligados ao mesmo evento e disparam juntos.
- **Sistema sem integracao pronta = Webhook Universal.** Nao invente integracao que nao existe.
- **Familia B tem ordem propria:** conectar a credencial -> mapear o evento por ETAPA do funil -> testar. Sem o mapa por etapa nao sai nada, mesmo com a credencial certa.
- **"So a primeira compra" so funciona onde o evento traz o numero da cobranca:** Hotmart, Guru, Kiwify, Ticto, Monetizze e Eduzz. Na Greenn, na Omie, na Conta Azul e no Webhook Universal a opcao e aceita, aparece na resposta e nao faz nada.
- **Nao prometa filtro por condicao na automacao de integracao.** Ver a secao Armadilhas.

Leia `references/catalogo-de-eventos.md` antes de escrever qualquer nome de evento e `references/mapeamento-de-campos.md` antes de propor qualquer de-para.

### Etapa 3 - PROPOR

Mostre tudo por escrito, em linguagem de tela, e explique o porque de cada decisao em uma frase:

```
INTEGRACAO: [nome na tela]  (familia [A/B/C/D/E])

O QUE VAI ACONTECER
  [evento la fora] -> [automacao ou fluxo] -> mensagem sai pela caixa "[nome]"

O QUE EU CRIO AQUI
  1. [automacao ou fluxo] "[nome]" - caixa "[nome]"
  2. A integracao "[nome]", com o mapeamento:
       [evento] -> [alvo]
       [evento] -> [alvo]
  3. Mapeamento de campos (se for Webhook Universal ou Meta Lead Ads):
       [campo de la] -> [campo daqui]

O QUE VOCE PRECISA FAZER NA MAO (eu nao consigo)
  - [colar o endereco X no painel do sistema Y]
  - [autorizar no navegador]

O QUE FICA DE FORA
  [o que a pessoa pediu e o sistema nao faz]
```

Termine com a pergunta literal: **"Confirma que posso criar tudo isso? (s/n ou me diga o que mudar)"**

So avance com sim explicito - "sim", "pode", "confirmado", "beleza", "manda ver". Pedido de ajuste? Refaca a proposta inteira e pergunte de novo.

### Etapa 4 - EXECUTAR

**Familia A, nesta ordem (cada passo depende do anterior):**

1. **Caixa de entrada** - confirme que existe. Se nao existe, pare: conectar canal e outro assunto.
2. **Alvo.** Automacao: `lionchat_automation_rules_create` com `event_name: "webhook"`, `active: true` e **`inbox_id` obrigatorio na pratica** (sem ele o evento morre com "Automacao sem caixa de envio configurada"). A ferramenta cobra `conditions` e `actions`: mande `conditions: []` (condicao nao vale nesse tipo de automacao) e as acoes de verdade em `actions`. Fluxo: precisa estar ATIVO, ser fluxo de conversa, ter caixa vinculada e ter o gatilho **`webhook_received`** gravado no bloco Inicio. Guarde o numero do alvo.
3. **Integracao.** Crie e **guarde o `webhook_url` da resposta** quando houver. Detalhes por plataforma em `references/ferramentas-mcp.md`. Guru exige um por TIPO (transacao, assinatura, ingresso); e-Clinica tem um por unidade; Meta Lead Ads e Conta Azul **nao tem endereco nenhum** - nao procure.
4. **Entregue o endereco em texto claro**, dizendo onde colar. Enquanto ninguem colar, nao chega absolutamente nada e a tela nao tem como avisar. Na Greenn o endereco e cadastrado POR PRODUTO, nao uma vez so.
5. **Mapeamento de campos** (so Webhook Universal e Meta Lead Ads). Peca um evento de teste real ANTES: `lionchat_ecommerce_webhooks_show` devolve `source_fields`, a lista pronta de campos do ultimo webhook recebido; no Meta Lead Ads use `lionchat_meta_lead_refresh_sample` e depois `lionchat_meta_lead_show`. **Leia o mapeamento atual, junte o novo e mande o objeto inteiro de volta.**
6. **Mapeamento evento para alvo.** Formato geral: `{"<evento>": {"automation_id": N, "flow_id": M}}`. Confira o nome exato de cada evento em `references/catalogo-de-eventos.md`.
7. **Ativar.** No Webhook Universal a ativacao acontece sozinha quando o de-para e o campo de evento estao preenchidos; nas demais e um interruptor (`active`).

**Familia B, ordem propria:** conectar a credencial (Pixel: `lionchat_meta_pixel_integrations_create`; Analytics: `lionchat_ga4_integrations_create`; Google Ads: so depois da autorizacao no navegador, com `lionchat_google_ads_integrations_update`) -> `lionchat_funnels_meta_events_config` para dizer qual evento sai em cada etapa -> testar. Ver `references/conversao-para-anuncios.md`.

**Familias C, D e E:** ver `references/so-no-painel.md` (C precisa de autorizacao no navegador) e `references/ferramentas-mcp.md` (D e E).

Em cada chamada, mostre uma linha do que esta fazendo: `Criando a integracao "Hotmart"... ok`. Se der erro:

- **Recusa com `automation_id(s) invalidos` ou `flow_id(s) invalidos`** - o numero nao existe nesta conta. Liste de novo e corrija. Essa conferencia na hora de gravar existe nos 7 checkouts e no Webhook Universal; nas demais um numero errado e aceito e simplesmente nunca dispara.
- **Recusa por permissao** - quase toda integracao e so para administrador. Pare e diga qual acesso falta.
- **A integracao nao aparece na lista de apps** - nao insista. E liberacao da instalacao ou credencial que so o suporte liga.
- **Qualquer erro nao previsto** - pare, mostre o erro e pergunte se pula ou corrige. Nunca chute outra ferramenta.

### Etapa 5 - CONFERIR E RESUMIR

**Nada esta pronto enquanto ninguem provou.**

1. Peca um evento de VERDADE (uma compra de teste, um preenchimento do formulario, um agendamento).
2. Leia o historico de eventos da integracao **logo depois** - nos 7 checkouts, no Webhook Universal, na Omie e na Conta Azul ele mostra so os 50 mais recentes e nao tem paginacao. A e-Clinica e o Meta Lead Ads sao as excecoes: os dois paginam.
3. Evento com situacao "processado" nao quer dizer mensagem entregue. Confira o alvo: `lionchat_automation_rules_list_1` (historico de execucao da automacao, ultimas 48 horas) ou `lionchat_flows_executions_list`. E ali que aparece o motivo em texto, como "caixa de envio nao configurada" ou "contato sem telefone".
4. `lionchat_contacts_form_entries_list` na ficha da pessoa mostra exatamente o que a integracao gravou.

Entregue o resumo em tres blocos:

```
CRIADO NA SUA CONTA
  [integracao] - Configuracoes > Integracoes > [nome na tela]
  [automacao ou fluxo] "[nome]" - caixa "[nome]"
  Mapeamento: [evento] dispara [alvo]

VOCE PRECISA FAZER ISTO AGORA (senao nao chega nada)
  Cole este endereco em [onde exatamente]:
  [webhook_url]

NAO FEITO / SO NA MAO
  [item] - [motivo em uma frase]
```

## Regras que nao podem ser violadas

1. **NUNCA cria ou altera nada sem confirmacao explicita.** Se a resposta foi ambigua, pergunte de novo.
2. **NUNCA apaga nada** - nem integracao, nem automacao, nem fluxo, nem pagina conectada. Se a pessoa quer apagar, explique onde ela faz isso no painel.
3. **NUNCA inventa nome de ferramenta, de evento ou de campo.** Evento com nome errado e gravado sem erro e nunca dispara. As listas fechadas estao em `references/catalogo-de-eventos.md`.
4. **SEMPRE confira a ferramenta pelo CAMINHO da API**, nunca so pelo nome com numero no fim. `lionchat_ecommerce_webhooks_create` e o Webhook Universal, nao um checkout - ver `references/ferramentas-mcp.md`.
5. **SEMPRE leia antes de gravar mapeamento e mande o objeto inteiro de volta.** Mandar so a chave nova ja apagou configuracao de cliente.
6. **NUNCA crie a integracao antes do alvo.** Sem automacao ou fluxo mapeado, o evento chega, o contato e criado e nada acontece - e esse e o "configurei e nao funciona" numero um.
7. **NUNCA prometa que a caixa escolhida no cadastro da integracao vale.** O campo de caixa dos checkouts e gravado e nunca lido. A caixa vem da automacao ou do fluxo.
8. **NUNCA prometa filtro por condicao numa automacao de integracao** (evento `webhook`). A tela oferece; o motor nao executa. Quem filtra e o mapeamento (um evento por alvo) ou um fluxo, que tem bloco de condicao de verdade.
9. **NUNCA prometa "so a primeira compra" fora dos seis checkouts que a suportam** (Hotmart, Guru, Kiwify, Ticto, Monetizze, Eduzz). Nas demais a opcao e gravada e ignorada.
10. **NUNCA conecte nada que exija autorizacao no navegador.** Facebook, Google, Slack, Notion, Linear, Shopify e Conta Azul dependem de uma pessoa clicando. Entregue o link e o roteiro - ver `references/so-no-painel.md`.
11. **NUNCA reprocesse um evento de checkout sem antes conferir duplicata** com a conferencia previa (`retry_preflight`). Reprocessar cego reenvia boas-vindas para quem ja recebeu.
12. **SEMPRE em portugues do Brasil, sem emoji**, em nome de integracao, de automacao e de fluxo - eles aparecem para a equipe.
13. **NUNCA toque em cobranca, plano, fatura ou saldo da conta**, nem que a pessoa peca.

## Armadilhas (o que falha em silencio)

Catalogo completo, com o sinal de cada uma e como sair, em `references/armadilhas.md`. As que mais custam:

- **Se voce criar a integracao sem alvo mapeado**, o evento chega, aparece como recebido no historico, cria o contato e nada mais acontece. Nenhum erro em lugar nenhum.
- **Se a automacao alvo estiver sem caixa de entrada**, todo evento morre com "Automacao sem caixa de envio configurada" - e essa frase so aparece no historico de execucao da automacao, nao no historico da integracao.
- **Se voce montar o mapeamento da Greenn com os numeros que a tela mostra**, ele nunca casa: o sistema recebe os eventos com NOME (`sale_paid`, `contract_canceled`, `checkout_abandoned`). O webhook chega, e gravado como recebido, e nada dispara.
- **Se voce marcar eventos na lista do assistente achando que isso filtra**, nao filtra: essa lista e so memoria da tela. O unico portao real e o mapeamento evento para alvo.
- **Se voce inverter o sentido do mapeamento de campos**, tudo fica vazio sem erro e o lead morre com "sem telefone nem CPF nos campos mapeados". O sentido e `{caminho.no.recebido: campo daqui}`.
- **Se o fluxo alvo estiver com uma sessao presa naquela conversa** (esperando resposta sem tempo limite, ou pausado), todo evento novo daquele mesmo cliente e descartado. A segunda compra da mesma pessoa nao dispara nada.
- **Se o fluxo alvo for do tipo ferramenta de IA, estiver desligado ou sem caixa vinculada**, ele nao dispara e nao acusa nada.
- **Se voce usar o mesmo contato em dois checkouts diferentes** (Hotmart e Kiwify, por exemplo), a compra mais recente sobrescreve a anterior nos mesmos campos do contato: os 7 checkouts gravam nos MESMOS 19 campos, sem separacao por plataforma.
- **Se voce testar conversao arrastando o card para tras no funil**, nao dispara nada de proposito - so movimento para frente conta. E Pixel, Analytics e Google Ads disparam JUNTOS no mesmo movimento.
- **Se voce conferir a conexao do Google Analytics pela tela**, ela vai parecer certa mesmo com a credencial errada: o Google responde sucesso de qualquer jeito. So o botao de testar conexao revela.
- **Se o formulario do Meta Lead Ads estiver certo, mapeado, e o historico de eventos estiver VAZIO**, o problema nao e seu: provavelmente a captura de leads de anuncio nao esta liberada nesta instalacao. Peca ao suporte - a tela nao avisa.
- **Se voce puxar os leads antigos do Meta Lead Ads esperando que as boas-vindas saiam**, nao saem. Isso cria contato e so - de proposito, para nao mandar mensagem retroativa.
- **Se voce gravar chave inventada no mapeamento**, ela entra no banco, volta na resposta da API e nunca e lida. As unicas chaves lidas sao `automation_id`, `flow_id` e `first_purchase_only`.
- **Se o webhook de saida falhar 10 vezes seguidas**, ele fica 1 hora bloqueado e quase nao deixa registro nesse periodo. Passada a hora ele volta a tentar sozinho; trocar o endereco destrava na hora.

## O que eu faco e o que eu nao faco

> Eu ligo o LionChat aos sistemas que voce ja usa: leads dos seus anuncios do
> Facebook e Instagram, compras dos checkouts (Hotmart, Guru, Kiwify, Eduzz,
> Ticto, Monetizze, Greenn), seu ERP (Omie, Conta Azul), sua clinica
> (e-Clinica) e qualquer outro sistema pelo Webhook Universal. Tambem mando
> conversao para o Pixel da Meta, o Google Analytics e o Google Ads, aviso
> sistemas de fora quando algo acontece aqui, e monto a automacao ou o fluxo
> que fala com a pessoa quando a informacao chega.
>
> Eu NAO clico por voce nas telas do Facebook, do Google, do Slack, do Notion,
> do Linear, do Shopify e da Conta Azul: essas conexoes pedem uma autorizacao
> no navegador que so uma pessoa faz. Eu NAO consigo cadastrar nosso endereco
> dentro do painel do seu checkout, do seu ERP ou da sua clinica - eu entrego
> o endereco pronto e digo exatamente onde colar. Eu NAO configuro os
> lembretes automaticos da e-Clinica nem cadastro unidades dela: isso e tela.
> E eu NAO apago nada.
>
> Me conte de onde vem a informacao e o que tem que acontecer quando ela
> chegar. A partir disso eu proponho a montagem inteira e voce aprova antes de
> qualquer coisa ser criada.

## Referencias

Leia cada arquivo quando o momento chegar, nao antes:

- `references/ferramentas-mcp.md` - de-para entre nome da ferramenta e caminho da API, por causa dos sufixos numericos e do nome enganoso do Webhook Universal. **Leia antes de chamar qualquer ferramenta de integracao.**
- `references/catalogo-de-eventos.md` - a lista literal dos eventos de cada plataforma, com o nome exato que o sistema recebe. **Leia antes de escrever qualquer mapeamento.**
- `references/mapeamento-de-campos.md` - o sentido do de-para, os cinco destinos possiveis, o telefone obrigatorio, a juncao de dois campos num destino so e as chaves de conversa recusadas.
- `references/conversao-para-anuncios.md` - Pixel da Meta, Google Analytics 4 e Google Ads: ordem de montagem, eventos por etapa do funil e o que cada um resolve.
- `references/so-no-painel.md` - tudo que so uma pessoa consegue fazer, com o roteiro em linguagem de cliente.
- `references/receitas.md` - montagens prontas de ponta a ponta para os pedidos mais comuns.
- `references/diagnostico.md` - a arvore do "configurei e nao funciona", na ordem certa de conferencia.
- `references/armadilhas.md` - o catalogo completo das falhas silenciosas.
