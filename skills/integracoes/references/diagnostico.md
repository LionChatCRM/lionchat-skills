# "Configurei e nao acontece nada"

Siga na ordem. Cada no elimina uma causa. Pular um no faz voce consertar a coisa errada.

Antes de comecar, pergunte uma coisa so: **o evento ja aconteceu de verdade la fora depois que a
integracao foi ligada?** Compra antiga, lead antigo e consulta antiga nao disparam nada.

---

## No 0 - A integracao existe para esta conta?

`lionchat_integrations_apps_list`. Se aquela integracao **nao aparece na lista**, a conta nao pode
usa-la: e liberacao da instalacao ou credencial que so o suporte configura. Pare aqui, avise e nao
tente outro caminho.

---

## No 1 - O endereco foi cadastrado no sistema de fora?

Esta e a causa mais comum de "zero eventos no historico".

Pergunte, com estas palavras: "voce ja colou o endereco que eu te passei dentro do painel do
[sistema]?" Se a resposta for "acho que sim", peca um print ou peca para conferir.

Lembretes: o **Guru** precisa de um endereco por tipo; a **Greenn**, um por produto; a **e-Clinica**,
um por unidade. Quem **nao tem endereco nenhum**: Meta Lead Ads e Conta Azul - nesses dois, pule para
o No 2.

---

## No 2 - O evento chegou aqui?

Leia o historico de eventos da integracao (a ferramenta esta em `ferramentas-mcp.md`).

**Leia LOGO depois do teste:** nos 7 checkouts, no Webhook Universal, na Omie e na Conta Azul sao os
50 mais recentes, sem paginacao (a e-Clinica e o Meta Lead Ads paginam). Numa conta movimentada o
evento de teste some da vista em minutos e voce vai concluir "nao chegou" quando chegou.

- **Historico VAZIO** - nada chegou. Volte ao No 1. Se o endereco esta certo mesmo, confira do lado
  de la se o evento esta marcado para ser enviado, e se o token de conferencia bate.
- **Caso especial do Meta Lead Ads:** formulario ativo, mapeamento certo e historico vazio quase
  sempre significa que a captura de leads de anuncio nao esta ligada nesta instalacao. O lead e
  descartado antes de virar registro. Peca ao suporte.
- **Historico com evento** - siga.

> Nao aceite "a outra plataforma diz que entregou" como prova. O nosso endereco responde sucesso de
> proposito mesmo quando falha ao gravar, para a plataforma de fora nao ficar tentando de novo.

---

## No 3 - O evento foi processado, ou falhou?

Na mesma listagem, olhe a situacao de cada evento.

- **Falhou com "Lead sem telefone nem CPF nos campos mapeados"** (Webhook Universal e Meta Lead Ads)
  - o de-para de campos esta vazio ou invertido. Va para `mapeamento-de-campos.md`.
- **Falhou com "Falha ao criar contato: sem telefone, email nem documento"** (checkouts, Omie, Conta
  Azul, e-Clinica) - o evento chegou sem nenhum dado que identifique a pessoa. Confira no proprio
  conteudo recebido se o telefone veio.
- **Falhou com "regra de automacao nao encontrada"** - o numero da automacao no mapeamento aponta para
  algo que nao existe (ou e de outra conta). Liste as automacoes e corrija.
- **Processado** - o evento foi tratado. **Isso NAO quer dizer que a mensagem saiu.** Siga.

---

## No 4 - O evento estava mapeado?

Leia a integracao e olhe o mapeamento evento para alvo.

- **Mapeamento vazio** - achou. O evento chega, cria contato e para ali. Grave o alvo.
- **Mapeamento existe, mas nao tem a chave daquele evento** - o nome nao casa. Compare letra por
  letra com `catalogo-de-eventos.md`. Suspeitos de sempre: Greenn com numeros no lugar dos nomes,
  Eduzz com sublinhado no lugar do ponto.
- **Chave estranha no mapeamento** (algo que nao seja `automation_id`, `flow_id` ou
  `first_purchase_only`) - foi gravada e nunca e lida. Refaca.
- **`first_purchase_only` ligado e era uma renovacao** - foi pulado de proposito. Fora de Hotmart,
  Guru, Kiwify, Ticto, Monetizze e Eduzz essa opcao nao faz nada, entao ela nunca e a explicacao.

---

## No 5 - O alvo rodou?

**Se o alvo e automacao:** `lionchat_automation_rules_list_1` (historico de execucao, ultimas 48
horas). E aqui que o motivo aparece em texto:

| O que voce le | O que significa | O que fazer |
|---|---|---|
| `caixa_de_envio_nao_configurada` | a automacao nao tem caixa de entrada | grave `inbox_id` na automacao |
| `contato_sem_telefone` | caixa de WhatsApp e contato sem numero valido | confira o de-para do telefone; confirme o numero |
| `falha_ao_criar_conversa` | nao foi possivel abrir a conversa | confira a caixa e o contato |
| nenhuma execucao registrada | a automacao nao foi acionada | volte ao No 4; e confira se ela esta ativa |

**Se o alvo e fluxo:** `lionchat_flows_executions_list`. Confira, nesta ordem:

1. o fluxo esta ATIVO?
2. e fluxo de conversa (nao ferramenta de IA)?
3. tem caixa de entrada vinculada?
4. tem o gatilho "Webhook recebido" (`webhook_received`) no bloco Inicio?
5. **ja existe uma sessao aberta daquele fluxo naquela conversa?** Se sim, o evento novo foi
   descartado. Esta e a causa do "funciona para uns clientes e para outros nao". Sessao presa
   (esperando resposta sem tempo limite, ou pausada) bloqueia aquele cliente para sempre.

---

## No 6 - A automacao rodou mas nao filtrou como esperado

Se a automacao disparou em casos que nao deveria: **as condicoes nao valem nesse tipo de automacao**.
A tela oferece, o motor nao executa - ela roda em todo evento mapeado.

Saida: separe em automacoes diferentes, uma por evento, ou troque por um fluxo, que tem bloco de
condicao de verdade.

---

## No 7 - Tudo certo e o dado nao aparece na ficha

`lionchat_contacts_form_entries_list` na ficha da pessoa mostra, origem por origem, exatamente o que
foi gravado. Se o campo esta vazio ali:

- o caminho do de-para nao existe no que chegou (releia `source_fields`, nao invente o caminho);
- o tipo do campo nao aceitou o valor (texto num campo de numero, por exemplo);
- e um atributo de CONVERSA e nao havia automacao nem fluxo mapeado - sem alvo nao nasce conversa e o
  valor e descartado;
- o valor veio de dois campos apontando para o mesmo destino e saiu colado, sem espaco.

---

## No 8 - A conversao nao apareceu no Facebook, no Google Analytics ou no Google Ads

1. O cartao andou para FRENTE no funil? Para tras nao dispara nada.
2. A etapa esta ligada E com nome de evento escolhido?
3. A credencial esta ligada? O Pixel nasce desligado e so liga depois de um teste com sucesso.
4. O cartao tem contato vinculado?
5. Leia o historico por funil (`meta_capi_events`, `ga4_events`, `google_ads_conversions`): ele mostra
   o que foi mandado e o que a plataforma respondeu.
6. No Google Analytics, rode o teste de conexao. Ele responde sucesso mesmo com credencial errada -
   o teste e a unica prova.
7. No Google Ads, confira se o de-para para a acao de conversao esta preenchido. Sem ele, a conversao
   e pulada.

---

## No 9 - O aviso para o sistema de fora parou de chegar

Leia as entregas (`lionchat_webhooks_deliveries_list`): cada tentativa traz a situacao, o erro e o
tempo. O endereco tem poucos segundos para responder. Depois de 10 falhas seguidas o envio fica 1
hora bloqueado e quase nao deixa registro nesse periodo; passada a hora ele volta a tentar sozinho.
Trocar o endereco zera o bloqueio na hora.

---

## Como reprocessar com seguranca

Achou a causa, corrigiu, e quer aproveitar o evento que ja tinha chegado?

- **Nos 7 checkouts:** chame primeiro a conferencia previa (`retry_preflight`). Se ela responder que
  ja existe um evento igual processado, **nao reprocesse** - a pessoa receberia a mensagem duas
  vezes. Confirme com o cliente antes.
- **No Webhook Universal, Omie, Conta Azul e e-Clinica:** nao ha conferencia. O reprocessamento e
  cego e pode duplicar mensagem. Pergunte antes.
- **Sempre um evento por vez.** Nao existe reprocessamento em lote.
