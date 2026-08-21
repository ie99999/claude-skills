---
name: api-design
description: admin-core 의 REST 엔드포인트·공통 응답 봉투·Controller 작성 규약. 컨트롤러나 Request/Response 를 만들 때 사용한다.
---

# API 규약 (admin-core)

## 공통 응답 봉투

모든 REST 응답은 `ApiResponse<T>(code, message, data, error)` 를 쓴다.
`code` 는 실제 HTTP 상태와 일치해야 하며 `toResponseEntity()` 가 이 값을 상태 코드로 사용한다.

```json
{ "code": 200, "message": "요청이 정상적으로 처리되었습니다.", "data": { }, "error": null }
```

- `code` 는 **정수**다 (문자열 아님)
- 성공 시 `error` 는 `null`, 실패 시 `data` 가 `null`
- 생성은 `ApiResponse.created(data)` → 201
- 데이터 없는 성공은 `ApiResponse.ok()` → `data: null`
- 쿠키 동반 응답은 `toResponseEntity(cookie)`
- Excel 다운로드는 `ApiResponse.toExcelResponseEntity(fileName, bytes)` 로 `byte[]` 반환

## 목록 응답

페이지 응답은 `items` / `totalCount` / `page` / `size` 를 쓴다.
`content`·`hasNext`·`totalPage`·`pageNo`·`pageSize` 는 이 프로젝트의 형식이 **아니다**.

```json
{ "data": { "items": [ ], "totalCount": 100, "page": 0, "size": 10 } }
```

페이지 없는 목록은 `items` 만 둔다 (`MenuListResponse`).

## Controller

Input Port 만 호출한다. Output Port·JPA·Redis 를 직접 부르지 않고,
Domain 상태를 판단·변경하지 않으며, **private 업무 메서드나 private 응답 조립 메서드를 만들지 않는다.**
endpoint 흐름이 한 눈에 보여야 한다. 표준 형태는 한 줄이다.

```java
return ApiResponse.ok(UserListResponse.from(queryUseCase.getUsers(request.toQuery()))).toResponseEntity();
```

**Swagger 애노테이션을 붙이지 않는다.** 문서는 REST Docs 문서화 테스트가 생성한다.
과거의 `*Spec` 인터페이스는 전부 제거했고 `io.swagger.v3.oas.annotations.*` 는 운영 코드에서 쓰지 않는다.

## Request / Response

- Request 는 `adapter.in.web.{업무}.request`, Response 는 `.response` — 전역 패키지에 모으지 않는다
- Request record 에 Jakarta Validation 을 붙이고 `toCommand()` / `toQuery()` 로 변환
- Response 는 Result 에서 `from(result)` 로 변환
- 조회 조건은 `@ModelAttribute` 로 받고, 생략된 페이지 값은 `toQuery()` 에서 0 / 10 으로 보정

## URL

- 버전 prefix `/api/v1/`, 복수형 명사, 행위는 HTTP 메서드로 (동사 URL 금지)
- 중첩 리소스 `/api/v1/roles/{roleId}/menus`
- 포트는 API `10086`, Actuator `12086`

## 예외

- 업무 예외는 `BusinessException(ErrorCode, message)` (`application.exception`)
- Domain 예외(`UserNotFoundException` 등)는 `domain.exception` 에 두고 `GlobalExceptionHandler` 가 매핑
- 핸들러가 `ApiFailure` 를 거쳐 공통 봉투로 변환한다

## 보안 등록

신규 API 는 반드시 `config/SecurityConfig` 의 `authorizeHttpRequests` 에 등록한다.

| API 성격 | 규칙 |
|---|---|
| Access Token 없이 시작하는 인증 API | `permitAll()` |
| 로그인 사용자 공통 | `authenticated()` |
| 업무 조회 | `permissions.canView("MENU_CODE")` |
| 업무 생성·수정·삭제 | `permissions.canEdit("MENU_CODE")` |

**같은 URL 이면 GET 조회 규칙을 변경 규칙보다 먼저 선언한다.** 순서가 뒤바뀌면 GET 에도 EDIT 권한을 요구하게 된다.
컬렉션과 하위 경로(`"/x"`, `"/x/**"`)를 함께 등록한다.
메뉴 코드는 `USERS`, `ROLES`, `LOGIN_HISTORY`, `FILE_HISTORY`.

## 문서

새 엔드포인트는 `adapter/in/web/{업무}/{업무}ControllerTest` 에 문서화 테스트를 추가하고
`src/docs/asciidoc/{업무}.adoc` 에 절을 추가한다. 조각만 만들고 `.adoc` 에 넣지 않으면 문서에 나오지 않는다.
