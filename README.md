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

## 스킬 수정·추가

기존 스킬 수정은 `skills/<이름>/SKILL.md` 를 고치고 push 하면 된다.

**새 스킬은 `skills/<이름>/SKILL.md` 폴더만 만들면 된다.**
`marketplace.json` 은 손대지 않는다 — `skills` 배열을 두지 않아 폴더가 자동 인식된다.

설치한 쪽에서 반영하는 방법:

```bash
claude plugin marketplace update espay-backend-skills   # 저장소에서 최신 내용 가져오기
claude plugin update admin-core-conventions             # 플러그인 갱신 (재시작 필요)
```

마켓플레이스를 로컬 경로로 등록한 경우 `marketplace update` 만으로 즉시 반영된다.

## 다른 프로젝트를 추가하려면

`marketplace.json` 의 `plugins` 배열에 항목을 추가한다. 한 저장소가 여러 플러그인을 내보낼 수 있다.
