# Day 9: 데이터베이스 기초 (TypeORM)

## 학습 목표
- 데이터베이스 기본 개념 이해
- ORM과 TypeORM 개념 파악
- Entity와 Repository 사용법 습득

---

## C# 개발자를 위한 핵심 차이점

> **TypeORM = Entity Framework Core**
> EF Core를 알고 있다면 TypeORM은 매우 쉽습니다!

| 개념 | Entity Framework Core | TypeORM |
|------|----------------------|---------|
| ORM | EF Core | TypeORM |
| Entity | Entity class | `@Entity()` class |
| DbContext | `DbContext` | `Repository<T>` |
| DbSet<T> | `DbSet<T>` | `Repository<T>` |
| Migration | `Add-Migration` | `migration:generate` |
| `[Key]` | `[Key]` | `@PrimaryGeneratedColumn()` |
| `[Column]` | - | `@Column()` |
| LINQ | LINQ | QueryBuilder / find 메서드 |

```csharp
// Entity Framework Core
public class User {
    [Key]
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    [Required]
    public string Email { get; set; }
}

// DbContext
public class AppDbContext : DbContext {
    public DbSet<User> Users { get; set; }
}

// 사용
var users = await _context.Users.ToListAsync();
var user = await _context.Users.FindAsync(1);
await _context.Users.AddAsync(newUser);
await _context.SaveChangesAsync();
```

```typescript
// TypeORM - 거의 동일!
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 100 })
  name: string;

  @Column({ unique: true })
  email: string;
}

// Repository 사용
const users = await userRepository.find();
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.save(newUser);
// SaveChanges 불필요 - save()가 바로 저장
```

---

## 공부

### 1. ORM이란?

> **C#의 Entity Framework와 같은 역할!**
> SQL 없이 객체로 데이터베이스 조작

```csharp
// EF Core - LINQ로 쿼리
var user = await _context.Users
    .Where(u => u.Id == 1)
    .FirstOrDefaultAsync();
```

```typescript
// TypeORM - 비슷한 방식
const user = await userRepository.findOne({
  where: { id: 1 }
});
```

### 2. TypeORM 설치

```bash
npm install @nestjs/typeorm typeorm

# SQLite (파일 기반 - 개발용 간편)
npm install better-sqlite3

# SQL Server (C# 개발자에게 익숙!)
npm install mssql

# MySQL
npm install mysql2

# PostgreSQL
npm install pg
```

### 3. Entity - EF Core Entity와 비교

```csharp
// Entity Framework Core
public class User {
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    [Required]
    [Column(TypeName = "nvarchar(100)")]
    public string Name { get; set; }

    [Required]
    public string Email { get; set; }

    public bool IsActive { get; set; } = true;

    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

```typescript
// TypeORM
@Entity()
export class User {
  @PrimaryGeneratedColumn()  // [Key] + Identity
  id: number;

  @Column({ length: 100 })   // [Column(TypeName = "nvarchar(100)")]
  name: string;

  @Column({ unique: true })  // [Required] + Unique
  email: string;

  @Column({ default: true }) // = true 기본값
  isActive: boolean;

  @CreateDateColumn()        // 자동 생성 시간
  createdAt: Date;
}
```

**Column 데코레이터 비교:**
| EF Core | TypeORM | 설명 |
|---------|---------|------|
| `[Key]` | `@PrimaryGeneratedColumn()` | 기본키 |
| `[Required]` | `nullable: false` (기본값) | 필수 |
| `[MaxLength(100)]` | `@Column({ length: 100 })` | 길이 |
| `[Column(TypeName=)]` | `@Column({ type: })` | 타입 지정 |
| 없음 | `@Column({ unique: true })` | 유니크 |
| `= defaultValue` | `@Column({ default: })` | 기본값 |

### 4. TypeORM 설정 - EF Core DbContext와 비교

```csharp
// EF Core - Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

```typescript
// NestJS - app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'better-sqlite3',   // 또는 'mssql', 'mysql', 'postgres'
      database: 'database.db',
      entities: [User],
      synchronize: true,        // 개발용: 자동 마이그레이션
    }),
    UsersModule,
  ],
})
export class AppModule {}
```

### 5. Repository 패턴 - EF Core DbSet과 비교

```csharp
// EF Core - DbContext 주입
public class UsersService {
    private readonly AppDbContext _context;

    public UsersService(AppDbContext context) {
        _context = context;
    }

    public async Task<List<User>> GetAllAsync()
        => await _context.Users.ToListAsync();

    public async Task<User?> GetByIdAsync(int id)
        => await _context.Users.FindAsync(id);

    public async Task<User> CreateAsync(User user) {
        await _context.Users.AddAsync(user);
        await _context.SaveChangesAsync();
        return user;
    }
}
```

```typescript
// TypeORM - Repository 주입
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

  async create(data: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(data);
    return this.usersRepository.save(user);
    // SaveChanges 불필요!
  }
}
```

**Module 등록:**
```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],  // DbSet 등록과 비슷
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

### 6. CRUD 메서드 비교

| EF Core | TypeORM | 설명 |
|---------|---------|------|
| `.ToListAsync()` | `.find()` | 전체 조회 |
| `.FindAsync(id)` | `.findOne({ where: { id } })` | ID로 조회 |
| `.FirstOrDefaultAsync(x => ...)` | `.findOne({ where: {...} })` | 조건 조회 |
| `.AddAsync(entity)` | `.create()` + `.save()` | 생성 |
| `.Update(entity)` | `.save(entity)` | 수정 |
| `.Remove(entity)` | `.delete(id)` 또는 `.remove()` | 삭제 |
| `.SaveChangesAsync()` | 불필요 (자동) | 저장 |

```csharp
// EF Core
var users = await _context.Users
    .Where(u => u.IsActive)
    .OrderByDescending(u => u.CreatedAt)
    .Take(10)
    .ToListAsync();
```

```typescript
// TypeORM
const users = await this.usersRepository.find({
  where: { isActive: true },
  order: { createdAt: 'DESC' },
  take: 10,
});
```

### 7. 조건 검색 - LINQ vs TypeORM

```csharp
// EF Core LINQ
var user = await _context.Users
    .FirstOrDefaultAsync(u => u.Email == "kim@test.com");

var users = await _context.Users
    .Where(u => u.IsActive && u.Name.Contains("Kim"))
    .ToListAsync();

var userIds = new[] { 1, 2, 3 };
var users = await _context.Users
    .Where(u => userIds.Contains(u.Id))
    .ToListAsync();
```

```typescript
// TypeORM
const user = await this.usersRepository.findOne({
  where: { email: 'kim@test.com' }
});

const users = await this.usersRepository.find({
  where: {
    isActive: true,
    name: Like('%Kim%')  // LINQ Contains와 비슷
  }
});

// IN 조건
import { In } from 'typeorm';
const users = await this.usersRepository.find({
  where: { id: In([1, 2, 3]) }
});
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. Entity 정의 (C#: public class User)
@_______()
export class User {
  // 2. 기본키 자동 증가 (C#: [Key])
  @_______()
  id: number;

  // 3. 컬럼 정의 (C#: [MaxLength(100)])
  @_______({ length: 100 })
  name: string;

  // 4. 생성 시간 자동 기록
  @_______()
  createdAt: Date;
}

// 5. Repository 주입 (C#: DbSet<User>)
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
| 1 | TypeORM은 C#의 Entity Framework Core와 비슷한 역할이다 | |
| 2 | `@PrimaryGeneratedColumn()`은 C#의 `[Key]`와 같은 역할이다 | |
| 3 | TypeORM에서 데이터를 저장할 때 `SaveChanges()`를 호출해야 한다 | |
| 4 | `findOne({ where: { id } })`는 EF Core의 `FindAsync(id)`와 비슷하다 | |
| 5 | `Repository<T>`는 EF Core의 `DbSet<T>`와 비슷한 역할이다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. X (TypeORM의 `save()`는 자동으로 저장, SaveChanges 불필요)
4. O
5. O

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
    private usersRepository: Repository<User>,  // 주입
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
// 1번: 데코레이터는 함수이므로 괄호 필요
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}

// 2번: @InjectRepository 데코레이터 필요
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}
}

// 3번: save()를 호출해야 실제 저장됨
async create(dto: CreateUserDto) {
  const user = this.usersRepository.create(dto);
  return this.usersRepository.save(user);  // save 필요!
}
```

</details>

### 연습 4: EF Core → TypeORM 변환

| EF Core | TypeORM |
|---------|---------|
| `[Key]` | `@_______()` |
| `[Required]` | `nullable: _______` (기본값) |
| `[MaxLength(100)]` | `@Column({ _______: 100 })` |
| `DbSet<User>` | `Repository<_______>` |
| `ToListAsync()` | `._______()` |
| `FindAsync(id)` | `.findOne({ _______: { id } })` |
| `AddAsync` + `SaveChanges` | `.create()` + `._______()` |

<details>
<summary>정답 보기</summary>

| EF Core | TypeORM |
|---------|---------|
| `[Key]` | `@PrimaryGeneratedColumn()` |
| `[Required]` | `nullable: false` (기본값) |
| `[MaxLength(100)]` | `@Column({ length: 100 })` |
| `DbSet<User>` | `Repository<User>` |
| `ToListAsync()` | `.find()` |
| `FindAsync(id)` | `.findOne({ where: { id } })` |
| `AddAsync` + `SaveChanges` | `.create()` + `.save()` |

</details>

---

## 숙제

### 숙제 1: CRUD 메서드 매칭

**문제**: EF Core와 TypeORM의 CRUD 메서드를 매칭하세요.

| 약자 | 전체 | EF Core | TypeORM |
|------|------|---------|---------|
| C | Create | `AddAsync` + `SaveChanges` | |
| R | Read | `ToListAsync` / `FindAsync` | |
| U | Update | `Update` + `SaveChanges` | |
| D | Delete | `Remove` + `SaveChanges` | |

### 숙제 2: Product Entity 설계

**문제**: C# Entity를 참고하여 TypeORM Entity를 작성하세요.

```csharp
// C# Entity
public class Product {
    [Key] public int Id { get; set; }
    [Required][MaxLength(100)] public string Name { get; set; }
    [Required] public decimal Price { get; set; }
    public int Stock { get; set; } = 0;
    public string? Description { get; set; }  // nullable
    public DateTime CreatedAt { get; set; }
}
```

```typescript
// TypeORM Entity로 변환
@Entity()
export class Product {
  // 여기에 코드 작성

}
```

### 숙제 3: Products Service CRUD 구현

**문제**: EF Core 스타일로 TypeORM Service를 구현하세요.

**요구사항**:
- `findAll()`: 전체 조회
- `findOne(id)`: ID로 조회
- `create(dto)`: 생성
- `update(id, dto)`: 수정
- `remove(id)`: 삭제

```typescript
@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private productsRepository: Repository<Product>,
  ) {}

  // 여기에 CRUD 메서드 구현


}
```

---

## 핵심 정리

| TypeORM | Entity Framework Core 대응 |
|---------|---------------------------|
| TypeORM | EF Core |
| `@Entity()` | Entity class |
| `@PrimaryGeneratedColumn()` | `[Key]` |
| `@Column()` | 속성 (기본) |
| `Repository<T>` | `DbSet<T>` |
| `.find()` | `.ToListAsync()` |
| `.findOne()` | `.FindAsync()` / `.FirstOrDefault()` |
| `.save()` | `.Add()` + `.SaveChanges()` |
| `.delete()` | `.Remove()` + `.SaveChanges()` |
| `synchronize: true` | Auto Migration |
