# Day 4: Node.js와 npm 기초

## 학습 목표
- Node.js의 개념과 역할 이해
- npm 패키지 매니저 사용법 습득
- 모듈 시스템 기초 파악

---

## C# 개발자를 위한 핵심 차이점

> **Node.js**는 .NET Runtime과 비슷한 역할을 합니다.
> **npm**은 NuGet과 동일한 역할을 합니다.

| 개념 | C# / .NET | Node.js / npm |
|------|-----------|---------------|
| 런타임 | .NET Runtime | Node.js |
| 패키지 매니저 | NuGet | npm |
| 패키지 설정 파일 | `.csproj` | `package.json` |
| 패키지 잠금 파일 | `packages.lock.json` | `package-lock.json` |
| 패키지 저장 폴더 | `packages/` 또는 글로벌 캐시 | `node_modules/` |
| 프로젝트 생성 | `dotnet new` | `npm init` |
| 패키지 설치 | `dotnet add package` | `npm install` |
| 실행 | `dotnet run` | `node app.js` 또는 `npm start` |

```bash
# C# / .NET
dotnet new console
dotnet add package Newtonsoft.Json
dotnet run

# Node.js / npm - 비슷한 흐름!
npm init -y
npm install lodash
node index.js
```

---

## 공부

### 1. Node.js란?

> **.NET이 C#을 실행**하듯, **Node.js가 JavaScript를 실행**합니다.

```
[C#]
C# 코드 → .NET Runtime → 실행

[JavaScript]
JS 코드 → Node.js → 실행
```

**Node.js로 할 수 있는 것** (C#으로 할 수 있는 것과 동일):
- 웹 서버 구축 (ASP.NET Core ↔ Express/NestJS)
- API 개발
- CLI 도구 개발
- 파일 시스템 접근

### 2. npm (Node Package Manager)

> **NuGet의 JavaScript 버전**입니다!

```bash
# C# - NuGet
dotnet add package Newtonsoft.Json

# JavaScript - npm (동일한 개념!)
npm install lodash
```

### 3. package.json (= .csproj)

> **프로젝트 설정 파일**입니다. C#의 `.csproj`와 같은 역할!

```bash
# 프로젝트 초기화
npm init -y   # dotnet new와 비슷
```

**C# .csproj vs package.json 비교:**

```xml
<!-- C# .csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.1" />
  </ItemGroup>
</Project>
```

```json
// package.json - 같은 역할, JSON 형식
{
  "name": "my-project",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "build": "tsc"
  },
  "dependencies": {
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

### 4. npm 명령어 - NuGet과 비교

| 작업 | C# (dotnet/NuGet) | Node.js (npm) |
|------|-------------------|---------------|
| 프로젝트 생성 | `dotnet new console` | `npm init -y` |
| 패키지 설치 | `dotnet add package 패키지명` | `npm install 패키지명` |
| 개발용 패키지 | 없음 (구분 안함) | `npm install -D 패키지명` |
| 전역 설치 | `dotnet tool install -g` | `npm install -g` |
| 패키지 제거 | `dotnet remove package` | `npm uninstall` |
| 의존성 복원 | `dotnet restore` | `npm install` |
| 실행 | `dotnet run` | `npm start` |

```bash
# 패키지 설치
npm install lodash        # 프로덕션 의존성
npm install -D typescript # 개발 의존성 (-D = --save-dev)
npm install -g @nestjs/cli # 전역 설치

# 모든 의존성 설치 (다른 사람 프로젝트 받았을 때)
npm install   # dotnet restore와 같음
```

**dependencies vs devDependencies (C#에는 없는 개념):**
| 구분 | 설명 | C# 비교 |
|------|------|---------|
| dependencies | 실행에 필요 | 일반 PackageReference |
| devDependencies | 개발/빌드에만 필요 | 빌드 시에만 쓰는 도구 |

### 5. 모듈 시스템 - C# using과 비교

**C# namespace/using:**
```csharp
// MathHelper.cs
namespace MyProject.Helpers {
    public static class MathHelper {
        public static int Add(int a, int b) => a + b;
    }
}

// Program.cs
using MyProject.Helpers;
Console.WriteLine(MathHelper.Add(1, 2));
```

**JavaScript ES Modules (현재 표준):**
```javascript
// math.js - export로 내보내기
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// 또는 default export (클래스 하나만 내보낼 때)
export default class Calculator {
  add(a, b) { return a + b; }
}

// index.js - import로 가져오기
import { add, subtract } from './math.js';      // C#: using static
import Calculator from './calculator.js';        // default import
import * as MathHelper from './math.js';         // C#: using 네임스페이스

console.log(add(1, 2));
console.log(MathHelper.subtract(5, 3));
```

**ES Modules 사용 설정:**
```json
// package.json에 추가
{
  "type": "module"
}
```

### 6. 유용한 내장 모듈 - C# BCL과 비교

| 용도 | C# | Node.js |
|------|-----|---------|
| 파일 시스템 | `System.IO` | `fs` |
| 경로 | `System.IO.Path` | `path` |
| HTTP | `HttpClient` | `http` / `fetch` |
| JSON | `System.Text.Json` | 내장 (JSON.parse) |

```javascript
// C#: using System.IO;
import fs from 'fs';
const data = fs.readFileSync('file.txt', 'utf-8');

// C#: Path.Combine(dir, "folder", "file.txt")
import path from 'path';
const fullPath = path.join(__dirname, 'folder', 'file.txt');
```

---

## scripts - C#의 dotnet 명령어 커스터마이징

```json
// package.json
{
  "scripts": {
    "start": "node index.js",      // npm start → node index.js
    "dev": "node --watch index.js", // npm run dev → 파일 변경 감지
    "build": "tsc",                 // npm run build → TypeScript 컴파일
    "test": "jest"                  // npm test → 테스트 실행
  }
}
```

```bash
npm start       # "start"는 run 생략 가능
npm run dev     # 다른 스크립트는 run 필요
npm test        # "test"도 run 생략 가능
npm run build
```

---

## 연습

### 연습 1: 빈칸 채우기

```bash
# 1. 프로젝트 초기화 (C#의 dotnet new console과 비슷)
npm _______ -y

# 2. 패키지 설치 (C#의 dotnet add package lodash)
npm _______ lodash

# 3. 개발용 패키지 설치 (--save-dev)
npm install _______ typescript

# 4. 의존성 복원 (C#의 dotnet restore)
npm _______

# 5. 스크립트 실행
npm run _______
```

<details>
<summary>정답 보기</summary>

1. `init`
2. `install`
3. `-D` 또는 `--save-dev`
4. `install`
5. `start` (또는 정의된 스크립트 이름)

</details>

### 연습 2: O/X 퀴즈

| 번호 | 설명 | O/X |
|------|------|-----|
| 1 | Node.js는 브라우저에서 JavaScript를 실행하기 위한 환경이다 | |
| 2 | npm은 C#의 NuGet과 비슷한 역할을 한다 | |
| 3 | `package.json`은 C#의 `.csproj`와 비슷한 역할을 한다 | |
| 4 | `dependencies`와 `devDependencies`는 동일한 역할을 한다 | |
| 5 | `npm install`은 `dotnet restore`와 비슷한 역할을 한다 | |

<details>
<summary>정답 보기</summary>

1. X (Node.js는 브라우저 밖에서 JavaScript를 실행하기 위한 런타임)
2. O
3. O
4. X (dependencies는 실행에 필요, devDependencies는 개발에만 필요)
5. O

</details>

### 연습 3: 틀린 곳 찾기

```javascript
// 1번
const fs = require 'fs';

// 2번
export function add(a, b) {
  return a + b;
}
// 위 모듈을 가져올 때
import add from './math.js';

// 3번 - package.json
{
  "name": "my-project",
  "dependencies": {
    typescript: "^5.0.0"
  }
}
```

<details>
<summary>정답 보기</summary>

```javascript
// 1번: require 다음에 괄호가 필요
const fs = require('fs');

// 2번: default export가 아닌 named export이므로 중괄호 필요
import { add } from './math.js';
// 또는 export default로 변경

// 3번: 패키지명에 따옴표 필요
{
  "name": "my-project",
  "dependencies": {
    "typescript": "^5.0.0"
  }
}
```

</details>

### 연습 4: C# → Node.js 변환

| C# / .NET | Node.js / npm |
|-----------|---------------|
| `dotnet new console` | `npm _______ -y` |
| `dotnet add package lodash` | `npm _______ lodash` |
| `dotnet run` | `npm _______` 또는 `node index.js` |
| `dotnet restore` | `npm _______` |
| `using System.IO;` | `import _______ from 'fs';` |

<details>
<summary>정답 보기</summary>

| C# / .NET | Node.js / npm |
|-----------|---------------|
| `dotnet new console` | `npm init -y` |
| `dotnet add package lodash` | `npm install lodash` |
| `dotnet run` | `npm start` 또는 `node index.js` |
| `dotnet restore` | `npm install` |
| `using System.IO;` | `import fs from 'fs';` |

</details>

---

## 숙제

### 숙제 1: npm 프로젝트 생성 및 설정

**문제**: 새 Node.js 프로젝트를 만들고 package.json을 설정하세요.

**요구사항**:
- `my-node-project` 폴더 생성
- npm 초기화
- `lodash` 패키지 설치
- `typescript`를 개발 의존성으로 설치

```bash
# 여기에 명령어 작성 (4줄)




```

### 숙제 2: 모듈 만들기

**문제**: C#의 클래스 분리처럼 모듈을 만들어 사용하세요.

**요구사항**:
- `math.js`: `add`, `subtract`, `multiply` 함수를 export
- `index.js`: math.js를 import해서 사용

**힌트**: ES Modules 사용 (`export`, `import`)

```javascript
// math.js - 여기에 코드 작성


// index.js - 여기에 코드 작성


// 실행 결과: 각 연산 결과 출력
```

### 숙제 3: package.json scripts 작성

**문제**: 다음 명령어들을 package.json의 scripts에 정의하세요.

**요구사항**:
- `start`: `node index.js` 실행
- `dev`: `node --watch index.js` 실행 (파일 변경 감지)
- `test`: `echo "테스트 실행"` 출력

```json
// package.json의 scripts 부분 작성
{
  "scripts": {

  }
}
```

---

## 핵심 정리

| Node.js / npm | C# / .NET 대응 |
|---------------|----------------|
| Node.js | .NET Runtime |
| npm | NuGet |
| `package.json` | `.csproj` |
| `node_modules/` | 패키지 캐시 |
| `npm init` | `dotnet new` |
| `npm install` | `dotnet add package` |
| `npm start` | `dotnet run` |
| `import/export` | `using/namespace` |
| `dependencies` | PackageReference |
| `devDependencies` | 빌드 도구 (구분 없음) |
