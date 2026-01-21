# Day 10: 인증과 API 보안 기초

## 학습 목표
- 인증(Authentication)과 인가(Authorization) 구분
- JWT 개념 이해
- Guard 사용법 파악

---

## C# 개발자를 위한 핵심 차이점

> **ASP.NET Core 인증과 거의 동일한 개념입니다!**

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 인증 | Authentication | Authentication |
| 인가 | Authorization | Authorization |
| JWT | `JwtBearer` | `@nestjs/jwt` |
| 인가 필터 | `[Authorize]` | `@UseGuards()` |
| 역할 기반 | `[Authorize(Roles = "Admin")]` | `@Roles('admin')` + Guard |
| 현재 사용자 | `User.Identity` | `@Request() req.user` |
| 설정 | `appsettings.json` | `.env` + ConfigService |

```csharp
// ASP.NET Core
[Authorize]  // 인증 필요
[ApiController]
public class UsersController : ControllerBase {
    [HttpGet("profile")]
    public IActionResult GetProfile() {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        return Ok(new { userId });
    }

    [Authorize(Roles = "Admin")]  // 관리자만
    [HttpGet("admin")]
    public IActionResult AdminOnly() => Ok("Admin");
}
```

```typescript
// NestJS - 거의 동일!
@UseGuards(AuthGuard)  // [Authorize]
@Controller('users')
export class UsersController {
  @Get('profile')
  getProfile(@Request() req) {
    return { userId: req.user.sub };
  }

  @UseGuards(AuthGuard, RolesGuard)  // [Authorize(Roles = "Admin")]
  @Roles('admin')
  @Get('admin')
  adminOnly() { return 'Admin'; }
}
```

---

## 공부

### 1. 인증 vs 인가 (C#과 동일 개념)

| 구분 | 인증 (Authentication) | 인가 (Authorization) |
|------|----------------------|---------------------|
| 질문 | "누구세요?" | "권한이 있나요?" |
| ASP.NET | `Authentication` 미들웨어 | `[Authorize]` 속성 |
| NestJS | AuthGuard | RolesGuard 등 |

### 2. JWT (C#과 동일 개념)

> ASP.NET Core의 JWT Bearer 인증과 동일!

```csharp
// ASP.NET Core - JWT 설정
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
        options.TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(key),
            // ...
        };
    });
```

```typescript
// NestJS - JWT 설정
@Module({
  imports: [
    JwtModule.register({
      secret: 'your-secret-key',
      signOptions: { expiresIn: '1h' },
    }),
  ],
})
export class AuthModule {}
```

**JWT 구조 (C#과 동일):**
```
헤더.페이로드.서명
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9.xxx

// 페이로드 (Claims)
{
  "sub": 1,        // Subject (User ID) - ClaimTypes.NameIdentifier
  "email": "...",  // ClaimTypes.Email
  "role": "admin"  // ClaimTypes.Role
}
```

### 3. 인증 서비스 - ASP.NET Core와 비교

```csharp
// ASP.NET Core
public class AuthService {
    private readonly IConfiguration _config;

    public string GenerateToken(User user) {
        var claims = new[] {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Email, user.Email)
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            claims: claims,
            expires: DateTime.Now.AddHours(1),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

```typescript
// NestJS - 더 간단!
@Injectable()
export class AuthService {
  constructor(private jwtService: JwtService) {}

  async login(user: User) {
    const payload = { sub: user.id, email: user.email };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }

  validateToken(token: string) {
    try {
      return this.jwtService.verify(token);
    } catch {
      return null;
    }
  }
}
```

### 4. Guard - ASP.NET Core [Authorize]와 비교

```csharp
// ASP.NET Core - 커스텀 Authorization Handler
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement> {
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement) {
        // 검증 로직...
        context.Succeed(requirement);
        return Task.CompletedTask;
    }
}
```

```typescript
// NestJS Guard - 비슷한 패턴!
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);

    if (!token) return false;

    try {
      const payload = this.jwtService.verify(token);
      request.user = payload;  // User.Identity와 비슷
      return true;
    } catch {
      return false;
    }
  }

  private extractToken(request: any): string | null {
    const auth = request.headers.authorization;
    if (auth?.startsWith('Bearer ')) {
      return auth.substring(7);
    }
    return null;
  }
}
```

### 5. Guard 사용 - [Authorize] 속성과 비교

```csharp
// ASP.NET Core
[Authorize]  // 인증 필요
public class UsersController : ControllerBase {
    [AllowAnonymous]  // 인증 불필요
    [HttpGet("public")]
    public IActionResult Public() => Ok();

    [HttpGet("profile")]
    public IActionResult Profile() {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        return Ok(new { userId });
    }
}

[Authorize(Roles = "Admin")]  // 컨트롤러 전체에 적용
public class AdminController : ControllerBase { }
```

```typescript
// NestJS
@Controller('users')
export class UsersController {
  // [AllowAnonymous] - Guard 없음
  @Get('public')
  public() { return 'Public'; }

  // [Authorize]
  @UseGuards(AuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return { userId: req.user.sub };
  }
}

// 컨트롤러 전체에 적용 - [Authorize(Roles = "Admin")]
@UseGuards(AuthGuard)
@Controller('admin')
export class AdminController { }
```

### 6. 환경 변수 - appsettings.json과 비교

```csharp
// ASP.NET Core - appsettings.json
{
  "Jwt": {
    "Key": "your-secret-key",
    "Issuer": "your-app"
  },
  "ConnectionStrings": {
    "Default": "Server=..."
  }
}

// 사용
var key = _configuration["Jwt:Key"];
```

```typescript
// NestJS - .env 파일
JWT_SECRET=your-secret-key
DATABASE_URL=postgres://...

// 설정
npm install @nestjs/config

// app.module.ts
@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true })],
})
export class AppModule {}

// 사용
@Injectable()
export class AuthService {
  constructor(private configService: ConfigService) {}

  getSecret() {
    return this.configService.get('JWT_SECRET');
  }
}
```

**.gitignore:**
```
# ASP.NET Core
appsettings.Development.json

# NestJS
.env
```

---

## 인증 흐름 비교

```csharp
// ASP.NET Core Controller
[ApiController]
[Route("auth")]
public class AuthController : ControllerBase {
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginDto dto) {
        var user = await _userService.ValidateUser(dto.Email, dto.Password);
        if (user == null) return Unauthorized();

        var token = _authService.GenerateToken(user);
        return Ok(new { access_token = token });
    }
}
```

```typescript
// NestJS Controller - 거의 동일!
@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  async login(@Body() dto: LoginDto) {
    const user = await this.authService.validateUser(dto.email, dto.password);
    if (!user) {
      throw new UnauthorizedException();
    }
    return this.authService.login(user);
  }
}
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. Guard 정의 (C#: AuthorizationHandler)
@Injectable()
export class AuthGuard implements _______ {
  canActivate(context: ExecutionContext): boolean {
    // 인증 로직
    return true;
  }
}

// 2. Guard 적용 (C#: [Authorize])
@_______( AuthGuard )
@Get('profile')
getProfile(@Request() req) {
  return req.user;
}

// 3. JWT 서비스 주입
constructor(private _______: JwtService) {}

// 4. 토큰 생성
const token = this.jwtService._______( payload );

// 5. 토큰 검증
const decoded = this.jwtService._______( token );
```

<details>
<summary>정답 보기</summary>

1. `CanActivate`
2. `UseGuards`
3. `jwtService`
4. `sign`
5. `verify`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | 인증(Authentication)은 "누구세요?"를 확인하는 것이다 | |
| 2 | 인가(Authorization)는 "권한이 있나요?"를 확인하는 것이다 | |
| 3 | NestJS의 Guard는 ASP.NET Core의 `[Authorize]`와 비슷한 역할이다 | |
| 4 | JWT 토큰은 Header, Payload, Signature 세 부분으로 구성된다 | |
| 5 | `@UseGuards(AuthGuard)`는 인증 없이 접근 가능하게 한다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. O
4. O
5. X (Guard를 적용하면 인증이 필요함)

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext) {
    // boolean을 반환해야 하는데...
  }
}

// 2번
@UseGuards  // Guard 적용
@Get('profile')
getProfile() {}

// 3번
@Controller('auth')
export class AuthController {
  @Post('login')
  login(@Body() body) {
    const token = jwtService.sign({ sub: 1 });  // 토큰 생성
    return { access_token: token };
  }
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: boolean 또는 Promise<boolean>을 반환해야 함
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    return true;  // 또는 false
  }
}

// 2번: UseGuards는 함수이므로 괄호와 Guard 클래스 필요
@UseGuards(AuthGuard)
@Get('profile')
getProfile() {}

// 3번: jwtService가 주입되지 않음, this 필요
@Controller('auth')
export class AuthController {
  constructor(private jwtService: JwtService) {}  // 주입 필요

  @Post('login')
  login(@Body() body) {
    const token = this.jwtService.sign({ sub: 1 });  // this 필요
    return { access_token: token };
  }
}
```

</details>

### 연습 4: ASP.NET Core → NestJS 변환

| ASP.NET Core | NestJS |
|--------------|--------|
| `[Authorize]` | `@_______( AuthGuard )` |
| `User.FindFirst(ClaimTypes.NameIdentifier)` | `req.user._______` |
| `JwtBearer` | `@nestjs/_______` |
| `Authorization Handler` | `_______ implements CanActivate` |
| `appsettings.json` | `._______` + ConfigService |

<details>
<summary>정답 보기</summary>

| ASP.NET Core | NestJS |
|--------------|--------|
| `[Authorize]` | `@UseGuards( AuthGuard )` |
| `User.FindFirst(ClaimTypes.NameIdentifier)` | `req.user.sub` |
| `JwtBearer` | `@nestjs/jwt` |
| `Authorization Handler` | `Guard implements CanActivate` |
| `appsettings.json` | `.env` + ConfigService |

</details>

---

## 숙제

### 숙제 1: 인증과 인가 차이점 정리

**문제**: 인증과 인가의 차이점을 정리하세요.

| 구분 | 인증 (Authentication) | 인가 (Authorization) |
|------|----------------------|---------------------|
| 영어 | Authentication | Authorization |
| 질문 | | |
| ASP.NET Core | | `[Authorize]` |
| NestJS | | |
| 예시 | | |

### 숙제 2: JWT 구조 설명

**문제**: JWT의 세 부분과 각 역할을 설명하세요.

```
1. Header:
   - 역할:
   - 내용:

2. Payload (Claims):
   - 역할:
   - 내용 예시:

3. Signature:
   - 역할:
   - 생성 방법:
```

### 숙제 3: 간단한 AuthGuard 구현

**문제**: Bearer 토큰을 검증하는 간단한 Guard를 구현하세요.

**요구사항**:
- `Authorization: Bearer {token}` 헤더에서 토큰 추출
- JwtService로 토큰 검증
- 검증 성공 시 `request.user`에 페이로드 저장
- 실패 시 `false` 반환

**힌트**: C#의 Authorization Handler와 비슷한 패턴

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();

    // 여기에 코드 작성
    // 1. 헤더에서 토큰 추출
    // 2. 토큰 검증
    // 3. request.user에 페이로드 저장
    // 4. 성공/실패 반환

  }
}
```

---

## 핵심 정리

| NestJS | ASP.NET Core 대응 |
|--------|------------------|
| 인증/인가 | Authentication/Authorization |
| JWT | JwtBearer |
| Guard | `[Authorize]` / Authorization Handler |
| `@UseGuards()` | `[Authorize]` |
| `@Request() req.user` | `User.Identity` / Claims |
| ConfigService | IConfiguration |
| `.env` | `appsettings.json` |

---

## 학습 완료 체크리스트

10일 과정을 마친 후 확인하세요:

- [ ] JavaScript 기본 문법을 이해한다 (C# 대응 가능)
- [ ] async/await를 이해한다 (C# Task와 동일)
- [ ] TypeScript 타입을 안다 (C#과 매우 유사)
- [ ] npm을 사용할 수 있다 (NuGet과 동일 개념)
- [ ] NestJS 구조를 안다 (ASP.NET Core와 유사)
- [ ] Controller를 만들 수 있다 (C# Controller와 동일)
- [ ] Service와 DI를 이해한다 (ASP.NET Core DI와 동일)
- [ ] Module과 DTO를 안다 (프로젝트 구조화)
- [ ] TypeORM을 안다 (EF Core와 유사)
- [ ] 인증 기초를 안다 (ASP.NET Core 인증과 동일)
