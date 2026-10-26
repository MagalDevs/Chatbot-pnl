# Plano de implementação e publicação futuras

[Voltar ao README](../README.md) | [Ferramentas](02-ferramentas-e-frameworks.md) | [Hospedagem](11-hospedagem-e-publicacao.md) | [Testes](09-testes-e-metricas.md)

## 1. Situação e limite desta entrega

A entrega atual contém planejamento. Não há protótipo implementado, banco publicado ou link funcional para compartilhar. Implementação, cadastro dos serviços e publicação só devem começar quando solicitados.

O alvo futuro é uma demonstração web acessível por link, em vários navegadores/dispositivos, com conversas isoladas e painel restrito.

## 2. Fases previstas

| Fase | Trabalho futuro | Entrega verificável |
|---|---|---|
| 1. Validar | Rever dados fictícios, fluxos e operação da recepção. | Escopo de demonstração fechado. |
| 2. Preparar projeto | Criar Python/FastAPI, dependências, PostgreSQL de teste e migrações. | Ambiente reproduzível e versão das dependências registrada. |
| 3. Implementar conversa | Chat, menu, FAQ, pedidos, resumo e estados. | Caminhos principais funcionando. |
| 4. Implementar acesso | Sessões por visitante, painel autenticado, CSRF e limites. | Isolamento e encaminhamento corretos. |
| 5. Validar | Executar testes de regras, dados, concorrência e permissões. | Evidências dos critérios atendidos. |
| 6. Preparar hospedagem | Configurar Render e Neon, variáveis e migrações. | Ambiente pronto, sem apagar dados a cada início. |
| 7. Publicar e conferir | Disponibilizar aplicação e testar HTTPS por outra rede/dispositivo. | Link funcional, conversa isolada e painel restrito. |
| 8. Demonstrar | Compartilhar link, acompanhar testes e registrar feedback. | Participantes conseguem testar de forma independente. |

A fase 6 é uma atividade futura, não configuração a realizar durante este planejamento. O [documento de hospedagem](11-hospedagem-e-publicacao.md) orienta a escolha e seus limites.

## 3. Cronograma sugerido

Estimativa de 10 dias úteis, a revisar conforme experiência da dupla e disponibilidade de contas.

| Dias | Foco |
|---|---|
| 1 e 2 | Validar escopo, preparar projeto e banco de testes. |
| 3 e 4 | Chat, FAQ, pedidos e persistência. |
| 5 e 6 | Recepção, sessões, isolamento e atualização do histórico. |
| 7 e 8 | Testes e preparação futura da hospedagem. |
| 9 | Publicação de teste e conferência externa. |
| 10 | Correções, documentação e ensaio. |

Conta ou recurso externo indisponível exige revisar o cronograma; uma execução em localhost não comprova a publicação.

## 4. Estrutura a implementar

Exemplo documental; não criar estes arquivos nesta etapa.

```text
chatbot-odonto/
  README.md
  docs/
  pyproject.toml
  uv.lock
  .env.example
  .gitignore
  alembic.ini
  migrations/
  config/
    clinic.json
    faq.json
  app/
    main.py
    api/
    channels/web.py
    conversation/
    requests/
    handover/
    persistence/
    security/
    templates/
    static/
  tests/
    unit/
    integration/
    scenarios/
```

Manter regras fora das rotas e um único modelo de persistência. Worker, integração Meta e frontend separado não são necessários ao escopo.

## 5. Checklist futuro de publicação

1. Concluir implementação e testes locais com banco separado.
2. Criar/configurar as contas dos serviços escolhidos, somente quando autorizada a implementação.
3. Criar banco persistente e guardar sua conexão nas variáveis do host.
4. Conectar o repositório, escolher runtime e definir build/start adequados ao FastAPI.
5. Executar migrações revisadas; criar usuários restritos por procedimento administrativo.
6. Publicar com domínio do provedor e HTTPS.
7. Testar dois visitantes independentes e o login da recepção.
8. Reiniciar/republicar e verificar persistência.
9. Registrar o URL confirmado no README; só então compartilhar como link funcional.

Não incluir segredos no repositório, nem aplicar reset global no startup. Um rollback de aplicação não desfaz automaticamente migrações.

## 6. Roteiro da apresentação futura

| Passo | Demonstração | Evidência |
|---|---|---|
| 1 | Abrir o link em computador e celular. | Acesso externo responsivo. |
| 2 | Iniciar conversas separadas. | Um participante não vê o histórico do outro. |
| 3 | Consultar FAQ e criar pedido com correção do resumo. | Resposta e pedido pendente persistidos. |
| 4 | Entrar na recepção e assumir um caso. | Controle de acesso e bot pausado. |
| 5 | Responder e registrar decisão fictícia. | Mensagem aparece no chat correto. |
| 6 | Testar duas entradas inválidas e um relato prioritário. | Encaminhamento e destaque na fila. |
| 7 | Encerrar e avaliar. | Resultado e satisfação registrados. |

Usar dados fictícios e identificar o produto como demonstração. Não apresentar decisões como reservas clínicas reais. Informar quando não houver equipe acompanhando a fila.

## 7. Critérios futuros de pronto

- Link HTTPS testado em rede/dispositivo diferente.
- Fluxos RF01–RF12 disponíveis.
- Persistência após reinício/redeploy.
- Sessões isoladas e recepção protegida.
- Repetição de envio sem pedido duplicado.
- Sem reset global público ou credencial padrão.
- Estados de erro/carregamento e retomada segura após inatividade.
- Fontes de dados fictícias e documentação atualizadas com o que foi implementado.

Nenhum critério está declarado aprovado nesta entrega. Os casos estão em [testes e métricas](09-testes-e-metricas.md).

## 8. Evoluções

Depois da demonstração, avaliar integração WhatsApp, agenda, lembretes e materiais profissionais. Essas etapas não são necessárias para planejar ou testar o chat web.
