# Conversao para plataforma de anuncio (Pixel da Meta, Google Analytics 4, Google Ads)

Esta familia e o contrario das outras: nada entra, a informacao SAI. Quando um cartao anda no funil,
o LionChat avisa a plataforma de anuncio que aquele lead avancou. Serve para duas coisas diferentes,
e vale explicar a diferenca ao cliente antes de montar:

| Ferramenta | Para que serve de verdade |
|---|---|
| **Pixel da Meta (Conversions API)** | Ensina o Facebook e o Instagram quem vale a pena. A entrega dos anuncios melhora. Funciona mesmo quando o navegador da pessoa bloqueia rastreamento, porque sai do nosso servidor. |
| **Google Analytics 4** | So MEDIR. Ver de onde vem quem realmente compra. Nao muda a entrega de anuncio nenhum. |
| **Google Ads (Conversoes Aprimoradas)** | O Google usa a conversao para ajustar os lances sozinho. E o de maior retorno para quem gasta serio no Google, e o de configuracao mais pesada. |

---

## A ordem e diferente das outras integracoes

1. **Conectar a credencial.**
2. **Dizer qual evento sai em cada ETAPA do funil.**
3. **Testar.**

Sem o passo 2 nao sai nada, por mais certa que a credencial esteja. E o erro mais comum aqui.

---

## Passo 1 - conectar

**Pixel da Meta** - `lionchat_meta_pixel_integrations_create` com `pixel_id` e `access_token` (a
pessoa pega os dois no Gerenciador de Eventos da Meta). **Nasce desligado** e liga sozinho depois de
um envio de teste com sucesso (`lionchat_meta_pixel_integrations_test_event`). O campo
`test_event_code` mostra o evento chegando ao vivo na Meta, mas a janela de teste fecha sozinha
rapido - ha uma acao para reabri-la.

**Google Analytics 4** - `lionchat_ga4_integrations_create` com `measurement_id` (formato `G-` e mais
letras e numeros, em Admin, Fluxos de dados) e `api_secret` (no mesmo lugar, em Segredos do
Measurement Protocol). Nasce ativa.

> **Cuidado com o Analytics:** ele responde sucesso mesmo com a credencial errada. A tela fica toda
> verde, o historico diz "enviado" e nada aparece no painel do Google. A UNICA forma de saber e
> `lionchat_ga4_integrations_test_connection`. Rode sempre.

**Google Ads** - **nao da para conectar pelo conector**: e autorizacao no navegador. Depois de
conectado: `lionchat_google_ads_integrations_list_customers` para escolher a conta de anuncio
(`customer_id`, 10 digitos sem hifen; `login_customer_id` quando a conta esta debaixo de uma
agencia), `lionchat_google_ads_integrations_list_conversion_actions` para ver as acoes de conversao
disponiveis, e `lionchat_google_ads_integrations_update` para gravar o de-para e ligar (`enabled`).
Sem o de-para (`conversion_action_map`) a conversao e simplesmente pulada.

Os tres aceitam `funnel_id`: amarra aquela credencial a UM funil (util para clinica ou agencia com
varios profissionais, cada um com o pixel dele). Vazio significa "o padrao da conta" - e so pode
haver um padrao.

---

## Passo 2 - qual evento sai em cada etapa

Ferramenta: `lionchat_funnels_meta_events_config` (PATCH em `/funnels/{id}/meta_events_config`).

```
meta_events_config: {
  "stages": {
    "<chave da etapa>": {
      "enabled": true,
      "name": "Lead",
      "value_strategy": "none"
    },
    "<outra etapa>": {
      "enabled": true,
      "name": "Schedule",
      "value_strategy": "card"
    }
  },
  "won":  { "enabled": true, "name": "Purchase", "value_strategy": "card", "currency": "BRL" },
  "lost": { "enabled": true, "name": "Lead_Lost", "value_strategy": "none" }
}
```

Campos aceitos em cada bloco - **qualquer outro e descartado em silencio**: `enabled`, `name`,
`is_standard`, `value_strategy`, `value_fixed`, `currency` e `messaging_name`.

- `enabled` - liga o disparo naquela etapa. Etapa ligada sem nome de evento e ignorada.
- `name` - o evento que sai. Os sete padrao da Meta oferecidos na tela: `Lead`, `Contact`,
  `CompleteRegistration`, `Schedule`, `InitiateCheckout`, `Subscribe`, `Purchase`. Nome
  personalizado tambem vale.
- `value_strategy` - de onde vem o valor em dinheiro: `none` (sem valor), `card` (usa o valor do
  cartao) ou `fixed` (usa `value_fixed`). O valor fixo da etapa so passou a ser lido em 17/09/2026.
- `currency` - moeda no padrao de tres letras (`BRL`, `USD`). Ausente vale `BRL`.
- `won` e `lost` - o que dispara quando o cartao e marcado como Ganho ou Perdido.
- `messaging_name` - o evento quando o lead veio de ANUNCIO DE WHATSAPP (so etapas e `won`; `lost` e
  sempre evento de site). A Meta so aceita 14 nomes ali: `LeadSubmitted`, `QualifiedLead`,
  `ViewContent`, `AddToCart`, `InitiateCheckout`, `Purchase`, `OrderCreated`, `OrderShipped`,
  `OrderDelivered`, `OrderCanceled`, `OrderReturned`, `CartAbandoned`, `RatingProvided`,
  `ReviewProvided`. Tres estados: chave AUSENTE = automatico (Lead e Contact viram `LeadSubmitted`,
  Purchase e InitiateCheckout ficam iguais; sem sugestao sai como site); `""` = nao mandar como
  WhatsApp (sai como site); um dos 14 = escolha. Valor fora disso e descartado e vira ausente.

**Purchase sem valor sai com R$ 1,00 na Meta** (17/09/2026): nada configurado, card sem valor ou
valor zero. A Meta recusa Purchase de anuncio de WhatsApp sem valor. O valor fixo configurado e lido
ANTES desse piso - quem configurou R$ 9.000 continua mandando R$ 9.000.

**Este e o unico mapeamento que JUNTA com o que ja existe.** Mande so o que muda: etapa nao citada
fica como esta. A resposta devolve a configuracao inteira depois da juncao. Quem grava: administrador
ou cargo com `marketing_integrations_manage` (403 senao).

**A chave da etapa e a chave, nao o nome visivel.** Pegue em `lionchat_funnels_show`.

Este mesmo mapa vale para os TRES ao mesmo tempo. O Analytics traduz os nomes: `Lead` vira
`generate_lead`, `Purchase` vira `purchase`, `Contact` vira `contact`, `CompleteRegistration` e
`Subscribe` viram `sign_up`, `Schedule` vira `schedule`. Nome personalizado passa igual.

---

## Passo 3 - provar

- `lionchat_kanban_items_meta_capi_fire`, `lionchat_kanban_items_ga4_mp_fire` e
  `lionchat_kanban_items_google_ads_fire` disparam a conversao a mao a partir de um cartao, aceitando
  valor e moeda. Serve para testar sem precisar mover cartao.
- O historico por funil mostra o que foi mandado e o que a plataforma respondeu:
  `lionchat_funnels_meta_capi_events_list` / `_show` / `_retry`, e os equivalentes de Analytics
  (`ga4_events`) e de Google Ads (`google_ads_conversions`). E o diagnostico de "nao apareceu no
  Facebook".

---

## Anuncio de WhatsApp na Meta: para onde o evento vai

Quando o lead veio de um anuncio que abre o WhatsApp (o clique fica guardado na conversa), o evento
sai como evento de MENSAGEM, que e o que a Meta credita na campanha. O destino depende da caixa:

- **Caixa WhatsApp Oficial:** vai para o conjunto de dados ligado a conta do WhatsApp Business
  (WABA), com o token da propria CAIXA. O vinculo e feito sozinho (caixa nova na criacao; caixa antiga
  por uma varredura automatica). Plano B: `lionchat_inboxes_capi_dataset_link` (escrita, so
  administrador, ate 10 por hora). Vale para todas as caixas da mesma WABA.
- **Caixa WhatsApp por QR Code** (desde 01/10/2026): nao tem WABA; a identidade e a Pagina do Facebook
  que rodou o anuncio e o destino e o proprio pixel da conta. Nenhuma conexao nova.
- **Plano B automatico:** se a Meta recusar o evento de mensagem, ou nao responder em nenhuma das
  tentativas, ele e reenviado como evento de SITE e o historico mostra o motivo em `fallback_reason`
  (por exemplo `lionchat:dataset_nao_configurado`, `lionchat:caixa_token_recusado`,
  `lionchat:meta_sem_resposta`). Nao e erro do cliente; a conversao foi entregue como site.
- **Recusa do token da CAIXA nunca desliga o pixel** e nunca pede reautorizacao da caixa.

---

## Conversao pelo fluxo (sem card)

O bloco de acao **"Enviar conversao"** de um fluxo manda o evento na hora, com ou sem card:
`destinations` (meta, ga4, google_ads), um evento por destino em `event_names`, valor opcional e,
para a Meta, `messaging_event_names` (mesmo tri-estado do `messaging_name`). O card usado e o que
iniciou o fluxo ou, sem ele, o card mais recente da conversa; sem card nenhum, o evento sai com o
clique do anuncio guardado na CONVERSA. Falha de conversao nunca para o fluxo. Montar o bloco e
assunto da skill `criar-fluxos`.

---

## O que precisa estar dito ao cliente antes

1. **A etapa dispara venha o cartao de onde vier** (para frente ou para tras, desde 17/09/2026) -
   mas UMA vez por cartao+evento+etapa. Voltar a uma etapa que ja mandou nao manda de novo, nos tres
   destinos. Quem testa arrastando o mesmo cartao de um lado para o outro conclui que "parou".
2. **Os tres disparam JUNTOS.** Quem liga so o Analytics achando que e "so medicao" passa a mandar
   tambem para o Pixel e para o Google Ads, se essas duas estiverem ligadas. Um movimento de cartao
   vira ate tres envios.
3. **Cartao criado JA numa etapa com evento ligado tambem dispara** - nao so o movimento.
4. **O cartao precisa ter contato resolvido.** Cartao sem pessoa vinculada nao gera conversao. Sem
   card no funil, o caminho e o bloco "Enviar conversao" do fluxo.
5. **Nada disso comeca sozinho.** Se a conta nao tiver a funcionalidade liberada, a integracao nem
   aparece na lista de apps - isso e liberacao da instalacao, nao erro do cliente.
6. **Google Ads tem exigencias do lado de fora** que levam dias: conta gerenciadora, token de
   desenvolvedor aprovado e verificacao do acesso no Google Cloud. A tela deixa conectar antes disso
   e o envio nao funciona.

---

## O que NAO da para fazer

- **Conectar o Google Ads** - autorizacao no navegador.
- **Ligar "evento Lead automatico ao criar cartao"** - o motor le essa opcao, mas nenhum caminho da
  API nem da tela permite grava-la. Nao prometa.
- **Criar evento de conversao dentro do Facebook, do Analytics ou do Google Ads** - isso e no painel
  de la.
