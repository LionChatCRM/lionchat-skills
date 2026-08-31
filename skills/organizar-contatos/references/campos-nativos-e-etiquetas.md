# O que ja existe pronto, e as regras de etiqueta, empresa, vinculo e origem

Leia antes de criar qualquer campo ou etiqueta. A pergunta que este arquivo responde e: **isso ja
existe? preciso mesmo criar alguma coisa?**

## Indice

1. A identidade da ficha
2. Dados cadastrais brasileiros (CPF, RG, CNPJ, endereco)
3. Origem do lead
4. Campos de sistema (rastreamento) — existem, mas ficam escondidos
5. Nomes proibidos ao criar campo
6. Etiquetas: as regras e as duas etiquetas diferentes
7. Empresas
8. Vinculo entre duas fichas
9. Variaveis da conta
10. Notas, Documentos e Preenchimentos
11. Ficha duplicada e mesclagem

---

## 1. A identidade da ficha

Nao crie campo personalizado para nada disto — ja existe na ficha:

| Ja existe | Observacao |
|---|---|
| Nome | Nas mensagens: nome inteiro, primeiro nome e sobrenome saem prontos |
| E-mail | Unico na conta, gravado sempre em minusculas |
| Telefone | Precisa vir com o codigo do pais. Unico na conta |
| Identificador | O codigo da pessoa em OUTRO sistema (ERP, clinica, escola). Unico na conta |
| Bloqueado | Mensagem de quem esta bloqueado e descartada na entrada |
| Cidade e Pais | Ficam na ficha e sao filtraveis |
| Empresa | Nome da empresa da pessoa |
| Tipo do contato | Visitante, Lead ou Cliente |
| Foto | O avatar da ficha |

Sobre o **tipo do contato**: a ficha que chega por um canal ou pelo chat do site nasce Visitante e
vira **Lead sozinha** assim que ganha e-mail, telefone ou perfil de rede social. Ja a ficha criada a
mao — pelo painel ou por ferramenta — nasce **Lead direto**, e por isso aparece na lista mesmo com
so o nome preenchido.

Isso importa por um motivo pratico: **contato que ainda seja Visitante e nao tenha telefone, e-mail
nem identificador nao aparece na lista de contatos** — nem entra em filtro, segmento ou exportacao.
Se o cliente disser que uma ficha "sumiu", verifique isso primeiro.

Sobre o **identificador**: e o campo certo para o codigo do paciente, do aluno, do pedido. Nao crie
um campo personalizado "codigo_do_sistema" — use o identificador, que ja e unico e ja entra na busca.

## 2. Dados cadastrais brasileiros (CPF, RG, CNPJ, endereco)

Existem prontos, com validacao propria, busca por documento e variaveis prontas para mensagem.
**Criar um campo personalizado chamado "CPF" e um erro de projeto**: quebra tudo isso.

| Dado | Regra |
|---|---|
| CPF, RG, CNPJ, Passaporte | So a PRIMEIRA gravacao vale |
| Data de nascimento | So a PRIMEIRA gravacao vale |
| Genero | So a PRIMEIRA gravacao vale. Valores aceitos: `m`, `f`, `o`, `na` |
| Estado civil | Pode ser alterado. Valores: `solteiro`, `casado`, `uniao_estavel`, `divorciado`, `separado`, `viuvo` (aceita a forma feminina e converte) |
| Profissao | Pode ser alterado, texto livre |
| Endereco | Pode ser alterado. CEP, rua, numero, complemento, bairro, cidade, UF e pais |

Pontos que mudam o que voce diz ao cliente:

- **A imutabilidade e real e silenciosa.** Uma segunda gravacao com valor diferente e ignorada e vira
  um pedido de confirmacao. Nunca afirme "gravei o CPF" sem ler a ficha de volta. Quem pode
  sobrescrever de verdade e a pessoa, na tela.
- **Genero e estado civil so guardam os valores da lista acima.** O sistema converte sozinho algumas
  formas faladas ("masculino", "feminino", "solteira", "casada", "uniao estavel"); qualquer outro
  texto e recusado, e a recusa derruba o pedido inteiro, nao so aquele campo.
- **Ha uma ferramenta de busca de CEP** que devolve rua, bairro, cidade e UF: use antes de pedir o
  endereco inteiro digitado.
- **A cidade e a UF do endereco sao copiadas para os campos da ficha**, entao filtrar por cidade
  continua funcionando.
- Esses dados sao gravados por uma ferramenta PROPRIA de dados cadastrais, nao pela ferramenta de
  editar contato.

## 3. Origem do lead

A conta ja nasce com **17 campos automaticos** de origem: 12 na ficha e 5 na conversa.

Na ficha: 1a Origem (plataforma, tipo, campanha, conjunto, criativo e data) e Ultima Origem (os
mesmos seis). Na conversa: plataforma, tipo, campanha, conjunto e criativo daquele atendimento.

Eles alimentam o relatorio **Origem dos Leads**. Nao crie campo "de onde veio".

**Origens personalizadas** (indicacao, evento, panfleto, o que o cliente inventar) sao cadastradas a
parte, no menu Origens de Lead. Regras:

- Nome de ate 60 caracteres, no maximo **50 origens ativas** por conta.
- **Excluir nao exclui: desativa.** E de proposito, para nao quebrar relatorio antigo.
- **Criar de novo com o mesmo nome REATIVA a antiga** em vez de duplicar.
- **O apelido tecnico e imutavel.** Renomear muda so o rotulo na tela; o valor ja gravado nos leads
  continua sendo o apelido original.
- Nomes reservados (nao pode usar): facebook, instagram, google, tiktok, linkedin, email, messenger,
  direct, booking, whatsapp, other, custom, manual, organic, referral.
- Criar e editar origem exige perfil de administrador.

**Marcar a origem de um contato a mao** tem duas regras que confundem:

- A **Ultima Origem e sempre sobrescrita**.
- A **1a Origem so e gravada se estiver vazia**. Para reescrever uma 1a Origem ja preenchida, e
  preciso pedir isso explicitamente na chamada.
- Marcar com valor vazio REMOVE a marcacao manual.
- Valores aceitos: facebook, instagram, google, tiktok, linkedin, email, messenger, direct, ou uma
  origem personalizada ATIVA da conta. **A personalizada nao vai pelo nome**: vai como
  `custom:` seguido do apelido tecnico dela (o campo `slug` que a listagem de origens devolve).
  Qualquer outro valor e recusado.

## 4. Campos de sistema (rastreamento) — existem, mas ficam escondidos

Origem, UTM, cliques de anuncio, dados vindos de integracao de pagamento e de agenda: sao campos
personalizados marcados como "de sistema".

- **Nao aparecem** na tela de Atributos Personalizados nem no seletor de variaveis.
- **Nao podem ser renomeados nem excluidos** (a plataforma recusa).
- **Funcionam normalmente em filtro e segmento.**

Consequencia direta: se voce listar os campos da conta sem pedir os de sistema, vai concluir que eles
nao existem e criar um campo duplicado. **Sempre peca a listagem incluindo os campos de sistema**
antes de propor a criacao de qualquer coisa parecida com origem, campanha ou anuncio.

## 5. Nomes proibidos ao criar campo

A criacao e recusada se a chave colidir com um campo nativo.

**Campo de CONTATO — proibido:** `name`, `email`, `phone_number`, `identifier`, `country_code`,
`city`, `created_at`, `last_activity_at`, `referer`, `blocked`.

**Campo de CONVERSA — proibido:** `status`, `priority`, `assignee_id`, `inbox_id`, `team_id`,
`display_id`, `campaign_id`, `labels`, `browser_language`, `country_code`, `referer`, `created_at`,
`last_activity_at`.

Repare que `status` e `labels` sao nomes muito naturais para um leigo pedir num campo de conversa — e
sao exatamente os que a plataforma recusa. Proponha outro nome ("situacao_do_orcamento",
"classificacao").

**Variavel da conta — proibido** tudo que a plataforma usa internamente para assinatura e cobranca
(as chaves que comecam com `subscription_`), alem de `plan_name`, `timezone`, `industry`,
`company_size`, `logo` e as chaves de estado interno da conta. Se a criacao for recusada, escolha
outro nome; nao tente contornar.

## 6. Etiquetas: as regras e as duas etiquetas diferentes

**A lista de etiquetas e uma so na conta** e serve para contatos E para conversas. Mas **a APLICACAO
e separada**: marcar a conversa nao marca a ficha, e vice-versa.

Regras do nome:

- **Sem espaco.** So letras, numeros, `_` e `-`.
- **Minimo 2 caracteres**, e nao pode comecar com `_` nem com `-`.
- **E sempre gravado em minusculas.** "Lead Quente" e recusado; use `lead-quente`.
- Alem do titulo, a etiqueta tem cor, descricao e a opcao de aparecer como atalho no menu lateral.

Duas armadilhas serias:

1. **A ferramenta de etiquetas do contato SUBSTITUI a lista inteira da ficha.** Mandar `["vip"]` numa
   ficha que tinha `cliente` e `inadimplente` deixa so `vip`. Para so ACRESCENTAR, ha dois caminhos:
   ler as etiquetas atuais da ficha e mandar a lista somada, ou usar a **acao em massa de contatos**,
   que so adiciona (ela nao tem remocao para contato).
2. **Etiqueta aplicada sem que exista na lista da conta fica invisivel.** A marca funciona no filtro,
   mas a etiqueta nao aparece na ficha, nem com cor, nem no seletor. **Crie a etiqueta primeiro**,
   sempre.

Onde a confusao entre as duas etiquetas mais custa caro:

| Onde | O que ele marca |
|---|---|
| Fluxo, bloco "Adicionar etiqueta no contato" | a FICHA |
| Fluxo, bloco "Adicionar etiqueta na conversa" | o ATENDIMENTO |
| Automacao, acao de etiqueta | o ATENDIMENTO |
| Campanha, publico "Tag do contato" | busca na FICHA |
| Campanha, publico "Tag da conversa" | busca no ATENDIMENTO |
| Filtro de contatos, chave `labels` | a FICHA |
| Filtro de contatos, chave `conversation_labels` | o ATENDIMENTO |

Se o fluxo etiqueta a conversa e a campanha procura tag do contato, **o publico sai vazio sem nenhum
aviso**. Ao montar a dupla "marca agora, dispara depois", confirme que os dois lados falam do mesmo
lugar.

## 7. Empresas

Agrupa contatos de uma mesma organizacao. Cada contato pertence a no maximo uma empresa.

- Campos: nome (ate 100 caracteres), dominio e descricao (ate 1000).
- **O dominio e o que faz a associacao automatica funcionar**: contato novo com e-mail corporativo
  daquele dominio e ligado sozinho a empresa. O dominio e unico na conta.
- **Empresa NAO aceita campo personalizado.** Se o cliente quiser guardar dados da empresa, ou vira
  campo do contato, ou vira descricao.
- **O modulo Empresas nasce DESLIGADO na conta** (e depende do plano). Cuidado: a ferramenta cria a
  empresa mesmo assim, e a tela de Empresas continua escondida do cliente — ou seja, voce cria e ele
  nao ve. **Antes de criar qualquer empresa, pergunte se ele consegue abrir a tela de Empresas no
  painel.** Se nao conseguir, nao crie: registre no resumo que o modulo precisa ser liberado.

## 8. Vinculo entre duas fichas

Liga duas fichas par a par: mae e filho, responsavel e dependente, socio e empresa.

- **Nao existe ferramenta para isso por aqui.** So pela aba **Vinculos** da ficha, no painel.
- E par a par, sem grupo e sem cascata: se A esta ligado a B e B a C, o B enxerga os dois, mas o A so
  enxerga o B.
- Aparece **so na aba Vinculos** — nao entra em busca, lista nem conversa.

**Nao trate vinculo como enfeite.** Ele e pre-requisito de uma funcionalidade real: agendar para
outra pessoa. Quando a consulta e da ficha do filho (que muitas vezes so tem o nome) e o aviso precisa
ir para o WhatsApp da mae, **o vinculo precisa existir ANTES** — sem ele o agendamento e recusado, e
o aviso so pode ir para quem estiver vinculado. Se o cliente descrever esse cenario, mande criar o
vinculo no painel antes de qualquer outra coisa.

## 9. Variaveis da conta

Guardam um valor unico da EMPRESA, nao da pessoa: endereco da loja, link do cardapio, telefone,
chave de integracao.

- Ficam em Configuracoes, **Variaveis da conta**. Exige perfil de administrador.
- Sao usadas em texto como `{{account.custom_attribute.chave}}` — **com o `custom_attribute` no
  meio**. Sem isso, sai em branco.
- Tipos que a tela oferece: Texto, Data, Hora e **Confidencial (criptografado)**.
- **Variavel Confidencial nunca aparece em mensagem** — resolve vazia de proposito, e o valor nunca e
  devolvido em leitura (so se sabe se existe valor ou nao). Ela serve para chamada de API dentro do
  Fluxo. Nao proponha Confidencial para nada que va aparecer numa conversa.

## 10. Notas, Documentos e Preenchimentos

Tres abas da ficha que resolvem pedidos que o cliente as vezes tenta resolver criando campo:

- **Notas** — anotacao interna livre, com autor e data. Nao aparece para o cliente final. Se ele
  quer "um espaco para escrever observacoes", e nota, nao campo personalizado.
- **Documentos** — arquivos anexados a ficha (contrato, RG digitalizado, comprovante). Pode ser
  arquivo novo ou um espelho de um anexo que ja existe num card ou numa conversa, sem duplicar.
- **Preenchimentos** — tudo que a pessoa preencheu (formulario publico, lead do Meta, webhook) numa
  lista so. Antes de criar campo para guardar resposta de formulario, veja se ja nao esta aqui.

## 11. Ficha duplicada e mesclagem

A duplicata mais comum no Brasil: **o mesmo celular vira duas fichas quando uma tem o nono digito e
a outra nao.** A busca ja procura nas duas formas, mas os pares que ja existem continuam separados.

Juntar duas fichas e **irreversivel**. Regras:

- As duas precisam ser da mesma conta. Escolhe-se uma ficha BASE (que fica) e uma que e absorvida.
- Vao para a base: conversas, mensagens, enderecos de canal, notas, preenchimentos, vinculos,
  etiquetas, foto, e os campos que estiverem VAZIOS na base.
- Depois de juntar, por caixa de entrada, a conversa mais recente fica aberta e as outras sao
  encerradas.

**Nunca junte por conta propria.** Liste as suspeitas ao cliente — mesmo telefone em formas
diferentes, mesmo e-mail, mesmo nome — e deixe ele decidir par a par.

E o outro lado: **apagar um contato apaga junto todas as conversas dele.** A acao em massa com
"todos os contatos do filtro" mais exclusao e a operacao mais destrutiva da plataforma. A skill nao
apaga; mande o cliente fazer na mao, se ele quiser mesmo.
