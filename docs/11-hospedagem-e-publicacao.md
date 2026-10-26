# Hospedagem e publicação para testes

[Voltar ao README](../README.md) | [Ferramentas](02-ferramentas-e-frameworks.md) | [Arquitetura](05-arquitetura.md) | [Dados](06-dados-e-seguranca.md) | [Implementação](08-plano-de-implementacao.md) | [Testes](09-testes-e-metricas.md)

## 1. Objetivo e situação atual

O protótipo deverá ter um endereço público para que outras pessoas possam testar o chat pelo navegador, sem instalar ferramentas e sem depender de acesso à máquina da dupla.

**Esta entrega é somente planejamento.** Ainda não existem aplicação executável, banco remoto configurado, conta de hospedagem vinculada ou link funcional do chatbot. Nenhuma publicação é declarada concluída neste documento.

O chat publicado continuará sendo uma demonstração com dados fictícios. Ele não realizará consultas odontológicas, reservas em agenda real ou envio pelo WhatsApp. A integração de mensagens permanece uma evolução separada.

## 2. Proposta de infraestrutura

| Parte | Escolha proposta | Responsabilidade |
|---|---|---|
| Aplicação web | Render Web Service | Executar FastAPI/Uvicorn e servir chat e painel Jinja2. |
| Banco publicado | PostgreSQL no Neon | Persistir conversas, pedidos, sessões e indicadores entre reinícios. |
| Acesso dos visitantes | Subdomínio HTTPS fornecido pela hospedagem | Permitir abrir o chat em computador ou celular. |
| Código | Repositório Git conectado ao serviço | Registrar a versão publicada e permitir voltar a uma versão anterior. |
| Persistência da aplicação | SQLAlchemy com driver PostgreSQL | Reaproveitar os serviços de negócio com banco compatível com o ambiente público. |

O Render possui [orientação oficial para FastAPI](https://render.com/docs/deploy-fastapi) e fornece um endereço `onrender.com` para o serviço. O [TLS gerenciado](https://render.com/docs/tls) cobre o subdomínio padrão. Não é necessário adquirir um domínio próprio para esta demonstração.

O Neon documenta a [conexão com SQLAlchemy](https://neon.com/docs/guides/sqlalchemy). O driver e a configuração efetivos deverão ser validados durante a implementação, com dependências fixadas no lockfile.

## 3. Arquitetura do ambiente publicado

```text
Visitantes em navegadores distintos
                |
       Endereço público HTTPS
                |
       FastAPI + Jinja2 no Render
                |
     Motor de regras e serviços
                |
        SQLAlchemy + PostgreSQL
                |
          Banco Neon remoto

Recepção -> login do painel -> mesma aplicação e banco
```

Manter uma aplicação e um processo inicialmente, sem worker ou broker. O navegador do visitante acessa somente a aplicação; a credencial do banco fica no servidor.

SQLite poderá apoiar o ensaio local. Ele não será o armazenamento persistente da publicação: o disco da hospedagem proposta é efêmero. A compatibilidade entre SQLite e PostgreSQL exige revisão de tipos, transações, restrições e migrações; não presumir que basta trocar a URL. Os testes do ambiente publicado deverão usar PostgreSQL.

## 4. Experiência de teste público

- O visitante acessa o link e inicia uma conversa fictícia própria.
- Uma sessão autoriza apenas o histórico associado àquele visitante; trocar IDs na URL não revela conversas de terceiros.
- O chat apresenta o cenário como demonstração e pede que sejam usados dados fictícios.
- Não exigir CPF, cadastro pessoal ou login de rede social para experimentar o chat.
- O painel da recepção exige conta restrita à dupla ou aos avaliadores designados.
- Pedidos aparecem no painel, e respostas humanas podem ser observadas no chat do visitante correspondente.
- Quem testa não pode assumir a recepção, confirmar pedidos ou restaurar a base por uma rota pública.
- Mostrar mensagem de espera ou falha recuperável quando o serviço ainda estiver iniciando ou o banco estiver indisponível.
- O comportamento fora do expediente pode ser demonstrado com a configuração do cenário, sem alterar o relógio do servidor.

Como o painel será acessível pela internet, autorização, sessão, HTTPS, CSRF e limitação de tentativas de login integram a primeira publicação. Esses controles não devem ser adiados como se o sistema ainda estivesse restrito a `localhost`. Consulte [dados e segurança](06-dados-e-seguranca.md).

## 5. Pré-requisitos para a etapa de publicação

1. Protótipo implementado e testes relevantes aprovados.
2. Repositório Git contendo código, dependências fixadas e migrações revisadas.
3. Contas dos responsáveis na hospedagem e no banco, com acesso confirmado por eles.
4. Permissão de leitura do repositório concedida à hospedagem.
5. Banco de demonstração isolado de qualquer informação real de pacientes.
6. Usuário da recepção e configuração de sessão preparados sem senha pública padrão.
7. Configurações de ambiente e origem pública definidas, sem segredos no Git.
8. Responsável pelo período de teste, pela fila da recepção e pelo encerramento da demonstração.

As contas, permissões e configurações acima são dependências futuras. Este levantamento não as criou nem alterou.

## 6. Sequência futura de publicação

| Passo | Ação planejada | Evidência esperada |
|---|---|---|
| 1 | Concluir o protótipo e congelar a versão de demonstração. | Commit identificado e testes locais registrados. |
| 2 | Criar o projeto de banco apropriado e guardar a conexão em configuração protegida. | Conectividade conferida sem expor credencial. |
| 3 | Revisar e aplicar migrações no banco remoto de teste. | Esquema esperado e massa fictícia carregados. |
| 4 | Conectar o repositório ao Web Service e selecionar a configuração aprovada. | Serviço aponta para a branch e o commit corretos. |
| 5 | Definir runtime, comando de instalação, início e variáveis da aplicação. | Build reprodutível e servidor escutando a porta fornecida pela plataforma. |
| 6 | Publicar e verificar saúde, HTTPS e carregamento do chat. | Endereço real fornecido pela plataforma abre em navegador externo. |
| 7 | Testar paciente e recepção em dispositivos ou sessões diferentes. | Pedido, assunção, resposta e encerramento percorrem o fluxo completo. |
| 8 | Reiniciar ou republicar e conferir os registros. | Dados permanecem no PostgreSQL. |
| 9 | Registrar o endereço verificado no README e compartilhar com os avaliadores. | Link público funcional, associado à versão testada. |

O comando de início será definido a partir do módulo realmente implementado; deve usar o host e a porta exigidos pela plataforma. Não publicar um comando presumido como se já tivesse sido executado neste repositório.

Migrações não devem ser disparadas indiscriminadamente por toda requisição ou por cada processo que iniciar. Definir uma etapa controlada compatível com as capacidades da hospedagem escolhida.

## 7. Configuração e operação previstas

| Configuração | Uso no ambiente público |
|---|---|
| `DATABASE_URL` | Conexão PostgreSQL protegida, com requisitos de TLS do provedor. |
| `APP_BASE_URL` | Origem HTTPS efetivamente gerada. |
| `APP_ENV` | Identificar demonstração publicada, distinguindo-a do ensaio local. |
| Configuração de sessão | Cookies seguros, validade, revogação e segredo quando aplicável ao mecanismo escolhido. |
| Porta e host de execução | Valores definidos pela plataforma e pelo servidor ASGI. |
| `LOG_LEVEL` | Diagnóstico técnico sem senha, token ou conteúdo completo das conversas. |

Guardar segredos na configuração da hospedagem. A página pública e seu JavaScript não recebem acesso direto ao banco. O período de teste deve prever acompanhamento de erros, volume de dados e pendências humanas.

Preparar restauração da massa fictícia por procedimento administrativo restrito. Não apagar o banco automaticamente em cada publicação. Uma troca de versão do código não desfaz migrações nem recupera dados removidos; testar a recuperação separadamente.

## 8. Planos e limites - nota técnica interna, fora dos slides

Esta seção atende à restrição interna de não contratar serviços pagos na primeira etapa. **Não transpor o tema de custo, as cotas ou essa justificativa para os slides.** Os materiais de apresentação podem mostrar o link, os fluxos e a arquitetura do protótipo.

Configuração proposta: **Render Free + Neon Free**, sem ativar plano pago, depender de crédito promocional ou comprar domínio. Confirmar a disponibilidade e as condições efetivas das contas antes da publicação.

| Serviço | Limites verificados em 26/10/2026 | Implicação técnica |
|---|---|---|
| Render Free Web Service | Suspende após 15 minutos sem tráfego e pode levar cerca de um minuto para retomar; 750 horas mensais por workspace; disco efêmero. | Exibir espera na primeira visita, acompanhar uso e manter dados fora do disco do serviço. |
| Render Postgres Free | Expira 30 dias após criação. | Não foi escolhido para a persistência desta proposta. |
| Neon Free | Sem prazo anunciado e sem cartão exigido na página de preços; 1 GB por projeto, 100 CU-horas e 5 GB de transferência mensais; compute suspende após 5 minutos ocioso. | Adequado ao teste pequeno, com dados preservados durante a inatividade e uso acompanhado. |

Fontes: [Render Free](https://render.com/docs/free), [Neon Pricing](https://neon.com/pricing) e [Neon Plans](https://neon.com/docs/introduction/plans).

O [guia inicial do Render](https://render.com/docs/your-first-deploy) descreve recursos sem pagamento. A [FAQ do Render](https://render.com/docs/faq) informa que excedentes podem gerar cobrança quando existe método de pagamento; sem ele, serviços ou builds são restringidos. Não cadastrar cartão nem aceitar ativação paga para contornar uma limitação nesta etapa.

No Neon Free, esgotar compute ou transferência suspende o compute até o período seguinte; atingir armazenamento bloqueia operações que aumentariam o uso, sem apagar dados. A conta não deve ser promovida automaticamente para atender uma cota. Rever o volume de testes caso um limite seja alcançado, conforme a [documentação de planos](https://neon.com/docs/introduction/plans).

O endereço público depende dos provedores e suas cotas; não equivale a disponibilidade contínua garantida. Evitar polling desnecessário e mecanismos artificiais de manter o serviço acordado. Abrir e conferir o link antes da apresentação é uma verificação operacional, não uma solução de disponibilidade permanente.

## 9. Critérios de aceite da publicação futura

- Link HTTPS real abre fora da máquina da dupla, inclusive em celular em outra rede.
- Visitante inicia e percorre os quatro caminhos do menu sem instalar dependências.
- Conversas de visitantes diferentes permanecem isoladas.
- Painel restrito permite assunção e resposta sem reativação indevida do bot.
- Pedido continua pendente até a decisão fictícia da recepção.
- Mensagem repetida não cria um segundo pedido.
- Registros persistem após reinício ou nova publicação da aplicação.
- Erros de banco e espera de inicialização são apresentados sem falso sucesso.
- Não há segredos, credencial do banco ou senha da recepção no frontend ou repositório.
- A massa é fictícia, a restauração é restrita e o endereço foi verificado antes de compartilhado.
- README informa a versão, o endereço efetivo e o estágio real, sem afirmar integração WhatsApp.

Esses são critérios futuros. A sua inclusão no planejamento não significa que já foram executados. A matriz correspondente está em [testes e métricas](09-testes-e-metricas.md), e a relação de fontes em [referências](10-referencias.md).
