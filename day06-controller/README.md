# Day 6: Controller와 REST API 기초

## 학습 목표
- Controller가 무엇이고 어떤 역할인지 이해
- HTTP 메서드(GET, POST, PUT, DELETE)의 의미 파악
- REST API가 무엇인지 이해

---

## 1. Controller란?

### 한 줄 정리
> **사용자의 요청을 받는 안내 데스크**

### 왜 필요해?

사용자가 서버에 요청을 보내면, **누군가 그 요청을 받아야** 합니다.
Controller가 그 역할을 합니다.

```
사용자: "회원 목록 보여줘!"
   ↓
Controller: "네, 접수했습니다. 잠시만요."
   ↓
Service에게 전달 → 결과 받음
   ↓
Controller: "여기 회원 목록입니다."
```

| 비유 | 설명 |
|------|------|
| 식당 직원 | 손님 주문 받음 → 주방에 전달 → 음식 가져다줌 |
| 안내 데스크 | 방문객 요청 받음 → 담당 부서에 연결 → 결과 전달 |
| 콜센터 상담원 | 전화 받음 → 담당자 연결 → 답변 전달 |

### Controller의 역할

| 역할 | 설명 |
|------|------|
| 요청 받기 | 사용자가 보낸 요청을 받음 |
| 데이터 전달 | 받은 데이터를 Service에게 넘김 |
| 응답 반환 | Service에서 처리한 결과를 사용자에게 돌려줌 |

**중요:** Controller는 직접 일하지 않고, **Service에게 일을 시킵니다.**

---

## 2. HTTP 메서드

### 한 줄 정리
> **"어떤 행동을 원하는지"를 알려주는 신호**

### 왜 필요해?

같은 URL이라도 **어떤 행동**을 원하는지 알아야 합니다.

```
/users  ← 이 URL로
- 목록을 보고 싶어? → GET
- 새로 만들고 싶어? → POST
```

### 4가지 기본 메서드

| 메서드 | 의미 | 비유 |
|--------|------|------|
| **GET** | "가져와줘" (조회) | 책 빌려보기 |
| **POST** | "새로 만들어줘" (생성) | 새 책 등록 |
| **PUT** | "바꿔줘" (수정) | 책 정보 수정 |
| **DELETE** | "지워줘" (삭제) | 책 삭제 |

### 실제 예시

| 행동 | HTTP 메서드 | URL |
|------|-------------|-----|
| 모든 사용자 조회 | GET | /users |
| 1번 사용자 조회 | GET | /users/1 |
| 새 사용자 생성 | POST | /users |
| 1번 사용자 수정 | PUT | /users/1 |
| 1번 사용자 삭제 | DELETE | /users/1 |

---

## 3. REST API

### 한 줄 정리
> **URL 설계 규칙** - "URL은 이렇게 만들어라"

### 왜 필요해?

규칙 없이 URL을 만들면:
```
❌ 나쁜 예시
/getUserList
/createNewUser
/deleteUserById?id=1
/user_update
```

규칙을 따르면:
```
✅ 좋은 예시 (REST API)
GET    /users      → 목록 조회
GET    /users/1    → 1번 조회
POST   /users      → 생성
PUT    /users/1    → 1번 수정
DELETE /users/1    → 1번 삭제
```

### REST API 규칙

| 규칙 | 설명 | 예시 |
|------|------|------|
| 명사 사용 | 동사 대신 명사 | `/users` (O) `/getUsers` (X) |
| 복수형 | 리소스는 복수형 | `/users` (O) `/user` (X) |
| 계층 구조 | 슬래시로 계층 표현 | `/users/1/posts` (1번 유저의 게시글) |
| 소문자 | URL은 소문자 | `/users` (O) `/Users` (X) |

### C# 개발자를 위한 비교

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 컨트롤러 | `[ApiController]` | `@Controller()` |
| GET | `[HttpGet]` | `@Get()` |
| POST | `[HttpPost]` | `@Post()` |
| PUT | `[HttpPut]` | `@Put()` |
| DELETE | `[HttpDelete]` | `@Delete()` |

---

## 4. Controller 만들기

### 기본 구조

```typescript
@Controller('users')  // 기본 경로: /users
export class UsersController {

  @Get()              // GET /users
  findAll() {
    return '모든 사용자 조회';
  }

  @Get(':id')         // GET /users/1, /users/2, ...
  findOne(@Param('id') id: string) {
    return `${id}번 사용자 조회`;
  }

  @Post()             // POST /users
  create(@Body() body: any) {
    return '사용자 생성';
  }

  @Put(':id')         // PUT /users/1
  update(@Param('id') id: string, @Body() body: any) {
    return `${id}번 사용자 수정`;
  }

  @Delete(':id')      // DELETE /users/1
  remove(@Param('id') id: string) {
    return `${id}번 사용자 삭제`;
  }
}
```

### 코드 설명

| 코드 | 의미 |
|------|------|
| `@Controller('users')` | 이 컨트롤러는 `/users` 경로 담당 |
| `@Get()` | GET 요청 처리 |
| `@Get(':id')` | GET 요청 + URL에서 id 받기 |
| `@Post()` | POST 요청 처리 |
| `@Put(':id')` | PUT 요청 + URL에서 id 받기 |
| `@Delete(':id')` | DELETE 요청 + URL에서 id 받기 |

---

## 5. 데이터 받는 방법

### 세 가지 방법

```
URL: /users/1?sort=name
     └─┬──┘ └─┬───┘
       │      └── Query (?뒤)
       └── Param (경로에 포함)

Body: POST 요청 시 본문에 담아서
```

### @Param() - URL 경로에서 받기

```typescript
// URL: /users/1
@Get(':id')
findOne(@Param('id') id: string) {
  return `${id}번 사용자`;  // "1번 사용자"
}

// URL: /users/1/posts/5
@Get(':userId/posts/:postId')
findPost(
  @Param('userId') userId: string,
  @Param('postId') postId: string,
) {
  return `${userId}번 유저의 ${postId}번 게시글`;
}
```

| 비유 | 설명 |
|------|------|
| 주소 | `/강남구/역삼동/123번지` → 각 부분이 의미 있음 |
| 폴더 경로 | `/문서/사진/여행/` → 계층적 의미 |

### @Query() - URL ?뒤에서 받기

```typescript
// URL: /users?page=2&limit=10
@Get()
findAll(
  @Query('page') page: string,   // "2"
  @Query('limit') limit: string, // "10"
) {
  return `${page}페이지, ${limit}개씩`;
}

// 전체를 객체로 받기
@Get()
findAll(@Query() query: any) {
  console.log(query);  // { page: '2', limit: '10' }
}
```

| 비유 | 설명 |
|------|------|
| 검색 옵션 | "이름순 정렬, 10개씩 보기" 같은 부가 조건 |
| 필터 | "가격: 1만원~5만원, 색상: 빨강" |

### @Body() - 요청 본문에서 받기

```typescript
// POST /users
// 요청 본문: { "name": "Kim", "email": "kim@test.com" }
@Post()
create(@Body() body: any) {
  console.log(body);  // { name: 'Kim', email: 'kim@test.com' }
  return '사용자 생성됨';
}

// 특정 필드만 받기
@Post()
create(@Body('name') name: string) {
  return `${name}님 생성됨`;
}
```

| 비유 | 설명 |
|------|------|
| 택배 내용물 | 박스(요청) 안에 들어있는 실제 물건(데이터) |
| 신청서 내용 | 양식에 적힌 정보들 |

### 정리

| 데코레이터 | 어디서? | 언제 사용? | 예시 |
|------------|---------|-----------|------|
| `@Param()` | URL 경로 | ID 등 필수 식별자 | `/users/1` |
| `@Query()` | URL ?뒤 | 정렬, 필터, 페이징 | `?page=2&sort=name` |
| `@Body()` | 요청 본문 | 생성/수정할 데이터 | `{ name: "Kim" }` |

### C# 개발자를 위한 비교

| NestJS | ASP.NET Core |
|--------|--------------|
| `@Param('id')` | `[FromRoute]` 또는 `{id}` |
| `@Query('page')` | `[FromQuery]` |
| `@Body()` | `[FromBody]` |

---

## 6. 응답 처리

### 기본 응답

```typescript
@Get()
findAll() {
  // 객체/배열 반환 → 자동으로 JSON 변환
  return { users: [] };
}
```

### 상태 코드 변경

```typescript
@Post()
@HttpCode(201)  // 생성 성공: 201
create() {
  return { message: 'Created' };
}
```

### 에러 응답

```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  const user = this.findUser(id);

  if (!user) {
    throw new NotFoundException('사용자를 찾을 수 없습니다');
    // 자동으로 404 응답
  }

  return user;
}
```

### 자주 쓰는 예외

| 예외 | HTTP 코드 | 의미 |
|------|----------|------|
| `NotFoundException` | 404 | 찾을 수 없음 |
| `BadRequestException` | 400 | 잘못된 요청 |
| `UnauthorizedException` | 401 | 인증 필요 |
| `ForbiddenException` | 403 | 권한 없음 |

---

## 연습

### 연습 1: HTTP 메서드 매칭

| 행동 | HTTP 메서드 |
|------|-------------|
| 게시글 목록 보기 | |
| 새 게시글 작성 | |
| 게시글 수정 | |
| 게시글 삭제 | |

<details>
<summary>정답 보기</summary>

| 행동 | HTTP 메서드 |
|------|-------------|
| 게시글 목록 보기 | GET |
| 새 게시글 작성 | POST |
| 게시글 수정 | PUT |
| 게시글 삭제 | DELETE |

</details>

### 연습 2: 빈칸 채우기

```typescript
@Controller('posts')
export class PostsController {

  // 1. GET /posts
  @_______()
  findAll() { }

  // 2. GET /posts/1
  @Get(':id')
  findOne(@_______('id') id: string) { }

  // 3. POST /posts (본문에서 데이터 받기)
  @_______()
  create(@_______() body: any) { }

  // 4. DELETE /posts/1
  @_______(':id')
  remove(@Param('id') id: string) { }
}
```

<details>
<summary>정답 보기</summary>

1. `Get`
2. `Param`
3. `Post`, `Body`
4. `Delete`

</details>

### 연습 3: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | Controller는 요청을 받고 응답을 보내는 역할이다 | |
| 2 | GET 메서드는 새로운 데이터를 생성할 때 사용한다 | |
| 3 | `@Param('id')`는 URL 경로에서 id 값을 가져온다 | |
| 4 | `@Query()`는 POST 요청의 본문을 가져온다 | |
| 5 | REST API에서 리소스는 복수형으로 표현한다 | |

<details>
<summary>정답 보기</summary>

1. O
2. X (GET은 조회, POST가 생성)
3. O
4. X (`@Query()`는 URL ?뒤, 본문은 `@Body()`)
5. O

</details>

### 연습 4: 틀린 곳 찾기

```typescript
// 1번
@Controller('users')
export class UsersController {
  @Get(id)  // GET /users/:id
  findOne(@Param('id') id: string) { }
}

// 2번
@Post()
create(@Query() body: any) {  // POST 본문 받기
  return body;
}

// 3번
@Get(':id')
findOne(@Param() id: string) {  // id 파라미터 받기
  return `User ${id}`;
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 라우트 파라미터는 문자열로 ':id'
@Get(':id')
findOne(@Param('id') id: string) { }

// 2번: POST 본문은 @Body()로 받음
@Post()
create(@Body() body: any) {
  return body;
}

// 3번: @Param()에 파라미터 이름 필요
@Get(':id')
findOne(@Param('id') id: string) {
  return `User ${id}`;
}
```

</details>

### 연습 5: REST API URL 설계

다음 기능을 위한 REST API URL을 설계하세요.

| 기능 | HTTP 메서드 | URL |
|------|-------------|-----|
| 모든 상품 조회 | | |
| 특정 상품 조회 | | |
| 상품 등록 | | |
| 상품 수정 | | |
| 상품 삭제 | | |

<details>
<summary>정답 보기</summary>

| 기능 | HTTP 메서드 | URL |
|------|-------------|-----|
| 모든 상품 조회 | GET | /products |
| 특정 상품 조회 | GET | /products/:id |
| 상품 등록 | POST | /products |
| 상품 수정 | PUT | /products/:id |
| 상품 삭제 | DELETE | /products/:id |

</details>

---

## 핵심 정리

| 용어 | 한 줄 설명 |
|------|-----------|
| Controller | 요청을 받는 안내 데스크 |
| HTTP 메서드 | 어떤 행동을 원하는지 알려주는 신호 |
| GET | "가져와줘" (조회) |
| POST | "만들어줘" (생성) |
| PUT | "바꿔줘" (수정) |
| DELETE | "지워줘" (삭제) |
| REST API | URL 설계 규칙 |
| `@Param()` | URL 경로에서 데이터 받기 |
| `@Query()` | URL ?뒤에서 데이터 받기 |
| `@Body()` | 요청 본문에서 데이터 받기 |
