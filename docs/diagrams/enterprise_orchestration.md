```mermaid
graph TD
    subgraph "User & External Interfaces"
        direction TB
        UI_Portal[User Portal (Web/Power Apps)]
        API_GW[API Gateway (Azure API Mgt)]
        ExternalData[External Data Sources (APIs, Feeds)]
    end

    subgraph "Orchestration & Integration Layer"
        direction TB
        Orchestrator[Workflow Orchestrator (Logic Apps / Durable Func)]
        EventBus[Enterprise Service Bus (Azure Service Bus / Event Grid)]
    end

    subgraph "Core Agent Services (Microservices)"
        direction LR
        RiskAgent[Risk Modeling Agent]
        DemandAgent[Demand Modeling Agent]
        FraudAgent[Fraud Detection Agent]
        ClaimsAgent[Claims Processor Agent]
        UnderwritingAgent[Underwriting Agent]
    end

    subgraph "Supporting Services & Data"
        direction TB
        DataPlatform[Central Data Platform (Synapse, ADLS, SQL)]
        Monitoring[Observability (Azure Monitor)]
        Identity[Identity & Access (Azure AD / Entra ID)]
        HumanTasks[Human Task Service]
    end

    %% Core Flows
    UI_Portal -- Requests --> API_GW
    ExternalData -- Ingest --> API_GW
    API_GW -- Triggers/Data --> Orchestrator
    API_GW -- Ingest Events --> EventBus

    Orchestrator -- Commands/Tasks --> EventBus
    Orchestrator -- Reads/Updates --> DataPlatform
    EventBus -- Events/Commands --> RiskAgent
    EventBus -- Events/Commands --> DemandAgent
    EventBus -- Events/Commands --> FraudAgent
    EventBus -- Events/Commands --> ClaimsAgent
    EventBus -- Events/Commands --> UnderwritingAgent

    RiskAgent -- Results --> EventBus
    DemandAgent -- Results --> EventBus
    FraudAgent -- Results/Alerts --> EventBus
    ClaimsAgent -- Results/Status --> EventBus
    UnderwritingAgent -- Results/Decisions --> EventBus

    EventBus -- Agent Results/Events --> Orchestrator

    %% Data Access by Agents
    RiskAgent -- Reads/Writes --> DataPlatform
    DemandAgent -- Reads/Writes --> DataPlatform
    FraudAgent -- Reads/Writes --> DataPlatform
    ClaimsAgent -- Reads/Writes --> DataPlatform
    UnderwritingAgent -- Reads/Writes --> DataPlatform

    %% Human Interaction Loop
    Orchestrator -- Assign Task --> HumanTasks
    HumanTasks -- Displays Task --> UI_Portal
    UI_Portal -- Task Input/Decision --> HumanTasks
    HumanTasks -- Task Result --> Orchestrator

    %% Supporting Services Connections
    API_GW -- Logs/Metrics --> Monitoring
    Orchestrator -- Logs/Metrics/State --> Monitoring
    RiskAgent -- Logs/Metrics --> Monitoring
    DemandAgent -- Logs/Metrics --> Monitoring
    FraudAgent -- Logs/Metrics --> Monitoring
    ClaimsAgent -- Logs/Metrics --> Monitoring
    UnderwritingAgent -- Logs/Metrics --> Monitoring

    API_GW -- Authentication --> Identity
    UI_Portal -- Authentication --> Identity
    Orchestrator -- AuthZ (Managed ID) --> Identity
    RiskAgent -- AuthZ (Managed ID) --> Identity
    DemandAgent -- AuthZ (Managed ID) --> Identity
    FraudAgent -- AuthZ (Managed ID) --> Identity
    ClaimsAgent -- AuthZ (Managed ID) --> Identity
    UnderwritingAgent -- AuthZ (Managed ID) --> Identity
    DataPlatform -- Access Control --> Identity

```