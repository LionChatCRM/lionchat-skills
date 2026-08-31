# Armadilhas do Agente de IA

Cada item abaixo ja aconteceu com cliente de verdade. Estao em ordem de gravidade: primeiro as que falham
**em silencio** - a coisa e criada, parece certa na tela e nao funciona.

## Indice

1. Falham em silencio
2. Falham com aviso (o caso bom)
3. Erros de entendimento do produto
4. Como conferir que ficou certo

---

## 1. Falham em silencio

### O agente perfeito que nunca atende ninguem

**Se voce fizer:** montar o agente inteiro e nao ligar em lugar nenhum.
**Acontece:** ele fica perfeito e mudo. Nao existe mais vincular agente a caixa de entrada; a IA so
responde numa conversa que tem o agente atribuido.
**Conserto:** automacao, bloco de fluxo, lista suspensa da conversa, ou acao em massa. Ver
`references/ativar-e-diagnosticar.md`.

### O arquivo cadastrado que a IA nao enxerga

**Se voce fizer:** cadastrar na Biblioteca de Midia e nao liberar o id no agente.
**Acontece:** o arquivo aparece na tela da biblioteca, e as tres ferramentas de midia nem carregam. A IA
nao sabe que o arquivo existe.
**Conserto:** `config.media_asset_ids` no agente, com a lista COMPLETA. Vale igual para produtos
(`config.offer_ids`) e agendas (`config.booking_event_type_ids`).

### A ferramenta que salva no cenario e nao existe no atendimento

**Se voce fizer:** escrever `[Agendar](tool://create_booking)` numa conta sem agenda.
**Acontece:** o salvamento passa. Na hora do atendimento a ferramenta nao carrega, a IA le o passo mandando
agendar, nao tem como agendar e escreve que agendou.
**Por que:** a validacao do cenario confere o CATALOGO de ferramentas, nao se o recurso da conta existe.
**Conserto:** confirme o recurso antes (`lionchat_funnels_list`, `lionchat_booking_event_types_list`,
`lionchat_teams_list`, `lionchat_offers_list`) ou reescreva o cenario sem a promessa.

### A reuniao confirmada que nunca existiu

**Se voce fizer:** escrever no cenario "confirme o agendamento para o cliente".
**Acontece:** a IA com agenda liberada consulta os horarios de verdade, valida o pedido e, no ultimo passo,
escreve "sua reuniao esta confirmada" **sem marcar nada**. Caso real: cliente numa sala vazia.
**Conserto:** escreva o passo com a acao e a ordem: "marque de fato com [Agendar](tool://create_booking) e
so diga que esta confirmado DEPOIS que a ferramenta responder com sucesso".

### A variavel que some do texto

**Se voce fizer:** escrever `{{alguma.coisa}}` sem conferir a lista.
**Acontece:** nao da erro nenhum. A variavel some e a IA entrega a frase com o buraco. Caso real: a
instrucao usava um campo inventado e a IA escreveu "aqui esta o link:" seguido de nada.
**Conserto:** `lionchat_captain_liquid_variables_list` antes de escrever. O seletor de variaveis de outras
telas do produto oferece mais coisa do que a IA preenche - nao copie de la.

### O campo que o servidor descarta sem avisar

**Se voce fizer:** mandar um campo que nao esta na lista aceita.
**Acontece:** a resposta diz sucesso e o valor nunca foi gravado. Ja aconteceu com lista aninhada mais de
uma vez.
**Conserto:** releia com `lionchat_captain_assistants_show` depois de toda gravacao. Configuracao nao esta
pronta enquanto ninguem releu.

### A lista que apaga o resto

**Se voce fizer:** mandar `config.media_asset_ids: [5]` para acrescentar um arquivo.
**Acontece:** os outros ids somem. A mistura do `config` e parcial, mas ARRAY dentro dele e substituido
inteiro.
**Conserto:** leia o valor atual e mande a lista completa. Mesma coisa para `disabled_tools`, `offer_ids` e
`booking_event_type_ids`. E `playground_contact` precisa ir COMPLETO, porque a mistura e rasa: um objeto
parcial substitui a chave inteira.

### A ferramenta de fluxo que some dos outros agentes

**Se voce fizer:** vincular mandando so o id novo em `lionchat_flow_tools_assistants_update`.
**Acontece:** o endereco SINCRONIZA - a lista enviada substitui a inteira e todo id ausente e
desvinculado, em silencio.
**Conserto:** `lionchat_flow_tools_assistants_list` primeiro, some o novo, mande a lista completa.

### O acompanhamento que cai fora da janela

**Se voce fizer:** somar mais de 23 horas nas etapas de acompanhamento em caixa WhatsApp Oficial.
**Acontece:** a janela de 24 horas conta da mensagem DO CLIENTE, mas a cadencia so comeca a contar da
resposta da IA. Medido em producao: soma de 1430 minutos disparou 24h01m depois do cliente, a Meta recusou
e 48 acompanhamentos sumiram em 7 dias, sem nenhum aviso.
**Conserto:** teto de 1380 minutos (23 horas) na SOMA. Hoje o salvamento recusa acima disso, mas
**configuracao antiga acima do teto continua ativa e falhando** - a validacao so morde quando alguem salva
de novo. Se o cliente reclamar de acompanhamento que nao chega, releia a cadencia dele.

### O acompanhamento que nunca dispara

**Se voce fizer:** tratar "responder na hora" e acompanhamento como a mesma coisa.
**Acontece:** sao perguntas diferentes - uma e "ela fala agora?", a outra e "ela cobra depois se ninguem
falar?". Ja houve um periodo em que ficaram amarrados por engano e 140 conversas ficaram com IA ativa,
cliente parado e nenhuma cobranca.
**Conserto:** configure os dois separadamente e teste os dois separadamente.

### O time e a etiqueta sem descricao

**Se voce fizer:** criar time ou etiqueta so com o nome.
**Acontece:** e a DESCRICAO, nao o nome, que a IA le para decidir rotear e etiquetar. Sem ela, ou voce
chumba o nome exato em todo cenario (fragil), ou ela simplesmente nao usa.
**Conserto:** descreva QUANDO mandar para la, nao o que o time e. E lembre: a busca pelo nome e tolerante,
mas nunca adivinha entre dois candidatos - "Suporte" com "Suporte Tecnico" e "Suporte Comercial" nao
resolve.

### O campo "Descricao" que a tela chama de interno

**Se voce fizer:** escrever "agente da loja X, criado em agosto" no campo `description`.
**Acontece:** a tela diz "apenas para referencia interna", mas o texto vai INTEIRO para o comportamento da
IA, logo abaixo da identidade dela. Vira instrucao.
**Conserto:** escreva o papel dele em 1 a 3 frases.

### Os campos que nao fazem nada

`config.welcome_message` (aceito, gravado, ninguem le), `config.feature_citation` (o motor que responde o
cliente nao le), `config.activation_label` (jeito antigo), e a tela "Configuracoes de IA" da conta
(escolher modelo por funcao - nenhum servico le esses valores hoje). Nao gaste tempo com eles e nao mande o
cliente escolher o modelo la.

### A saudacao da caixa que para de sair

**Se voce fizer:** ligar a IA numa caixa que tinha saudacao configurada.
**Acontece:** com agente atribuido, a saudacao, o aviso de ausencia e a coleta de e-mail daquela caixa
param de ser enviados. O cliente jura que voce quebrou algo.
**Conserto:** avise ANTES e transforme a saudacao na primeira fala da IA (instrucao, com "responder na
hora" ligado).

### A pesquisa de satisfacao que dispara sozinha

**Se voce fizer:** deixar a IA encerrar a conversa numa caixa com pesquisa de satisfacao configurada.
**Acontece:** encerrar dispara a pesquisa.
**Conserto:** se o cliente nao quer, desligue `resolve_conversation` em `config.disabled_tools`.

### A resposta que chega picada

**Se voce fizer:** deixar uma linha com tres tracos sozinha na instrucao ou no cenario.
**Acontece:** e o sinal de separar a resposta em varias mensagens. O cliente recebe a mensagem quebrada.
**Conserto:** tire os tracos. A revisao de qualidade aponta isso.

### A criatividade que nao muda nada

**Se voce fizer:** ajustar a criatividade num modelo de raciocinio (`o1`, `o3`, `o3-mini`, `o4-mini`,
`gpt-5`, `gpt-5-mini`, `gpt-5.5`).
**Acontece:** o valor grava sem erro e nao produz efeito nenhum - esses modelos so aceitam o padrao, entao
o sistema nem manda o ajuste. Quem promete ao cliente "deixei mais solta" esta prometendo nada.
**Conserto:** escolha um modelo que aceita criatividade (coluna "Aceita criatividade" da tabela em
`references/modelo-e-ajustes.md`) ou ajuste o tom pelo texto das diretrizes de resposta.

### O agente mudo por causa do modelo

**Se voce fizer:** escolher um modelo da familia gpt-5.6.
**Acontece:** o campo do modelo NAO e validado pelo servidor - qualquer texto e aceito e gravado. Essa
familia recusa ferramentas, e como a IA usa ferramenta em praticamente toda resposta, 100% das chamadas
falham e ela fica muda, em silencio.
**Conserto:** use um dos 16 modelos da tabela em `references/modelo-e-ajustes.md`.

---

## 2. Falham com aviso (o caso bom)

- **Cenario citando ferramenta que o agente nao tem** volta erro dizendo que o texto contem ferramenta
  invalida. Isso e bom: avisa. Conserte o texto, nao insista.
- **Cenario citando uma das cinco ferramentas que nao podem ser citadas** (`check_agent_availability`,
  `get_flow_result`, `lookup_media_content`, `list_labels`, `schedule_self_callback`) da o mesmo erro. Elas
  carregam sozinhas; escreva o passo em portugues, sem a mencao.
- **Criar agente sem vaga no plano** volta erro de limite.
- **Cenario com `[@Nome](document://ID)` apontando documento de OUTRO agente** volta erro no salvamento.
- **Somar mais de 23 horas no acompanhamento** e recusado no salvamento.
- **Tipo de arquivo que nao casa com o tipo real** e recusado no cadastro da midia.

---

## 3. Erros de entendimento do produto

### "Instrucao maior deixa a IA melhor"

Nao. Instrucao gigante nao e problema de custo - e de CONFUSAO: a IA nao sabe QUANDO cada regra vale. E ha
um efeito grave: a rede de seguranca que cobra a IA quando ela diz que fez algo e nao fez **so roda em
agente COM cenario**. Quem joga tudo na instrucao fica sem essa protecao.

### "Escrevi 'nao invente' na instrucao, entao ela nao vai inventar"

Instrucao de comportamento nao e trava. Dois casos reais: uma conta com a regra "endereco nunca se
inventa" escrita NOVE vezes no texto e a IA passando endereco inexistente; e um modelo pequeno, sob a ordem
"nao invente", gravando na ficha um dado que o cliente nunca disse. O que segura e estrutura: base bem
organizada, cenario com passo a passo, ferramenta de verdade para executar.

### "Testei no Playground, esta pronto"

O Playground e fiel no raciocinio, mas toda acao de escrita roda em ensaio, o que exige contato e conversa
de verdade nao entra, e as redes de seguranca do atendimento real nao sao reproduzidas. Depois de ligar,
acompanhe as primeiras conversas de verdade.

### "A IA nao responde, entao o prompt esta errado"

Na maioria das vezes nao e o texto. Rode a lista de diagnostico da secao 9 de
`references/ativar-e-diagnosticar.md` primeiro - caixa WhatsApp Oficial fora da janela de 24 horas, chave
da OpenAI, agente pausado, IA desligada na mao, comentario de Instagram, e assim por diante.

### "O Copiloto e a mesma coisa"

Nao. O Copiloto ajuda o ATENDENTE dentro do painel e nao fala com o cliente. Ele tem motor proprio (modelo,
criatividade e comportamento separados) e conversas proprias. Mexer nele achando que esta configurando o
atendente virtual nao muda nada no atendimento.

### "Um agente por canal"

Um agente por PAPEL, nunca por canal. O mesmo agente atende WhatsApp, Instagram e site. Varios agentes so
fazem sentido quando os PAPEIS sao diferentes (vendas x suporte) ou quando se quer comparar duas versoes.

### "A IA nao entende audio nem imagem"

Entende. O audio do cliente e transcrito e entra no contexto dela, e a imagem ganha uma descricao gerada
automaticamente. **Nao escreva na instrucao "peca ao cliente para digitar o que ele falou no audio".** Um
detalhe: a descricao da imagem e gerada por um processo separado, entao uma foto recem-chegada pode entrar
na vez dela ainda sem descricao.

---

## 4. Como conferir que ficou certo

1. Reabra o agente (`lionchat_captain_assistants_show`) e confira campo por campo o que voce mandou.
2. Liste os cenarios e confira que o texto de cada um esta como voce escreveu.
3. Liste os documentos e confira que todos estao disponiveis, nenhum em processamento ou com erro.
4. Rode `lionchat_captain_assistants_tools` com o `assistant_id` e mostre ao cliente a lista real do que a
   IA pode fazer.
5. Rode `lionchat_captain_assistants_quality` e resolva os achados.
6. Teste no Playground com os 5 casos.
7. So entao ligue - e depois acompanhe as primeiras conversas de verdade pelo Supervisor.
