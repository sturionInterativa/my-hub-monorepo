# 🏗️ Arquitetura do Novo HUB Multi-tenant com RabbitMQ (Solução 1)

Este documento esmiuça o desenho técnico da nova engrenagem do nosso HUB. Estamos saindo do modelo antigo (monolítico por cliente) para uma estrutura **SaaS Multi-tenant**, escalável e isolada no Kubernetes (K8s), utilizando o **RabbitMQ** como o nosso brocador de mensagens.

---

## 📌 1. Visão Geral do Ecossistema (Kubernetes)

No Kubernetes, o HUB principal e as Integrações rodam em **Pods totalmente separados e isolados**. Eles não se conhecem diretamente; toda a prosa e distribuição de carga é feita pelas filas do **RabbitMQ**.

```mermaid
graph TD
    subgraph Clientes e Gatilhos Externos
        API[Chamadas de API / Webhooks]
    end

    subgraph Kubernetes Cluster
        HUB[Pod HUB-Principal <br><i>O Relógio / Orquestrador</i>]
        RB[RabbitMQ <br><i>Message Broker</i>]
        INT_A[Pod Integração A <br><i>Multi-tenant</i>]
        INT_B[Pod Integração B <br><i>Multi-tenant</i>]
    end

    subgraph Camada de Dados
        DB_CENTRAL[(Banco Central HUB <br><i>Apenas Agendamentos Leves</i>)]
        DB_MONGO[(MongoDB Atlas <br><i>Multi-Database per Tenant</i>)]
        DB_A[(banco: messageAutomation_clienteA)]
        DB_B[(banco: messageAutomation_clienteB)]
    end

    %% Fluxo Principal
    API --> HUB
    HUB <-->|Busca rápida de minutos| DB_CENTRAL
    HUB -->|Posta IDs nas Filas| RB
    
    RB -->|Fila A| INT_A
    RB -->|Fila B| INT_B

    %% Conexões com os Bancos dos Clientes
    INT_A --> DB_A
    INT_B --> DB_B
    
    DB_MONGO --- DB_A
    DB_MONGO --- DB_B

    style HUB fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    style RB fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    style DB_CENTRAL fill:#fbb,stroke:#333,stroke-width:2px,color:#000
