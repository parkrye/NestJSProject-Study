# Day 9: 데이터베이스 기초 (TypeORM)

## 학습 목표
- 데이터베이스와 ORM이 무엇인지 이해
- Entity가 무엇이고 어떻게 만드는지 파악
- Repository로 데이터를 다루는 방법 이해

---

## 1. 데이터베이스란?

### 한 줄 정리
> **데이터를 저장하는 창고**

### 왜 필요해?

서버를 껐다 켜면 메모리의 데이터는 사라집니다.
**데이터베이스에 저장하면** 서버를 꺼도 데이터가 유지됩니다.

```
[메모리 저장]
서버 켜짐 → 데이터 저장 → 서버 꺼짐 → 데이터 사라짐 ❌

[데이터베이스 저장]
서버 켜짐 → DB에 저장 → 서버 꺼짐 → 데이터 유지 ✅
```

| 비유 | 메모리 | 데이터베이스 |
|------|--------|-------------|
| 메모 | 포스트잇 (떨어지면 사라짐) | 노트북 (계속 남음) |
| 저장 | RAM | 하드디스크 |

---

## 2. ORM이란?

### 한 줄 정리
> **코드로 데이터베이스를 다루게 해주는 도구**

### 왜 필요해?

데이터베이스는 원래 SQL이라는 별도 언어로 다룹니다.
ORM을 쓰면 **익숙한 코드(TypeScript)로 데이터베이스를 다룰 수 있습니다.**

```sql
-- SQL 직접 작성 (어려움)
SELECT * FROM users WHERE id = 1;
INSERT INTO users (name, email) VALUES ('Kim', 'kim@test.com');
```

```typescript
// ORM 사용 (쉬움!)
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.save({ name: 'Kim', email: 'kim@test.com' });
```

| 비유 | 설명 |
|------|------|
| 통역사 | 한국어(코드) ↔ 영어(SQL) 변환 |
| 리모컨 | TV를 직접 조작 안 하고, 리모컨 버튼으로 제어 |
| 자동차 | 엔진 원리 몰라도 핸들/페달로 운전 가능 |

### TypeORM

NestJS에서 가장 많이 쓰는 ORM입니다.

```bash
# 설치 (참고용)
npm install @nestjs/typeorm typeorm
npm install better-sqlite3  # SQLite (간단한 파일 기반 DB)
```

---

## 3. Entity란?

### 한 줄 정리
> **데이터베이스 테이블의 설계도**

### 왜 필요해?

데이터베이스에 "사용자 테이블은 이런 구조야"라고 알려줘야 합니다.
Entity 클래스가 그 설계도 역할을 합니다.

| 비유 | 설명 |
|------|------|
| 엑셀 시트 헤더 | "이름, 이메일, 나이" 열이 있다고 정의 |
| 신청서 양식 | "이 칸에는 이름, 저 칸에는 이메일" |
| 건물 도면 | 어떤 구조로 지을지 설계 |

### Entity 만들기

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
} from 'typeorm';

@Entity()  // "이건 데이터베이스 테이블이야"
export class User {
  @PrimaryGeneratedColumn()  // 자동 증가하는 고유 번호 (1, 2, 3, ...)
  id: number;

  @Column({ length: 100 })   // 일반 컬럼 (최대 100글자)
  name: string;

  @Column({ unique: true })  // 유니크 (중복 불가)
  email: string;

  @Column({ default: true }) // 기본값 설정
  isActive: boolean;

  @CreateDateColumn()        // 생성 시간 자동 기록
  createdAt: Date;
}
```

### 데코레이터 설명

| 데코레이터 | 의미 | 예시 |
|------------|------|------|
| `@Entity()` | "이건 DB 테이블이야" | 클래스에 붙임 |
| `@PrimaryGeneratedColumn()` | 자동 증가 고유 번호 | id: 1, 2, 3... |
| `@Column()` | 일반 컬럼 | name, email 등 |
| `@Column({ length: 100 })` | 최대 길이 지정 | 100글자 이하 |
| `@Column({ unique: true })` | 중복 불가 | 이메일은 유일해야 함 |
| `@Column({ default: true })` | 기본값 | 안 넣으면 true |
| `@Column({ nullable: true })` | null 허용 | 없어도 됨 |
| `@CreateDateColumn()` | 생성 시간 자동 기록 | 2024-01-01 12:00:00 |
| `@UpdateDateColumn()` | 수정 시간 자동 기록 | 수정할 때마다 갱신 |

### Entity = 테이블

```
[User Entity]              [users 테이블]
@Entity()                  ┌────┬──────┬─────────────┬──────────┐
class User {               │ id │ name │ email       │ isActive │
  id: number       →       ├────┼──────┼─────────────┼──────────┤
  name: string     →       │ 1  │ Kim  │ kim@a.com   │ true     │
  email: string    →       │ 2  │ Lee  │ lee@b.com   │ true     │
  isActive: boolean→       └────┴──────┴─────────────┴──────────┘
}
```

---

## 4. Repository란?

### 한 줄 정리
> **데이터베이스와 대화하는 창구**

### 왜 필요해?

데이터베이스에서 데이터를 가져오거나, 저장하거나, 삭제하려면
**Repository를 통해** 명령을 내립니다.

| 비유 | 설명 |
|------|------|
| 창고 관리자 | "물건 가져와줘", "물건 넣어줘" 요청하면 처리 |
| 은행 창구 | "입금해주세요", "출금해주세요" 요청 |
| 도서관 사서 | "이 책 찾아주세요", "이 책 반납할게요" |

### Repository 사용하기

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)                    // User Entity의 Repository 주입
    private usersRepository: Repository<User>, // Repository 타입
  ) {}

  // 전체 조회
  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  // 하나 조회
  findOne(id: number): Promise<User | null> {
    return this.usersRepository.findOne({ where: { id } });
  }

  // 생성
  async create(data: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(data);  // 객체 생성
    return this.usersRepository.save(user);          // DB에 저장
  }

  // 수정
  async update(id: number, data: UpdateUserDto): Promise<User | null> {
    await this.usersRepository.update(id, data);
    return this.findOne(id);
  }

  // 삭제
  async remove(id: number): Promise<void> {
    await this.usersRepository.delete(id);
  }
}
```

---

## 5. CRUD 메서드

### CRUD란?

| 약자 | 의미 | 설명 |
|------|------|------|
| **C** | Create | 생성 |
| **R** | Read | 조회 |
| **U** | Update | 수정 |
| **D** | Delete | 삭제 |

### Repository 메서드 정리

| 작업 | 메서드 | 예시 |
|------|--------|------|
| 전체 조회 | `.find()` | `repository.find()` |
| 하나 조회 | `.findOne()` | `repository.findOne({ where: { id: 1 } })` |
| 조건 조회 | `.find({ where })` | `repository.find({ where: { isActive: true } })` |
| 생성 | `.create()` + `.save()` | `repository.save(repository.create(data))` |
| 수정 | `.update()` | `repository.update(id, data)` |
| 삭제 | `.delete()` | `repository.delete(id)` |

### 상세 예시

```typescript
// 전체 조회
const users = await this.usersRepository.find();

// 조건으로 조회
const activeUsers = await this.usersRepository.find({
  where: { isActive: true },
});

// 정렬
const sortedUsers = await this.usersRepository.find({
  order: { createdAt: 'DESC' },  // 최신순
});

// 개수 제한
const topUsers = await this.usersRepository.find({
  take: 10,  // 10개만
});

// 하나만 조회
const user = await this.usersRepository.findOne({
  where: { id: 1 },
});

// 이메일로 조회
const user = await this.usersRepository.findOne({
  where: { email: 'kim@test.com' },
});

// 생성
const newUser = this.usersRepository.create({
  name: 'Kim',
  email: 'kim@test.com',
});
await this.usersRepository.save(newUser);

// 수정
await this.usersRepository.update(1, { name: 'Lee' });

// 삭제
await this.usersRepository.delete(1);
```

---

## 6. Module 설정

### TypeORM 전역 설정

```typescript
// app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'better-sqlite3',  // DB 종류 (SQLite)
      database: 'database.db', // DB 파일명
      entities: [User],        // Entity 목록
      synchronize: true,       // 자동 테이블 생성 (개발용)
    }),
    UsersModule,
  ],
})
export class AppModule {}
```

| 옵션 | 설명 |
|------|------|
| `type` | 데이터베이스 종류 |
| `database` | 데이터베이스 이름/파일 |
| `entities` | Entity 클래스 목록 |
| `synchronize` | Entity 변경 시 테이블 자동 수정 (개발용만!) |

### Module에서 Entity 등록

```typescript
// users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],  // User Entity 등록
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

---

## 7. 전체 예시

### Entity

```typescript
// user.entity.ts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 100 })
  name: string;

  @Column({ unique: true })
  email: string;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;
}
```

### Service

```typescript
// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  findOne(id: number): Promise<User | null> {
    return this.usersRepository.findOne({ where: { id } });
  }

  create(dto: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(dto);
    return this.usersRepository.save(user);
  }

  async update(id: number, dto: UpdateUserDto): Promise<User | null> {
    await this.usersRepository.update(id, dto);
    return this.findOne(id);
  }

  async remove(id: number): Promise<void> {
    await this.usersRepository.delete(id);
  }
}
```

### Controller

```typescript
// users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    return this.usersService.update(+id, dto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(+id);
  }
}
```

---

## C# 개발자를 위한 비교

| 개념 | Entity Framework Core | TypeORM |
|------|----------------------|---------|
| ORM | EF Core | TypeORM |
| Entity | Entity class | `@Entity()` class |
| DbContext | `DbContext` | `Repository<T>` |
| `[Key]` | Primary Key | `@PrimaryGeneratedColumn()` |
| `[Required]` | 필수 | `nullable: false` (기본값) |
| `DbSet<T>` | 테이블 접근 | `Repository<T>` |
| `ToListAsync()` | 전체 조회 | `.find()` |
| `FindAsync(id)` | ID로 조회 | `.findOne({ where: { id } })` |
| `Add` + `SaveChanges` | 저장 | `.create()` + `.save()` |

```csharp
// EF Core
var users = await _context.Users.ToListAsync();
var user = await _context.Users.FindAsync(1);
```

```typescript
// TypeORM - 비슷!
const users = await this.usersRepository.find();
const user = await this.usersRepository.findOne({ where: { id: 1 } });
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. Entity 정의
@_______()
export class User {
  // 2. 자동 증가 기본키
  @_______()
  id: number;

  // 3. 일반 컬럼
  @_______({ length: 100 })
  name: string;

  // 4. 생성 시간 자동 기록
  @_______()
  createdAt: Date;
}

// 5. Repository 주입
constructor(
  @_______( User )
  private usersRepository: Repository<User>,
) {}
```

<details>
<summary>정답 보기</summary>

1. `Entity`
2. `PrimaryGeneratedColumn`
3. `Column`
4. `CreateDateColumn`
5. `InjectRepository`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | ORM을 사용하면 SQL 없이 데이터베이스를 다룰 수 있다 | |
| 2 | Entity는 데이터베이스 테이블의 설계도이다 | |
| 3 | `@PrimaryGeneratedColumn()`은 자동 증가하는 고유 번호를 만든다 | |
| 4 | Repository의 `.save()`는 데이터를 삭제한다 | |
| 5 | `synchronize: true`는 프로덕션에서 사용해야 한다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. O
4. X (`.save()`는 저장, `.delete()`가 삭제)
5. X (`synchronize: true`는 개발용, 프로덕션에서는 위험)

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번
@Entity
export class User {
  @PrimaryGeneratedColumn
  id: number;
}

// 2번
@Injectable()
export class UsersService {
  constructor(
    private usersRepository: Repository<User>,
  ) {}
}

// 3번
async create(dto: CreateUserDto) {
  const user = this.usersRepository.create(dto);
  return user;  // 저장 완료?
}
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: 괄호 필요
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}

// 2번: @InjectRepository 필요
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)  // 추가!
    private usersRepository: Repository<User>,
  ) {}
}

// 3번: save()를 호출해야 DB에 저장됨
async create(dto: CreateUserDto) {
  const user = this.usersRepository.create(dto);
  return this.usersRepository.save(user);  // save 필요!
}
```

</details>

### 연습 4: CRUD 메서드 매칭

| 작업 | Repository 메서드 |
|------|------------------|
| 전체 조회 | |
| 하나 조회 | |
| 생성 | |
| 수정 | |
| 삭제 | |

<details>
<summary>정답 보기</summary>

| 작업 | Repository 메서드 |
|------|------------------|
| 전체 조회 | `.find()` |
| 하나 조회 | `.findOne({ where: { id } })` |
| 생성 | `.create()` + `.save()` |
| 수정 | `.update(id, data)` |
| 삭제 | `.delete(id)` |

</details>

---

## 데이터 흐름 정리

```
[요청]
   │
   ↓
[Controller] - 요청 접수
   │
   ↓
[Service] - 비즈니스 로직
   │
   │  this.usersRepository.find()
   ↓
[Repository] - DB 명령 전달
   │
   │  SQL 변환 (ORM이 자동으로)
   ↓
[Database] - 실제 데이터 저장소
   │
   │  데이터 반환
   ↓
[응답]
```

---

## 핵심 정리

| 용어 | 한 줄 설명 |
|------|-----------|
| 데이터베이스 | 데이터를 저장하는 창고 |
| ORM | 코드로 DB를 다루게 해주는 도구 |
| TypeORM | NestJS에서 쓰는 ORM |
| Entity | DB 테이블의 설계도 |
| `@Entity()` | "이건 DB 테이블이야" |
| `@PrimaryGeneratedColumn()` | 자동 증가 고유 번호 |
| `@Column()` | 테이블의 열 (컬럼) |
| Repository | DB와 대화하는 창구 |
| `.find()` | 전체 조회 |
| `.findOne()` | 하나 조회 |
| `.save()` | 저장 |
| `.delete()` | 삭제 |
