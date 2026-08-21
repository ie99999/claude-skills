---
name: spring-boot
description: admin-core 의 Spring Boot 작성 규약 — 의존성 주입, 검증, 인증, 로깅, 테스트 전략. 서비스·필터·설정 클래스를 만들 때 사용한다.
---

# Spring Boot 규약 (admin-core)

Spring Boot 3.3.7 / Java 21 / Gradle 8.5.
`gradlew` 에 실행 권한이 없어 `sh gradlew` 로 실행하고, Gradle 8.5 는 JDK 25 에서 동작하지 않으므로
`export JAVA_HOME=$(/usr/libexec/java_home -v 21)` 가 매 세션 필요하다.

## 의존성 주입

- 생성자 주입만 사용, `@Autowired` 필드 주입 금지
- `@RequiredArgsConstructor` + `private final`
- Lombok: `@Getter`, `@Builder` 사용. Aggregate 에는 `@Setter` 금지 (업무 메서드로 상태 변경)

## 검증

- Request record 에 Jakarta Validation (`@NotBlank`, `@Size`, `@Email`, `@Pattern`)
- Controller 파라미터에 `@Valid`
- `MethodArgumentNotValidException` 은 `GlobalExceptionHandler` 가 공통 봉투로 변환

## 인증·인가

Stateless JWT 다. `JwtAuthenticationFilter` 가 전체 요청에 적용되므로 **API 마다 필터를 추가하지 않는다.**

로그인 흐름: ID/비밀번호 → Pre-Auth JWT + Redis 상태 → OTP 검증 → Access Token + HttpOnly Refresh Cookie.
CSRF 는 `POST /api/v1/auth/refresh` 와 `/logout` 에만 적용된다.

## Redis

- 단순 CRUD 는 `@RedisHash` + `ListCrudRepository`
- 조건 확인과 변경이 분리되어 경쟁 조건이 생기는 작업(사전 인증 상태 소비, OTP 실패 횟수 증가·한도 도달 시 삭제,
  Refresh Token 해시 교체)만 Repository Fragment + Lua Script
- **Application Service 에서 `StringRedisTemplate`·`RedisScript` 를 직접 쓰지 않는다**

## 로깅

- SLF4J + `@Slf4j`, `System.out.println` 금지
- 업무 흐름 `log.info`, 예외 `log.error`(stacktrace 포함), 디버깅 `log.debug`

## 테스트 전략

**이 프로젝트의 테스트는 DB·Redis·Docker 없이 돈다. 이 특성을 깨지 않는다.**

- 업무 로직은 `@ExtendWith(MockitoExtension.class)` 순수 단위 테스트
- **`@SpringBootTest`, `@DataJpaTest` 를 추가하지 않는다.** Testcontainers 는 선언만 되어 있고 쓰지 않는다
- Spring 컨텍스트 검증이 필요하면 `ApplicationContextRunner` (`ApplicationServiceComponentScanTest` 참고)
- `@WebMvcTest` 는 `adapter/in/web/**/*ControllerTest` 의 **문서 생성 목적**으로만 쓴다.
  업무 로직 검증용으로 새로 만들지 않는다
- 기준선: 전체 130개 중 `OtpVerificationServiceTest` 3건이 최초 커밋부터 실패한다. 자신이 깨뜨린 것이 아니다

## 주석

코드 문장을 반복하지 않고 **처리 이유와 업무 의미**를 쓴다.
보안 검증 순서·원자 연산·실패 이력처럼 순서에 이유가 있으면 그 이유를 남긴다.
Aggregate 와 public 팩터리·행위 메서드는 Javadoc 으로 필드의 업무 의미, `null` 가능 조건,
시간·TTL 단위, 해시/Secret 의 보안 의미를 설명한다. 문서와 주석은 한국어로 쓴다.
