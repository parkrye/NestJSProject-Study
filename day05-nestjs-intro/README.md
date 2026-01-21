# Day 5: NestJS 소개와 프로젝트 구조

## 학습 목표
- NestJS가 무엇인지 이해
- NestJS 프로젝트 생성 방법 습득
- 기본 파일 구조 파악

---

## C# 개발자를 위한 핵심 차이점

> **NestJS는 ASP.NET Core의 JavaScript 버전**이라고 생각하면 됩니다!
> 구조, 개념, 패턴이 매우 유사합니다.

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 프레임워크 타입 | 백엔드 프레임워크 | 백엔드 프레임워크 |
| 언어 | C# | TypeScript |
| DI 컨테이너 | 내장 DI | 내장 DI |
| 컨트롤러 | `[ApiController]` | `@Controller()` |
| 서비스 | DI 등록 | `@Injectable()` |
| 미들웨어 | Middleware | Middleware |
| 필터 | Exception Filter | Exception Filter |
| 데코레이터 | Attribute `[...]` | Decorator `@...` |

```csharp
// ASP.NET Core
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase {
    private readonly IUserService _userService;

    public UsersController(IUserService userService) {
        _userService = userService;
    }

    [HttpGet]
    public IActionResult GetAll() => Ok(_userService.GetAll());
}
```

```typescript
// NestJS - 거의 동일한 구조!
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Get()
  getAll() {
    return this.userService.getAll();
  }
}
```

---

## 공부

### 1. NestJS란?

> **ASP.NET Core처럼 구조화된 Node.js 프레임워크**

```
Express (자유로움) vs NestJS (구조적)
   ↓                    ↓
Flask/FastAPI       ASP.NET Core
(Python)            (C#)
```

**NestJS의 장점** (ASP.NET Core와 공통점):
- 구조화된 아키텍처 (Controller-Service 패턴)
- TypeScript 기본 지원 (C#처럼 타입 안전)
- 의존성 주입 (DI) 내장
- 데코레이터 기반 (`@` = C#의 `[Attribute]`)
- 테스트하기 쉬움

### 2. 프로젝트 생성

```bash
# NestJS CLI 전역 설치 (dotnet CLI처럼)
npm install -g @nestjs/cli

# 새 프로젝트 생성 (dotnet new webapi와 비슷)
nest new my-project

# 개발 서버 실행 (dotnet run과 비슷)
cd my-project
npm run start:dev
```

### 3. 프로젝트 구조 - ASP.NET Core와 비교

**ASP.NET Core:**
```
MyProject/
├── Program.cs              # 앱 시작점
├── Startup.cs              # 서비스/미들웨어 설정
├── Controllers/
│   └── UsersController.cs
├── Services/
│   └── UserService.cs
├── MyProject.csproj        # 프로젝트 설정
└── appsettings.json        # 환경 설정
```

**NestJS:**
```
my-project/
├── src/
│   ├── main.ts             # 앱 시작점 (Program.cs)
│   ├── app.module.ts       # 루트 모듈 (Startup.cs의 일부)
│   ├── app.controller.ts   # 컨트롤러
│   └── app.service.ts      # 서비스
├── package.json            # 프로젝트 설정 (.csproj)
└── tsconfig.json           # TypeScript 설정
```

### 4. 핵심 파일 설명 - ASP.NET Core 대응

**main.ts = Program.cs**
```csharp
// ASP.NET Core - Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

```typescript
// NestJS - main.ts (같은 역할!)
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

**app.module.ts = Startup.cs의 ConfigureServices**
```csharp
// ASP.NET Core
public void ConfigureServices(IServiceCollection services) {
    services.AddControllers();
    services.AddScoped<IUserService, UserService>();
}
```

```typescript
// NestJS - app.module.ts
@Module({
  imports: [],                    // 다른 모듈 가져오기
  controllers: [AppController],   // 컨트롤러 등록
  providers: [AppService],        // 서비스 등록 (DI)
})
export class AppModule {}
```

**app.controller.ts = Controller**
```csharp
// ASP.NET Core
[ApiController]
[Route("[controller]")]
public class AppController : ControllerBase {
    private readonly IAppService _appService;

    public AppController(IAppService appService) {
        _appService = appService;
    }

    [HttpGet]
    public string Get() => _appService.GetHello();
}
```

```typescript
// NestJS - 거의 동일!
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

**app.service.ts = Service**
```csharp
// ASP.NET Core
public interface IAppService {
    string GetHello();
}

public class AppService : IAppService {
    public string GetHello() => "Hello World!";
}
```

```typescript
// NestJS
@Injectable()  // C#의 서비스 등록과 같은 의미
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

### 5. 데코레이터 = C# Attribute

> `@`로 시작하는 것은 C#의 `[Attribute]`와 같습니다!

| NestJS 데코레이터 | ASP.NET Core Attribute |
|------------------|------------------------|
| `@Controller()` | `[ApiController]` |
| `@Get()` | `[HttpGet]` |
| `@Post()` | `[HttpPost]` |
| `@Injectable()` | DI 등록 |
| `@Module()` | - |
| `@Body()` | `[FromBody]` |
| `@Param()` | `[FromRoute]` |
| `@Query()` | `[FromQuery]` |

### 6. 실행 스크립트

```bash
# 개발 모드 (dotnet watch run과 비슷)
npm run start:dev

# 프로덕션 빌드 (dotnet build)
npm run build

# 프로덕션 실행 (dotnet run)
npm run start:prod

# 테스트 (dotnet test)
npm run test
```

---

## CLI 명령어 비교

| 작업 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 프로젝트 생성 | `dotnet new webapi` | `nest new 프로젝트명` |
| 컨트롤러 생성 | 수동 | `nest g controller 이름` |
| 서비스 생성 | 수동 | `nest g service 이름` |
| 모듈 생성 | - | `nest g module 이름` |
| 전체 리소스 생성 | - | `nest g resource 이름` |

```bash
# NestJS CLI로 빠른 scaffolding
nest g controller users   # UsersController 생성
nest g service users      # UsersService 생성
nest g module users       # UsersModule 생성
nest g resource users     # 위 3개 + DTO 한번에!
```

---

## 연습

### 연습 1: 빈칸 채우기

```bash
# 1. NestJS CLI 전역 설치
npm install _______ @nestjs/cli

# 2. 새 프로젝트 생성 (C#: dotnet new webapi)
nest _______ my-project

# 3. 개발 서버 실행 (C#: dotnet watch run)
npm run _______
```

```typescript
// 4. 모듈 데코레이터 (C#: ConfigureServices)
@_______({
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

// 5. 서비스 데코레이터 (C#: DI 등록)
@_______()
export class AppService {}
```

<details>
<summary>정답 보기</summary>

1. `-g`
2. `new`
3. `start:dev`
4. `Module`
5. `Injectable`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | NestJS는 ASP.NET Core와 비슷한 구조를 가진 Node.js 프레임워크이다 | |
| 2 | `main.ts`는 C#의 `Program.cs`와 비슷한 역할을 한다 | |
| 3 | NestJS의 `@Controller()`는 C#의 `[ApiController]`와 같은 역할이다 | |
| 4 | `@Injectable()`은 컨트롤러를 정의할 때 사용한다 | |
| 5 | `nest g controller users` 명령어는 UsersController를 자동 생성한다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. O
4. X (`@Injectable()`은 서비스/프로바이더에 사용, 컨트롤러는 `@Controller()`)
5. O

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번
@Controller
export class AppController {}

// 2번
@Module({
  controller: [AppController],
  provider: [AppService],
})
export class AppModule {}

// 3번
@Injectable
export class AppService {
  getHello(): string {
    return 'Hello';
  }
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 데코레이터는 함수이므로 괄호 필요
@Controller()
export class AppController {}

// 2번: 복수형이어야 함 (controllers, providers)
@Module({
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

// 3번: 데코레이터는 함수이므로 괄호 필요
@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello';
  }
}
```

</details>

### 연습 4: ASP.NET Core → NestJS 변환

| ASP.NET Core | NestJS |
|--------------|--------|
| `dotnet new webapi` | `nest _______ 프로젝트명` |
| `dotnet watch run` | `npm run _______` |
| `[ApiController]` | `@_______()` |
| `[HttpGet]` | `@_______()` |
| `Program.cs` | `_______.ts` |
| `ConfigureServices` | `@Module({ _______ })` |

<details>
<summary>정답 보기</summary>

| ASP.NET Core | NestJS |
|--------------|--------|
| `dotnet new webapi` | `nest new 프로젝트명` |
| `dotnet watch run` | `npm run start:dev` |
| `[ApiController]` | `@Controller()` |
| `[HttpGet]` | `@Get()` |
| `Program.cs` | `main.ts` |
| `ConfigureServices` | `@Module({ providers })` |

</details>

---

## 숙제

### 숙제 1: NestJS 프로젝트 생성

**문제**: `study-nest`라는 이름으로 새 NestJS 프로젝트를 생성하고 실행하세요.

**요구사항**:
- NestJS CLI로 프로젝트 생성
- 개발 모드로 서버 실행
- `http://localhost:3000` 에서 "Hello World!" 확인

```bash
# 여기에 명령어 작성 (2줄)


```

### 숙제 2: 파일 역할 정리

**문제**: NestJS 파일과 ASP.NET Core 대응을 완성하세요.

| NestJS 파일 | 역할 | ASP.NET Core 대응 |
|-------------|------|------------------|
| main.ts | | Program.cs |
| app.module.ts | | |
| app.controller.ts | | |
| app.service.ts | | |

### 숙제 3: 새 엔드포인트 추가

**문제**: `GET /info` 엔드포인트를 추가하세요.

**요구사항**:
- URL: `/info`
- 반환: `{ name: "Study NestJS", version: "1.0.0" }`
- C#의 `[HttpGet("info")]`와 동일한 기능

**힌트**: `@Get('info')` 데코레이터 사용

```typescript
// src/app.controller.ts에 추가
@Get('info')
getInfo() {
  // 여기에 코드 작성
}
```

---

## 핵심 정리

| NestJS | ASP.NET Core 대응 |
|--------|------------------|
| NestJS | ASP.NET Core |
| `main.ts` | `Program.cs` |
| `@Module()` | `ConfigureServices` |
| `@Controller()` | `[ApiController]` |
| `@Injectable()` | DI 등록 |
| `@Get()`, `@Post()` | `[HttpGet]`, `[HttpPost]` |
| `nest new` | `dotnet new webapi` |
| `npm run start:dev` | `dotnet watch run` |
