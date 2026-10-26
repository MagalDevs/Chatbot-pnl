# Arquitetura planejada para testes públicos

[Voltar ao README](../README.md) | [Ferramentas](02-ferramentas-e-frameworks.md) | [Dados](06-dados-e-seguranca.md) | [Hospedagem](11-hospedagem-e-publicacao.md)

## 1. Estilo e limites

Planejar um monólito modular Python/FastAPI/Jinja2, com PostgreSQL persistente. O chat será público por HTTPS; a recepção terá painel restrito. Visitantes poderão utilizar computadores e celulares diferentes.

Nada está implementado ou publicado. Os nomes de módulos, rotas e serviços abaixo são especificações futuras.

## 2. Componentes

```text
Visitante A         Visitante B         Recepção autorizada
     |                   |                      |
     +---------------- HTTPS -------------------+
                          |
                FastAPI + Jinja2 (Render)
                          |
             Motor de regras e serviços
             - FAQ e etapas
             - pedidos administrativos
             - fila humana e permissões
                          |
                SQLAlchemy + Psycopg
                          |
                PostgreSQL persistente
                        (Neon)
```

| Módulo futuro | Responsabilidade |
|---|---|
| `api` | Rotas web e validação. |
| `channels/web` | Entrada e apresentação das mensagens do chat. |
| `conversation` | Menu, estados, contador de falhas e retomada. |
| `requests` | Pedidos pendentes e decisões fictícias da demonstração. |
| `handover` | Fila, atribuição e pausa do bot. |
| `content` | Textos fixos e FAQ versionados. |
| `persistence` | Modelos e transações PostgreSQL. |
| `security` | Sessões, autorização e CSRF. |
| `templates/static` | Interface responsiva e arquivos visuais. |

Não há LLM, worker ou integração Meta no MVP. O servidor processa uma mensagem curta por requisição; a interface consulta novidades por polling limitado, pausado quando a página fica inativa.

## 3. Caminho de uma mensagem

1. O navegador envia texto e `client_message_id`, acompanhado da sessão e proteção da ação.
2. A API confere limites e autorização para a conversa. Um ID na URL não concede acesso.
3. Verifica se o envio já existe. Mesmo ID/conteúdo retorna o resultado existente; conteúdo diferente com o mesmo ID gera conflito.
4. Uma transação curta lê a conversa com controle de concorrência.
5. Em `bot_active`, aplica as regras. Em modo humano, registra a fala sem executar menus.
6. Grava entrada, novo estado, pedido quando aplicável, resposta e eventos necessários na mesma transação.
7. Após o commit, devolve o resultado. O polling apresenta novidades sem duplicar mensagens na tela.

Uma falha não pode produzir confirmação de registro. A repetição após interrupção de rede reutiliza a chave original.

## 4. Persistência e concorrência

Conversas, sessões e pedidos ficam no PostgreSQL, não na memória do processo nem no filesystem do host. Reinício ou nova publicação não deve apagar dados ou trocar o segredo de configuração.

Usar uma sessão SQLAlchemy por operação e transações curtas, sem compartilhar sessões entre requisições. Empregar versão da conversa e atualização condicional ou bloqueio curto por linha para coordenar bot, visitante e recepção.

As restrições únicas evitam duplicação por envio e confirmação lógica. Duas tentativas de assumir a conversa não podem sobrescrever silenciosamente o responsável.

Usar pool pequeno e limites de conexão adequados à hospedagem/banco. Não abrir uma conexão ilimitada por visitante nem manter transações abertas aguardando interação humana. Referências: [sessões SQLAlchemy](https://docs.sqlalchemy.org/en/20/orm/session_basics.html) e [bloqueios PostgreSQL](https://www.postgresql.org/docs/18/explicit-locking.html).

SQLite opcionalmente ajuda um ensaio local; não é o banco de homologação desse desenho. O [disco efêmero do Render](https://render.com/docs/free) não servirá como persistência.

## 5. Encaminhamento e solicitações

Modos: `bot_active`, `waiting_human`, `human_active` e `closed`. A etapa de pergunta é separada do modo.

Pedido confirmado pelo visitante ou encaminhamento cria pendência no painel e passa a conversa para `waiting_human`. O bot envia o comprovante permitido pela transição e pausa menus. A recepção assume por ação autenticada e responde pelo mesmo histórico.

O usuário deve saber se a demonstração não está sendo acompanhada naquele momento. Não prometer resposta humana imediata. Encerrar ou devolver ao bot exige ação explícita da recepção.

Pedidos nascem em `pending_review`. Decisões da demonstração não reservam horário em agenda clínica real.

## 6. Rotas conceituais

| Rota | Finalidade e limite |
|---|---|
| `GET /chat` | Interface pública identificada como demonstração. |
| `POST /api/conversations` | Criar conversa vinculada à sessão do visitante. |
| `POST /api/conversations/{id}/messages` | Escrita idempotente apenas na conversa autorizada. |
| `GET /api/conversations/{id}/messages` | Histórico autorizado, com paginação/limite. |
| `GET/POST /login`, `POST /logout` | Acesso da recepção. |
| `GET /reception/conversations` | Fila restrita. |
| `POST /reception/conversations/{id}/claim`, `/reply`, `/close`, `/return-to-bot` | Ações autenticadas, autorizadas e protegidas contra CSRF. |
| `POST /reception/requests/{id}/decision` | Resultado fictício do pedido. |
| `GET /health` | Verificação simples sem segredos. |

Não criar endpoint público de reset global, exportação de históricos ou cadastro administrativo. “Nova conversa” só afeta o contexto do próprio visitante.

## 7. Ambientes

No desenvolvimento, executar em loopback e usar banco de testes separado. Na publicação futura, escutar na interface/porta esperadas pelo host e servir chat/painel pelo mesmo domínio HTTPS.

A aplicação dependerá de internet para o público. Uma demonstração local pode servir de ensaio, mas não substitui o teste do link em outra rede/dispositivo.

Prever estado de carregamento e repetição segura após suspensão por inatividade. Cold start não é confirmação de falha do pedido; consultar o resultado antes de criar outro.

## 8. Publicação futura e evolução

A sequência será implementar, testar, configurar serviços/segredos, aplicar migrações, publicar e verificar acesso externo. Nenhuma dessas ações está sendo executada agora. O [plano de hospedagem](11-hospedagem-e-publicacao.md) descreve os passos futuros.

WhatsApp, agenda e lembretes exigirão novos contratos e testes. Só reavaliar filas duráveis/worker quando essa evolução justificar.
