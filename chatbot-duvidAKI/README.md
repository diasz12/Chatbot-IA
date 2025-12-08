# DuvidAKI - Chatbot IA com Base de Conhecimento

Chatbot inteligente integrado ao Slack que utiliza RAG (Retrieval Augmented Generation) para responder perguntas baseadas em documentação do Confluence e repositórios do GitHub.

## Características

- **Base de Conhecimento RAG**: Busca semântica com ChromaDB e embeddings OpenAI
- **Integração Confluence**: Extrai e indexa documentação de espaços Confluence
- **Integração GitHub**: Indexa READMEs, documentação e código de repositórios
- **Bot Slack**: Responde perguntas via menções, DMs e slash commands
- **Respostas Contextualizadas**: Usa LLM (GPT-4) com contexto recuperado

## Arquitetura

```
┌─────────────────┐
│  Slack Bot      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  RAG Service    │
└────────┬────────┘
         │
         ├──────────────┬──────────────┐
         ▼              ▼              ▼
┌──────────────┐ ┌─────────────┐ ┌──────────────┐
│ Confluence   │ │   GitHub    │ │ Vector Store │
│   Crawler    │ │   Crawler   │ │  (ChromaDB)  │
└──────────────┘ └─────────────┘ └──────────────┘
```

## Requisitos

- Python 3.9+
- OpenAI API Key
- Slack App (Bot Token, App Token, Signing Secret)
- Confluence API Token (opcional)
- GitHub Personal Access Token (opcional)

## Instalação

### 1. Clone o repositório

```bash
git clone https://github.com/diasz12/chatbot-duvidAKI.git
cd chatbot-duvidAKI
```

### 2. Crie ambiente virtual

```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate  # Windows
```

### 3. Instale dependências

```bash
pip install -r requirements.txt
```

### 4. Configure variáveis de ambiente

```bash
cp .env.example .env
```

Edite o arquivo `.env` com suas credenciais:

```env
# OpenAI (OBRIGATÓRIO)
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o-mini

# Slack (OBRIGATÓRIO para bot)
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...
SLACK_SIGNING_SECRET=...

# Confluence (OPCIONAL)
CONFLUENCE_URL=https://seu-dominio.atlassian.net
CONFLUENCE_EMAIL=seu-email@example.com
CONFLUENCE_API_TOKEN=...
CONFLUENCE_SPACE_KEY=SPACE

# GitHub (OPCIONAL)
GITHUB_TOKEN=ghp_...
GITHUB_REPOS=owner/repo1,owner/repo2
```

## Configuração do Slack

### 1. Criar Slack App

1. Acesse https://api.slack.com/apps
2. Clique em "Create New App" → "From scratch"
3. Nomeie o app (ex: DuvidAKI) e selecione o workspace

### 2. Configurar Bot Token Scopes

Em **OAuth & Permissions**, adicione os seguintes scopes:

```
- app_mentions:read
- channels:history
- chat:write
- commands
- im:history
- im:read
- im:write
```

### 3. Habilitar Socket Mode

1. Vá em **Socket Mode** e habilite
2. Gere um App-Level Token com escopo `connections:write`
3. Copie o token (começa com `xapp-`)

### 4. Habilitar Event Subscriptions

Em **Event Subscriptions**, inscreva-se nos eventos:

```
- app_mention
- message.im
```

### 5. Criar Slash Commands

Em **Slash Commands**, crie:

- `/duvidaki` - Fazer uma pergunta
- `/duvidaki-stats` - Ver estatísticas

### 6. Instalar no Workspace

1. Vá em **OAuth & Permissions**
2. Clique em "Install to Workspace"
3. Copie o **Bot User OAuth Token** (começa com `xoxb-`)

### 7. Obter Signing Secret

Em **Basic Information**, copie o **Signing Secret**

## Configuração Confluence

### Gerar API Token

1. Acesse https://id.atlassian.com/manage-profile/security/api-tokens
2. Crie um novo token
3. Use no `.env` como `CONFLUENCE_API_TOKEN`

## Configuração GitHub

### Gerar Personal Access Token

1. Acesse https://github.com/settings/tokens
2. Gere um token com escopo `repo` (para repositórios privados) ou `public_repo` (públicos)
3. Use no `.env` como `GITHUB_TOKEN`

## Uso

### 1. Indexar Base de Conhecimento

```bash
# Indexar tudo (Confluence + GitHub)
python main.py index --all

# Indexar apenas Confluence
python main.py index --confluence

# Indexar apenas GitHub
python main.py index --github
```

### 2. Iniciar Bot Slack

```bash
python main.py start
```

### 3. Testar Queries Localmente

```bash
python main.py query "Como fazer deploy?"
```

### 4. Ver Estatísticas

```bash
python main.py stats
```

### 5. Resetar Base de Conhecimento

```bash
python main.py reset
```

## Como Usar no Slack

### Mencionar o Bot

```
@DuvidAKI Como funciona o processo de CI/CD?
```

### Direct Message

Envie uma mensagem direta ao bot:

```
Qual a política de segurança da empresa?
```

### Slash Command

```
/duvidaki Como configurar o ambiente de desenvolvimento?
```

## Estrutura do Projeto

```
chatbot-duvidAKI/
├── src/
│   ├── config.py              # Configurações
│   ├── crawlers/
│   │   ├── confluence_crawler.py
│   │   └── github_crawler.py
│   ├── services/
│   │   ├── vector_store.py    # ChromaDB
│   │   ├── document_processor.py
│   │   └── rag_service.py     # Lógica RAG
│   ├── integrations/
│   │   └── slack_bot.py       # Bot Slack
│   └── utils/
│       └── logger.py
├── data/                      # Dados persistidos
├── tests/
├── main.py                    # Entry point
├── requirements.txt
├── .env.example
└── README.md
```

## Como Funciona

1. **Indexação**:
   - Crawlers extraem conteúdo do Confluence e GitHub
   - Documentos são divididos em chunks
   - Embeddings são gerados via OpenAI
   - Armazenados no ChromaDB

2. **Consulta**:
   - Usuário faz pergunta no Slack
   - Query é convertida em embedding
   - Busca semântica no ChromaDB
   - Top-K documentos relevantes são recuperados
   - LLM gera resposta baseada no contexto

3. **Resposta**:
   - Resposta formatada é enviada ao Slack
   - Inclui citações das fontes quando possível

## Custos Estimados

### OpenAI API

- **Embeddings** (text-embedding-3-small): ~$0.02 / 1M tokens
- **Chat** (gpt-4o-mini): ~$0.15 / 1M input tokens, ~$0.60 / 1M output tokens

**Estimativa para 1000 documentos:**
- Indexação: ~$1-2 (uma vez)
- Consultas: ~$0.01-0.05 por pergunta

### Infraestrutura

- ChromaDB: Gratuito (local)
- Slack: Gratuito
- Confluence/GitHub: Já existentes

## Deploy em Produção

### Opção 1: VM/VPS

```bash
# Use um process manager
pip install supervisor
# ou
pip install pm2
```

### Opção 2: Docker

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "main.py", "start"]
```

### Opção 3: Cloud (Heroku, AWS, GCP)

Configure as variáveis de ambiente e execute:

```bash
python main.py start
```

## Manutenção

### Sincronização Periódica

Configure um cron job para reindexar periodicamente:

```bash
# Crontab: reindexar diariamente às 2h
0 2 * * * cd /path/to/chatbot-duvidAKI && python main.py index --all
```

### Monitoramento

Monitore os logs:

```bash
tail -f logs/app.log
```

## Troubleshooting

### Erro: "Slack not configured"

Verifique se todas as variáveis `SLACK_*` estão no `.env`

### Erro: "OpenAI API error"

Verifique:
- API key válida
- Créditos disponíveis na conta OpenAI

### Bot não responde

Verifique:
- Bot está rodando (`python main.py start`)
- Base de conhecimento indexada (`python main.py stats`)
- Bot tem permissões corretas no Slack

## Contribuindo

1. Fork o projeto
2. Crie uma branch (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -m 'Add nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

## Licença

MIT License

## Contato

Para dúvidas ou sugestões, abra uma issue no GitHub.
