# O que a voz faz sozinha e o que ela repassa (motor gptlive)

## A voz faz sozinha, durante a ligacao
1. Conversa seguindo `objective`, `first_message` e `style`.
2. Consulta a agenda: horarios livres de um tipo de agendamento, agenda do dia, disponibilidade de um
   atendente. SO se essas ferramentas estiverem ligadas no agente de texto executor.
3. Desliga no `max_duration_seconds`.

## A voz REPASSA ao agente de texto (ferramenta interna "pedir ao agente")
Tudo o que precisa ser FEITO: mandar link, endereco, proposta ou informacao pelo WhatsApp; marcar
agendamento; gravar atributo no contato; criar nota privada; mover card; rodar um fluxo. O agente de texto
executa na conversa daquele contato, com as ferramentas e a configuracao DELE, e decide se escreve ao
cliente. Fica uma pilula na conversa a cada pedido.

Para isso funcionar, o agente de texto precisa estar com a ferramenta correspondente LIGADA:
- mandar mensagem: nada a ligar (ele escreve como em qualquer turno);
- agendar: ferramentas de agendamento + tipo de agendamento configurado;
- gravar atributo: ferramenta de atualizar atributo + o atributo criado na conta (organizar-contatos);
- nota privada: ferramenta de nota privada;
- mover card: ferramentas do funil.

## Regras fixas dos pedidos
- **No maximo 3 pedidos por ligacao.** Passou disso, vira aviso para a equipe. Por isso o roteiro deve juntar
  tudo num pedido UNICO no fim da ligacao (ver `roteiro.md`).
- O pedido roda mesmo com atendente humano na conversa ou com a IA pausada.
- Nao liga a IA de texto na conversa nem mexe na IA que ja estava la.
- Nao passa para humano, nao encerra a conversa, nao transfere e nao agenda retorno.
- WhatsApp oficial fora da janela de 24 horas (o cliente nao escreveu nas ultimas 24 h): o agente de texto
  faz as acoes internas (atributo, nota, card) mas NAO consegue escrever ao cliente. A voz e avisada e diz que
  a equipe vai entrar em contato.
- A voz nunca diz "ja mandei" nem dita link, endereco ou documento em voz alta. Diz que pediu e que vai chegar
  no WhatsApp.
- O agente de texto nao inventa dado: sem o link/preco real, nao manda nada.

## O que NAO funciona
- Escrever as perguntas da ligacao nas instrucoes ou cenarios do agente de texto: a voz nao le.
- Gravar as respostas da ligacao num bloco do fluxo depois: o fluxo so sabe se atendeu, nao o que foi dito.
- Pedido vago ("salve as respostas da pesquisa"): o agente de texto nao sabe em que campo por cada resposta.
  O pedido tem de citar o nome exato de cada atributo.
