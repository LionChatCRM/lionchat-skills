# Importar uma lista de contatos

Leia quando o cliente falar em planilha, lista, base antiga ou importacao.

## Indice

1. O que voce PODE e o que NAO PODE fazer daqui
2. Antes de mandar o cliente importar: a lista de conferencia
3. As colunas que a importacao entende
4. O formato de cada tipo de campo
5. A regra do telefone (a que mais recusa linha)
6. Como o sistema decide se cria ou atualiza
7. O que a importacao SOBRESCREVE
8. Casar os valores de um campo de Lista
9. A conferencia previa: o que ela testa e o que ela NAO testa
10. Ler o resultado da importacao
11. O caso "subi a planilha e ja quero disparar"
12. Criar a lista pelas ferramentas, sem arquivo

---

## 1. O que voce PODE e o que NAO PODE fazer daqui

**NAO da para subir um arquivo por aqui.** As ferramentas de importacao aparecem no catalogo, mas
todas exigem envio de arquivo, e isso nao passa por esta conexao. **Nunca prometa "vou importar sua
planilha"** — voce vai ficar sem entregar.

Os dois caminhos que funcionam de verdade:

- **O cliente importa na tela**, em Contatos, e voce prepara tudo antes (os campos de destino, as
  etiquetas, o formato do telefone) e le o resultado depois com ele.
- **Voce cria a lista pela ferramenta de criacao em lote** (ate 1000 contatos por chamada). Ela existe
  so no conector remoto e tem limitacoes — ver secao 12.

Importar, conferir o arquivo e exportar exigem **perfil de administrador** (ou um cargo
personalizado com a permissao de importar e exportar contatos).

## 2. Antes de mandar o cliente importar: a lista de conferencia

Faca isso ANTES, sempre. Quase todo desastre de importacao e um destes cinco:

1. **Os campos de destino existem?** Coluna apontada para um campo que nao existe e **descartada em
   silencio**: sem erro, sem contador. O cliente importa 5.000 contatos e descobre semanas depois que
   o campo mais importante esta vazio em todos. Crie os campos primeiro.
2. **O telefone tem o codigo do pais?** Sem ele a linha inteira e recusada. Peca uma amostra de 3
   linhas ao cliente e confira com os proprios olhos.
3. **O formato do arquivo serve?** A tela aceita `.csv`, `.xlsx` e `.xls` — a planilha do Excel e
   convertida para CSV no proprio navegador antes de subir. So a PRIMEIRA aba do arquivo e lida.
4. **A coluna de telefone esta formatada como texto?** Numero longo no Excel vira notacao cientifica
   (`9,88888E+12`) e a linha e recusada com erro proprio.
5. **Se houver campo de Lista**, os valores da planilha batem com as opcoes cadastradas? Valor fora
   da lista deixa o campo em branco.

Diga ao cliente que existe um **modelo de exemplo para baixar** na propria tela de importacao, ja com
uma coluna de cada tipo preenchida no formato certo.

## 3. As colunas que a importacao entende

A primeira linha do arquivo e o cabecalho. Os nomes das colunas podem ser quaisquer: na tela o
cliente faz o de-para, dizendo qual coluna vai para qual campo.

**Destinos nativos:** nome, e-mail, telefone, empresa, cidade, etiquetas, e os perfis de rede social
(Instagram, Facebook, LinkedIn, Twitter).

**Destinos personalizados:** qualquer campo personalizado de CONTATO que a conta ja tenha.

**Destinos cadastrais:** CPF, RG, CNPJ, passaporte, data de nascimento, genero, estado civil,
profissao e o endereco completo (CEP, rua, numero, complemento, bairro, cidade, UF, pais).

Duas ausencias que valem registrar:

- **Nao ha coluna de identificador** na importacao. Se o cliente precisa trazer o codigo do outro
  sistema, isso e feito contato a contato pelas ferramentas, nao pela planilha.
- **Campo de CONVERSA nao entra na importacao.** A planilha preenche a ficha, nao o atendimento.

Teto: **100.000 linhas por importacao**.

## 4. O formato de cada tipo de campo

A importacao respeita o tipo do campo de destino. Valor que nao bate com o tipo **nao derruba a
linha**: o contato entra e so aquele campo fica em branco, contado a parte no resultado.

| Tipo do campo | Como preencher | Exemplos que passam | O que da problema |
|---|---|---|---|
| Texto | qualquer texto | `Cliente desde 2020` | (sempre aceito) |
| Numero, Moeda, Porcentagem | so numeros | `42`, `1.234,56` (Brasil), `1,234.56` (EUA) | `abc`, `vinte` |
| Data | dia/mes/ano | `25/12/1990`, `25-12-1990` | `32/13/2025`, `ontem` |
| Hora | HH:MM em 24 horas | `14:30`, `9:30`, `14h30` | `25:00`, `2 da tarde` |
| Lista | uma das opcoes cadastradas | `ouro`, se a lista tiver "Ouro" | valor que nao esta na lista |
| Caixa de selecao | sim ou nao | `sim`, `s`, `yes`, `1`, `x`, `v`, `verdadeiro` / `nao`, `n`, `no`, `0`, `falso` | `talvez` |

Detalhes que evitam retrabalho:

- **Campo do tipo Data e Hora fica de fora**: a importacao nao converte esse tipo e a conferencia
  previa nao o testa — o texto entra cru e provavelmente inutil. Para planilha, prefira dois campos
  separados, um de Data e outro de Hora.
- **Data e sempre no padrao brasileiro, dia primeiro.**
- **Numero aceita a virgula decimal brasileira** com ponto de milhar.
- **Lista casa primeiro pelo valor exato** (sem diferenciar maiuscula) e, se nao achar, tenta de novo
  ignorando acento. Ainda assim, opcao que nao existe na lista fica em branco.
- **Genero** na planilha: `m`, `f`, `o`, `na`. **Estado civil**: `solteiro`, `casado`,
  `uniao_estavel`, `divorciado`, `separado`, `viuvo`.
- **CPF, CNPJ e CEP** podem vir com ou sem pontuacao.

## 5. A regra do telefone (a que mais recusa linha)

**O telefone precisa vir com o codigo do pais.** O sistema NAO completa o 55 e NAO adivinha o pais.

- Aceito: `5511988887777`, `+5511988887777`, `+351912345678`. O sinal de `+` e opcional.
- **Recusado**: `11 99999-8888` (sem o codigo do pais), numero comecando com 0, e numero em notacao
  cientifica.
- **Telefone preenchido e invalido recusa a LINHA INTEIRA** — o contato nao e criado. Isso e de
  proposito: contato mudo na base, sem como receber mensagem, e pior do que contato nao criado.
- **Coluna de telefone VAZIA continua criando o contato.** Lista de nome mais e-mail e caso legitimo.
- Numero brasileiro com o 55 e aceito mesmo no formato antigo, sem o nono digito.

A importacao **nao consulta o WhatsApp** para saber se o numero existe — de proposito, para nao
arriscar a conexao do cliente com milhares de consultas. Ou seja: planilha nunca marca ninguem como
"nao tem WhatsApp". Essa conferencia acontece depois, quando alguem abre a conversa ou manda a
primeira mensagem.

## 6. Como o sistema decide se cria ou atualiza

Para cada linha, na ordem: procura por **e-mail**; se nao achar, procura por **telefone**, **nas duas
formas do nono digito** (com e sem). Achou, atualiza a ficha existente. Nao achou, cria.

Detalhe importante: quando a ficha e encontrada pela outra forma do nono digito, **o telefone gravado
nao e trocado**. O que esta na ficha e o numero que funciona no WhatsApp ha anos; o da planilha e so
a outra grafia. Trocar seria estragar a base inteira de uma vez.

## 7. O que a importacao SOBRESCREVE

Esta e a diferenca que mais causa estrago em reimportacao:

| Campo | Comportamento |
|---|---|
| **Nome** | **SEMPRE gravado por cima**, sem excecao |
| E-mail | so grava se vier preenchido na planilha |
| Empresa, cidade | so gravam se vierem preenchidos |
| Campos personalizados | so gravam se vierem preenchidos |
| Dados cadastrais | so gravam se vierem preenchidos, e os imutaveis so na primeira vez |
| Etiquetas | **ACRESCENTAM**, nao substituem |

Consequencia real: reimportar a mesma lista com o nome pior ("MARIA S.", so o primeiro nome, tudo em
caixa alta) **troca o nome de toda a base de uma vez**. Avise o cliente antes de qualquer
reimportacao.

Duas regras a mais:

- **Nome em branco recusa a linha** na importacao normal ("Nome e obrigatorio").
- **E-mail que ja pertence a outro contato** nao quebra a linha: o contato entra sem o e-mail, com
  aviso no resultado.

Sobre as etiquetas da planilha: a coluna aceita varias separadas por virgula, o sistema **baixa para
minusculas e troca espaco por hifen** ("Lead Quente" vira `lead-quente`) e **cria a etiqueta na
conta se ela ainda nao existir** — o resultado da importacao conta quantas foram criadas.

## 8. Casar os valores de um campo de Lista

Quando uma coluna aponta para um campo de Lista, a tela mostra um quadro de correspondencia: para
cada valor diferente encontrado na coluna, o cliente escolhe qual opcao da lista ele representa, ou
manda ignorar (o campo fica em branco).

- Valores identicos a uma opcao ja vem marcados sozinhos.
- Acima de 50 valores diferentes, o quadro mostra os 50 primeiros; o resto segue a regra automatica.
- **Ignorar um valor deixa o campo em branco sem aviso nenhum.** Se o cliente marcar muita coisa como
  ignorar, ele vai achar depois que "o campo nao importou".

Caso real que vale contar ao cliente: uma integracao gravava o nome de uma cidade sem acento num
campo cujas opcoes tinham acento, e 1.315 contatos ficaram com o campo aparentemente vazio na tela e
sumiram do segmento por unidade. Hoje a comparacao tolera acento — mas continua nao tolerando opcao
que nao existe.

## 9. A conferencia previa: o que ela testa e o que ela NAO testa

Na tela ha o botao **"Conferir arquivo"**, que le a planilha inteira antes de importar e aponta linha
a linha o que nao bate com o tipo do campo (`Linha 23, Aniversario: '32/13/2025' nao e uma data
valida`).

**Ela testa o formato dos campos. Ela NAO testa telefone.** Passar verde ali nao garante importacao
sem erro de codigo do pais — esses erros so aparecem no resultado final. Diga isso ao cliente, senao
ele vai confiar no verde.

Depois da conferencia da para voltar e corrigir a planilha, ou importar mesmo assim: importando, o
contato entra e so o campo com problema fica em branco. **Ninguem perde contato por campo errado.**

## 10. Ler o resultado da importacao

O resultado traz contagens separadas. Traduza cada uma para o cliente:

| O que aparece | O que significa |
|---|---|
| criados | fichas novas |
| atualizados | fichas que ja existiam e foram atualizadas |
| erros | linhas recusadas — cada uma com o numero da linha e o motivo |
| telefones invalidos | quantas linhas cairam pela regra do codigo do pais |
| campos ignorados | valores que nao bateram com o tipo; o contato entrou, o campo ficou em branco |
| etiquetas criadas | etiquetas novas que a planilha criou |
| linhas em branco | linhas 100% vazias, descartadas |

**A lista detalhada de erros tem teto de 50 linhas** na importacao normal. Se der mais que isso, o
contador continua certo, mas nem todo erro aparece na lista.

**As linhas em branco confundem a contagem.** O Excel deixa centenas de linhas vazias no fim do
arquivo; elas sao descartadas e contadas em separado. Uma planilha de 300 linhas com 2 preenchidas
mostra "2 conferidas" — sem explicar isso, o cliente acha que perdeu 298 contatos.

## 11. O caso "subi a planilha e ja quero disparar"

Se o pedido do cliente e disparar para a lista que ele acabou de subir, **a importacao normal nao e o
caminho**. Dentro do formulario da campanha existe a opcao de **subir a planilha ali mesmo**
("Lista de planilha"), e ela tem regras opostas:

- **Telefone e obrigatorio**; o nome e opcional (e ha um texto reserva para quem estiver sem nome).
- **Ficha que ja existe e preservada** — nada e sobrescrito.
- **E-mail que ja pertence a outro contato** e descartado com aviso, em vez de quebrar a linha.
- Cada pessoa da lista recebe uma marca automatica daquele disparo, e opcionalmente uma etiqueta.

Ou seja: para "importar minha base" use a tela de Contatos; para "disparar para esta lista" use a
planilha de dentro da campanha. Escolher errado faz o cliente cacar depois como selecionar aquele
lote.

## 12. Criar a lista pelas ferramentas, sem arquivo

Quando a lista for pequena e o cliente puder passar os dados no chat, da para criar sem arquivo:

- **Um contato por vez**: a ferramenta de criar contato aceita nome, e-mail, telefone, identificador
  e os campos personalizados. Nao aceita etiqueta nem dados cadastrais — esses tem ferramentas
  proprias.
- **Ate 1000 de uma vez**: a ferramenta de criacao em lote monta a lista internamente e entrega para
  a importacao nativa (mesma deduplicacao por telefone e e-mail). Ela existe **so no conector
  remoto** — confirme que esta disponivel antes de prometer.

Limitacoes da criacao em lote que precisam ser ditas:

- **Cada campo personalizado precisa JA EXISTIR** como campo de contato, senao o valor e descartado
  em silencio (mesma regra da planilha).
- **Nao existe identificador** nesse caminho.
- **Roda em segundo plano.** Ela responde antes de terminar, entao confirme depois buscando alguns
  contatos e conferindo se telefone, campos e etiquetas chegaram. Nao declare sucesso sem essa
  conferencia.
