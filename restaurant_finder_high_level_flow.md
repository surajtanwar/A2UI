# Restaurant Finder - High Level Flow Diagram

## System Architecture Overview

```mermaid
flowchart TB
    subgraph Client["🌐 Client Layer (Browser)"]
        UI[User Interface<br/>Lit Components]
        Renderer[A2UI Renderer<br/>JSON → Native UI]
        ClientSDK[A2A Client SDK<br/>Protocol Handler]
    end
    
    subgraph Transport["🔄 Transport Layer"]
        HTTP[HTTP/REST]
        A2A[A2A Protocol v0.3<br/>Agent-to-Agent]
        A2UI_EXT[A2UI Extension v0.8<br/>application/json+a2ui]
    end
    
    subgraph Server["⚙️ Server Layer (Python)"]
        A2AServer[A2A Server<br/>Starlette/Uvicorn]
        Executor[Agent Executor<br/>Request Handler]
        AgentLogic[Restaurant Agent<br/>ADK LlmAgent]
    end
    
    subgraph AI["🤖 AI Layer"]
        LLM[Gemini 2.5 Flash<br/>LLM Model]
        Tools[Function Tools<br/>get_restaurants]
    end
    
    subgraph Data["📁 Data Layer"]
        RestaurantDB[(Restaurant Data<br/>restaurant_data.json)]
    end
    
    %% Request Flow (Left to Right)
    UI -->|1. User Query| ClientSDK
    ClientSDK -->|2. A2A Message| HTTP
    HTTP -->|3. POST /send| A2AServer
    A2AServer -->|4. Execute| Executor
    Executor -->|5. Stream Query| AgentLogic
    AgentLogic -->|6. LLM Call| LLM
    LLM -->|7. Tool Call| Tools
    Tools -->|8. Read| RestaurantDB
    
    %% Response Flow (Right to Left)
    RestaurantDB -.->|9. Data| Tools
    Tools -.->|10. Results| LLM
    LLM -.->|11. A2UI JSON| AgentLogic
    AgentLogic -.->|12. Validate & Format| Executor
    Executor -.->|13. Task Response| A2AServer
    A2AServer -.->|14. HTTP Response| HTTP
    HTTP -.->|15. A2A Message| ClientSDK
    ClientSDK -.->|16. A2UI Messages| Renderer
    Renderer -.->|17. Native Components| UI
    
    %% Styling
    classDef clientStyle fill:#e1f5ff,stroke:#0277bd,stroke-width:2px
    classDef serverStyle fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef aiStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef dataStyle fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    classDef transportStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    
    class UI,Renderer,ClientSDK clientStyle
    class A2AServer,Executor,AgentLogic serverStyle
    class LLM,Tools aiStyle
    class RestaurantDB dataStyle
    class HTTP,A2A,A2UI_EXT transportStyle
```

## Simplified Data Flow

```mermaid
graph LR
    A[👤 User] -->|Text Query| B[Browser]
    B -->|A2A Protocol| C[A2A Server]
    C -->|Execute| D[Agent]
    D -->|Prompt| E[🤖 LLM]
    E -->|Function Call| F[Tool]
    F -->|Query| G[(Data)]
    G -->|Results| F
    F -->|Data| E
    E -->|A2UI JSON| D
    D -->|Validate| C
    C -->|Response| B
    B -->|Render| H[UI Components]
    H -->|Display| A
    
    style A fill:#bbdefb
    style E fill:#f3e5f5
    style G fill:#c8e6c9
    style H fill:#ffe0b2
```

## Request/Response Flow

```mermaid
sequenceDiagram
    participant User
    participant Client
    participant Server
    participant Agent
    participant LLM
    participant Tool
    
    User->>Client: "Find Chinese restaurants"
    Client->>Server: A2A Message (text)
    Server->>Agent: Execute query
    Agent->>LLM: Process with instructions
    LLM->>Tool: get_restaurants()
    Tool-->>LLM: Restaurant data
    LLM-->>Agent: A2UI JSON
    Agent-->>Server: Validated response
    Server-->>Client: A2A Task (DataParts)
    Client-->>User: Rendered UI cards
```

## Component Interaction Matrix

```mermaid
flowchart LR
    subgraph Input
        A1[Text Query]
        A2[UI Event]
    end
    
    subgraph Processing
        B1{Extension<br/>Negotiation}
        B2[Session<br/>Management]
        B3[Tool<br/>Execution]
        B4[A2UI<br/>Generation]
        B5[Schema<br/>Validation]
    end
    
    subgraph Output
        C1[Text Response]
        C2[A2UI JSON]
        C3[UI Components]
    end
    
    A1 --> B1
    A2 --> B1
    B1 -->|A2UI Active| B2
    B2 --> B3
    B3 --> B4
    B4 --> B5
    B5 -->|Valid| C2
    B5 -->|Retry| B4
    B1 -->|Text Only| C1
    C2 --> C3
    
    style B1 fill:#fff3e0
    style B4 fill:#f3e5f5
    style B5 fill:#ffebee
    style C3 fill:#e8f5e9
```

## Technology Stack

```mermaid
mindmap
  root((Restaurant<br/>Finder))
    Frontend
      Lit Web Components
      TypeScript
      Vite Dev Server
      A2UI Renderer v0.8
    Backend
      Python 3.13+
      Starlette Framework
      Uvicorn Server
      A2A SDK v0.3
    AI/ML
      Google ADK
      Gemini 2.5 Flash
      LiteLLM
      Function Tools
    Protocols
      A2A Protocol v0.3
      A2UI Extension v0.8
      JSON-RPC
      HTTP/REST
    Data
      JSON Files
      In-Memory Sessions
      File System Storage
```

## Key Data Transformations

```mermaid
flowchart TD
    Start([User Input]) --> T1{Input Type?}
    
    T1 -->|Text| T2[Text String]
    T1 -->|UI Event| T3[ClientEvent JSON]
    
    T2 --> T4[A2A Message<br/>TextPart]
    T3 --> T5[A2A Message<br/>DataPart]
    
    T4 --> T6[Agent Query String]
    T5 --> T6
    
    T6 --> T7[LLM Prompt<br/>with Instructions]
    
    T7 --> T8[Tool Call Parameters<br/>cuisine, location, count]
    
    T8 --> T9[Restaurant Data<br/>JSON Array]
    
    T9 --> T10[A2UI JSON Format<br/>dataModelUpdate +<br/>surfaceUpdate]
    
    T10 --> T11{Schema<br/>Valid?}
    
    T11 -->|No| T7
    T11 -->|Yes| T12[A2A Task Response<br/>DataPart Array]
    
    T12 --> T13[Lit Components<br/>Cards, Buttons, Images]
    
    T13 --> End([Rendered UI])
    
    style T7 fill:#f3e5f5
    style T10 fill:#fff3e0
    style T11 fill:#ffebee
    style T13 fill:#e8f5e9
```

## Architecture Patterns

### 1. **Layered Architecture**
```
┌─────────────────────────────────┐
│   Presentation Layer (Lit UI)   │
├─────────────────────────────────┤
│   Protocol Layer (A2A/A2UI)     │
├─────────────────────────────────┤
│   Business Logic (Agent/LLM)    │
├─────────────────────────────────┤
│   Data Layer (JSON Files)       │
└─────────────────────────────────┘
```

### 2. **Request-Response Pattern**
```
Client → A2A Request → Server → Agent → LLM → Tool
   ↑                                              ↓
   └────── A2A Response ← Validate ← A2UI JSON ←─┘
```

### 3. **Event-Driven Pattern**
```
User Action → ClientEvent → DataPart → Agent Processing
                                              ↓
                                        New UI State
```

## Core Concepts

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Client** | Lit + TypeScript | Render UI components from A2UI JSON |
| **Transport** | A2A Protocol | Standardized agent communication |
| **Server** | Starlette + Python | Host agent and handle requests |
| **Agent** | Google ADK | Orchestrate LLM and tools |
| **AI** | Gemini LLM | Generate intelligent responses |
| **Format** | A2UI v0.8 | Declarative UI specification |

## Data Flow Summary

1. **Input**: User query (text) or UI event (JSON)
2. **Protocol**: Wrapped in A2A message format
3. **Processing**: Agent uses LLM + tools to process
4. **Generation**: LLM creates A2UI JSON response
5. **Validation**: Schema validation with retry logic
6. **Transport**: A2A protocol sends response
7. **Rendering**: Client renders native UI components
8. **Interaction**: User actions create new events → loop

## Security Model

```mermaid
flowchart TD
    A[User Input] --> B{Input Validation}
    B --> C[A2A Message Envelope]
    C --> D{Extension Check}
    D -->|Trusted| E[A2UI Processing]
    D -->|Unknown| F[Reject/Text Only]
    E --> G{Schema Validation}
    G -->|Valid| H[Render UI]
    G -->|Invalid| I[Retry/Error]
    H --> J{Component Catalog}
    J -->|Approved| K[Native Component]
    J -->|Unknown| L[Reject]
    
    style B fill:#ffebee
    style G fill:#ffebee
    style J fill:#ffebee
    style K fill:#e8f5e9
```

**Key Security Features:**
- ✅ **Declarative Format**: A2UI is data, not code
- ✅ **Schema Validation**: All JSON validated against schema
- ✅ **Component Catalog**: Only pre-approved components rendered
- ✅ **Extension Negotiation**: Client controls what features to enable
- ✅ **Sandbox Execution**: No arbitrary code execution

---

**Generated for Restaurant Finder Application - A2UI Project**

