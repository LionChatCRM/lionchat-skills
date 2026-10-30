# O que so pode ser feito na mao, no painel

Use este arquivo para escrever o bloco "Proximos passos manuais" do resumo final e para nao prometer
o que voce nao consegue entregar. Cada item traz o caminho exato do menu para voce copiar na resposta
ao cliente.

Regra geral: nao chumbe o endereco do painel. Pergunte ao cliente o endereco que ele usa para entrar
ou reaproveite o que ele ja colou - varias empresas usam o LionChat com marca e endereco proprios, e
mandar o cliente para o endereco errado significa que ele nao vai achar a conta dele.

---

## 1. Criar e excluir caixa de entrada

**Caminho:** Configuracoes, Caixas de Entrada, botao de adicionar.

Nao existe ferramenta para criar nem para excluir caixa. Voce escolhe o canal COM o cliente, explica
o que muda, e depois pede o numero (id) da caixa recem-criada para assumir a configuracao.

Passo a passo curto para passar ao cliente:

1. Configuracoes, Caixas de Entrada.
2. Botao de adicionar caixa.
3. Escolher o tipo de canal.
4. Preencher os dados do canal.
5. Escolher os agentes.
6. Confirmar.

Ao excluir, avise antes das consequencias: as conversas ficam em modo somente leitura, as campanhas
da caixa sao removidas junto e, em caixa por QR Code, o WhatsApp e desconectado do celular. So
administrador exclui e a confirmacao pede que ele digite o nome da caixa.

Desde 18/09/2026 a caixa excluida SOME da lista na hora e a vaga do plano fica livre na hora - da
para conectar outra (ate com o mesmo numero) logo em seguida. O resto do trabalho roda em segundo
plano; se a exclusao nao terminar, a caixa volta a aparecer sozinha em ate 30 minutos.

---

## 2. Ler o QR Code do WhatsApp

**Caminho:** dentro da caixa por QR Code, aba Conexao.

A leitura e sempre feita no celular, pela pessoa. Voce consegue acompanhar o estado da sessao e
entregar o texto do codigo, mas nao consegue escanear.

Existe tambem um LINK de conexao: uma pagina publica que permite a quem esta com o celular ler o
codigo sem ter login no painel - util quando o dono do numero nao e quem mexe no sistema. Ele dura 2
horas, so um link fica valido por vez (gerar outro invalida o anterior) e pode ser revogado. Desde
28/08/2026 ele TEM ferramenta (so administrador): `lionchat_inboxes_waha_connect_link_create` /
`_revoke` (caixa por QR Code), `lionchat_inboxes_coex_backup_connect_link_create` / `_revoke`
(conexao hibrida) e `lionchat_inboxes_lioncalls_connect_link_create` / `_revoke` (linha de ligacao -
detalhe na skill `agente-de-ligacao`). Revogar derruba a sessao que o link abriu. Entregue o link
so a quem vai ler o codigo e revogue ao terminar.

---

## 3. Conectar o WhatsApp Oficial

**Caminho:** Configuracoes, Caixas de Entrada, adicionar, WhatsApp.

O botao abre uma janela da Meta com login e escolha de portfolio de negocios. Nao ha como automatizar.
A alternativa e preencher os quatro dados na mao, que tambem e no painel.

---

## 4. Autorizar Instagram, Facebook e TikTok

**Caminho:** Configuracoes, Caixas de Entrada, adicionar, e depois a janela de autorizacao.

Alem da criacao, essas caixas pedem REAUTORIZACAO de tempos em tempos. Quando isso acontece aparece
um aviso na caixa e o cliente precisa refazer a autorizacao no painel - sem isso a caixa para de
receber.

---

## 5. Horario de atendimento

**Caminho da empresa inteira:** Configuracoes > Conta, secao Horario Comercial.
**Caminho do Chat do Site:** dentro daquela caixa, aba Horario de funcionamento.

Nem um nem outro pode ser gravado por ferramenta. A pausa de almoco so existe no horario da conta.

O que voce CONSEGUE fazer e ajustar o FUSO da conta, e isso deve ser feito ANTES de criar caixas,
porque caixa nova herda o fuso.

Sem o horario da conta cadastrado e ligado, nenhuma caixa de WhatsApp, Instagram, Facebook, Telegram
ou e-mail consegue enviar aviso de ausencia, e o SLA que deveria contar so no expediente passa a
contar 24 horas por dia.

---

## 6. Conversa unica

**Caminho:** dentro da caixa, aba Configuracoes.

O interruptor "Bloquear para conversa unica" fica nessa tela e nao passa pela ferramenta. Voce
propoe o valor; o cliente salva. Depois releia a caixa e confirme.

O aviso de ausencia, que ficava aqui, deixou de ser item de painel: `lionchat_inboxes_update` grava
o interruptor (`out_of_office_enabled`) e o texto (`out_of_office_message`). O horario da conta, do
qual ele depende, continua no painel (item 5).

---

## 7. Pesquisa de satisfacao

**Caminho:** dentro da caixa, aba CSAT.

Ligar a pesquisa, escolher o formato da nota, escrever a pergunta, o agradecimento e a resposta para
nota invalida, definir a espera depois de finalizar e o filtro por etiqueta - tudo no painel.

Voce escreve os textos prontos para o cliente colar e depois le os resultados pelas ferramentas de
pesquisa.

---

## 8. Interruptores do WhatsApp por QR Code

**Caminho:** dentro da caixa, aba Configuracao. **Tambem por ferramenta:**
`lionchat_inboxes_update` com `channel.additional_attributes` (ver `configurar-caixa.md`, secao 1).

Aceitar Grupos, Recusar ligacoes (com texto de resposta) e Marcar como visto. "Aceitar Canais" e
"Aceitar Lista de Transmissao" sumiram da tela em 09/10/2026 e ficam sempre desligados.

Avise que ligar ou desligar Aceitar Grupos reinicia a sessao do WhatsApp por alguns segundos.

---

## 9. Chat do Site: formulario pre-chat e construtor do widget

**Caminho:** dentro da caixa, abas Formulario Chat Pre e Construtor de Widget.

Telas proprias, com previa ao vivo. Nao ha ferramenta.

---

## 10. Supervisor de time

**Caminho:** Configuracoes, Times, o time, os membros.

Supervisor de CAIXA voce consegue definir por ferramenta. Supervisor de TIME nao: e no painel.

---

## 11. Cronometro do SLA a partir da atribuicao

**Caminho:** Configuracoes, SLA.

O ajuste que faz o prazo da primeira resposta comecar quando alguem assume a conversa nao esta
disponivel nas ferramentas.

---

## 12. Recursos de plano

Ligar SLA, cargos personalizados, o modo Equilibrado, a politica de capacidade e a ligacao nao e da
conta do cliente nem sua: e do time do LionChat. Se algum desses vier recusado, oriente o cliente a
falar com o suporte, informando exatamente qual recurso.

O mesmo vale para o limite de caixas, de agentes e de caixas de voz do plano.

---

## 13. Configuracao de e-mail

**Caminho:** Configuracoes, Caixas de Entrada, adicionar, E-mail.

Na hora de SALVAR os dados de recebimento e envio (IMAP e SMTP), o sistema testa a conexao de
verdade com o servidor e recusa se ele nao responder ou se a senha estiver errada. Tenha as
credenciais em maos antes de comecar.

---

## 14. Nada de cobranca

Plano, fatura, cartao, saldo e assinatura estao fora do que voce faz, por qualquer caminho. Se o
cliente pedir, oriente a falar com o suporte.
