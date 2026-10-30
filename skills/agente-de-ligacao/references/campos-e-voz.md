# Campos do agente de ligacao, motores e vozes

Ferramentas: `lionchat_captain_call_agents_list`, `_show`, `_create`, `_update`, `_duplicate`, `_destroy`.
O agente de ligacao NAO ocupa vaga de IA do plano.

| Campo | O que e | Limite / padrao |
|---|---|---|
| `name` | Nome interno | obrigatorio, ate 120 |
| `first_message` | A primeira frase que a voz fala quando a pessoa atende | ate 400. Aceita variaveis (`{{contact.name}}`) |
| `objective` | O ROTEIRO: o que fazer, o que perguntar, em que ordem, o que pedir no fim | ate 4000. Aceita variaveis |
| `style` | O jeito de falar (tom, ritmo, palavras proibidas) | ate 1200. NAO aceita variaveis |
| `voice_provider` | `openai` (padrao) ou `elevenlabs` | |
| `engine` | `gptlive` (padrao) ou `realtime`. So vale na OpenAI | |
| `voice_id` | A voz | ver abaixo |
| `elevenlabs_agent_id` | Id do agente criado na ElevenLabs | obrigatorio se o fornecedor for ElevenLabs |
| `max_duration_seconds` | Tempo maximo; no limite a ligacao cai sozinha | 30 a 600, padrao 180 |
| `captain_assistant_id` | O agente de TEXTO que executa os pedidos da ligacao | da mesma conta |

## Qual motor usar

| Fornecedor + motor | A voz conversa | A voz consulta agenda | A voz repassa pedidos ao agente de texto |
|---|---|---|---|
| OpenAI + `gptlive` (padrao, recomendado) | sim | sim | sim |
| OpenAI + `realtime` | sim | nao | nao |
| ElevenLabs | sim (o agente mora na ElevenLabs) | nao | nao |

Se a ligacao precisa FAZER alguma coisa (mandar mensagem, agendar, gravar dado, anotar), use OpenAI +
`gptlive` e preencha `captain_assistant_id`. Sem o agente de texto, a voz so conversa.

## Voz

- E PROPRIA do agente de ligacao. Nao mexe na voz do audio que o agente de texto manda no WhatsApp
  (`config.voice_id` do agente de texto continua sendo so do WhatsApp).
- No `gptlive`, liste com `lionchat_captain_voices_list` passando `engine=gptlive`. Aceita: alloy, ash, ballad,
  coral, echo, sage, shimmer, verse, **marin** e **cedar** (as duas mais naturais ao telefone). Voz fora da
  lista cai na marin.

## Variaveis no roteiro e na primeira frase

As mesmas do `{{` dos Cenarios (`{{contact.name}}`, atributos do contato e da conversa). Sao trocadas pelo
valor de quem vai atender ANTES de ligar. Numero que nunca conversou: a variavel sai vazia (nao quebra a
ligacao). Escreva a frase de um jeito que funcione com o nome vazio.

## Campos antigos

As chaves `config.call_*` do `captain_assistants_update` nao existem mais (desde 24/09/2026). Mandar elas nao
tem efeito. Tudo da ligacao fica no agente de ligacao.
