# Visão e escopo

[Voltar ao README](../README.md) · [Funcionalidades](03-funcionalidades.md) · [Arquitetura](05-arquitetura.md)

## 1. Identificação

| Campo | Definição |
| --- | --- |
| Nome de trabalho | Chatbot Odonto |
| Tema | Atendimento inicial, encaminhamento e pré-agendamento para clínica odontológica |
| Dupla definida no documento | Daniel Galvão Magalhães da Silva e Hosana Azevedo Pires |
| Público | Pacientes atuais e potenciais, aproximadamente entre 12 e 70 anos, com diferentes níveis de familiaridade digital |
| Canal da primeira etapa | Chat web demonstrativo acessível por link público, acompanhado de painel restrito da recepção |
| Canal pretendido para evolução | WhatsApp |
| Estágio do projeto | Planejamento e documentação; nenhuma funcionalidade está implementada |
| Referência de origem | PDF “Definições para Chatbot (1).pdf”, páginas 1 a 4, fornecido pelo usuário |

## 2. Problema e objetivo

A recepção precisa responder repetidamente às mesmas perguntas, coletar informações para marcar consultas e reconhecer quando um atendimento precisa de atenção da equipe. O chatbot pretende organizar esse primeiro contato e reduzir a digitação manual.

A primeira etapa planejada será um **protótipo web para apresentação e testes por link público**. O chat poderá ser aberto por diferentes pessoas, navegadores e dispositivos, sem instalação. Um painel separado e autenticado permitirá à dupla representar a recepção. As interfaces compartilharão o mesmo motor de regras e um banco persistente, com cada visitante limitado às suas próprias conversas.

O resultado esperado da primeira versão é uma solicitação fictícia estruturada ou uma dúvida respondida na demonstração. Uma solicitação de pré-agendamento **não é uma consulta confirmada**. A pessoa que representa a recepção poderá analisar o pedido e registrar um resultado simulado no painel; isso não altera uma agenda real.

O bot responderá às informações cadastradas enquanto o ambiente publicado estiver disponível. Os cenários de recepção disponível e fora do expediente serão demonstrativos. O link não significará atendimento clínico real nem recepção acompanhando continuamente; quando ninguém estiver supervisionando, a interface informará essa condição sem prometer retorno humano.

## 3. Requisito original e simplificação proposta

O PDF descreve o produto completo. Para a primeira entrega, a proposta é separar uma versão mínima funcional demonstrável, chamada MVP, das evoluções que exigem integração e operação adicionais.

| Tema do documento original | Primeira versão proposta | Evolução |
| --- | --- | --- |
| Identificação por CPF ou telefone | Nome e contato fictícios no chat web; acesso à conversa vinculado à sessão do visitante | Integração com cadastro e procedimento de verificação antes de consultar informações pessoais |
| Agendamento com horários disponíveis | Coleta de preferência de data e turno, com confirmação humana | Consulta de disponibilidade e reserva integrada à agenda |
| Remarcação e cancelamento | Registro de solicitação pendente para a recepção | Alteração da agenda com validações e registro da operação |
| Triagem de sintomas | Roteiro fixo de encaminhamento, validado pelo dentista | Ajustes do roteiro com supervisão profissional e acompanhamento da operação |
| Dúvidas frequentes | Base fixa revisada pela clínica | Gestão de versões e manutenção pelo painel |
| Atendimento humano | Fila no painel restrito, resumo, prioridade e resposta manual exibida somente no chat do visitante correspondente | Integração com canal real, distribuição entre atendentes e controles operacionais adicionais |
| Lembretes de 24 e 2 horas | Fora da primeira versão | Disparos vinculados a consultas confirmadas e à integração futura |
| Orientações sobre procedimentos | Fora da primeira versão | Envio de textos e arquivos aprovados pelo dentista |
| Métricas | Contagens do fluxo, solicitações, encaminhamentos e satisfação | Comparação de faltas e tempos após integração com consultas reais |

Essas simplificações definem a etapa de apresentação. O canal pretendido no produto completo continua sendo o WhatsApp; seus pré-requisitos e regras pertencem exclusivamente à [fase futura de integração](07-integracao-whatsapp.md).

## 4. Dentro do MVP

- Chat web acessível por link público e painel da recepção protegido por autenticação.
- Conversas isoladas por visitante, inclusive quando várias pessoas testarem ao mesmo tempo.
- Mesmo motor de fluxo para as mensagens do chat e para as ações da recepção.
- Saudação, menu numerado e retorno ao início.
- Respostas sobre localização, funcionamento, convênios e pagamentos.
- Registro de pré-agendamento, remarcação e cancelamento.
- Perguntas fixas para encaminhar relatos de dor ou ocorrências à equipe.
- Encaminhamento por escolha do paciente ou por duas entradas inválidas consecutivas.
- Fila de atendimento humano, com histórico mínimo necessário e pausa do bot enquanto a recepção atende.
- Tratamento explícito de solicitações fora do expediente.
- Avaliação de satisfação de 1 a 5 ao encerrar o atendimento.
- Persistência dos registros e indicadores operacionais básicos.

## 5. Fora do MVP

- Diagnóstico, prescrição ou recomendações clínicas geradas livremente.
- Reserva automática de horários ou alteração autônoma da agenda.
- Consulta de prontuário ou exposição de dados de consultas apenas com um número de telefone.
- Cobrança, pagamento no chat ou cálculo automático do valor de tratamento.
- Integração com WhatsApp, agenda ou cadastro reais.
- IA generativa, busca vetorial ou treinamento de modelos.
- Aplicativo móvel e site institucional separados do chat web de demonstração.
- Interpretação automática de áudios, imagens e documentos. A entrada não suportada recebe orientação para texto ou encaminhamento humano.
- Campanhas, lembretes e envio automático de materiais clínicos nesta etapa.

## 6. Linguagem e experiência

As mensagens deverão ser curtas, acolhedoras e diretas. Cada estado solicitará uma informação por vez. As opções numeradas funcionarão no chat web, sem depender de recursos do WhatsApp. A tela identificará o ambiente como demonstração e solicitará dados fictícios.

Palavras como `menu` e `recepção` devem ter comportamento previsível. O roteiro deve evitar termos técnicos e nunca afirmar que o bot diagnosticou uma emergência.

O público planejado inclui adolescentes. Na apresentação serão usados apenas personagens e dados fictícios. O procedimento de atendimento e de participação de responsáveis deverá ser definido pela clínica antes de uma eventual etapa com dados reais.

## 7. Premissas e dependências

1. A apresentação e os testes por link utilizarão somente dados fictícios e mensagens identificadas como demonstração.
2. Os visitantes poderão representar pacientes de teste. Apenas integrantes autorizados da dupla representarão a recepção no painel. Um ticket criado não equivale a uma pessoa já atendendo.
3. A primeira versão terá uma aplicação única com Python/FastAPI, SQLAlchemy, Alembic e páginas Jinja2. A hospedagem proposta será Render, com PostgreSQL Neon no ambiente publicado e no desenvolvimento integrado, conforme a [arquitetura](05-arquitetura.md).
4. O chat será planejado para acesso por HTTPS em navegadores e dispositivos diferentes. Uma sessão autorizará somente as conversas do próprio visitante; conhecer um identificador não concederá acesso a outro histórico.
5. O banco persistente armazenará os dados de demonstração entre reinícios e publicações. SQLite poderá ser usado apenas em ensaio local opcional, sem substituir a homologação com PostgreSQL. O prazo de inatividade e o procedimento administrativo de preparação da massa serão definidos antes dos testes; não haverá reset global disponível ao público.
6. Os dados da clínica no PDF são conteúdo fornecido para o projeto; endereço, horários e condições comerciais serão tratados como conteúdo de exemplo. Uma operação real dependeria de validação da clínica e dos controles em [Dados e segurança](06-dados-e-seguranca.md).
7. O roteiro de encaminhamento será apresentado como uma simulação administrativa, sem diagnóstico nem orientação clínica. Antes de qualquer uso real, a clínica e o dentista responsável deverão aprovar seus textos.
8. A configuração da hospedagem, a publicação e a emissão do link ocorrerão somente na etapa de implementação. Este planejamento não representa uma aplicação implementada ou um ambiente já publicado.

## 8. Entrega e definição de pronto

A entrega de implementação será considerada pronta para demonstração quando os quatro caminhos funcionarem em um link público HTTPS, pessoas em sessões diferentes puderem testar sem acessar dados umas das outras, os pedidos persistirem no PostgreSQL e a recepção autorizada puder assumir e concluir um atendimento. Os casos de erro, indisponibilidade humana e retomada após reinício também deverão ser verificados.

A demonstração será apresentada como um protótipo web com dados e decisões simulados. A conclusão dessa etapa não representará integração real com WhatsApp nem operação da clínica. Neste momento, somente a documentação está sendo elaborada; não existe link de teste emitido.

Os critérios detalhados estão em [Funcionalidades](03-funcionalidades.md), [Testes e métricas](09-testes-e-metricas.md) e [Plano de implementação](08-plano-de-implementacao.md).

## 9. Decisões que precisam de validação antes do uso real

| Decisão | Responsável sugerido |
| --- | --- |
| Horário real da recepção e prazo de retorno informado nas mensagens | Clínica/recepção |
| Texto de contingência para relatos prioritários fora do expediente | Dentista responsável |
| Endereço, convênios, pagamentos e condições de parcelamento | Clínica/recepção |
| Dados estritamente necessários para identificar o pedido | Clínica e equipe do projeto |
| Tratamento de pacientes menores e participação de responsáveis | Clínica |
| Condições da integração futura do canal, documentadas na [fase 07](07-integracao-whatsapp.md) | Clínica e equipe técnica |

## Referência

Fonte interna: PDF “Definições para Chatbot (1).pdf”, especialmente páginas 1 e 2 para tema, público e funcionalidades; páginas 3 e 4 para base e métricas. Consulte [Referências](10-referencias.md) para a relação de fontes do projeto.
