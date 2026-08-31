# Receitas prontas

Cada receita e a montagem inteira, na ordem. Continua valendo tudo do manual: entreviste, proponha
por escrito e so execute depois de "sim".

Em todas elas, o **passo do cliente** e obrigatorio - sem ele nao chega nada.

---

## 1. Compra aprovada no checkout dispara boas-vindas no WhatsApp

*Serve para Hotmart, Guru, Kiwify, Eduzz, Ticto, Monetizze e Greenn.*

1. Confirme que existe caixa de entrada de WhatsApp.
2. Crie a automacao: `lionchat_automation_rules_create` com `event_name: "webhook"`,
   `inbox_id` da caixa de WhatsApp, `active: true`, `conditions: []` e a acao de enviar mensagem em
   `actions`. **Sem `inbox_id` nada sai.** Guarde o numero dela.
   - Na mensagem, use os campos que o checkout ja grava sozinho no contato - por exemplo o nome do
     produto e o valor da compra. A lista esta em `mapeamento-de-campos.md`.
   - **Nao monte condicoes nessa automacao** - elas nao valem aqui.
3. Crie a integracao do checkout com o mapeamento:
   - Hotmart: `{"PURCHASE_APPROVED": {"automation_id": N}}`
   - Guru (webhook de tipo `transaction`): `{"approved": {"automation_id": N}}`
   - Kiwify: `{"compra_aprovada": {"automation_id": N}}`
   - Eduzz: `{"myeduzz.invoice_paid": {"automation_id": N}}`
   - Ticto: `{"authorized": {"automation_id": N}}`
   - Monetizze: `{"2": {"automation_id": N}}`
   - Greenn: `{"sale_paid": {"automation_id": N}}` - **nunca o numero**
4. Se a pessoa nao quiser mandar de novo em renovacao de assinatura, acrescente
   `"first_purchase_only": true` dentro do bloco. **So funciona em Hotmart, Guru, Kiwify, Ticto,
   Monetizze e Eduzz** - na Greenn a opcao e aceita e ignorada, entao la a renovacao dispara do
   mesmo jeito e isso precisa estar dito ao cliente.
5. **Passo do cliente:** colar o `webhook_url` da resposta no painel do checkout. Na Greenn, por
   produto. No Guru, um por tipo.
6. Provar: peca uma compra de teste, leia o historico logo depois, e confira o historico de execucao
   da automacao.

---

## 2. Carrinho abandonado dispara uma conversa de recuperacao

Aqui o alvo e FLUXO, nao automacao: recuperacao pede pergunta, espera e caminhos.

1. O fluxo precisa: estar ativo, ser fluxo de conversa, ter a caixa de WhatsApp vinculada e ter o
   gatilho **"Webhook recebido"** no bloco Inicio. Guarde o numero dele.
2. No mapeamento do checkout, aponte o evento de carrinho abandonado para o fluxo:
   - Hotmart: `PURCHASE_OUT_OF_SHOPPING_CART`
   - Guru: `abandoned`
   - Kiwify: `carrinho_abandonado`
   - Eduzz: `sun.cart_abandonment`
   - Ticto: `abandoned_cart`
   - Monetizze: `7`
   - Greenn: `checkout_abandoned`
   Formato: `{"<evento>": {"flow_id": M}}`.
3. **Passo do cliente:** conferir no painel do checkout que o evento de carrinho abandonado esta
   marcado para ser enviado - varios vem desligados de fabrica.
4. Avise: se a pessoa comprar depois e o mesmo fluxo ainda estiver aberto na conversa dela, o evento
   novo e descartado. Prever um caminho de saida no fluxo evita isso.

---

## 3. Lead do anuncio do Facebook cai num fluxo de qualificacao

1. **Passo do cliente:** conectar a pagina pelo navegador (roteiro em `so-no-painel.md`) e marcar so
   as paginas que interessam.
2. `lionchat_meta_lead_list` para ver as paginas; `lionchat_meta_lead_sync_forms` para buscar os
   formularios daquela pagina.
3. Prepare o fluxo de qualificacao (ativo, de conversa, com caixa vinculada, gatilho "Webhook
   recebido"). Guarde o numero.
4. `lionchat_meta_lead_refresh_sample` e depois `lionchat_meta_lead_show` para ver o exemplo.
5. `lionchat_meta_lead_update` (dentro de `form`):
   - `field_mapping` com os caminhos no formato `lead.field_data.<nome da pergunta>`. **Telefone e
     obrigatorio.** Nome e sobrenome separados podem apontar os dois para `contact_name` - saem
     colados, sem espaco.
   - `event_automation_mapping`: `{"flow_id": M}` (ou `{"automation_id": N, "flow_id": M}`).
6. Ative o formulario (`lionchat_meta_lead_create_2`).
7. Provar: peca um preenchimento de teste no anuncio e leia `lionchat_meta_lead_list_1`.
   **Historico vazio com tudo certo = peca ao suporte para conferir a liberacao da instalacao.**
8. Se a pessoa quiser os leads antigos, use o "puxar leads antigos" - mas avise que ele **cria
   contato e nao dispara mensagem nenhuma**, de proposito.

---

## 4. Consulta marcada na clinica vira lembrete um dia antes

1. Confira a integracao com `lionchat_eclinica_integrations_list`: situacao, unidades e enderecos.
2. Prepare o alvo (automacao com caixa, ou fluxo com caixa e gatilho de webhook). Guarde o numero.
3. **Aqui a montagem final e TELA.** O mapeamento de evento e, principalmente, as reguas de lembrete
   (quantos dias antes, qual alvo, o filtro "so quando" por procedimento ou profissional, e a hora
   fixa) sao configuradas em Configuracoes, Integracoes, e-Clinica. Entregue o roteiro e o numero do
   alvo que voce criou.
4. **Passo do cliente:** cadastrar o endereco de CADA unidade no painel da e-Clinica. Uma filial sem
   o endereco dela simplesmente nao manda nada.
5. Provar: `lionchat_eclinica_reminder_history_list` mostra os lembretes disparados; ha reenvio de um
   deles se precisar.
6. Avise: dois lembretes que caiam no MESMO agendamento com o mesmo numero de dias de antecedencia
   disputam a mesma linha e o paciente recebe so um - mesmo que os filtros sejam diferentes.

---

## 5. Sistema proprio avisa o LionChat (Webhook Universal)

Para qualquer sistema sem integracao pronta: ERP caseiro, formulario de site, painel interno,
ferramenta de automacao de terceiros.

1. Prepare o alvo (automacao com caixa, ou fluxo com caixa e gatilho de webhook).
2. `lionchat_ecommerce_webhooks_create` com `name`. **Guarde o `webhook_url`.** Nao mande `flow_id`
   no cadastro - isso cria outra coisa (o webhook embutido do editor de fluxo).
3. **Passo do cliente:** fazer o sistema mandar um envio de teste para aquele endereco. Peca que
   mande so os campos que interessam - conteudo acima de 1 MB e recusado.
4. `lionchat_ecommerce_webhooks_show`: use a lista `source_fields` (caminhos ja prontos) para montar
   o de-para. **Telefone obrigatorio.**
5. Defina `event_field` - qual campo carrega o tipo do evento (por exemplo `status`) - e o mapeamento
   por valor:
   `{"aprovado": {"automation_id": N}, "cancelado": {"flow_id": M}}`.
   Se o sistema so manda um tipo de evento, use `{"__all__": {"automation_id": N}}`.
6. A integracao ativa sozinha assim que o de-para e o campo de evento estao preenchidos.
7. Dentro do fluxo, o conteudo recebido fica disponivel como variavel no formato `{{webhook.campo}}`.
8. Provar: novo envio de teste, historico logo depois, e a aba de preenchimentos da ficha do contato.

---

## 6. Cobranca do ERP vira aviso no WhatsApp

**Omie:** conecte com `lionchat_omie_integrations_create` (a chave e o segredo que a pessoa gera no
painel da Omie). A integracao nasce como pendente e a conferencia das credenciais roda em segundo
plano - releia com `lionchat_omie_integrations_show` antes de dizer que conectou. Ligue so as
categorias que interessam
(`enabled_categories`) e mapeie por categoria:
`{"recebimentos": {"pix_gerado": {"automation_id": N}, "vencimento_3d": {"automation_id": P}}}`.
**Passo do cliente:** cadastrar o `webhook_url` nos webhooks nativos da Omie. Nao ofereca "so a
primeira compra" aqui.

**Conta Azul:** crie a integracao (nasce pendente) e chame `..._connect` - **passo do cliente:** abrir
o link no navegador e autorizar, em ate 15 minutos. Nao ha endereco para cadastrar em lugar nenhum: o
LionChat consulta a Conta Azul de 5 em 5 minutos. Depois, mapeie a regua:
`{"vencimento_3d": {"automation_id": N}, "vencido_7d": {"automation_id": P}}`.

Nos dois, os dados da cobranca ficam disponiveis como campos do contato (prefixo `omie_` e `ca_`)
para usar na mensagem - inclusive linha digitavel do boleto e PIX copia e cola.

---

## 7. Cartao ganho avisa o Facebook e o Google

1. Conecte a credencial (ver `conversao-para-anuncios.md`). Google Ads exige autorizacao no
   navegador.
2. `lionchat_funnels_meta_events_config` no funil:
   ```
   { "won": { "enabled": true, "name": "Purchase", "value_strategy": "card", "currency": "BRL" } }
   ```
   E, se a pessoa quiser marcar tambem o comeco: uma etapa com `{"enabled": true, "name": "Lead"}`.
3. Teste com `lionchat_kanban_items_meta_capi_fire` num cartao real, sem precisar move-lo.
4. Confira no historico por funil o que foi mandado e o que a plataforma respondeu.
5. Avise: cartao que anda para tras nao dispara, e Pixel, Analytics e Google Ads disparam juntos.

---

## 8. Avisar um sistema de fora quando algo acontece aqui

Para ligar o LionChat a uma ferramenta de automacao ou a um sistema proprio.

1. `lionchat_webhooks_create` com `url` (unico por conta), `subscriptions` (so nomes da lista fechada
   - ver `ferramentas-mcp.md`) e, se quiser restringir, `inbox_id`.
2. Escolha poucos avisos. `message_created` numa conta movimentada e um volume enorme.
3. **Passo do cliente:** o endereco precisa responder rapido - ha um limite de poucos segundos.
4. Confira as entregas em `lionchat_webhooks_deliveries_list`.
5. Avise: 10 falhas seguidas bloqueiam o envio por 1 hora. Passada a hora ele volta a tentar
   sozinho; trocar o endereco destrava na hora.
