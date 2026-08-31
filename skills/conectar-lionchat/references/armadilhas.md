# Armadilhas: o que quebra, e o que quebra CALADO

Indice:

1. As que falham em silencio (as piores)
2. As que quebram a conexao inteira
3. As que enganam voce sobre o que a ferramenta faz
4. As que enganam voce sobre o que a resposta diz
5. As de fluxo (Flows)
6. As de conexao e sessao

Cada item esta escrito como: **se voce fizer X, acontece Y** — e como fazer certo.

---

## 1. As que falham em silencio (as piores)

Aqui a coisa e criada, parece certa e nao funciona. Nada avisa. So a releitura pega.

### Se voce mandar um campo que a ferramenta nao conhece, ele e descartado sem aviso

A resposta volta como sucesso e o painel segue com o valor antigo. Aconteceu de verdade: uma
configuracao de acompanhamento do agente de IA existia no sistema, na documentacao e ate na
descricao da ferramenta, mas nunca tinha entrado como campo — editar aquilo pela IA nunca surtiu
efeito, e ninguem percebeu.

**Certo:** so mande campos que aparecem na ficha da ferramenta. Depois releia.

### Se o campo inventado for o UNICO que voce mandou, o erro engana

O corpo da chamada fica vazio e volta "parametro faltando". Parece problema de formato de envio, mas
o problema e que o campo nao existe.

### Se voce filtrar por um nome que a ferramenta aceita mas o sistema nao usa, o filtro nao acontece

A chamada passa como sucesso, a lista volta, e **o filtro simplesmente nao foi aplicado**. O numero
que voce entrega ao cliente esta errado e nada avisa.

Caso real e documentado na propria ficha: `lionchat_search_list` aceita `inbox_id` e o **ignora em
silencio** — pedir "as conversas da caixa Comercial" por ali devolve TODAS, com cara de filtrado.
Para recortar por caixa e `lionchat_conversations_filter`.

**Certo:** leia a ficha do parametro antes de confiar nele, e teste mudando o valor do filtro. Se o
resultado nao mudar, o filtro nao esta valendo — nao entregue aquele numero.

### Se voce usar `lionchat_inbox_members_create` para adicionar um atendente, voce apaga os outros

Ele nao adiciona: **define a lista completa de membros da caixa.** Uma caixa com 111 membros fica
com 1.

**Certo:** atendente NOVO, use `lionchat_agents_create` ja com os times e as caixas dele (esse e
aditivo por atendente). Atendente que ja existe, leia a lista atual, junte o novo e mande a lista
COMPLETA. Cuidado: `lionchat_team_members_create` **e aditivo** — os dois contratos sao diferentes
de proposito.

### Se voce mandar um campo da configuracao do canal de uma caixa oficial, apaga os outros

Inclusive a chave que faz a caixa receber mensagem. **A caixa para de receber, em silencio.** Vale
para toda lista e para toda configuracao em bloco: etapas do funil, motivos de ganho e perda,
conversas vinculadas, distribuicao automatica.

**Certo:** leia com o `show`, junte na mao, reenvie inteiro.

### Se voce buscar com menos de 3 letras, volta vazio de proposito

Vazio ali **nao quer dizer "nao existe"**: quer dizer que a busca nao rodou. Concluir que o contato
nao existe cria um contato duplicado.

**Certo:** busque com 3 letras ou mais. So numeros e isento nas buscas novas, mas nao na busca
antiga de conversas.

### Se voce embrulhar o corpo ao criar um contato, ele nasce com o nome vazio

Criar contato e a unica excecao da plataforma: ali os campos vao na raiz. Todo o resto aceita os
dois jeitos.

### Se voce calcular data sem ler o fuso da conta, o relatorio sai errado

Nem toda conta e o fuso de Sao Paulo. Mato Grosso, Mato Grosso do Sul, Amazonas, Acre e clientes
fora do Brasil sao outro fuso. O fuso da conta vem em `lionchat_account_show`.

### Se voce criar um titulo de etiqueta com espaco, e recusado

`Lead Emive` nao passa; `lead-emive` passa. Outras regras da mesma familia: telefone no formato
internacional sem espacos, chave de etapa de funil em minusculo com sublinhado, motivos de ganho e
perda como objetos e nao texto solto, conversas vinculadas como objetos e nao numeros.

---

## 2. As que quebram a conexao inteira

### Se voce filtrar por area com o valor errado, o cliente fica sem nenhuma ferramenta daquela area

Sem erro nenhum. No conector local o filtro usa SLUGS; no remoto usa o nome da area em portugues. E
os dois nao sao iguais.

Erros comuns: `campaigns` nao casa nada (o slug e `campanhas`); `integrations` nao casa nada (e
`integracoes`); no remoto, `conversations` nao casa nada (la e `conversas`).

**Certo:** confira na tabela de `catalogo.md`, secao 3. Nao confie na documentacao do pacote, que
esta desatualizada.

---

## 3. As que enganam voce sobre o que a ferramenta faz

### Se voce deduzir pelo nome, voce vai chamar a coisa errada

246 ferramentas tem sufixo numerico (`_1` ate `_19`) e **nao sao variantes** da ferramenta base. Sao
operacoes diferentes que por acaso mexem no mesmo recurso:

- `lionchat_contacts_create_3` mexe nas ETIQUETAS do contato — e **SUBSTITUI a lista inteira**
- `lionchat_contacts_create_4` MESCLA dois contatos — **nao tem volta**
- `lionchat_contacts_create_5` EXPORTA a base e manda por e-mail

**Certo:** leia a ficha inteira — titulo E descricao. O caso das etiquetas mostra por que: o titulo
diz "Adicionar Labels" e a descricao diz "Substitui os labels do contato pelo array enviado". Quem
parou no titulo apagou as outras etiquetas da pessoa. Para acrescentar uma etiqueta, leia as atuais
com `lionchat_contacts_list_4` ("Labels do Contato"), junte na mao e mande a lista COMPLETA.

### Se voce chamar `lionchat_conversations_unread` achando que le, voce escreve

Ela **marca a conversa como nao lida**. Para listar as nao lidas, use a listagem de conversas com o
tipo "nao lidas"; para contar, `lionchat_conversations_meta`.

### Se voce usar `lionchat_conversations_search`, a sessao pode travar

E o caminho antigo e lento (medido entre 15 e 22 segundos em producao). Use
`lionchat_search_list` ou `lionchat_search_search`.

### `lionchat_account_list` nao lista contas da plataforma

Ele devolve o perfil do usuario logado, com as contas dele dentro. E o unico caminho de multi-conta
no conector local.

### Se voce tentar importar planilha, a ferramenta existe e sempre falha

`lionchat_contacts_import`, `lionchat_contacts_import_validate` e `lionchat_kanban_items_import`
exigem um arquivo, e enviar arquivo **nao existe** na conexao por IA. Elas aparecem na sua lista e
sempre recusam.

**Certo:** para criar muitos contatos, `lionchat_contacts_bulk_create` (so no conector remoto, ate
1000 por chamada). Ou mande o cliente importar pela tela.

### Existem 4 nomes de ferramenta que circulam por ai e NAO existem

| Nome que nao existe | Nome certo |
|---|---|
| `lionchat_contacts_export` | `lionchat_contacts_create_5` |
| `lionchat_conversations_update_last_seen` | `lionchat_conversations_create_6` |
| `lionchat_upload_limits` | `lionchat_upload_limits_show` |
| `lionchat_team_members_list` | `lionchat_teams_list_2` |

---

## 4. As que enganam voce sobre o que a resposta diz

### Se voce tentar ler uma chave de integracao, ela volta censurada — sempre

Nao ha como pedir a versao completa, nem com a opcao de resposta completa. Insistir vira laco.
Chave, senha e token nunca voltam. A unica excecao e o codigo publico do widget do site, que e
publico por desenho.

**Certo:** se o cliente precisa conferir uma chave, ele ve no painel.

### Se voce concluir "a conta nao tem horario de atendimento" porque veio um marcador, voce errou

Campos pesados (modelos de WhatsApp, horario de atendimento, historico de importacao, fotos de
perfil) vem podados por padrao para caber no limite de tamanho — vem um marcador de texto no lugar.
Podado nao e vazio.

**Certo:** use a opcao de resposta completa (existe em cerca de 28 ferramentas principais) ou chame
a ferramenta dedicada — para os modelos, `lionchat_inboxes_whatsapp_templates_list`.

### Se voce parar na primeira pagina, o relatorio sai incompleto com cara de completo

Conversas voltam 25 por pagina e contatos 15, **fixos** — pedir paginas maiores nao muda nada. E se
a resposta trouxer o aviso de lista enxugada (mostrando N de M), o trabalho nao acabou.

### Se voce ler `lionchat_list_categories` como a lista do que voce tem, erra

Ele conta o catalogo INTEIRO, inclusive ferramentas que aquela sessao nao carregou. As de
agendamento e pesquisa de satisfacao publicos sao desligadas de fabrica mas continuam contadas.

---

## 5. As de fluxo (Flows)

### Se voce montar o fluxo de cabeca, o gatilho pode nascer errado

Ja aconteceu de verdade: 53 fluxos de 9 contas ficaram rodando com o bloco Inicio EM BRANCO no
painel, porque o gatilho foi gravado num formato que a tela nao lia — e um clique em "Concluir"
naquele modal apagava o gatilho em silencio. Hoje a plataforma conserta esse formato sozinha na
hora de salvar, entao esse caso especifico nao volta.

O que continua valendo: o desenho do fluxo tem muitas regras que so quebram na execucao, nunca no
salvamento. **Chame `lionchat_flows_schema_reference` ANTES de criar ou alterar qualquer fluxo** e
peca tambem `lionchat://docs/flowbuilder-design-guide`. Nao invente o formato.

### Se uma saida do bloco nao tiver fio ligado, o fluxo TERMINA ali

Nao "cai no proximo". Vale para botao, opcao, condicao, tempo esgotado e resposta parcial. Espera
com tempo esgotado e sem fio encerra a sessao quando o tempo acaba.

### Se voce criar ferramenta da IA com `lionchat_flows_create`, e recusado

Ferramenta da IA usa `lionchat_flow_tools_create`. O `lionchat_flows_create` so cria fluxo de
conversa, e fluxo de conversa nao aceita bloco "Fim" — a chamada e recusada. O tipo do fluxo
tambem nao pode ser mudado depois: se nascer errado, e criar de novo pela porta certa.

Varios blocos "Fim" no mesmo desenho sao permitidos e uteis (um para o caminho normal, outro para
o caminho de erro): o motor usa o "Fim" onde a execucao REALMENTE chegou.

### Se voce ativar um fluxo que colide com outro, e recusado

Confira antes com `lionchat_flows_check_conflicts`, que testa sem salvar.

---

## 6. As de conexao e sessao

### Se voce passar `account_id` no conector remoto, ele e ignorado

La a conta vem do login e se troca com `lionchat_switch_account`. No local e o contrario: nao existe
troca de conta, e o campo `account_id` de cada chamada e o unico caminho. Skill que trate so um dos
dois vai mexer na conta ERRADA quando o cliente tem varias empresas.

### Se o cliente autorizar o conector remoto com a conta errada aberta no painel, ele leva recusa

A conta julgada e a que estava ABERTA no painel na hora de clicar em Autorizar — mesmo que ele seja
administrador de outra. E e nessa conta que a sessao nasce.

**Certo:** abrir a conta certa no painel ANTES de autorizar.

### Se o cliente tiver mais de um conector do LionChat no ChatGPT, eles se desconectam

O ChatGPT mantem um vivo e abandona os outros, que expiram e aparecem como "Desconectado".

**Certo:** um conector so por usuario. Para trocar de empresa, use `lionchat_switch_account` dentro
do proprio chat, nunca criando outro conector.

### Se voce paginar centenas de itens no remoto, pode levar recusa por frequencia no meio

O remoto aceita 120 chamadas por minuto. Espere um minuto e continue de onde parou — nao recomece,
e nao repita escritas que ja podem ter passado.

### Se o cliente reclamar que "conecta e desconecta" a cada poucos minutos no Claude Code

Era um defeito do servidor remoto, ja corrigido. Se ainda acontece, a instalacao dele esta com uma
versao antiga do conector remoto — nao e a skill nem a configuracao.

### Se voce repetir uma chamada que falhou por permissao ou por parametro invalido, so gasta tempo

Sao falhas permanentes. E chamada de CRIACAO que falhou nunca deve ser repetida direto: confira com
uma leitura antes, senao voce cria dois registros.

### Se voce disser "nao existe funcao para isso" sem pesquisar, provavelmente esta errado

Ja aconteceu em producao: a sessao que afirmou nao haver funcao de listar conversas achou dezenas na
primeira pesquisa. Pesquise pelo padrao `lionchat_<area>_<acao>` antes de negar.
