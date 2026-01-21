# Day 7: Service와 의존성 주입

## 학습 목표
- Service의 역할 이해
- 의존성 주입(DI) 개념 파악
- Controller와 Service 분리 이유 습득

---

## C# 개발자를 위한 핵심 차이점

> **NestJS DI = ASP.NET Core DI**
> 개념과 사용법이 거의 동일합니다!

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 서비스 정의 | `public class UserService` | `@Injectable() class UserService` |
| DI 등록 | `services.AddScoped<IService, Service>()` | `providers: [Service]` |
| 생성자 주입 | `public Controller(IService service)` | `constructor(private service: Service)` |
| 인터페이스 | `IUserService` | 선택적 (없어도 됨) |

```csharp
// ASP.NET Core
public interface IUserService {
    List<User> GetAll();
}

public class UserService : IUserService {
    public List<User> GetAll() => new List<User>();
}

// Startup.cs
services.AddScoped<IUserService, UserService>();

// Controller
public class UsersController : ControllerBase {
    private readonly IUserService _userService;
    public UsersController(IUserService userService) {
        _userService = userService;
    }
}
```

```typescript
// NestJS - 더 간단! (인터페이스 선택적)
@Injectable()
export class UserService {
  getAll(): User[] { return []; }
}

// Module
@Module({ providers: [UserService] })

// Controller
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}
}
```

---

## 공부

### 1. Service란?

> **ASP.NET Core의 Service 계층과 동일**

```
[요청 흐름 - C#과 동일]

Client → Controller → Service → Repository/DB
            ↓           ↓
       요청/응답     비즈니스 로직
```

**C#처럼 분리하는 이유:**
- Controller: 요청 받고 응답 보내기만 (Thin Controller)
- Service: 실제 비즈니스 로직
- 테스트 용이, 재사용성

### 2. Service 기본 구조 - ASP.NET Core와 비교

```csharp
// ASP.NET Core
public interface IUserService {
    List<User> FindAll();
    User? FindOne(int id);
    User Create(CreateUserDto dto);
    User Update(int id, UpdateUserDto dto);
    void Remove(int id);
}

public class UserService : IUserService {
    private readonly List<User> _users = new();

    public List<User> FindAll() => _users;

    public User? FindOne(int id) => _users.FirstOrDefault(u => u.Id == id);

    public User Create(CreateUserDto dto) {
        var user = new User { Id = _users.Count + 1, Name = dto.Name };
        _users.Add(user);
        return user;
    }
    // ...
}
```

```typescript
// NestJS - @Injectable()만 추가하면 됨!
@Injectable()  // C#에서는 자동, NestJS에서는 명시
export class UserService {
  private users: User[] = [];

  findAll(): User[] {
    return this.users;
  }

  findOne(id: number): User | undefined {
    return this.users.find(u => u.id === id);
  }

  create(dto: CreateUserDto): User {
    const user = { id: this.users.length + 1, ...dto };
    this.users.push(user);
    return user;
  }

  update(id: number, dto: UpdateUserDto): User | undefined {
    const user = this.findOne(id);
    if (user) Object.assign(user, dto);
    return user;
  }

  remove(id: number): User | undefined {
    const index = this.users.findIndex(u => u.id === id);
    if (index > -1) return this.users.splice(index, 1)[0];
    return undefined;
  }
}
```

### 3. 의존성 주입 - C#과 동일한 개념!

> **C# 개발자에게 익숙한 생성자 주입!**

```csharp
// ASP.NET Core - 생성자 주입
public class UsersController : ControllerBase {
    private readonly IUserService _userService;

    public UsersController(IUserService userService) {
        _userService = userService;  // DI 컨테이너가 주입
    }
}
```

```typescript
// NestJS - 동일한 패턴!
@Controller('users')
export class UsersController {
  // TypeScript 축약 문법 (C# primary constructor와 비슷)
  constructor(private readonly userService: UserService) {}
  // 자동으로 this.userService = userService; 처리됨
}
```

**의존성 주입의 장점 (C#과 동일):**
1. 테스트 시 Mock 주입 가능
2. 느슨한 결합 (Loose Coupling)
3. 단일 책임 원칙

### 4. Controller에서 Service 사용

```csharp
// ASP.NET Core
[ApiController]
[Route("users")]
public class UsersController : ControllerBase {
    private readonly IUserService _userService;

    public UsersController(IUserService userService) {
        _userService = userService;
    }

    [HttpGet]
    public IActionResult FindAll() => Ok(_userService.FindAll());

    [HttpGet("{id}")]
    public IActionResult FindOne(int id) => Ok(_userService.FindOne(id));

    [HttpPost]
    public IActionResult Create([FromBody] CreateUserDto dto)
        => Ok(_userService.Create(dto));
}
```

```typescript
// NestJS - 거의 동일!
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findAll() {
    return this.userService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(+id);  // +로 숫자 변환
  }

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.userService.create(dto);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    return this.userService.update(+id, dto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.userService.remove(+id);
  }
}
```

### 5. Module에 등록 - ASP.NET Core ConfigureServices와 비교

```csharp
// ASP.NET Core - Startup.cs / Program.cs
public void ConfigureServices(IServiceCollection services) {
    services.AddControllers();
    services.AddScoped<IUserService, UserService>();  // DI 등록
}
```

```typescript
// NestJS - Module의 providers
@Module({
  controllers: [UsersController],
  providers: [UserService],  // DI 등록 (services.AddScoped과 같음)
})
export class UsersModule {}
```

**서비스 수명 비교:**
| ASP.NET Core | NestJS | 설명 |
|--------------|--------|------|
| `AddScoped` | 기본값 | 요청당 하나 |
| `AddSingleton` | `{ scope: Scope.DEFAULT }` | 앱당 하나 |
| `AddTransient` | `{ scope: Scope.TRANSIENT }` | 매번 새로 |

### 6. CLI로 Service 생성

```bash
# NestJS - 자동 생성!
nest g service users

# 생성 결과
# src/users/users.service.ts
# src/users/users.service.spec.ts

# Controller와 Service 한번에
nest g resource users
```

---

## C# 인터페이스 vs NestJS

```csharp
// C# - 인터페이스 권장 (테스트를 위해)
public interface IUserService { }
public class UserService : IUserService { }
services.AddScoped<IUserService, UserService>();
```

```typescript
// NestJS - 인터페이스 없어도 됨 (TypeScript 특성상)
@Injectable()
export class UserService { }

// 테스트 시 그냥 클래스를 Mock 가능
// 필요하면 인터페이스 추가 가능하지만 보통 생략
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. 서비스 데코레이터 (C#에서 DI 등록과 같음)
@_______()
export class UserService {
  findAll() { return []; }
}

// 2. 모듈에서 서비스 등록 (C#: services.AddScoped<UserService>())
@Module({
  controllers: [UsersController],
  _______: [UserService],
})
export class UsersModule {}

// 3. 컨트롤러에서 서비스 주입 (C#: 생성자 주입)
@Controller('users')
export class UsersController {
  _______( private readonly userService: UserService ) {}

  @Get()
  findAll() {
    return _______.userService.findAll();
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
| 1 | `@Injectable()` 데코레이터는 서비스를 DI 컨테이너에 등록하기 위해 필요하다 | |
| 2 | NestJS의 DI는 ASP.NET Core의 DI와 다른 패턴을 사용한다 | |
| 3 | 서비스를 사용하려면 Module의 `providers` 배열에 등록해야 한다 | |
| 4 | Controller에서 Service를 사용할 때 `new Service()`로 직접 생성한다 | |
| 5 | `nest g service users` 명령어로 서비스를 자동 생성할 수 있다 | |

<details>
<summary>정답 보기</summary>

1. O
2. X (ASP.NET Core와 거의 동일한 생성자 주입 패턴)
3. O
4. X (생성자 주입으로 DI 컨테이너가 자동으로 주입)
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
  constructor(userService: UserService) {}  // 주입

  @Get()
  findAll() {
    return userService.findAll();  // 사용
  }
}

// 3번
@Module({
  controllers: [UsersController],
  services: [UserService],  // 서비스 등록
})
export class UsersModule {}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 데코레이터는 함수이므로 괄호 필요
@Injectable()
export class UserService {
  findAll() { return []; }
}

// 2번: private readonly 필요, this 필요
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findAll() {
    return this.userService.findAll();
  }
}

// 3번: services가 아니라 providers
@Module({
  controllers: [UsersController],
  providers: [UserService],
})
export class UsersModule {}
```

</details>

### 연습 4: ASP.NET Core → NestJS 변환

| ASP.NET Core | NestJS |
|--------------|--------|
| `public class UserService` | `@_______() export class UserService` |
| `services.AddScoped<UserService>()` | `providers: [_______]` |
| `public Controller(IService svc)` | `constructor(private _______: Service)` |
| `_userService.FindAll()` | `this._______.findAll()` |

<details>
<summary>정답 보기</summary>

| ASP.NET Core | NestJS |
|--------------|--------|
| `public class UserService` | `@Injectable() export class UserService` |
| `services.AddScoped<UserService>()` | `providers: [UserService]` |
| `public Controller(IService svc)` | `constructor(private readonly svc: Service)` |
| `_userService.FindAll()` | `this.userService.findAll()` |

</details>

---

## 숙제

### 숙제 1: Controller와 Service 분리 이유

**문제**: Controller와 Service를 분리하는 이유 3가지를 작성하세요.
(C#에서도 동일한 이유입니다)

```
1.
2.
3.
```

### 숙제 2: Posts Service 구현

**문제**: Posts 기능을 위한 Service를 구현하세요.

**요구사항**:
- `findAll()`: 모든 게시글 조회
- `findOne(id)`: 특정 게시글 조회
- `create(dto)`: 게시글 생성
- `update(id, dto)`: 게시글 수정
- `remove(id)`: 게시글 삭제

**힌트**: C#의 Service 패턴과 동일

```typescript
@Injectable()
export class PostsService {
  private posts = [
    { id: 1, title: '첫 번째 글', content: '내용1' },
  ];

  // 여기에 메서드 구현


}
```

### 숙제 3: Controller에서 Service 사용

**문제**: 위에서 만든 PostsService를 사용하는 Controller를 구현하세요.

**요구사항**:
- `GET /posts`: 전체 조회
- `GET /posts/:id`: 단일 조회
- `POST /posts`: 생성
- `PUT /posts/:id`: 수정
- `DELETE /posts/:id`: 삭제

```typescript
@Controller('posts')
export class PostsController {
  // 생성자 주입


  // CRUD 메서드 구현


}
```

---

## 핵심 정리

| NestJS | ASP.NET Core 대응 |
|--------|------------------|
| `@Injectable()` | 클래스 + DI 등록 |
| `providers: [Service]` | `services.AddScoped<Service>()` |
| `constructor(private svc: Service)` | 생성자 주입 (동일!) |
| Module | `ConfigureServices` |
| `nest g service` | 수동 생성 |
