# Day 2: JavaScript 함수와 비동기

## 학습 목표
- 함수 선언 방식 이해
- 콜백 함수 개념 파악
- Promise와 async/await 기초 습득

---

## C# 개발자를 위한 핵심 차이점

> C#의 async/await를 알고 있다면 JavaScript도 거의 동일합니다!

| 개념 | C# | JavaScript |
|------|-----|------------|
| 일반 함수 | `void Method() {}` | `function fn() {}` |
| 람다/화살표 | `(x) => x * 2` | `(x) => x * 2` (동일!) |
| 비동기 함수 | `async Task Method()` | `async function fn()` |
| 대기 | `await task;` | `await promise;` |
| 비동기 반환 | `Task<T>` | `Promise<T>` |
| Action/Func | `Action<T>`, `Func<T,R>` | 그냥 함수 (타입 구분 없음) |

```csharp
// C#
Func<int, int, int> add = (a, b) => a + b;
async Task<User> GetUserAsync() {
    var user = await httpClient.GetAsync(...);
    return user;
}
```

```javascript
// JavaScript - 거의 동일!
const add = (a, b) => a + b;
async function getUser() {
    const user = await fetch(...);
    return user;
}
```

---

## 학습

### 1. 함수 선언 방식

```javascript
// 1. 함수 선언문 - C#의 메서드와 비슷
function add(a, b) {
  return a + b;
}

// 2. 함수 표현식 - C#의 Func 델리게이트와 비슷
const subtract = function(a, b) {
  return a - b;
};

// 3. 화살표 함수 - C#의 람다와 동일!
const multiply = (a, b) => {
  return a * b;
};

// 화살표 함수 축약형 (한 줄일 때)
// C#: Func<int,int,int> divide = (a, b) => a / b;
const divide = (a, b) => a / b;

// 매개변수가 하나일 때 괄호 생략 가능
// C#: Func<int,int> square = x => x * x;
const square = x => x * x;
```

**C# 람다 vs JavaScript 화살표 함수:**
```csharp
// C# - 타입 명시 필요
Func<int, int, int> add = (int a, int b) => a + b;
Action<string> log = msg => Console.WriteLine(msg);
```

```javascript
// JavaScript - 타입 없이 간결
const add = (a, b) => a + b;
const log = msg => console.log(msg);
```

### 2. 콜백 함수

**C#의 델리게이트/Action과 비슷한 개념:**

```csharp
// C# - 델리게이트로 콜백
void ProcessData(Action callback) {
    // 작업 후
    callback();
}
ProcessData(() => Console.WriteLine("완료!"));
```

```javascript
// JavaScript - 함수를 직접 전달
function processData(callback) {
  // 작업 후
  callback();
}
processData(() => console.log('완료!'));
```

**배열 메서드에서의 콜백 - C# LINQ와 유사:**
```javascript
const numbers = [1, 2, 3, 4, 5];

// C#: numbers.Select(n => n * 2)
const doubled = numbers.map(n => n * 2);

// C#: numbers.Where(n => n % 2 == 0)
const evens = numbers.filter(n => n % 2 === 0);

// C#: numbers.FirstOrDefault(n => n > 3)
const found = numbers.find(n => n > 3);
```

### 3. 비동기 처리가 필요한 이유

> C#에서 UI 스레드 블로킹 방지를 위해 async를 쓰는 것과 같은 이유!

```javascript
// JavaScript는 싱글 스레드
// C#처럼 Thread.Sleep()으로 막으면 전체가 멈춤!

console.log('1');
setTimeout(() => console.log('2'), 1000);  // 1초 후 실행
console.log('3');
// 출력: 1, 3, 2 (비동기!)
```

### 4. Promise (C#의 Task와 동일한 개념!)

> **핵심**: JavaScript의 `Promise` = C#의 `Task`

| C# Task | JavaScript Promise |
|---------|-------------------|
| `Task<T>` | `Promise<T>` |
| `Task.FromResult(value)` | `Promise.resolve(value)` |
| `Task.FromException(ex)` | `Promise.reject(error)` |
| `.Result` (블로킹) | 없음 (항상 비동기) |
| `.ContinueWith()` | `.then()` |
| `try-catch` | `.catch()` |

```csharp
// C# Task
Task<User> GetUserAsync(int id) {
    return Task.Run(() => {
        // 작업...
        return new User { Id = id };
    });
}

// 사용
GetUserAsync(1)
    .ContinueWith(task => Console.WriteLine(task.Result));
```

```javascript
// JavaScript Promise - 구조가 비슷!
function getUser(id) {
  return new Promise((resolve, reject) => {
    // 성공 시 resolve (C#의 Task 완료)
    // 실패 시 reject (C#의 Task 예외)
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, name: 'User' + id });
      } else {
        reject('Invalid ID');
      }
    }, 1000);
  });
}

// 사용 - .then()은 C#의 .ContinueWith()
getUser(1)
  .then(user => console.log(user))
  .catch(err => console.log(err));
```

### 5. async/await (C#과 거의 동일!)

> C#의 async/await를 안다면 이미 알고 있는 것입니다!

```csharp
// C#
public async Task<User> GetUserAsync() {
    var user = await userRepository.FindAsync(1);
    return user;
}

// 에러 처리
public async Task<User> GetUserSafeAsync() {
    try {
        var user = await userRepository.FindAsync(1);
        return user;
    } catch (Exception ex) {
        Console.WriteLine($"에러: {ex.Message}");
        return null;
    }
}
```

```javascript
// JavaScript - 거의 동일!
async function getUser() {
  const user = await userRepository.findOne(1);
  return user;
}

// 에러 처리 - C#과 동일!
async function getUserSafe() {
  try {
    const user = await userRepository.findOne(1);
    return user;
  } catch (error) {
    console.log(`에러: ${error.message}`);
    return null;
  }
}
```

**여러 비동기 작업:**
```csharp
// C# - Task.WhenAll
var results = await Task.WhenAll(
    GetUser1Async(),
    GetUser2Async()
);
```

```javascript
// JavaScript - Promise.all (동일 개념!)
const [user1, user2] = await Promise.all([
  getUser(1),
  getUser(2)
]);
```

---

## 연습

### 연습 1: 빈칸 채우기

```javascript
// 1. 화살표 함수로 두 수를 곱하는 함수 (한 줄)
const multiply = (a, b) _____ a * b;

// 2. async 함수 선언
_________ function fetchData() {
  const data = _______ fetch('/api/data');
  return data;
}

// 3. C#의 Task.WhenAll과 같은 것
const [a, b] = await Promise.______([taskA, taskB]);

// 4. Promise의 에러 처리 (then/catch 방식)
fetchData()
  .then(data => console.log(data))
  ._______(err => console.log(err));
```

<details>
<summary>정답 보기</summary>

1. `=>`
2. `async`, `await`
3. `all`
4. `catch`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | JavaScript의 `Promise`는 C#의 `Task`와 비슷한 역할을 한다 | |
| 2 | `async` 함수는 항상 `Promise`를 반환한다 | |
| 3 | `await`는 일반 함수에서도 사용할 수 있다 | |
| 4 | `Promise.all()`은 C#의 `Task.WhenAll()`과 같다 | |
| 5 | 화살표 함수에서 매개변수가 하나면 괄호를 생략할 수 있다 | |

<details>
<summary>정답 보기</summary>

1. O
2. O
3. X (async 함수 안에서만 사용 가능)
4. O
5. O

</details>

### 연습 3: 틀린 곳 찾기

```javascript
// 1번: async/await 사용
function fetchUser() {
  const user = await fetch('/api/user');
  return user;
}

// 2번: 화살표 함수
const add = (a, b) {
  return a + b;
}

// 3번: Promise 에러 처리
fetchData()
  .then(data => console.log(data))
  .error(err => console.log(err));
```

<details>
<summary>정답 보기</summary>

```javascript
// 1번: async 키워드 누락
async function fetchUser() {
  const user = await fetch('/api/user');
  return user;
}

// 2번: => 누락
const add = (a, b) => {
  return a + b;
}

// 3번: .error가 아니라 .catch
fetchData()
  .then(data => console.log(data))
  .catch(err => console.log(err));
```

</details>

### 연습 4: C# → JavaScript 변환

| C# | JavaScript |
|----|------------|
| `async Task<User> GetUser()` | `_______ function getUser()` |
| `await task;` | `_______ promise;` |
| `Task.WhenAll(t1, t2)` | `Promise._______(p1, p2)` |
| `Task.FromResult(value)` | `Promise._______(value)` |
| `Func<int, int> fn = x => x * 2;` | `const fn = x _____ x * 2;` |

<details>
<summary>정답 보기</summary>

| C# | JavaScript |
|----|------------|
| `async Task<User> GetUser()` | `async function getUser()` |
| `await task;` | `await promise;` |
| `Task.WhenAll(t1, t2)` | `Promise.all(p1, p2)` |
| `Task.FromResult(value)` | `Promise.resolve(value)` |
| `Func<int, int> fn = x => x * 2;` | `const fn = x => x * 2;` |

</details>

---

## 숙제

### 숙제 1: 지연 실행 함수

**문제**: 주어진 시간(ms) 후에 메시지를 출력하는 `delay` 함수를 만드세요.

**요구사항**:
- Promise를 반환해야 함
- async/await로 사용 가능해야 함

```javascript
// 여기에 delay 함수 작성
function delay(ms) {
  // Promise를 반환하세요
}

// 테스트
async function test() {
  console.log('시작');
  await delay(2000);  // 2초 대기
  console.log('2초 후');
}
test();
```

### 숙제 2: 순차 실행 vs 병렬 실행

**문제**: 아래 두 방식의 차이를 이해하고, 각각 실행해보세요.

```javascript
// 가짜 API 호출 함수 (1초 소요)
function fetchUser(id) {
  return new Promise(resolve => {
    setTimeout(() => resolve({ id, name: `User${id}` }), 1000);
  });
}

// 방법 1: 순차 실행 (3초 소요) - 완성하세요
async function sequential() {
  console.time('순차');
  // user1, user2, user3를 순차적으로 가져오세요

  console.timeEnd('순차');
}

// 방법 2: 병렬 실행 (1초 소요) - 완성하세요
async function parallel() {
  console.time('병렬');
  // Promise.all을 사용해서 user1, user2, user3를 동시에 가져오세요

  console.timeEnd('병렬');
}
```

### 숙제 3: 배열 메서드 체이닝

**문제**: 다음 조건을 만족하는 코드를 작성하세요.

**요구사항**:
1. 숫자 배열 `[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]`에서
2. 짝수만 필터링 (filter)
3. 각 숫자를 2배로 (map)
4. 모든 숫자의 합 구하기 (reduce)

**힌트**: C# LINQ 체이닝과 동일한 방식입니다.

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const result = numbers
  // 여기에 체이닝 작성

console.log(result);  // 60 (2+4+6+8+10을 각각 2배: 4+8+12+16+20 = 60)
```

---

## 핵심 정리

| JavaScript | C# 대응 |
|------------|---------|
| `(a, b) => a + b` | 람다 표현식 (동일!) |
| 콜백 함수 | Action, Func 델리게이트 |
| `Promise` | `Task` |
| `Promise.resolve()` | `Task.FromResult()` |
| `Promise.all()` | `Task.WhenAll()` |
| `.then()` | `.ContinueWith()` |
| `async function` | `async Task Method()` |
| `await` | `await` (동일!) |
| `try/catch` | `try/catch` (동일!) |
