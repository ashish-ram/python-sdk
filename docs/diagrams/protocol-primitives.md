# MCP Protocol Primitives

This document explains the three core primitives of the Model Context Protocol and how they interact within the SDK.

## The Three Primitives

```mermaid
graph TB
    subgraph "MCP Core Primitives"
        subgraph "Tools - Model Controlled"
            T1[Tool Definition]
            T2[Input Schema]
            T3[Output Schema]
            T4[Tool Execution]
            
            T1 --> T2
            T1 --> T3
            T1 --> T4
        end

        subgraph "Resources - Application Controlled"
            R1[Resource URI]
            R2[Resource Content]
            R3[Resource Templates]
            R4[Subscriptions]
            
            R1 --> R2
            R1 --> R3
            R2 --> R4
        end

        subgraph "Prompts - User Controlled"
            P1[Prompt Template]
            P2[Arguments]
            P3[Message Generation]
            P4[Context Assembly]
            
            P1 --> P2
            P2 --> P3
            P3 --> P4
        end
    end

    subgraph "Control Flow"
        User[User]
        App[Application]
        Model[LLM Model]
    end

    User --> P1
    App --> R1
    Model --> T1

    T4 --> App
    R2 --> Model
    P4 --> Model

    style T1 fill:#FFE4E1
    style R1 fill:#E0F7FA
    style P1 fill:#FFF9C4
```

## Primitive Interactions

```mermaid
sequenceDiagram
    participant User
    participant Client
    participant Server
    participant LLM
    participant Backend

    Note over User,Backend: Prompt Flow (User-Controlled)
    User->>Client: Select Prompt Template
    Client->>Server: get_prompt(name, args)
    Server-->>Client: Prompt Messages
    Client->>LLM: Context + Prompt

    Note over User,Backend: Resource Flow (App-Controlled)
    Client->>Server: list_resources()
    Server-->>Client: Available Resources
    Client->>Server: read_resource(uri)
    Server->>Backend: Fetch Data
    Backend-->>Server: Data
    Server-->>Client: Resource Content
    Client->>LLM: Add to Context

    Note over User,Backend: Tool Flow (Model-Controlled)
    LLM->>Client: call_tool(name, args)
    Client->>Server: call_tool Request
    Server->>Backend: Execute Action
    Backend-->>Server: Result
    Server-->>Client: Tool Result
    Client-->>LLM: Continue Reasoning
```

## Data Flow Through SDK

```mermaid
flowchart LR
    subgraph "Client Side"
        A[Client App] --> B{Request Type?}
        B -->|Tool Call| C[call_tool]
        B -->|Resource Read| D[read_resource]
        B -->|Prompt Get| E[get_prompt]
    end

    subgraph "Transport"
        C --> F[Serialize]
        D --> F
        E --> F
        F --> G[Send over Transport]
        G --> H[Receive Response]
        H --> I[Deserialize]
    end

    subgraph "Server Side"
        J[Route Request] --> K{Handler Type?}
        K -->|Tool| L[@tool decorator]
        K -->|Resource| M[@resource decorator]
        K -->|Prompt| N[@prompt decorator]
        
        L --> O[Execute Function]
        M --> O
        N --> O
        
        O --> P[Generate Response]
        P --> Q[Validate Output]
        Q --> R[Return Result]
    end

    G --> J
    R --> H

    style F fill:#FFE4B5
    style I fill:#FFE4B5
    style O fill:#90EE90
```

## Tool Execution Detail

```mermaid
stateDiagram-v2
    [*] --> ToolDiscovery: list_tools()
    ToolDiscovery --> ToolSelection: LLM chooses tool
    ToolSelection --> InputValidation: call_tool(name, args)
    InputValidation --> ToolExecution: args valid
    InputValidation --> Error: args invalid
    
    ToolExecution --> ContextInjection: inject Context
    ContextInjection --> FunctionCall: call handler
    FunctionCall --> OutputValidation: return result
    
    OutputValidation --> StructuredOutput: has output schema
    OutputValidation --> UnstructuredOutput: no schema
    
    StructuredOutput --> Response: format response
    UnstructuredOutput --> Response: format response
    
    Response --> [*]
    Error --> [*]
```

## Resource Lifecycle

```mermaid
stateDiagram-v2
    [*] --> ResourceDefinition: @resource decorator
    ResourceDefinition --> Registration: server startup
    Registration --> Listed: list_resources()
    
    Listed --> Reading: read_resource(uri)
    Reading --> TemplateMatch: is template?
    
    TemplateMatch --> TemplateExpansion: yes
    TemplateMatch --> DirectRead: no
    
    TemplateExpansion --> DirectRead: resolve URI
    DirectRead --> ContentGeneration: call handler
    ContentGeneration --> ContentReturn: return content
    
    ContentReturn --> Subscription: subscribed?
    Subscription --> Notify: resource updated
    Subscription --> [*]: not subscribed
    
    Notify --> Listed: send update notification
```

## Prompt Processing

```mermaid
flowchart TD
    A[User Selects Prompt] --> B[get_prompt request]
    B --> C{Has Arguments?}
    
    C -->|Yes| D[Complete Arguments]
    C -->|No| E[Use Defaults]
    
    D --> F[Validate Arguments]
    F --> G{Valid?}
    G -->|No| H[Error]
    G -->|Yes| I[Execute Prompt Handler]
    
    E --> I
    
    I --> J{Return Type?}
    J -->|String| K[Create UserMessage]
    J -->|Messages| L[Return Messages]
    
    K --> M[Format Response]
    L --> M
    
    M --> N[Send to Client]
    N --> O[Client Sends to LLM]
    
    style I fill:#90EE90
    style M fill:#87CEEB
```

## Context Injection System

```mermaid
graph TB
    subgraph "Handler Definition"
        A[Function Signature] --> B{Has Context Param?}
        B -->|Yes| C[Extract Context Type]
        B -->|No| D[No Injection]
    end

    subgraph "Request Processing"
        E[Incoming Request] --> F[Create Context Object]
        F --> G[Populate Context]
        
        G --> H[request_id]
        G --> I[client_id]
        G --> J[session]
        G --> K[fastmcp]
        G --> L[request_context]
        
        L --> M[lifespan_context]
        L --> N[meta]
        L --> O[request]
    end

    subgraph "Handler Execution"
        C --> P[Inject Context]
        P --> Q[Call Handler with Context]
        
        H --> Q
        I --> Q
        J --> Q
        K --> Q
        M --> Q
        N --> Q
        O --> Q
    end

    Q --> R[Handler Accesses Context]
    R --> S{Context Methods}
    S --> T[ctx.info/debug/warning]
    S --> U[ctx.report_progress]
    S --> V[ctx.read_resource]
    S --> W[ctx.elicit]

    style F fill:#FFE4B5
    style Q fill:#90EE90
```

## Structured Output Flow

```mermaid
flowchart LR
    A[Tool Function] --> B{Return Type?}
    
    B -->|Pydantic Model| C[Structured]
    B -->|TypedDict| C
    B -->|Dataclass| C
    B -->|dict str, T| C
    B -->|Primitive| D[Wrap in dict]
    B -->|Generic Type| D
    B -->|Untyped Class| E[Unstructured]
    
    C --> F[Generate JSON Schema]
    D --> F
    
    F --> G[Validate Output]
    G --> H{Valid?}
    
    H -->|Yes| I[Create structuredContent]
    H -->|No| J[Validation Error]
    
    I --> K[Create Backward-Compatible content]
    K --> L[Return CallToolResult]
    
    E --> M[Create content only]
    M --> L
    
    style F fill:#FFE4B5
    style G fill:#FFB6C1
    style I fill:#90EE90
```

## Authentication Flow

```mermaid
sequenceDiagram
    participant Client
    participant RS as Resource Server
    participant AS as Authorization Server
    participant User

    Client->>RS: Request Protected Resource
    RS-->>Client: 401 + WWW-Authenticate header
    
    Client->>RS: GET /.well-known/oauth-protected-resource
    RS-->>Client: AS metadata (issuer, etc.)
    
    Client->>AS: Discover OAuth endpoints
    AS-->>Client: Authorization/Token URLs
    
    Client->>User: Redirect to Authorization
    User->>AS: Authenticate & Consent
    AS-->>Client: Authorization Code
    
    Client->>AS: Exchange Code for Token
    AS-->>Client: Access Token + Refresh Token
    
    Client->>RS: Request with Bearer Token
    RS->>AS: Validate Token (or local validation)
    AS-->>RS: Token Valid
    RS-->>Client: Protected Resource
```

## Summary

These diagrams illustrate:

1. **Three Primitives**: Tools (model-controlled), Resources (app-controlled), Prompts (user-controlled)
2. **Request Flow**: How requests move through the SDK layers
3. **State Machines**: Lifecycle of tools, resources, and prompts
4. **Context System**: How context is injected and used
5. **Structured Output**: Type-based output formatting
6. **Authentication**: OAuth 2.1 flow for protected resources

Each primitive serves a specific purpose in the MCP ecosystem, enabling flexible and powerful AI-application integrations.
