---
name: software-architecture
description: admin-core 의 DDD·헥사고날 패키지 구조와 계층 의존 규칙. 새 클래스를 어디에 둘지, 계층 간 호출이 허용되는지 판단할 때 사용한다.
---

# 아키텍처 규약 (admin-core)

이 규약은 문서가 아니라 `src/test/java/com/espay/admincore/architecture/` 의 테스트가 강제한다.
위반하면 빌드가 깨진다. 애매하면 `docs/architecture/아키텍처_가이드.md` 를 본다.

## 의존 방향

```
adapter.in  ──→ application ──→ domain
adapter.out ──→ application ──→ domain
config      ──→ adapter / application
```

역방향 의존은 없다. domain 은 아무것도 모른다.

## 패키지 구조

```
com.espay.admincore
├─ domain.model.{업무}        Aggregate 와 업무 규칙 (Java 표준 타입에만 의존)
├─ application
│  ├─ dto.{업무}              Command, Query, Result, Source
│  ├─ port.in.{업무}          UseCase 인터페이스
│  ├─ port.out.{업무}         외부 시스템 경계
│  └─ service.{업무}          UseCase 구현
├─ adapter
│  ├─ in.web.{업무}           Controller + request/ + response/
│  ├─ in.security             JWT 필터, 메뉴 권한 AuthorizationManager
│  └─ out                     persistence(JPA/QueryDSL), redis, security, mail, file.excel
├─ config                     Spring Bean·Security·CORS 설정만 (업무 규칙 금지)
└─ common                     api(공통 응답), excel(DSL), logging
```

업무 단위는 `auth`, `user`, `role`, `menu`, `history`, `file` 여섯 가지다.
새 클래스는 먼저 소유 업무를 정한다. `service.impl` 처럼 기술 종류로 묶거나
`util`·`helper`·`manager` 패키지를 만들지 않는다.

## import 금지 규칙 (CoreDependencyRuleTest)

- `application/` 금지: `jakarta.`, `org.apache.poi.`, `adapter.`, `config.`
- `application/` 에서 허용하는 Spring import 는 **정확히 두 개**:
  `org.springframework.stereotype.Service`, `org.springframework.transaction.annotation.Transactional`
- `domain/` 금지: `org.springframework.`, `jakarta.`, `org.apache.poi.`, `application.`, `adapter.`, `common.`, `config.`
- `common/excel/` 은 Java 표준 타입만 (POI 는 `adapter/out/file/excel/` 에만)

## Aggregate

- Aggregate Root(`AdminUser`, `AdminRole`, `LoginHistory`, `FileHistory`)는
  `record` 가 아닌 **`final class`**, **모든 생성자는 `private`**
- 신규 생성 `create`, DB 복원 `reconstitute`, 변경은 업무 이름(`changePassword`, `updateOtpSecret`)
- 이력은 사건 이름(`LoginHistory.otpFailed(...)`)
- `Data`, `Info`, `Manager`, `Helper`, `Util` 접미사 금지
- `domain/model` 에 `*CreateData.java`, `*UpdateData.java` 금지

## Application Service

- **다른 Service 를 호출하지 않는다** — `port.in` 타입 필드를 가질 수 없다
- 여러 Aggregate 와 Port 의 순서만 조정한다
- `@Service` 필수, **클래스 레벨 `@Transactional` 금지**
- 트랜잭션은 public 유스케이스 메서드에 선언: 조회 `@Transactional(readOnly = true)`, 변경 `@Transactional`
- private 메서드에 붙이지 않고 self-invocation 에 트랜잭션을 기대하지 않는다
- 감사 이력은 본 작업 실패와 무관해야 하므로 Persistence Adapter 에서
  `@Transactional(propagation = Propagation.REQUIRES_NEW)`

## DTO 규칙 (MethodParameterConventionTest)

`application.dto.*` 의 `*Command` / `*Query` / `*Source` record 는

- 정적 `of(...)` 팩터리를 반드시 제공
- **필드 3개 이상** (2개 이하면 Parameter Object 를 만들지 말고 직접 파라미터로)
- **선언 파일 밖에서 `new` 금지** (운영·테스트 코드 모두)

Aggregate 영속성 Port 의 `save` 는 Aggregate Root 를 그대로 받고 그대로 반환한다 (중계 Command 금지).

## 변환 메서드 명명

| 이름 | 사용 시점 |
|---|---|
| `of(...)` | 여러 명시적 값을 조립 (Command·Query·Source 의 유일한 생성 경로) |
| `from(source)` | 다른 객체 하나에서 변환 (Response 는 Result 에서 `from`) |
| `toCommand()` / `toQuery()` | 현재 객체가 변환의 출발점 (Request 에서) |
| `toDomain()` / `toEntity()` | Persistence Mapper 의 변환 방향 |

같은 변환에 `from` 과 `toX` 를 동시에 만들지 않는다.

## 신규 기능 순서

1. `domain.model.{업무}` 업무 규칙
2. `application.dto` Command / Query / Result
3. `port.in`, `port.out`
4. `application.service` 조정만
5. `adapter.in.web.{업무}` Request·Response·Controller
6. `adapter.out` 기술 구현
7. 변경 유스케이스에 `@Transactional`
8. `SecurityConfig` 접근 규칙 등록
9. 문서화 테스트 + `docs/api/API_정의서.md`
