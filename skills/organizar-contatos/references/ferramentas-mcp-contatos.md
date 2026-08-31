# Qual ferramenta faz o que, nesta area

Leia quando precisar escolher a ferramenta certa. Varios nomes desta area nao dizem o que a
ferramenta faz — a lista abaixo existe por causa disso.

Os nomes aparecem aqui sem o prefixo do conector. Na sua sessao eles vem com o prefixo do servidor
LionChat que estiver ligado; use o nome exato como ele aparece na sua lista de ferramentas.

## Indice

1. Antes de tudo: os dois conectores nao sao iguais
2. A ficha do contato
3. Etiquetas do contato
4. Campos personalizados (as definicoes)
5. Etiquetas da conta, segmentos, empresas e origens
6. Variaveis da conta e respostas prontas
7. Acao em massa de contatos
8. As abas da ficha: notas, documentos, preenchimentos, cards, conversas
9. Ferramentas que NAO funcionam por aqui
10. Ferramenta que responde vazio de proposito
11. O que so existe no painel

---

## 1. Antes de tudo: os dois conectores nao sao iguais

Existem dois conectores do LionChat, e eles NAO expoem o mesmo conjunto de ferramentas. Uma
ferramenta que existe num pode nao existir no outro.

Diferencas que afetam esta area:

- **Criacao de contatos em lote** (`lionchat_contacts_bulk_create`) existe **so no conector remoto**.
- **Trocar de conta e ver a conta ativa** (`lionchat_switch_account`, `lionchat_current_account`,
  `lionchat_list_my_accounts`) existem **so no conector remoto**.
- **Testar a conexao** (`lionchat_ping`) existe **so no conector local**.

**Nunca cite uma ferramenta sem antes ver que ela existe na sua lista.** Se nao existir, descreva a
acao em palavras e diga ao cliente que aquilo se faz no painel.

## 2. A ficha do contato

| Ferramenta | O que faz |
|---|---|
| `lionchat_contacts_create` | Cria UMA ficha. Aceita nome, e-mail, telefone, identificador e campos personalizados. **Nao aceita etiqueta nem dados cadastrais** — cada um tem ferramenta propria |
| `lionchat_contacts_show` | Le uma ficha |
| `lionchat_contacts_list` | Lista contatos. Da para ordenar por nome, e-mail, telefone, ultima atividade, criacao, empresa, cidade e pais, e filtrar rapido por etiqueta |
| `lionchat_contacts_search` | Busca por texto, ignorando acento e maiuscula, em **nome, e-mail, telefone e identificador** — e so nesses quatro. Um termo por vez: espaco nao separa termos aqui, e ela NAO acha por CPF, CNPJ nem RG |
| `lionchat_search_list_1` | A busca AMPLA de contatos (a mesma da lupa do painel). Alem dos quatro campos acima, alcanca os campos personalizados e os documentos (CPF, CNPJ, RG por digitos). Espaco entre termos funciona como "e", ate 5 termos; termo com letra exige 3 caracteres, so digitos nao. **Use esta quando o cliente buscar por documento** |
| `lionchat_contacts_filter` | O recorte avancado, 15 por pagina. Mesma linguagem do segmento salvo — ver `tipos-de-campo-e-filtros.md` |
| `lionchat_contacts_update` | Edita a ficha. **Nunca sobrescreva telefone, e-mail ou nome ja preenchido sem confirmar** |
| `lionchat_contacts_update_cadastral` | Grava CPF, RG, CNPJ, passaporte, nascimento, genero, estado civil, profissao e endereco. Os seis primeiros so aceitam a PRIMEIRA gravacao |
| `lionchat_cep_lookup` | Busca rua, bairro, cidade e UF por CEP, para preencher o endereco sem digitar |
| `lionchat_contacts_create_6` | Apaga chaves especificas dos campos personalizados de uma ficha |
| `lionchat_contacts_create_2` | Cria o endereco do contato numa caixa de entrada — necessario para abrir conversa com quem nunca escreveu |
| `lionchat_contacts_list_3` | Mostra em quais caixas o contato pode ser contatado |
| `lionchat_contact_inboxes_filter` | Descobre qual contato tem um determinado endereco numa caixa. **Busca exata** — no WhatsApp por QR o mesmo contato pode ter tres formatos de endereco |
| `lionchat_contacts_create_5` | Exporta contatos em arquivo (roda em segundo plano). Por padrao exporta tudo: dados cadastrais, etiquetas, empresa, cidade, pais, campos de contato e de conversa, ultima conversa e card. Respeita os filtros enviados |
| `lionchat_contacts_set_origin` | Marca a mao de onde veio o lead — ver as regras em `campos-nativos-e-etiquetas.md` |
| `lionchat_contacts_destroy_2` | Remove so a foto da ficha |

Duas de risco, com regra propria:

| Ferramenta | Cuidado |
|---|---|
| `lionchat_contacts_create_4` | **MESCLA duas fichas duplicadas. Irreversivel.** Nunca use sem o cliente escolher qual ficha fica |
| `lionchat_contacts_destroy` | **Apaga o contato PARA SEMPRE e apaga junto as conversas dele.** A skill nao usa esta ferramenta |

## 3. Etiquetas do contato

Duas ferramentas, e a diferenca entre elas e a armadilha mais cara desta area:

| Ferramenta | O que faz de verdade |
|---|---|
| `lionchat_contacts_list_4` | Le as etiquetas atuais da ficha |
| `lionchat_contacts_create_3` | **SUBSTITUI a lista inteira** de etiquetas da ficha pelo que voce mandar |

Para **acrescentar** uma etiqueta sem apagar as outras, ha dois caminhos:

1. ler com `lionchat_contacts_list_4`, somar a nova, e mandar a lista completa em
   `lionchat_contacts_create_3`; ou
2. usar a acao em massa (secao 7) com um unico contato — ela **so adiciona**, nunca remove.

Em qualquer um dos dois: **a etiqueta precisa existir na conta antes**, senao a marca funciona no
filtro mas nao aparece no painel.

## 4. Campos personalizados (as definicoes)

| Ferramenta | O que faz |
|---|---|
| `lionchat_custom_attributes_list` | Lista as definicoes. **Peca tambem os campos de sistema** quando quiser ver origem, UTM e rastreamento — sem isso eles somem da lista |
| `lionchat_custom_attributes_show` | Le uma definicao |
| `lionchat_custom_attributes_create` | Cria. **O padrao e campo de CONVERSA** — para campo de contato mande o modelo explicitamente |
| `lionchat_custom_attributes_update` | Edita nome, descricao e opcoes. **Nao renomeie a chave de um campo com dado gravado**: o dado nao vem junto |
| `lionchat_custom_attributes_destroy` | Exclui a definicao. A skill nao exclui nada |

Todas exigem perfil de administrador. Campo marcado como de sistema nao pode ser renomeado nem
excluido: a plataforma recusa.

## 5. Etiquetas da conta, segmentos, empresas e origens

| Ferramenta | O que faz |
|---|---|
| `lionchat_labels_list` / `_show` / `_create` / `_update` / `_destroy` | A lista de etiquetas da conta: titulo (sem espaco), cor, descricao e se aparece no menu lateral |
| `lionchat_custom_filters_list` / `_show` / `_create` / `_update` / `_destroy` | Segmentos (filtros salvos). Use tipo `contact`. As condicoes vao **dentro de `query`, embrulhadas em `payload`** — ver `tipos-de-campo-e-filtros.md`, secao 8; embrulho errado cria um segmento que nao filtra nada, sem erro. **Cada pessoa so enxerga os proprios segmentos** |
| `lionchat_companies_list` / `_show` / `_search` / `_create` / `_update` / `_destroy` | Empresas. Nao aceitam campo personalizado. O modulo nasce desligado na conta |
| `lionchat_lead_origins_list` / `_create` / `_update` / `_destroy` | Origens personalizadas. Excluir apenas desativa; criar com nome existente reativa |
| `lionchat_lead_origin_reports` | O relatorio de Origem dos Leads: de onde vieram, com taxa de conversao |
| `lionchat_liontrack_journey_stages_*` | Regras que classificam paginas do site em fases da jornada. So funciona se o modulo estiver ligado na conta |
| `lionchat_journey_funnel_reports` | O mapa de navegacao dos leads no site. Janela maxima de 30 dias |

## 6. Variaveis da conta e respostas prontas

| Ferramenta | O que faz |
|---|---|
| `lionchat_account_variables_list` / `_show` / `_create` / `_update` / `_destroy` | Variaveis da empresa. Exige administrador. O valor Confidencial nunca e devolvido na leitura |
| `lionchat_canned_responses_list` / `_create` / `_update` / `_destroy` | Respostas Prontas, inclusive as de varios blocos com midia, pausa entre blocos e botoes |
| `lionchat_captain_liquid_variables_list` | **Lista as variaveis de contato, conversa e conta que existem de verdade** naquela conta (e a lista dos textos do AI Agente). Exige administrador. Consulte ANTES de escrever texto com variavel: variavel que nao existe nao da erro, sai em branco. Ela NAO cobre toda tela do produto — cada tela tem seu proprio seletor, entao confirme na tela em que o texto vai viver |

## 7. Acao em massa de contatos

A ferramenta se chama `lionchat_kanban_bulk_bulk_actions` — **o nome esconde o que ela faz**. E ela
que aplica etiqueta em lote e exclui em lote CONTATOS (e tambem serve para conversas).

Regras que fazem a chamada falhar ou nao fazer nada:

- **O tipo do alvo vai no SINGULAR: `Contact`.** No plural (`Contacts`) e recusado e a acao inteira
  falha. Escreva sempre `Contact`, com inicial maiuscula.
- **Sem o campo de etiquetas, a chamada responde sucesso e NAO FAZ NADA.** E o erro silencioso mais
  comum: "coloquei a etiqueta em 40 contatos" e nada aconteceu.
- Para contato, a etiqueta so tem **adicionar** — nao existe remover em lote.
- Teto de **300 fichas por chamada**.
- Existe a opcao de aplicar a acao a **todos os contatos de um filtro**, sem teto. Nesse modo nao se
  mandam as fichas, manda-se o filtro (ou uma lista de etiquetas). **Se nao mandar nem um nem outro,
  a acao atinge TODOS os contatos da conta.**
- A opcao de exclusao combinada com "todos do filtro" **e a operacao mais destrutiva da plataforma**.
  A skill nao a usa.
- A acao roda em segundo plano: ela responde antes de terminar. Confira o resultado depois.

## 8. As abas da ficha: notas, documentos, preenchimentos, cards, conversas

| Ferramenta | Aba |
|---|---|
| `lionchat_contacts_create_1` / `lionchat_contacts_list_2` / `lionchat_contacts_notes_show` / `lionchat_contacts_update_1` / `lionchat_contacts_destroy_1` | Notas internas do contato |
| `lionchat_contacts_documents_list` / `_create` / `_update` / `_destroy` / `_mirror` / `_destroy_mirror` | Documentos. O espelho aponta para um anexo que ja existe num card ou numa conversa, sem duplicar o arquivo |
| `lionchat_contacts_form_entries_list` / `_show` | Preenchimentos: tudo que a pessoa preencheu, numa lista so |
| `lionchat_contacts_kanban_items_list` | Os cards ligados as conversas desse contato |
| `lionchat_contacts_list_1` | As conversas do contato — o historico de atendimento |
| `lionchat_contacts_tracking_events_list` / `_presence` / `_debug_timeline` | Navegacao do lead no site |

## 9. Ferramentas que NAO funcionam por aqui

Aparecem no catalogo e **falham**, porque exigem envio de arquivo:

- `lionchat_contacts_import`
- `lionchat_contacts_import_validate`
- `lionchat_contacts_create_7` (previa da importacao)
- `lionchat_contacts_create_8` (importacao pelo assistente de planilha)

**Nunca prometa importar a planilha do cliente.** Alternativas: `lionchat_contacts_bulk_create` (ate
1000, so no conector remoto) ou mandar o cliente usar a tela de Contatos.

## 10. Ferramenta que responde vazio de proposito

`lionchat_contacts_active` esta **desligada na plataforma**: responde sempre lista vazia, mesmo com
gente navegando. **Nao use para descobrir quem esta online** e nao interprete o vazio como "nenhum
contato ativo". Para achar contato, use a listagem, a busca ou o filtro.

## 11. O que so existe no painel

Nao ha ferramenta para nada disto — mande o cliente fazer na tela e registre no resumo final:

- **Importar arquivo de planilha** (Contatos, botao de importar).
- **Criar, ver e desfazer vinculo entre duas fichas** (aba Vinculos da ficha).
- **Criar campo de CARD do Kanban** — a ferramenta de campos personalizados so faz contato e
  conversa; campo de card mora na configuracao do Kanban.
- **Marcar quais campos de conversa sao obrigatorios para encerrar um atendimento** — fica nas
  configuracoes da conta, e vale a pena avisar o efeito: com isso ligado, o atendente **nao consegue
  encerrar** a conversa sem preencher. E, na pratica, o motivo numero um para existir campo de
  conversa. So esta disponivel nos planos que incluem esse recurso.
- **Baixar o modelo de planilha de exemplo** (so na tela de importacao).
- **Escolher qual campo personalizado entra no formulario do chat do site** (configuracao da caixa).
- **Renomear a chave de um campo movendo os dados ja gravados** — nem a tela nem as ferramentas fazem
  isso. Se precisar, crie um campo novo.
