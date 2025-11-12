---
config:
  layout: elk
  look: classic
---
graph TB
    %% ============================================
    %% Internet & Users
    %% ============================================
    subgraph Internet["🌍 Internet"]
        User["👤 일반 사용자"]
        ChatGPT["🤖 ChatGPT API"]
    end
    
    %% ============================================
    %% DMZ (Demilitarized Zone)
    %% ============================================
    subgraph DMZ["🌐 DMZ Layer"]
        direction TB
        NGINX["🔧 Nginx<br/>Reverse Proxy<br/>(Load Balancer)"]
        
        subgraph WebLayer["Presentation Layer"]
            SurveyPage["📱 Survey Page<br/>(Next.js)"]
            AIBOT["🤖 AI Bot<br/>(ChatGPT Interface)"]
        end
    end
    
    %% ============================================
    %% Private Network
    %% ============================================
    subgraph PrivateZone["🔒 Private Zone (Internal Network)"]
        direction TB
        
        subgraph AppLayer["Application Layer"]
            SurveyWAS["⚙️ Survey WAS<br/>(Spring Boot)"]
        end
        
        subgraph AdminLayer["Management Layer"]
            AdminUI["🖥️ Admin Console<br/>(Management UI)"]
            InternalUser["👤 내부 직원<br/>(Administrator)"]
        end
    end
    
    %% ============================================
    %% Data Layer
    %% ============================================
    subgraph DataZone["💾 Data Layer"]
        direction LR
        PostgreSQL[("📊 PostgreSQL<br/>(설문 메타데이터)")]
        MongoDB[("📄 MongoDB<br/>(설문 응답 데이터)")]
        Redis[("⚡ Redis<br/>(세션/캐시)")]
    end
    
    %% ============================================
    %% External Systems
    %% ============================================
    subgraph ExternalSystems["☁️ External Systems (Enterprise)"]
        direction TB
        EMS["📧 SNUH EMS<br/>(SMS Gateway)"]
        CDW["🏥 ODS CDW<br/>(Clinical Data<br/>Warehouse)"]
    end
    
    %% ============================================
    %% Connections
    %% ============================================
    
    %% Internet to DMZ
    User -->|"HTTPS<br/>(443)"| NGINX
    
    %% DMZ Internal
    NGINX -.->|"/view"| SurveyPage
    NGINX -.->|"/api<br/>/management"| SurveyWAS
    NGINX -.->|"/ai"| AIBOT
    
    %% AI Bot to External
    AIBOT -->|"HTTPS API"| ChatGPT
    
    %% Admin Access
    InternalUser -->|"VPN/Internal<br/>Network"| AdminUI
    AdminUI -->|"REST API"| SurveyWAS
    
    %% Application to Database
    SurveyWAS -->|"JDBC"| PostgreSQL
    SurveyWAS -->|"MongoDB<br/>Driver"| MongoDB
    SurveyWAS -->|"Redis<br/>Client"| Redis
    
    %% Application to External Systems
    SurveyWAS -.->|"대상자 조회<br/>(SQL/API)"| CDW
    SurveyWAS -.->|"SMS 발송<br/>(API)"| EMS
    
    %% ============================================
    %% Styles
    %% ============================================
    classDef internetStyle fill:#fff9c4,stroke:#f57f17,stroke-width:3px,color:#000
    classDef dmzStyle fill:#c8e6c9,stroke:#388e3c,stroke-width:3px,color:#000
    classDef privateStyle fill:#e1f5fe,stroke:#0277bd,stroke-width:3px,color:#000
    classDef dataStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:3px,color:#000
    classDef externalStyle fill:#ffe0b2,stroke:#e65100,stroke-width:3px,color:#000
    classDef componentStyle fill:#ffffff,stroke:#333,stroke-width:2px,color:#000
    
    class Internet internetStyle
    class DMZ,WebLayer dmzStyle
    class PrivateZone,AppLayer,AdminLayer privateStyle
    class DataZone dataStyle
    class ExternalSystems externalStyle
    class User,InternalUser,ChatGPT,NGINX,SurveyPage,AIBOT,SurveyWAS,AdminUI,PostgreSQL,MongoDB,Redis,EMS,CDW componentStyle
