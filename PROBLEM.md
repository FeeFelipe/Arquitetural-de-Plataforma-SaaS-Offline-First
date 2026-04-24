Desafio Técnico
Principal / Staff Engineer
Plano de Reestruturação Arquitetural de Plataforma SaaS Offline-First

1. Contexto da Empresa
   Você foi convidado(a) para um desafio técnico como parte do processo seletivo para a posição de Principal / Staff Engineer. A empresa opera uma plataforma SaaS B2B madura, com as seguintes características:
   Time de Engenharia e Produto: ~40 pessoas (engenharia, produto, design, QA)
   Orçamento anual (Infra): R$ 1.5MM
   Base de usuários ativos: ~50.000
   Meta de crescimento: triplicar a base em 24 meses (~150.000 usuários)
   Carga atual: ~450 transações por segundo
   Perfil do produto: plataforma offline-first com forte componente mobile (sincronização bidirecional), painel web administrativo e integrações externas
   Considere o time de Platform&Arch: 2 Devs seniors e 2 SREs/DevOps
   A empresa atingiu um ponto de inflexão: a arquitetura atual sustentou o crescimento até aqui, mas apresenta sinais claros de fadiga arquitetural que ameaçam o próximo ciclo de crescimento.
2. Arquitetura Atual (AS-IS)
   2.1 Visão geral
   A plataforma é composta por três grandes frentes:
   Web Client (App.com.br): painel administrativo servido por ASP.NET MVC em múltiplas instâncias EC2 atrás de ELB, com WAF e Route 53.
   Mobile (.NET MAUI + SQLite local): aplicativo offline-first que sincroniza via AWS IoT Core e SNS (canal multidirecional por codadmin/codusuario).
   Backend compartilhado: serviços em ECS Fargate, cerca de uma centena de Lambdas, RDS MySQL, DynamoDB (deduplicação de sync), Amazon SQS, SES, SendGrid, SignalR, Hangfire (agendamentos e relatórios) e um IDP in-house.


2.2 Stack detalhada


2.3 Diagramas de referência
Os dois diagramas fornecidos em anexo ilustram:
Infraestrutura Web + Cloud: Web Client → Route 53 → WAF → ELB → 4x EC2 ASP.NET MVC → camada de serviços AWS (DocumentDB, RDS MySQL, S3, SQS, SNS, IoT Core, .NET Core, Hangfire, Fargate, SignalR, SendGrid, API Gateway + Lambda) → canal multidirecional com o Mobile.
Infraestrutura Mobile: Frontend → IoT Core → tópicos multidirecionais ({codadmin}-{codusuario}, codadmin, gpsDados, gpsDadosColetados, {codadmin}-{codusuario}-status) → SNS + IoT Core → app .NET MAUI com SQLite local → WAF → ELB → ECS Fargate com múltiplos serviços acessando toda a camada de dados AWS.


3. Dores Conhecidas
   3.1 Instabilidades recorrentes (diárias)
   AcquireRequestState: sessão do usuário registrada na sessão do servidor Windows gera contenção, timeouts e comportamento errático em cenários de múltiplas abas, load balancer sem sticky session confiável e picos de concorrência.
   CPU do banco topando: MySQL único atendendo OLTP (web, mobile, serviços, Lambdas) e OLAP (relatórios).
   Relatórios pesados: geração síncrona/assíncrona ou mal isolada competindo por recursos com o transacional.
   3.2 Observabilidade insuficiente
   Ausência de logs transacionais correlacionados entre serviços.
   Sem distributed tracing, sem SLOs formais.
   3.3 Governança e segurança
   Sem CI completo: ausência de gates de qualidade, SAST, DAST, SCA, testes automatizados bloqueantes.
   Riscos de segurança: IDP proprietário sem auditoria externa, falta de WAF rules específicas, ausência de API Gateway centralizando authN/Z, rate limiting e throttling.
   Fragmentação de versões .NET (4.6.1 até 10): superfície de CVEs ampla, difícil padronização, custo alto de manutenção.
   3.4 Custo de cloud
   RDS MySQL único superdimensionado para absorver picos.
   Fargate + ~100 Lambdas com alto overhead de cold start e conexões simultâneas ao banco (risco de esgotamento de connection pool).
   Falta de FinOps estruturado.
4. Objetivos do Desafio
   Como Principal/Staff Engineer recém-contratado(a), você é o(a) responsável técnico(a) por liderar a reestruturação da plataforma nos próximos 18–24 meses, de forma que, ao final, a empresa tenha:
   Capacidade de suportar 3x o tráfego atual (~1.350 TPS) com headroom.
   Disponibilidade ≥ 99,9% com SLOs por domínio de negócio.
   Redução mensurável de custo unitário por transação (mesmo com crescimento absoluto de cloud).
   Pipeline completo de CI/CD com gates de qualidade e segurança.
   Observabilidade ponta a ponta (logs, métricas, traces, eventos de negócio).
   Ambientes Dev/Staging/Prod segregados.
   Arquitetura que permita ao time de 40 pessoas entregar com autonomia e sem bloqueios cruzados.
   Uso pragmático de IA para acelerar cada etapa (desenvolvimento, revisão, migração, operação).
5. O Que Você Deve Entregar
   Um documento técnico (e, se desejar, diagramas complementares) contendo:
   5.1 Diagnóstico e princípios arquiteturais
   Sua leitura crítica da arquitetura atual: o que é patrimônio (manter), o que é dívida (refatorar) e o que é risco existencial (substituir com urgência).
   Princípios arquiteturais que guiarão as decisões (ex.: API-first, event-driven onde fizer sentido, schema ownership por domínio, fail-fast, observabilidade nativa, segurança por padrão, custo como requisito funcional).
   Target Architecture (TO-BE) proposta em alto nível, com justificativas.
   5.2 Plano em etapas (roadmap)
   Um plano faseado (sugestão: 4 a 6 ondas de 1 a 2 meses cada), e para cada etapa:
   Objetivo da etapa e qual dor de negócio/técnica ela endereça.
   Escopo técnico (o que sai, o que entra, o que permanece).
   Ganhos esperados, de preferência mensuráveis (latência, custo, disponibilidade, velocity, MTTR, etc.).
   Riscos e estratégias de mitigação (incluindo estratégia de migração: strangler fig, expand/contract de banco, feature flags, blue/green, canary).
   Como IA acelera esta etapa concretamente (não genericamente).
   Estimativa de esforço em pessoas-mês e dependências.
   Critérios de saída (definition of done da etapa).
   5.3 Decisões técnicas que você precisa tomar e defender
   Seja explícito em decidir e justificar (trade-offs, não apenas listar opções):
   Linguagem e runtime: manter .NET como stack principal? Consolidar em qual versão (LTS)? Introduzir outra linguagem para algum domínio (ex.: Go/Rust para serviços críticos de baixa latência, Python para workloads de dados/IA)?
   Frontend web: continuar com ASP.NET MVC server-rendered ou migrar para SPA (React/Next.js/Remix) com BFF? Como desacoplar sem big-bang?
   Dados: como sair do MySQL único? Réplica de leitura, CQRS, segregação por bounded context, migração para Aurora (MySQL/PostgreSQL compatível), introdução de data warehouse (Redshift/Snowflake/BigQuery) para relatórios, eventual consistency onde couber.
   API Gateway e malha de serviços: qual solução (Amazon API Gateway, Kong, Envoy, service mesh como Istio/App Mesh)? Como tratar authN/Z centralizada, rate limiting, versionamento de API?
   IDP: manter in-house, migrar para Cognito, Auth0, Keycloak, Azure AD B2C, WorkOS? Caminho de migração sem quebrar clientes existentes.
   Mensageria e sync mobile: manter SNS + IoT Core ou consolidar em uma stack (ex.: EventBridge + AppSync, Kafka/MSK, Ably, PowerSync/ElectricSQL para offline-first)?
   Lambdas vs containers: critério de quando usar cada um, como consolidar a centena de Lambdas (muitas provavelmente são candidatas a virar endpoints em serviços maiores).
   Cache: Redis continua? Estratégia de invalidação, cache aside vs write-through, quando usar cache local em container.
   Observabilidade: stack proposta (OpenTelemetry + Datadog/New Relic/Grafana Cloud/Honeycomb/Elastic). Como instrumentar legado sem reescrever tudo.
   CI/CD: ferramenta (GitHub Actions, GitLab CI, Buildkite, Azure DevOps), estratégia de trunk-based vs GitFlow, gates obrigatórios, preview environments por PR.
   IaC: Terraform, Pulumi, CDK? Como introduzir sem reescrever toda a infra existente.
   Estratégia de testes: pirâmide, contract testing (Pact), testes de carga (k6/Gatling), chaos engineering.
   5.4 Estratégia de uso de IA ao longo do programa
   Detalhe onde e como IA entra — com foco em impacto real, não em hype:
   Desenvolvimento: Claude Code, Cursor, Copilot para modernização de código (.NET 4.6.1 → .NET 10), geração de testes, refactor assistido, tradução ASP.NET MVC → SPA.
   Migração de dados e código: agentes para análise estática de dependências, geração de scripts de migração, detecção de código morto entre as ~100 Lambdas.
   Revisão de código e segurança: revisão automática de PRs, detecção de vulnerabilidades, análise de conformidade arquitetural.
   Operação: AIOps para correlação de incidentes, root cause analysis assistida, geração de postmortems a partir de telemetria.
   Produto: copilotos internos para time de suporte/CS consultando base de conhecimento e logs.
   Governança: como você vai garantir qualidade do código gerado por IA, propriedade intelectual, custos com tokens, e não criar dependência cega.
   5.5 FinOps e redução de custo
   Como você atacaria o custo de cloud dado o cenário atual (~450 TPS).
   Onde estão os maiores desperdícios prováveis (RDS superdimensionado, Fargate com baixa densidade, Lambdas com configuração default, egress, NAT Gateway, etc.).
   Savings plans, Graviton, rightsizing, consolidação de Lambdas, cache em edge (CloudFront), compressão e paginação de APIs.
   Meta: proponha uma redução de custo unitário por transação realista e como medi-la.
   5.6 Organização e processo
   Ritos e artefatos (RFC/ADR, tech radar, guilds, on-call, blameless postmortems).
   Como balancear entrega de produto (features) com o programa de reestruturação (sugestão clássica: 70/20/10, mas defenda sua proposta).
   5.7 Riscos, KPIs e comunicação executiva
   Top 5 riscos do programa e seus planos de contingência.
   KPIs técnicos e de negócio que você reportaria mensalmente para C-level.
   Como você comunicaria trade-offs difíceis (ex.: congelar features por um trimestre em um domínio) para CEO/CPO/CFO/CTO em uma empresa de médio porte.
6. Restrições e Regras do Desafio
   Não existe greenfield. A plataforma atende 50k usuários pagantes hoje; nenhuma fase pode implicar janela de downtime relevante nem congelamento total de features do produto.
   O orçamento é finito: R$ 1MM/ano. Propostas que explodam o orçamento sem justificativa de ROI serão penalizadas.
   Assume-se que o time atual tem competência em .NET e AWS, e pouca experiência prévia com outras stacks. Mudanças de stack precisam considerar curva de aprendizado.
   A empresa é brasileira com operação também em LATAM, com clientes majoritariamente no Brasil; considere LGPD, latência regional (São Paulo) ou migração de região, e requisitos de compliance do setor B2B.
   Você pode (e deve) discordar de premissas do enunciado se tiver argumentação técnica sólida.
7. Formato e Tempo Sugerido
   Formato livre: documento em markdown, PDF ou deck. Diagramas são bem-vindos (C4 model, fluxos de sequência, diagramas de deployment).
   Tempo sugerido: até 8 horas de trabalho efetivo, distribuídas ao longo de uma semana.
   Valorizamos densidade e clareza mais do que volume.
   Envie também uma lista de perguntas que você faria aos stakeholders (CEO, CTO, CPO, líderes de engenharia) caso pudesse. Bons Staff/Principal Engineers fazem as perguntas certas.



8. Critérios de Avaliação
   Seu material será avaliado em cinco eixos, com peso aproximado:


9. Apresentação Final
   Candidatos finalistas farão uma apresentação de 60 a 120 minutos (50% de pitch + 50% de Q&A) para um painel composto por CTO, CPO e um Staff Engineer externo convidado. Espere perguntas duras sobre trade-offs, custos, riscos e a primeira onda do plano, aquela que você começaria a executar na segunda-feira seguinte ao aceite da oferta.
10. O Que NÃO Estamos Avaliando
    Nome de ferramentas da moda. Preferimos ver você defender PostgreSQL sobre uma solução exótica do que o contrário.
    Perfeição. Esta é uma decisão sob incerteza; queremos ver como você raciocina.
    Concordância com nossas dores. Se achar que nosso diagnóstico está errado em algum ponto, diga e justifique.

Boa sorte. Estamos ansiosos para ler seu plano.