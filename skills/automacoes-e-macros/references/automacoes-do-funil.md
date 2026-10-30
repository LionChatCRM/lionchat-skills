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

Quase nada desse conteudo e validado: nome errado de gatilho ou de acao grava com sucesso e a
automacao simplesmente nunca dispara, sem erro e sem lugar nenhum para olhar. A unica excecao e a
acao "Criar tarefa" (`create_task`): regra nova ou alterada e conferida e volta 422 dizendo qual
regra e qual campo consertar. Regra sem `id` ganha um `id` do servidor ao salvar.

**Ordem:** as automacoes do mesmo gatilho rodam na ORDEM da lista. Quer que uma rode antes da outra,
ponha antes na lista.

---

## Os 3 gatilhos reais

| Gatilho | Nome tecnico | O valor que acompanha |
|---|---|---|
| Card criado | `card_created` | a palavra `card_created` |
| Card chegou numa etapa | `stage_moved` | a chave da etapa de **destino** |
| Card marcado como Ganho, Perdido ou Adiado | `status_change` | `won`, `lost` ou `snoozed` (11/10/2026) |

Nao existe `status_changed` com "d" no fim. Nome errado = automacao morta em silencio. `snoozed`
dispara quando o card PASSA a ser adiado (trocar a data de um card ja adiado nao dispara). Nao existe
gatilho "Aberto" aqui - para "card voltou do adiamento" use o gatilho "Card aberto" de um Flow.

O valor que acompanha o gatilho e **obrigatorio**: automacao sem esse valor, sem acao escolhida, ou
desligada, e descartada pela tela no proximo salvamento do funil pelo cliente.

---

## As 9 acoes reais

| Acao | Nome tecnico | Configuracao |
|---|---|---|
| Mover para etapa | `move_to_stage` | `{"stage": "chave_da_etapa"}` |
| Atribuir agente | `assign_agent` | `{"agent_id": 12}` |
| Criar nota | `create_note` | `{"note_text": "texto"}` |
| Aplicar modelo de checklist | `apply_checklist_template` | `{"template_id": 3}` |
| Atualizar checklist | `update_checklist` | `{"checklist_updates": [...]}` |
| Avisar a equipe | `notify_team` | `{"message": "texto"}` — NAO faz nada (ver armadilhas) |
| Copiar o card para outro funil | `duplicate_item` | `{"funnel_id": 7, "stage": "chave_da_etapa"}` |
| Enviar webhook | `send_webhook` | `{"webhook_url": "https://..."}` |
| Criar tarefa (20/10/2026) | `create_task` | `{title, description, task_type, agenda_id, priority, occupies_agenda, duration_minutes, due: {amount, unit, at, business_days}, responsible: {mode, user_id, team_id, fallback_user_id}}` — so com a Agenda unificada ligada; os mesmos campos da acao "Criar tarefa" do bloco Acoes do Flow (skill `criar-fluxos`) |

Nao existe `duplicate_to_funnel`. A acao de copiar card chama-se `duplicate_item`.

---

## Armadilhas

- **"Avisar a equipe" nao avisa ninguem.** Ela apenas registra no servidor, e desde 22/10/2026 a
  tela nem oferece mais essa opcao (regra antiga com ela continua aparecendo, com aviso de "sem
  efeito"). Nao proponha - para alertar a equipe use a automacao normal, com a acao de e-mail para o
  time.
- **"Atribuir agente" nos gatilhos Ganho e Perdido nao faz nada**: a automacao roda com o card ja
  fechado, e card fechado nao aceita responsavel novo. A tela tambem tirou essa combinacao.
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
- **Nada disso manda mensagem para o cliente.** As 9 acoes mexem no card, no checklist, na Agenda e
  em avisos internos. Mensagem para o cliente sai por automacao normal (imediata) ou por Flow (com espera).
