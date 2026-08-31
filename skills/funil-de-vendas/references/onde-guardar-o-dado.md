# Onde guardar cada informacao

Este e o erro de montagem mais comum e o mais caro: escolher o lugar errado para guardar uma informacao. A escolha e praticamente definitiva - mover o dado de um escopo para outro depois exige mutirao manual, cartao por cartao.

## Indice

1. As quatro caixas e a pergunta que separa cada uma
2. Arvore de decisao
3. Etiqueta
4. Campo do contato
5. Campo da conversa
6. Campo do cartao
7. O que ja e nativo e NAO deve virar campo
8. Exemplos resolvidos
9. Como criar cada um
10. A armadilha do campo do cartao gravado no lugar errado

---

## 1. As quatro caixas e a pergunta que separa cada uma

| Caixa | A pergunta que ela responde | Vive em | Muda quando |
|---|---|---|---|
| **Etiqueta** | "Isso e verdade ou nao?" (sem valor) | na conversa | o atendente ou uma automacao coloca e tira |
| **Campo do contato** | "Como e essa PESSOA?" | na ficha da pessoa | a pessoa muda de endereco, de plano, de empresa |
| **Campo da conversa** | "Do que se tratava ESTE atendimento?" | naquela conversa | some quando a pessoa abre outra conversa |
| **Campo do cartao** | "Como e ESTE negocio?" | no cartao do Kanban | a mesma pessoa pode ter dois cartoes com valores diferentes |

## 2. Arvore de decisao

Faca as perguntas nesta ordem. Pare na primeira que responder sim.

1. **Isso ja existe pronto no sistema?** (motivo de ganho, motivo de perda, valor, prioridade, prazo, responsavel, origem do trafego) -> use o nativo. Ver a secao 7.
2. **E so um sim ou nao usado para separar e filtrar conversas, sem guardar valor?** -> **etiqueta**.
3. **Se essa pessoa comprar de novo daqui a um ano, o valor continua o mesmo?** -> **campo do contato**.
4. **O valor pertence a um NEGOCIO especifico, e a mesma pessoa poderia ter dois negocios com valores diferentes ao mesmo tempo?** -> **campo do cartao**.
5. **O valor so faz sentido dentro daquele atendimento e nao precisa sobreviver a ele?** -> **campo da conversa**.

## 3. Etiqueta

Marca simples, sem valor: ou esta la, ou nao esta. Aplicada na conversa e visivel na ficha da pessoa. Filtra conversa, dispara automacao, e aparece no filtro do quadro do Kanban atraves das conversas ligadas ao cartao.

- Titulo unico na conta, com cor. **O sistema poe tudo em minusculas sozinho e NAO aceita espaco**:
  use hifen ou sublinhado (`cliente-antigo`, nao "Cliente Antigo"). Acento e permitido; comecar por
  hifen ou sublinhado, nao.
- Boa para: origem do lead (`fonte-instagram`, `fonte-indicacao`), objecao (`objecao-preco`), estado de triagem (`aguardando-documentos`).
- Ruim para: qualquer coisa que tenha um valor - "orcamento de R$ 8.000" nao e etiqueta, e campo.
- Cuidado com a explosao: quinze etiquetas de estado que so um vendedor entende poluem a lista de todo mundo.

Ferramentas: `lionchat_labels_list`, `lionchat_labels_create`.

## 4. Campo do contato

Dado da PESSOA, que atravessa qualquer negocio: CPF, endereco, data de nascimento, empresa, plano de saude, profissao.

Tipos aceitos: texto, numero, moeda, porcentagem, link, data, lista de opcoes, caixa de marcar, campo secreto, hora, data e hora.

Ferramentas: `lionchat_custom_attributes_list`, `lionchat_custom_attributes_create` com o modelo de contato.

**Regra dura: o agente de IA nunca sobrescreve o telefone que ja esta gravado no contato.** Nao proponha campo nem automacao que dependa disso.

## 5. Campo da conversa

Dado daquele ATENDIMENTO: motivo do contato, protocolo, numero do pedido tratado naquela conversa, canal por onde a pessoa chegou naquele dia.

Mesmos tipos do campo do contato. Ferramentas: as mesmas, com o modelo de conversa.

**Cuidado:** ele nao acompanha a pessoa. Se ela abrir uma conversa nova amanha, o campo volta vazio. Se o dado precisa ser encontrado depois, ele nao pertence aqui.

**Nunca grave chave de sistema nos dados extras da conversa** a partir de um mapeamento de integracao - existe uma lista de nomes reservados justamente porque um deles tira a conversa das telas, sem nenhum erro.

## 6. Campo do cartao

Dado do NEGOCIO: condicao de pagamento, numero da proposta, unidade que vai atender, produto de interesse daquele ciclo.

- A definicao (nome, tipo, se e lista, quais opcoes) e **por CONTA** e vale para TODOS os funis. Nao existe campo de cartao so em um funil.
- Tipos que a tela oferece: texto, numero, data, sim/nao. A tela tambem exibe hora. Nao existe o tipo "lista": lista e um MODO (`is_list` ligado, com as opcoes em `list_values`).
- O valor preenchido mora dentro do cartao, em `item_details.custom_attributes`, num formato de LISTA de itens `{name, type, value}` - nao um par chave e valor. Ver a secao 10.

Ferramentas: ler com `lionchat_kanban_config_list`, gravar a definicao com `lionchat_kanban_config_update` (a lista `global_custom_attributes`, sempre completa). O valor de um cartao especifico entra pelo `lionchat_kanban_items_update`.

## 7. O que ja e nativo e NAO deve virar campo

Criar campo para qualquer um destes quebra uma tela ou um relatorio que ja funciona:

| A pessoa pede | Ja existe como | O que quebra se virar campo |
|---|---|---|
| "motivo da perda" | motivos de perda da conta | o vendedor nao ve a lista na janela ao marcar Perdido, e o relatorio de motivos fica vazio |
| "motivo do ganho" | motivos de ganho da conta | idem |
| "lista de tarefas padrao" | modelos de checklist da conta | a automacao de checklist nao acha o modelo |
| "valor do negocio" | valor do cartao (soma das ofertas) | some do total da coluna, da previsao e da receita |
| "urgencia" | prioridade do cartao | some do filtro e do relatorio de prioridade |
| "prazo" | prazo do cartao | some do aviso de cor no cartao e do filtro de agendamento |
| "vendedor" | responsavel do cartao | some do ranking de vendedores e das regras de quem ve o cartao |
| "de onde veio o lead" (anuncio, campanha, origem do trafego) | o proprio sistema ja copia isso para dentro do cartao quando o cartao nasce | voce duplica um dado que ja e coletado sozinho, e a sua copia nao acompanha os relatorios de origem |

Sobre a origem do trafego: e uma FOTO tirada da conversa no momento em que o cartao nasce, e e so leitura - nao da para escrever. Cartao duplicado mantem a foto do original, e cartao criado sem conversa (importacao sem telefone valido) nao herda nada.

## 8. Exemplos resolvidos

| A pessoa diz | Vai para | Por que |
|---|---|---|
| "preciso saber o CPF" | campo do contato | e da pessoa, vale para sempre |
| "preciso saber se e cliente antigo" | etiqueta | e sim ou nao, sem valor |
| "preciso saber de onde ele veio" | etiqueta de origem (ou o que o sistema ja copia) | separacao simples |
| "preciso saber a forma de pagamento deste orcamento" | campo do cartao | pode ser diferente em cada negocio da mesma pessoa |
| "preciso saber o numero da proposta" | campo do cartao | e do negocio |
| "preciso saber o protocolo deste atendimento" | campo da conversa | nasce e morre naquele atendimento |
| "preciso saber por que ele nao comprou" | motivo de perda (nativo) | ja existe |
| "preciso saber quanto ele vai pagar" | valor do cartao via oferta | ja existe |
| "preciso saber a unidade que vai atender" | campo do cartao se muda por negocio; campo do contato se a pessoa e sempre da mesma unidade | depende da pergunta 3 da arvore |
| "preciso saber quantas parcelas" | campo do cartao | e do negocio |
| "preciso saber o aniversario" | campo do contato | e da pessoa |

## 9. Como criar cada um

| Caixa | Ferramenta | Detalhe que evita retrabalho |
|---|---|---|
| Etiqueta | `lionchat_labels_create` | liste antes com `lionchat_labels_list` e compare por titulo |
| Campo do contato | `lionchat_custom_attributes_create` | escolha o modelo de contato; a chave e em letras minusculas com sublinhado |
| Campo da conversa | `lionchat_custom_attributes_create` | e o modelo padrao quando nada e informado |
| Campo do cartao | `lionchat_kanban_config_update` | leia antes e mande a lista completa; a lista enviada substitui a anterior inteira |

Criar campo personalizado exige perfil de administrador.

## 10. A armadilha do campo do cartao gravado no lugar errado

O cartao tem DOIS lugares com nome parecido:

- `custom_attributes` solto no cartao - existe, aceita qualquer coisa, e **nenhuma tela le**.
- `item_details.custom_attributes` - **e este** que a aba Dados Adicionais mostra e que o filtro por campo do cartao procura.

Gravar no primeiro nao da erro nenhum. O dado fica salvo, invisivel e nao filtravel - e a conclusao da pessoa e "esse campo nao funciona".

O formato do segundo tambem engana: e uma LISTA de itens, cada um com nome, tipo e valor:

```
item_details.custom_attributes = [
  { "name": "forma_pagamento", "type": "string", "value": "Boleto" },
  { "name": "numero_proposta",  "type": "number", "value": 4471 }
]
```

Nao e `{"forma_pagamento": "Boleto"}`. Escrever no formato de par chave e valor tem o mesmo desfecho silencioso.

E o campo so aparece na tela se a definicao dele existir na configuracao do Kanban da conta. Valor gravado num cartao para um nome que nunca foi definido nao aparece em lugar nenhum.
