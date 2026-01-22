# Day 8: Module과 DTO

## 학습 목표
- Module이 무엇이고 왜 필요한지 이해
- DTO가 무엇이고 어디에 쓰는지 파악
- 유효성 검사가 무엇인지 이해

---

## 1. Module이란?

### 한 줄 정리
> **관련 기능들을 묶어놓은 폴더/그룹**

### 왜 필요해?

프로젝트가 커지면 파일이 수십, 수백 개가 됩니다.
**관련된 기능끼리 묶어두면** 찾기 쉽고 관리하기 쉬워집니다.

```
[모듈 없이]
src/
├── user.controller.ts
├── user.service.ts
├── product.controller.ts
├── product.service.ts
├── order.controller.ts
├── order.service.ts
└── ... (파일이 섞여있음)

[모듈로 정리]
src/
├── users/           ← 사용자 관련 모든 것
│   ├── users.module.ts
│   ├── users.controller.ts
│   └── users.service.ts
├── products/        ← 상품 관련 모든 것
│   ├── products.module.ts
│   ├── products.controller.ts
│   └── products.service.ts
└── orders/          ← 주문 관련 모든 것
```

| 비유 | 설명 |
|------|------|
| 회사 부서 | 인사팀, 개발팀, 마케팅팀 - 기능별로 나눔 |
| 서랍장 | 양말 서랍, 속옷 서랍, 셔츠 서랍 - 종류별로 정리 |
| 앱 폴더 | 게임 폴더, 업무 폴더, SNS 폴더 |

### Module 구조

```typescript
@Module({
  imports: [],                    // 다른 모듈 가져오기
  controllers: [UsersController], // 컨트롤러 등록
  providers: [UsersService],      // 서비스 등록
  exports: [UsersService],        // 외부에 공개
})
export class UsersModule {}
```

| 옵션 | 설명 | 비유 |
|------|------|------|
| `imports` | 다른 모듈의 기능 가져오기 | 다른 부서에서 사람 빌려오기 |
| `controllers` | 이 모듈의 컨트롤러들 | 이 부서의 접수 창구들 |
| `providers` | 이 모듈의 서비스들 | 이 부서의 직원들 |
| `exports` | 다른 모듈에게 공개할 것 | 다른 부서에게 빌려줄 수 있는 것 |

---

## 2. 모듈 간 연결

### 왜 연결해?

주문(Order) 기능에서 사용자(User) 정보가 필요할 수 있습니다.
이럴 때 UsersModule의 기능을 OrdersModule에서 **빌려다 씁니다.**

### 연결 방법

```typescript
// 1. UsersModule에서 UsersService를 exports로 공개
@Module({
  providers: [UsersService],
  exports: [UsersService],  // 외부 공개!
})
export class UsersModule {}

// 2. OrdersModule에서 UsersModule을 imports
@Module({
  imports: [UsersModule],  // UsersModule 가져오기
  providers: [OrdersService],
})
export class OrdersModule {}

// 3. OrdersService에서 UsersService 사용 가능!
@Injectable()
export class OrdersService {
  constructor(private usersService: UsersService) {}

  createOrder(userId: number) {
    const user = this.usersService.findOne(userId);
    // 주문 생성 로직...
  }
}
```

| 단계 | 설명 |
|------|------|
| 1. exports | "이 서비스는 다른 모듈에서 써도 돼" |
| 2. imports | "저 모듈의 기능을 가져올게" |
| 3. 사용 | 가져온 서비스를 주입받아 사용 |

---

## 3. DTO란?

### 한 줄 정리
> **데이터의 형식을 정의한 것** (Data Transfer Object)

### 왜 필요해?

사용자가 보내는 데이터가 올바른지 확인해야 합니다.
DTO로 **"이런 형식의 데이터를 받겠다"** 를 정의합니다.

```typescript
// ❌ DTO 없이 - 무슨 데이터가 올지 모름
@Post()
create(@Body() body: any) {
  // body에 뭐가 들어있지...?
}

// ✅ DTO 사용 - 형식이 명확함
@Post()
create(@Body() body: CreateUserDto) {
  // body.name, body.email이 있다는 걸 앎!
}
```

| 비유 | 설명 |
|------|------|
| 신청서 양식 | "이름, 연락처, 주소를 적어주세요" |
| 택배 송장 | "받는 사람, 주소, 연락처 형식을 맞춰야 배송 가능" |
| 회원가입 폼 | "이메일, 비밀번호, 닉네임이 필요해요" |

### DTO 만들기

```typescript
// create-user.dto.ts
export class CreateUserDto {
  name: string;      // 이름 (필수)
  email: string;     // 이메일 (필수)
  age?: number;      // 나이 (선택 - ?가 붙으면 선택)
}

// update-user.dto.ts
export class UpdateUserDto {
  name?: string;     // 모든 필드가 선택
  email?: string;
  age?: number;
}
```

| 구분 | 설명 | 필드 |
|------|------|------|
| CreateDto | 생성할 때 필요한 데이터 | 대부분 필수 |
| UpdateDto | 수정할 때 필요한 데이터 | 대부분 선택 |

---

## 4. 유효성 검사

### 한 줄 정리
> **"데이터가 올바른지 검사하는 것"**

### 왜 필요해?

사용자가 잘못된 데이터를 보낼 수 있습니다.
- 이메일 칸에 "asdf" 입력
- 나이에 -5 입력
- 필수 항목 비워둠

**서버에서 검사해서 잘못된 데이터를 거부**해야 합니다.

### 설치 (참고용)

```bash
npm install class-validator class-transformer
```

### 유효성 검사 데코레이터

```typescript
import {
  IsString,
  IsEmail,
  IsNumber,
  IsNotEmpty,
  IsOptional,
  MinLength,
  MaxLength,
  Min,
  Max,
} from 'class-validator';

export class CreateUserDto {
  @IsString()           // 문자열이어야 함
  @IsNotEmpty()         // 비어있으면 안 됨
  @MaxLength(50)        // 최대 50글자
  name: string;

  @IsEmail()            // 이메일 형식이어야 함
  email: string;

  @IsString()
  @MinLength(8)         // 최소 8글자
  password: string;

  @IsOptional()         // 선택 (없어도 됨)
  @IsNumber()
  @Min(0)               // 0 이상
  @Max(150)             // 150 이하
  age?: number;
}
```

### 자주 쓰는 검사 데코레이터

| 데코레이터 | 의미 | 예시 |
|------------|------|------|
| `@IsString()` | 문자열 | "hello" |
| `@IsNumber()` | 숫자 | 123 |
| `@IsEmail()` | 이메일 형식 | "a@b.com" |
| `@IsNotEmpty()` | 비어있으면 안 됨 | "" (X) |
| `@IsOptional()` | 없어도 됨 | undefined (O) |
| `@MinLength(n)` | 최소 n글자 | 비밀번호 8자 이상 |
| `@MaxLength(n)` | 최대 n글자 | 이름 50자 이하 |
| `@Min(n)` | 최소값 n | 나이 0 이상 |
| `@Max(n)` | 최대값 n | 수량 100 이하 |

### ValidationPipe 설정

유효성 검사를 활성화하려면 main.ts에 설정이 필요합니다.

```typescript
// main.ts
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

| 옵션 | 설명 |
|------|------|
| `whitelist` | DTO에 정의 안 된 필드 자동 제거 |
| `forbidNonWhitelisted` | 정의 안 된 필드 있으면 에러 |
| `transform` | 문자열 "1"을 숫자 1로 자동 변환 |

---

## 5. 실제 예시

### 상품 DTO

```typescript
// create-product.dto.ts
export class CreateProductDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(100)
  name: string;         // 상품명 (필수, 100자 이하)

  @IsNumber()
  @Min(0)
  price: number;        // 가격 (필수, 0 이상)

  @IsNumber()
  @Min(0)
  @Max(10000)
  stock: number;        // 재고 (필수, 0~10000)

  @IsOptional()
  @IsString()
  description?: string; // 설명 (선택)
}

// update-product.dto.ts
export class UpdateProductDto {
  @IsOptional()
  @IsString()
  @MaxLength(100)
  name?: string;

  @IsOptional()
  @IsNumber()
  @Min(0)
  price?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  stock?: number;

  @IsOptional()
  @IsString()
  description?: string;
}
```

### Controller에서 사용

```typescript
@Controller('products')
export class ProductsController {
  constructor(private productsService: ProductsService) {}

  @Post()
  create(@Body() dto: CreateProductDto) {
    // dto는 이미 유효성 검사 완료!
    // dto.name, dto.price 등이 올바른 값임을 보장
    return this.productsService.create(dto);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() dto: UpdateProductDto) {
    return this.productsService.update(+id, dto);
  }
}
```

---

## 6. 에러 메시지 커스터마이징

```typescript
export class CreateUserDto {
  @IsString({ message: '이름은 문자열이어야 합니다' })
  @IsNotEmpty({ message: '이름은 필수입니다' })
  name: string;

  @IsEmail({}, { message: '올바른 이메일 형식이 아닙니다' })
  email: string;

  @MinLength(8, { message: '비밀번호는 8자 이상이어야 합니다' })
  password: string;
}
```

잘못된 데이터 전송 시 응답:
```json
{
  "statusCode": 400,
  "message": [
    "이름은 필수입니다",
    "올바른 이메일 형식이 아닙니다"
  ],
  "error": "Bad Request"
}
```

---

## C# 개발자를 위한 비교

| 개념 | ASP.NET Core | NestJS |
|------|-------------|--------|
| 모듈 | Assembly / 폴더 구조 | `@Module()` |
| DTO | DTO class | DTO class (동일!) |
| `[Required]` | Data Annotations | `@IsNotEmpty()` |
| `[EmailAddress]` | Data Annotations | `@IsEmail()` |
| `[StringLength]` | Data Annotations | `@MaxLength()` |
| `[Range]` | Data Annotations | `@Min()`, `@Max()` |
| nullable `?` | nullable | `@IsOptional()` |

```csharp
// ASP.NET Core
public class CreateUserDto {
    [Required]
    [StringLength(50)]
    public string Name { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }
}
```

```typescript
// NestJS - 비슷한 패턴!
export class CreateUserDto {
  @IsNotEmpty()
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;
}
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. 모듈 정의
@_______({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// 2. DTO 유효성 검사
export class CreateUserDto {
  @IsString()
  @_______()     // 필수값
  name: string;

  @_______()     // 이메일 형식
  email: string;

  @_______()     // 선택적
  @IsNumber()
  age?: number;
}
```

<details>
<summary>정답 보기</summary>

1. `Module`
2. `IsNotEmpty`
3. `IsEmail`
4. `IsOptional`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | Module은 관련 기능들을 묶어놓은 것이다 | |
| 2 | exports에 등록된 서비스만 다른 모듈에서 사용할 수 있다 | |
| 3 | DTO는 데이터의 형식을 정의한 것이다 | |
| 4 | `@IsOptional()`은 필수값을 의미한다 | |
| 5 | ValidationPipe를 설정하면 자동으로 유효성 검사가 된다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. O
4. X (`@IsOptional()`은 선택값, 없어도 됨)
5. O

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번 - 모듈
@Module({
  controller: [UsersController],
  provider: [UsersService],
})
export class UsersModule {}

// 2번 - DTO
export class CreateUserDto {
  @IsString
  @IsNotEmpty
  name: string;
}

// 3번 - 선택적 필드
export class UpdateUserDto {
  @IsString()
  name?: string;  // 선택적 필드
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 복수형이어야 함
@Module({
  controllers: [UsersController],  // controllers
  providers: [UsersService],       // providers
})
export class UsersModule {}

// 2번: 괄호 필요
export class CreateUserDto {
  @IsString()      // 괄호!
  @IsNotEmpty()    // 괄호!
  name: string;
}

// 3번: 선택적 필드는 @IsOptional() 필요
export class UpdateUserDto {
  @IsOptional()    // 추가!
  @IsString()
  name?: string;
}
```

</details>

### 연습 4: 주문 DTO 만들기

다음 요구사항에 맞는 DTO를 작성하세요.

**요구사항:**
- `productId`: 숫자, 필수, 1 이상
- `quantity`: 숫자, 필수, 1~100 사이
- `address`: 문자열, 필수, 10자 이상
- `memo`: 문자열, 선택

```typescript
export class CreateOrderDto {
  // 여기에 작성
}
```

<details>
<summary>정답 보기</summary>

```typescript
export class CreateOrderDto {
  @IsNumber()
  @Min(1)
  productId: number;

  @IsNumber()
  @Min(1)
  @Max(100)
  quantity: number;

  @IsString()
  @MinLength(10)
  address: string;

  @IsOptional()
  @IsString()
  memo?: string;
}
```

</details>

---

## 전체 구조 정리

```
AppModule (루트)
├── imports: [UsersModule, ProductsModule, OrdersModule]
│
├── UsersModule
│   ├── controllers: [UsersController]
│   ├── providers: [UsersService]
│   └── exports: [UsersService]  ← 다른 모듈에서 사용 가능
│
├── ProductsModule
│   ├── controllers: [ProductsController]
│   └── providers: [ProductsService]
│
└── OrdersModule
    ├── imports: [UsersModule]   ← UsersService 사용
    ├── controllers: [OrdersController]
    └── providers: [OrdersService]
```

---

## 핵심 정리

| 용어 | 한 줄 설명 |
|------|-----------|
| Module | 관련 기능들을 묶어놓은 그룹 |
| `imports` | 다른 모듈의 기능 가져오기 |
| `exports` | 다른 모듈에게 공개할 것 |
| `providers` | 서비스 등록 |
| DTO | 데이터의 형식을 정의한 것 |
| 유효성 검사 | 데이터가 올바른지 검사 |
| `@IsNotEmpty()` | 필수값 |
| `@IsEmail()` | 이메일 형식 |
| `@IsOptional()` | 선택값 (없어도 됨) |
| `@Min()`, `@Max()` | 최소/최대값 |
| ValidationPipe | 유효성 검사 활성화 |
