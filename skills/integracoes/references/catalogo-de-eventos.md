# Catalogo de eventos, por plataforma

O nome do evento e a CHAVE do mapeamento. Evento com nome errado e gravado sem erro nenhum e nunca
dispara: o webhook chega, aparece como recebido no historico, e a mensagem nao sai. Copie o nome
daqui, letra por letra.

Formato do mapeamento (vale para os checkouts, o Webhook Universal e o Meta Lead Ads):

```
{
  "<nome do evento>": { "automation_id": 12, "flow_id": 3, "first_purchase_only": true }
}
```

- `automation_id` e `flow_id` podem coexistir - os dois disparam.
- Os dois numeros precisam ser da propria conta. Nos 7 checkouts e no Webhook Universal isso e
  conferido na hora de gravar (o cadastro e recusado com `automation_id(s) invalidos` ou
  `flow_id(s) invalidos`); nas demais o numero errado e aceito e simplesmente nunca dispara.
- O casamento nao diferencia maiuscula de minuscula.
- **Chave inventada e aceita, volta na resposta da API e nunca e lida.** As unicas lidas sao
  `automation_id`, `flow_id` e `first_purchase_only`.
- **`first_purchase_only` so tem efeito onde o evento traz o numero da cobranca:** Hotmart, Guru,
  Kiwify, Ticto, Monetizze e Eduzz. Na Greenn, na Omie, na Conta Azul e no Webhook Universal a chave
  e gravada, volta na resposta e nao faz nada.
- Formato antigo `{"<evento>": 12}` ainda vale e significa `automation_id: 12`.

---

## Hotmart (15 eventos)

| Evento | O que e |
|---|---|
| `PURCHASE_APPROVED` | Compra aprovada |
| `PURCHASE_COMPLETE` | Compra completa |
| `PURCHASE_BILLET_PRINTED` | Boleto ou PIX gerado |
| `PURCHASE_CANCELED` | Compra cancelada |
| `PURCHASE_REFUNDED` | Reembolso |
| `PURCHASE_CHARGEBACK` | Estorno |
| `PURCHASE_EXPIRED` | Compra expirada |
| `PURCHASE_DELAYED` | Pagamento atrasado |
| `PURCHASE_PROTEST` | Contestacao |
| `PURCHASE_OUT_OF_SHOPPING_CART` | Carrinho abandonado |
| `SUBSCRIPTION_CANCELLATION` | Assinatura cancelada |
| `SWITCH_PLAN` | Troca de plano |
| `UPDATE_SUBSCRIPTION_CHARGE_DATE` | Data de renovacao alterada |
| `CLUB_FIRST_ACCESS` | Primeiro acesso a area de membros |
| `CLUB_MODULE_COMPLETED` | Modulo concluido |

## Guru (um webhook por TIPO)

O Guru exige `webhook_type` na criacao, e cada tipo gera um endereco proprio para cadastrar la.

**`transaction` (13):** `approved`, `waiting_payment`, `abandoned`, `refused`, `canceled`,
`expired`, `refunded`, `chargeback`, `pix_generated`, `boleto_printed`, `in_analysis`, `recovery`,
`trial`.

**`subscription` (6):** `active`, `trial`, `past_due`, `unpaid`, `canceled`, `expired`.

**`eticket` (4):** `invited`, `confirmed`, `checked_in`, `canceled`.

Cliente que so criou o de transacao e reclama que evento de assinatura nao chega: falta o segundo
webhook.

## Kiwify (10)

`compra_aprovada`, `compra_recusada`, `compra_reembolsada`, `chargeback`, `boleto_gerado`,
`pix_gerado`, `carrinho_abandonado`, `subscription_renewed`, `subscription_canceled`,
`subscription_late`.

## Eduzz (14) - **o nome tem PONTO**

`myeduzz.invoice_paid`, `myeduzz.invoice_canceled`, `myeduzz.invoice_refunded`,
`myeduzz.invoice_waiting_refund`, `myeduzz.invoice_refused`, `myeduzz.invoice_expired`,
`myeduzz.invoice_overdue`, `myeduzz.invoice_opened`, `myeduzz.invoice_waiting_payment`,
`myeduzz.invoice_scheduled`, `sun.cart_abandonment`, `myeduzz.contract_created`,
`myeduzz.contract_updated`, `myeduzz.contract_card_attempted`.

Escrever com sublinhado no lugar do ponto (`myeduzz_invoice_paid`) nao casa.

## Ticto (21)

`authorized`, `refused`, `refunded`, `chargeback`, `claimed`, `close`, `waiting_payment`,
`bank_slip_created`, `bank_slip_delayed`, `pix_created`, `pix_expired`, `abandoned_cart`, `trial`,
`trial_started`, `trial_ended`, `subscription_canceled`, `subscription_delayed`, `all_charges_paid`,
`card_exchanged`, `extended`, `uncanceled`.

## Monetizze (11) - **por CODIGO numerico**

| Codigo | O que e |
|---|---|
| `1` | Aguardando pagamento |
| `2` | Compra aprovada |
| `3` | Compra cancelada |
| `4` | Compra reembolsada |
| `5` | Compra bloqueada |
| `6` | Compra completa |
| `7` | Carrinho abandonado |
| `101` | Assinatura ativa |
| `102` | Assinatura inadimplente |
| `103` | Assinatura cancelada |
| `104` | Assinatura aguardando pagamento |

Aqui o numero e o nome de verdade - a Monetizze envia o codigo.

## Greenn (11) - **ATENCAO: por NOME, nunca por numero**

A tela da Greenn oferece codigos numericos (1, 2, 3, 101...), copiados da Monetizze. **Isso e um
defeito conhecido.** O sistema recebe os eventos da Greenn com NOME, entao um mapeamento montado com
numeros nunca casa: o webhook chega, e gravado como recebido, e nada dispara - sem erro nenhum.

Monte sempre com estes nomes:

| Nome | O que e |
|---|---|
| `sale_paid` | Compra aprovada |
| `sale_refused` | Compra recusada |
| `sale_refunded` | Compra reembolsada |
| `sale_chargedback` | Estorno |
| `sale_waiting_payment` | Aguardando pagamento |
| `checkout_abandoned` | Carrinho abandonado |
| `contract_paid` | Assinatura paga |
| `contract_trialing` | Periodo de teste |
| `contract_pending_payment` | Assinatura aguardando pagamento |
| `contract_unpaid` | Assinatura nao paga |
| `contract_canceled` | Assinatura cancelada |

Na Greenn o endereco tambem e cadastrado POR PRODUTO, nao uma vez so. E **"so a primeira compra" nao
vale aqui**: a chave e aceita e ignorada.

---

## Omie ERP (29 eventos em 7 categorias)

O mapeamento e agrupado por categoria:
`{"recebimentos": {"pagamento_confirmado": {"automation_id": 12}}}`. Ligue so as categorias que
interessam em `enabled_categories`.

| Categoria | Eventos |
|---|---|
| `recebimentos` | `cobranca_criada`, `boleto_gerado`, `pix_gerado`, `vencimento_3d`, `vencimento_1d`, `pagamento_confirmado`, `cobranca_vencida`, `cobranca_cancelada` |
| `pagamentos` | `despesa_criada`, `despesa_paga`, `despesa_vencida`, `despesa_cancelada` |
| `vendas` | `pedido_criado`, `pedido_faturado`, `pedido_cancelado`, `pedido_etapa_alterada` |
| `os` | `os_criada`, `os_faturada`, `os_concluida`, `os_cancelada` |
| `contratos` | `contrato_criado`, `contrato_faturado`, `contrato_cancelado` |
| `clientes` | `cliente_cadastrado`, `fornecedor_cadastrado` |
| `fiscais` | `nfe_autorizada`, `nfe_cancelada`, `nfse_emitida`, `nfse_cancelada` |

`vencimento_3d` e `vencimento_1d` nao vem da Omie: sao calculados aqui, uma vez por dia.

**"So a primeira compra" nao vale na Omie.** A chave e aceita, aparece na resposta e nao muda nada.

## Conta Azul (14 eventos, regua de cobranca completa)

Mapeamento plano: `{"vencimento_3d": {"automation_id": 12}}`. **"So a primeira compra" tambem nao
vale aqui.**

| Grupo | Eventos |
|---|---|
| Cobranca | `cobranca_criada` |
| Antes de vencer | `vencimento_7d`, `vencimento_3d`, `vencimento_1d`, `vencimento_hoje` |
| Depois de vencer | `vencido`, `vencido_3d`, `vencido_7d`, `vencido_15d`, `vencido_30d` |
| Desfecho | `pagamento_confirmado`, `pagamento_parcial`, `cancelado`, `renegociado` |

Aqui nao ha webhook: o LionChat consulta a Conta Azul de 5 em 5 minutos.

## e-Clinica (15 eventos)

`cliente_novo`, `cliente_alteracao`, `agendamento_novo`, `agendamento_alterado`,
`agendamento_transferido`, `agendamento_desmarcado`, `agendamento_atendido`,
`agendamento_aguardando` (paciente chegou), `falta`, `pagamento`, `cliente_baixa_pagamento`,
`cliente_inclusao_pagamento`, `controle_laboratorio_novo`, `controle_laboratorio_alterado`,
`odontograma_aprovado`.

**O mapeamento da e-Clinica e as reguas de lembrete sao tela** - o conector so le. Ver
`so-no-painel.md`.

---

## Webhook Universal - o evento e voce quem define

Nao ha lista fechada. Voce diz, em `event_field`, qual campo do que chega carrega o tipo do evento
(por exemplo `status`, `data.evento`, `pedido.situacao` - caminho com ponto), e o mapeamento usa os
VALORES que aquele campo assume.

```
event_field: "status"
event_automation_mapping: {
  "aprovado":  { "automation_id": 12 },
  "cancelado": { "flow_id": 3 }
}
```

A chave especial `__all__` aceita qualquer evento e tem prioridade sobre as outras:

```
event_automation_mapping: { "__all__": { "flow_id": 3 } }
```

**"So a primeira compra" nao existe no Webhook Universal.**

Bonus util: o conteudo cru recebido fica disponivel dentro do fluxo como variavel, no formato
`{{webhook.campo}}` (respeitando a estrutura recebida). Conteudo muito grande nao vira variavel.

## Meta Lead Ads - um alvo so, sem nome de evento

Aqui nao ha tipo de evento: chegou lead, dispara. O mapeamento aceita as duas formas:

```
{ "automation_id": 12, "flow_id": 3 }
```
ou
```
{ "__all__": { "automation_id": 12, "flow_id": 3 } }
```

---

## Eventos de conversao (Pixel, Analytics e Google Ads)

Sao outra coisa: nao entram, SAEM. A lista fica em `conversao-para-anuncios.md`.
