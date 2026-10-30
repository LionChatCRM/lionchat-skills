# LionChat Skills

Manuais que ensinam o Claude a operar a sua conta LionChat — para você pedir em português o que quer,
sem precisar conhecer o sistema.

## O que é isso?

O LionChat tem muita coisa: agente de IA, fluxos, automações, funil, campanhas, agenda, integrações.
Cada uma com dezenas de opções, e várias delas falham em silêncio se forem montadas errado — a coisa
é criada, parece certa na tela e simplesmente não funciona.

Estas skills são o manual de operação de cada uma dessas áreas, escrito para a IA ler. Com elas
instaladas, você conversa:

> "monta um agente de IA que atende meu WhatsApp e agenda consulta"

E o Claude sabe o que perguntar, em que ordem montar, o que ligar em quê, e o que **não** prometer
para você. Ele entrevista, mostra a proposta inteira e só executa depois que você confirma.

Cada afirmação técnica destes manuais foi conferida contra o código do LionChat.

## Como instalar

### Pré-requisitos

- [Claude Code](https://docs.claude.com/en/docs/claude-code) instalado
- Conta no [LionChat](https://app.lionchat.com.br) com acesso de administrador
- O conector do LionChat (MCP) ligado no seu Claude Code — é por ele que o Claude mexe na sua conta.
  A skill `conectar-lionchat` ensina a ligar, e o que fazer se o recurso ainda não estiver liberado
  na sua conta.

### Instalação

```bash
mkdir -p ~/.claude/plugins
git clone https://github.com/LionChatCRM/lionchat-skills ~/.claude/plugins/lionchat-skills
```

Pronto. Abra (ou reinicie) o Claude Code — o plugin é detectado automaticamente.

**Verificar que funcionou:** dentro do Claude Code, digite `/` — o autocomplete deve sugerir as skills `lionchat-skills:...`.

### Atualização

```bash
cd ~/.claude/plugins/lionchat-skills && git pull
```

## Como usar

Peça em português o que você quer. O Claude escolhe a skill certa sozinho:

> "monta um agente de IA que atende meu WhatsApp e agenda consulta"

> "cria um fluxo que cobra quem pediu orçamento e sumiu"

> "organiza meu funil de vendas"

> "sou dentista, faço implantes — configura minha conta"

Ou chame uma direto, pelo nome: `/lionchat-skills:agente-de-ia`

Em qualquer caso o Claude **entrevista você antes**, mostra a proposta inteira e **só cria depois que
você confirma**.

## Skills disponíveis

### `conectar-lionchat` — a base

Ensina o Claude a se conectar na sua conta, descobrir o que ele pode fazer ali e as regras que valem para tudo. As outras skills se apoiam nela.

### `agente-de-ia` — Agente de IA

Monta o atendente virtual de ponta a ponta: instruções, cenários, base de conhecimento, ferramentas, biblioteca de arquivos, quando transferir para uma pessoa e teste antes de soltar no cliente.

### `criar-fluxos` — Fluxos

Desenha fluxos que funcionam, nos três tipos — mensagem, ferramenta da IA e o Fluxo de Ações, que não fica preso a caixa nenhuma e só executa ações: todos os blocos e gatilhos (inclusive o aviso de card parado e o "Ligar com IA"), variáveis, espera, o que fazer com quem não responde, e como achar onde travou.

### `automacoes-e-macros` — Automações e macros

O que o sistema faz sozinho e o que o atendente dispara com um clique. Os gatilhos, as condições e as ações de cada um — inclusive as 42 ações da macro, que hoje grava atributo, divide entre atendentes em rodízio, manda conversão para os anúncios e dispara um Fluxo de Ações.

### `funil-de-vendas` — Funil de vendas (CRM)

Monta o funil pelo seu negócio: etapas, campos do card, checklists, automações de etapa, ganho e perdido, card adiado, tarefa automática, importar vendas, venda dividida com outro vendedor e o "Analisar com IA".

### `caixas-e-atendimento` — Caixas e atendimento

Os canais e o que muda entre eles, saudação, horário, distribuição das conversas, times, permissões e prazos de atendimento.

### `organizar-contatos` — Contatos e dados

Campos, etiquetas, segmentos, importação de planilha — e a regra de quando um dado deve ser etiqueta e quando deve ser campo.

### `campanhas-e-modelos` — Campanhas e modelos

Disparo em massa sem queimar o número, montagem do público, e tudo sobre modelo aprovado do WhatsApp.

### `agenda-e-relatorios` — Agenda e relatórios

Agendamento online com lembretes, várias agendas (unidades), programas em sessões, presença (compareceu/faltou), e o que cada número do relatório significa para o dono do negócio.

### `integracoes` — Integrações

Liga o LionChat a sistemas de fora: anúncios, plataformas de pagamento, sistemas de gestão e conversões.

### `agente-de-ligacao` — Ligação com IA

A IA liga para o contato pelo WhatsApp e conversa: o agente de ligação (roteiro, primeira frase, voz, tempo máximo), o agente de texto que executa o que o cliente pede na ligação, a ligação de teste e o bloco "Ligar com IA" no fluxo. Com modelos prontos de triagem de lead, confirmação de consulta e pesquisa de satisfação.

### `assinatura-eletronica` — Contratos e assinatura

Modelos de contrato, quem assina, documentos pedidos, envio pelo WhatsApp ou e-mail, acompanhamento e envio pelo fluxo. Nunca envia um contrato sem a sua autorização.

### `formulario-whatsapp` — Formulário do WhatsApp

O formulário nativo que abre dentro do WhatsApp por um botão de modelo: telas e perguntas, publicação na Meta, o botão no modelo e o que fazer com as respostas.

### `configurar-conta` — Configuração inicial

Monta o esqueleto da conta do zero numa conversa só: funil, etiquetas, campos, respostas prontas e automações.

**O que nenhuma delas faz** (precisa de gente no painel):

- Ler o QR Code do WhatsApp no celular (o Claude pode gerar o link de conexão para você mandar a quem tem o celular)
- Conectar Instagram e Facebook (autorização pelo navegador)
- Pegar o token da plataforma de pagamento (Guru, Hotmart, Kiwify) — esse está do lado de lá

## Segurança

- Suas credenciais ficam no conector do LionChat, na sua máquina — não neste repositório
- Nada é enviado para nenhum servidor além do seu próprio LionChat
- Elas criam e ajustam, e sempre pedem sua confirmação antes. O que não tem volta (excluir uma etapa, aplicar a revisão da IA no funil, enviar contrato) pede uma confirmação própria
- Se algum dia o Claude precisar de um token, ele nunca mostra o valor completo na tela

## Suporte

- Documentação da API: [api-docs do LionChat](https://app.lionchat.com.br/docs)
- Issues desta skill: [github.com/LionChatCRM/lionchat-skills/issues](https://github.com/LionChatCRM/lionchat-skills/issues)

## Licença

MIT
