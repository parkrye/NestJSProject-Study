# Day 4: Node.js와 npm 기초

## 학습 목표
- Node.js가 무엇이고 왜 필요한지 이해
- npm이 무엇이고 어디에 쓰는지 파악
- package.json의 역할 이해

---

## 1. Node.js란?

### 한 줄 정리
> **JavaScript를 컴퓨터에서 실행하게 해주는 프로그램**

### 왜 필요해?

원래 JavaScript는 **브라우저(크롬, 사파리) 안에서만** 실행할 수 있었습니다.

```
[과거]
JavaScript → 브라우저에서만 실행 가능 → 웹페이지 꾸미기만 가능

[Node.js 등장 후]
JavaScript → 컴퓨터에서 직접 실행 가능 → 서버도 만들 수 있음!
```

| 비유 | 설명 |
|------|------|
| 게임기 | JavaScript는 게임이고, Node.js는 게임을 실행하는 콘솔(플스, 닌텐도) |
| 앱 실행기 | 앱(JavaScript)을 실행하려면 운영체제(Node.js)가 필요 |

### Node.js로 할 수 있는 것
- **웹 서버 만들기** - 백엔드 개발
- **API 개발** - 앱/웹에 데이터 제공
- **CLI 도구** - 터미널 명령어 프로그램
- **파일 읽기/쓰기** - 컴퓨터 파일 조작

### C# 개발자를 위한 비교

| 개념 | C# | JavaScript |
|------|-----|------------|
| 코드 실행기 | .NET Runtime | Node.js |
| 역할 | C# 코드를 실행 | JavaScript 코드를 실행 |

```
C# 코드 → .NET Runtime → 실행
JS 코드 → Node.js      → 실행
```

---

## 2. npm이란?

### 한 줄 정리
> **다른 사람이 만든 코드를 가져다 쓰는 도구** (코드 앱스토어)

### 왜 필요해?

모든 기능을 직접 만들면 시간이 너무 오래 걸립니다.
누군가 이미 만들어둔 코드(패키지)를 가져다 쓰면 개발 시간을 크게 단축할 수 있습니다.

| 비유 | 설명 |
|------|------|
| 앱스토어 | 필요한 앱을 다운받듯이, 필요한 코드를 다운받음 |
| 레고 조각 | 남이 만든 레고 조각(패키지)을 내 작품에 조합 |
| 요리 재료 | 직접 농사 안 짓고, 마트에서 재료 사다 요리 |

### 어디에 사용해?

```bash
# 예시: 날짜를 다루는 패키지 설치
npm install dayjs

# 이제 코드에서 바로 사용 가능!
import dayjs from 'dayjs';
console.log(dayjs().format('YYYY-MM-DD'));
```

**자주 쓰는 패키지 예시:**
| 패키지 | 용도 |
|--------|------|
| `lodash` | 배열/객체 편하게 다루기 |
| `dayjs` | 날짜/시간 처리 |
| `axios` | HTTP 요청 보내기 |
| `uuid` | 고유 ID 생성 |

### C# 개발자를 위한 비교

| 개념 | C# | Node.js |
|------|-----|---------|
| 패키지 매니저 | NuGet | npm |
| 패키지 설치 | `dotnet add package` | `npm install` |
| 패키지 저장소 | nuget.org | npmjs.com |

---

## 3. package.json이란?

### 한 줄 정리
> **프로젝트 설명서** - "이 프로젝트는 이런 패키지들이 필요해요"

### 왜 필요해?

프로젝트를 다른 사람에게 공유할 때, "이 프로젝트 실행하려면 이런 패키지들이 필요해요"를 알려줘야 합니다.
`package.json`에 필요한 패키지 목록이 적혀있어서, `npm install` 한 번이면 전부 설치됩니다.

| 비유 | 설명 |
|------|------|
| 요리 레시피 | "이 요리에는 이런 재료가 필요해요" 목록 |
| 조립 설명서 | "이 제품 조립에는 이런 부품이 필요해요" |

### 구조 살펴보기

```json
{
  "name": "my-project",        // 프로젝트 이름
  "version": "1.0.0",          // 버전
  "scripts": {                 // 실행 명령어 모음
    "start": "node index.js",
    "dev": "node --watch index.js"
  },
  "dependencies": {            // 실행에 필요한 패키지
    "lodash": "^4.17.21"
  },
  "devDependencies": {         // 개발할 때만 필요한 패키지
    "typescript": "^5.0.0"
  }
}
```

### dependencies vs devDependencies

| 구분 | 설명 | 예시 |
|------|------|------|
| dependencies | 프로그램 **실행**에 필요 | lodash, axios |
| devDependencies | **개발**할 때만 필요 | typescript, eslint |

```bash
npm install lodash        # dependencies에 추가
npm install -D typescript # devDependencies에 추가 (-D 옵션)
```

### C# 개발자를 위한 비교

| 개념 | C# | Node.js |
|------|-----|---------|
| 프로젝트 설정 파일 | `.csproj` | `package.json` |
| 의존성 잠금 파일 | `packages.lock.json` | `package-lock.json` |
| 패키지 저장 폴더 | `packages/` | `node_modules/` |

---

## 4. import/export (모듈 시스템)

### 한 줄 정리
> **코드를 파일별로 나누고, 필요한 것만 가져다 쓰는 방법**

### 왜 필요해?

모든 코드를 한 파일에 쓰면 수천 줄이 되어 관리가 불가능합니다.
기능별로 파일을 나누고, 필요한 것만 가져다 씁니다.

```javascript
// math.js - 수학 함수 모음
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// index.js - 필요한 함수만 가져오기
import { add, subtract } from './math.js';
console.log(add(1, 2));  // 3
```

### 내보내기 (export) 방법

```javascript
// 방법 1: 개별 export
export const name = 'Kim';
export function greet() { }

// 방법 2: default export (파일당 하나만)
export default class Calculator { }
```

### 가져오기 (import) 방법

```javascript
// 개별 import (중괄호 필요)
import { add, subtract } from './math.js';

// default import (중괄호 없음)
import Calculator from './calculator.js';

// 전체 import
import * as MathUtils from './math.js';
MathUtils.add(1, 2);
```

### C# 개발자를 위한 비교

| 개념 | C# | JavaScript |
|------|-----|------------|
| 내보내기 | `public` | `export` |
| 가져오기 | `using` | `import` |
| 파일 단위 | namespace | 모듈 (파일) |

---

## 5. 명령어 정리 (참고용)

```bash
# 프로젝트 초기화
npm init -y

# 패키지 설치
npm install 패키지명       # 일반 의존성
npm install -D 패키지명    # 개발 의존성
npm install -g 패키지명    # 전역 설치

# 의존성 복원 (다른 사람 프로젝트 받았을 때)
npm install

# 스크립트 실행
npm start                 # start 스크립트 실행
npm run 스크립트명         # 다른 스크립트 실행

# 패키지 제거
npm uninstall 패키지명
```

---

## 연습

### 연습 1: 빈칸 채우기

```bash
# 1. 프로젝트 초기화
npm _______ -y

# 2. lodash 패키지 설치
npm _______ lodash

# 3. typescript를 개발 의존성으로 설치
npm install _______ typescript

# 4. 다른 사람 프로젝트 받았을 때 의존성 복원
npm _______
```

<details>
<summary>정답 보기</summary>

1. `init`
2. `install`
3. `-D`
4. `install`

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | Node.js는 브라우저에서 JavaScript를 실행하기 위한 것이다 | |
| 2 | npm은 다른 사람이 만든 코드를 가져다 쓸 수 있게 해준다 | |
| 3 | `package.json`은 프로젝트에 필요한 패키지 목록을 담고 있다 | |
| 4 | `dependencies`와 `devDependencies`는 같은 역할이다 | |

<details>
<summary>정답 보기</summary>

1. X (브라우저 밖, 컴퓨터에서 실행하기 위한 것)
2. O
3. O
4. X (dependencies는 실행용, devDependencies는 개발용)

</details>

### 연습 3: import/export 연습

```javascript
// utils.js - 이 파일에서 add와 multiply 함수를 export
export const add = (a, b) => a + b;
_______ const multiply = (a, b) => a * b;

// index.js - utils.js에서 두 함수를 import
_______ { add, multiply } from './utils.js';

console.log(add(2, 3));
console.log(multiply(2, 3));
```

<details>
<summary>정답 보기</summary>

```javascript
// utils.js
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// index.js
import { add, multiply } from './utils.js';
```

</details>

---

## 핵심 정리

| 용어 | 한 줄 설명 |
|------|-----------|
| Node.js | JavaScript를 컴퓨터에서 실행하게 해주는 프로그램 |
| npm | 다른 사람이 만든 코드(패키지)를 가져다 쓰는 도구 |
| package.json | 프로젝트 설정 + 필요한 패키지 목록 |
| dependencies | 프로그램 실행에 필요한 패키지 |
| devDependencies | 개발할 때만 필요한 패키지 |
| export | 코드를 다른 파일에서 쓸 수 있게 내보내기 |
| import | 다른 파일의 코드를 가져오기 |
| node_modules | 설치된 패키지들이 저장되는 폴더 |
