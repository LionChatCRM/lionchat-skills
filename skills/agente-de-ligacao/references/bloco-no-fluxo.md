# Bloco "Ligar com IA" (type ai_call) no fluxo

So em conta com Ligacao com IA e Ligacao pelo WhatsApp liberadas. A IA liga para o TELEFONE DO CONTATO da
conversa do fluxo.

```json
{ "id": "ligar", "type": "ai_call", "position": {"x": 600, "y": 0},
  "data": { "aiCallAgentId": 4, "aiCallInboxId": 82 } }
```

- `aiCallAgentId`: id de AGENTE DE LIGACAO (`lionchat_captain_call_agents_list`). Nunca o de agente de texto.
- `aiCallInboxId`: id de `meta.dial_inboxes` da mesma lista.
- `aiCallAgentName` / `aiCallInboxName`: opcionais, so para o resumo no desenho.
- O salvar recusa agente ou caixa escolhidos/trocados que nao existem na conta ("ai_call block uses a call
  agent that does not exist in this account" / "...an inbox without a calling line...").
- Proibido em fluxo de acoes, em fluxo de grupo e em ferramenta de IA (o salvar recusa).

## Saidas
- `answered`: a pessoa atendeu.
- `no_answer`: nao atendeu ou recusou.
- `error`: nao deu para ligar; o motivo vai em texto no historico da execucao (ver `diagnostico.md`).
- `answered`/`no_answer` sem fio = o fluxo termina ali; `error` sem fio = a execucao fica como "Falhou".
- O fluxo NAO recebe o que foi dito na ligacao. Quem age sobre o conteudo e a voz, pelos pedidos.

## Comportamento
- Liga UMA vez. Para insistir: outro `ai_call` depois de `no_answer`, com um bloco de espera no meio.
- NUNCA ligue a saida `error` direto em outro `ai_call`: em alguns erros nao da para saber se a ligacao saiu.
- Nao confere horario comercial: ponha uma condicao antes.
- A IA sempre deixa uma vaga da linha para a equipe. Linha ocupada: espera a vez, por ordem de chegada, por
  ate 2 horas; passou disso, sai por `error`. Linha que so aceita uma ligacao por vez = erro.
- O cartao da ligacao (gravacao e transcricao) vai para a conversa do contato na caixa da linha.

## Receitas
- **A IA de atendimento pede a ligacao:** o cenario manda por a etiqueta `ligar` quando o cliente pede
  telefone; fluxo de conversa com gatilho "etiqueta adicionada: ligar" -> acao remover etiqueta `ligar` (LOGO
  depois do inicio) -> `ai_call`.
- **Lead de anuncio pago:** gatilho de atributo da conversa (origem paga) -> condicao de plataforma ->
  `ai_call`; `no_answer` e `error` -> ativar o agente de texto no WhatsApp.
- **Nao atendeu, tenta de novo:** `ai_call` -> `no_answer` -> esperar 2 horas -> `ai_call` -> `no_answer` ->
  mensagem no WhatsApp.
