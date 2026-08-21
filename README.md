# claude-skills

espay 백엔드 프로젝트용 Claude Code 스킬 마켓플레이스.

## admin-core 팀원 설치

```bash
claude plugin marketplace add github:ie99999/claude-skills
cd <admin-core 클론 경로>
claude plugin install admin-core-conventions@espay-backend-skills -s project
```

`-s project` 로 설치하면 해당 프로젝트에서만 활성화되고 다른 프로젝트에 딸려가지 않는다.
적용은 다음 세션부터다.

## 제공 스킬 (admin-core-conventions)

| 스킬 | 사용 시점 |
|---|---|
| `software-architecture` | 새 클래스 위치, 계층 의존, Aggregate·DTO 규칙 판단 시 |
| `api-design` | 컨트롤러, Request/Response, 공통 응답 봉투, SecurityConfig 등록 시 |
| `mysql-design` | 테이블, JPA 엔티티, Mapper, QueryDSL 작성 시 |
| `spring-boot` | 의존성 주입, 검증, 인증, 로깅, 테스트 전략 |

내용은 admin-core 저장소의 `CLAUDE.md` 와 `src/test/.../architecture/` 가 강제하는 규약을 따른다.
규약이 바뀌면 이 저장소도 함께 갱신한다.

## 수정

`skills/<이름>/SKILL.md` 를 고치고 push 한다.
설치한 쪽에서는 `claude plugin update admin-core-conventions` 로 반영한다(재시작 필요).

## 다른 프로젝트를 추가하려면

`marketplace.json` 의 `plugins` 배열에 항목을 추가한다. 한 저장소가 여러 플러그인을 내보낼 수 있다.
