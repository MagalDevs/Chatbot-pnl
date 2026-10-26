# Funcionalidades e critérios de aceite

[Voltar ao README](../README.md) · [Visão e escopo](01-visao-e-escopo.md) · [Fluxos](04-fluxos-de-conversa.md)

## Como ler este documento

Os requisitos abaixo descrevem o comportamento planejado para o MVP web de apresentação e testes por link público. Os identificadores `RF` permitem ligar implementação, testes e apresentação. Nenhuma função é declarada como implementada ou publicada.

O PDF de origem define o produto mais amplo. A primeira entrega planejada usará chat web público e painel restrito da recepção, com o mesmo motor de regras e dados fictícios persistidos no PostgreSQL. As pessoas poderão testar de navegadores ou dispositivos diferentes. Integrações com WhatsApp, agenda, cadastro e lembretes pertencerão às evoluções.

**Regra transversal da demonstração:** as mensagens serão exibidas no chat web do visitante correspondente, e somente a recepção autenticada poderá responder pelo painel. A sessão de cada visitante autorizará apenas suas próprias conversas, sem envio para números reais. Hospedagem web não equivalerá a integração com WhatsApp; as regras desse canal estarão exclusivamente na [fase futura de integração](07-integracao-whatsapp.md).

## Requisitos do MVP

### RF01 — Boas-vindas e menu principal

**Comportamento:** identificar o sistema como assistente virtual e mostrar quatro opções: pedidos de consulta, dor ou ocorrência, informações e recepção.

**Critérios de aceite:**

- Uma nova conversa recebe saudação e menu.
- A interface poderá ser aberta por link público HTTPS, sem instalação, e indicará que o teste utiliza dados fictícios.
- O usuário pode responder com número, mesmo sem botões.
- A opção de recepção permanece acessível durante o fluxo automatizado.
- Uma conversa já assumida por uma pessoa não é reiniciada pelo bot a cada mensagem.

### RF02 — Navegação e retomada

**Comportamento:** permitir retorno ao menu e encerrar uma coleta sem criar pedido incompleto.

**Critérios de aceite:**

- `menu` retorna ao início quando o bot está ativo.
- Voltar ao menu durante a coleta não cria uma solicitação finalizada.
- A retomada após inatividade segue o prazo e a política de sessão configurados antes da apresentação e não reapresenta informações sem necessidade.
- Iniciar um novo teste criará somente um novo episódio do próprio visitante, sem apagar pedidos persistidos ou afetar outros visitantes. A preparação global da massa será uma operação administrativa restrita.
- No atendimento humano, comandos de navegação não reativam o bot automaticamente.

### RF03 — Dúvidas frequentes

**Comportamento:** apresentar respostas fixas por categoria e oferecer próximo passo.

**Critérios de aceite:**

- As quatro categorias básicas são localização, funcionamento, convênios e pagamentos.
- A resposta sobre convênios segue a confirmação posterior do usuário: não há atendimento por convênio, mas há emissão de nota fiscal para reembolso.
- Endereço com CEP, horários e condições de pagamento devem corresponder aos textos confirmados na base ao final deste documento, incluindo até 10x sem juros no carnê/cartão.
- O bot não inventa valores de tratamento, regras de reembolso ou condições de parcelamento.
- Informação ausente oferece encaminhamento humano.
- A base identifica a origem e a data de confirmação administrativa. Endereço, horários, convênio e pagamentos já foram confirmados pelo usuário; roteiro clínico e orientações de procedimentos continuam sujeitos a aprovação profissional. O atendimento permanece demonstrativo.

### RF04 — Identificação mínima do solicitante

**Comportamento:** coletar nome e contato fictícios para compor o cadastro de demonstração. Perguntar se já é paciente representa uma autodeclaração do personagem, sem consulta a cadastro real.

**Critérios de aceite:**

- Perguntas são feitas uma por vez.
- Campos vazios ou inválidos recebem uma mensagem de correção.
- O contato digitado servirá somente ao cenário demonstrativo e não disparará comunicação externa.
- Nome, contato ou identificador digitado não concederá acesso a uma conversa; esse acesso será conferido pela sessão no servidor.
- O CPF não é obrigatório no MVP proposto; sua necessidade deverá ser justificada caso a integração com cadastro seja adicionada.
- Os testes e a apresentação usam pessoas e dados fictícios.

### RF05 — Solicitação de pré-agendamento

**Comportamento:** coletar tipo de atendimento, preferência de dia e turno, mostrar resumo e pedir confirmação dos dados.

**Critérios de aceite:**

- A lista inicial inclui avaliação geral, limpeza, ortodontia e outros procedimentos.
- O usuário consegue corrigir os dados antes do envio.
- Somente a confirmação do resumo cria a solicitação definitiva.
- A solicitação nasce como `pending_review` (rótulo: “Pendente de análise”), com identificador de acompanhamento.
- O pedido entra na fila da recepção e a conversa passa para `waiting_human`, preservando o resumo para análise.
- A mensagem final informará que a solicitação demonstrativa depende da análise de um integrante autorizado no painel, sem prometer atendimento humano quando não houver supervisão.
- Uma resposta manual da recepção aparece no chat do paciente; registrar um resultado simulado não altera uma agenda real.
- Sem integração com agenda, nenhuma data é apresentada como vaga garantida.
- Uma entrega repetida da mesma mensagem não cria pedidos duplicados; o tratamento técnico está em [Arquitetura](05-arquitetura.md).

### RF06 — Solicitação de remarcação ou cancelamento

**Comportamento:** coletar identificação do solicitante, referência da consulta informada pelo paciente e o pedido desejado.

**Critérios de aceite:**

- O usuário escolhe entre remarcar e cancelar.
- Na remarcação, o bot registra a nova preferência de dia/turno.
- A mensagem final deixa claro que o pedido aguarda análise da recepção.
- O bot não cancela nem altera uma consulta por conta própria.
- Informações de consultas de terceiros não são exibidas com base apenas no nome ou telefone informado.

### RF07 — Encaminhamento de dor ou ocorrência

**Comportamento:** aplicar perguntas fixas de demonstração e encaminhar o relato à pessoa que representa a recepção no painel.

**Critérios de aceite:**

- O roteiro oferece descrições curtas, em linguagem simples, e uma opção para falar com a equipe.
- Casos marcados pelo roteiro recebem prioridade na fila da recepção.
- O bot não afirma que diagnosticou uma emergência, não prescreve e não gera recomendações clínicas.
- A ausência de uma opção de prioridade não é apresentada como garantia de que o caso pode esperar.
- No cenário fora do expediente, o sistema usa a mensagem específica de demonstração e informa a indisponibilidade simulada da equipe.
- O roteiro e a mensagem de contingência são identificados como exemplos de demonstração; sua aprovação profissional será necessária antes de qualquer operação real.

### RF08 — Atendimento humano

**Comportamento:** criar um atendimento na fila do painel restrito, permitir que a pessoa autorizada que representa a recepção o assuma, leia o resumo e responda pelo chat do visitante correspondente.

**Critérios de aceite:**

- A fila diferencia atendimento pendente, assumido e encerrado, além da prioridade.
- O resumo inclui motivo e dados necessários, evitando repetição de perguntas.
- A recepção poderá registrar o resultado simulado e enviar uma mensagem que aparecerá somente na conversa autorizada do visitante correspondente.
- O bot fica pausado enquanto o atendimento humano estiver pendente ou ativo.
- A reativação ocorre de forma explícita após encerramento, conforme [Fluxos](04-fluxos-de-conversa.md).
- Criar um registro na fila não produz a mensagem falsa de que uma pessoa já assumiu a conversa.
- O painel exige autenticação e controles de acesso, detalhados em [Dados e segurança](06-dados-e-seguranca.md).
- O visitante público não poderá assumir conversas, listar atendimentos de terceiros, alterar decisões da recepção ou usar uma credencial compartilhada do painel.
- Sem integrante acompanhando a demonstração, o chat informará a indisponibilidade humana e permitirá iniciar outro teste próprio, sem prometer resposta posterior.

### RF09 — Entrada inválida e conteúdo não suportado

**Comportamento:** reapresentar opções após uma entrada não compreendida e encaminhar após duas falhas consecutivas.

**Critérios de aceite:**

- A primeira falha gera uma orientação simples, sem perder o estado atual.
- A segunda falha consecutiva cria encaminhamento humano com o motivo registrado.
- Uma resposta válida zera a contagem de falhas.
- Áudio, imagem ou arquivo não interpretado recebe orientação explícita para texto ou atendimento humano.
- O bot não responde a um arquivo como se tivesse compreendido seu conteúdo.

### RF10 — Funcionamento fora do expediente

**Comportamento:** demonstrar o cenário de recepção fora do expediente ou sem supervisão, mantendo FAQ e registro de solicitações disponíveis no chat web enquanto a aplicação estiver disponível.

**Critérios de aceite:**

- O cálculo considera `America/Sao_Paulo`, e não o horário padrão do servidor.
- O horário base confirmado é de segunda a sexta-feira, das 08h às 19h, e aos sábados, das 08h às 12h. A configuração deve reproduzi-lo e permitir cenários demonstrativos fora do expediente.
- Feriados e exceções ainda não foram informados. O horário não deve ser convertido em promessa de prazo máximo de resposta, que permanece sem definição.
- Mensagens não prometem atendimento “imediato” ou “em instantes” quando não houver equipe disponível.
- Relatos prioritários seguem o tratamento específico de RF07.
- O pedido permanecerá pendente no painel; quando um integrante autorizado acompanhar o teste, ele poderá assumir e responder pelo chat web. A mensagem de espera não garantirá esse retorno.

### RF11 — Satisfação e métricas

**Comportamento:** oferecer uma avaliação opcional de 1 a 5 quando um atendimento for encerrado e registrar os eventos necessários aos indicadores.

**Critérios de aceite:**

- A nota é opcional e aceita apenas valores da escala informada.
- Recusar ou ignorar a avaliação não impede atendimento futuro.
- A pesquisa não é exibida enquanto o usuário aguarda atendimento prioritário.
- FAQ resolvido, solicitação registrada e consulta confirmada são resultados distintos.
- A escala de 1 a 5 é tratada como satisfação/CSAT, sem ser chamada de NPS.

### RF12 — Registro e acompanhamento

**Comportamento:** guardar o estado necessário da conversa, pedidos, encaminhamentos e resultado do atendimento.

**Critérios de aceite:**

- Reiniciar a aplicação não apaga pedidos já confirmados pelo usuário.
- A recepção consegue distinguir uma solicitação pendente de uma concluída.
- Alterações feitas pela recepção identificam responsável e momento da ação.
- Logs técnicos não expõem desnecessariamente o conteúdo completo dos relatos fictícios.
- O ambiente publicado persistirá os dados fictícios no PostgreSQL entre reinícios e novas publicações; um banco SQLite opcional de ensaio não comprovará esse critério.
- A sessão de um visitante não poderá consultar ou alterar pedidos de outro, mesmo ao modificar IDs em URLs ou requisições.
- A restauração global da massa não será uma ação pública. Iniciar um novo teste afetará somente o episódio do próprio visitante.
- O prazo de retenção dos registros demonstrativos e a expiração das sessões serão definidos antes de liberar o link.

## Evoluções posteriores

| ID | Funcionalidade | Dependência |
| --- | --- | --- |
| EV01 | Consultar e reservar horários reais | Integração com agenda, prevenção de conflitos e confirmação transacional |
| EV02 | Localizar paciente cadastrado | Integração com cadastro e verificação de identidade |
| EV03 | Efetivar cancelamento e remarcação | Identificação da consulta, regras da clínica e controle de concorrência |
| EV04 | Enviar lembretes de 24 e 2 horas | Consultas confirmadas, agendamento de tarefas e [integração futura do canal](07-integracao-whatsapp.md) |
| EV05 | Enviar orientações pré e pós-procedimento | Materiais aprovados e seleção correta pelo profissional |
| EV06 | Acompanhar faltas e tempo de marcação | Histórico de consultas, comparecimento e referência de comparação |

## Base de conhecimento e publicação

Fonte administrativa atual: confirmação do usuário em 26/10/2026, que complementa e atualiza o PDF. Os dados abaixo estão confirmados para o planejamento; os registros dos visitantes permanecem fictícios.

| Categoria | Conteúdo confirmado ou situação atual | Tratamento no MVP |
| --- | --- | --- |
| Localização | R. Vinte e Quatro de Maio, 1474 - Centro, Indaiatuba - SP, 13339-315 | Exibir o endereço completo, com CEP. |
| Funcionamento da recepção | Segunda a sexta-feira, das 08h às 19h; sábados, das 08h às 12h. | Usar como horário base em `America/Sao_Paulo`; exceções e feriados ainda não definidos. |
| Convênio | Não realiza atendimento por convênio, mas emite nota fiscal para reembolso. | Informar emissão da nota sem garantir aprovação ou valor de reembolso. |
| Pagamentos | PIX, cartões de débito e crédito (em até 10x sem juros no carnê/cartão) e dinheiro. Condições facilitadas para tratamentos completos. | Reproduzir as condições confirmadas, sem inventar valor mínimo, limite, elegibilidade ou outras regras de parcelamento. |
| Prazo de retorno | Não foi informado um tempo máximo de resposta. | Exibir o horário e o status do pedido, sem prometer retorno em minutos, horas ou no próximo expediente. |
| Valores | Não há tabela de preços no documento | Encaminhar para a recepção |
| Cuidados sobre procedimentos | Exemplo de orientação no PDF | Não publicar automaticamente; depende de revisão do dentista e fica como evolução |

## Referência

Fontes: PDF “Definições para Chatbot (1).pdf”, páginas 1 a 4, e confirmação administrativa fornecida pelo usuário em 26/10/2026. Em caso de divergência nesses quatro temas, a confirmação posterior do usuário é a referência atual. A seleção para o MVP e os critérios de aceite continuam sendo propostas de implementação. Veja [Referências](10-referencias.md).
