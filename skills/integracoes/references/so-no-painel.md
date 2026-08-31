# O que so uma pessoa consegue fazer

Nada aqui e limitacao sua: sao passos que dependem de um clique num site de fora, de um cadastro no
painel de outra empresa, ou de uma liberacao que so o suporte faz. Diga isso na hora de PROPOR, nunca
so no fim - a pessoa precisa saber que vai ter trabalho.

Para cada item abaixo, entregue o roteiro em texto simples e ofereca conferir depois que ela fizer.

---

## 1. Conectar pelo Facebook (Meta Lead Ads)

Conectar exige um acesso que so o login do Facebook dentro do navegador produz. Voce nao consegue.

**Roteiro para a pessoa:**

1. Menu lateral, Configuracoes, Integracoes, Meta Lead Ads.
2. Clique em "Conectar nova pagina".
3. Faca login com a conta que administra a pagina do Facebook ou Instagram onde os anuncios estao, e
   autorize as permissoes.
4. De volta ao LionChat, aparece a janela "Escolha as paginas". **Marque so as paginas que
   interessam** - com tudo marcado, dezenas de paginas entram de uma vez.
5. Clique em Conectar.

**Depois disso voce faz tudo:** listar as paginas, buscar os formularios, montar o de-para dos
campos, ligar o alvo, ativar e desativar formulario, ver o historico e remover paginas em lote.

**Cuidado:** a telinha do proprio Facebook nao serve de filtro - ela nasce com tudo marcado e nem
aparece para quem ja autorizou antes.

---

## 2. Autorizar no navegador: Google, Slack, Notion, Linear, Shopify, Conta Azul

Todas essas conexoes acontecem fora do LionChat, numa tela da outra empresa. A parte que voce faz e
entregar o link e cuidar do que vem depois.

| Integracao | Como comeca | O que voce faz depois |
|---|---|---|
| **Google Calendar** | `lionchat_google_calendar_list_1` devolve o link para a pessoa abrir | escolher a agenda, ligar a sincronizacao, escolher agendas compartilhadas, reativar |
| **Google Contatos** | `lionchat_google_contacts_connect` devolve o link | ligar a sincronizacao e mandar de uma vez os contatos que ja existem |
| **Conta Azul** | crie com `lionchat_conta_azul_integrations_create` (nasce pendente) e chame `..._connect`: ele devolve o link, que vale 15 minutos | mapear os eventos, forcar a varredura, ver o historico |
| **Slack** | a conexao inicial e no navegador | escolher o canal que recebe, listar canais, desconectar |
| **Notion** | conexao no navegador | buscar paginas e trazer para a base de conhecimento da IA |
| **Linear** | conexao no navegador | criar tarefa a partir da conversa, vincular e desvincular |
| **Shopify** | conexao no navegador | ver os pedidos do cliente dentro da conversa |

Google Calendar e Google Contatos: **uma conexao por conta e so administrador**. Desconectar o
Calendar apaga a escolha das agendas compartilhadas, e ela nao volta ao reconectar.

---

## 3. Cadastrar o nosso endereco no painel do outro sistema

**Este e o passo que mais trava cliente.** Toda integracao de entrada com endereco proprio depende
de alguem colar aquele endereco la. Enquanto isso nao acontece, nao chega absolutamente nada - e o
painel nao tem como avisar.

Quem gera endereco (`webhook_url` na resposta): os 7 checkouts, o Webhook Universal, a Omie e a
e-Clinica (esta com um endereco por unidade, alem do geral).

Quem **nao** gera endereco nenhum: **Meta Lead Ads** e **Conta Azul**. Nao procure.

Detalhes que mudam a instrucao:

- **Guru:** um endereco por tipo (transacao, assinatura, ingresso). Cadastre os que forem usados.
- **Greenn:** la o endereco e cadastrado **por produto**, nao uma vez so.
- **e-Clinica:** um endereco por unidade. Cada filial precisa do seu.
- **Omie:** o endereco vai nos webhooks nativos da Omie.

---

## 4. e-Clinica: token, unidades e lembretes sao tela

Pelo conector a e-Clinica e **so leitura**. Ficam na tela (Configuracoes, Integracoes, e-Clinica):

- cadastrar e editar unidade (apelido, token da clinica);
- colar o token gerado pelo usuario principal no painel da e-Clinica;
- o mapeamento de qual evento dispara o que;
- **as reguas de lembrete automatico**: quantos dias antes, qual fluxo ou automacao dispara, o
  filtro "so quando" (por procedimento, por profissional) e a hora fixa do disparo.

O que voce consegue: ler a situacao, ver o historico de eventos, reprocessar um evento, ver o
historico dos lembretes disparados e reenviar um deles.

---

## 5. Liberacao da instalacao (fale com o suporte)

Algumas integracoes so aparecem para a conta depois que a empresa que hospeda o LionChat libera -
ou porque a funcionalidade nasce desligada, ou porque a credencial e da instalacao, nao do cliente.

Como reconhecer: **`lionchat_integrations_apps_list` nao traz aquela integracao**. A lista ja vem
filtrada pelo que a conta pode usar.

Nesses casos: pare, explique em uma frase que aquela integracao precisa ser liberada pelo suporte, e
nao tente outro caminho. Insistir so gera erro sem explicacao.

**Um caso especial e traicoeiro - Meta Lead Ads.** Ele pode APARECER na lista, conectar, sincronizar
formularios, aceitar o de-para e mostrar tudo verde, enquanto a captura de leads esta desligada na
instalacao. O sinal e este: **formulario ativo, mapeamento certo, e ZERO eventos no historico**.
Quando bater essa combinacao, nao procure erro de configuracao - peca ao suporte para conferir se a
captura de leads de anuncio esta ligada nesta instalacao.

---

## 6. Outras coisas que nao existem pela API

- **Reprocessar eventos em lote.** O reprocessamento e um evento por vez, em todas as integracoes.
- **Desbloquear webhook de saida travado na hora.** Nao ha botao. Depois de 10 falhas seguidas ele
  fica 1 hora bloqueado e passada a hora volta a tentar sozinho; a unica forma de destravar antes
  disso e trocar o endereco.
- **Ligar "evento Lead automatico ao criar cartao"** nos eventos de conversao do funil.
- **Cadastrar a chave da TopSend.** Ela e gravada na tela (Configuracoes, Integracoes, TopSend). Sem
  a chave, nem o teste de conexao nem a criacao de campanha funcionam.
- **Subir arquivo de audio para a TopSend.** Use um audio ja cadastrado la (`audio_id`) ou um
  endereco publico (`audio_url`).
- **Criar evento de conversao dentro do Facebook, do Google Analytics ou do Google Ads.**
- **Apagar registro de entrega do webhook de saida** - as entregas sao so leitura.
