# Fluxos de conversa

[Voltar ao README](../README.md) · [Funcionalidades](03-funcionalidades.md) · [Arquitetura](05-arquitetura.md)

## 1. Princípios do roteiro

- Fazer uma pergunta por mensagem e manter as opções numeradas.
- Usar mensagens fixas de demonstração; antes de eventual uso real, obter aprovação da clínica.
- Identificar quando a informação ainda é apenas uma preferência ou solicitação.
- Permitir `menu` e `recepção` enquanto o bot estiver ativo.
- Preservar o estado em respostas inválidas e pausar a automação no atendimento humano.
- Usar o horário de São Paulo e os cenários de disponibilidade configurados para a recepção.
- Planejar a conversa no chat web público e as ações humanas em painel restrito, usando o mesmo motor e dados fictícios persistidos no PostgreSQL.
- Vincular cada conversa à sessão do visitante e conferir essa autorização no servidor, inclusive em atualizações de histórico.

Os textos abaixo são propostas para o protótipo web da apresentação e dos testes por link. Pessoas poderão testar em navegadores e dispositivos diferentes; nenhuma mensagem será enviada a um número real. Dados comerciais, roteiro de encaminhamento e mensagem para relatos prioritários fora do expediente precisarão de aprovação antes de eventual operação real. O chat, o painel e o ambiente público ainda não foram implementados ou publicados.

O prazo de inatividade do roteiro será configurado antes de liberar o link. A ação de iniciar novo teste criará outro episódio do próprio visitante, sem apagar pedidos ou alterar conversas de outras pessoas. Não haverá reset global na interface pública. O WhatsApp será o canal pretendido para evolução, com suas regras tratadas exclusivamente em [Integração futura WhatsApp](07-integracao-whatsapp.md).

### Mapeamento entre roteiro e modo técnico da conversa

| Estados do roteiro | Modo técnico planejado |
| --- | --- |
| Menu, FAQ e coleta automatizada | `bot_active` |
| `HUMANO_PENDENTE` | `waiting_human` |
| `HUMANO_ATIVO` | `human_active` |
| `HUMANO_ENCERRADO` ou `ENCERRADO` | `closed` |

Os nomes dos estados indicam a etapa do diálogo; o modo controla quem pode processar mensagens. Um encerramento seguido de nova mensagem do paciente permite a transição explícita de `closed` para `bot_active`. Enquanto o atendimento humano estiver pendente ou ativo, apenas reiniciar o roteiro não é suficiente para retomar o bot.

## 2. Menu principal

**Estado: `MENU_PRINCIPAL`**

> Olá! Este é um chatbot de demonstração para uma clínica odontológica. Use somente dados fictícios neste teste. Como posso ajudar?
>
> 1 — Agendar, remarcar ou cancelar uma consulta
>
> 2 — Estou com dor ou tive algum problema
>
> 3 — Consultar informações da clínica
>
> 4 — Falar com a recepção
>
> Digite o número da opção desejada.

| Entrada | Próximo estado |
| --- | --- |
| `1` | `CONSULTA_ACAO` |
| `2` | `RELATO_CATEGORIA` |
| `3` | `FAQ_MENU` |
| `4` ou `recepção` | `HUMANO_PENDENTE` |
| Outra entrada | Aplicar regra de falhas |

## 3. Pré-agendamento

| Estado | Mensagem proposta | Resultado esperado |
| --- | --- | --- |
| `CONSULTA_ACAO` | “Você deseja: 1 — Agendar; 2 — Remarcar; 3 — Cancelar?” | Escolher o tipo de solicitação |
| `CONSULTA_PACIENTE` | “Você já é paciente da clínica? 1 — Sim; 2 — Não.” | Registrar autodeclaração, sem consultar prontuário |
| `CONSULTA_NOME` | “Qual nome fictício você quer usar neste teste?” | Coletar nome fictício |
| `CONSULTA_CONTATO` | “Continuaremos por este chat. Informe um contato fictício para este cadastro de demonstração.” | Coletar contato fictício, sem comunicação externa |
| `CONSULTA_TIPO` | “Qual atendimento procura? 1 — Avaliação geral; 2 — Limpeza; 3 — Ortodontia; 4 — Outros.” | Registrar categoria |
| `CONSULTA_DIA` | “Qual dia você prefere? Essa é uma preferência; a recepção verificará a disponibilidade.” | Registrar uma data futura válida ou indicação de flexibilidade |
| `CONSULTA_TURNO` | “Qual turno prefere? 1 — Manhã; 2 — Tarde; 3 — Sem preferência.” | Registrar turno |
| `CONSULTA_RESUMO` | “Confira sua solicitação: [resumo]. 1 — Enviar; 2 — Corrigir; 3 — Voltar ao menu.” | Confirmar ou editar antes do registro |
| `CONSULTA_REGISTRADA` | “Sua solicitação [identificador] foi registrada e aguarda análise da recepção. O horário ainda não está confirmado.” | Persistir uma solicitação pendente |

O resumo deve conter nome, contato, tipo de atendimento, dia e turno solicitados. A opção de correção permite selecionar o campo a alterar e retorna ao resumo.

O contato informado será fictício e servirá para demonstrar o cadastro. Ele não será utilizado para consultar informações reais nem para enviar mensagens externas. As respostas do bot e da recepção aparecerão apenas no chat web autorizado para a sessão do visitante; informar o mesmo nome ou contato não abrirá o histórico de outra pessoa.

O registro final e o identificador só são enviados após a persistência bem-sucedida. Na mesma operação, criar ou atualizar a entrada na fila da recepção e mover a conversa para `waiting_human` (`HUMANO_PENDENTE`), pois a análise do pedido depende da equipe. A mensagem de registro pode ser enviada, mas novas entradas não reiniciam os menus. Se houver falha técnica, a mensagem deve informar que o registro não foi concluído e oferecer nova tentativa ou recepção. Não exibir sucesso antecipadamente.

O pedido persistido terá status `pending_review`, apresentado como “Pendente de análise”. Se a recepção estiver fechada, acrescentar a mensagem de horário descrita na seção 8. Sem integração com agenda, a confirmação do resumo **não reserva um horário**.

A pessoa autorizada que representa a recepção poderá comunicar o resultado simulado pelo painel, com exibição da mensagem na conversa correspondente. Registrar o resultado interno não significará que uma mensagem foi exibida. Quando não houver supervisão humana, o chat informará essa condição sem prometer retorno e oferecerá a opção de iniciar outro teste próprio.

## 4. Remarcação e cancelamento

| Estado | Mensagem proposta | Resultado esperado |
| --- | --- | --- |
| `ALTERACAO_NOME` | “Qual nome fictício você quer usar neste teste?” | Identificar o personagem do cenário |
| `ALTERACAO_CONTATO` | “Continuaremos por este chat. Informe um contato fictício para este cadastro de demonstração.” | Registrar contato fictício |
| `ALTERACAO_REFERENCIA` | “Informe o dia e o horário da consulta que deseja alterar, se souber.” | Registrar a referência informada, sem revelar consultas |
| `ALTERACAO_PREFERENCIA` | “Qual novo dia e turno você prefere?” | Aplicável somente à remarcação |
| `ALTERACAO_RESUMO` | “Confira seu pedido de [remarcação/cancelamento]: [resumo]. 1 — Enviar; 2 — Corrigir; 3 — Voltar ao menu.” | Confirmar o pedido |
| `ALTERACAO_REGISTRADA` | “Seu pedido [identificador] foi registrado e aguarda análise da recepção. A alteração ainda não foi efetivada.” | Criar solicitação com status `pending_review` (“Pendente de análise”) |

Se o paciente não souber a data ou o horário, registrar essa condição para a recepção. O bot não deve inventar uma referência nem bloquear o pedido por esse motivo. Após o registro, incluir o pedido na fila e mover a conversa para `waiting_human`, como no pré-agendamento.

Mensagem inadequada para esta etapa: “Sua consulta foi cancelada”. Esse texto só poderá ser usado pela equipe após efetivar a alteração e registrar o resultado.

## 5. Dor ou ocorrência

**Estado: `RELATO_CATEGORIA`**

> Para encaminhar seu atendimento à equipe, escolha a opção que mais se aproxima do seu relato:
>
> 1 — Dor forte ou sangramento
>
> 2 — Dente quebrado ou restauração que caiu
>
> 3 — Sensibilidade ou desconforto
>
> 4 — Prefiro falar com a recepção

As categorias vêm do documento original e precisam de revisão do dentista antes de eventual uso real. Na apresentação, servem para demonstrar encaminhamento; não representam diagnóstico nem classificação clínica conclusiva.

| Entrada | Ação proposta |
| --- | --- |
| `1` | Criar encaminhamento com prioridade conforme roteiro demonstrativo |
| `2` ou `3` | Oferecer “1 — Solicitar avaliação; 2 — Falar com a recepção”, sem afirmar que o caso pode esperar |
| `4` | Criar encaminhamento humano |

**Estado: `RELATO_ENCAMINHADO`**

> Vou encaminhar seu relato à nossa equipe com a prioridade prevista no roteiro de atendimento.

Só acrescentar que há atendimento humano em andamento quando a pessoa que representa a recepção assumir a conversa no painel. O cenário de indisponibilidade utiliza uma mensagem fixa de demonstração, conforme a seção 8, sem oferecer orientação clínica.

Não utilizar a frase do exemplo original “Identificamos uma emergência” como conclusão do sistema. Não enviar orientações clínicas improvisadas.

## 6. Informações da clínica

**Estado: `FAQ_MENU`**

> Sobre o que deseja saber?
>
> 1 — Endereço
>
> 2 — Horários
>
> 3 — Convênios
>
> 4 — Pagamentos
>
> 5 — Valores e outras dúvidas

| Estado de resposta | Conteúdo base proposto |
| --- | --- |
| `FAQ_ENDERECO` | “O endereço informado para a clínica é Rua 24 de Maio, nº 1474, Centro, Indaiatuba/SP.” |
| `FAQ_HORARIOS` | “O horário informado é de segunda a sexta, das 8h às 19h, e sábado, das 8h às 12h.” |
| `FAQ_CONVENIOS` | “A clínica não atende por convênio e emite nota fiscal para solicitação de reembolso. Para detalhes, fale com a recepção.” |
| `FAQ_PAGAMENTOS` | “A clínica informa que aceita PIX, débito, crédito e dinheiro. A recepção pode esclarecer as condições de parcelamento.” |
| `FAQ_OUTRAS` | “A recepção pode esclarecer valores e outras dúvidas. Deseja que eu encaminhe seu atendimento?” |

Na apresentação, essas respostas são conteúdo de exemplo derivado do PDF. Uma eventual operação real dependeria de dados aprovados pela clínica. A base e as pendências de revisão estão em [Funcionalidades](03-funcionalidades.md).

Após responder:

> Posso ajudar com mais alguma coisa? 1 — Solicitar consulta; 2 — Ver outras informações; 3 — Falar com a recepção; 4 — Encerrar.

## 7. Encaminhamento humano e pausa do bot

**Gatilhos:** escolha do paciente, regra de prioridade, pergunta sem resposta na base ou duas entradas inválidas consecutivas.

**Estado: `HUMANO_PENDENTE`**

> Seu pedido demonstrativo entrou na fila da recepção. Um integrante autorizado poderá consultar o resumo deste teste.

**Estado: `HUMANO_ATIVO`**, somente após ação real de um atendente:

> A recepção assumiu seu atendimento. Você pode continuar a conversa por aqui.

Durante os dois estados, o bot não executa menus nem cria novos pedidos a cada mensagem. As novas mensagens entram no histórico do atendimento e ficam disponíveis no painel.

Resumo mínimo para o painel:

- Motivo e categoria do encaminhamento.
- Prioridade conforme roteiro, quando aplicável.
- Nome e contato já coletados.
- Dados do pedido de consulta, se houver.
- Últimas mensagens necessárias para compreender a solicitação.

**Estado: `HUMANO_ENCERRADO`**: a recepção registra o resultado e encerra o atendimento. O retorno à automação deve ser explícito. Uma nova mensagem do paciente após o encerramento pode iniciar o menu; uma mensagem recebida antes dele continua no atendimento humano.

O chat web público e o painel autenticado serão as duas interfaces da demonstração, acessíveis de dispositivos diferentes. Uma resposta autorizada da recepção será registrada no banco e exibida somente na conversa correspondente. Os visitantes não terão acesso à fila geral ou às ações humanas. Não haverá integração com uma plataforma externa de mensagens nesta etapa.

Quando não houver integrante acompanhando a fila, a mensagem adicional será:

> Esta demonstração está sem recepção acompanhando no momento. Seu pedido fictício ficou registrado, mas não há previsão de resposta humana. Você pode iniciar outro teste pelo botão “Nova conversa”.

Esse botão não apagará pedidos anteriores ou dados de outras pessoas. O comando `menu` continuará sem reativar automaticamente uma conversa em atendimento humano.

## 8. Fora do expediente

Para FAQ, continuar respondendo normalmente. Para pré-agendamento e pedidos de alteração, registrar o pedido e acrescentar:

> Este cenário está fora do expediente da recepção. Seu pedido fictício ficou registrado, mas esta demonstração não garante retorno humano. Horário do cenário: [horário configurado da recepção].

O cenário simulará a ausência da recepção, sem depender de uma equipe real. Para demonstrar a retomada, uma pessoa autorizada da dupla assumirá o pedido no painel e responderá pelo chat web. Sem supervisão, o visitante poderá iniciar outro teste próprio, sem alterar a fila dos demais.

Para encaminhamento comum, criar a fila pendente e usar a mesma informação de indisponibilidade. Não informar que há uma transferência imediata em andamento.

Para relatos prioritários, informar a indisponibilidade e manter o caso destacado no painel. Na apresentação, usar uma mensagem fixa identificada como demonstração, sem orientação clínica. Uma operação real dependeria de texto específico aprovado pelo dentista.

O fluxo web permitirá demonstrar resposta posterior no chat, quando um integrante autorizado acompanhar o teste, sem aplicar regras do WhatsApp. A integração desse canal pertencerá à etapa futura; veja [Integração WhatsApp](07-integracao-whatsapp.md).

## 9. Entradas inválidas e falhas

| Situação | Resposta e comportamento |
| --- | --- |
| Primeira entrada inválida | “Não consegui identificar a opção. Digite um dos números abaixo ou escreva ‘recepção’.” Reapresentar opções do estado atual. |
| Segunda entrada inválida consecutiva | “Vou encaminhar sua dúvida à recepção para ajudar.” Criar o atendimento humano, considerando horário de funcionamento. |
| Entrada válida após uma falha | Processar a escolha e zerar a contagem de falhas. |
| Áudio, imagem ou arquivo não suportado | “Nesta versão, consigo seguir por texto ou opções numeradas. Você também pode escrever ‘recepção’.” Aplicar a regra de falhas se a conversa não avançar. |
| Data inválida | Pedir correção e oferecer opção de informar apenas flexibilidade de dia/turno. |
| Erro ao registrar pedido | Informar que o registro não foi concluído; não enviar confirmação de sucesso. |
| Envio repetido pelo chat web | Não duplicar o pedido ao repetir a mesma ação identificada; tratar idempotência na aplicação. |

Atualizações da interface web não serão novas mensagens do paciente. Recarregar a página não deverá reenviar o último pedido automaticamente. Uma sessão diferente não poderá recuperar a conversa apenas conhecendo seu identificador; a aplicação recusará esse acesso sem revelar o histórico.

## 10. Encerramento e satisfação

**Estado: `AVALIACAO`**

> De 1 a 5, como você avalia este atendimento? Se preferir, digite “pular”.

**Estado: `ENCERRADO`**

> Obrigado pelo contato! Quando precisar, envie uma nova mensagem.

A avaliação é opcional e só ocorre após encerramento adequado. Não enviar pesquisa enquanto um caso prioritário estiver aguardando a equipe. Uma solicitação registrada ainda depende da confirmação humana; encerrar a conversa automatizada não muda o status do pedido.

## Referência

Fonte interna: PDF “Definições para Chatbot (1).pdf”, páginas 2 e 3 para o fluxo original e páginas 3 e 4 para a base. As mensagens desta página adaptam o roteiro à versão simples proposta, removendo promessas que dependem de disponibilidade ou ação humana. Consulte [Referências](10-referencias.md).
