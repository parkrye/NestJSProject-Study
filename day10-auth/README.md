# Day 10: 인증과 API 보안 기초

## 학습 목표
- 인증(Authentication)과 인가(Authorization)의 차이 이해
- JWT가 무엇이고 어떻게 동작하는지 파악
- Guard가 무엇이고 어디에 쓰는지 이해

---

## 1. 인증 vs 인가

### 한 줄 정리
> **인증: "누구세요?" / 인가: "권한이 있나요?"**

### 왜 구분해?

로그인(인증)과 권한 확인(인가)은 다른 개념입니다.

```
[인증 (Authentication)]
"당신이 누구인지 확인"
예: 로그인 - 아이디/비밀번호로 본인 확인

[인가 (Authorization)]
"당신이 이걸 할 수 있는지 확인"
예: 관리자만 삭제 가능 - 권한 확인
```

| 구분 | 인증 (Authentication) | 인가 (Authorization) |
|------|----------------------|---------------------|
| 질문 | "누구세요?" | "권한이 있나요?" |
| 예시 | 로그인 | 관리자 전용 페이지 접근 |
| 비유 | 신분증 확인 | VIP 라운지 입장 자격 |
| 시점 | 먼저 (누구인지 확인) | 나중 (뭘 할 수 있는지 확인) |

### 실제 예시

```
1. 로그인 (인증)
   사용자: "저는 kim@test.com이에요"
   서버: "비밀번호 확인... 맞네요, 김철수님이군요!" ✅

2. 관리자 페이지 접근 (인가)
   김철수: "관리자 페이지 보여줘"
   서버: "김철수님은 일반 사용자라서 안 돼요" ❌

   관리자: "관리자 페이지 보여줘"
   서버: "관리자님이시군요, 여기요!" ✅
```

---

## 2. JWT란?

### 한 줄 정리
> **로그인 증명서** (JSON Web Token)

### 왜 필요해?

HTTP는 **매 요청이 독립적**입니다. (상태를 기억 안 함)
그래서 "나 로그인했어"를 매번 증명해야 합니다.

```
[JWT 없이]
요청1: "게시글 보여줘" → 서버: "누구세요?"
요청2: "내 정보 보여줘" → 서버: "누구세요?"
요청3: "글 쓸게" → 서버: "누구세요?"
(매번 로그인해야 함)

[JWT 사용]
로그인 → JWT 발급 받음 (증명서)
요청1: "게시글 보여줘" + JWT → 서버: "김철수님이군요, 여기요"
요청2: "내 정보 보여줘" + JWT → 서버: "김철수님이군요, 여기요"
(JWT만 보여주면 OK)
```

| 비유 | 설명 |
|------|------|
| 놀이공원 팔찌 | 입장 시 받고, 놀이기구 탈 때마다 보여주면 됨 |
| 회원 카드 | 카드 보여주면 "우리 회원이시네요" |
| 출입증 | 목에 걸고 다니며 출입할 때 보여줌 |

### JWT 구조

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9.xxxxx
└──────┬──────────┘ └─────┬─────┘ └──┬──┘
     Header          Payload      Signature
     (헤더)           (내용)        (서명)
```

| 부분 | 설명 | 내용 |
|------|------|------|
| Header | 토큰 타입, 암호화 방식 | `{ "alg": "HS256" }` |
| Payload | 실제 데이터 (사용자 정보) | `{ "sub": 1, "email": "..." }` |
| Signature | 위조 방지 서명 | 비밀키로 생성 |

### Payload 예시

```json
{
  "sub": 1,           // Subject - 사용자 ID
  "email": "kim@test.com",
  "role": "admin",    // 역할 (관리자/일반)
  "iat": 1234567890,  // 발급 시간
  "exp": 1234571490   // 만료 시간
}
```

---

## 3. JWT 인증 흐름

### 전체 과정

```
[1. 로그인]
사용자 → "아이디: kim, 비밀번호: 1234" → 서버
                                          ↓
                                    비밀번호 확인 ✅
                                          ↓
사용자 ← JWT 토큰 발급 ←──────────────── 서버

[2. 이후 요청]
사용자 → 요청 + JWT 토큰 → 서버
                            ↓
                      토큰 검증 ✅
                            ↓
사용자 ← 응답 ←──────────── 서버
```

### 토큰 전달 방법

```
HTTP 요청 헤더에 담아서 전송:

Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOjF9.xxxxx
               └─┬──┘ └──────────────┬────────────────────┘
              타입                 JWT 토큰
```

---

## 4. Guard란?

### 한 줄 정리
> **요청을 통과시킬지 결정하는 문지기**

### 왜 필요해?

특정 API는 **로그인한 사용자만** 접근해야 합니다.
Guard가 요청을 검사해서 **통과/차단**을 결정합니다.

```
[Guard 없이]
아무나 → GET /profile → 다른 사람 정보 볼 수 있음 ❌

[Guard 있음]
비로그인 → GET /profile → Guard: "로그인 안 됨, 차단!" ❌
로그인됨 → GET /profile → Guard: "토큰 확인, 통과!" ✅
```

| 비유 | 설명 |
|------|------|
| 경비원 | 출입증 확인하고 통과/차단 |
| 클럽 바운서 | 성인만 입장 가능 확인 |
| 지하철 게이트 | 카드 찍어야 통과 |

### Guard 만들기

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();

    // 1. 헤더에서 토큰 꺼내기
    const token = this.extractToken(request);
    if (!token) {
      return false;  // 토큰 없음 → 차단
    }

    try {
      // 2. 토큰 검증
      const payload = this.jwtService.verify(token);

      // 3. 사용자 정보를 request에 저장
      request.user = payload;

      return true;  // 통과
    } catch {
      return false;  // 토큰 이상함 → 차단
    }
  }

  private extractToken(request: any): string | null {
    const auth = request.headers.authorization;
    if (auth?.startsWith('Bearer ')) {
      return auth.substring(7);  // "Bearer " 다음 부분
    }
    return null;
  }
}
```

### Guard 사용하기

```typescript
@Controller('users')
export class UsersController {

  // Guard 없음 - 누구나 접근 가능
  @Get('public')
  public() {
    return '누구나 볼 수 있음';
  }

  // Guard 있음 - 로그인한 사용자만
  @UseGuards(AuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return { userId: req.user.sub };  // Guard에서 저장한 user 사용
  }
}

// 컨트롤러 전체에 Guard 적용
@UseGuards(AuthGuard)
@Controller('admin')
export class AdminController {
  // 이 컨트롤러의 모든 메서드에 Guard 적용
}
```

---

## 5. 인증 서비스 만들기

### AuthService

```typescript
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  // 로그인
  async login(email: string, password: string) {
    // 1. 사용자 찾기
    const user = await this.usersService.findByEmail(email);
    if (!user) {
      throw new UnauthorizedException('이메일 또는 비밀번호가 틀렸습니다');
    }

    // 2. 비밀번호 확인 (실제로는 암호화된 비밀번호 비교)
    if (user.password !== password) {
      throw new UnauthorizedException('이메일 또는 비밀번호가 틀렸습니다');
    }

    // 3. JWT 토큰 생성
    const payload = { sub: user.id, email: user.email };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}
```

### AuthController

```typescript
@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  login(@Body() body: { email: string; password: string }) {
    return this.authService.login(body.email, body.password);
  }

  @UseGuards(AuthGuard)
  @Get('me')
  getMe(@Request() req) {
    return req.user;  // 현재 로그인한 사용자 정보
  }
}
```

---

## 6. 환경 변수

### 왜 필요해?

JWT 비밀키 같은 민감한 정보는 **코드에 직접 쓰면 안 됩니다.**
`.env` 파일에 따로 저장합니다.

```
# .env 파일
JWT_SECRET=my-super-secret-key-12345
DATABASE_URL=postgres://localhost:5432/mydb
```

### 설치 및 설정 (참고용)

```bash
npm install @nestjs/config
```

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),  // 환경 변수 로드
    // ...
  ],
})
export class AppModule {}
```

### 사용

```typescript
@Injectable()
export class AuthService {
  constructor(private configService: ConfigService) {}

  getSecret() {
    return this.configService.get('JWT_SECRET');
  }
}
```

**중요:** `.env` 파일은 `.gitignore`에 추가해서 Git에 올리지 않습니다!

```
# .gitignore
.env
```

---

## C# 개발자를 위한 비교

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 인증 | Authentication | Authentication |
| 인가 | Authorization | Authorization |
| JWT | JwtBearer | `@nestjs/jwt` |
| 인가 필터 | `[Authorize]` | `@UseGuards()` |
| 역할 기반 | `[Authorize(Roles = "Admin")]` | `@Roles('admin')` + RolesGuard |
| 현재 사용자 | `User.Identity` | `@Request() req.user` |
| 설정 | `appsettings.json` | `.env` + ConfigService |

```csharp
// ASP.NET Core
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile() {
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    return Ok(new { userId });
}
```

```typescript
// NestJS - 비슷!
@UseGuards(AuthGuard)
@Get('profile')
getProfile(@Request() req) {
  return { userId: req.user.sub };
}
```

---

## 연습

### 연습 1: 인증 vs 인가 구분

| 상황 | 인증? 인가? |
|------|-----------|
| 로그인 | |
| 관리자만 삭제 가능 | |
| 회원가입 | |
| VIP만 접근 가능 | |
| 아이디/비밀번호 확인 | |

<details>
<summary>정답 보기</summary>

| 상황 | 인증? 인가? |
|------|-----------|
| 로그인 | 인증 |
| 관리자만 삭제 가능 | 인가 |
| 회원가입 | 둘 다 아님 (새 사용자 생성) |
| VIP만 접근 가능 | 인가 |
| 아이디/비밀번호 확인 | 인증 |

</details>

### 연습 2: 빈칸 채우기

```typescript
// 1. Guard 인터페이스 구현
@Injectable()
export class AuthGuard implements _______ {
  canActivate(context: ExecutionContext): boolean {
    // 검증 로직
    return true;
  }
}

// 2. Guard 적용
@_______( AuthGuard )
@Get('profile')
getProfile(@Request() req) {
  return req.user;
}

// 3. JWT 토큰 생성
const token = this.jwtService._______( payload );

// 4. JWT 토큰 검증
const decoded = this.jwtService._______( token );
```

<details>
<summary>정답 보기</summary>

1. `CanActivate`
2. `UseGuards`
3. `sign`
4. `verify`

</details>

### 연습 3: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | 인증은 "누구세요?"를 확인하는 것이다 | |
| 2 | 인가는 "권한이 있나요?"를 확인하는 것이다 | |
| 3 | JWT는 서버에 저장된다 | |
| 4 | Guard는 요청을 통과시킬지 결정한다 | |
| 5 | JWT 비밀키는 코드에 직접 써도 된다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. X (JWT는 클라이언트가 저장하고, 요청 시 전송)
4. O
5. X (환경 변수로 분리해야 함)

</details>

### 연습 4: 틀린 곳 찾기

```typescript
// 1번
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext) {
    // boolean 반환 안 함
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
    const token = jwtService.sign({ sub: 1 });
    return { access_token: token };
  }
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: boolean 반환 필요
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    return true;  // 또는 false
  }
}

// 2번: 괄호와 Guard 클래스 필요
@UseGuards(AuthGuard)
@Get('profile')
getProfile() {}

// 3번: jwtService 주입 필요, this 필요
@Controller('auth')
export class AuthController {
  constructor(private jwtService: JwtService) {}  // 주입

  @Post('login')
  login(@Body() body) {
    const token = this.jwtService.sign({ sub: 1 });  // this 필요
    return { access_token: token };
  }
}
```

</details>

---

## 전체 인증 흐름

```
[로그인]
사용자 ──POST /auth/login──→ AuthController
        { email, password }        │
                                   ↓
                             AuthService.login()
                                   │
                                   ↓
                             비밀번호 확인 ✅
                                   │
                                   ↓
                             JWT 토큰 생성
                                   │
사용자 ←── { access_token } ←──────┘


[보호된 API 접근]
사용자 ──GET /users/profile──→ Guard
        + Authorization: Bearer xxx    │
                                       ↓
                                 토큰 검증
                                       │
                              ┌────────┴────────┐
                              ↓                 ↓
                          통과 ✅            차단 ❌
                              │             401 Unauthorized
                              ↓
                         Controller
                              │
                              ↓
사용자 ←─── 응답 ←────────────┘
```

---

## 핵심 정리

| 용어 | 한 줄 설명 |
|------|-----------|
| 인증 (Authentication) | "누구세요?" - 로그인 |
| 인가 (Authorization) | "권한이 있나요?" - 접근 제어 |
| JWT | 로그인 증명서 토큰 |
| Payload | JWT 안의 사용자 정보 |
| Guard | 요청 통과/차단 문지기 |
| `@UseGuards()` | Guard 적용 |
| `CanActivate` | Guard가 구현하는 인터페이스 |
| `jwtService.sign()` | 토큰 생성 |
| `jwtService.verify()` | 토큰 검증 |
| `.env` | 환경 변수 파일 (비밀키 저장) |

---

## 학습 완료 체크리스트

Day 1~10 과정을 마친 후 확인하세요:

- [ ] JavaScript 기본 문법을 이해한다
- [ ] async/await를 이해한다
- [ ] TypeScript 타입을 안다
- [ ] npm을 사용할 수 있다
- [ ] NestJS 구조를 안다 (Module, Controller, Service)
- [ ] Controller로 API를 만들 수 있다
- [ ] Service로 비즈니스 로직을 분리할 수 있다
- [ ] DTO로 데이터 형식을 정의할 수 있다
- [ ] TypeORM으로 데이터베이스를 다룰 수 있다
- [ ] JWT 인증의 개념을 이해한다
