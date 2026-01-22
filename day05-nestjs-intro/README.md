# Day 5: NestJS 소개와 프로젝트 구조

## 학습 목표
- NestJS가 무엇이고 왜 사용하는지 이해
- NestJS 프로젝트의 기본 구조 파악
- 각 파일의 역할 이해

---

## 1. NestJS란?

### 한 줄 정리
> **백엔드 서버를 체계적으로 만들게 해주는 도구 모음 (프레임워크)**

### 왜 필요해?

Node.js로 서버를 만들 수 있지만, 규칙 없이 코드를 짜면:
- 파일이 어디 있는지 찾기 힘듦
- 다른 사람이 내 코드를 이해하기 어려움
- 프로젝트가 커지면 엉망이 됨

NestJS는 **"서버는 이렇게 만들어라"** 라는 정해진 구조를 제공합니다.

| 비유 | 설명 |
|------|------|
| 건축 설계도 | 집을 아무렇게나 짓지 않고, 설계도대로 지으면 튼튼한 집 완성 |
| 레고 설명서 | 설명서대로 조립하면 예쁜 작품 완성 |
| 요리 레시피 | 레시피대로 하면 맛있는 요리 완성 |

### NestJS의 장점

| 장점 | 설명 |
|------|------|
| 구조화 | 정해진 규칙이 있어서 코드가 깔끔하게 정리됨 |
| 협업 용이 | 모두가 같은 방식으로 개발하니까 이해하기 쉬움 |
| TypeScript | 타입이 있어서 실수가 줄어듦 |
| 확장성 | 프로젝트가 커져도 관리하기 쉬움 |

### C# 개발자를 위한 비교

NestJS는 **ASP.NET Core와 매우 유사한 구조**입니다.

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 프레임워크 타입 | 백엔드 프레임워크 | 백엔드 프레임워크 |
| 언어 | C# | TypeScript |
| 컨트롤러 | `[ApiController]` | `@Controller()` |
| 서비스 | DI 등록 | `@Injectable()` |
| 데코레이터 | `[Attribute]` | `@Decorator()` |

---

## 2. NestJS 프로젝트 구조

### 폴더 구조 살펴보기

```
my-project/
├── src/                    ← 소스 코드 폴더
│   ├── main.ts             ← 서버 시작점
│   ├── app.module.ts       ← 앱 설정 (부품 목록)
│   ├── app.controller.ts   ← 요청 받는 곳
│   └── app.service.ts      ← 실제 일하는 곳
├── package.json            ← 프로젝트 설정
└── tsconfig.json           ← TypeScript 설정
```

### 각 파일의 역할

```
[요청 흐름]

사용자 요청 → main.ts → controller → service → 응답
              (시작)    (접수)      (처리)
```

| 파일 | 역할 | 비유 |
|------|------|------|
| main.ts | 서버 시작 | 가게 문 열기 |
| app.module.ts | 부품 목록 | 가게 메뉴판 |
| app.controller.ts | 요청 접수 | 카운터 직원 |
| app.service.ts | 실제 처리 | 주방 요리사 |

---

## 3. 각 파일 자세히 보기

### main.ts - 서버 시작점

> **"서버야, 켜져!"**

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);  // 앱 생성
  await app.listen(3000);                           // 3000번 포트에서 대기
}
bootstrap();
```

| 비유 | 설명 |
|------|------|
| 가게 오픈 | "오늘부터 영업 시작합니다!" |
| 전화 개통 | "3000번으로 전화 오면 받겠습니다" |

### app.module.ts - 부품 목록

> **"이 앱은 이런 기능들이 있어요"**

```typescript
@Module({
  imports: [],                    // 다른 모듈 가져오기
  controllers: [AppController],   // 컨트롤러 등록
  providers: [AppService],        // 서비스 등록
})
export class AppModule {}
```

| 옵션 | 설명 | 비유 |
|------|------|------|
| imports | 다른 모듈 가져오기 | 다른 부서에서 기능 빌려오기 |
| controllers | 컨트롤러 목록 | 접수 창구 목록 |
| providers | 서비스 목록 | 작업자 목록 |

### app.controller.ts - 요청 받는 곳

> **"요청이 오면 여기로!"** (안내 데스크)

```typescript
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()  // GET 요청이 오면
  getHello(): string {
    return this.appService.getHello();  // 서비스에게 일 시킴
  }
}
```

| 비유 | 설명 |
|------|------|
| 식당 직원 | 손님 주문 받고 → 주방에 전달 → 음식 가져다줌 |
| 안내 데스크 | 방문객 요청 받고 → 담당 부서에 연결 → 결과 전달 |

### app.service.ts - 실제 일하는 곳

> **"실제 일은 내가 해"** (작업 부서)

```typescript
@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';  // 실제 작업 수행
  }
}
```

| 비유 | 설명 |
|------|------|
| 주방 | 실제로 요리하는 곳 |
| 작업 부서 | 실제 업무를 처리하는 곳 |

---

## 4. 데코레이터 (@)

### 한 줄 정리
> **코드에 특별한 기능을 부여하는 표시** (스티커 같은 것)

### 왜 필요해?

"이 클래스는 컨트롤러야", "이 클래스는 서비스야"를 NestJS에게 알려줘야 합니다.
`@` 기호로 시작하는 데코레이터가 그 역할을 합니다.

```typescript
@Controller()      // "이건 컨트롤러야"
export class UsersController { }

@Injectable()      // "이건 주입 가능한 서비스야"
export class UsersService { }

@Module({...})     // "이건 모듈이야"
export class UsersModule { }
```

### 자주 쓰는 데코레이터

| 데코레이터 | 의미 | 어디에 붙임 |
|------------|------|------------|
| `@Module()` | "이건 모듈이야" | 모듈 클래스 |
| `@Controller()` | "이건 컨트롤러야" | 컨트롤러 클래스 |
| `@Injectable()` | "이건 주입 가능해" | 서비스 클래스 |
| `@Get()` | "GET 요청 처리해" | 메서드 |
| `@Post()` | "POST 요청 처리해" | 메서드 |

### C# 개발자를 위한 비교

| NestJS | ASP.NET Core |
|--------|--------------|
| `@Controller()` | `[ApiController]` |
| `@Get()` | `[HttpGet]` |
| `@Post()` | `[HttpPost]` |
| `@Injectable()` | DI 등록 |

---

## 5. 명령어 정리 (참고용)

```bash
# NestJS CLI 설치
npm install -g @nestjs/cli

# 새 프로젝트 생성
nest new 프로젝트명

# 개발 서버 실행 (파일 변경 감지)
npm run start:dev

# 프로덕션 빌드
npm run build

# 프로덕션 실행
npm run start:prod
```

### CLI로 파일 자동 생성

```bash
nest g controller users   # UsersController 생성
nest g service users      # UsersService 생성
nest g module users       # UsersModule 생성
nest g resource users     # 위 3개 + DTO 한번에 생성!
```

---

## 연습

### 연습 1: 파일 역할 매칭

| 파일 | 역할 |
|------|------|
| main.ts | |
| app.module.ts | |
| app.controller.ts | |
| app.service.ts | |

<details>
<summary>정답 보기</summary>

| 파일 | 역할 |
|------|------|
| main.ts | 서버 시작점 (앱 실행) |
| app.module.ts | 부품 목록 (컨트롤러, 서비스 등록) |
| app.controller.ts | 요청 접수 (안내 데스크) |
| app.service.ts | 실제 처리 (작업 부서) |

</details>

### 연습 2: 빈칸 채우기

```typescript
// 1. 모듈 데코레이터
@_______({
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

// 2. 컨트롤러 데코레이터
@_______()
export class AppController { }

// 3. 서비스 데코레이터
@_______()
export class AppService { }

// 4. GET 요청 처리
@_______()
getHello() { }
```

<details>
<summary>정답 보기</summary>

1. `Module`
2. `Controller`
3. `Injectable`
4. `Get`

</details>

### 연습 3: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | NestJS는 백엔드 서버를 만들기 위한 프레임워크이다 | |
| 2 | main.ts는 서버의 시작점이다 | |
| 3 | Controller는 실제 비즈니스 로직을 처리한다 | |
| 4 | `@Injectable()`은 컨트롤러에 붙이는 데코레이터이다 | |
| 5 | `nest g resource users`는 모듈, 컨트롤러, 서비스를 한번에 생성한다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. X (Controller는 요청을 받고, 실제 처리는 Service에서)
4. X (`@Injectable()`은 서비스에, 컨트롤러에는 `@Controller()`)
5. O

</details>

### 연습 4: 틀린 곳 찾기

```typescript
// 1번 - 데코레이터
@Controller
export class AppController {}

// 2번 - 모듈
@Module({
  controller: [AppController],
  provider: [AppService],
})
export class AppModule {}

// 3번 - 서비스
@Injectable
export class AppService {}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 데코레이터는 함수이므로 괄호 필요
@Controller()
export class AppController {}

// 2번: 복수형이어야 함
@Module({
  controllers: [AppController],  // controllers
  providers: [AppService],       // providers
})
export class AppModule {}

// 3번: 괄호 필요
@Injectable()
export class AppService {}
```

</details>

---

## 전체 흐름 다시 보기

```
[사용자]
   |
   | HTTP 요청 (예: GET /hello)
   ↓
[main.ts] - 서버가 요청을 받음
   |
   ↓
[app.module.ts] - 어떤 컨트롤러가 처리할지 확인
   |
   ↓
[app.controller.ts] - 요청 접수, 서비스에게 일 시킴
   |
   ↓
[app.service.ts] - 실제 작업 수행
   |
   ↓
[사용자] - 응답 받음 (예: "Hello World!")
```

---

## 핵심 정리

| 용어 | 한 줄 설명 |
|------|-----------|
| NestJS | 백엔드 서버를 체계적으로 만드는 프레임워크 |
| main.ts | 서버 시작점 (앱을 켜는 곳) |
| Module | 관련 기능들을 묶어놓은 것 (부품 목록) |
| Controller | 요청을 받는 곳 (안내 데스크) |
| Service | 실제 일을 하는 곳 (작업 부서) |
| @데코레이터 | 코드에 특별한 기능을 부여하는 표시 |
| `@Controller()` | "이건 컨트롤러야" |
| `@Injectable()` | "이건 주입 가능한 서비스야" |
| `@Module()` | "이건 모듈이야" |
