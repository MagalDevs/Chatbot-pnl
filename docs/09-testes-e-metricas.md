# Plano de testes e métricas

[Voltar ao README](../README.md) · [Funcionalidades](03-funcionalidades.md) · [Arquitetura](05-arquitetura.md) · [Dados e segurança](06-dados-e-seguranca.md) · [Canal simulado e evolução](07-integracao-whatsapp.md)

## 1. Situação desta entrega

Este arquivo define a validação planejada de um protótipo web para apresentação e testes por link público: Python, FastAPI, SQLAlchemy síncrono, PostgreSQL Neon e páginas Jinja2, com hospedagem proposta no Render. Uma aplicação atenderá ao chat demonstrativo e ao painel restrito da recepção. Visitantes poderão acessar por navegadores e dispositivos diferentes, com sessões isoladas e dados fictícios.

O repositório contém planejamento e documentação; a aplicação e os testes executáveis ainda não foram implementados, e nenhum ambiente foi publicado. Nenhum caso abaixo deverá ser apresentado como aprovado ou executado nesta etapa. A publicação web será parte da implementação futura; não haverá envio a números reais ou integração com WhatsApp no aceite atual planejado.

A validação da entrega documental verifica coerência entre requisitos, fluxos, arquitetura, ferramentas, métricas e links. A validação do software ocorrerá durante a implementação, com resultados registrados por caso de teste.

As metas de automação e encaminhamento são extraídas de *Definições para Chatbot (1).pdf*, página 4. As regras de medição detalhadas neste arquivo são propostas de implementação. A identificação das fontes está em [Referências](10-referencias.md).

## 2. Estratégia de validação do software

| Camada | O que verificar | Ferramentas previstas |
| --- | --- | --- |
| Regras e fluxos | Estado da conversa, menu, normalização de entradas, contador de falhas e regras administrativas. | pytest. |
| Integração do backend | Estado, pedidos, respostas, transações curtas e chaves únicas nos endpoints do chat. | pytest, TestClient/HTTPX e PostgreSQL isolado de testes, com SQLAlchemy síncrono. |
| Fluxo completo web | Chat, pedido persistido, fila e resposta humana pelo painel restrito. | Navegadores, perfis e dispositivos diferentes, com roteiro demonstrativo. |
| Acesso e isolamento | Sessões de visitantes, autorização por conversa, login da recepção e CSRF. | Clientes HTTP e sessões independentes; inspeção de cookies no navegador. |
| Persistência e recuperação | Reinício e nova publicação sem perda de pedidos, histórico ou modo humano. | Banco persistente, registros fictícios conhecidos e conferência antes/depois. |
| Publicação | Link HTTPS, inicialização após inatividade, configuração e falhas de conexão. | Ambiente de homologação com a configuração prevista para Render e Neon. |
| Qualidade estática | Formatação, imports e problemas detectáveis sem executar a aplicação. | Ruff, conforme a configuração do projeto. |

Usar PostgreSQL no desenvolvimento integrado, nos testes de transação e na homologação do ambiente publicado. Separar dados e credenciais de teste dos registros usados na apresentação; não executar rotinas destrutivas da suíte sobre o banco demonstrativo compartilhado. SQLite poderá servir a um ensaio local opcional, sem comprovar concorrência ou persistência na arquitetura publicada.

Usar nomes, contatos e relatos fictícios. O navegador se comunicará com a aplicação por HTTPS; o backend acessará o banco por conexão protegida. CSS e JavaScript deverão ser servidos pela própria aplicação, sem tokens de banco, credenciais da recepção ou chamadas à Meta no cliente. Não haverá requisito de executar o teste público sem internet.

## 3. Matriz funcional

Os IDs RF correspondem à [especificação de funcionalidades](03-funcionalidades.md). Os relatos e as regras de prioridade são exemplos fictícios para demonstrar encaminhamento, sem diagnóstico ou prescrição. Validação profissional será necessária antes de qualquer uso clínico real.

| Caso | Requisito | Cenário | Critério de aceite |
| --- | --- | --- | --- |
| F01 | RF01 — Boas-vindas e menu | Visitante abre o link e envia a primeira mensagem. | A interface identifica a demonstração, solicita dados fictícios e apresenta as quatro opções principais, em linguagem clara. |
| F02 | RF02 — Navegação e retomada | Paciente pede `menu` durante o preenchimento. | Retorna ao menu sem confirmar ou alterar uma consulta. O tratamento de um rascunho deve seguir a regra documentada no fluxo. |
| F03 | RF04 — Identificação mínima | Paciente fictício inicia pré-agendamento. | Solicita apenas os dados definidos para essa finalidade; permite revisão e não exige CPF no protótipo. |
| F04 | RF04 | Paciente informa dados incompletos ou inválidos. | Explica o campo a corrigir sem apagar os demais dados válidos ou revelar registros de outra pessoa. |
| F05 | RF03 — FAQ | Visitante consulta endereço, horários, convênio ou pagamentos. | Responde com os dados confirmados pelo usuário: R. Vinte e Quatro de Maio, 1474 - Centro, Indaiatuba - SP, 13339-315; segunda a sexta-feira, 08h–19h, e sábados, 08h–12h; sem atendimento por convênio, com nota fiscal para reembolso; PIX, débito, crédito em até 10x sem juros no carnê/cartão e dinheiro, com condições facilitadas para tratamentos completos. Não inventa preços, garantia de reembolso ou outras condições. |
| F06 | RF03 | Pergunta sobre preço não cadastrado ou condição não esclarecida. | Não inventa informação; oferece encaminhamento à recepção. |
| F07 | RF05 — Pré-agendamento | Paciente escolhe atendimento e preferência de dia/turno. | Exibe resumo para revisão e, somente após confirmação, registra uma única solicitação como `pending_review` (“Pendente de análise”), com identificador de acompanhamento. |
| F08 | RF05 | Paciente corrige ou desiste antes de enviar o pedido. | A correção aparece no resumo; a desistência não gera consulta ou solicitação concluída. |
| F09 | RF05 | Pedido é registrado sem integração com agenda real. | A resposta diz que a disponibilidade depende da recepção; não anuncia consulta confirmada nem horário garantido. |
| F10 | RF06 — Remarcação e cancelamento | Paciente pede remarcação ou cancelamento. | Registra o pedido para revisão humana, sem alterar automaticamente uma consulta existente. |
| F11 | RF06 | Chat tenta acessar pedido de outra conversa fictícia. | O endpoint restringe o acesso ao contexto da conversa; não entrega todos os pedidos ao chat do paciente. |
| F12 | RF07 — Dor ou ocorrência | Respostas correspondem ao roteiro fictício de prioridade. | Destaca o atendimento no painel, preserva o contexto e não emite diagnóstico ou prescrição. |
| F13 | RF07 e RF10 | Horário simulado fica fora do expediente ou não há supervisão em caso prioritário. | Informa indisponibilidade e apresenta o texto de demonstração, sem prometer transferência imediata ou retorno humano garantido. |
| F14 | RF08 — Atendimento humano | Paciente escolhe falar com a recepção. | A conversa aparece na caixa de entrada, com resumo e status de espera; o fluxo automático fica suspenso. |
| F15 | RF08 | Recepção autenticada assume e responde por outro navegador ou dispositivo. | A mensagem aparece somente no chat do visitante correspondente, com autoria e estado coerentes. O bot permanece pausado. |
| F16 | RF08 | Atendente encerra ou devolve a conversa ao bot. | A ação autorizada é explícita e registrada; a conversa assume o estado definido para a ação. |
| F17 | RF09 — Fallback | Uma entrada não é reconhecida. | Explica as opções válidas e incrementa o contador da etapa. Uma entrada válida zera o contador conforme a regra do fluxo. |
| F18 | RF09 | Duas entradas consecutivas não são reconhecidas. | Cria encaminhamento humano com o motivo registrado, preservando os dados já coletados. |
| F19 | RF11 — Satisfação e métricas | Paciente recebe a avaliação de atendimento. | Aceita uma nota inteira de 1 a 5; uma resposta inválida não se torna nota nem inicia ciclo infinito de perguntas. |
| F20 | RF11 | Paciente ignora a avaliação ou responde duas vezes. | A ausência não vira nota zero. A mesma avaliação não é contada duas vezes; uma eventual correção segue regra explícita. |
| F21 | RF02 | Há retomada após inatividade ou comando `menu` durante atendimento humano. | Aplica a política da sessão sem reapresentar dados desnecessários; não reativa o bot em conversa humana. |
| F22 | RF09 | Uma entrada de conteúdo não suportado é simulada. | Explica a limitação e orienta texto ou atendimento humano, sem afirmar que compreendeu um arquivo. Não é necessário implementar upload. |
| F23 | RF10 — Fora do expediente | Mensagem chega dentro ou fora dos intervalos confirmados, em seus limites, em exceção cadastrada ou sem integrante acompanhando o teste. | Usa `America/Sao_Paulo` e o horário base de segunda a sexta-feira, 08h–19h, e sábados, 08h–12h. Mantém FAQ e registro disponíveis; exibe o horário completo e o status de supervisão, sem prometer tempo de resposta ou supor regras de feriados ainda não informadas. |
| F24 | RF11 | Atendimento prioritário continua pendente ou eventos de resultados são registrados. | Não solicita nota durante a espera; FAQ resolvido, pedido registrado e consulta confirmada geram resultados distintos. |
| F25 | RF12 — Registro e acompanhamento | Aplicação reinicia depois da confirmação dos dados pelo paciente. | Preserva pedido e identificador, sem duplicação; a recepção distingue pendente de concluído. |
| F26 | RF12 | Recepção altera o resultado de um pedido. | Registra responsável e momento da ação; logs técnicos não expõem relato completo ou credenciais. |

## 4. Matriz técnica do protótipo público planejado

| Caso | Cenário | Critério de aceite |
| --- | --- | --- |
| T01 | Link aberto por navegadores, perfis e dispositivos diferentes. | O chat responde por HTTPS sem instalação; cada visitante recebe uma sessão própria. A recepção acessa o painel somente após login. |
| T02 | Interface e recursos da aplicação carregam pelo link. | CSS e JavaScript são servidos pela aplicação; o navegador não recebe credenciais do banco ou da recepção nem realiza chamadas à Meta. |
| T03 | Falha entre atualização do pedido e gravação da resposta. | Uma transação curta grava pedido, estado e resposta juntos ou desfaz tudo. A interface não informa sucesso antes do commit. |
| T04 | Clique duplo ou repetição HTTP com o mesmo `client_message_id` e conteúdo. | Restrição única e deduplicação retornam o resultado já registrado, sem nova resposta, pedido ou encaminhamento. |
| T05 | Mesmo `client_message_id` é reutilizado com conteúdo diferente. | Rejeita o conflito, preserva o registro anterior e orienta atualização da interface. |
| T06 | Confirmação do mesmo pedido é repetida com o mesmo identificador de solicitação. | Reutiliza o pedido existente e não cria uma segunda solicitação, mesmo quando o comando de confirmação é repetido. |
| T07 | Duas abas alteram simultaneamente a mesma conversa. | Escrita curta e controle de versão serializam as alterações; uma versão antiga é rejeitada ou relida sem sobrescrever estado mais recente. |
| T08 | Escritas próximas disputam a mesma conversa no PostgreSQL. | Transações curtas e versão da conversa evitam sobrescrita silenciosa. Falha ou conflito causa rollback e tratamento limitado, sem confirmar gravação inexistente. |
| T09 | Processo reinicia ou ocorre uma nova publicação. | Banco persistente preserva pedidos, histórico, notas e modo humano; não há seed destrutivo automático. Sessões ainda válidas seguem a política de retomada. |
| T10 | Recepção assume enquanto chega nova mensagem ao chat. | O estado persistido define a ordem das operações. Mensagens posteriores ao modo humano não geram decisão automática do bot. |
| T11 | Recepção repete o envio de uma resposta web. | A chave da operação evita duplicação e a mensagem fica visível somente na conversa autorizada, sem exigir worker ou plataforma de mensagens. |
| T12 | Visitante tenta resetar a base ou inicia outro teste próprio. | Não existe reset global público. Nova conversa não apaga pedidos anteriores nem afeta outros visitantes; preparação da massa é administrativa e restrita. |
| T13 | Conexão com banco indisponível ou erro de gravação. | Não apresenta pedido como registrado; faz rollback, trata a conexão e mostra erro recuperável. Logs não contêm URL com senha ou relato completo. |
| T14 | Acesso ao painel sem login, sessão encerrada ou ação sem token CSRF válido. | Rejeita acesso e mudanças não autorizadas. A sessão do paciente não permite assumir conversa ou registrar decisão da recepção. |
| T15 | Visitante A tenta consultar ou alterar a conversa e o pedido do visitante B. | GET e POST validam a sessão no servidor e recusam o acesso sem revelar histórico, inclusive quando os IDs são conhecidos. |
| T16 | Cookies e ações autenticadas no domínio publicado. | Cookies usam as proteções previstas para HTTPS; origem e CSRF são validados nas alterações. Logout e expiração revogam o acesso da recepção. |
| T17 | Primeira abertura ou envio após inatividade do host ou do banco. | A interface apresenta carregamento ou erro recuperável durante a inicialização, permite tentativa limitada e reutiliza o ID do envio, sem criar pedido duplicado. Não anuncia sucesso antes do commit. |
| T18 | Requisições excessivas ou mensagens acima dos limites configurados. | A aplicação aplica limites de tamanho/frequência e retorna resposta controlada, sem expor detalhes internos ou afetar a autorização das outras sessões. |
| T19 | Vários testadores executam os caminhos principais simultaneamente. | Mantém isolamento e estado coerente no volume definido para a apresentação; registra limitações observadas sem alegar capacidade não medida. |
| T20 | Publicação com configuração incompleta ou credencial padrão. | Não libera o painel com senha padrão pública; configuração obrigatória é validada. Segredos permanecem no backend e fora do repositório/arquivos servidos. |

As chamadas HTTP de vários visitantes poderão ocorrer próximas, mesmo com uma instância pequena. Usar uma sessão SQLAlchemy por operação, pool compatível com o banco e transações curtas. Um worker ou mecanismo de reserva de tarefas não será necessário para o fluxo síncrono planejado. O teste T17 verificará o comportamento de inicialização sem fixar um tempo de espera que não tenha sido medido.

## 5. Aceite da demonstração

- Caminhos RF01 a RF12 cobertos pela matriz F01 a F26 e resultados registrados quando o protótipo for implementado.
- Chat demonstrativo acessível por link HTTPS em navegadores e dispositivos diferentes, com painel restrito.
- Casos T01 a T20 conferidos com PostgreSQL e dados fictícios, incluindo autorização, concorrência, conexão e inicialização após inatividade.
- Sessões de visitantes isoladas; nenhum histórico ou pedido acessível apenas pela troca do identificador.
- Reinício e nova publicação preservam os dados. Não há reset global público, e a preparação da massa é administrativa.
- Diferença entre pedido registrado, pedido em análise e consulta confirmada visível nas mensagens e no painel.
- Indicadores identificam massa fictícia ou testes do protótipo, sem alegar desempenho em pacientes reais.
- Condição de recepção sem supervisão é informada sem promessa de retorno humano; iniciar outro teste afeta somente o próprio visitante.
- Resultados de publicação e testes somente serão registrados após sua execução futura. Nesta entrega não existe link emitido ou caso aprovado.

### Testes futuros, fora do aceite da apresentação

Caso seja aprovado um projeto de integração real, será necessário outro plano de validação para:

- Meta/WhatsApp: handshake, assinatura do webhook, reentregas, status de entrega, falhas externas, janela do canal e templates.
- Confiabilidade externa: inbox/outbox, worker, recuperação e tratamento de envios com resultado incerto, se essa arquitetura vier a ser adotada.
- Uso clínico: aprovação do conteúdo, tratamento de dados reais e operação efetiva da recepção.

Esses itens ficam fora da publicação web demonstrativa. Acesso remoto, autenticação, isolamento, PostgreSQL e persistência já integram o aceite atual planejado, sem depender da futura conexão com WhatsApp. Consulte o [plano de implementação](08-plano-de-implementacao.md) e o [canal simulado e sua evolução](07-integracao-whatsapp.md).

## 6. Regras de medição

### Unidade de análise

Registrar um identificador para cada episódio iniciado no chat simulado. O mesmo paciente fictício pode ter vários episódios. Repetir o mesmo `client_message_id` ou confirmar novamente o mesmo pedido não cria novo episódio nem multiplica contadores.

O episódio da etapa automática termina com um dos desfechos exclusivos:

- `resolved_by_bot`: dúvida ou tarefa elegível concluída sem ação da recepção.
- `handed_over`: demanda direcionada à equipe, inclusive pré-agendamento que depende de confirmação humana.
- `abandoned`: paciente fictício deixa de responder antes de um desfecho, segundo prazo de inatividade definido para a demonstração.
- `ended_unresolved`: encerramento explícito sem resolução ou falha técnica definitiva.

Um atendimento humano pode continuar aberto depois de o episódio automático terminar como `handed_over`. A data da transferência encerra a medição da etapa automática; a conclusão pela recepção pertence à medição operacional humana. O sistema não deve contar a transferência como resolução automática.

O prazo para classificar abandono deve ser fixado antes da demonstração, aplicado de forma consistente e informado nos relatórios. Até esse prazo, o episódio permanece `open`. Um relógio controlado no teste permite verificar a regra sem aguardar longos intervalos; isso não simula uma janela externa de mensagens.

Para um período de relatório, considerar os episódios encerrados naquele período. Exibir também a quantidade iniciada, ainda aberta e encerrada por abandono. Uma comparação só é válida quando usa a mesma regra de encerramento e o mesmo público elegível.

### Métricas do protótipo

| Indicador | Definição e denominador | Interpretação |
| --- | --- | --- |
| Taxa de automação | `resolved_by_bot / total de episódios automáticos encerrados × 100`. O denominador inclui encaminhados, abandonados e encerrados sem resolução. | Referência do PDF: pelo menos 60%, sujeita a avaliação futura. Não é resultado obtido nem condição de aprovação da apresentação. |
| Taxa de encaminhamento humano | `handed_over / total de episódios automáticos encerrados × 100`. | Referência do PDF: até 40%, sujeita a avaliação futura. Não limita transferências necessárias nem reprova a apresentação. |
| Taxa de abandono | `abandoned / total de episódios automáticos encerrados × 100`. | Mostra dificuldades ou interrupções do fluxo; deve acompanhar as duas taxas anteriores. |
| Episódios ainda abertos | Contagem de episódios sem desfecho no instante de corte do relatório. | Exibir separadamente; não tratá-los como sucesso ou falha antecipadamente. |
| Solicitações registradas | Contagem de IDs únicos de pedidos efetivamente persistidos, por tipo. | Medir registro, remarcação e cancelamento separadamente; não chamar esse número de consultas confirmadas. |
| Consultas confirmadas | Contagem de solicitações marcadas como confirmadas pela recepção fictícia no período. | É uma simulação administrativa; não representa reserva em agenda real. |
| Tempo de registro do pedido | Mediana e percentil 95 do tempo entre início do fluxo e persistência do pedido, entre pedidos concluídos. | Mostrar quantidade de pedidos medida; abandonos ficam em indicador próprio. |
| Tempo até atendimento humano | Mediana e percentil 95 entre transferência e primeira resposta humana, apenas para casos respondidos; pendentes aparecem à parte. | Separar dentro e fora do expediente para evitar comparação enganosa. |
| Falha de operação da aplicação | `operações de gravação encerradas com erro / operações de gravação encerradas × 100`. | Separar falhas induzidas e tentativas após inicialização; repetição da mesma operação não cria múltiplos resultados de negócio. Não representa entrega pelo WhatsApp. |

Automação e encaminhamento não somam necessariamente 100%, porque há abandonos e encerramentos sem resolução. Se todos os pré-agendamentos dependem da recepção, a composição das demandas pode tornar a referência de 60% inadequada; uma operação futura deverá reavaliá-la. Também é útil mostrar automação apenas do FAQ, com o denominador identificado, sem substituir silenciosamente a taxa geral.

Na apresentação, identificar se os indicadores vieram de massa fictícia preparada ou de sessões de teste do protótipo público. Não misturar esses grupos sem informar sua origem. Com denominador zero, mostrar “sem dados”, nunca 0% como se houvesse amostra. Tempos e notas dos testadores não demonstram desempenho clínico ou resultados com pacientes reais.

### Satisfação de 1 a 5: CSAT

Pergunta proposta: “De 1 a 5, qual é a sua nota para este atendimento?”

- Registrar uma avaliação válida por episódio e identificar se o objeto avaliado foi o bot ou o atendimento humano.
- Publicar a média das notas: `soma das notas válidas / quantidade de avaliações válidas`.
- Se publicar CSAT favorável, definir como `notas 4 ou 5 / avaliações válidas × 100`.
- Informar quantidade de respostas e taxa de resposta: `avaliações válidas / convites apresentados no chat × 100`.
- Ausência de resposta não equivale a nota zero. Convites que não foram persistidos/apresentados não entram no denominador.
- A escala de 1 a 5 aqui proposta é uma avaliação de satisfação; não deve ser denominada NPS.

## 7. Indicadores para evolução futura

| Indicador | Pré-requisito para medição confiável | Regra a definir |
| --- | --- | --- |
| Resolução no primeiro contato — FCR | Identificação consistente da demanda e registro de conclusão pela equipe. | Definir o que conta como mesma demanda e o período de observação de novo contato. Transferir para humano não comprova resolução nem reprova automaticamente FCR. |
| Taxa de faltas — no-show | Agenda real, consultas confirmadas e marcação de comparecimento. | `faltas sem cancelamento válido / consultas confirmadas elegíveis × 100`, com política para cancelamento e remarcação. |
| Redução de faltas | Histórico anterior comparável e lembretes em operação. | Comparar períodos com regras equivalentes; mostrar volume e mudança na composição das consultas. |
| Impacto dos lembretes | Registro de cada lembrete e seu status, associado à consulta. | Distinguir mensagem aceita, entregue e ação do paciente. Uma redução observada isoladamente não demonstra causalidade. |

Nenhuma dessas métricas é exigida como resultado real na demonstração. O projeto futuro deverá prever a coleta dos eventos necessários antes de relatar FCR, faltas ou impacto de lembretes.

## 8. Registro dos resultados futuros

Para cada execução, anotar: ID do caso, versão da aplicação, ambiente, massa fictícia utilizada, resultado esperado, resultado observado, data, responsável e evidência sem dados sensíveis. Resultados possíveis: `não executado`, `aprovado`, `reprovado` e `bloqueado`, com motivo.

Uma alteração de fluxo ou contrato deve reexecutar os casos afetados. Uma correção em deduplicação, estado, transação ou transbordo exige rever os cenários de concorrência e recuperação associados. Testes não relacionados só precisam ser repetidos quando houver efeito demonstrável ou risco identificado.

## 9. Conferência desta documentação

Na entrega atual, a revisão deve confirmar:

- Correspondência entre cada RF, seu fluxo e ao menos um critério verificável.
- Coerência das definições de solicitação, consulta confirmada, atendimento humano e prioridade.
- Tratamento explícito de dados fictícios, simulações e ausência de implementação.
- Denominadores identificados e distinção entre episódios abertos, encaminhados e abandonados.
- Links relativos válidos para os arquivos da documentação.
- Ausência de afirmações de testes, integração ou implantação já concluídos.

Essa conferência documental não substitui a execução futura da matriz de testes.
