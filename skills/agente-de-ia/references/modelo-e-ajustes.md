# Modelo, criatividade e os ajustes de comportamento

Tudo que se ajusta no agente fora do texto: qual modelo de IA usar, o quanto ele pode ser criativo, tempo
de resposta, o que fazer quando um atendente humano assume, cobranca automatica de quem sumiu e resposta em
audio.

## Indice

1. Onde cada coisa mora (topo x config)
2. Escolher o modelo
3. Criatividade (temperature)
4. Tempo de resposta
5. Quando o humano assume
6. Acompanhamento automatico (cobrar quem sumiu)
7. Resposta em audio
8. Campos que NAO fazem nada
9. Como o update se comporta

---

## 1. Onde cada coisa mora

O agente tem campos no TOPO e campos dentro de `config`. Errar o lugar faz o valor ser ignorado.

**No topo:** `name`, `description`, `paused`, `guardrails`, `response_guidelines`.

**Dentro de `config`:** `model`, `temperature`, `instructions`, `min_response_time`, `max_response_time`,
`feature_faq`, `feature_memory`, `feature_pause_on_human_reply`, `pause_on_human_reply_minutes`,
`feature_follow_up`, `follow_up_steps`, `follow_up_skip_conditions`, `disabled_tools`, `media_asset_ids`,
`booking_event_type_ids`, `offer_ids`, `voice_reply_mode`, `voice_random_percent`, `voice_provider`,
`voice_id`, `voice_instructions`, `playground_contact`.

`paused: true` e o **botao de panico**: a IA para de responder, de cobrar e de cumprir promessas agendadas
em TODAS as conversas dela, sem perder nada da configuracao. Fica registrado quem pausou.

---

## 2. Escolher o modelo

`config.model`. **Desde 08/09 o servidor confere este campo**: trocar para um nome que nao esta na lista
volta 422 e nada e gravado. Sao 22 modelos aceitos, os da tabela abaixo. (Agente antigo que ja tinha gravado
um modelo fora da lista continua salvando os outros campos; so a TROCA para um nome invalido e recusada.)

Custo, inteligencia e velocidade sao aproximados. O preco real por milhao de tokens (entrada /
saida, em dolar) esta entre parenteses quando o sistema registra.

| Modelo | Custo | Inteligencia | Velocidade | Aceita criatividade |
|---|---|---|---|---|
| `gpt-4.1-nano` | baixo | baixa | alta | sim |
| `gpt-4o-mini` | baixo | media | alta | sim |
| `gpt-4.1-mini` | baixo | media | alta | sim |
| `gpt-5.4-nano` | baixo | baixa | alta | sim |
| `gpt-5.4-mini` | baixo | media | alta | sim |
| `gpt-4o` | medio | alta | media | sim |
| `gpt-4.1` | medio | alta | media | sim (padrao do sistema) |
| `gpt-5-mini` | medio | alta | media | NAO |
| `gpt-5.4` | medio | alta | media | sim |
| `o3-mini` | medio | alta | media | NAO |
| `o4-mini` | medio | alta | media | NAO |
| `gpt-5` | alto | alta | media | NAO |
| `gpt-5.2` | alto | alta | media | sim |
| `gpt-5.5` | alto | alta | media | NAO |
| `o1` | alto | alta | baixa | NAO |
| `o3` | alto | alta | baixa | NAO |
| `gpt-6-luna` | baixo (0,10 / 0,50) | media | alta | NAO |
| `gpt-5.6-luna` | baixo (0,20 / 1,20) | media | alta | NAO |
| `gpt-5.6-terra` | medio (2,00 / 12,00) | alta | media | NAO |
| `gpt-6-sol` | medio (2,00 / 10,00) | alta | media | NAO |
| `gpt-5.6-sol` | alto (4,00 / 20,00) | alta | media | NAO |
| `gpt-5.1` | - | - | - | sim. **Legado**: aceito para nao travar agente antigo, mas nao aparece em tela nenhuma. Nao escolha para agente novo |

**As familias `gpt-5.6` e `gpt-6` estao liberadas e funcionam com ferramentas.** Elas ficaram fora da tela
de 29/07 ao inicio de setembro porque recusavam ferramenta pelo caminho antigo da OpenAI; hoje o sistema fala
com elas por um caminho proprio, sem nada a configurar. Se voce leu em algum lugar "nunca use gpt-5.6, a IA
fica muda", isso e de antes do conserto e nao vale mais. O que continua valendo: nenhuma delas aceita o ajuste
de criatividade (secao 3).

### Tabela de decisao

| Tipo de atendimento | Modelo sugerido | Por que |
|---|---|---|
| Roteiro fixo, cobranca de dado, triagem simples | `gpt-4.1-mini` ou `gpt-5.4-mini` | rapido e barato; o trabalho esta no cenario, nao no modelo |
| Agendamento | `gpt-4.1` | precisa seguir passo a passo sem improvisar |
| Venda conversacional, contorno de objecao | `gpt-4.1` ou `gpt-5.4` | precisa entender nuance e sustentar conversa |
| Suporte tecnico com base grande | `gpt-4.1` | precisa ler bem o material antes de responder |
| Caso com muita regra de negocio junta | `gpt-5.2` | ganha em raciocinio, custa mais |
| Volume enorme e pergunta simples | `gpt-4.1-nano`, `gpt-5.4-nano` ou `gpt-6-luna` | so use com base de conhecimento boa |
| Custo baixo com mais raciocinio | `gpt-6-luna` | e o padrao de conta nova (ver abaixo). Nao aceita criatividade: o tom vai pelas diretrizes |

Se o agente nascer sem modelo escolhido, ele herda o modelo padrao cadastrado na integracao da OpenAI da
conta - e, se nao houver, cai no padrao do sistema (`gpt-4.1`).

**Padrao de conta nova (desde 22/10): `gpt-6-luna`.** E o modelo que a integracao da OpenAI ja traz marcado
quando alguem conecta a chave pela primeira vez. Conta ja conectada mantem o modelo que escolheu. A lista da
integracao da conta tem os mesmos modelos do agente, menos o `gpt-5.1`, e um a mais (`gpt-5.5-pro`) que vale
so como padrao da conta: no agente ele e recusado.

### Detalhe importante sobre a conta

O modelo escolhido na integracao da OpenAI da conta e usado por TODAS as funcoes auxiliares de IA do
painel (reescrever texto, resumir conversa, sugerir resposta, sugerir etiqueta). O `config.model` do agente
vale so para o atendimento dele. A conta tambem aceita ate 3 chaves reserva da OpenAI, com troca automatica
quando a principal falha - **e a resposta certa quando o cliente disser "minha chave estourou o limite"**.
Nada disso se configura pelo conector: e no painel, em Integracoes.

---

## 3. Criatividade

`config.temperature`, de 0.0 a 2.0. Perto de 0 = obediente e repetitivo. Perto de 1 = mais solto.

| Uso | Valor |
|---|---|
| Agendamento, cobranca de dado, roteiro fixo | 0.2 a 0.3 |
| Suporte e duvida tecnica | 0.3 a 0.5 |
| Atendimento geral | 0.5 |
| Venda conversacional | 0.7 |

**Nos modelos de raciocinio** (`o1`, `o3`, `o3-mini`, `o4-mini`, `gpt-5`, `gpt-5-mini`, `gpt-5.5`) e nas
familias `gpt-5.6` (luna, terra, sol) e `gpt-6` (luna, sol) este campo **nao faz nada**: eles so aceitam o valor padrao, entao o sistema simplesmente nao manda o ajuste.
O valor grava e a tela do painel nem mostra o controle. Nao ha erro - o efeito e que voce nao controla o
tom por ai. Se o cliente precisa desse controle, escolha um modelo da coluna "sim".

---

## 4. Tempo de resposta

`config.min_response_time` e `config.max_response_time`, em segundos (padrao 6 e 8). O servidor exige de
1 a 60 e recusa maximo menor que o minimo. E a espera antes de responder, para nao parecer robo instantaneo - e tambem a janela em que varias
mensagens seguidas do cliente sao juntadas numa resposta so. Cliente que digita em rajada agradece um
valor um pouco maior.

---

## 5. Quando o humano assume

- `config.feature_pause_on_human_reply` (verdadeiro/falso): quando um atendente de verdade escreve na
  conversa, pelo painel ou pelo celular, a IA se cala para nao atropelar.
- `config.pause_on_human_reply_minutes`: por quanto tempo. `0` (padrao) significa **para sempre naquela
  conversa** - a IA e desvinculada. `10`, `15`, `30` ou `60` significam silencio temporario: o vinculo
  continua e ela volta sozinha depois do prazo, quando o cliente escrever de novo. Valor fora dessa lista
  vira 0.

Pergunte ao cliente qual dos dois ele quer. Quase todo mundo espera o segundo comportamento e configura o
primeiro sem perceber.

---

## 6. Acompanhamento automatico

Cobrar quem sumiu depois da IA responder.

- `config.feature_follow_up`: liga.
- `config.follow_up_steps`: ate 3 etapas, cada uma `{after_minutes, prompt}`. O tempo conta a partir da
  mensagem ANTERIOR. Cada etapa precisa de pelo menos 5 minutos. O `prompt` de cada etapa tem teto de 2000
  letras: o que passar disso e CORTADO ao salvar, sem erro.
- **Desde 24/10 cada etapa pode USAR ferramenta**, nao so mandar mensagem. Marque a ferramenta no texto da
  etapa com a mesma sintaxe do cenario, `[Rotulo](tool://identificador)` (ex.: "se ainda nao respondeu, ponha
  a etiqueta [Etiquetar](tool://add_label_to_conversation) `sem-resposta` e avise que vamos encerrar"). So
  vale ferramenta que este agente tem de verdade (ligada e com o recurso da conta); marcada fora disso, a
  etapa roda como antes, so com mensagem, e a tela avisa. `schedule_self_callback` e `list_assistants` nunca
  entram no acompanhamento. Etapa sem nenhuma ferramenta marcada segue o caminho antigo (so mensagem).
- **A SOMA das etapas nao pode passar de 1380 minutos (23 horas).** Acima disso o salvamento e recusado, e
  isso nao e defeito: a janela de 24 horas do WhatsApp conta da mensagem DO CLIENTE, e a cobranca so comeca
  a contar depois da resposta da IA. Ja foi medido em producao: soma de 1430 disparou 24h01m depois do
  cliente, a Meta recusou e 48 acompanhamentos sumiram em 7 dias, em silencio.
- Cuidado: a tela mostra "de 24h" no contador, mas o teto real e 23 horas. E configuracao antiga acima do
  teto continua ativa e falhando - a validacao so morde quando alguem salva de novo.
- `config.follow_up_skip_conditions`: ate 3 condicoes para NAO cobrar (logica OU entre elas). Tipos:
  - `label` - `labels: []` com uma ou mais etiquetas da conversa. Operadores SO `present` (tem qualquer
    uma delas) ou `absent` (nao tem nenhuma).
  - `contact_attr` e `conversation_attr` - `attribute` com a chave e `value` com o esperado. Operadores:
    `equal`, `not_equal`, `contains`, `present`, `blank`, `gt`, `lt` (os dois ultimos comparam numero).
  - `time_window` - `start` e `end` de 0 a 23, no fuso de Sao Paulo. Janela que cruza a meia-noite vale;
    `start` igual a `end` e ignorado.

  Use para nao cobrar quem ja comprou, quem tem etiqueta de nao perturbar, e para nao cobrar de madrugada.
- Formato antigo de uma etapa so (`follow_up_time` e `follow_up_prompt`) ainda funciona, mas so quando
  `follow_up_steps` esta vazio. Prefira sempre `follow_up_steps`.

**"Responder na hora" e acompanhamento sao coisas DIFERENTES e nunca podem ser acopladas.** "Responder na
hora" pergunta *ela fala agora?*; acompanhamento pergunta *ela cobra depois se ninguem falar?*. Ja houve um
periodo em que os dois foram amarrados por engano e 140 conversas ficaram com IA ativa, cliente parado e
nenhuma cobranca - justamente no caso que mais precisava.

Sugestao de cadencia que funciona bem: 60 minutos, depois 240, depois 720 (soma 1020, dentro do teto).

---

## 7. Resposta em audio

- `config.voice_reply_mode`: `off` (padrao), `mirror` (so responde falando quando a ultima mensagem do
  cliente foi audio) ou `random` (sorteia). **Nao existe modo "sempre falando"** - foi decisao de produto.
- `config.voice_random_percent`: no modo sorteio, quantas de cada 100 respostas saem faladas (padrao 30).
- `config.voice_provider`: `openai` (usa a mesma chave da conta, vozes prontas) ou `elevenlabs` (exige
  chave propria cadastrada em Integracoes, e permite voz clonada).
- `config.voice_id`: com OpenAI aceita `alloy`, `ash`, `ballad`, `coral`, `echo`, `fable`, `nova`, `onyx`,
  `sage`, `shimmer`, `verse` (padrao `onyx`). Com ElevenLabs exige o identificador da voz - liste com
  `lionchat_captain_voices_list`.
- `config.voice_instructions`: como falar (sotaque, tom, ritmo). Nao e o que dizer.

Tocar a amostra da voz so da pela tela.

---

## 8. Campos que NAO fazem nada

Existem, sao aceitos, sao gravados e **nenhum codigo le**. Nao gaste tempo com eles e avise o cliente:

- `config.welcome_message` - nao existe mensagem de boas-vindas do agente. Quem da as boas-vindas e a
  propria IA, pela instrucao, quando ela fala primeiro ao ser ativada.
- `config.feature_citation` - nao muda nada no atendimento: o motor que responde o cliente hoje nao le
  esse campo.
- `config.activation_label` - jeito antigo de ligar a IA. Nao use.
- A tela "Configuracoes de IA" da conta, que deixa escolher um modelo por funcao (editor, agente,
  copiloto) e ligar busca na Central de Ajuda e transcricao: **os valores dela nao sao lidos por ninguem
  hoje**. Quem manda de verdade e o modelo da integracao da OpenAI da conta e o `config.model` do agente.

Cuidado com um rotulo enganoso: o campo `description` aparece na tela como "Descricao" com a observacao
"apenas para referencia interna". **Isso esta errado.** O texto vai INTEIRO para o comportamento da IA,
logo abaixo da identidade dela. Escreva o papel dele ali, nunca uma anotacao de bastidor tipo "agente da
loja X, criado em agosto".

---

## 9. Como o update se comporta

`lionchat_captain_assistants_update` faz **mistura parcial** do `config`: mandar so `disabled_tools`
preserva as outras chaves.

Mas com duas pegadinhas:

1. **Array dentro do `config` e substituido inteiro.** Mandar `media_asset_ids: [5]` apaga os outros ids
   que estavam la. Leia o valor atual antes de escrever.
2. **A mistura e rasa.** `playground_contact` (a ficha ficticia do simulador) tem que ir COMPLETO: um
   objeto parcial substitui a chave inteira.

E a regra geral desta area: **campo que o servidor nao aceita some SEM erro.** A resposta diz sucesso e o
valor nunca chegou. Ja aconteceu com lista aninhada mais de uma vez. Sempre releia com
`lionchat_captain_assistants_show` depois de gravar.
