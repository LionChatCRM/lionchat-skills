# Como escrever o roteiro (objective) e a primeira frase

## Principios
- Ao telefone, uma pergunta por vez, frases curtas. Escreva o roteiro como passos numerados.
- Diga o que fazer quando a pessoa nao pode falar, nao tem interesse ou pede para parar: agradecer e encerrar.
- Tudo o que precisa ACONTECER vai num pedido UNICO no fim, citando o nome exato de cada atributo (limite de
  3 pedidos por ligacao).
- Nunca deixe a voz prometer preco, desconto, endereco ou link que ela nao recebeu no roteiro.
- `first_message` curta: quem fala, de onde, por que liga e se pode falar agora.
- `style`: "cordial, frases curtas, uma pergunta por vez, sem jargao, portugues do Brasil" serve de base.

## Modelo 1 - Pesquisa de satisfacao
Antes: criar no contato os atributos `nota_atendimento` (numero), `recomendaria` (sim/nao) e `sugestao`
(texto); no agente de texto executor, ligar atualizar atributo e nota privada.

first_message: "Oi, {{contact.name}}, aqui e da [EMPRESA]. Posso te fazer tres perguntas rapidas sobre o seu
atendimento? Leva um minuto."

objective:
```
Voce faz uma pesquisa de satisfacao curta. Uma pergunta por vez, esperando a resposta:
1. "De 0 a 10, que nota voce da para o atendimento que recebeu?"
2. "Voce recomendaria a [EMPRESA] para um amigo ou parente? Sim ou nao?"
3. "Tem alguma coisa que a gente poderia melhorar?"
Se a pessoa nao quiser responder, agradeca e encerre.
No final, faca UM unico pedido ao agente: "Salve no contato: nota_atendimento = [nota],
recomendaria = [sim/nao], sugestao = [texto]. Registre uma nota privada com o resumo da pesquisa."
Agradeca e se despeca. Nao diga que ja salvou; diga que registrou.
```

## Modelo 2 - Triagem de lead de anuncio
Antes: atributos da triagem criados; agente de texto com atualizar atributo, nota privada e, se for marcar,
as ferramentas de agendamento.

objective:
```
Voce liga para quem pediu informacao sobre [PRODUTO/SERVICO] num anuncio.
1. Confirme se e um bom momento. Se nao for, pergunte o melhor horario, agradeca e encerre.
2. Pergunte [PERGUNTA 1]. 3. Pergunte [PERGUNTA 2]. 4. Pergunte [PERGUNTA 3].
5. Se a pessoa quiser marcar [AVALIACAO/VISITA], consulte os horarios livres e ofereca duas opcoes.
No final, faca UM unico pedido ao agente: "Salve no contato: [atributo1] = ..., [atributo2] = ...
[Se marcou: agende [TIPO] para [dia e hora] escolhidos.] Registre uma nota privada com o resumo da ligacao.
[Mande no WhatsApp: o que foi combinado.]"
Nunca fale preco nem prometa desconto. Diga que registrou e que vai chegar no WhatsApp.
```

## Modelo 3 - Confirmacao de consulta
objective:
```
Voce liga para confirmar a consulta de {{contact.name}}.
1. Diga que e da [CLINICA] e que esta confirmando a consulta marcada.
2. Pergunte se a pessoa confirma presenca.
3. Se nao puder ir, consulte horarios livres e ofereca duas opcoes.
No final, UM unico pedido ao agente: "Registre nota privada: confirmou / quer remarcar para [dia e hora] /
cancelou. [Se remarcou: remarque para o horario escolhido.]"
```
