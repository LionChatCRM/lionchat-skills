# Erros: o que dizer ao cliente e o que NAO repetir

Indice:

1. Tabela rapida
2. Recusa por acesso (o caso mais comum com cliente novo)
3. Recusa por limite do plano
4. "Parametro faltando" — o erro que engana
5. Nao existe, ou eu nao achei?
6. Regra de retentativa

---

## 1. Tabela rapida

| O que aconteceu | O que significa | Repetir adianta? | O que dizer ao cliente |
|---|---|---|---|
| **Acesso negado / nao autorizado** | o token ou o login perdeu validade | nao | "Sua conexao com o LionChat perdeu a validade. Refaca a conexao (ou gere o Token de acesso de novo em Configuracoes do Perfil)." |
| **Acesso proibido, mensagem de MCP nao habilitado** | portao de liberacao: a conta nao tem o recurso ou o usuario nao e administrador | nao | ver secao 2 |
| **Acesso proibido, sem mensagem de MCP** | o usuario nao tem permissao para AQUELA acao, ou o recurso especifico esta desligado na conta (por exemplo SLA, que nasce desligado) | nao | "Essa parte nao esta liberada na sua conta. Pule ou fale com o suporte." Pule o item e reporte no resumo final |
| **Pagamento necessario** | bateu no limite do plano (atendentes, caixas) | nao | ver secao 3 |
| **Nao encontrado** | o item nao existe, OU voce nao tem acesso a ele | nao, sem antes conferir | "Nao achei esse item. Confere o numero comigo?" Nunca afirme so uma das duas hipoteses |
| **Dado invalido** | algum campo esta em formato errado | nao, corrija primeiro | leia a mensagem, corrija o campo, tente uma vez |
| **Muitas chamadas** | freio de frequencia do conector remoto (120 por minuto) | so depois de 1 minuto | "Vou continuar em um minuto, o sistema pediu uma pausa." Continue de onde parou |
| **Erro do servidor** | falha temporaria da plataforma | uma vez so | tente uma vez; se falhar de novo, pule e reporte |
| **Recusa pedindo `confirm:true`** | **nao e erro.** E o sistema exigindo o OK do cliente | sim, depois do "sim" dele | descreva o efeito, espere o sim, reenvie a MESMA chamada com `confirm:true` |
| **Ferramenta desconhecida** | o nome que voce usou nao existe | nao | pesquise pelo padrao `lionchat_<area>_<acao>`. Ver `armadilhas.md`, secao 3 |

---

## 2. Recusa por acesso: "o acesso via MCP nao esta habilitado"

**Este e o final de caminho mais comum com cliente novo, e nao e defeito dele.** O recurso de
conectar IA nasce DESLIGADO em conta nova. Passa quem tem a marca individual de acesso; sem ela,
precisa das duas coisas: o recurso ligado na conta e ser administrador dela.

O que dizer:

> O acesso por IA e liberado por conta e ainda nao esta ligado na sua. Isso nao e erro de
> configuracao sua nem da conexao.
>
> Se voce e administrador da conta: fale com o suporte do LionChat para ativar no seu plano.
> Se voce nao e administrador: peca a um administrador da conta para verificar.
>
> Depois de liberado, nao precisa reconectar nada — eu volto a funcionar sozinho.

**Nao insista.** A recusa e permanente ate alguem liberar. Repetir a chamada nao muda nada.

Se a recusa aparecer **na hora de autorizar** o conector remoto, some a isso: a conta julgada e a
que estava ABERTA no painel naquele momento. Peca ao cliente para abrir a empresa certa e refazer.

Se a recusa aparecer **no painel**, ao criar um conector novo: aparece uma faixa amarela e o botao
"Adicionar conector" fica desligado. Listar e revogar continuam funcionando.

---

## 3. Recusa por limite do plano

Criar atendente ou caixa de entrada passa pelo teto do plano. Estourou, a plataforma recusa com uma
mensagem de licenca.

**Nao existe nenhuma ferramenta de assinatura, fatura, cartao ou saldo** — foram removidas de
proposito e nao voltam. Mas voce CONSEGUE ler o quadro: `lionchat_account_show` traz o nome do
plano, a situacao da assinatura e, em `resource_limits`, quantos atendentes e caixas ja existem
contra o que o plano permite. Leia isso ANTES de propor criar atendente ou caixa, e avise o cliente
se ja estiver no teto — e melhor do que descobrir na recusa.

O que dizer quando a recusa vier:

> Seu plano nao comporta mais [atendentes / caixas de entrada]. Isso se resolve no painel, na area
> de assinatura da conta — eu nao mexo em plano nem em pagamento por seguranca.

E siga com o resto do trabalho, reportando esse item no resumo final.

---

## 4. "Parametro faltando" — o erro que engana

Se voltar uma mensagem dizendo que falta um parametro, **desconfie do NOME do campo antes de
desconfiar do formato.**

O que acontece: voce mandou um campo que a ferramenta nao conhece. Ele foi descartado, o corpo da
chamada ficou vazio, e a plataforma reclama que nao veio nada. Parece problema de embrulho, mas nao
e.

**Certo:** confira os nomes dos campos na ficha da ferramenta. So mande os que estao la.

---

## 5. Nao existe, ou eu nao achei?

Antes de dizer ao cliente que a plataforma nao faz alguma coisa, percorra nesta ordem:

1. **Pesquise o catalogo** pelo padrao `lionchat_<area>_<acao>`. Alguns clientes de IA nao carregam
   as 850 ferramentas de uma vez, e ja aconteceu de uma sessao negar uma funcao que existia.
2. **Confira a lista de nomes errados** em `armadilhas.md`, secao 3 — ha 4 nomes que circulam por ai
   e nao existem.
3. **Confira `nao-tem-caminho.md`.** Se estiver la, a resposta certa nao e "nao existe": e "isso
   voce faz na tela, assim".
4. So entao diga que nao encontrou, e diga com essas palavras: "nao encontrei um caminho para isso
   pela conexao de IA" — nunca "o LionChat nao faz isso".

---

## 6. Regra de retentativa

- **Falha de acesso, permissao ou dado invalido: permanente.** Nao repita. Corrija ou reporte.
- **Falha de frequencia:** espere um minuto e continue de onde parou. Nao recomece o trabalho.
- **Falha do servidor:** uma tentativa a mais, so.
- **Falha numa chamada que CRIA alguma coisa:** nunca repita direto. Confira com uma leitura antes —
  se ela criou e falhou depois, repetir cria dois registros na conta do cliente.
- **Errou duas vezes: pare e reporte.** Nunca entre em laco de tentativas.
