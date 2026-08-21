# claude-skills

Spring Boot 백엔드 개발 규약을 담은 Claude Code 스킬 마켓플레이스.

## 설치

```bash
claude plugin marketplace add github:YoungHoSeong/claude-skills
cd <프로젝트>
claude plugin install backend-conventions@espay-backend-skills -s project
```

`-s project` 로 설치하면 해당 프로젝트에서만 활성화되고 다른 프로젝트에 딸려가지 않는다.

## 제공 스킬

| 스킬 | 사용 시점 |
|---|---|
| `spring-boot` | 컨트롤러·서비스·리포지토리·시큐리티·설정 작성 시 |
| `software-architecture` | 모듈 구성, 프로젝트 구조, 서비스 설계 시 |
| `mysql-design` | 테이블·JPA 엔티티·스키마 설계 시 |
| `api-design` | REST 엔드포인트와 공통 응답 형식 설계 시 |

## 수정

`skills/<이름>/SKILL.md` 를 고치고 커밋하면 된다.
설치한 쪽에서는 `claude plugin update backend-conventions` 로 반영한다(재시작 필요).
