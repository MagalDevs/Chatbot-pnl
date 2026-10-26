# Integração futura com WhatsApp

[Voltar ao README](../README.md) | [Arquitetura](05-arquitetura.md) | [Implementação futura](08-plano-de-implementacao.md)

## 1. Situação desta etapa

A futura apresentação utilizará um chat web hospedado e um painel restrito para a recepção, ambos ligados ao mesmo motor e banco PostgreSQL persistente. A publicação web não exige conta Meta, número empresarial, credenciais da Cloud API ou webhook do WhatsApp. Hospedagem e HTTPS do site são planejados em [publicação](11-hospedagem-e-publicacao.md). Nada está implementado ou publicado nesta etapa.

Não haverá envio de mensagens a números reais, automação de WhatsApp Web ou chamadas à Cloud API. A interface web demonstrará os fluxos e não será apresentada como integração WhatsApp já realizada.

Este arquivo registra somente uma evolução possível. Os requisitos externos abaixo não integram o aceite do protótipo.

## 2. Evolução de canal

Se houver decisão de conectar o sistema ao WhatsApp real, usar a **Cloud API oficial da Meta** por um adaptador de canal. O motor de regras poderá ser reaproveitado, mas transporte, persistência e operação precisarão de revisão e testes próprios.

A [coleção oficial Meta no Postman](https://www.postman.com/meta/whatsapp-business-platform/collection/wlk6lh4/whatsapp-cloud-api) descreve ativos, tokens e exemplos. O [Developer Hub](https://whatsappbusiness.com/developers/developer-hub/) apresenta recursos de desenvolvimento. Não depender de SDK arquivado nem presumir coexistência automática com o aplicativo comercial.

## 3. Pré-requisitos exclusivos da evolução

| Item | Finalidade futura |
|---|---|
| Aplicativo e portfólio empresarial Meta | Associar ativos da clínica. |
| WABA e número habilitado | Identificar a conta e o remetente. |
| Token e permissões adequadas | Autorizar chamadas da API. |
| Endpoint público HTTPS | Receber eventos externos. |
| Operação da recepção | Acompanhar fila, prazo de retorno e pendências. |
| Infraestrutura persistente e monitorada | Manter dados e processamento entre reinícios. |

Verificar elegibilidade, suporte da versão da Graph API e condições efetivas da conta nessa etapa. Nenhuma conta ou contratação deve ser tratada como tarefa da preparação da apresentação atual.

## 4. Contrato futuro de webhook

- **GET:** validar `hub.mode=subscribe` e `hub.verify_token`; devolver `hub.challenge` quando válido.
- **POST:** validar `X-Hub-Signature-256` por HMAC-SHA256 sobre os bytes originais do corpo, com o segredo do aplicativo e comparação segura.
- O verify token do GET e o app secret do POST são segredos de funções diferentes.
- Separar mensagens recebidas de atualizações de entrega e tratar todos os itens de um lote.

Referências: [exemplo oficial de assinatura](https://github.com/fbsamples/whatsapp-api-examples/blob/main/signature-validation-with-webhooks-payloads/app.py) e [Graph API Webhooks](https://developers.facebook.com/docs/graph-api/webhooks/getting-started/). O exemplo tem simplificações que precisam ser revisadas; a referência canônica teve acesso limitado na pesquisa de 26/10/2026.

Variáveis exclusivas dessa evolução: `META_APP_SECRET`, `WHATSAPP_VERIFY_TOKEN`, `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID` e `WHATSAPP_API_VERSION`. Não exigir essas variáveis no chat web planejado.

## 5. Persistência e envio externos

Ao integrar o provedor, persistir eventos antes de confirmar recebimento, deduplicar por contexto do número e ID da mensagem, e atualizar estado e intenção de resposta na mesma transação.

O chat web já prevê PostgreSQL. Inbox/outbox e worker durável serão avaliados se a integração de mensagens exigir entrega assíncrona e recuperação de tarefas. Esse desenho não está implementado e não é requisito do chat web. O recebimento externo não deve depender apenas de memória ou da duração da requisição.

Endpoint conceitual: `POST https://graph.facebook.com/{versao}/{phone_number_id}/messages`, com token no cabeçalho. Configurar timeout, erros definitivos/transitórios e revisão de resultados incertos. Aceitação da API não equivale a entrega ou leitura. Não prometer envio exatamente uma vez após uma falha de rede.

## 6. Janela, templates e recepção

A [política oficial](https://whatsappbusiness.com/policy/) consultada permite respostas livres dentro da janela de 24 horas da última mensagem do usuário e exige templates aprovados fora dela. Isso também vale para mensagens humanas enviadas pelo painel.

Definir na integração futura se a operação usará templates ou aguardará nova mensagem do paciente quando a janela fechar. Não prometer retorno posterior sem um caminho permitido de envio. Lembretes e confirmações exigem regras de contato e templates apropriados quando aplicável.

**Essa janela não se aplica ao chat web e não deve bloquear sua demonstração.** O prazo de inatividade do fluxo é outro controle.

O menu de quatro opções poderá continuar em texto numerado. Listas e botões precisam acompanhar os formatos e limites vigentes da [plataforma](https://whatsappbusiness.com/products/business-platform-features/).

## 7. Testes exclusivos da integração futura

- Handshake, assinatura válida/inválida e corpo modificado.
- Reentrega, lotes e eventos de status.
- Queda entre persistência e processamento, recuperação de tarefas e reenvio.
- Credencial expirada, limite de taxa e timeout com possível aceitação.
- Janela, templates e atendimento humano pelo canal real.
- Envio/recebimento controlado com destinatários autorizados.

Esses testes não são necessários para apresentar o chat web e não podem ser declarados executados por uma simulação de mensagens.

## 8. Condições técnicas futuras - uso interno

A adoção real deverá revisar hospedagem, operação e condições comerciais. As fontes oficiais de [FAQ](https://whatsappbusiness.com/resources/faq/) e [preços](https://whatsappbusiness.com/products/platform-pricing/) apresentaram divergência na consulta; conferir as regras efetivas quando essa evolução for considerada.

Não usar franquia, crédito ou período de teste como dependência da etapa atual. Esta nota é técnica interna e não integra o conteúdo dos slides.

## 9. Referências e acesso

As fontes acessíveis na pesquisa de 26/10/2026 incluíram coleção Meta no Postman, Developer Hub, política, funcionalidades e exemplo de assinatura. O [Get Started canônico](https://developers.facebook.com/documentation/business-messaging/whatsapp/get-started) exigiu login; as páginas canônicas de webhooks apresentaram limitação de acesso.

Revalidar a documentação no início da integração futura. A relação de fontes do projeto está em [referências](10-referencias.md).
