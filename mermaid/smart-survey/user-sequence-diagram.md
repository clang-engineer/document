sequenceDiagram
    autonumber
    
    actor User as 👤 사용자
    participant Gateway as 문진 ui gateway
    participant Auth as 문진 인증 확인
    participant Survey as 서베이 메인
    participant SurveyAPI as survey 시스템
    participant Redis as redis 시스템
    participant PostgreSQL as postgresql 시스템
    participant MongoDB as mongodb 시스템
    
    %% 초기 링크 클릭 및 인증
    Note over User,Auth: 📝 문진 링크 클릭 단계
    User->>Gateway: 문진 링크 클릭
    Gateway->>Auth: 문진 인증 확인 요청<br/>(or 유효하지 않은 링크 발생)
    
    %% 서베이 페이지 출력
    Note over Auth,Survey: 📄 서베이 페이지 출력
    Auth->>Survey: 서베이 페이지 출력
    
    %% 메시지 검증
    Note over Survey,SurveyAPI: ✅ 메시지 검증 요청
    Survey->>SurveyAPI: 메시지 검증 요청
    SurveyAPI->>SurveyAPI: 메시지 검증<br/>(유효한 링크인지, 기간-권한 여부등 검증)
    
    %% 인증 수단 적용
    Note over SurveyAPI,Auth: 🔐 문진 인증 수단 (세션방식) 적용
    SurveyAPI->>Auth: 문진 인증 수단 (세션방식) 적용
    Auth->>SurveyAPI: 문진 인증 결과
    
    %% 메타 정보 조회
    Note over SurveyAPI,PostgreSQL: 📊 문진 메타 조회
    SurveyAPI->>PostgreSQL: 문진 메타 조회
    PostgreSQL-->>SurveyAPI: 메시지 조회
    SurveyAPI-->>Survey: 메시지 조회
    Survey-->>User: 웨시지 응답
    
    %% 사용자 응답 입력
    Note over User,MongoDB: ✏️ 사용자 응답 제출
    User->>Survey: 웨시지 조회
    Survey->>Redis: 메시지 조회
    Redis-->>Survey: 웨시지 응답
    Survey->>SurveyAPI: 문진 정보 조회 (해시키)
    SurveyAPI->>PostgreSQL: 인증해야 할
    PostgreSQL-->>SurveyAPI: 단계별 응답
    SurveyAPI-->>Survey: 문진 정보 응답
    Survey-->>User: 문진 정보 표시
    
    %% 최종 제출
    Note over User,MongoDB: 💾 문진 정보 조회 (해시키)
    User->>Survey: 문진 제출, 수정
    Survey->>SurveyAPI: 문진 제출 요청
    SurveyAPI->>MongoDB: 문진 제출, 수정
    MongoDB-->>SurveyAPI: 저장 완료
    SurveyAPI-->>Survey: 제출 완료
    Survey-->>User: 완료 메시지
