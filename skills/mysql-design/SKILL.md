---
name: mysql-design
description: admin-core 의 DB 스키마와 영속성 계층 규약 — 테이블·JPA 엔티티·Mapper·QueryDSL. 테이블이나 엔티티를 만들 때 사용한다.
---

# DB·영속성 규약 (admin-core)

DB 는 **MariaDB** 다 (로컬 Docker, 포트 13308). 상세는 `docs/db/DB_정의서.md`.

## 스키마 관리

**Flyway 를 쓰지 않는다.** 스키마는 기동 시 `spring.sql.init` 이 적용한다.

```
spring.sql.init.schema-locations: classpath:db/schema/admin_core_schema.sql
spring.jpa.hibernate.ddl-auto: validate
```

- 테이블 변경은 `src/main/resources/db/schema/admin_core_schema.sql` 을 고친다
- `ddl-auto` 가 `validate` 이므로 엔티티와 스키마가 어긋나면 **기동이 실패한다**
- 역할·사용자·메뉴 권한 초기 데이터는 자동 적용되지 않는다 (`docs/db/DB_정의서.md` 3.2절 INSERT)

## 테이블 규칙

- 테이블명·컬럼명 snake_case
- Y/N 플래그는 `char(1)` (`result_yn`, `use_yn`)
- 생성·수정 시각은 `created_at`, `updated_at`
- 문자열은 용도에 맞는 길이를 명시 (`login_reason` 100, `fail_reason` 255, `client_ip` 64)

## 도메인과 엔티티를 분리한다

**Aggregate 에 JPA 애노테이션을 붙이지 않는다.** `domain/` 은 `jakarta.` import 자체가 금지다
(`CoreDependencyRuleTest` 가 검사한다). 영속성 모델은 별도로 둔다.

```
adapter/out/persistence/{업무}/
├─ entity/{Domain}JpaEntity.java        JPA 매핑 전용
├─ mapper/{Domain}PersistenceMapper.java  toDomain() / toEntity()
├─ repository/{Domain}JpaRepository.java
├─ repository/{Domain}QueryDslRepository.java
└─ {Domain}PersistenceAdapter.java      port.out 구현
```

## JPA 엔티티

```java
@Getter
@Entity
@Table(name = "login_logs")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class LoginHistoryJpaEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "hist_id")
    private Long id;

    @Column(name = "result_yn", nullable = false, length = 1, columnDefinition = "char(1)")
    private String resultYn;
}
```

- `@Setter` 금지, 기본 생성자는 `PROTECTED`
- 컬럼명을 `@Column(name = ...)` 으로 명시한다 (필드명과 다른 경우가 많다)
- 연관관계는 LAZY 기본

## Port 와 Adapter

- `port.out` 의 `save` 는 **Aggregate Root 를 그대로 받고 그대로 반환**한다 (중계 Command 금지)
- Adapter 가 Mapper 로 도메인 ↔ 엔티티를 변환한다
- 감사 이력 저장은 본 작업 실패와 무관해야 하므로 Adapter 에서
  `@Transactional(propagation = Propagation.REQUIRES_NEW)`

## QueryDSL

- 동적 검색 조건은 QueryDSL 을 기본으로 한다 (`{Domain}QueryDslRepository`)
- 네이티브 쿼리는 QueryDSL 로 불가능한 경우에만
- Q 클래스는 annotationProcessor 가 생성하므로 컴파일이 선행되어야 한다
