# Chatbot para clínica odontológica

Planejamento de um assistente de atendimento inicial, com respostas fixas, pré-agendamento e encaminhamento para a recepção.

**Situação atual:** somente planejamento e documentação. Não existe aplicativo implementado nem link funcional de teste. A implementação e a publicação serão etapas futuras, iniciadas apenas quando solicitadas.

## Objetivo e acesso previsto

A futura demonstração será um **chat web acessível por link público**, para que outras pessoas possam testar pelo computador ou celular. Cada visitante terá sua própria conversa; o painel da recepção terá acesso restrito.

O sistema demonstrará respostas frequentes com os dados administrativos da clínica confirmados pelo usuário. Nomes, contatos, relatos, pedidos e decisões dos testadores continuarão fictícios. Pré-agendamento significa solicitação pendente de análise, sem reserva real.

O planejamento parte de **Definições para Chatbot (1).pdf**, da atividade final de Daniel Galvão Magalhães da Silva e Hosana Azevedo Pires. O WhatsApp continua como evolução; a primeira publicação será do chat web.

Horários da recepção, endereço completo, convênios e formas de pagamento já foram confirmados pelo usuário e estão registrados na [base de conhecimento](docs/03-funcionalidades.md#base-de-conhecimento-e-publicação). O prazo máximo de retorno e as exceções de expediente ainda não foram definidos.

## Funcionalidades planejadas

- Menu com consultas, relato de problema, informações e recepção.
- FAQ de localização, horários, convênios e pagamentos.
- Solicitações de agendamento, remarcação e cancelamento.
- Roteiro fixo de encaminhamento prioritário, sem diagnóstico.
- Painel restrito para assumir, responder e encerrar conversas.
- Pausa do bot durante atendimento humano e avaliação de satisfação.
- Sessões separadas por visitante, persistência e histórico autorizado.
- Uso em navegadores e dispositivos diferentes, sem depender da máquina da apresentação.

## Stack proposta

| Parte | Escolha | Finalidade |
|---|---|---|
| Linguagem | Python 3.14.x | Backend e regras. |
| Aplicação | FastAPI + Uvicorn | Chat web, painel e rotas HTTP. |
| Validação | Pydantic + pydantic-settings | Entradas e configuração. |
| Dados publicados | PostgreSQL no Neon + SQLAlchemy 2.x + Psycopg 3 | Persistência entre reinícios da hospedagem. |
| Migrações | Alembic | Versionar o esquema. |
| Interface | Jinja2, HTML, CSS e JavaScript simples | Interface responsiva e atualização das mensagens. |
| Hospedagem proposta | Render | Aplicação acessível por HTTPS e subdomínio do provedor. |
| Qualidade | pytest, HTTPX e Ruff | Testes e padronização. |
| Desenvolvimento | uv e Git | Dependências e histórico. |

As escolhas são propostas, sem instalação ou conta configurada para essa arquitetura. Versões exatas serão fixadas na implementação. As condições de infraestrutura ficam nas notas técnicas de [ferramentas](docs/02-ferramentas-e-frameworks.md) e [hospedagem](docs/11-hospedagem-e-publicacao.md).

## Arquitetura prevista

```text
Visitantes em computador/celular       Recepção autorizada
                |                              |
                +---------- HTTPS ------------+
                                |
                   FastAPI + Jinja2 no Render
                                |
                   Regras e serviços de conversa
                                |
                     SQLAlchemy + Psycopg
                                |
                     PostgreSQL no Neon
```

Uma aplicação única, sem worker ou serviço de IA no MVP. Sessões, mensagens e pedidos ficam no banco; não dependem da memória ou do disco temporário da hospedagem. O acesso público ao chat não libera o painel ou os históricos dos outros visitantes.

## Documentação

| Documento | Conteúdo |
|---|---|
| [01 - Visão e escopo](docs/01-visao-e-escopo.md) | Problema, público, demonstração e evoluções. |
| [02 - Ferramentas e frameworks](docs/02-ferramentas-e-frameworks.md) | Stack e decisões técnicas internas. |
| [03 - Funcionalidades](docs/03-funcionalidades.md) | Requisitos e critérios de aceite. |
| [04 - Fluxos de conversa](docs/04-fluxos-de-conversa.md) | Menus, estados e mensagens. |
| [05 - Arquitetura](docs/05-arquitetura.md) | Componentes e processamento. |
| [06 - Dados e segurança](docs/06-dados-e-seguranca.md) | Modelo, sessões e acesso restrito. |
| [07 - Integração WhatsApp futura](docs/07-integracao-whatsapp.md) | Evolução de canal. |
| [08 - Plano de implementação](docs/08-plano-de-implementacao.md) | Fases futuras e roteiro de demonstração. |
| [09 - Testes e métricas](docs/09-testes-e-metricas.md) | Validação prevista para o link público. |
| [10 - Referências](docs/10-referencias.md) | Documento de origem e fontes oficiais. |
| [11 - Hospedagem e publicação](docs/11-hospedagem-e-publicacao.md) | Proposta de infraestrutura e passos futuros. |

## Próximas etapas previstas

1. Revisar e validar o planejamento.
2. Quando solicitado, implementar chat, regras, banco e painel.
3. Testar isolamento de visitantes e acesso da recepção.
4. Preparar hospedagem, configurar variáveis e aplicar migrações.
5. Publicar e verificar o link em outro dispositivo.
6. Compartilhar o endereço de teste com os participantes.

Nenhuma dessas etapas de implementação ou publicação será executada nesta entrega documental. Consulta às referências: **26/10/2026**.
