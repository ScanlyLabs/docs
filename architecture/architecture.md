# Scanly 아키텍처 문서

## 1. 아키텍처 개요

Scanly는 **DDD(Domain-Driven Design) 기반 Layered Architecture**를 채택합니다.

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                       │
│              (Controller, Request/Response DTO)             │
├─────────────────────────────────────────────────────────────┤
│                    Application Layer                        │
│           (Application Service, Command/Query DTO)          │
├─────────────────────────────────────────────────────────────┤
│                      Domain Layer                           │
│    (Entity, Value Object, Domain Service, Repository I/F)   │
├─────────────────────────────────────────────────────────────┤
│                   Infrastructure Layer                      │
│  (Repository Impl, External API, Persistence, Messaging)    │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 레이어별 책임

### 2.1 Presentation Layer
외부 요청을 받아 Application Layer로 전달하고 응답을 반환합니다.

| 구성요소 | 책임 |
|---------|------|
| Controller | HTTP 요청/응답 처리, 입력 검증 |
| Request DTO | 클라이언트 요청 데이터 구조 |
| Response DTO | 클라이언트 응답 데이터 구조 |
| Exception Handler | 전역 예외 처리 및 에러 응답 변환 |

**규칙**
- 비즈니스 로직 포함 금지
- Application Service만 의존
- Request 검증은 Bean Validation 활용

### 2.2 Application Layer
유스케이스를 조율하고 트랜잭션 경계를 관리합니다.

| 구성요소 | 책임 |
|---------|------|
| Application Service | 유스케이스 조율, 트랜잭션 관리 |
| Command DTO | 상태 변경 요청 데이터 |
| Query DTO | 조회 요청/응답 데이터 |
| Event Publisher | 도메인 이벤트 발행 |

**규칙**
- 비즈니스 로직은 Domain Layer에 위임
- 하나의 메서드 = 하나의 유스케이스
- 여러 도메인을 조율하는 역할

### 2.3 Domain Layer
핵심 비즈니스 로직과 규칙을 포함합니다.

| 구성요소 | 책임 |
|---------|------|
| Entity | 고유 식별자를 가진 도메인 객체 |
| Value Object | 불변 값 객체, 동등성 비교 |
| Domain Service | 여러 Entity에 걸친 비즈니스 로직 |
| Repository Interface | 영속성 추상화 인터페이스 |
| Domain Event | 도메인 내 발생하는 이벤트 |

**규칙**
- 외부 레이어 의존 금지 (순수 Java)
- 풍부한 도메인 모델 (Rich Domain Model)
- 불변성 최대한 유지

### 2.4 Infrastructure Layer
기술적 구현 세부사항을 담당합니다.

| 구성요소 | 책임 |
|---------|------|
| Repository Impl | JPA/QueryDSL 기반 구현 |
| External API Client | 외부 서비스 연동 (OAuth, Push 등) |
| Message Publisher | 이벤트/메시지 발행 구현 |
| Config | 기술 설정 (DB, Redis, S3 등) |

**규칙**
- Domain Layer의 인터페이스 구현
- 기술 종속적 코드 격리
- 외부 서비스 장애 격리

---

## 3. 의존성 규칙

```
Presentation → Application → Domain ← Infrastructure
                              ↑
                              │
                    (의존성 역전 - DIP)
```

- 상위 레이어는 하위 레이어에만 의존
- Domain Layer는 어떤 레이어에도 의존하지 않음
- Infrastructure는 Domain의 인터페이스를 구현 (DIP)

---

## 4. 도메인 구조

### 4.1 Bounded Context

```
┌─────────────────────────────────────────────────────────────┐
│                        Scanly                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │    Member    │  │     Card     │  │   CardBook   │      │
│  │   Context    │←→│   Context    │←→│   Context    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         ↑                                    ↓              │
│         │              ┌──────────────┐      │              │
│         └──────────────│   Meeting    │←─────┘              │
│                        │   Context    │                     │
│                        └──────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 도메인별 Entity/Value Object

#### Member Context
```
Member (Entity)
├── MemberId (VO)
├── Email (VO)
├── Password (VO)
├── Username (VO)
└── MemberStatus (VO - Enum)
```

#### Card Context
```
Card (Entity)
├── CardId (VO)
├── MemberId (VO)
├── Profile (VO)
│   ├── Name
│   ├── Title
│   ├── Company
│   ├── Phone
│   └── Email
├── SocialLinks (VO Collection)
└── QrCode (VO)

SocialLink (Entity)
├── SocialLinkId (VO)
├── LinkType (VO - Enum)
└── Url (VO)
```

#### CardBook Context
```
CardBook (Entity)
├── CardBookId (VO)
├── MemberId (VO)
├── CardId (VO)
├── ProfileSnapshot (VO)
├── GroupId (VO - nullable)
├── Memo (VO)
├── IsFavorite (VO)
├── SavedAt (VO)
└── UpdatedAt (VO)

Group (Entity)
├── GroupId (VO)
├── MemberId (VO)
└── GroupName (VO)

Tag (Entity)
├── TagId (VO)
├── MemberId (VO)
├── TagName (VO)
└── Color (VO)

ScanHistory (Entity)
├── ScanHistoryId (VO)
├── ScannerId (VO)
├── ScannedCardId (VO)
└── ScannedAt (VO)
```

#### Meeting Context (Phase 3)
```
Meeting (Entity)
├── MeetingId (VO)
├── RequesterId (VO)
├── ReceiverId (VO)
├── MeetingInfo (VO)
├── ProposedTimes (VO Collection)
└── MeetingStatus (VO - Enum)

MeetingAvailability (Entity)
├── AvailabilityId (VO)
├── MemberId (VO)
└── TimeSlots (VO Collection)
```

---

## 5. 패키지 구조

```
com.scanly
├── member/
│   ├── presentation/
│   │   ├── MemberController.java
│   │   ├── AuthController.java
│   │   └── dto/
│   │       ├── request/
│   │       └── response/
│   ├── application/
│   │   ├── MemberService.java
│   │   ├── AuthService.java
│   │   └── dto/
│   │       ├── command/
│   │       └── query/
│   ├── domain/
│   │   ├── Member.java
│   │   ├── MemberRepository.java
│   │   └── vo/
│   │       ├── MemberId.java
│   │       ├── Email.java
│   │       ├── Password.java
│   │       └── Username.java
│   └── infrastructure/
│       ├── MemberRepositoryImpl.java
│       ├── MemberJpaRepository.java
│       └── oauth/
│           ├── GoogleOAuthClient.java
│           ├── AppleOAuthClient.java
│           └── KakaoOAuthClient.java
│
├── card/
│   ├── presentation/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
│
├── cardbook/
│   ├── presentation/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
│
├── meeting/
│   ├── presentation/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
│
└── common/
    ├── exception/
    │   ├── BusinessException.java
    │   ├── ErrorCode.java
    │   └── GlobalExceptionHandler.java
    ├── response/
    │   └── ApiResponse.java
    ├── config/
    │   ├── JpaConfig.java
    │   ├── RedisConfig.java
    │   └── S3Config.java
    └── util/
        └── ...
```

---

## 6. 기술 스택

### Backend
| 기술                | 용도 |
|-------------------|------|
| Java 21           | 언어 |
| Spring Boot 4.0.1 | 프레임워크 |
| Spring Security   | 인증/인가 |
| Spring Data JPA   | ORM |
| QueryDSL          | 동적 쿼리 |
| PostgreSQL        | 주 데이터베이스 |
| Redis             | 캐시, 세션 |
| AWS S3            | 이미지/QR 저장 |

### Infrastructure
| 기술 | 용도 |
|-----|------|
| AWS/GCP | 클라우드 |
| CloudFront | CDN |
| FCM | 푸시 알림 |
| Docker | 컨테이너 |

---

## 7. 횡단 관심사

### 7.1 인증/인가
```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│    Client    │───→│ JWT Filter   │───→│  Controller  │
└──────────────┘    └──────────────┘    └──────────────┘
                           │
                    ┌──────┴──────┐
                    │ Token Valid? │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
           [Valid]    [Expired]    [Invalid]
              │            │            │
              ↓            ↓            ↓
          Continue    Refresh/401    401 Error
```

- Access Token: JWT (1시간)
- Refresh Token: Redis 저장 (14일)
- 소셜 로그인: OAuth 2.0

### 7.2 예외 처리
```java
// 도메인 예외
BusinessException
├── MemberNotFoundException
├── DuplicateEmailException
├── InvalidPasswordException
├── CardNotFoundException
└── ...

// 인프라 예외
InfrastructureException
├── ExternalApiException
├── FileUploadException
└── ...
```

### 7.3 로깅
- 요청/응답 로깅 (AOP)
- 에러 로깅 (Slack 연동)
- 성능 로깅 (메트릭)

### 7.4 캐싱 전략
| 대상 | TTL | 무효화 시점 |
|-----|-----|-----------|
| 명함 조회 (타인) | 5분 | 명함 수정 시 |
| 사용자 정보 | 10분 | 정보 수정 시 |
| QR 코드 | 1시간 | - |

---

## 8. 데이터 흐름 예시

### 명함 저장 + 상호 교환
```
[Client]
    │
    ▼ POST /api/cardbook
[CardBookController]
    │
    ▼ SaveCardCommand
[CardBookService]
    │
    ├─→ [CardRepository.findById()]     // 명함 조회
    │
    ├─→ [SavedCard.create()]            // 도메인 생성
    │
    ├─→ [SavedCardRepository.save()]    // 저장
    │
    └─→ [EventPublisher.publish()]      // CardSavedEvent
              │
              ▼
    [CardExchangeEventHandler]
              │
              ├─→ [NotificationService] // 푸시 알림
              │
              └─→ [SavedCardRepository] // 상대방 명함첩 저장
```

---

## 9. API 설계 원칙

### RESTful 규칙
- 리소스 중심 URI
- HTTP Method 의미에 맞게 사용
- 적절한 상태 코드 반환

### 응답 형식
```json
{
  "success": true,
  "data": { ... },
  "error": null
}

{
  "success": false,
  "data": null,
  "error": {
    "code": "MEMBER_NOT_FOUND",
    "message": "회원을 찾을 수 없습니다."
  }
}
```

### 페이징
```json
{
  "success": true,
  "data": {
    "content": [...],
    "page": 0,
    "size": 20,
    "totalElements": 100,
    "totalPages": 5,
    "hasNext": true
  }
}
```

---

## 10. 확장 고려사항

### 향후 확장
- **CQRS 패턴**: 읽기/쓰기 분리 (트래픽 증가 시)
- **Event Sourcing**: 이벤트 기반 상태 관리 (감사 로그 필요 시)
- **Microservice**: 도메인별 서비스 분리 (규모 확장 시)

### 현재 MVP 접근
- 모놀리식 구조 유지
- 도메인 경계만 명확히 분리
- 향후 분리 가능하도록 설계
