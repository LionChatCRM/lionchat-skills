# Armadilhas do funil de vendas

Este arquivo e o catalogo das falhas que **nao dao erro**. A coisa e criada, parece certa, e nao funciona. Cada item tem: o que voce faz, o que acontece, o sinal e como sair.

## Indice

1. Substituicoes silenciosas ao gravar
2. Chave contra nome da etapa
3. A etapa fantasma
4. Automacoes que somem ou nunca rodam
5. Valor do cartao e ofertas
6. Checklist
7. Conversa do cartao
8. Cartao que "sumiu"
9. Ganho, perdido e a data do desfecho
10. Importacao de planilha
11. Permissoes
12. Coisas que parecem existir e nao existem
13. A conferencia final que pega quase tudo

---

## 1. Substituicoes silenciosas ao gravar

### As configuracoes do funil sao substituidas por inteiro

**Voce faz:** grava o funil mandando so as automacoes.
**Acontece:** as pessoas, os times e as metas somem. O funil que fica com a lista de pessoas vazia vira ABERTO a todos, e o rodizio de responsaveis reinicia.
**Sinal:** ninguem recebe erro. Alguem so percebe quando um vendedor ve os negocios dos outros.
**Como sair:** sempre leia com `lionchat_funnels_show`, monte o bloco completo (pessoas + times + metas + automacoes) e so entao grave. Releia depois.

### As etapas tambem sao substituidas por inteiro

**Voce faz:** grava as etapas repetindo tres das quatro que existiam.
**Acontece:** a quarta e EXCLUIDA. Se ela tiver cartoes, o sistema recusa com 422 `STAGE_HAS_ITEMS` (bom); se estiver vazia, some calada.
**Sinal:** a coluna desaparece do quadro.
**Como sair:** monte a lista de etapas a partir do que voce acabou de ler, nunca do zero. E lembre que a chave de uma etapa apagada nao deve ser reaproveitada em automacao nenhuma.

### "Esvaziar a etapa antes de apagar" dispara tudo

**Voce faz:** para apagar uma etapa com cartoes, transfere os cartoes com `lionchat_funnels_create_2` e depois regrava o funil sem a etapa (a receita antiga).
**Acontece:** cada cartao transferido conta como CHEGADA na etapa de destino: dispara as automacoes da etapa, as regras da conta, os fluxos (que podem falar com o cliente), os avisos externos e as conversoes de anuncio. E a etapa de origem continua la, so vazia.
**Como sair:** excluir etapa com cartoes e `lionchat_funnels_remove_stage`, que move em silencio e apaga a etapa. So com pedido explicito do dono.

### As listas da configuracao do Kanban tambem

**Voce faz:** grava so os modelos de checklist novos.
**Acontece:** os motivos de ganho, os de perda e os campos do cartao continuam intactos (voce nao os enviou), mas os modelos de checklist que ja existiam somem - a lista enviada substitui a anterior.
**Como sair:** leia com `lionchat_kanban_config_list`, junte o novo ao antigo e mande a lista completa. So as preferencias se misturam com o que ja existia.

## 2. Chave contra nome da etapa

**Voce faz:** escreve "Proposta Enviada" onde o sistema espera `proposta_enviada`.
**Acontece:** a automacao e salva e NUNCA dispara; o filtro devolve vazio; o relatorio de etapa levanta erro de etapa nao encontrada.
**Sinal:** na maioria dos casos, nenhum. A automacao aparece bonita na tela.
**Onde isso morre:** valor do gatilho da automacao, lista de etapas do filtro e etapa do relatorio.
(A importacao de planilha e a excecao: ela aceita tanto a chave quanto o nome visivel da etapa.)
**Como sair:** guarde o mapa nome para chave assim que criar o funil e use SEMPRE a chave. Confira em `lionchat_funnels_show`.

## 3. A etapa fantasma

**Voce faz:** cria um cartao apontando para uma etapa que nao existe mais.
**Acontece:** o sistema NAO da erro. Ele redireciona em silencio, tentando primeiro o destino registrado quando a etapa foi apagada, depois a etapa da mesma posicao, depois a primeira etapa.
**Sinal:** o cartao aparece numa coluna diferente da pedida e ninguem percebe.
**Como sair:** **sempre leia da resposta em qual etapa o cartao caiu de verdade**, e nao presuma que foi para onde voce pediu.

## 4. Automacoes que somem ou nunca rodam

| O que voce faz | O que acontece |
|---|---|
| grava sem o valor do gatilho | e DESCARTADA no proximo salvamento do funil pela tela - some do banco sem aviso |
| grava mover sem etapa, dar dono sem pessoa, duplicar sem funil ou etapa, avisar sistema externo sem endereco, aplicar checklist sem modelo | idem |
| grava "Criar tarefa" sem titulo, sem responsavel ou com prazo fora da faixa | recusada na hora com 422 dizendo a posicao da regra e o campo |
| grava a automacao desligada "para ativar depois" | ela fica guardada e NAO roda ate alguem ligar (ate outubro de 2026 a tela a apagava) |
| grava "dar dono" com o gatilho Ganho ou Perdido | e aceita e nunca faz nada: cartao fechado nao aceita responsavel novo |
| inventa um nome de gatilho (`status_changed`, `card_moved`, `on_enter`, `on_leave`) | o valor e SALVO sem erro e a automacao nunca roda |
| inventa um nome de acao (`send_message`, `set_priority`) | mesma coisa |
| espera o evento "cartao movido" na regra da CONTA | nao existe. La so ha cartao criado e cartao mudou de etapa |
| combina "chegou na etapa" com "mover de etapa" | o proprio motor bloqueia, para nao criar vaivem infinito |
| espera que "avisar o time" notifique alguem | ele so escreve uma linha de registro interno. Ninguem recebe nada |
| espera que a automacao de etapa mande mensagem ao cliente | nenhuma das nove acoes envia mensagem |
| aplica o mesmo modelo de checklist duas vezes | cria DOIS blocos iguais no cartao - nao ha verificacao de repeticao |
| poe como responsavel alguem sem acesso ao funil | pela automacao, e ignorado em silencio; pela ferramenta direta, da erro |

## 5. Valor do cartao e ofertas

### Valor escrito a mao em cartao com oferta

**Voce faz:** grava o valor do cartao.
**Acontece:** antes de salvar, o valor e substituido pela soma das ofertas.
**Sinal:** "o campo de valor nao salva".
**Como sair:** em cartao com oferta, mude a oferta. O valor digitado so vale em cartao sem nenhuma oferta.

### Editar a oferta esperando corrigir cartoes antigos

**Voce faz:** corrige o preco no catalogo.
**Acontece:** nada muda nos cartoes que ja receberam aquela oferta. A oferta dentro do cartao e uma foto congelada.
**Sinal:** o relatorio continua com o valor velho.
**Como sair:** o preco novo vale para cartoes criados dali para frente. Para corrigir um cartao antigo, troque a oferta dentro daquele cartao. Isso e regra de negocio, nao defeito.

## 6. Checklist

### Ler o campo de checklist procurando as tarefas

**Voce faz:** le o cartao e olha o campo de checklist.
**Acontece:** na maioria das leituras (detalhe do cartao, filtro, busca, cartao dentro da conversa) vem so um RESUMO de contagem, sem nenhum texto.
**Sinal:** voce conclui que um cartao com 12 tarefas nao tem checklist.
**Como sair:** para ler de verdade, use `lionchat_kanban_items_kanban_checklist_list`. A unica leitura em que o checklist vem completo e a lista de cartoes por funil.

### Somar os blocos achando que da o total

**Acontece:** tarefa avulsa (sem bloco) conta no total geral e nao aparece na lista de blocos. A soma dos blocos e MENOR que o total.

### Mandar checklist ou historico no salvamento comum do cartao

**Acontece:** sao ignorados. Isso e protecao: antes, um salvamento comum podia apagar o checklist aplicado por automacao e mutilar o historico.
**Como sair:** toda mudanca de checklist passa pelas ferramentas proprias.

## 7. Conversa do cartao

### Reenviar o numero da conversa num salvamento comum

**Acontece:** o sistema DESCARTA o valor cheio de proposito. Se nao descartasse, o numero antigo sobrescreveria o vinculo que o sistema ja moveu para a conversa mais recente. So o valor nulo passa - e ele significa desvincular.

### Tratar o numero da conversa como fixo

**Acontece:** o sistema reaponta o cartao para a conversa mais recente daquela pessoa naquela caixa. Uma integracao que guardou o numero antigo passa a mexer na conversa errada.
**Como sair:** releia o cartao antes de agir sobre a conversa dele. Para relatorio de origem existe um segundo ponteiro, o da conversa em que o cartao NASCEU, e esse nao muda.

### Gravar as conversas ligadas como numeros soltos

**Acontece:** quebra a leitura do cartao inteiro. O formato e uma lista de objetos, cada um com o numero dentro.

## 8. Cartao que "sumiu"

Antes de investigar dado, cheque acesso e situacao:

- **Esta adiado?** Cartao adiado sai da coluna e so aparece com o botao **Adiados** ligado. Leia `effective_status` e `snoozed_until` no cartao: ele volta sozinho na data.
- Quem nao e administrador so ve cartao cuja CONVERSA esteja numa caixa de entrada que ele acessa (salvo o cartao atribuido a ele e o sem dono de funil que ele acessa).
- A visibilidade tem tres niveis (ver `catalogo-de-campos.md`, secao 14): atendente sem cargo personalizado ve todos os cartoes dos funis que alcanca; cargo personalizado sem opcao de Kanban ve so os dele e os sem dono; cargo com o menu Kanban oculto nao ve nenhum.
- Cartao Ganho ou Perdido nao aparece por padrao: existem botoes de mostrar ou esconder ganhos, perdidos e adiados na barra superior do quadro.
- Funil arquivado some das abas - e some tambem para o agente de IA.

## 9. Ganho, perdido e a data do desfecho

| O que voce faz | O que acontece |
|---|---|
| marca Ganho um cartao que ja estava Ganho | a data NAO e empurrada para frente - o carimbo so e gravado quando o estado MUDA |
| reabre um cartao Ganho para "aberto" | a data do ganho, quem marcou, a quem foi creditado e o co-participante sao APAGADOS: aquele ganho some dos relatorios |
| marca hoje como Ganho uma venda que fechou no mes passado | conta no mes de HOJE. Use `retroativo_em` junto com o Ganho, ou o "Importar vendas" (`registrar-vendas.md`) |
| tenta corrigir a data de um ganho mandando Ganho de novo com outra data | nada muda: o carimbo so e escrito quando o estado muda. Reabra (`open`) e marque de novo com a data certa |
| tenta trocar o responsavel de um cartao Ganho ou Perdido | e recusado - os responsaveis ficam congelados |
| conta os ganhos do mes pela data de atualizacao ou de criacao | erro de semanas. Use o filtro por data de desfecho junto com o estado ganho |
| espera que a acao "criar cartao" reaproveite um cartao ja fechado | nao reaproveita, de proposito. A pessoa que volta ganha cartao NOVO e o fechado fica como historico. Ja a acao de MOVER continua movendo cartao fechado |
| cria coluna "Fechado" ou "Ganho" no funil | ganho e perdido sao ESTADO, nao coluna. Com coluna, o relatorio de receita e o ranking de vendedor param de fazer sentido |

## 10. Importacao de planilha

| O que voce faz | O que acontece |
|---|---|
| importa com as automacoes ligadas | **cada linha conta como cartao novo**: 5.000 linhas viram 5.000 disparos da automacao de cartao criado e do evento de etapa. Se a acao for avisar um sistema externo, sao 5.000 envios; se for dar dono, todos caem na mesma pessoa; se for duplicar, sao 5.000 copias. E, com o envio de conversao para anuncio ligado, milhares de conversoes falsas de negocio antigo |
| importa com telefone sem o codigo do pais | o telefone e considerado invalido e o cartao nasce SEM contato e SEM conversa, em silencio. O cartao fica orfao e nunca fala com ninguem |
| espera trazer valor, prioridade ou estado na planilha | **nao da.** A importacao aproveita titulo, descricao, nome do cliente, e-mail, telefone, etapa e (desde 23/10/2026) campos do cartao e atributos do contato escolhidos na tela. Todo cartao nasce ABERTO, sem valor e sem prioridade. Venda ja fechada entra pelo "Importar vendas" (`registrar-vendas.md`) |
| liga atributo do CONTATO na importacao de cartoes | contato que ja existia tem o valor SUBSTITUIDO, e isso conta como alteracao: acorda o fluxo "Atributo do contato muda" e o aviso externo de contato atualizado, uma vez por linha |
| passa de 10.000 linhas | a importacao e recusada |
| deixa a linha sem titulo, sem nome e sem telefone | so essa linha e recusada, com o erro no relatorio da importacao. Havendo nome ou telefone, eles viram o titulo do cartao |

**A ordem certa e importar ANTES de ligar as automacoes**, ou desliga-las durante a carga. Se a base ja foi importada com as automacoes ligadas, avise o cliente do que provavelmente disparou.

## 11. Permissoes

| O que voce supoe | A realidade |
|---|---|
| "quem gerencia o Kanban pode excluir cartao" | nao. Excluir e uma permissao separada |
| "quem gerencia o Kanban pode baixar a planilha" | nao. Exportar e importar e outra permissao separada (e e ela que aplica o "Importar vendas"). Ela expoe todos os leads, valores e telefones do funil |
| "quem gerencia o Kanban pode usar o Analisar com IA" | nao. E a permissao `kanban_ai_analyze` (ou administrador), com o recurso de plano ligado e a chave da OpenAI da conta |
| "lista de pessoas vazia fecha o funil" | ao contrario: lista efetiva vazia deixa o funil ABERTO a todos. Time sem membros conta como vazio |
| "excluir o vendedor mexe so nos cartoes dele" | hoje sim - mas ate agosto de 2026 havia um defeito que arrastava junto todo cartao SEM responsavel: numa conta, 1.289 cartoes trocaram de dono e nenhum era do vendedor excluido. Ja corrigido. Mesmo assim, leia a previa de reatribuicao antes de confirmar |

## 12. Coisas que parecem existir e nao existem

| Pedido | Realidade |
|---|---|
| aviso quando o cartao ficar parado | EXISTE desde 28/10/2026, mas so pelo FLUXO (gatilho "Card aberto na etapa" ou "Card aberto no funil"). Na automacao de etapa nao existe |
| notificacao no sininho de Kanban | nao existe nenhum tipo |
| o modelo de mensagem da etapa disparando sozinho | nao. So sai por acao manual, no menu do cartao |
| gatilho de saida de etapa | nao existe. So chegada |
| gatilho de exclusao de cartao | nao existe |
| exclusao de cartoes em massa | existe so na TELA (selecao de cartoes no quadro). Pelo conector nao |
| botao de duplicar funil | **existe na tela** (Kanban > engrenagem > Funis, o icone de copiar ao lado de cada funil) e nasce como "Nome (copia)". **Cuidado:** a copia ganha CHAVES NOVAS de etapa e as automacoes vem copiadas apontando para as chaves ANTIGAS - elas nunca disparam no funil novo. Nao ha ferramenta para isso; pelo conector e ler o funil e criar outro, o que e criacao e exige confirmacao |
| botao de duplicar cartao na tela | existe no codigo e nao esta ligado a nenhum menu. Quem duplica de verdade e a automacao de etapa |
| campo do funil chamado "atributos globais" | existe e **nenhuma tela do cartao le**. Os campos do cartao vem da configuracao do Kanban da conta |
| interruptores das preferencias do Kanban | varios sao gravados e nao mudam nada na tela. Nao prometa comportamento a partir deles |

## 13. A conferencia final que pega quase tudo

Configuracao nao esta pronta enquanto ninguem releu. Ja aconteceu de tudo passar nos testes e a tela reabrir vazia.

1. `lionchat_funnels_show` - as etapas, as pessoas, os times, as metas e as automacoes voltaram exatamente como voce gravou?
2. `lionchat_kanban_config_list` - os motivos, os modelos de checklist e os campos do cartao continuam la, **inclusive os que existiam antes de voce mexer**?
3. Crie um cartao de teste, mova-o pela primeira automacao e marque Ganho. Leia a resposta de cada passo: em qual etapa ele caiu? A automacao rodou? O motivo apareceu na lista?
4. Se algo nao voltou, nao tente adivinhar: releia, mostre a diferenca ao cliente e pergunte.
