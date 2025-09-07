# RAG System Query Processing Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend as Frontend<br/>(script.js)
    participant API as FastAPI<br/>(app.py)
    participant RAG as RAG System<br/>(rag_system.py)
    participant Session as Session Manager<br/>(session_manager.py)
    participant AI as AI Generator<br/>(ai_generator.py)
    participant Tools as Tool Manager<br/>(search_tools.py)
    participant Vector as Vector Store<br/>(vector_store.py)
    participant ChromaDB as ChromaDB<br/>(Database)

    User->>Frontend: Types query & clicks send
    
    Frontend->>Frontend: Disable input, show loading
    Frontend->>API: POST /api/query<br/>{query, session_id}
    
    API->>RAG: query(request.query, session_id)
    
    RAG->>Session: get_conversation_history(session_id)
    Session-->>RAG: Previous chat context
    
    RAG->>AI: generate_response()<br/>+ tools + history
    
    AI->>AI: Call Claude API with<br/>search tool available
    
    Note over AI: Claude decides to search<br/>based on query content
    
    AI->>Tools: execute("search_course_content")<br/>{query, course_name?, lesson_number?}
    
    Tools->>Vector: search(query, filters)
    
    Vector->>Vector: Embed query using<br/>sentence transformers
    Vector->>ChromaDB: Semantic similarity search
    ChromaDB-->>Vector: Ranked chunks with metadata
    Vector-->>Tools: SearchResults with sources
    
    Tools-->>AI: Formatted search results
    
    AI->>AI: Claude generates response<br/>using search context
    AI-->>RAG: Final response
    
    RAG->>Tools: get_last_sources()
    Tools-->>RAG: Source references
    
    RAG->>Session: add_exchange(session_id, query, response)
    
    RAG-->>API: (response, sources)
    API-->>Frontend: JSON {answer, sources, session_id}
    
    Frontend->>Frontend: Remove loading, display response
    Frontend->>User: Show answer + collapsible sources
```

## Architecture Components

```mermaid
graph TB
    subgraph "Frontend Layer"
        UI[Web Interface<br/>HTML/CSS/JS]
    end
    
    subgraph "API Layer"
        FastAPI[FastAPI Server<br/>app.py]
    end
    
    subgraph "RAG Orchestration"
        RAGSys[RAG System<br/>rag_system.py]
        Session[Session Manager<br/>session_manager.py]
    end
    
    subgraph "AI Processing"
        AIGen[AI Generator<br/>ai_generator.py]
        Claude[Anthropic Claude<br/>API]
    end
    
    subgraph "Search & Retrieval"
        Tools[Tool Manager<br/>search_tools.py]
        Vector[Vector Store<br/>vector_store.py]
        ChromaDB[(ChromaDB<br/>Vector Database)]
    end
    
    subgraph "Document Processing"
        DocProc[Document Processor<br/>document_processor.py]
        Docs[Course Documents<br/>docs/]
    end
    
    UI --> FastAPI
    FastAPI --> RAGSys
    RAGSys --> Session
    RAGSys --> AIGen
    AIGen --> Claude
    AIGen --> Tools
    Tools --> Vector
    Vector --> ChromaDB
    DocProc --> Vector
    Docs --> DocProc
    
    style Claude fill:#f9f,stroke:#333,stroke-width:2px
    style ChromaDB fill:#bbf,stroke:#333,stroke-width:2px
```

## Data Flow Detail

```mermaid
flowchart TD
    A[User Query] --> B[Frontend Processing]
    B --> C[API Request]
    C --> D{Session Exists?}
    D -->|No| E[Create New Session]
    D -->|Yes| F[Get History]
    E --> G[RAG System Query]
    F --> G
    
    G --> H[AI Generation with Tools]
    H --> I{Claude Decides<br/>Search Needed?}
    
    I -->|No| J[Generate from<br/>General Knowledge]
    I -->|Yes| K[Execute Search Tool]
    
    K --> L[Vector Similarity Search]
    L --> M[Retrieve Relevant Chunks]
    M --> N[Format Search Results]
    N --> O[Claude Generates<br/>Context-Aware Response]
    
    J --> P[Final Response]
    O --> P
    
    P --> Q[Update Session History]
    Q --> R[Return Response + Sources]
    R --> S[Frontend Display]
    
    style I fill:#ffffcc
    style L fill:#ccffcc
    style O fill:#ffcccc
```

## Key Features Illustrated

1. **Session Management**: Maintains conversation context across queries
2. **Tool-Based Search**: Claude intelligently decides when to search
3. **Semantic Retrieval**: Vector similarity matching in ChromaDB
4. **Source Attribution**: Tracks and returns document sources
5. **Async Processing**: Non-blocking frontend with loading states
6. **Error Handling**: Graceful failures at each layer