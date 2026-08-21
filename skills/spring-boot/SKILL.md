---
name: spring-boot
description: Spring Boot development best practices. Use when creating controllers, services, repositories, security, or configuration.
---

# Spring Boot Development

## When to activate
- Creating REST API controllers, services, repositories
- JPA entity and MySQL table design
- Spring Security, JWT authentication
- Exception handling, validation
- Configuration and properties

## Coding Rules
- 생성자 주입 사용, @Autowired 필드 주입 금지
- @RequiredArgsConstructor + private final
- Lombok 활용: @Getter, @Builder, @NoArgsConstructor(access = AccessLevel.PROTECTED)
- Entity에 @Setter 사용 금지 (비즈니스 메서드로 상태 변경)

## Validation
- @Valid + Jakarta Validation 어노테이션 사용
- @NotBlank, @NotNull, @Size, @Email, @Pattern
- Request DTO에 validation 어노테이션 적용
- MethodArgumentNotValidException 전역 처리

## Security
- Spring Security + JWT 기반 인증
- SecurityFilterChain으로 설정
- BCryptPasswordEncoder로 비밀번호 암호화
- Role 기반 인가: @PreAuthorize

## Logging
- SLF4J + @Slf4j 사용
- 비즈니스 로직: log.info
- 예외 상황: log.error (stacktrace 포함)
- 디버깅: log.debug
- System.out.println 사용 금지

## Test
- @SpringBootTest: 통합 테스트
- @WebMvcTest: Controller 단위 테스트
- @DataJpaTest: Repository 단위 테스트
- Mockito로 Service 단위 테스트
- Given-When-Then 패턴
