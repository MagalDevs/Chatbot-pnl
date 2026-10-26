# Ferramentas e frameworks

[Voltar ao README](../README.md) | [Arquitetura](05-arquitetura.md) | [Hospedagem](11-hospedagem-e-publicacao.md)

## 1. Escolha para a demonstração

O alvo planejado é um chat web público, acessível por link em vários dispositivos. O desenvolvimento ocorre localmente; a futura publicação terá uma aplicação e banco persistente externo.

Este documento especifica ferramentas. Não autoriza instalação, criação de contas ou publicação nesta etapa.

## 2. Stack principal proposta

| Ferramenta | Tipo | Uso |
|---|---|---|
| Python 3.14.x | Linguagem | API e regras de conversa. |
| FastAPI | Framework web | Chat, painel e rotas de entrada/saída. |
| Uvicorn | Servidor ASGI | Executar a aplicação na porta indicada pela hospedagem. |
| Pydantic | Validação | Mensagens, formulários e comandos. |
| pydantic-settings | Configuração | URLs, ambiente, segredos e parâmetros. |
| PostgreSQL | Banco relacional | Conversas, pedidos, mensagens, sessões e indicadores. |
| SQLAlchemy 2.x | Persistência | Modelos, consultas e transações síncronas. |
| Psycopg 3 | Driver | Conexão PostgreSQL. |
| Alembic | Migrações | Evolução versionada do banco. |
| Jinja2 | Templates | Chat e painel renderizados pelo servidor. |
| HTML, CSS e JavaScript simples | Interface | Responsividade e atualização das mensagens. |
| pwdlib com Argon2 | Hash de senha | Usuários autorizados da recepção. |
| python-multipart | Formulários | Necessário se forem usados formulários do FastAPI. |

Fontes: [FastAPI](https://fastapi.tiangolo.com/features/), [templates](https://fastapi.tiangolo.com/advanced/templates/), [Pydantic](https://docs.pydantic.dev/latest/), [SQLAlchemy](https://docs.sqlalchemy.org/en/20/orm/session_basics.html), [Psycopg](https://www.psycopg.org/psycopg3/docs/), [Alembic](https://alembic.sqlalchemy.org/en/latest/) e [hash de senhas](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/). Os motivos de adequação ao projeto são decisões deste planejamento.

## 3. Desenvolvimento e testes

| Ferramenta | Uso planejado |
|---|---|
| Git e editor Python | Histórico, revisão e autoria. |
| uv | Ambiente virtual, dependências e lockfile. |
| pytest e HTTPX | Testes das regras, rotas e controles de acesso. |
| Ruff | Lint e formatação. |
| PostgreSQL de testes | Homologar o mesmo mecanismo usado no ambiente publicado. |
| Navegadores distintos | Testar visitantes isolados e painel restrito. |
| Docker Compose, opcional | Facilitar PostgreSQL local; instalação nativa é alternativa. |

Referências: [uv](https://docs.astral.sh/uv/), [pytest](https://docs.pytest.org/en/stable/), [HTTPX](https://www.python-httpx.org/), [Ruff](https://docs.astral.sh/ruff/) e [Compose](https://docs.docker.com/compose/).

SQLite pode auxiliar um ensaio isolado, mas seus testes não comprovam as transações e a concorrência do PostgreSQL. Não usar SQLite no disco temporário da aplicação publicada como fonte permanente de dados.

## 4. Hospedagem e persistência propostas

| Serviço | Função | Decisão |
|---|---|---|
| Render Web Service | Executar FastAPI e servir interface/arquivos estáticos | Usar subdomínio do provedor e HTTPS. |
| Neon PostgreSQL | Guardar dados persistentes | Usar conexão protegida e configuração fora do código. |
| Repositório Git | Origem da publicação | Criar/conectar somente na fase de implementação. |

O Render documenta [suporte a FastAPI](https://render.com/docs/deploy-fastapi). Seu [disco é efêmero](https://render.com/docs/free): arquivos locais podem desaparecer em reinício, suspensão ou nova publicação. Por isso, o desenho prevê banco externo e não depende de um arquivo SQLite no servidor.

O [Neon](https://neon.com/pricing) é a proposta de PostgreSQL persistente. Escolher região próxima da aplicação, limitar conexões e validar a URL/SSL fornecidos pelo serviço. Nenhuma conta Render ou Neon foi criada para este planejamento.

## 5. Simplicidade e acesso

- Um backend, sem microserviços, Redis, Celery, inbox/outbox ou LLM.
- Sessão por visitante; histórico guardado no banco e autorizado pelo servidor.
- Painel da recepção com contas individuais, sessão e proteção CSRF.
- Respostas humanas e automáticas no mesmo histórico web.
- Arquivos da interface servidos pela aplicação; não depender de CDN para o visual.
- Nenhum botão público de reset global ou conta administrativa com senha padrão.
- WhatsApp, agenda real e lembretes como evoluções.

## 6. Condições da primeira etapa - notas internas

**Restrição interna:** manter a futura demonstração nos planos gratuitos e sem contratação paga, respeitando as franquias dos serviços. Não ativar upgrade, cadastrar cartão ou aceitar cobrança para concluir a etapa. Se o serviço exigir pagamento, revisar a alternativa antes de prosseguir.

Esta restrição e os detalhes financeiros não devem aparecer nos slides. O material de apresentação deve descrever funcionalidades e arquitetura.

As fontes consultadas em 26/10/2026 indicam: Render Free com suspensão por inatividade e limites mensais; Neon Free com franquias e sem cartão na página de planos. As condições devem ser revistas antes da configuração. Exceder uma franquia pode indisponibilizar o teste; não prometer disponibilidade ilimitada. Detalhes e fontes em [hospedagem](11-hospedagem-e-publicacao.md).

Não escolher o PostgreSQL gratuito do Render como persistência duradoura da apresentação: a documentação consultada informa expiração após 30 dias. Não usar um período promocional como requisito de funcionamento.

## 7. Versões e alternativas

Manter Python/FastAPI/Jinja2, sem trocar de linguagem ou framework para publicar. Resolver releases estáveis compatíveis e registrar patches no lockfile durante a implementação.

PostgreSQL deve usar uma major suportada pelo provedor e pelos testes. SQLite permanece opcional para ensaios locais, com migração e validação explícitas se houver dados a transferir.

A escolha de outro host exige conferir runtime Python, HTTPS, persistência, limites e compatibilidade de conexão. Apenas hospedar arquivos estáticos não atenderia ao histórico compartilhado com a recepção.

## 8. Estado da preparação interrompida - uso interno

Uma preparação anterior criou um cadastro Sites privado e vazio, sem versões e sem publicação. O identificador é `appgprj_6adf4685de208191b2a5d5d147337583`. A configuração de recepção foi removida e a estrutura local de aplicação foi descartada. O conector disponível não oferece exclusão desse cadastro.

Esse registro não é a hospedagem escolhida neste plano e não representa um protótipo funcional. Não criar outro recurso nem retomar sua configuração sem solicitação de implementação.
