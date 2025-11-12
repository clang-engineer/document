sequenceDiagram
    autonumber
    
    participant System as 🖥️ 문진시스템
    participant ODS as ODS
    participant Redis as Redis
    participant PostgreSQL as PostgreSQL
    participant SMS as 📱 SMS
    
    %% Phase 1: 초기 설정 및 대상자 조회
    rect rgb(255, 245, 230)
        Note over System,ODS: Phase 1: 발송 대상 세팅
        System -->> ODS: 문진 발송 목룩 추출 <br> (각 진료과에서 설정한 조건에 따른 환자 목록. ex> 진료예약 30일전 항암치료 중인 환자)
        ODS-->>System: 대상자 리스트
        
        System->>Redis: 문진해쉬키(진료일자 + 환자번호) + 정보(생년월일, 진료일자, 문진템플릿 id) 등록 
    end
    
    %% Phase 2: 최초 발송 (신규)
    rect rgb(230, 245, 255)
        Note over System,SMS: Phase 2: 최초 문자 발송 (신규 대상)
        System->>Redis: 최초 대상자 조회 (ex> 진료 7일전)
        Redis-->>System: 대상자 데이터
        System->>PostgreSQL: 등록된 문진키가 있는지 확인
        
        alt 최초 대상: 
            System->>SMS: 최초 문자 전송
            SMS-->>System: 전송 완료
            
            System->>PostgreSQL: 문진키, 상태, 발송기록 저장
            PostgreSQL-->>System: 저장 완료
        end
    end
    
    %% Phase 3: 독려 발송 (기존)
    rect rgb(240, 255, 240)
        Note over System,SMS: Phase 3: 독려 문자 발송 (미완료 대상)
        
        System->>Redis: 독려 대상자 조회 (ex> 진료 3일전)
        Redis-->>System: 대상자 데이터
        
        System->>PostgreSQL: 문진키 존재 & 상태 완료 x ?
        
        alt 문진키 존재하고 미완료 상태
            System->>SMS: 독려 문자 전송
            SMS-->>System: 전송 완료
            
            System->>PostgreSQL: 문진 상태, 발송 기록 저장
            PostgreSQL-->>System: 저장 완료
        end
    end
