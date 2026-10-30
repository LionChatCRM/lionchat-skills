---
name: agente-de-ligacao
description: Monta, revisa e corrige a Ligacao com IA do LionChat pelo conector - o agente de ligacao (roteiro, primeira frase, jeito de falar, voz, tempo maximo), o agente de texto que executa os pedidos feitos na ligacao, os numeros de teste, a ligacao de teste e o bloco "Ligar com IA" do fluxo. Use quando o usuario disser "quero que a IA ligue para o lead", "monta uma ligacao automatica", "a IA ligou e ficou muda", "o bloco Ligar com IA deu erro", "a ligacao nao sai", "quero uma pesquisa de satisfacao por telefone", "a IA de voz nao mandou a mensagem que prometeu", ou pedir agente de ligacao, voz da ligacao ou ligacao de teste. Sempre pergunta antes e so cria depois de confirmacao explicita.
---

# Ligacao com IA do LionChat

A Ligacao com IA faz uma voz de IA ligar para o contato pelo WhatsApp (linha de ligacao LionCalls), conversar
com ele e, quando o cliente pede alguma coisa, repassar o pedido para um agente de TEXTO executar (mandar
mensagem, agendar, gravar dado, anotar). Sao DUAS pecas diferentes, e confundir as duas e o erro mais comum:

| Peca | Onde fica | O que faz |
|---|---|---|
| **Agente de ligacao** (`captain_call_agents_*`) | Agente de IA > Agentes de ligacao | E a VOZ. Tem o roteiro, a primeira frase, o jeito de falar, a voz e o tempo maximo. E o UNICO texto que a voz le. |
| **Agente de texto** (`captain_assistants_*`) | Agente de IA > Agentes | E o EXECUTOR. A voz repassa os pedidos e ele faz com as ferramentas DELE. Ligado ao agente de ligacao pelo campo `captain_assistant_id`. |

Regras que nunca mudam:

- **A voz NAO le as instrucoes, os cenarios nem a base do agente de texto.** Tudo o que ela precisa saber vai
  nos campos do agente de ligacao (`objective`, `first_message`, `style`). Agente de ligacao com esses campos
  em branco liga sem saber o que dizer.
- **No bloco "Ligar com IA" vai o id do AGENTE DE LIGACAO**, que sai de `lionchat_captain_call_agents_list`.
  NUNCA o id de um agente de texto. O salvar recusa id que nao e agente de ligacao da conta.
- **Agente de ligacao que ja existe nao se recria.** Liste antes.
- Se `lionchat_captain_call_agents_list` nao estiver entre as suas ferramentas, PESQUISE pelo nome antes de
  dizer que nao existe.

## Fluxo obrigatorio (nao pule etapas)

### Etapa 1 - Conferir os pre-requisitos

Use `lionchat_account_show`, `lionchat_captain_call_agents_list` e `lionchat_captain_assistants_list`.

1. **Os dois recursos ligados na conta:** Ligacao com IA (`ai_calling`) e Ligacao pelo WhatsApp
   (`lioncalls_calling`). Sem eles, nada liga. Isso NAO se liga pelo conector: se faltar, PARE e avise o
   cliente.
2. **Chave do fornecedor da voz na conta:** OpenAI (padrao) ou ElevenLabs. Sem a chave a ligacao e recusada
   ("A conta nao tem chave da OpenAI configurada"). Nao se cadastra pelo conector.
3. **Uma caixa que faz ligacao:** vem em `meta.dial_inboxes` da lista de agentes de ligacao. Lista vazia =
   nenhuma caixa com a linha de ligacao configurada e conectada. Nao da para montar sem ela.
4. **Agentes de ligacao existentes:** se ja existe um, ajuste-o; nao crie outro.
5. **O agente de texto que vai executar os pedidos:** qual e, e se ele tem ligadas as ferramentas que os
   pedidos vao usar (`lionchat_captain_assistants_tools`). Ver `references/pedidos-ao-agente-de-texto.md`.

### Etapa 2 - Entender (1 ou 2 perguntas por vez)

1. Para que e a ligacao? (qualificar lead de anuncio, confirmar consulta, pesquisa de satisfacao, recuperar
   quem sumiu, avisar de algo)
2. O que a voz precisa PERGUNTAR, e em que ordem?
3. O que precisa ACONTECER depois (mandar mensagem, marcar horario, gravar respostas no contato, nota para a
   equipe, mover card)?
4. O que a voz NUNCA pode falar ou prometer?
5. Quanto tempo no maximo? (30 segundos a 10 minutos; padrao 3 minutos)
6. Quem dispara: um fluxo (bloco "Ligar com IA"), e com qual gatilho?

### Etapa 3 - Propor e confirmar

Mostre ao cliente, ANTES de gravar:
- os tres textos do agente de ligacao (primeira frase, roteiro, jeito de falar), a voz e o tempo maximo;
- o agente de texto que executa e as ferramentas que ele precisa ter ligadas;
- o desenho do fluxo (gatilho, condicoes, "Ligar com IA" e o que acontece em cada saida).
So grave depois de um "sim" explicito.

### Etapa 4 - Montar

1. Agente de ligacao: `lionchat_captain_call_agents_create` (ou `_update` no existente). Campos em
   `references/campos-e-voz.md`. Roteiro: `references/roteiro.md`.
2. Agente de texto: ligar as ferramentas que faltam (skill `agente-de-ia`).
3. Fluxo: bloco `ai_call` com `aiCallAgentId` = id do agente de ligacao e `aiCallInboxId` = id de
   `meta.dial_inboxes`. Ver `references/bloco-no-fluxo.md` e a skill `criar-fluxos`.

### Etapa 5 - Testar (sempre antes de ligar para cliente)

1. Numero de teste: `lionchat_captain_call_test_phones_list`; se nao houver, peca ao cliente o numero e
   cadastre com `_create`. A ligacao de teste so vai para numero cadastrado (o telefone nunca vai no pedido,
   so o id).
2. **Peca autorizacao antes:** `lionchat_captain_call_agents_test_call` LIGA DE VERDADE, cria contato e
   conversa, pode disparar automacoes de "conversa criada" e fica gravada. Limite de 10 por hora na conta.
3. Acompanhe com `lionchat_captain_call_agents_test_call_status` (tocando / atendeu / nao atendeu, e a
   conversa).
4. Confira com o cliente: a voz seguiu o roteiro? Os pedidos viraram mensagem, atributo e nota na conversa?
5. Teste o fluxo de verdade num contato de teste e confira a execucao (`lionchat_flows_executions_list` e
   `lionchat_flow_sessions_show`): o bloco `ai_call` mostra o motivo em texto quando da erro.

### Etapa 6 - Relatar

Diga o que foi criado, com os ids, o que foi testado e o que ficou para o cliente conferir. Nunca diga que
"esta funcionando" sem um teste de verdade.

## Referencias

- `references/campos-e-voz.md` - campos do agente de ligacao, motores e vozes.
- `references/roteiro.md` - como escrever o roteiro, com modelos prontos (triagem de lead, confirmacao de
  consulta, pesquisa de satisfacao).
- `references/pedidos-ao-agente-de-texto.md` - o que a voz faz sozinha, o que ela repassa e os limites.
- `references/bloco-no-fluxo.md` - o bloco "Ligar com IA", saidas, fila da linha e receitas.
- `references/diagnostico.md` - cada mensagem de erro e o que fazer.
