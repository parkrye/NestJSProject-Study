# Day 8: Module과 DTO

## 학습 목표
- Module의 역할과 구조 이해
- DTO 개념과 사용법 습득
- 유효성 검사 기초 파악

---

## C# 개발자를 위한 핵심 차이점

> **DTO와 Validation은 C#과 거의 동일합니다!**

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| DTO | `public class CreateUserDto` | `export class CreateUserDto` |
| 유효성 검사 | Data Annotations | class-validator |
| `[Required]` | `@IsNotEmpty()` |
| `[StringLength]` | `@MinLength()`, `@MaxLength()` |
| `[EmailAddress]` | `@IsEmail()` |
| `[Range]` | `@Min()`, `@Max()` |
| 모듈 | Assembly / 폴더 구조 | `@Module()` |

```csharp
// ASP.NET Core DTO with Data Annotations
public class CreateUserDto {
    [Required]
    [StringLength(50, MinimumLength = 2)]
    public string Name { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [MinLength(8)]
    public string Password { get; set; }
}
```

```typescript
// NestJS DTO - 거의 동일한 패턴!
export class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```

---

## 공부

### 1. Module이란?

> **C#의 Assembly나 프로젝트 분리와 비슷한 개념**

```
[NestJS 모듈 구조]

AppModule (루트 = Startup.cs)
├── UsersModule    (Users 폴더/기능)
├── ProductsModule (Products 폴더/기능)
└── OrdersModule   (Orders 폴더/기능)
```

### 2. Module 구조 - ASP.NET Core와 비교

```csharp
// ASP.NET Core - Startup.cs에서 모든 것 등록
public void ConfigureServices(IServiceCollection services) {
    services.AddControllers();
    services.AddScoped<IUserService, UserService>();
    services.AddScoped<IProductService, ProductService>();
    // 모든 서비스가 한 곳에...
}
```

```typescript
// NestJS - 모듈별로 분리!
@Module({
  imports: [],              // 다른 모듈 가져오기
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],  // 외부에 공개
})
export class UsersModule {}

// 루트 모듈에서 조합
@Module({
  imports: [UsersModule, ProductsModule, OrdersModule],
})
export class AppModule {}
```

**Module 옵션:**
| 옵션 | 설명 | C# 비교 |
|------|------|---------|
| imports | 다른 모듈 가져오기 | 프로젝트 참조 |
| controllers | 컨트롤러 등록 | - |
| providers | 서비스 등록 | AddScoped 등 |
| exports | 외부 공개 | public 접근자 |

### 3. 모듈 간 의존성 - C# 프로젝트 참조와 비슷

```csharp
// C# - 프로젝트 참조로 다른 프로젝트의 서비스 사용
// Orders.csproj → Users.csproj 참조 추가
public class OrdersService {
    private readonly IUserService _userService;
    public OrdersService(IUserService userService) { }
}
```

```typescript
// NestJS - imports/exports로 모듈 간 연결
// users.module.ts
@Module({
  providers: [UsersService],
  exports: [UsersService],  // 외부 공개!
})
export class UsersModule {}

// orders.module.ts
@Module({
  imports: [UsersModule],  // UsersModule 가져오기
  providers: [OrdersService],
})
export class OrdersModule {}

// orders.service.ts - UsersService 주입 가능!
@Injectable()
export class OrdersService {
  constructor(private usersService: UsersService) {}
}
```

### 4. DTO - C# Data Annotations과 비교

```csharp
// ASP.NET Core
public class CreateUserDto {
    [Required(ErrorMessage = "이름은 필수입니다")]
    public string Name { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }
}

public class UpdateUserDto {
    public string? Name { get; set; }  // Nullable = Optional
    public string? Email { get; set; }
}
```

```typescript
// NestJS
export class CreateUserDto {
  @IsString()
  @IsNotEmpty({ message: '이름은 필수입니다' })
  name: string;

  @IsEmail()
  email: string;
}

export class UpdateUserDto {
  @IsOptional()  // C#의 Nullable과 비슷
  @IsString()
  name?: string;

  @IsOptional()
  @IsEmail()
  email?: string;
}
```

### 5. 유효성 검사 데코레이터 비교

| ASP.NET Core | NestJS (class-validator) | 설명 |
|--------------|-------------------------|------|
| `[Required]` | `@IsNotEmpty()` | 필수값 |
| `[StringLength(max)]` | `@MaxLength(max)` | 최대 길이 |
| `[MinLength(min)]` | `@MinLength(min)` | 최소 길이 |
| `[EmailAddress]` | `@IsEmail()` | 이메일 형식 |
| `[Range(min,max)]` | `@Min()`, `@Max()` | 숫자 범위 |
| `[RegularExpression]` | `@Matches(regex)` | 정규식 |
| nullable (`?`) | `@IsOptional()` | 선택적 |

```bash
# 설치
npm install class-validator class-transformer
```

```typescript
// NestJS DTO 예시
import {
  IsString, IsNumber, IsEmail, IsBoolean,
  IsOptional, IsNotEmpty, MinLength, MaxLength,
  Min, Max, IsArray, IsEnum
} from 'class-validator';

export class CreateProductDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(100)
  name: string;

  @IsNumber()
  @Min(0)
  price: number;

  @IsNumber()
  @Min(0)
  @Max(10000)
  stock: number;

  @IsOptional()
  @IsString()
  description?: string;

  @IsArray()
  @IsString({ each: true })  // 배열 각 요소 검사
  tags: string[];

  @IsEnum(['active', 'inactive'])
  status: string;
}
```

### 6. ValidationPipe 설정 - C# ModelState와 비슷

```csharp
// ASP.NET Core - 자동 유효성 검사
[ApiController]  // 이것만 붙이면 ModelState 자동 검사
public class UsersController : ControllerBase { }
```

```typescript
// NestJS - main.ts에 전역 설정
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,           // DTO에 없는 속성 제거
    forbidNonWhitelisted: true, // DTO에 없는 속성 있으면 에러
    transform: true,           // 타입 자동 변환
  }));

  await app.listen(3000);
}
```

### 7. CLI로 Module 생성

```bash
# 모듈 생성
nest g module users

# 전체 리소스 생성 (Module + Controller + Service + DTO)
nest g resource users
# CRUD 자동 생성 옵션 선택 가능!
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. 모듈 정의 (C#: Assembly/프로젝트)
@_______({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// 2. DTO 유효성 검사 (C#: [Required])
export class CreateUserDto {
  @IsString()
  @_______()  // 필수값
  name: string;

  @_______()  // 이메일 형식
  email: string;

  @IsNumber()
  @_______(0)  // 최소값 0
  age: number;

  @_______()  // 선택적 (C#: string?)
  @IsString()
  nickname?: string;
}
```

<details>
<summary>정답 보기</summary>

1. `Module`
2. `IsNotEmpty`
3. `IsEmail`
4. `Min`
5. `IsOptional`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | NestJS의 Module은 C#의 프로젝트/Assembly 분리와 비슷한 역할이다 | |
| 2 | `exports`에 등록된 서비스만 다른 모듈에서 사용할 수 있다 | |
| 3 | `@IsNotEmpty()`는 C#의 `[Required]`와 같은 역할이다 | |
| 4 | `@IsOptional()`은 해당 필드가 반드시 있어야 함을 의미한다 | |
| 5 | ValidationPipe를 사용하면 C#의 ModelState 검사처럼 자동 유효성 검사가 된다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. O
4. X (`@IsOptional()`은 선택적 필드, 있어도 되고 없어도 됨)
5. O

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번
@Module({
  controller: [UsersController],
  provider: [UsersService],
})
export class UsersModule {}

// 2번
export class CreateUserDto {
  @IsString
  @IsNotEmpty
  name: string;
}

// 3번
export class UpdateUserDto {
  @IsString()
  name?: string;  // 선택적 필드인데...
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 복수형이어야 함 (controllers, providers)
@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// 2번: 데코레이터는 함수이므로 괄호 필요
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;
}

// 3번: 선택적 필드는 @IsOptional() 필요
export class UpdateUserDto {
  @IsOptional()
  @IsString()
  name?: string;
}
```

</details>

### 연습 4: C# Data Annotations → NestJS 변환

| C# Data Annotations | NestJS class-validator |
|--------------------|------------------------|
| `[Required]` | `@_______()` |
| `[EmailAddress]` | `@_______()` |
| `[StringLength(100)]` | `@_______(100)` |
| `[MinLength(5)]` | `@_______(5)` |
| `[Range(0, 100)]` | `@Min(0)`, `@_______(100)` |
| `string?` (nullable) | `@_______()` + `?:` |

<details>
<summary>정답 보기</summary>

| C# Data Annotations | NestJS class-validator |
|--------------------|------------------------|
| `[Required]` | `@IsNotEmpty()` |
| `[EmailAddress]` | `@IsEmail()` |
| `[StringLength(100)]` | `@MaxLength(100)` |
| `[MinLength(5)]` | `@MinLength(5)` |
| `[Range(0, 100)]` | `@Min(0)`, `@Max(100)` |
| `string?` (nullable) | `@IsOptional()` + `?:` |

</details>

---

## 숙제

### 숙제 1: 게시글 DTO 설계

**문제**: C# Data Annotations 스타일로 먼저 설계한 후 NestJS로 변환하세요.

```csharp
// C#로 먼저 설계
public class CreatePostDto {
    [Required]
    [StringLength(200)]
    public string Title { get; set; }

    [Required]
    public string Content { get; set; }

    [Required]
    [StringLength(50)]
    public string Author { get; set; }

    public string[] Tags { get; set; }  // 선택적
}
```

```typescript
// NestJS로 변환
export class CreatePostDto {
  // 여기에 코드 작성

}
```

### 숙제 2: 모듈 구조 설계

**문제**: Posts 기능을 위한 모듈을 설계하세요.

**요구사항**:
- PostsModule 정의
- PostsController 등록
- PostsService 등록 및 외부 공개 (exports)

```typescript
@Module({
  // 여기에 코드 작성

})
export class PostsModule {}
```

### 숙제 3: 주문 DTO 유효성 검사

**문제**: 다음 주문 DTO에 유효성 검사 데코레이터를 추가하세요.

**요구사항**:
- `productId`: 숫자, 필수, 1 이상
- `quantity`: 숫자, 필수, 1~100 사이
- `address`: 문자열, 필수, 10자 이상
- `memo`: 문자열, 선택적

```typescript
export class CreateOrderDto {
  // productId - 데코레이터 추가

  productId: number;

  // quantity - 데코레이터 추가

  quantity: number;

  // address - 데코레이터 추가

  address: string;

  // memo - 데코레이터 추가

  memo?: string;
}
```

---

## 핵심 정리

| NestJS | ASP.NET Core 대응 |
|--------|------------------|
| `@Module()` | Assembly / 폴더 구조 |
| `imports` | 프로젝트 참조 |
| `exports` | public 접근 |
| `providers` | `services.AddScoped()` |
| DTO class | DTO class (동일!) |
| `@IsNotEmpty()` | `[Required]` |
| `@IsEmail()` | `[EmailAddress]` |
| `@Min()/@Max()` | `[Range()]` |
| `@IsOptional()` | nullable `?` |
| ValidationPipe | ModelState 자동 검사 |
