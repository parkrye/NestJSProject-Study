# Day 3: TypeScript 기초

## 학습 목표
- TypeScript의 필요성 이해
- 기본 타입 지정 방법 습득
- 인터페이스와 클래스 기초 파악

---

## C# 개발자를 위한 핵심 차이점

> **좋은 소식**: TypeScript는 C# 창시자(Anders Hejlsberg)가 만들었습니다!
> C#과 매우 유사한 문법과 개념을 가지고 있어 빠르게 익힐 수 있습니다.

| 개념 | C# | TypeScript |
|------|-----|------------|
| 타입 선언 | `int x = 1;` | `let x: number = 1;` |
| 인터페이스 | `interface IUser` | `interface User` (I 접두사 안 씀) |
| 클래스 | `public class User` | `class User` |
| 접근 제한자 | `public`, `private`, `protected` | 동일! |
| 제네릭 | `List<T>` | `Array<T>` 또는 `T[]` |
| Nullable | `int?` | `number \| null` |
| 상속 | `: BaseClass` | `extends BaseClass` |
| 구현 | `: IInterface` | `implements Interface` |

```csharp
// C#
public interface IUser {
    string Name { get; set; }
    int Age { get; set; }
}

public class User : IUser {
    public string Name { get; set; }
    public int Age { get; set; }
}
```

```typescript
// TypeScript - 매우 비슷!
interface User {
  name: string;
  age: number;
}

class UserImpl implements User {
  name: string;
  age: number;
}
```

---

## 학습

### 1. TypeScript란?

> **C#을 아는 당신에게**: JavaScript에 C# 같은 타입 시스템을 추가한 것!

```typescript
// JavaScript - 런타임 에러 (실행해봐야 앎)
let name = 'Kim';
name = 123;  // 에러 없이 실행, 나중에 문제 발생

// TypeScript - 컴파일 에러 (바로 발견)
let name: string = 'Kim';
name = 123;  // 에러! C#처럼 컴파일 시점에 발견
```

### 2. 기본 타입 - C#과 비교

| C# | TypeScript | 비고 |
|----|------------|------|
| `int`, `double`, `float` | `number` | 숫자는 하나로 통합 |
| `string` | `string` | 동일 |
| `bool` | `boolean` | 이름만 다름 |
| `object` | `any` | 모든 타입 (사용 자제) |
| `dynamic` | `any` | 비슷한 개념 |
| `int[]` | `number[]` | 배열 |
| `List<int>` | `number[]` 또는 `Array<number>` | 배열 |
| `int?` | `number \| null` | Nullable |
| `void` | `void` | 동일 |

```typescript
// C# 스타일로 생각하면 쉬움!

// C#: string name = "Kim";
let name: string = 'Kim';

// C#: int age = 25;
let age: number = 25;

// C#: bool isActive = true;
let isActive: boolean = true;

// C#: string[] names = { "Kim", "Lee" };
let names: string[] = ['Kim', 'Lee'];

// C#: int? nullableAge = null;
let nullableAge: number | null = null;
```

### 3. 인터페이스 - C#과 거의 동일!

```csharp
// C#
public interface IUser {
    int Id { get; }
    string Name { get; set; }
    string? Email { get; set; }  // Nullable
}
```

```typescript
// TypeScript
interface User {
  readonly id: number;    // C#의 { get; }
  name: string;
  email?: string;         // ? = 선택적 (C#의 Nullable과 비슷)
}

// 사용
const user: User = {
  id: 1,
  name: 'Kim',
  // email 생략 가능 (optional)
};
```

**인터페이스 확장 - C#과 동일:**
```csharp
// C#
public interface IAdmin : IUser {
    string Role { get; set; }
}
```

```typescript
// TypeScript
interface Admin extends User {
  role: string;
}
```

### 4. 타입 별칭 (Type Alias) - C#의 using alias 확장판

```csharp
// C# - 제한적
using UserId = System.Int32;
```

```typescript
// TypeScript - 더 강력!
type ID = number | string;  // Union 타입 (C#에 없음!)

let userId: ID = 123;
userId = 'abc123';  // 둘 다 OK

// 객체 타입도 정의 가능
type Product = {
  name: string;
  price: number;
};
```

### 5. 클래스 - C#과 매우 유사!

```csharp
// C#
public class User {
    public int Id { get; }
    public string Name { get; set; }
    private string _email;

    public User(int id, string name, string email) {
        Id = id;
        Name = name;
        _email = email;
    }

    public string Greet() => $"Hello, I'm {Name}";
}
```

```typescript
// TypeScript - 거의 동일!
class User {
  public id: number;
  public name: string;
  private email: string;

  constructor(id: number, name: string, email: string) {
    this.id = id;
    this.name = name;
    this.email = email;
  }

  greet(): string {
    return `Hello, I'm ${this.name}`;
  }
}

// 사용 - C#과 동일!
const user = new User(1, 'Kim', 'kim@test.com');
console.log(user.greet());
```

**축약 문법 (C#의 Primary Constructor와 비슷):**
```typescript
// TypeScript - 생성자 매개변수에 접근 제한자 붙이면 자동으로 속성 생성!
class User {
  constructor(
    public id: number,
    public name: string,
    private email: string
  ) {}
  // 자동으로 this.id = id; 등이 처리됨
}
```

### 6. 함수 타입 - C# Func/Action과 비슷

```csharp
// C#
Func<int, int, int> add = (a, b) => a + b;
Action<string> log = msg => Console.WriteLine(msg);
```

```typescript
// TypeScript
const add: (a: number, b: number) => number = (a, b) => a + b;
const log: (msg: string) => void = (msg) => console.log(msg);

// 또는 타입 추론 활용 (더 일반적)
const add = (a: number, b: number): number => a + b;
const log = (message: string): void => console.log(message);
```

### 7. 제네릭 - C#과 동일!

```csharp
// C#
public class Repository<T> where T : class {
    public T? Find(int id) { ... }
    public List<T> FindAll() { ... }
}
```

```typescript
// TypeScript
class Repository<T> {
  find(id: number): T | null { ... }
  findAll(): T[] { ... }
}

// 사용
const userRepo = new Repository<User>();
const user = userRepo.find(1);
```

---

## 연습

### 연습 1: 빈칸 채우기

```typescript
// 1. 숫자 타입 지정
let age: ________ = 25;

// 2. 문자열 배열 타입
let names: ________[] = ['Kim', 'Lee'];

// 3. 선택적 속성 (C#의 string?)
interface User {
  name: string;
  email______ string;  // 있어도 되고 없어도 됨
}

// 4. 클래스 상속 키워드 (C#의 : BaseClass)
class Admin _________ User {
}

// 5. 인터페이스 구현 키워드 (C#의 : IInterface)
class UserImpl _________ User {
}
```

<details>
<summary>정답 보기</summary>

1. `number`
2. `string`
3. `?:` (물음표 뒤에 콜론)
4. `extends`
5. `implements`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | TypeScript에서 `int`와 `float`는 모두 `number` 타입이다 | |
| 2 | TypeScript 인터페이스 이름은 C#처럼 I로 시작해야 한다 | |
| 3 | `string?`은 TypeScript에서 선택적 속성을 의미한다 | |
| 4 | TypeScript의 `private`는 C#의 `private`와 같은 역할을 한다 | |
| 5 | TypeScript에서 상속은 `extends`, 구현은 `implements`를 사용한다 | |

<details>
<summary>정답 보기</summary>

1. O
2. X (TypeScript에서는 I 접두사 안 씀)
3. X (`?:`로 표시. `string | null`이 nullable)
4. O
5. O

</details>

### 연습 3: 틀린 곳 찾기

```typescript
// 1번
let age: int = 25;

// 2번
interface IUser {
  name: String;
  age: Number;
}

// 3번
class User extends IUser {
  name: string;
  age: number;
}

// 4번
let nullable: number? = null;
```

<details>
<summary>정답 보기</summary>

```typescript
// 1번: int가 아니라 number
let age: number = 25;

// 2번: I 접두사 불필요, String/Number가 아니라 string/number (소문자)
interface User {
  name: string;
  age: number;
}

// 3번: 인터페이스는 extends가 아니라 implements
class User implements IUser {
  name: string;
  age: number;
}

// 4번: number?가 아니라 number | null
let nullable: number | null = null;
```

</details>

### 연습 4: C# → TypeScript 변환

| C# | TypeScript |
|----|------------|
| `int x = 10;` | `let x: ________ = 10;` |
| `string? name = null;` | `let name: string \| ________ = null;` |
| `public int Age { get; }` | `_________ age: number;` |
| `interface IUser` | `interface ________` |
| `class User : IUser` | `class User _________ User` |

<details>
<summary>정답 보기</summary>

| C# | TypeScript |
|----|------------|
| `int x = 10;` | `let x: number = 10;` |
| `string? name = null;` | `let name: string \| null = null;` |
| `public int Age { get; }` | `readonly age: number;` |
| `interface IUser` | `interface User` |
| `class User : IUser` | `class User implements User` |

</details>

---

## 숙제

### 숙제 1: Product 인터페이스 만들기

**문제**: 다음 조건을 만족하는 `Product` 인터페이스를 작성하세요.

**요구사항**:
- `id`: 숫자, 읽기 전용
- `name`: 문자열, 필수
- `price`: 숫자, 필수
- `description`: 문자열, 선택적
- `tags`: 문자열 배열, 선택적

```typescript
// 여기에 interface 작성


// 테스트
const product: Product = {
  id: 1,
  name: 'Laptop',
  price: 1500000
};
```

### 숙제 2: User 클래스 만들기

**문제**: 다음 조건을 만족하는 `User` 클래스를 작성하세요.

**요구사항**:
- 속성: `id` (읽기 전용), `name`, `email` (private)
- 생성자에서 모든 속성 초기화
- `getInfo()` 메서드: `"이름: {name}, ID: {id}"` 형식 반환
- `getEmail()` 메서드: email 반환

```typescript
// 여기에 class 작성


// 테스트
const user = new User(1, 'Kim', 'kim@test.com');
console.log(user.getInfo());  // "이름: Kim, ID: 1"
console.log(user.getEmail()); // "kim@test.com"
```

### 숙제 3: 제네릭 Repository 만들기

**문제**: C#의 Repository 패턴처럼, 제네릭 `Repository` 클래스를 만드세요.

**요구사항**:
- `items: T[]` 배열을 private으로 가짐
- `add(item: T): void` - 아이템 추가
- `getAll(): T[]` - 모든 아이템 반환
- `findById(id: number): T | undefined` - id로 찾기 (item.id와 비교)

```typescript
// 여기에 class 작성


// 테스트
interface Item {
  id: number;
  name: string;
}

const repo = new Repository<Item>();
repo.add({ id: 1, name: 'Item1' });
repo.add({ id: 2, name: 'Item2' });
console.log(repo.getAll());        // [{id:1,name:'Item1'}, {id:2,name:'Item2'}]
console.log(repo.findById(1));     // {id:1,name:'Item1'}
```

---

## 핵심 정리

| TypeScript | C# 대응 |
|------------|---------|
| `let x: number` | `int x` |
| `interface User` | `interface IUser` |
| `class User` | `class User` |
| `extends` | `:` (상속) |
| `implements` | `:` (구현) |
| `public/private` | 동일 |
| `x?: string` | 선택적 속성 |
| `x: string \| null` | `string?` (Nullable) |
| `<T>` | `<T>` (제네릭 동일) |
