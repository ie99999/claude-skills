---
name: mysql-design
description: MySQL table and JPA entity design conventions. Use when creating tables, entities, or discussing database schema.
---

# MySQL Design Convention

## When to activate
- Creating new JPA entities or tables
- Database schema design or migration
- Writing Flyway migration DDL

## Table Rules
- 테이블명: snake_case, 복수형 (users, orders, order_items)
- PK: BIGINT AUTO_INCREMENT, 컬럼명 `id`
- FK 컬럼명: `{참조테이블_단수}_id` (user_id, order_id)
- **FK 제약조건 사용하지 않음** (애플리케이션 레벨에서 관리)
- 모든 테이블 필수 컬럼:
  - `created_date` DATETIME NOT NULL
  - `last_modified_date` DATETIME NOT NULL
- charset: utf8mb4, collation: utf8mb4_unicode_ci
- **모든 테이블과 컬럼에 COMMENT 필수** (V1 패턴 준수)
  - 테이블: `) COMMENT='반려동물 정보'`
  - 컬럼: `name VARCHAR(50) NOT NULL COMMENT '반려동물 이름'`

## Column Rules
- 컬럼명: snake_case
- BOOLEAN → TINYINT(1)
- 금액: DECIMAL(12,0)
- 좌표: DECIMAL(10,7)
- ENUM 대신 VARCHAR 사용
- **Java enum 매핑 컬럼은 VARCHAR(1)** — `CHAR(1)` 금지
  (Hibernate 가 CHAR(1) 을 DB ENUM 으로 요구해 마이그레이션을 또 추가하게 됨)
- 문자열 기본: VARCHAR(255), 긴 텍스트: TEXT
- NOT NULL을 기본으로, NULL 허용은 명시적 이유 필요

## Flyway Migration
- `src/main/resources/db/migration/V{번호}__{설명}.sql`
- **적용된 마이그레이션 파일은 절대 수정하지 않는다** — 새 마이그레이션 생성
- 버전 번호는 순차 증가 (V1, V2, ...)

## Index Rules
- WHERE, JOIN에 자주 사용되는 컬럼에 INDEX
- 복합 인덱스: 카디널리티 높은 컬럼을 앞에 배치
- 유니크 제약: UNIQUE INDEX 사용
- 인덱스명: `idx_{테이블}_{컬럼}` (예: idx_payments_user_id)

## JPA Entity Mapping
- @Entity, @Table(name = "테이블명")
- @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
- @Column(nullable = false, length = 100)
- CreatedAndLastModifiedDate 베이스 클래스 상속 (@CreatedDate, @LastModifiedDate)
- 연관관계: 기본 LAZY 로딩
- @ManyToOne(fetch = FetchType.LAZY)
- @OneToMany(mappedBy = "...", cascade, orphanRemoval)
- Entity에 @Setter 사용 금지 (비즈니스 메서드로 상태 변경)

## Naming Convention (JPA ↔ DB)
- Entity: PascalCase (OrderItem)
- Table: snake_case 복수형 (order_items)
- Java field: camelCase (createdDate)
- DB column: snake_case (created_date)

## QueryDSL
- 쿼리 작성 시 QueryDSL 사용을 기본으로 한다
- 네이티브 쿼리는 QueryDSL로 불가능한 경우에만 사용
- QueryDsl{Domain}Repository 인터페이스 + QueryDsl{Domain}RepositoryImpl 구현
