---
config:
  layout: elk
  look: classic
---
flowchart TB
    %% 외부 사용자
    User["👤 일반 사용자"]
    
    %% DMZ 영역
    subgraph DMZ["🌐 DMZ (Public Network)"]
        NGINX["Nginx Reverse Proxy<br/>(dmz)<br/>/view, /api, /management, /ai"]
        SurveyPage["Survey Page<br/>(Next.js)<br/>/view"]
        AIBOT["AI Bot<br/>/ai"]
    end
    
    %% Private Network
    subgraph PrivateNetwork["🔒 Private Network"]
        SurveyWAS["Survey WAS<br/>(Application Server)"]
        AdminUI["Admin UI<br/>(Management Console)"]
        InternalUser["👤 병원 내부 직원"]
    end
    
    %% Database Layer
    subgraph Database["💾 Database Layer"]
        PostgreSQL[("PostgreSQL<br/>(Main DB)")]
        MongoDB[("MongoDB<br/>(Document Store)")]
        Redis[("Redis<br/>(Cache/Session)")]
    end
    
    %% External Systems
    subgraph External["☁️ External Systems"]
        EMS["SNUH EMS<br/>(SMS Service)"]
        ODS_CDW["ODS CDW<br/>(Data Warehouse)"]
        ChatGPT["🤖 ChatGPT API"]
    end
    
    %% User Flow
    User -->|"설문 시스템 접근"| NGINX
    NGINX -->|"라우팅: /view"| SurveyPage
    NGINX -->|"라우팅: /api, /management"| SurveyWAS
    NGINX -->|"라우팅: /ai"| AIBOT
    
    %% AI Bot
    AIBOT -->|"AI 응답 생성"| ChatGPT
    
    %% Internal User
    InternalUser -.->|"문진 메타 관리<br/>& 모니터링"| SurveyWAS
    InternalUser -.->|"관리 콘솔 접근"| AdminUI
    
    %% WAS Connections
    SurveyWAS -->|"설문 메타 CRUD"| PostgreSQL
    SurveyWAS -->|"설문 데이터 CRUD"| MongoDB
    SurveyWAS -->|"세션/문진키 관리"| Redis
    SurveyWAS -.->|"설문 대상자 추출"| ODS_CDW
    SurveyWAS -.->|"SMS 발송 요청"| EMS
    AdminUI -->|"관리 API 호출"| SurveyWAS
    
    %% Styles
    classDef userStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px,color:#000
    classDef dmzStyle fill:#c8e6c9,stroke:#388e3c,stroke-width:2px,color:#000
    classDef privateStyle fill:#e1f5fe,stroke:#0277bd,stroke-width:2px,color:#000
    classDef dbStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    classDef externalStyle fill:#ffe0b2,stroke:#e65100,stroke-width:2px,color:#000
    
    class User,InternalUser userStyle
    class NGINX,SurveyPage,AIBOT dmzStyle
    class SurveyWAS,AdminUI privateStyle
    class PostgreSQL,MongoDB,Redis dbStyle
    class EMS,ODS_CDW,ChatGPT externalStyle
