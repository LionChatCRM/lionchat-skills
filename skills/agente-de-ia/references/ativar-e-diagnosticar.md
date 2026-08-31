# Ligar a IA, testar e descobrir por que ela nao respondeu

O passo que quase todo mundo esquece, os efeitos colaterais de ligar, e a lista de motivos pelos quais uma
IA bem montada fica muda.

## Indice

1. Nao existe mais vincular agente a caixa de entrada
2. Os cinco caminhos para ligar
3. "Responder na hora"
4. O que MUDA na caixa quando a IA esta ativa
5. Como a conversa termina
6. Testar no Playground
7. Revisao de qualidade
8. Supervisor: comparar agentes
9. Diagnostico: por que ela nao respondeu
10. Botao de panico e desligar

---

## 1. Nao existe mais vincular agente a caixa de entrada

Isto e o mais importante do documento. **Nao ha mais nenhum lugar onde se conecta um agente a uma caixa de
entrada.** O vinculo e por CONVERSA: cada conversa guarda qual agente atende ela. Sem esse vinculo, a IA
nao responde, por melhor que esteja montada.

Documentacao antiga e material de treinamento ainda ensinam o passo "vincule o assistente a uma ou mais
caixas de entrada". Esse passo nao existe mais em tela nenhuma. Se o cliente perguntar por ele, explique
que hoje se liga por automacao, por fluxo ou na propria conversa.

---

## 2. Os cinco caminhos para ligar

| Caminho | Quando usar | Como |
|---|---|---|
| **Automacao** | atender toda conversa nova de uma caixa. E o caminho padrao | `lionchat_automation_rules_create` com evento de conversa criada, restrita a caixa, e a acao `assign_captain_assistant` |
| **Fluxo** | quando ha triagem antes: o fluxo faz as perguntas e entrega ao agente certo | bloco de acao "Ativar Agente IA" no editor de fluxos, com o id do agente |
| **Na conversa, pelo atendente** | caso a caso | a lista suspensa de Agente de IA na barra lateral da conversa |
| **Pelo conector** | quando voce esta operando uma conversa especifica | `lionchat_conversations_update` com `captain_assistant_id` |
| **Em massa** | ligar em conversas que ja estao abertas | `lionchat_kanban_bulk_bulk_actions` com `type: "Conversation"` e `fields: { captain_assistant_id: N }` |

Formato da acao na automacao:

```
action_name: "assign_captain_assistant"
action_params: [{ "assistant_id": 12, "proactive": true }]
```

Lembre da regra do conector para automacao: a ULTIMA condicao da regra precisa ter o conector de ligacao
nulo. Conector na ultima condicao salva com sucesso, aparece certo na tela e **a regra nunca dispara**.

**Cuidado com o caminho em massa:** ligar a IA em lote NAO arma o acompanhamento automatico e NAO faz a IA
responder na hora - diferente de ligar conversa a conversa. Se o cliente usar esse caminho, avise que as
conversas so vao ser atendidas quando o cliente escrever de novo.

---

## 3. "Responder na hora"

No momento em que a IA e ligada numa conversa, ela pode falar imediatamente (se apresentar, responder o que
esta pendente) ou ficar ativa esperando o cliente falar.

- Pelo conector: `captain_reply_now` (verdadeiro por padrao) junto do `captain_assistant_id`. So vale no
  momento da ATIVACAO.
- Na automacao e no fluxo: o campo `proactive` (verdadeiro por padrao).

**"Responder na hora" e acompanhamento automatico sao coisas diferentes e independentes.** Desligar um nao
desliga o outro. Ja houve um periodo em que ficaram amarrados por engano e conversas ficaram com IA ativa,
cliente parado e nenhuma cobranca.

Use "responder na hora" desligado quando o fluxo ja mandou uma mensagem e a IA so precisa assumir dali em
diante - senao ela fala duas vezes seguidas.

---

## 4. O que MUDA na caixa quando a IA esta ativa

**Avise o cliente antes de ligar.** Com um agente atribuido a conversa, aquela caixa para de enviar:

- a mensagem de saudacao;
- a mensagem de ausencia (fora do horario de atendimento);
- a coleta de e-mail do chat do site.

Isso e proposital: quem da as boas-vindas passa a ser a propria IA. Se o cliente tinha uma saudacao
caprichada, ela precisa virar a primeira fala da IA (pela instrucao, com "responder na hora" ligado) ou o
cliente vai jurar que voce quebrou alguma coisa.

---

## 5. Como a conversa termina

Tres coisas para alinhar com o cliente antes de ligar:

1. **A IA encerrando a conversa dispara a pesquisa de satisfacao daquela caixa**, se ela estiver
   configurada. Se o cliente nao quer isso, desligue `resolve_conversation` em `config.disabled_tools`.
2. **Descanso pos-fechamento de 3 minutos.** Logo depois de encerrar, um "obrigado!" do cliente NAO reabre
   a conversa nem acorda a IA. Fica registrado na linha do tempo. E de proposito, para nao reabrir por
   cortesia.
3. **A conversa NAO e encerrada sozinha por causa da IA.** O encerramento automatico que existe e o da
   conta inteira (Configuracoes, por tempo de inatividade) e vale para qualquer conversa, com ou sem IA.

---

## 6. Testar no Playground

`lionchat_captain_assistants_playground` com `message_content` (a mensagem do cliente de teste) e
`message_history` (lista de trocas anteriores).

A ficha do cliente ficticio fica salva no agente em `config.playground_contact` (nome, e-mail, telefone,
campos personalizados) - preencha antes, senao a IA testa sem contexto nenhum.

Rode pelo menos estes casos:

1. Cliente pergunta preco de algo que esta na base.
2. Cliente pergunta algo que NAO esta na base (para ver se ela consulta a equipe em vez de inventar).
3. Cliente quer agendar, se o agente agenda.
4. Cliente irritado, reclamando.
5. Cliente pedindo para falar com uma pessoa.

**O Playground e fiel no raciocinio, mas tres coisas divergem de proposito:**

- toda acao de escrita roda em ensaio - nada e criado, nenhuma requisicao externa sai, nenhuma mensagem
  chega a cliente nenhum;
- o que exige contato e conversa de verdade (anotacoes, card, compromissos, arquivos ja enviados) nao
  entra, porque o contato de teste nao existe no banco;
- as redes de seguranca do atendimento real nao sao reproduzidas.

Por isso: **testar so no Playground nao e testar.** Depois de ligar, acompanhe as primeiras conversas de
verdade.

Duas ferramentas para investigar conversa real: `lionchat_conversations_captain_reasoning` mostra o
raciocinio que a IA guardou naquela conversa (estrategia, objecoes, proxima acao) e ajuda a entender "por
que ela respondeu isso"; `lionchat_conversations_captain_history_reset` zera o que a IA enxerga daquela
conversa, para testar do zero sem apagar mensagem nenhuma.

---

## 7. Revisao de qualidade

`lionchat_captain_assistants_quality`. So leitura, nao gasta IA, sao conferencias mecanicas. Roda sempre
antes de ligar. Devolve os achados com o trecho exato do texto:

| Achado | O que quer dizer | Conserto |
|---|---|---|
| `unknown_variable` | ha `{{alguma.coisa}}` que ninguem preenche - sai em branco na conversa | trocar pela certa, ou criar o campo personalizado |
| `manual_scheduling` | o agente tem agenda ligada e o texto manda ele montar horario por conta propria | reescrever o trecho |
| `unknown_tool` | o texto cita uma ferramenta que este agente nao tem | corrigir a mencao |
| `long_instructions_no_scenarios` | instrucao acima de 2.500 letras e nenhum cenario | quebrar em cenarios |
| `no_knowledge` | nenhum documento e nenhuma pergunta frequente | montar a base |
| `no_guardrails` | protecoes vazias | escrever as fronteiras do negocio |
| `balloon_separator` | linha com tres tracos sozinha - a resposta chega picada ao cliente | tirar os tracos |

Consertos: `lionchat_captain_assistants_quality_apply` aplica o conserto mecanico;
`lionchat_captain_assistants_quality_rewrite` pede uma proposta de reescrita com IA (nao grava);
`lionchat_captain_assistants_quality_apply_proposal` grava a proposta aprovada;
`lionchat_captain_assistants_quality_undo` desfaz.

**Mostre a proposta ao cliente antes de aplicar** - ela reescreve o texto dele.

---

## 8. Supervisor: comparar agentes

`lionchat_captain_supervisor_show`. So leitura, zero chamada de IA. Compara TODAS as IAs da conta num
periodo: conversas atendidas, quantas o humano teve que assumir, engajamento, agendamentos, etiquetas,
fluxos acionados, satisfacao, cards ganhos e a taxa de conversao pela regua que voce escolher.

- `days`: de 1 a 180 (padrao 30).
- `assistant_ids`: quais estao em comparacao. **So muda os avisos do topo** - os numeros voltam de todas.
- A regua e "de cada X, quantas viraram Y": `conversion[base]` aceita `conversations` (padrao), `engaged`
  (o cliente respondeu), `booked` (chegou a agendar), `resolved` ou `cards`; `conversion[result]` precisa
  caber na base escolhida. Quando o resultado e etiqueta ou fluxo, `conversion[target]` diz qual.
  **Combinacao invalida cai no padrao SEM erro** - confira o que voltou.
- Dois avisos importantes: volume baixo (abaixo de 30 conversas num braco, a diferenca e sorte, nao
  resultado) e desequilibrio de caixa (uma IA pegou publico diferente da outra).

Para fazer um teste A/B de verdade, a divisao nao se faz aqui: use o bloco de divisao do editor de fluxos
(que divide de forma exata, nao por sorteio) seguido do bloco de ativar IA.

---

## 9. Diagnostico: por que ela nao respondeu

**Confira esta lista ANTES de mexer no texto.** Na maioria das vezes o problema nao e o prompt.

1. **A conversa nao tem agente atribuido.** Motivo numero 1. Confira na conversa.
2. **Caixa WhatsApp Oficial fora da janela de 24 horas.** A IA nem gera resposta, de proposito - se ela
   nao pode enviar, nao gasta chamada. Isso NAO acontece em caixa por QR Code, chat do site nem e-mail.
3. **Nao ha chave da OpenAI cadastrada na conta**, ou a chave estourou o limite. A conta aceita ate 3
   chaves reserva - e a solucao para "minha chave estourou".
4. **O agente esta pausado** (`paused: true`).
5. **Um atendente humano escreveu e o agente esta configurado para calar.** Se o tempo estiver em "Sempre",
   a IA foi desvinculada daquela conversa e nao volta sozinha.
6. **Alguem desligou a IA na conversa** (pela lista suspensa, pela acao em massa ou pela ferramenta de
   transferir para humano). Isso tira o agente daquela conversa; se houver automacao que atribui a IA e
   ela voltar a disparar naquela conversa, a IA e religada.
7. **Comentario de post do Instagram nunca aciona a IA** - so mensagem direta. Comentario e tarefa do
   humano.
8. **Descanso de 3 minutos depois de encerrar**, e so para mensagem curta de cortesia (secao 5).
9. **Mensagem cujo conteudo o canal nao entrega** e cortada antes de gastar IA.
10. **Conta suspensa** nao agenda resposta nenhuma.
11. **O modelo escolhido e da familia gpt-5.6.** Ela recusa ferramentas e a IA fica 100% muda.
12. **O modelo escolhido nao existe.** O campo do modelo nao e conferido pelo servidor: um nome com erro
    de digitacao grava normal e a IA fica muda. Confira contra a tabela de `references/modelo-e-ajustes.md`.

Se nada disso explicar, ai sim va para o texto: rode a revisao de qualidade e leia o raciocinio da conversa.

---

## 10. Botao de panico e desligar

- **Pausar tudo:** `lionchat_captain_assistants_update` com `paused: true` no topo (fora de `config`). A IA
  para de responder, de cobrar e de cumprir promessas agendadas em TODAS as conversas dela, sem perder
  nenhuma configuracao. Fica registrado quem pausou.
- **Desligar numa conversa:** `lionchat_conversations_update` com `captain_assistant_id` nulo. Tira o
  agente daquela conversa. **Nao e trava**: automacao que atribui a IA pode religar na proxima vez que
  disparar ali. Para parar de vez, pause o agente ou desligue a automacao.
- **Desligar um cenario, um arquivo ou uma ferramenta:** `enabled: false` no proprio item.
- **NUNCA apague o agente para "desligar".** Pausar resolve e nao perde nada. Exclusao e decisao do
  cliente, no painel.
