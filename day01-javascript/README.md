# Day 1: JavaScript 기초

## 학습 목표
- 변수 선언 방식의 차이 이해
- 기본 데이터 타입 파악
- 조건문과 반복문 사용법 익히기

---

## C# 개발자를 위한 핵심 차이점

> C#을 알고 있다면 JavaScript는 빠르게 익힐 수 있습니다. 주요 차이점만 파악하세요.

| 개념 | C# | JavaScript |
|------|-----|------------|
| 타입 시스템 | 정적 타입 (컴파일 시 체크) | 동적 타입 (런타임에 결정) |
| 변수 선언 | `int x = 1;` `var x = 1;` | `let x = 1;` `const x = 1;` |
| 상수 | `const int X = 1;` | `const x = 1;` |
| 문자열 보간 | `$"Hello {name}"` | `` `Hello ${name}` `` (백틱) |
| 배열 | `int[] arr = {1,2,3};` | `const arr = [1,2,3];` |
| 딕셔너리/객체 | `Dictionary<K,V>` | `{ key: value }` (객체 리터럴) |
| null 체크 | `x?.Property` | `x?.property` (동일!) |
| 세미콜론 | 필수 | 선택 (권장) |

```csharp
// C#
var user = new { Name = "Kim", Age = 25 };
Console.WriteLine($"이름: {user.Name}");
```

```javascript
// JavaScript - 거의 비슷!
const user = { name: 'Kim', age: 25 };
console.log(`이름: ${user.name}`);
```

---

## 학습

### 1. 변수 선언

JavaScript에서 변수를 선언하는 3가지 방법:

| 키워드 | C# 비교 | 재할당 | 스코프 | 사용 권장 |
|--------|---------|--------|--------|-----------|
| `var` | ❌ (C# var와 다름!) | O | 함수 | X (구식) |
| `let` | C#의 일반 변수 | O | 블록 | O (변하는 값) |
| `const` | C#의 `readonly` 비슷 | X | 블록 | O (상수) |

> **주의**: JavaScript의 `var`는 C#의 `var`와 완전히 다릅니다!
> - C# `var`: 타입 추론 (컴파일 시 타입 결정)
> - JS `var`: 함수 스코프 변수 (호이스팅 문제)

```javascript
// JavaScript
let changeable = '변경 가능';  // C#: string changeable = "변경 가능";
const fixed = '고정값';        // C#: readonly와 비슷

changeable = '새 값';  // OK
// fixed = '에러';     // Error! const는 재할당 불가
```

### 2. 데이터 타입

**C#과의 비교:**

| C# | JavaScript | 비고 |
|----|------------|------|
| `int`, `double`, `float` | `number` | JS는 숫자 타입 하나! |
| `string` | `string` | 동일 |
| `bool` | `boolean` | 이름만 다름 |
| `null` | `null` | 동일 |
| `class`, `struct` | `object` | JS 객체는 더 유연 |
| `List<T>`, `T[]` | `Array` | JS 배열은 타입 제한 없음 |

```javascript
// JavaScript - 타입 선언 없이 사용
const name = 'Kim';       // string
const age = 25;           // number (int/double 구분 없음)
const price = 19.99;      // number
const isActive = true;    // boolean
const empty = null;       // null
let notAssigned;          // undefined (C#의 default와 비슷)
```

**배열 - C# List처럼 유연함:**
```javascript
// C#: List<string> fruits = new List<string> { "apple", "banana" };
const fruits = ['apple', 'banana', 'orange'];
console.log(fruits[0]);     // 'apple'
console.log(fruits.length); // 3 (C#의 .Count와 비슷)

// C#과 달리 타입 혼합 가능!
const mixed = [1, 'two', true, { name: 'Kim' }];
```

**객체 - C# 익명 타입/Dictionary 비슷:**
```javascript
// C#: var user = new { Name = "Kim", Age = 25 };
const user = {
  name: 'Kim',    // C#처럼 PascalCase 아닌 camelCase 사용
  age: 25,
  isAdmin: false
};

console.log(user.name);     // 'Kim' - 점 표기법
console.log(user['age']);   // 25 - 대괄호 표기법 (Dictionary처럼)
```

### 3. 조건문

C#과 거의 동일합니다:

```javascript
// if-else - C#과 동일!
const score = 85;

if (score >= 90) {
  console.log('A');
} else if (score >= 80) {
  console.log('B');
} else {
  console.log('F');
}

// 삼항 연산자 - C#과 동일!
const result = score >= 60 ? '합격' : '불합격';

// switch - C#과 동일!
const day = 'monday';
switch (day) {
  case 'monday':
    console.log('월요일');
    break;
  default:
    console.log('기타');
}
```

### 4. 반복문

```javascript
// for문 - C#과 동일!
for (let i = 0; i < 5; i++) {
  console.log(i);
}

// while - C#과 동일!
let count = 0;
while (count < 3) {
  console.log(count);
  count++;
}

// foreach 대체 - for...of
// C#: foreach (var color in colors)
const colors = ['red', 'green', 'blue'];
for (const color of colors) {
  console.log(color);
}

// 배열 메서드 - C# LINQ와 비슷!
// C#: colors.ForEach(c => Console.WriteLine(c));
colors.forEach((color, index) => {
  console.log(`${index}: ${color}`);
});
```

### 5. C# LINQ vs JavaScript 배열 메서드

C#의 LINQ를 알고 있다면 JavaScript 배열 메서드는 쉽습니다:

| C# LINQ | JavaScript | 설명 |
|---------|------------|------|
| `.Select(x => ...)` | `.map(x => ...)` | 변환 |
| `.Where(x => ...)` | `.filter(x => ...)` | 필터링 |
| `.FirstOrDefault(x => ...)` | `.find(x => ...)` | 첫 번째 찾기 |
| `.Any(x => ...)` | `.some(x => ...)` | 하나라도 만족? |
| `.All(x => ...)` | `.every(x => ...)` | 모두 만족? |
| `.Sum()` | `.reduce((a,b) => a+b)` | 합계 |
| `.OrderBy(x => ...)` | `.sort((a,b) => ...)` | 정렬 |

```csharp
// C# LINQ
var doubled = numbers.Select(n => n * 2).ToList();
var evens = numbers.Where(n => n % 2 == 0).ToList();
var found = numbers.FirstOrDefault(n => n > 3);
```

```javascript
// JavaScript - 거의 동일!
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);  // === 사용 주의!
const found = numbers.find(n => n > 3);
```

> **주의**: JavaScript는 `==` 대신 `===` (엄격한 비교) 사용을 권장합니다.
> - `==`: 타입 변환 후 비교 (`'1' == 1` → true)
> - `===`: 타입까지 비교 (`'1' === 1` → false)

---

## 연습

### 연습 1: 빈칸 채우기

다음 코드의 빈칸을 채우세요.

```javascript
// 1. 변경 불가능한 상수를 선언하려면?
_______ PI = 3.14;

// 2. 변경 가능한 변수를 선언하려면?
_______ count = 0;

// 3. C#의 $"Hello {name}"과 같은 문자열 보간은?
const name = 'Kim';
console.log(______Hello ${name}______);  // 어떤 기호로 감싸야 할까요?

// 4. 배열의 길이를 구하려면? (C#의 .Count와 비슷)
const arr = [1, 2, 3];
console.log(arr._______);
```

<details>
<summary>정답 보기</summary>

1. `const`
2. `let`
3. 백틱 `` ` ``으로 감싸기: `` `Hello ${name}` ``
4. `length`

</details>

### 연습 2: O/X 퀴즈

다음 설명이 맞으면 O, 틀리면 X를 선택하세요.

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | JavaScript의 `var`는 C#의 `var`와 같은 역할을 한다 | |
| 2 | `const`로 선언한 변수는 값을 변경할 수 없다 | |
| 3 | JavaScript에서 `int`와 `double`은 모두 `number` 타입이다 | |
| 4 | `===`는 타입까지 비교하는 엄격한 비교 연산자이다 | |
| 5 | JavaScript 배열은 서로 다른 타입의 요소를 가질 수 있다 | |

<details>
<summary>정답 보기</summary>

1. X (JS var는 함수 스코프, C# var는 타입 추론)
2. O
3. O
4. O
5. O

</details>

### 연습 3: 틀린 곳 찾기

다음 코드에서 틀린 부분을 찾아 수정하세요.

```javascript
// 1번 코드
const age = 25;
age = 26;  // 나이가 바뀌었습니다

// 2번 코드
const user = { name: 'Kim', age: 25 };
console.log("이름: ${user.name}");  // 문자열 보간 사용

// 3번 코드
const numbers = [1, 2, 3];
console.log(numbers.Count);  // 배열 길이 출력
```

<details>
<summary>정답 보기</summary>

```javascript
// 1번: const는 재할당 불가 → let 사용
let age = 25;
age = 26;

// 2번: 문자열 보간은 백틱(`) 사용
console.log(`이름: ${user.name}`);

// 3번: JavaScript는 .length 사용 (.Count는 C#)
console.log(numbers.length);
```

</details>

### 연습 4: C# → JavaScript 변환

C# 코드를 JavaScript로 변환하세요.

| C# | JavaScript |
|----|------------|
| `numbers.Select(n => n * 2)` | `numbers._______(n => n * 2)` |
| `numbers.Where(n => n > 5)` | `numbers._______(n => n > 5)` |
| `numbers.FirstOrDefault(n => n > 10)` | `numbers._______(n => n > 10)` |
| `foreach (var item in items)` | `for (const item ___ items)` |

<details>
<summary>정답 보기</summary>

| C# | JavaScript |
|----|------------|
| `numbers.Select(n => n * 2)` | `numbers.map(n => n * 2)` |
| `numbers.Where(n => n > 5)` | `numbers.filter(n => n > 5)` |
| `numbers.FirstOrDefault(n => n > 10)` | `numbers.find(n => n > 10)` |
| `foreach (var item in items)` | `for (const item of items)` |

</details>

---

## 숙제

### 숙제 1: 구구단 출력기

**문제**: 2단부터 9단까지 구구단을 출력하는 코드를 작성하세요.

**요구사항**:
- 중첩 for문 사용
- 출력 형식: `2 x 1 = 2`

**힌트**: C#과 거의 동일합니다!

```javascript
// 여기에 코드 작성


```

### 숙제 2: 점수 등급 계산기

**문제**: 점수 배열을 받아서 각 점수의 등급을 반환하는 코드를 작성하세요.

**요구사항**:
- 90 이상: 'A', 80 이상: 'B', 70 이상: 'C', 60 이상: 'D', 그 외: 'F'
- 입력: `[95, 82, 76, 55, 88]`
- 출력: `['A', 'B', 'C', 'F', 'B']`

**힌트**: `.map()`과 조건문을 조합하세요.

```javascript
const scores = [95, 82, 76, 55, 88];

// 여기에 코드 작성
const grades =

console.log(grades);  // ['A', 'B', 'C', 'F', 'B']
```

### 숙제 3: 사용자 필터링

**문제**: 사용자 목록에서 성인(18세 이상)만 필터링하고, 이름만 추출하세요.

**요구사항**:
- `.filter()`와 `.map()` 사용
- 입력과 출력은 아래 참고

```javascript
const users = [
  { name: 'Kim', age: 25 },
  { name: 'Lee', age: 17 },
  { name: 'Park', age: 30 },
  { name: 'Choi', age: 15 }
];

// 여기에 코드 작성
// 성인만 필터링 후 이름만 추출
const adultNames =

console.log(adultNames);  // ['Kim', 'Park']
```

---

## 핵심 정리

| JavaScript | C# 대응 |
|------------|---------|
| `const`/`let` | 변수 선언 |
| `number` | int, double, float 통합 |
| `[]` 배열 | List<T>, 배열 |
| `{}` 객체 | 익명 타입, Dictionary |
| `for...of` | foreach |
| `.map()` | .Select() |
| `.filter()` | .Where() |
| `===` | == (타입까지 비교) |
| `` `${var}` `` | $"{var}" |
