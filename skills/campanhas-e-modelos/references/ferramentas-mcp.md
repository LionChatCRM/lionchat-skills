# Ferramentas do conector, campo a campo

Indice:

1. Como os nomes aparecem
2. Antes de criar: as ferramentas de apoio
3. Criar e ler campanha
4. Os campos de `lionchat_campaigns_create` por tipo
5. Acoes depois de receber (`bulk_actions`)
6. Acompanhar
7. Corrigir
8. Modelos de mensagem do WhatsApp
9. Fornecedor externo de SMS e voz
10. Erros comuns e o que dizer

---

## 1. Como os nomes aparecem

Os nomes abaixo sao os do conector do LionChat. Dependendo de como ele foi instalado, aparecem com
um prefixo antes do nome. Use SEMPRE o nome que aparecer na sua lista de ferramentas; nunca invente
nem adapte um nome. Se um nome deste arquivo nao existir na sua lista, pare e diga isso ao cliente
em vez de tentar outro parecido.

Nenhuma das ferramentas recebe o numero da conta: elas agem na conta ativa. Antes do primeiro
create, confirme em voz alta em qual conta voce vai mexer (`lionchat_account_show` traz o nome e o
fuso).

---

## 2. Antes de criar: as ferramentas de apoio

| Ferramenta | Para que |
|---|---|
| `lionchat_inboxes_list` | Listar as caixas e descobrir o tipo pelo campo de tipo de canal |
| `lionchat_inboxes_health` | Caixa oficial: veredito da Meta, nota de qualidade, faixa de limite e quanto ja foi usado nas ultimas 24 horas |
| `lionchat_inboxes_waha_status` | Caixa por QR Code: se a sessao esta conectada. `WORKING` = conectada |
| `lionchat_labels_list` | Identificadores de etiqueta, para o publico, a exclusao e as acoes pos-envio |
| `lionchat_funnels_list` | Identificador do funil e os identificadores curtos das etapas |
| `lionchat_custom_attributes_list` | Chaves dos atributos personalizados de contato e de conversa |
| `lionchat_agents_list` | Identificadores de atendente |
| `lionchat_teams_list` | Identificadores de time |
| `lionchat_account_variables_list` | Chaves das variaveis da conta |
| `lionchat_flows_list` | Com `with_campaign_trigger: true` e `inbox_id`, lista SO os fluxos elegiveis para Campanha de Fluxo |
| `lionchat_campaigns_estimate_audience` | Contar quantas pessoas recebem, e ver o teto do dia |

---

## 3. Criar e ler campanha

| Ferramenta | O que faz |
|---|---|
| `lionchat_campaigns_create` | Cria a campanha. O tipo vem da caixa; com fluxo informado vira Campanha de Fluxo |
| `lionchat_campaigns_list` | Lista todas as campanhas da conta |
| `lionchat_campaigns_show` | Le uma campanha inteira: publico, regras e configuracao de envio |
| `lionchat_campaigns_update` | Edita. So funciona em campanha que AINDA nao disparou |
| `lionchat_campaigns_destroy` | Apaga. DESTRUTIVO: leva junto a agenda do que ainda ia sair |
| `lionchat_inboxes_campaigns_list` | Lista as campanhas de UMA caixa |

O identificador usado por todas as ferramentas de campanha e o numero que aparece na tela, o mesmo
que a listagem devolve.

---

## 4. Os campos de `lionchat_campaigns_create` por tipo

### Campos comuns

| Campo | Obrigatorio | O que e |
|---|---|---|
| `title` | sim | Nome do disparo. Nao aparece para o cliente |
| `inbox_id` | sim | Por qual caixa sai. E ele que define o tipo |
| `message` | sim, menos em Campanha de Fluxo | O texto. Em caixa oficial e so um espelho do corpo do modelo |
| `audience` | nao valida, mas na pratica SIM | A lista de criterios. Sem ela a campanha e criada e nao manda para ninguem |
| `scheduled_at` | nao | Quando disparar, no formato completo COM fuso. Ausente = agora |
| `trigger_rules` | nao | Modo de combinacao, exclusao e modo do excedente |
| `template_params` | depende do tipo | Modelo e valores, ou o ritmo, mais as acoes pos-envio |
| `flow_id` | so em Campanha de Fluxo | Qual fluxo comeca para cada pessoa |
| `sender_id` | nao | Atendente remetente. No disparo em massa ele preenche a variavel de atendente no texto |
| `description` | nao | Anotacao interna |

Dois campos existem mas so valem na campanha do chat do site: o liga e desliga, e "so no horario
comercial". Em disparo em massa eles nao fazem nada.

### `trigger_rules`

```json
{ "audience_mode": "sum",
  "exclusion": [{ "type": "ContactLabel", "id": 44 }],
  "over_limit_mode": "batches" }
```

- `audience_mode`: `sum` (padrao) ou `all`. **Na criacao ele PRECISA ir aqui dentro** — mandado
  solto, e ignorado.
- `exclusion`: mesmo formato do publico.
- `over_limit_mode`: so em caixa oficial. Ausente ou qualquer outro valor = manda ate o limite e
  para. `batches` = divide o excedente em lotes diarios.
- Ha outras chaves ali dentro que o SISTEMA escreve sozinho (contagem congelada do publico, dados do
  limite, fila de lotes, relatorio da agenda). **Nunca escreva nessas chaves.** Ao editar, releia a
  campanha e mande de volta so o que voce quer mudar.

### `template_params` em caixa oficial

```json
{ "name": "aviso_black_friday", "language": "pt_BR", "category": "MARKETING",
  "processed_params": { "body": { "1": "{{contact.name}}", "2": "30%" } },
  "bulk_actions": { "contact_labels": [44] } }
```

- O modelo e encontrado pelo par NOME + IDIOMA, e so entre os aprovados.
- `processed_params.body` leva um valor por posicao. Posicao vazia vira um ponto na mensagem.
- `processed_params.header` so e preciso quando voce quer um arquivo diferente do padrao do modelo:
  `{ "media_url": "...", "media_type": "image", "media_name": "foto.png" }`, ou `{ "1": "texto" }`
  em cabecalho de texto.
- `processed_params.buttons` normalmente nao precisa: quando o modelo tem botao de link variavel, o
  apontamento do modelo ja resolve.

### `template_params` em caixa por QR Code

```json
{ "delay_min": 25, "delay_max": 60, "daily_cap": 300,
  "variations": ["texto alternativo 1", "texto alternativo 2"],
  "bulk_actions": { "labels": [7] } }
```

- `delay_min` e `delay_max` em SEGUNDOS. Padrao 20 e 40. O sistema sorteia um valor nessa faixa a
  cada envio.
- `daily_cap`: **chave AUSENTE assume 500 por dia.** Para mandar tudo no mesmo dia, mande a chave
  explicitamente vazia ou zero. Teto de 100.000 pessoas por campanha.
- `variations`: alternam com a mensagem principal por rodizio.

### `template_params` em Campanha de Fluxo

```json
{ "delay_min": 30, "delay_max": 90, "daily_cap": 200 }
```

- **Nao aceita `variations` nem `bulk_actions`** — quem faz isso sao os blocos do fluxo.
- `daily_cap` conta PESSOAS. Sem o campo, em caixa oficial herda o teto que a Meta libera; se nao
  der para ler, 500.
- Intervalo maximo aceito entre pessoas: 24 horas.

---

## 5. Acoes depois de receber (`bulk_actions`)

Vale em caixa oficial e em QR Code (nao na Campanha de Fluxo).

```json
{ "assignee_id": 3,
  "priority": "high",
  "labels": [7],
  "contact_labels": [44],
  "attribute_changes": [{ "scope": "contact", "key": "ultimo_disparo", "value": "black-friday" }] }
```

| Campo | O que faz |
|---|---|
| `assignee_id` | Poe um atendente como responsavel pela conversa |
| `priority` | `urgent`, `high`, `medium`, `low` ou vazio |
| `labels` | Etiquetas de CONVERSA |
| `contact_labels` | Etiquetas de CONTATO (as que ficam na ficha) |
| `attribute_changes` | Grava atributo. `scope` e `contact` ou `conversation` |

Quando isso e aplicado:

- Caixa oficial: so quando a Meta CONFIRMA a entrega. De segundos a minutos depois. Quem falhou nao
  recebe a marca, de proposito.
- Caixa por QR Code: ao fim de cada rodada de envio.

**Aviso obrigatorio ao cliente**: `assignee_id` muda quem responde aquela conversa. Isso e mudanca
de regra de negocio do time, entao diga em voz alta antes de criar.

---

## 6. Acompanhar

| Ferramenta | Para que | Serve para |
|---|---|---|
| `lionchat_campaigns_statistics` | Placar de entrega e motivos de falha | Campanha de mensagem |
| `lionchat_campaigns_flow_report` | Agendados, disparados, na fila, pulados com motivo, previsao de termino | Campanha de Fluxo |
| `lionchat_campaigns_dispatch_days` | A agenda dia a dia | Fluxo e QR Code com teto |
| `lionchat_campaigns_dispatch_day_entries` | QUEM sai num dia; e onde se acha o identificador de uma pessoa | Fluxo e QR Code com teto |
| `lionchat_inboxes_failed_messages_summary` | Saude geral do numero, com as falhas separadas por motivo | Diagnostico da caixa |

---

## 7. Corrigir

| Ferramenta | O que faz | Destrutivo |
|---|---|---|
| `lionchat_campaigns_resend_failures` | Reenvia as falhas. So caixa oficial. MANDA MENSAGEM DE VERDADE | envia |
| `lionchat_campaigns_reschedule_batch` | Muda a hora do proximo lote diario | nao |
| `lionchat_campaigns_stop_batches` | Esvazia a fila de lotes diarios | SIM, sem desfazer |
| `lionchat_campaigns_reschedule_flow` | Empurra a fila do fluxo para outro horario | nao |
| `lionchat_campaigns_stop_flow` | Cancela quem ainda nao foi disparado no fluxo | SIM, sem desfazer |
| `lionchat_campaigns_skip_dispatch_day` | Pula um dia; as pessoas vao para o fim da fila | nao |
| `lionchat_campaigns_move_dispatch_day` | Move as pendentes de um dia para outro | nao |
| `lionchat_campaigns_shift_dispatch_from` | Empurra um dia e todos os seguintes | nao |
| `lionchat_campaigns_reprogram_dispatch` | Refaz a agenda so do que falta | nao |
| `lionchat_campaigns_remove_dispatch_entry` | Tira UMA pessoa da campanha | SIM |

Datas de agenda: ano-mes-dia. Data e hora de remarcar: formato completo COM fuso e no futuro.

---

## 8. Modelos de mensagem do WhatsApp

Todas exigem caixa WhatsApp Oficial e perfil de ADMINISTRADOR.

| Ferramenta | O que faz |
|---|---|
| `lionchat_inboxes_whatsapp_templates_list` | Lista os modelos. Filtros por categoria, situacao, idioma e busca. A lista vem no campo `data` |
| `lionchat_inboxes_whatsapp_templates_create` | Cria e manda para analise da Meta |
| `lionchat_inboxes_whatsapp_templates_create_2` | EDITA um modelo existente. O nome engana: e a edicao. Volta para a fila da Meta |
| `lionchat_inboxes_whatsapp_templates_update` | Aponta de onde vem cada variavel do CORPO. NAO passa pela Meta |
| `lionchat_inboxes_whatsapp_templates_header_media` | Define, troca ou remove o arquivo padrao do cabecalho. NAO passa pela Meta |
| `lionchat_inboxes_whatsapp_templates_media_library` | Lista os arquivos ja anexados naquela caixa |
| `lionchat_inboxes_whatsapp_templates_destroy` | Exclui o modelo. Apaga TODAS as versoes de idioma e trava o nome por cerca de 30 dias |
| `lionchat_inboxes_sync_templates` | Puxa os modelos da Meta para a caixa. Use depois de criar ou editar |

Duas observacoes que evitam erro:

- **`lionchat_inboxes_whatsapp_templates_create_1` e um envio de arquivo.** Na pratica o conector nao
  envia arquivo: considere indisponivel e use a biblioteca de midia.
- Ao apontar variavel, o identificador do modelo e o numerico que vem na listagem, nao o nome.

---

## 9. Fornecedor externo de SMS e voz

| Ferramenta | O que faz |
|---|---|
| `lionchat_topsend_campaigns_create` | Cria campanha de SMS, SMS de tela, torpedo de voz ou URA |
| `lionchat_topsend_campaigns_list` | Lista as campanhas desse fornecedor, com estado e contadores |
| `lionchat_topsend_campaigns_show` | Le uma delas |
| `lionchat_topsend_campaigns_destroy` | Apaga |
| `lionchat_topsend_campaigns_test_connection` | Confere se a integracao esta funcionando |

Exige a integracao configurada e perfil de administrador. Nao tem edicao: so criar e apagar.

---

## 10. Erros comuns e o que dizer

| Resposta | O que costuma ser | O que dizer ao cliente |
|---|---|---|
| 403 | O usuario nao e administrador nem tem a permissao de gerenciar campanhas | "Seu usuario precisa ser administrador, ou receber a permissao de gerenciar campanhas" |
| 404 na campanha | O numero da campanha esta errado, ou ela foi apagada | Liste as campanhas e confirme o numero |
| 422 "tipo de caixa nao suportado" | A caixa nao aceita campanha (por exemplo e-mail ou Instagram) | Liste as caixas que aceitam |
| 422 na Campanha de Fluxo | O fluxo nao atende a alguma das regras; a mensagem diz qual | Corrija no desenho do fluxo, nao aqui |
| 422 "campanha ja concluida" | Ela ja disparou e nao aceita mais edicao | Use "Reusar configuracoes" para criar uma nova ja preenchida |
| 422 no reenvio de falhas | A campanha nao e de caixa oficial, ou nao ha falha reenviavel | Explique que reenvio so existe no WhatsApp Oficial |
| 422 ao mover um dia | O dia de destino nao cabe no teto | Ofereca empurrar aquele dia e os seguintes |
| 422 ao parar | Nao ha ninguem na fila | Mostre o relatorio: provavelmente ja terminou |
