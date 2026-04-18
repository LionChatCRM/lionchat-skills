# LionChat Skills

Skills para Claude Code que configuram sua conta LionChat através de uma conversa natural com o Claude.

## O que é isso?

Em vez de passar horas configurando funil, tags, respostas rápidas e automações manualmente no painel do LionChat, você conversa com o Claude Code sobre seu negócio e ele monta tudo pra você via API.

O Claude:
- Entrevista você sobre o que você vende, ciclo de venda, gargalos
- Propõe uma estrutura completa (funil, tags, respostas, campos, automações)
- Mostra tudo antes de criar
- Só executa depois que você confirma
- Usa seu API token — nada fica no servidor de ninguém

## Como instalar

### Pré-requisitos

- [Claude Code](https://docs.claude.com/en/docs/claude-code) instalado
- Conta no [LionChat](https://app.lionchat.com.br) com acesso de administrador
- API token da sua conta (Configurações > Perfil > API Access Token)

### Instalação

```bash
mkdir -p ~/.claude/plugins
git clone https://github.com/LionChatCRM/lionchat-skills ~/.claude/plugins/lionchat-skills
```

Pronto. Abra (ou reinicie) o Claude Code — o plugin é detectado automaticamente.

**Verificar que a skill foi reconhecida:** dentro do Claude Code, digite `/` e o autocomplete deve sugerir `/lionchat-skills:configurar-conta`.

### Atualização

```bash
cd ~/.claude/plugins/lionchat-skills && git pull
```

## Como usar

Dentro do Claude Code, digite:

```
/lionchat-skills:configurar-conta
```

Ou simplesmente peça em português:

> "Quero configurar minha conta LionChat. Sou dentista, faço implantes..."

O Claude reconhece a skill e entra no fluxo de configuração.

## Skills disponíveis

### `configurar-conta`

Configura uma conta LionChat do zero:

- Funis de venda com etapas personalizadas
- Tags de segmentação
- Respostas rápidas com variáveis
- Campos personalizados (contato e conversa)
- Automações de etapa (follow-up automático, atribuição, etc.)
- Regras de automação globais
- SLA de atendimento
- Times de atendimento
- Horários de trabalho
- Macros

**O que NÃO faz** (precisa fazer manual no painel):

- Conectar WhatsApp (precisa QR Code)
- Conectar Instagram/Facebook (OAuth)
- Configurar integrações de pagamento (Guru, Hotmart, etc.) — você precisa do token da outra plataforma
- Convidar agentes novos (envio de email)

## Segurança

- Seu API token fica apenas na sessão atual do Claude Code
- Nada é enviado pra nenhum servidor além do seu próprio LionChat
- O token nunca aparece em commits, logs ou arquivos do projeto
- Se quiser salvar pra próximas sessões, o Claude pergunta e salva em `~/.lionchat/credentials` (fora do seu projeto)

## Suporte

- Documentação da API: [api-docs do LionChat](https://app.lionchat.com.br/docs)
- Issues desta skill: [github.com/LionChatCRM/lionchat-skills/issues](https://github.com/LionChatCRM/lionchat-skills/issues)

## Licença

MIT
