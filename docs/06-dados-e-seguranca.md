# Dados, configuração e controle de acesso

[Voltar ao README](../README.md) | [Arquitetura](05-arquitetura.md) | [Hospedagem](11-hospedagem-e-publicacao.md)

## 1. Escopo planejado

A demonstração pública usará nomes, contatos, relatos, pedidos e decisões fictícios. O FAQ administrativo usará horário, endereço, convênio e pagamentos confirmados pelo usuário, conforme a [base de conhecimento](03-funcionalidades.md#base-de-conhecimento-e-publicação). Não será prontuário, não exigirá CPF e não consultará cadastro clínico. O visitante deve ser orientado a não informar dados pessoais ou relatos reais.

O modelo é conceitual. Não existem tabelas, migrações executadas ou banco remoto configurado. A proposta publicada utiliza PostgreSQL no Neon; SQLite opcionalmente serve para ensaios locais e não comprova o comportamento do banco publicado.

Usar identificadores internos não sequenciais, instantes em UTC e apresentação em `America/Sao_Paulo`. Datas preferidas para consulta são datas locais.

## 2. Entidades propostas

| Entidade | Campos principais | Regra |
|---|---|---|
| `contacts` | ID, nome fictício e contato fictício opcional | Personagem do cenário de teste. |
| `conversations` | contato, proprietário/sessão, modo, etapa, versão, falhas, prioridade, atendente e datas | Conversa autorizada por sessão, não apenas por ID informado. |
| `messages` | conversa, autor, texto, ordem, data e `client_message_id` | Restrição única por conversa e envio. |
| `appointment_requests` | conversa, tipo, preferência, referência fictícia, status, decisão, responsável e datas | Um pedido por confirmação lógica, sem reserva real. |
| `staff_users` | usuário, hash de senha, papel e ativo | Recepção autorizada, sem cadastro público. |
| `sessions` | hash de token opaco, tipo, usuário/conversa, validade e dados de proteção CSRF | Sessão persistente por visitante ou atendente. |
| `audit_events` | ator, ação, objeto, data e resultado | Atribuição e decisão administrativas. |
| `metric_events` | episódio, tipo, instante, chave única e nota opcional | Indicadores sem duplicação por repetição técnica. |

FAQ e dados administrativos confirmados da clínica podem ficar em JSON versionado, com registro de origem e revisão. Pessoas e pedidos da massa de teste continuarão fictícios. Segredos e bancos gerados não entram no Git.

## 3. Estados e integridade

Modos de conversa: `bot_active`, `waiting_human`, `human_active` e `closed`. A etapa da pergunta, como `CONSULTA_NOME`, permanece separada. Expiração do rascunho não encerra uma pendência humana.

Pedidos: `pending_review`, `needs_information`, `completed`, `declined` e `withdrawn`. Resultados como `appointment_confirmed`, `rescheduled` e `cancelled` serão decisões fictícias da recepção, sem alterar agenda real.

Salvar mensagem, estado, pedido e resposta em transação curta. Confirmar sucesso somente depois do commit. Mesmo envio/conteúdo retorna o resultado anterior; reutilizar a chave com outro conteúdo gera conflito.

Coordenar alterações por versão ou bloqueio curto da linha. Não manter sessão SQLAlchemy compartilhada entre requisições nem transação aguardando o paciente. Validar concorrência no PostgreSQL, inclusive duas pessoas tentando assumir o caso. Referência: [sessões SQLAlchemy](https://docs.sqlalchemy.org/en/20/orm/session_basics.html).

## 4. Sessões e isolamento dos visitantes

Gerar uma sessão opaca por visitante, com token aleatório de entropia adequada e registro persistido no servidor. Guardar apenas o identificador no cookie; não colocar histórico, nome ou permissão de recepção no navegador.

Cada leitura/escrita valida a associação entre sessão e conversa. Trocar o ID na URL não libera outro histórico. Não enviar uma lista global de pacientes ao cliente para filtrá-la apenas na interface.

“Nova conversa” cria um novo contexto do próprio visitante, sem apagar pedidos de outras pessoas. Não expor reset global, exportação administrativa ou alteração de papéis a visitantes.

Na versão HTTPS publicada, usar cookies `Secure`, `HttpOnly`, `SameSite` adequado e expiração. Um ajuste para HTTP de loopback deve ser exclusivo do desenvolvimento.

## 5. Recepção e proteção das ações

O chat público não libera o painel. Preparar contas individuais da dupla/recepção por procedimento administrativo restrito, senha com hash Argon2, sessão revogável e logout.

Exigir autorização no servidor e proteção CSRF nas ações. Não alterar dados por GET. Escapar textos do visitante nos templates e limitar tamanho/frequência de mensagens e tentativas de login. Não publicar senha padrão ou exigir credenciais pessoais para testar o chat.

Sem supervisão humana, a interface deve informar a condição da demonstração. A existência de uma fila não significa que alguém esteja atendendo.

Referências: [hash de senha FastAPI](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/), [sessões OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) e [CSRF OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).

## 6. Configuração futura

| Configuração | Uso |
|---|---|
| `APP_ENV` | Distinguir desenvolvimento, teste e demonstração publicada. |
| `APP_BASE_URL` | Origem HTTPS autorizada. |
| `DATABASE_URL` | Conexão PostgreSQL protegida, com credenciais no ambiente do host. |
| `SESSION_SECRET` | Segredo estável se necessário à implementação de sessão/CSRF; não regenerar a cada inicialização. |
| `TZ` | `America/Sao_Paulo` para apresentação dos horários. |
| `LOG_LEVEL` | Logs de IDs/códigos sem conteúdo completo de conversas. |
| Parâmetros de sessão e fluxo | Prazos de expiração, retomada e limites. |
| Arquivos JSON de cenário | FAQ administrativo confirmado e horário base da recepção; exceções quando informadas, separadas da massa fictícia de visitantes e pedidos. |

Os nomes são propostas documentais; não foram criados arquivos de ambiente ou valores reais. A porta/interface de execução seguirá a configuração da hospedagem. Não exigir variáveis da Meta.

O horário base confirmado é segunda a sexta-feira, das 08h às 19h, e sábados, das 08h às 12h, em `America/Sao_Paulo`. Prazo máximo de retorno, feriados e exceções ainda não foram informados; não preencher esses parâmetros com promessas presumidas.

Usar URL de conexão/SSL conforme o provedor, pool pequeno e conta de banco com permissões necessárias. Banco de testes e banco publicado devem ser separados.

## 7. Persistência, retenção e manutenção futuras

Reinícios e novas publicações devem preservar registros. O filesystem efêmero do host não servirá como banco, sessão ou backup. As migrações serão revisadas antes de aplicar; não executar reset ou seed destrutivo automaticamente no startup.

Preparar limpeza administrativa dos dados de demonstração, com acesso restrito e período de retenção definido antes de abrir o teste. O visitante pode iniciar nova conversa sem apagar históricos de terceiros. Prever backup/exportação administrativa compatível com o plano escolhido e testar a recuperação necessária.

Não guardar relatos completos ou credenciais nos logs. A coleta será fictícia; eventual uso real requer outra revisão de dados, conteúdo e operação.

## 8. Verificações previstas

- Visitantes diferentes não acessam os históricos uns dos outros.
- Visitante não assume conversa ou decide como recepção.
- Logout/expiração revogam acesso.
- Ações sem proteção válida são rejeitadas.
- Restart/redeploy preservam pedidos, mensagens e sessões válidas.
- Falha ou suspensão do host não produz confirmação falsa.
- Não existe reset global público.

Esses são critérios futuros, não testes já executados. Veja [testes e métricas](09-testes-e-metricas.md).
