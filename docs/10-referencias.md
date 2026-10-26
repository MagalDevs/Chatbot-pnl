# Referências e decisões de pesquisa

[Voltar ao README](../README.md)

Data da consulta técnica: **26/10/2026**. Os links abaixo são fontes oficiais dos projetos ou orientações técnicas primárias. O planejamento prevê chat web público, FastAPI, Render e PostgreSQL no Neon. As referências da Meta são exclusivas da evolução de canal. Não há implementação ou publicação em andamento.

## 1. Documento de origem

**Definições para Chatbot (1).pdf**, fornecido pelo usuário, com quatro páginas. Local de consulta: `C:\Users\danie\Downloads\Definições para Chatbot (1).pdf`. O PDF não foi copiado para o repositório e esse caminho descreve apenas o ambiente original.

| Páginas | Conteúdo utilizado |
|---|---|
| 1 | Dupla, tema, objetivos, público, WhatsApp e funcionalidades iniciais. |
| 2 e 3 | Fluxos, perguntas, prioridade e encaminhamento à recepção. |
| 3 e 4 | Base de perguntas e respostas da clínica. |
| 4 | Metas e indicadores propostos. |

O documento é uma fonte de requisitos. Seus exemplos não foram tratados como dados comerciais verificados, instruções clínicas aprovadas ou funcionalidades já implementadas. A separação entre requisitos originais e proposta de MVP está em [visão e escopo](01-visao-e-escopo.md).

## 2. Linguagem, API e interface

| Fonte | Utilidade |
|---|---|
| [Python: versões e suporte](https://devguide.python.org/versions/) | Conferir a linha base e manutenção do runtime. |
| [FastAPI: funcionalidades](https://fastapi.tiangolo.com/features/) | Framework HTTP, validação e contratos OpenAPI. |
| [FastAPI: templates](https://fastapi.tiangolo.com/advanced/templates/) | Integração de páginas renderizadas no servidor. |
| [FastAPI: tarefas em segundo plano](https://fastapi.tiangolo.com/tutorial/background-tasks/) | Referência de evolução; não é requisito do processamento síncrono do chat web. |
| [FastAPI: hash e autenticação](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/) | Referência para hash com pwdlib; o projeto propõe sessão no servidor em vez de exigir JWT. |
| [Pydantic](https://docs.pydantic.dev/latest/) | Validação dos dados nos limites da aplicação. |
| [Pydantic Settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/) | Configuração por ambiente. |
| [Jinja](https://jinja.palletsprojects.com/en/stable/) | Templates HTML e escape de conteúdo. |
| [HTTPX](https://www.python-httpx.org/) | Cliente HTTP com timeouts e uso em testes. |

## 3. Persistência e desenvolvimento

| Fonte | Utilidade |
|---|---|
| [SQLAlchemy: sessões](https://docs.sqlalchemy.org/en/20/orm/session_basics.html) | Escopo de sessão, transações e acesso concorrente. |
| [Python sqlite3](https://docs.python.org/3/library/sqlite3.html) | Referência opcional de ensaio local, sem homologar o banco publicado. |
| [SQLite: usos apropriados](https://www.sqlite.org/whentouse.html) | Limites da alternativa de ensaio. |
| [SQLite: transações](https://www.sqlite.org/lang_transaction.html) | Características distintas do PostgreSQL proposto. |
| [SQLAlchemy: SQLite](https://docs.sqlalchemy.org/en/20/dialects/sqlite.html) | Referência da alternativa local opcional. |
| [Psycopg 3](https://www.psycopg.org/psycopg3/docs/) | Driver proposto para PostgreSQL. |
| [Alembic](https://alembic.sqlalchemy.org/en/latest/) | Migrações versionadas. |
| [PostgreSQL: suporte](https://www.postgresql.org/support/versioning/) | Escolha de major suportada pelo provedor e testes. |
| [PostgreSQL: bloqueios](https://www.postgresql.org/docs/18/explicit-locking.html) | Coordenação de operações concorrentes por conversa. |
| [PostgreSQL: SELECT](https://www.postgresql.org/docs/18/sql-select.html) | Referência do banco; filas com `SKIP LOCKED` não são requisito do MVP. |
| [uv](https://docs.astral.sh/uv/) | Gestão de projetos e dependências Python. |
| [Ruff](https://docs.astral.sh/ruff/) | Lint e formatação. |
| [pytest](https://docs.pytest.org/en/stable/) | Testes e fixtures. |
| [Docker Compose](https://docs.docker.com/compose/) | Opção para preparar PostgreSQL de desenvolvimento, sem obrigatoriedade. |

## 4. WhatsApp Business Platform - evolução futura

| Fonte | Utilidade e condição de acesso |
|---|---|
| [Coleção oficial Meta no Postman](https://www.postman.com/meta/whatsapp-business-platform/collection/wlk6lh4/whatsapp-cloud-api) | Acessível; ativos, credenciais e exemplos da Cloud API. |
| [Developer Hub](https://whatsappbusiness.com/developers/developer-hub/) | Acessível; visão geral e recursos de teste. |
| [Política de mensagens](https://whatsappbusiness.com/policy/) | Acessível; janela, templates e caminho para atendimento humano. |
| [Funcionalidades da plataforma](https://whatsappbusiness.com/products/business-platform-features/) | Acessível; formatos e integração de atendimento. |
| [Exemplo Meta de validação de assinatura](https://github.com/fbsamples/whatsapp-api-examples/blob/main/signature-validation-with-webhooks-payloads/app.py) | Acessível; referência do handshake e HMAC, com simplificações a revisar. |
| [FAQ](https://whatsappbusiness.com/resources/faq/) | Acessível; cobrança com informação divergente da página de preços. |
| [Preços](https://whatsappbusiness.com/products/platform-pricing/) | Acessível; conferir condições efetivas junto à conta. |
| [Get Started canônico](https://developers.facebook.com/documentation/business-messaging/whatsapp/get-started) | Exigiu login na consulta; validar no cadastro real. |
| [Webhooks canônicos](https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/overview) | Acesso limitado; conferir novamente na implementação. |
| [Graph API Webhooks](https://developers.facebook.com/docs/graph-api/webhooks/getting-started/) | Acesso limitado por HTTP 429; não foi a única evidência usada no levantamento. |

Não foi adotado um prazo numérico de reentrega que não pôde ser confirmado em fonte acessível. As condições da conta serão revistas somente se a integração real for iniciada. As limitações da pesquisa e os requisitos futuros estão em [integração WhatsApp](07-integracao-whatsapp.md); nenhum deles bloqueia o chat web.

## 5. Segurança de aplicação

- [OWASP: gerenciamento de sessões](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html): proteção do acesso ao painel.
- [OWASP: prevenção de CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html): proteção das ações autenticadas por cookie.

Essas referências sustentam escolhas técnicas. O levantamento não estabelece diagnóstico, prescrição ou interpretação jurídica.

## 6. O que é decisão deste planejamento

FastAPI, monólito web, chat e painel Jinja2, PostgreSQL persistente, Render e Neon são escolhas propostas para este escopo. Não são exigências da Meta nem determinações do PDF. Worker e filas duráveis ficam para uma eventual evolução, sujeita a novo desenho e validação. Nada foi implementado ou publicado.

O cronograma é estimativo; as metas de 60% de automação e 40% de encaminhamento vêm do documento original e precisam ser confrontadas com uma demanda real em eventual uso futuro. A apresentação usa cenários fictícios, sem alegar eficácia clínica ou resultados operacionais medidos.

## 7. Hospedagem e acesso público planejados

| Fonte | Uso no levantamento |
|---|---|
| [Render: FastAPI](https://render.com/docs/deploy-fastapi) | Compatibilidade e configuração futura da aplicação. |
| [Render: plano gratuito e limitações](https://render.com/docs/free) | Inatividade, cotas, disco efêmero e expiração do banco gratuito. |
| [Render: TLS](https://render.com/docs/tls) | HTTPS no endereço do provedor. |
| [Render: primeira publicação](https://render.com/docs/your-first-deploy) | Etapas futuras de configuração. |
| [Render: FAQ](https://render.com/docs/faq) | Condições operacionais e de cobrança. |
| [Neon: planos](https://neon.com/pricing) | Opção de PostgreSQL e condições de conta. |
| [Neon: limites e comportamento dos planos](https://neon.com/docs/introduction/plans) | Persistência, suspensão e limites de uso. |

O [plano de hospedagem](11-hospedagem-e-publicacao.md) distingue os passos futuros das condições técnicas internas, que não devem integrar os slides.
