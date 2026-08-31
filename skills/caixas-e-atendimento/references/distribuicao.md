# Repartir as conversas: distribuicao automatica, politica, time e capacidade

Indice:
1. As quatro pecas e como escolher
2. Modo simples (na propria caixa)
3. Politica de atribuicao
4. Time
5. Politica de capacidade (teto por atendente)
6. Precedencia: quem manda quando ha mais de um
7. Quem entra no sorteio, na pratica
8. Presenca real x status escolhido no menu
9. Freio de rajada (Protecao contra acumulo)
10. Por que a conversa demorou a ser atribuida
11. Arvore de decisao pronta

---

## 1. As quatro pecas e como escolher

| Peca | Quando usar |
|---|---|
| Modo simples na caixa | Fila unica, uma ou poucas caixas. E o modo em que a caixa nasce |
| Politica de atribuicao | Varias caixas que devem seguir a MESMA regra, ou quando o cliente quer escolher qual conversa da fila sai primeiro |
| Time | O atendimento e dividido por area e a conversa e entregue ao time, nao a pessoa |
| Politica de capacidade | Existe um teto de conversas abertas por atendente, por caixa |

Modo simples e politica de atribuicao sao EXCLUDENTES na mesma caixa. Time e capacidade se somam a
qualquer um dos dois.

---

## 2. Modo simples (na propria caixa)

Reparte as conversas SEM responsavel entre os membros da caixa. Nao precisa criar nada.

| Ajuste | Valor | Nasce |
|---|---|---|
| `enable_auto_assignment` | ligado / desligado | LIGADO - caixa nova ja reparte sozinha |
| `assign_offline_agents` | ligado / desligado | desligado |
| `fair_distribution_limit` | quantas conversas por atendente na janela; 0 aqui significa sem freio | 10, mas ver a excecao do item 9 |
| `fair_distribution_window` | tamanho da janela em SEGUNDOS | 600 (10 minutos) |
| `assignment_order` | `round_robin` (rodizio) ou `balanced` (equilibrado) | rodizio |

Os quatro ultimos vivem dentro do bloco `auto_assignment_config`, e **esse bloco e SUBSTITUIDO
inteiro a cada gravacao, nunca mesclado**. Mandar so o freio apaga o "distribuir para offline" que o
cliente tinha, e vice-versa. O procedimento correto e sempre: ler a caixa com
`lionchat_inboxes_show`, juntar o que mudou ao que ja existia e mandar o bloco COMPLETO em
`lionchat_inboxes_update`.

O modo Equilibrado depende de um recurso de plano. Em conta sem ele, a opcao aparece BLOQUEADA na
tela (com selo de plano) e o cliente nao consegue escolher por ali. Nao ofereca Equilibrado antes de
confirmar com `lionchat_account_show` que o recurso esta ligado na conta.

---

## 3. Politica de atribuicao

Um conjunto de regras criado uma vez e vinculado a varias caixas. Na tela fica em Configuracoes >
Atribuicao de Agentes.

| Campo | Valores | Nasce |
|---|---|---|
| `name` | texto, unico na conta | obrigatorio |
| `assignment_order` | `round_robin` ou `balanced` | rodizio |
| `conversation_priority` | `earliest_created` (a mais antiga) ou `longest_waiting` (quem espera ha mais tempo) | mais antiga |
| `assign_offline_agents` | ligado / desligado | desligado |
| `fair_distribution_limit` | precisa ser MAIOR que zero (aqui o zero e recusado) | 10 |
| `fair_distribution_window` | segundos, maior que zero | 600 |
| `enabled` | ligado / desligado | ligado |

Passos: `lionchat_assignment_policies_create` cria; `lionchat_inboxes_assignment_policies_create`
vincula a caixa. Politica criada e nao vinculada nao faz nada.

**Quando existe politica vinculada, a secao de distribuicao SOME da tela da caixa** e o que estava
ajustado ali vira configuracao fantasma: o cliente nao ve e nao consegue corrigir, e o comportamento
passa a ser o da politica. Nunca configure os dois.

Politica com `enabled` desligado faz a caixa parar de distribuir - com uma excecao importante,
descrita no item 6.

---

## 4. Time

Agrupa atendentes. Quando a conversa e atribuida a um TIME, o sistema sorteia um membro.

| Campo | Valores | Nasce |
|---|---|---|
| `name` | texto, unico na conta | obrigatorio |
| `allow_auto_assign` | o time distribui sozinho para os membros | LIGADO |
| `assignment_mode` | `online_only` (so quem tem presenca real) ou `include_offline` (todos) | online_only |
| `assignment_order` | `round_robin`, `balanced` ou ausente | ausente = herda do nivel de cima |
| `fair_distribution_limit` / `fair_distribution_window` | freio proprio do time | 10 / 600 |

Membros: `lionchat_team_members_update` recebe a lista COMPLETA - quem nao estiver nela e removido do
time.

**Marcar supervisor de TIME nao e possivel pelas ferramentas.** O sistema aceita supervisor de time
(a pessoa continua membro para mencao, e-mail, relatorio e acesso a funil, mas fica fora do sorteio),
so que o campo nao esta declarado nas ferramentas de membros de time. Isso e feito no painel.

**Ser membro do time NAO da acesso a caixa.** O candidato do sorteio e a intersecao: precisa estar na
lista de membros do TIME e na lista de membros da CAIXA. Quem esta so no time nunca recebe nada
daquela caixa, e nao ha erro nenhum na tela.

---

## 5. Politica de capacidade (teto por atendente)

Teto de conversas ABERTAS que um atendente pode acumular POR CAIXA. Enquanto ele estiver no teto, o
sistema nao entrega mais nada para ele naquela caixa.

Sao TRES passos e todos sao obrigatorios:

1. `lionchat_capacity_policies_create` - cria a politica (nome, descricao e regras de excecao:
   ignorar conversas com certas etiquetas, ignorar conversas mais velhas que N horas).
2. `lionchat_capacity_policies_create_2` - define o limite de conversas para CADA caixa.
3. `lionchat_capacity_policies_create_1` - vincula CADA agente a politica, um por chamada.

Se faltar o passo 2, aquela caixa fica com capacidade ilimitada. Se faltar o passo 3 na conta
inteira, o filtro de capacidade nem chega a rodar. Nos dois casos o cliente acha que o teto esta
valendo e nao esta.

A politica de capacidade depende de DOIS recursos de plano ligados na conta. Se ela nao aparecer na
tela do cliente, e isso - nao e erro de configuracao.

---

## 6. Precedencia: quem manda quando ha mais de um

Para escolher rodizio ou equilibrado, a ordem e: **time, depois politica, depois caixa, depois
rodizio**. O time so entra na conta quando ele mesmo escolheu (ausente significa "nao escolhi" e a
decisao sobe de nivel).

Para distribuir a quem esta offline: **se ha politica vinculada, vale a politica**; senao vale o
ajuste da caixa.

Uma excecao que gera reclamacao legitima: **quando a conversa esta atribuida a um TIME, quem autoriza
a entrega e o `allow_auto_assign` do time, nao a caixa nem a politica.** O cliente pode desligar a
distribuicao automatica da caixa e continuar recebendo atribuicao automatica pela fila do time, e
reportar isso como defeito. Se ele quer parar tudo, desligue tambem a distribuicao do time.

---

## 7. Quem entra no sorteio, na pratica

Para receber uma conversa da caixa X, a pessoa precisa cumprir TODAS as condicoes:

1. Ser membro da caixa X.
2. Estar como AGENTE naquela caixa, nao como supervisor. Supervisor ve tudo e nunca e sorteado. Se o
   cliente marcar a equipe inteira como supervisor, a distribuicao para em silencio.
3. Estar com presenca real E com o status "online" - "ocupado" e "ausente" ficam de fora. A excecao
   e "distribuir para offline" ligado: ali entram TODOS os membros que sao agentes, sem olhar
   presenca nem status.
4. Se a conversa foi para um time: ser membro daquele time, e como membro sorteavel.
5. Nao estar no teto da politica de capacidade daquela caixa, se houver.

CUIDADO ao conferir: `lionchat_inboxes_list_1` e `lionchat_assignable_agents_list` respondem OUTRA
pergunta - quem pode ser ESCOLHIDO como responsavel na mao (membros da caixa mais TODOS os
administradores da conta). Elas nao olham supervisor, presenca nem teto, entao nao servem para
descobrir quem entra no sorteio automatico. Para isso, compare a lista de membros
(`lionchat_inbox_members_show`) com as condicoes acima.

---

## 8. Presenca real x status escolhido no menu

O motor exige PRESENCA REAL: a pessoa com o painel aberto. O status que ela escolheu no menu nao
basta. Equipe inteira marcada como online, com o painel fechado, e fila parada e ninguem percebe.

A excecao e o privilegio "permanecer disponivel" (`auto_offline` desligado): quem tem esse privilegio
conta como disponivel 24 horas, mesmo com o painel fechado. So administrador pode conceder.

Cuidado ao conceder para varias pessoas: de madrugada, quando os colegas fecham o painel e saem da
presenca, quem tem o privilegio vira o unico elegivel e leva TODOS os leads. Conceda com intencao,
nao por preferencia pessoal.

Para equipe que responde pelo celular sem manter o painel aberto, o ajuste certo NAO e esse
privilegio e sim ligar "distribuir tambem para agentes offline" na caixa ou na politica.

---

## 9. Freio de rajada (Protecao contra acumulo)

Limita quantas conversas o sistema entrega a CADA atendente dentro de uma janela de tempo. O padrao
vale sem ninguem configurar nada: 10 conversas a cada 10 minutos, por atendente e por caixa. A vaga
volta assim que a conversa deixa de estar aberta - resolve uma, recebe outra.

**Excecao importante:** caixa com "distribuir tambem para agentes offline" LIGADO e SEM numero
escolhido nasce SEM freio nenhum. E de proposito: com offline ligado todo membro esta sempre na
roleta, entao a divisao ja e igual e o freio so atrasaria a entrega. Numero escolhido a mao SEMPRE
vence - o interruptor so muda o padrao. Vale o mesmo para o time em modo "incluir offline".

Quando o freio age:

- **Rodizio**: so quando ha fila (mais de uma conversa esperando). Entrega isolada passa direto.
- **Equilibrado escolhido numa POLITICA de atribuicao**: age SEMPRE, em toda entrega. Ali ele e
  controle de capacidade, nao apenas drenagem de acumulo - quem espera entrega imediata em qualquer
  situacao vai ver conversa parada.
- **Equilibrado escolhido na propria caixa ou no time** (sem politica): segue a regra do rodizio, ou
  seja, so age quando ha fila.

Na caixa, `fair_distribution_limit: 0` e aceito e significa sem freio. Na politica de atribuicao o
zero e RECUSADO.

O freio e por par atendente e caixa. Conversa entregue pela fila do time tambem conta no freio da
caixa e vice-versa.

---

## 10. Por que a conversa demorou a ser atribuida

A distribuicao acontece por evento. Existem buracos reais em que nenhum evento dispara (por exemplo,
alguem voltar a ficar online, ou uma rodada barrada pelo freio). Uma varredura de seguranca roda a
cada minuto e reenfileira o que ficou parado.

Ou seja: espera de ate um minuto e comportamento normal, nao defeito. Antes de investigar, confirme
que se passou mais do que isso.

---

## 11. Arvore de decisao pronta

- Uma caixa so, fila unica, ninguem offline? Modo simples, rodizio, freio no padrao. Nao crie nada.
- Varias caixas com a mesma regra, ou o cliente quer que a conversa mais antiga saia primeiro?
  Politica de atribuicao, vinculada a cada caixa. Nao ajuste a distribuicao da caixa depois disso.
- Quer que o lead va para quem esta menos ocupado? Equilibrado (confirme que o recurso esta liberado
  na conta) e lembre que o freio passa a agir em toda entrega.
- Areas separadas? Times, com as pessoas tambem nas caixas correspondentes. Defina se o time entrega
  so para quem esta online.
- Equipe pelo celular? Ligue "distribuir para offline" no nivel que estiver mandando (politica se
  houver, senao caixa).
- Alguem so acompanha e nao atende? Coloque como supervisor da caixa.
- Existe teto por pessoa? Politica de capacidade com os tres passos.
