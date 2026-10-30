# Diagnostico - mensagem de erro e o que fazer

Onde olhar: `lionchat_flows_executions_list` (fluxo) e `lionchat_flow_sessions_show` (passo a passo; o bloco
`ai_call` traz `error_code` e o texto). Na ligacao de teste, o 422 traz o motivo em `message`.

| Mensagem | Causa | O que fazer |
|---|---|---|
| A Ligacao com IA nao esta liberada nesta conta | recurso `ai_calling` desligado | pedir a liberacao (nao e pelo conector) |
| A Ligacao pelo WhatsApp nao esta ligada nesta conta | recurso `lioncalls_calling` desligado | idem |
| Escolha um agente de ligacao que exista nesta conta | id errado no bloco (muitas vezes o id de um agente de TEXTO) ou agente apagado | trocar `aiCallAgentId` por um id de `captain_call_agents_list` |
| Este agente de ligacao nao e desta conta | id de outra conta | idem |
| Informe o id do agente da ElevenLabs | fornecedor ElevenLabs sem `elevenlabs_agent_id` | preencher ou voltar para OpenAI |
| A conta nao tem chave da OpenAI/ElevenLabs configurada | sem chave do fornecedor | cadastrar em Integracoes (nao e pelo conector) |
| Esta caixa nao tem ligacao configurada / A linha escolhida neste bloco nao existe mais | caixa sem linha de ligacao | escolher uma caixa de `meta.dial_inboxes` |
| A linha de ligacao desta caixa esta desconectada | linha caida | reconectar a linha na caixa |
| O contato desta conversa nao tem telefone / Telefone invalido | contato sem telefone valido | corrigir o telefone |
| A Ligacao com IA nao funciona em conversa de grupo | fluxo rodou num grupo | nao usar em grupo |
| A linha esta ocupada / ficou ocupada por mais de 2 horas | muitas ligacoes ao mesmo tempo | espacar os disparos |
| Esta linha faz uma ligacao por vez; a IA precisa de duas vagas | limite de ligacoes da linha = 1 | falar com o suporte (o limite da linha nao se muda pelo conector) |
| O servico de ligacao nao respondeu / recusou | motor de ligacao | tentar de novo em alguns minutos; persistindo, suporte |
| Nao foi possivel confirmar se a ligacao saiu | resposta do motor nao voltou | o fluxo NAO liga de novo sozinho, de proposito; conferir no historico de ligacoes |

## Sintomas sem mensagem de erro
- **A voz liga e nao sabe o que dizer / enrola:** `objective` e `first_message` vazios. A voz nao le o agente de
  texto.
- **A voz disse que ia mandar e nada chegou:** agente de ligacao sem `captain_assistant_id`, motor que nao e
  `gptlive`, ferramenta desligada no agente de texto, janela de 24h fechada no WhatsApp oficial ou mais de 3
  pedidos. Olhe as pilulas de pedido na conversa.
- **Os atributos nao foram gravados:** pedido vago (sem o nome do atributo) ou atributo inexistente na conta.
- **Ligacao atendida e muda:** testar de novo com o WhatsApp do destino so no celular; se repetir, suporte.
