---
name: api-design
description: RESTful API design with consistent response format for Spring Boot. Use when creating controllers or designing endpoints.
---

# API Design Convention

## When to activate
- Creating new REST controllers
- Designing API endpoints
- Discussing API structure

## Response Format
모든 API는 CommonResponse<T> 래퍼를 사용한다.

### Success
```json
{
  "code": "200",
  "message": "Success",
  "data": { ... }
}
```

### Error
```json
{
  "code": "400",
  "message": "에러 메시지",
  "data": null
}
```

### List with Pagination
```json
{
  "code": "200",
  "message": "Success",
  "data": {
    "content": [ ... ],
    "hasNext": false,
    "totalCount": 100,
    "totalPage": 10
  }
}
```

## Pagination Parameters
- `pageNo`: 페이지 번호 (0부터 시작)
- `pageSize`: 페이지 크기 (기본 10)

## URL Convention
- 복수형 명사 사용: `/api/v1/users`
- 행위는 HTTP method로 표현 (동사 URL 금지)
- 중첩 리소스: `/api/v1/users/{userId}/orders`
- 버전 prefix: `/api/v1/`
- kebab-case 사용: `/api/v1/order-items`
- Base URL: `http://localhost:8080/api/v1`

## HTTP Methods
- GET: 조회 (단건, 목록)
- POST: 생성
- PUT: 전체 수정
- PATCH: 부분 수정
- DELETE: 삭제

## Status Codes
- 200: 조회/수정/삭제 성공 (CommonResponse로 래핑)
- 400: 잘못된 요청 (validation)
- 401: 인증 실패
- 403: 권한 없음
- 404: 리소스 없음
- 500: 서버 오류

## Naming
- Request DTO: `Create{Entity}Request`, `Update{Entity}Request`
- Response DTO: `{Entity}Response`
- Controller method: `create{Entity}`, `get{Entity}`, `update{Entity}`, `delete{Entity}`

## Exception Handling
- ExceptionType enum에 에러 코드 정의
- BusinessException을 던지면 GlobalExceptionHandler가 CommonResponse로 변환
- 도메인별 에러 코드 범위 할당 (예: selfwalk 381~384, snack 391~392)

## REST Docs
- 모든 API는 Spring REST Docs 테스트를 작성한다
- Controller 테스트에서 자동으로 스니펫 생성
- API 수정 시 REST Docs도 반드시 업데이트
