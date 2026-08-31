# Automacoes DENTRO do funil (o segundo motor)

Leia isto so quando o cliente falar de automacao configurada **dentro do funil**, em
Kanban, Gerenciar Funis, secao Automacoes — ou quando ele disser "a automacao do funil nao roda" e
voce nao achar regra nenhuma na tela de Automacao.

---

## O que e, e por que confunde

Existem **dois sistemas de automacao** no LionChat, com nomes parecidos e comportamentos diferentes:

| | Automacao (Configuracoes, Automacao) | Automacao do funil (Kanban, Gerenciar Funis) |
|---|---|---|
| Onde fica | Tela propria, lista com liga/desliga | Dentro do funil |
| Aparece na lista de automacoes | Sim | **Nao** |
| Tem historico de execucoes | Sim (48 horas), com condicoes e passos | **Nao** — so uma linha na Atividade do card quando uma acao roda |
| Ferramenta propria | Sim | **Nao** — so mexendo no funil |
| O que ela alcanca | A conversa e o card | So o card |

Os gatilhos de card da tela de Automacao (Card Kanban Criado / Movido) sao do PRIMEIRO sistema. O que
esta descrito aqui e o segundo.

---

## Como se mexe nisso

Nao ha ferramenta dedicada. O caminho e:

1. Ler o funil inteiro com `lionchat_funnels_show`.
2. Pegar o bloco de configuracoes que voltou (ele carrega metas, agentes, times **e** automacoes).
3. Acrescentar a automacao nova a lista de automacoes.
4. Mandar o bloco **inteiro** de volta com `lionchat_funnels_update`.

**Gravar as configuracoes SUBSTITUI tudo.** O que voce nao mandar de volta some — metas, agentes,
times e as outras automacoes junto. Leia sempre antes de escrever.

Nao ha nenhuma validacao desse conteudo: nome errado de gatilho ou de acao grava com sucesso e a
automacao simplesmente nunca dispara, sem erro e sem lugar nenhum para olhar.

---

## Os 3 gatilhos reais

| Gatilho | Nome tecnico | O valor que acompanha |
|---|---|---|
| Card criado | `card_created` | a palavra `card_created` |
| Card chegou numa etapa | `stage_moved` | a chave da etapa de **destino** |
| Card marcado como Ganho ou Perdido | `status_change` | `won` ou `lost` |

Nao existe `status_changed` com "d" no fim. Nome errado = automacao morta em silencio.

O valor que acompanha o gatilho e **obrigatorio**: automacao sem esse valor, sem acao escolhida, ou
desligada, e descartada pela tela no proximo salvamento do funil pelo cliente.

---

## As 8 acoes reais

| Acao | Nome tecnico | Configuracao |
|---|---|---|
| Mover para etapa | `move_to_stage` | `{"stage": "chave_da_etapa"}` |
| Atribuir agente | `assign_agent` | `{"agent_id": 12}` |
| Criar nota | `create_note` | `{"note_text": "texto"}` |
| Aplicar modelo de checklist | `apply_checklist_template` | `{"template_id": 3}` |
| Atualizar checklist | `update_checklist` | `{"checklist_updates": [...]}` |
| Avisar a equipe | `notify_team` | `{"message": "texto"}` |
| Copiar o card para outro funil | `duplicate_item` | `{"funnel_id": 7, "stage": "chave_da_etapa"}` |
| Enviar webhook | `send_webhook` | `{"webhook_url": "https://..."}` |

Nao existe `duplicate_to_funnel`. A acao de copiar card chama-se `duplicate_item`.

---

## Armadilhas

- **"Avisar a equipe" nao avisa ninguem hoje.** Ela apenas registra no servidor. Nao proponha essa
  acao como forma de alertar a equipe — para isso use a automacao normal, com a acao de e-mail para o
  time.
- **"Mover para etapa" nunca roda no gatilho "Card chegou numa etapa"**, de proposito (senao dois
  cards ficariam se empurrando eternamente). A tela esconde a opcao; se ela for gravada por fora, o
  motor ignora.
- **"Copiar o card para outro funil" nao aparece no gatilho "Card criado"** (a tela esconde). Nao
  proponha essa combinacao: o cliente nao vai encontrar a opcao.
- **"Atualizar checklist" nao aparece na tela** — so existe por fora.
- **Nao ha historico de verdade.** O unico rastro e a aba **Atividades do card**: cada acao que roda
  deixa ali uma linha de acao automatica. Isso separa "rodou e nao fez nada" de "nunca rodou", mas
  nao diz o motivo, nao mostra a configuracao lida e some junto com o card. Nao ha nada equivalente
  ao Historico da automacao normal. Por isso, sempre que der para resolver o mesmo problema com a
  automacao normal, prefira a automacao normal.
- **Nada disso manda mensagem para o cliente.** As 8 acoes mexem no card, no checklist e em avisos
  internos. Mensagem para o cliente sai por automacao normal (imediata) ou por Flow (com espera).
