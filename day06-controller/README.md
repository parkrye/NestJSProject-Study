# Day 6: Controller와 REST API 기초

## 학습 목표
- HTTP 메서드의 역할 이해
- REST API 개념 파악
- NestJS Controller 사용법 습득

---

## C# 개발자를 위한 핵심 차이점

> **NestJS Controller = ASP.NET Core Controller**
> 거의 동일한 구조와 개념입니다!

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 컨트롤러 | `[ApiController]` | `@Controller()` |
| GET | `[HttpGet]` | `@Get()` |
| POST | `[HttpPost]` | `@Post()` |
| PUT | `[HttpPut]` | `@Put()` |
| DELETE | `[HttpDelete]` | `@Delete()` |
| 라우트 파라미터 | `[FromRoute]` 또는 `{id}` | `@Param()` |
| 쿼리 파라미터 | `[FromQuery]` | `@Query()` |
| 요청 본문 | `[FromBody]` | `@Body()` |
| 상태 코드 | `return Ok()` | `@HttpCode()` |

```csharp
// ASP.NET Core
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase {
    [HttpGet]
    public IActionResult GetAll() => Ok(_service.GetAll());

    [HttpGet("{id}")]
    public IActionResult GetById(int id) => Ok(_service.GetById(id));

    [HttpPost]
    public IActionResult Create([FromBody] CreateUserDto dto) => Created(...);
}
```

```typescript
// NestJS - 거의 동일!
@Controller('users')
export class UsersController {
  @Get()
  getAll() { return this.service.getAll(); }

  @Get(':id')
  getById(@Param('id') id: string) { return this.service.getById(+id); }

  @Post()
  create(@Body() dto: CreateUserDto) { return this.service.create(dto); }
}
```

---

## 공부

### 1. HTTP 메서드 (C#과 동일)

| 메서드 | 용도 | ASP.NET Core | NestJS |
|--------|------|-------------|--------|
| GET | 조회 | `[HttpGet]` | `@Get()` |
| POST | 생성 | `[HttpPost]` | `@Post()` |
| PUT | 전체 수정 | `[HttpPut]` | `@Put()` |
| PATCH | 일부 수정 | `[HttpPatch]` | `@Patch()` |
| DELETE | 삭제 | `[HttpDelete]` | `@Delete()` |

### 2. REST API 설계 (C#과 동일한 개념)

```
GET    /users      → 모든 사용자 조회
GET    /users/1    → ID가 1인 사용자 조회
POST   /users      → 새 사용자 생성
PUT    /users/1    → ID가 1인 사용자 수정
DELETE /users/1    → ID가 1인 사용자 삭제
```

### 3. Controller 기본 - ASP.NET Core와 비교

```csharp
// ASP.NET Core
[ApiController]
[Route("users")]
public class UsersController : ControllerBase {
    [HttpGet]
    public IActionResult FindAll() => Ok("모든 사용자");

    [HttpGet("{id}")]
    public IActionResult FindOne([FromRoute] int id) => Ok($"ID {id}");

    [HttpPost]
    public IActionResult Create([FromBody] object body) => Ok(body);

    [HttpPut("{id}")]
    public IActionResult Update([FromRoute] int id, [FromBody] object body) => Ok();

    [HttpDelete("{id}")]
    public IActionResult Remove([FromRoute] int id) => Ok();
}
```

```typescript
// NestJS - 데코레이터만 다르고 구조 동일!
@Controller('users')
export class UsersController {

  @Get()  // [HttpGet]
  findAll() {
    return '모든 사용자 조회';
  }

  @Get(':id')  // [HttpGet("{id}")]
  findOne(@Param('id') id: string) {  // [FromRoute] int id
    return `ID ${id} 사용자 조회`;
  }

  @Post()  // [HttpPost]
  create(@Body() body: any) {  // [FromBody] object body
    return `사용자 생성: ${JSON.stringify(body)}`;
  }

  @Put(':id')  // [HttpPut("{id}")]
  update(@Param('id') id: string, @Body() body: any) {
    return `ID ${id} 사용자 수정`;
  }

  @Delete(':id')  // [HttpDelete("{id}")]
  remove(@Param('id') id: string) {
    return `ID ${id} 사용자 삭제`;
  }
}
```

### 4. 요청 데이터 받기 - ASP.NET Core 대응

**@Param() = [FromRoute]**
```csharp
// ASP.NET Core
[HttpGet("{id}")]
public IActionResult GetById([FromRoute] int id) { }

[HttpGet("{userId}/posts/{postId}")]
public IActionResult GetPost(int userId, int postId) { }
```

```typescript
// NestJS
@Get(':id')
findOne(@Param('id') id: string) { }

@Get(':userId/posts/:postId')
findPost(@Param('userId') userId: string, @Param('postId') postId: string) { }
```

**@Query() = [FromQuery]**
```csharp
// ASP.NET Core
[HttpGet]
public IActionResult GetAll([FromQuery] int page, [FromQuery] int limit) { }
```

```typescript
// NestJS
@Get()
findAll(@Query('page') page: string, @Query('limit') limit: string) { }

// 전체 쿼리 객체로
@Get()
findAll(@Query() query: any) {
  console.log(query);  // { page: '1', limit: '10' }
}
```

**@Body() = [FromBody]**
```csharp
// ASP.NET Core
[HttpPost]
public IActionResult Create([FromBody] CreateUserDto dto) { }
```

```typescript
// NestJS
@Post()
create(@Body() body: CreateUserDto) { }

// 특정 필드만
@Post()
create(@Body('name') name: string) { }
```

### 5. 응답 처리 - ASP.NET Core와 비교

```csharp
// ASP.NET Core
[HttpGet]
public IActionResult GetAll() => Ok(new { users = new List<User>() });

[HttpPost]
[ProducesResponseType(StatusCodes.Status201Created)]
public IActionResult Create() => StatusCode(201, new { message = "Created" });

[HttpGet("{id}")]
public IActionResult GetById(int id) {
    var user = _service.Find(id);
    if (user == null) return NotFound("사용자를 찾을 수 없습니다");
    return Ok(user);
}
```

```typescript
// NestJS
@Get()
findAll() {
  return { users: [] };  // 자동 JSON 변환 (Ok() 불필요)
}

@Post()
@HttpCode(201)  // [ProducesResponseType(201)]
create() {
  return { message: 'Created' };
}

@Get(':id')
findOne(@Param('id') id: string) {
  const user = this.service.find(+id);
  if (!user) {
    throw new NotFoundException('사용자를 찾을 수 없습니다');
    // ASP.NET Core의 return NotFound()와 같음
  }
  return user;
}
```

**예외 클래스 - C# 대응:**
| NestJS | ASP.NET Core | HTTP 코드 |
|--------|-------------|----------|
| `NotFoundException` | `NotFound()` | 404 |
| `BadRequestException` | `BadRequest()` | 400 |
| `UnauthorizedException` | `Unauthorized()` | 401 |
| `ForbiddenException` | `Forbid()` | 403 |

### 6. CLI로 Controller 생성

```bash
# NestJS
nest g controller users

# 생성 결과 (ASP.NET Core는 수동)
# src/users/users.controller.ts
# src/users/users.controller.spec.ts
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. 컨트롤러 데코레이터 (C#: [Route("users")])
@_______('users')
export class UsersController {

  // 2. GET 요청 (C#: [HttpGet])
  @_______()
  findAll() { }

  // 3. URL 파라미터 (C#: [FromRoute] int id)
  @Get(':id')
  findOne(@_______('id') id: string) { }

  // 4. 요청 본문 (C#: [FromBody] CreateUserDto dto)
  @Post()
  create(@_______() body: CreateUserDto) { }

  // 5. 쿼리 파라미터 (C#: [FromQuery] int page)
  @Get()
  findAll(@_______('page') page: string) { }
}
```

<details>
<summary>정답 보기</summary>

1. `Controller`
2. `Get`
3. `Param`
4. `Body`
5. `Query`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | `@Get(':id')`는 C#의 `[HttpGet("{id}")]`와 동일하다 | |
| 2 | NestJS에서 POST 요청의 본문은 `@Query()`로 받는다 | |
| 3 | `@Param('id')`는 URL 경로의 파라미터를 받는다 | |
| 4 | NestJS Controller는 자동으로 JSON을 반환한다 | |
| 5 | `NotFoundException`을 throw하면 404 응답이 반환된다 | |

<details>
<summary>정답 보기</summary>

1. O
2. X (POST 본문은 `@Body()`로 받음)
3. O
4. O
5. O

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번
@Controller('users')
export class UsersController {
  @Get(id)
  findOne(@Param('id') id: string) { }
}

// 2번
@Post()
create(@Query() body: CreateUserDto) {
  return body;
}

// 3번
@Get(':id')
findOne(@Param() id: string) {
  return `User ${id}`;
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 라우트 파라미터는 문자열로 ':id' 형식
@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string) { }
}

// 2번: POST 본문은 @Query가 아닌 @Body
@Post()
create(@Body() body: CreateUserDto) {
  return body;
}

// 3번: @Param()에 파라미터 이름 필요
@Get(':id')
findOne(@Param('id') id: string) {
  return `User ${id}`;
}
```

</details>

### 연습 4: ASP.NET Core → NestJS 변환

| ASP.NET Core | NestJS |
|--------------|--------|
| `[HttpGet]` | `@_______()` |
| `[HttpPost]` | `@_______()` |
| `[HttpPut("{id}")]` | `@_______(':id')` |
| `[HttpDelete("{id}")]` | `@_______(':id')` |
| `[FromRoute] int id` | `@_______('id') id: string` |
| `[FromBody] UserDto dto` | `@_______() dto: UserDto` |
| `[FromQuery] int page` | `@_______('page') page: string` |

<details>
<summary>정답 보기</summary>

| ASP.NET Core | NestJS |
|--------------|--------|
| `[HttpGet]` | `@Get()` |
| `[HttpPost]` | `@Post()` |
| `[HttpPut("{id}")]` | `@Put(':id')` |
| `[HttpDelete("{id}")]` | `@Delete(':id')` |
| `[FromRoute] int id` | `@Param('id') id: string` |
| `[FromBody] UserDto dto` | `@Body() dto: UserDto` |
| `[FromQuery] int page` | `@Query('page') page: string` |

</details>

---

## 숙제

### 숙제 1: HTTP 메서드 정리

**문제**: ASP.NET Core와 NestJS의 HTTP 메서드 데코레이터를 비교하세요.

| 용도 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 조회 | `[HttpGet]` | |
| 생성 | `[HttpPost]` | |
| 전체 수정 | `[HttpPut]` | |
| 일부 수정 | `[HttpPatch]` | |
| 삭제 | `[HttpDelete]` | |

### 숙제 2: Posts 컨트롤러 만들기

**문제**: Posts 리소스를 위한 CRUD 컨트롤러를 작성하세요.

**요구사항**:
- 라우트: `/posts`
- 메모리 배열로 데이터 저장
- 5개의 기본 CRUD 엔드포인트 구현

```typescript
@Controller('posts')
export class PostsController {
  private posts = [
    { id: 1, title: '첫 번째 글', content: '내용1' },
  ];

  // GET /posts - 전체 조회


  // GET /posts/:id - 단일 조회


  // POST /posts - 생성


  // PUT /posts/:id - 수정


  // DELETE /posts/:id - 삭제

}
```

### 숙제 3: 쿼리 파라미터 검색 기능

**문제**: 검색과 페이징 기능을 추가하세요.

**요구사항**:
- `GET /posts?search=검색어` - 제목에 검색어 포함된 글 필터링
- `GET /posts?page=1&limit=10` - 페이징 처리

**힌트**: `@Query()` 데코레이터와 `filter()` 메서드 사용

```typescript
@Get()
findAll(
  @Query('search') search?: string,
  @Query('page') page?: string,
  @Query('limit') limit?: string,
) {
  // 여기에 코드 작성
}
```

---

## 핵심 정리

| NestJS | ASP.NET Core 대응 |
|--------|------------------|
| `@Controller('path')` | `[Route("path")]` |
| `@Get()` | `[HttpGet]` |
| `@Post()` | `[HttpPost]` |
| `@Put()` | `[HttpPut]` |
| `@Delete()` | `[HttpDelete]` |
| `@Param('id')` | `[FromRoute]` / `{id}` |
| `@Query('key')` | `[FromQuery]` |
| `@Body()` | `[FromBody]` |
| `throw new NotFoundException()` | `return NotFound()` |
