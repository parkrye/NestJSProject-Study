# Day 7: Service와 의존성 주입

## 학습 목표
- Service가 무엇이고 왜 필요한지 이해
- Controller와 Service를 분리하는 이유 파악
- 의존성 주입(DI)이 무엇인지 이해

---

## 1. Service란?

### 한 줄 정리
> **실제 일을 처리하는 작업 부서**

### 왜 필요해?

Controller가 모든 일을 직접 하면 코드가 너무 길어지고 복잡해집니다.
**역할을 나누면** 코드가 깔끔해지고 관리하기 쉬워집니다.

```
[역할 분담]

Controller (안내 데스크)     Service (작업 부서)
- 요청 받기                  - 실제 계산
- 응답 보내기                - 데이터 처리
                            - 비즈니스 로직
```

| 비유 | Controller | Service |
|------|------------|---------|
| 식당 | 직원 (주문 받음) | 주방 (요리함) |
| 병원 | 접수 창구 | 의사 (진료함) |
| 회사 | 안내 데스크 | 담당 부서 (업무 처리) |

### 분리하는 이유

| 이유 | 설명 |
|------|------|
| **역할 분리** | 각자 맡은 일만 하면 됨 |
| **재사용** | 같은 기능을 여러 곳에서 쓸 수 있음 |
| **테스트** | 각 부분을 따로 테스트 가능 |
| **유지보수** | 수정할 때 한 곳만 고치면 됨 |

---

## 2. Service 만들기

### 기본 구조

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()  // "이건 서비스야" 표시
export class UsersService {
  private users = [];  // 임시 데이터 저장소

  // 전체 조회
  findAll() {
    return this.users;
  }

  // 하나 조회
  findOne(id: number) {
    return this.users.find(user => user.id === id);
  }

  // 생성
  create(data: any) {
    const newUser = {
      id: this.users.length + 1,
      ...data,
    };
    this.users.push(newUser);
    return newUser;
  }

  // 수정
  update(id: number, data: any) {
    const user = this.findOne(id);
    if (user) {
      Object.assign(user, data);
    }
    return user;
  }

  // 삭제
  remove(id: number) {
    const index = this.users.findIndex(user => user.id === id);
    if (index > -1) {
      return this.users.splice(index, 1)[0];
    }
    return null;
  }
}
```

### 코드 설명

| 코드 | 의미 |
|------|------|
| `@Injectable()` | "이 클래스는 서비스야" (주입 가능 표시) |
| `findAll()` | 전체 목록 조회 |
| `findOne(id)` | 특정 항목 조회 |
| `create(data)` | 새 항목 생성 |
| `update(id, data)` | 기존 항목 수정 |
| `remove(id)` | 항목 삭제 |

---

## 3. 의존성 주입 (DI)

### 한 줄 정리
> **필요한 도구를 알아서 가져다 주는 것**

### 왜 필요해?

Controller가 Service를 사용하려면 Service 객체가 필요합니다.
직접 만들 수도 있지만, **NestJS가 대신 만들어서 넣어주는 게** 더 좋습니다.

```typescript
// ❌ 직접 만들기 (나쁜 방법)
class UsersController {
  private usersService = new UsersService();  // 직접 생성
}

// ✅ 주입받기 (좋은 방법)
class UsersController {
  constructor(private usersService: UsersService) {}
  // NestJS가 알아서 UsersService를 만들어서 넣어줌!
}
```

| 비유 | 직접 만들기 | 주입받기 |
|------|------------|---------|
| 식당 주방 | 요리사가 직접 칼 사러 감 | 식당이 칼을 준비해둠 |
| 회사 | 직원이 노트북 직접 삼 | 회사가 노트북 지급 |
| 카페 | 바리스타가 커피머신 직접 삼 | 카페가 커피머신 제공 |

### 의존성 주입의 장점

| 장점 | 설명 |
|------|------|
| **편리함** | 필요한 것을 알아서 가져다 줌 |
| **유연함** | 나중에 Service를 바꿔도 Controller 수정 불필요 |
| **테스트** | 가짜 Service를 넣어서 테스트 가능 |

---

## 4. Controller에서 Service 사용하기

### 연결 방법

```typescript
@Controller('users')
export class UsersController {
  // 생성자에서 Service 주입받기
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();  // Service에게 일 시킴
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);  // +id: 문자→숫자 변환
  }

  @Post()
  create(@Body() body: any) {
    return this.usersService.create(body);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() body: any) {
    return this.usersService.update(+id, body);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(+id);
  }
}
```

### 코드 설명

```typescript
constructor(private readonly usersService: UsersService) {}
```

| 부분 | 의미 |
|------|------|
| `constructor()` | 생성자 (객체가 만들어질 때 실행) |
| `private` | 이 클래스 안에서만 사용 |
| `readonly` | 변경 불가 (안전장치) |
| `usersService: UsersService` | UsersService 타입의 변수 |

이 한 줄로:
1. NestJS가 UsersService 객체를 만들고
2. usersService 변수에 넣어줌
3. `this.usersService`로 사용 가능

---

## 5. Module에 등록하기

### 왜 등록해야 해?

NestJS에게 "이 Service가 있어"라고 알려줘야 주입이 가능합니다.

```typescript
@Module({
  controllers: [UsersController],
  providers: [UsersService],  // 여기에 Service 등록!
})
export class UsersModule {}
```

| 옵션 | 의미 |
|------|------|
| `controllers` | 컨트롤러 목록 |
| `providers` | 서비스 목록 (주입 가능한 것들) |

### 전체 흐름

```
1. Module에 Service 등록
      ↓
2. NestJS가 Service 객체 생성
      ↓
3. Controller 생성 시 Service 주입
      ↓
4. Controller에서 Service 사용
```

---

## 6. 실제 CRUD 예시

### 완성된 Service

```typescript
@Injectable()
export class PostsService {
  private posts = [
    { id: 1, title: '첫 번째 글', content: '내용1' },
    { id: 2, title: '두 번째 글', content: '내용2' },
  ];

  findAll() {
    return this.posts;
  }

  findOne(id: number) {
    return this.posts.find(post => post.id === id);
  }

  create(data: { title: string; content: string }) {
    const newPost = {
      id: this.posts.length + 1,
      ...data,
    };
    this.posts.push(newPost);
    return newPost;
  }

  update(id: number, data: { title?: string; content?: string }) {
    const post = this.findOne(id);
    if (post) {
      Object.assign(post, data);
    }
    return post;
  }

  remove(id: number) {
    const index = this.posts.findIndex(post => post.id === id);
    if (index > -1) {
      return this.posts.splice(index, 1)[0];
    }
    return null;
  }
}
```

### 완성된 Controller

```typescript
@Controller('posts')
export class PostsController {
  constructor(private readonly postsService: PostsService) {}

  @Get()
  findAll() {
    return this.postsService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    const post = this.postsService.findOne(+id);
    if (!post) {
      throw new NotFoundException('게시글을 찾을 수 없습니다');
    }
    return post;
  }

  @Post()
  create(@Body() body: { title: string; content: string }) {
    return this.postsService.create(body);
  }

  @Put(':id')
  update(
    @Param('id') id: string,
    @Body() body: { title?: string; content?: string },
  ) {
    return this.postsService.update(+id, body);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.postsService.remove(+id);
  }
}
```

---

## 7. CLI로 Service 생성

```bash
# Service만 생성
nest g service users

# 결과:
# src/users/users.service.ts
# src/users/users.service.spec.ts (테스트 파일)

# Controller와 Service 한번에 생성
nest g resource users
```

---

## C# 개발자를 위한 비교

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 서비스 정의 | `public class UserService` | `@Injectable() class UserService` |
| DI 등록 | `services.AddScoped<Service>()` | `providers: [Service]` |
| 생성자 주입 | `public Controller(IService svc)` | `constructor(private svc: Service)` |
| 인터페이스 | 보통 사용 (IService) | 선택적 (없어도 됨) |

```csharp
// ASP.NET Core
public class UsersController : ControllerBase {
    private readonly IUserService _userService;

    public UsersController(IUserService userService) {
        _userService = userService;
    }
}
```

```typescript
// NestJS - 거의 동일!
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}
}
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. 서비스 데코레이터
@_______()
export class PostsService {
  findAll() { return []; }
}

// 2. 모듈에 서비스 등록
@Module({
  controllers: [PostsController],
  _______: [PostsService],  // 서비스 등록
})
export class PostsModule {}

// 3. 컨트롤러에서 서비스 주입
@Controller('posts')
export class PostsController {
  _______( private readonly postsService: PostsService ) {}

  @Get()
  findAll() {
    return _______.postsService.findAll();
  }
}
```

<details>
<summary>정답 보기</summary>

1. `Injectable`
2. `providers`
3. `constructor`
4. `this`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | Service는 실제 비즈니스 로직을 처리하는 곳이다 | |
| 2 | `@Injectable()` 데코레이터는 Controller에 붙인다 | |
| 3 | Service를 사용하려면 Module의 providers에 등록해야 한다 | |
| 4 | Controller에서 Service를 사용할 때 new로 직접 생성한다 | |
| 5 | 의존성 주입은 필요한 객체를 알아서 넣어주는 것이다 | |

<details>
<summary>정답 보기</summary>

1. O
2. X (`@Injectable()`은 Service에, Controller는 `@Controller()`)
3. O
4. X (new 대신 생성자 주입으로 받음)
5. O

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번
@Injectable
export class UserService {
  findAll() { return []; }
}

// 2번
@Controller('users')
export class UsersController {
  constructor(userService: UserService) {}

  @Get()
  findAll() {
    return userService.findAll();
  }
}

// 3번
@Module({
  controllers: [UsersController],
  services: [UserService],
})
export class UsersModule {}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 괄호 필요
@Injectable()
export class UserService {
  findAll() { return []; }
}

// 2번: private 필요, this 필요
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findAll() {
    return this.userService.findAll();  // this 필요
  }
}

// 3번: services가 아니라 providers
@Module({
  controllers: [UsersController],
  providers: [UserService],  // providers!
})
export class UsersModule {}
```

</details>

### 연습 4: Controller와 Service 분리 이유

Controller와 Service를 분리하는 이유 3가지를 작성하세요.

```
1. _______________
2. _______________
3. _______________
```

<details>
<summary>정답 보기</summary>

1. **역할 분리** - Controller는 요청/응답, Service는 비즈니스 로직
2. **재사용성** - 같은 Service를 여러 Controller에서 사용 가능
3. **테스트 용이** - 각 부분을 독립적으로 테스트 가능

(유지보수, 코드 정리 등도 정답)

</details>

---

## 요청 흐름 정리

```
[사용자]
   │
   │ GET /users
   ↓
[Controller] ─────────────────────┐
   │  "요청 접수!"                 │
   │                              │
   │  this.usersService.findAll() │
   ↓                              │
[Service] ←───────────────────────┘
   │  "내가 처리할게"
   │
   │  데이터 조회/가공
   ↓
[Controller]
   │  "결과 받았어"
   │
   │  return 결과
   ↓
[사용자]
   "응답 받음"
```

---

## 핵심 정리

| 용어 | 한 줄 설명 |
|------|-----------|
| Service | 실제 일을 처리하는 작업 부서 |
| `@Injectable()` | "이건 주입 가능한 서비스야" 표시 |
| 의존성 주입 (DI) | 필요한 객체를 알아서 가져다 주는 것 |
| `providers` | Module에서 Service를 등록하는 곳 |
| 생성자 주입 | `constructor(private svc: Service)` |
| `this.service` | 주입받은 Service 사용 |
