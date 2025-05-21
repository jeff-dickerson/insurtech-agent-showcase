```mermaid
graph TD
    subgraph "Agent Service Boundary (e.g., Claims Processor)"
        direction TB

        InputTrigger[API Request / Event Bus Message] --> InputParser[Input Parser/Validator]

        subgraph "LangChain Core Logic"
            direction TB
            AgentExecutor[LangChain AgentExecutor] -- Thought/Action Plan --> LLM[LLM (Azure OpenAI)]
            LLM -- Next Action/Response --> AgentExecutor
            AgentExecutor -- Accesses/Updates --> Memory[Agent Memory (Stateful)]
            Memory -- Loads/Saves --> MemoryStore[Persistent Memory Store (DB/Cache)]
        end

        InputParser -- Goal/Initial State --> AgentExecutor

        subgraph "Agent Tools (LangChain Tools)"
            direction LR
            ToolRegistry[Tool Registry]
            Tool_DB[DB Query Tool]
            Tool_API[Internal/External API Tool]
            Tool_Rule[Rules Engine Tool]
            Tool_Human[Request Human Review Tool]
            Tool_State[Persist/Resume State Tool]
        end

        AgentExecutor -- Selects & Uses --> ToolRegistry
        ToolRegistry -- Provides --> Tool_DB
        ToolRegistry -- Provides --> Tool_API
        ToolRegistry -- Provides --> Tool_Rule
        ToolRegistry -- Provides --> Tool_Human
        ToolRegistry -- Provides --> Tool_State

        subgraph "Interfaces to External Systems/Services"
            direction TB
            DataPlatform_IF[Data Platform Interface]
            Service_APIs_IF[Other Service APIs Interface]
            RulesEngine_IF[Rules Engine Interface]
            HumanTasks_IF[Human Task Service Interface]
            StateStore_IF[Persistent State Store Interface (CosmosDB, Redis)]
        end

        Tool_DB -- Calls --> DataPlatform_IF
        Tool_API -- Calls --> Service_APIs_IF
        Tool_Rule -- Calls --> RulesEngine_IF
        Tool_Human -- Calls --> HumanTasks_IF
        Tool_State -- Calls --> StateStore_IF

        AgentExecutor -- Final Answer/Status --> OutputFormatter[Output Formatter]
        OutputFormatter -- API Response / Event Bus Message --> OutputResponse[Response/Event]

    end

    %% Connections to Wider Enterprise Components (Outside Agent Boundary)
    DataPlatform_IF --> DataPlatform[(Enterprise Data Platform)]
    Service_APIs_IF --> API_GW[(Enterprise API Gateway)]
    HumanTasks_IF --> HumanTasks[(Enterprise Human Task Service)]
    StateStore_IF --> StateStore_DB[(Cosmos DB / Redis Cache)]

    %% Simplified connection to Orchestrator for State
    StateStore_IF -- Linked via Workflow ID --> Orchestrator[(Enterprise Orchestrator State)]
```