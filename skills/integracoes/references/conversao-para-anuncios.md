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
`is_standard`, `value_strategy`, `value_fixed`, `currency`.

- `enabled` - liga o disparo naquela etapa. Etapa ligada sem nome de evento e ignorada.
- `name` - o evento que sai. Os sete padrao da Meta oferecidos na tela: `Lead`, `Contact`,
  `CompleteRegistration`, `Schedule`, `InitiateCheckout`, `Subscribe`, `Purchase`. Nome
  personalizado tambem vale.
- `value_strategy` - de onde vem o valor em dinheiro: `none` (sem valor), `card` (usa o valor do
  cartao) ou `fixed` (usa `value_fixed`).
- `currency` - moeda no padrao de tres letras (`BRL`, `USD`). Ausente vale `BRL`.
- `won` e `lost` - o que dispara quando o cartao e marcado como Ganho ou Perdido.

**Este e o unico mapeamento que JUNTA com o que ja existe.** Mande so o que muda: etapa nao citada
fica como esta. A resposta devolve a configuracao inteira depois da juncao.

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

## O que precisa estar dito ao cliente antes

1. **Cartao que anda para TRAS no funil nao dispara nada.** De proposito. Quem testa arrastando o
   cartao de volta conclui que "nao funciona".
2. **Os tres disparam JUNTOS.** Quem liga so o Analytics achando que e "so medicao" passa a mandar
   tambem para o Pixel e para o Google Ads, se essas duas estiverem ligadas. Um movimento de cartao
   vira ate tres envios.
3. **Cartao criado JA numa etapa com evento ligado tambem dispara** - nao so o movimento.
4. **O cartao precisa ter contato resolvido.** Cartao sem pessoa vinculada nao gera conversao.
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
