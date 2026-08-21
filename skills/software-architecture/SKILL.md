---
name: software-architecture
description: Clean Architecture and DDD patterns for Spring Boot. Use when creating modules, discussing project structure, or designing services.
---

# Architecture Convention

## When to activate
- Creating new service or module
- Discussing project structure
- Designing domain boundaries

## 애플리케이션 구조
- Spring Boot 단일 애플리케이션 (모놀리스)
- 루트 패키지: com.petcare
- 계층형/헥사고날 아키텍처 기반

## Package Structure
```
com.petcare.{module}/
├── adapter/
│   ├── web/
│   │   ├── in/            # 컨트롤러 (@ApiV1Controller)
│   │   └── model/         # Request/Response DTO
│   └── persistence/       # 리포지토리 (JPA + QueryDSL)
├── service/
│   └── model/             # 서비스 레이어 DTO (필요 시)
└── domain/                # JPA 엔티티, Enum
```

공통 관심사는 `common/`에 위치:
- `config/` — SecurityConfig, WebMvcConfig, QueryDslConfig
- `controller/` — @ApiV1Controller 어노테이션
- `exception/` — BusinessException, ExceptionType enum, GlobalExceptionHandler
- `filter/` — JwtAuthenticationFilter
- `model/` — CommonResponse<T>
- `jpa/` — CreatedAndLastModifiedDate 베이스 클래스

## Layer Rules
- Controller → Service만 호출 (Repository 직접 접근 금지)
- Service → Repository 호출, 비즈니스 로직 담당
- Entity를 Controller에 직접 노출하지 않음 (DTO 변환 필수)
- 패키지 구조 변경 금지

## Aggregate & JPA 연관관계 규칙
- 같은 Aggregate(같은 모듈) 내부: JPA 연관관계 사용
- 다른 Aggregate(다른 모듈) 간: ID 참조 + Application Service로 협력 (JPA 연관관계 금지)
- Aggregate Root → Child (1:N): `@OneToMany(mappedBy, fetch=LAZY)` + `@Builder.Default`
- Child → Root (N:1): `@ManyToOne(fetch=LAZY)` + `@JoinColumn`
- 팩토리 메서드(`create()`)는 ID 대신 엔티티 객체를 파라미터로 받는다

## Dependency Injection
- 생성자 주입만 사용 (@Autowired 필드 주입 금지)
- @RequiredArgsConstructor + private final 조합

## Service Layer
- @Transactional은 Service에서만 사용
- 읽기 전용: @Transactional(readOnly = true)
- 하나의 Service 메서드 = 하나의 비즈니스 유스케이스

## Exception Handling
- ExceptionType enum에 에러 코드 + 메시지 정의
- BusinessException(ExceptionType)을 던짐
- GlobalExceptionHandler + @RestControllerAdvice로 전역 처리
- 예외 응답도 CommonResponse 포맷 사용

## Naming Convention
- Controller: {Domain}Controller
- Service: {Domain}Service (Impl 접미사 사용하지 않음)
- Repository: {Domain}Repository, QueryDsl{Domain}Repository
- Entity: {Domain} (단수형, PascalCase)
- DTO: Create{Domain}Request, {Domain}Response
