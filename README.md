# Arquitetural de Plataforma SaaS Offline First

Classificação dos Domínios

| Categoria                           | Domínio                                | Descrição                                                                                                                                                                          |
| ----------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Core Domain**                     | **Service Order (OS)**                 | Representa a ordem de serviço no atendimento em campo, incluindo seu ciclo de vida, execução, status, prioridade e regras operacionais que sustentam o valor principal do produto. |
| **Core Domain**                     | **Planning / Dispatch**                | Responsável pelo planejamento e distribuição das ordens de serviço, incluindo alocação por técnico/equipe, roteirização, priorização e organização da agenda operacional.          |
| **Supporting Domain**               | **Form Management**                    | Modela formulários dinâmicos associados à execução da OS, incluindo templates, perguntas, regras de validação e respostas coletadas em campo como parte do processo operacional.   |
| **Supporting Domain**               | **Tenant**                             | Representa a organização cliente da plataforma, incluindo sua estrutura, regras operacionais, parâmetros de negócio e configurações específicas por empresa.                       |
| **Supporting Domain**               | **Workforce**                          | Representa os usuários operacionais e sua organização dentro do tenant, incluindo técnicos, supervisores e suas relações com equipes e processos do negócio.                       |
| **Supporting Domain**               | **Profile & Group Access**             | Modela perfis, grupos e permissões funcionais de negócio, definindo o que cada papel pode executar dentro dos fluxos operacionais.                                                 |
| **Supporting Domain**               | **Notification & Communication**       | Responsável pela comunicação operacional e transacional com usuários, incluindo alertas, notificações e mensagens relacionadas aos eventos de negócio.                             |
| **Supporting Domain**               | **Partner Integration**                | Representa a integração com sistemas externos e parceiros, incluindo troca de informações de negócio e adaptação de contratos externos ao modelo interno.                          |
| **Supporting Domain (Condicional)** | **Device**                             | Representa os dispositivos utilizados na operação, incluindo vínculo com usuários e controle de uso, quando houver regras de negócio associadas ao device.                         |
| **Generic Domain**                  | **Identity & Access Management (IAM)** | Responsável pela autenticação e autorização técnica dos usuários, incluindo gestão de identidade, sessão e políticas de acesso.                                                    |
| **Generic Domain**                  | **Reporting & Analytics**              | Responsável pela geração de relatórios, métricas e análises para suporte à tomada de decisão, sem impactar o processamento transacional do sistema.                                |

