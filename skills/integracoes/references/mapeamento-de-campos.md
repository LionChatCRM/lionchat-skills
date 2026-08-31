# Mapeamento de campos: onde cada informacao vai parar

Duas integracoes exigem que voce diga de onde sai cada informacao: o **Webhook Universal** e o
**Meta Lead Ads**. Os checkouts, a Omie, a Conta Azul e a e-Clinica ja mapeiam sozinhos - neles nao
existe de-para para montar.

---

## 1. O sentido do de-para

E `{caminho do que chegou: campo daqui}`. Nunca o contrario.

```
field_mapping: {
  "cliente.telefone": "contact_phone",
  "cliente.nome":     "contact_name",
  "pedido.numero":    "contact_attr_numero_pedido"
}
```

Invertido, tudo fica vazio e nao aparece erro nenhum: o lead entra sem telefone e o evento morre com
"Lead sem telefone nem CPF nos campos mapeados".

**O caminho usa ponto** para descer nos niveis (`cliente.endereco.cidade`) e aceita posicao de lista
(`itens.0.nome`). Isso e mapeamento POSICIONAL: `itens.0` e sempre o primeiro item.

**No Meta Lead Ads o caminho tem prefixo fixo:** as respostas do formulario do anuncio vivem em
`lead.field_data.<nome da pergunta>`. Exemplo: `"lead.field_data.telefone": "contact_phone"`. O
sistema tenta adivinhar sozinho o de-para de nome, e-mail, telefone, empresa e cidade quando o
formulario e ativado pela primeira vez - confira e ajuste.

---

## 2. Nao mapeie no escuro

**Webhook Universal:** peca um envio de teste real e leia `lionchat_ecommerce_webhooks_show`. A
resposta traz:

- `sample_payload` - o **ultimo** conteudo recebido de verdade (o exemplo do cadastro so aparece se
  ainda nao chegou nada);
- `source_fields` - a lista ja pronta de `{path, value}`, achatada. **Use esta lista.** Nao invente o
  caminho olhando o conteudo cru.

**Meta Lead Ads:** `lionchat_meta_lead_refresh_sample` atualiza o exemplo (ele busca um lead real, e
se nao houver, monta a partir das perguntas do formulario), e `lionchat_meta_lead_show` devolve o
exemplo e o mapeamento atual.

---

## 3. Os cinco destinos

| Destino | O que faz |
|---|---|
| `contact_name`, `contact_email`, `contact_phone`, `contact_company`, `contact_city` | Campos fixos da ficha da pessoa |
| `contact_attr_<chave>` | Campo personalizado do CONTATO (dado que e da pessoa e vale sempre) |
| `conversation_attr_<chave>` | Campo personalizado da CONVERSA (dado daquele atendimento) |
| `cadastral_<campo>` | Campo cadastral nativo. Sao 13: `cpf`, `rg`, `cnpj`, `passport`, `date_of_birth`, `profession`, `address_cep`, `address_street`, `address_number`, `address_complement`, `address_neighborhood`, `address_city`, `address_state` |
| `social_<rede>` | Rede social na ficha. Sao 7: `instagram`, `facebook`, `linkedin`, `twitter`, `telegram`, `tiktok`, `github` |

Regras que valem para os cinco:

- **Telefone e obrigatorio de fato.** Sem `contact_phone` (ou `cadastral_cpf`), o lead e descartado.
  E se a caixa da automacao for de WhatsApp, contato sem telefone valido tambem para ali.
- **A chave do campo personalizado precisa existir na conta.** Crie antes com
  `lionchat_custom_attributes_create` e use a chave exata.
- **O tipo do campo importa.** Texto num campo de numero, ou texto solto num campo de data, e
  recusado na hora de gravar.
- **`conversation_attr_` so vale se houver automacao ou fluxo mapeado.** Sem alvo nao nasce conversa,
  e o valor e descartado em silencio. No "puxar leads antigos" do Meta Lead Ads isso sempre acontece.

---

## 4. Dois campos no mesmo destino sao JUNTADOS

E o caso mais comum do mundo real: o formulario manda nome e sobrenome separados, ou DDD e numero em
campos distintos.

```
"lead.field_data.nome":      "contact_name",
"lead.field_data.sobrenome": "contact_name"
```

Os dois valores sao **colados um no outro, sem separador**: "Maria" + "Silva" vira "MariaSilva".
Quem quiser o espaco precisa que ele venha de la (por exemplo, um campo intermediario que ja tenha o
espaco). Diga isso a pessoa antes de montar, e nao afirme que "so da para escolher um campo" - da
para juntar quantos quiser.

---

## 5. Chaves de conversa recusadas

Estas 14 chaves sao de sistema e o mapeamento para `conversation_attr_<chave>` e RECUSADO com elas:

`imported_from`, `import_digits`, `mail_subject`, `type`, `in_reply_to`, `initiated_by`,
`auto_reply`, `original_channel_type`, `conversation_language`, `agent_reply_time_window`,
`conference_sid`, `call_status`, `tiktok_capabilities`, `captain_media_asset_id`.

Alem dessas, toda chave que comeca com `captain_` (o caderno da IA) tambem e recusada.

A pior e `imported_from`: escrita, ela tira a conversa das telas. Hoje a recusa protege, mas o
sintoma para o cliente e "o atributo nao salvou" sem explicacao. Escolha outra chave.

---

## 6. O que cada integracao grava SOZINHA na ficha

Voce nao precisa mapear nada disso - ja vem pronto e pode ser usado em mensagem como campo
personalizado do contato.

**Os 7 checkouts gravam nos MESMOS 19 campos, sem separacao por plataforma:**

`product_name`, `offer_name`, `invoice_url`, `purchase_price`, `payment_method`, `purchase_email`,
`pix_code`, `pix_qr_code_url`, `pix_expiration_date`, `ticket_hash_url`, `event_name`, `event_date`,
`charged_times`, `subscription_cycle`, `subscription_status`, `customer_document`, `transaction_id`,
`payment_installments`, `payment_source`.

**Consequencia:** quem usa dois checkouts (Hotmart e Kiwify, por exemplo) tem a compra mais recente
sobrescrevendo a anterior no mesmo contato. Uma mensagem que cita o nome do produto pode citar o
produto da outra plataforma. So `payment_source` diz de onde veio.

**As outras tem prefixo proprio e nao se atropelam:**

| Integracao | Prefixo | Quantidade |
|---|---|---|
| Omie ERP | `omie_` | 35 campos (valor da cobranca, vencimento, linha digitavel do boleto, PIX copia e cola, numero do pedido, da OS, do contrato, links da nota fiscal...) |
| Conta Azul | `ca_` | 31 campos (valor total, valor pago, valor a pagar, situacao, vencimento, categoria, centro de custo...) |
| e-Clinica | `eclinica_` | dados da consulta, do profissional, do convenio, do laboratorio e do plano de tratamento |
| Meta Lead Ads | `meta_lead_` | 13 campos: `ad_id`, `ad_name`, `adset_id`, `adset_name`, `campaign_id`, `campaign_name`, `account_id`, `account_name`, `creative_id`, `creative_name`, `form_id`, `form_name`, `platform` |

Para escrever a mensagem sem chutar nome, liste os campos que existem de verdade na conta com
`lionchat_custom_attributes_list` e use a chave exata.

---

## 7. Ao ATUALIZAR um mapeamento

**Leia, junte e mande o objeto inteiro de volta.** Mandar so a chave nova ja apagou configuracao de
cliente: o que voce nao repetir some. Vale para `field_mapping` e para o mapeamento de evento.

Uma excecao, e so ela: `lionchat_funnels_meta_events_config` (eventos de conversao por etapa do
funil) JUNTA com o que ja existe. Ali mande so o que muda; etapa nao citada fica como esta.
