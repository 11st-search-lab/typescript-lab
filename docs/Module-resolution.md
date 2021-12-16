# Module Resolution

> https://typescript-kr.github.io/pages/module-resolution.html

**모듈 해석 (module resolution) 은 컴파일러가 import가 무엇을 참조하는지 알아내기 위해 사용하는 프로세스이다.**

`import { a } from "moduleA"`같은 import 문을 생각해보자. `a`의 모든 사용을 검사하기 위해, 컴파일러는 무엇을 참조하는지 정확히 알아야 할 필요가 있다. 그리고 moduleA 정의를 검사해야 할 필요가 있다. 이 시점에, 컴파일러는 "`moduleA`의 형태가 뭘까?"라고 질문할 것이다.

**간단해 보이지만, `moduleA`는 `.ts/.tsx` 파일에 정의되어 있거나 혹은 코드가 의존하는 `.d.ts`에 정의되어 있을 수 있다.**

첫 번째로, 컴파일러는 가져온 모듈을 나타내는 파일의 위치를 찾으려고 할 것이다. 

- 1. 클래식 방식으로 찾음
- 2. 노드 방식으로 찾음
- 3. 모듈 이름이 비-상대적이라면 컴파일러는 `ambient 모듈 선언`을 찾음
- 4. 컴파일러가 모듈을 해석할 수 없다면, 오류 로그가 발생

> `index.d.ts`의 기본 선언 방식을 `ambient declarations`라고 한다.

## 상대적 vs. 비-상대적 모듈 import (Relative vs. Non-relative module imports)


모듈 참조가 상대적 혹은 비-상대적이냐에 따라 모듈 import는 다르게 해석된다.

상대적 import 는 `/`, `./` 혹은 `../.` 중에 하나로 시작한다.

- `import Entry from "./components/Entry";`
- `import { DefaultHeaders } from "../constants/http";`
- `import "/mod";`

다른 모든 import는 비-상대적 으로 간주된다.

- `import * as $ from "jquery";`
- `import { Component } from "@angular/core";`

상대적 import는 가져온 파일에 상대적으로 해석되고 **`ambient 모듈 선언`으로 해석 될 수 없다.**

비-상대적 import는 baseUrl로 해석되거나, 경로 매핑으로 해석될 수 있다.

## 모듈 해석 전략 (Module Resolution Strategies)

두 가지 가능한 모듈 해석 전략이 있다. (노드, 클래식) `--moduleResolution` 플래그를 사용하여 모듈 해석 전략을 지정할 수 있다. 디폴트는 `--module AMD | System | ES2015`에서는 클래식이고, 나머지는 노드이다.

## 클래식 (Classic)

TypeScript의 디폴트 해석 전략으로 사용된다.

**상대적 import**는 import하는 파일에 상대적으로 해석된다. 그래서 소스 파일 `/root/src/folder/A.ts`안에 `import { b } from "./moduleB"`는 다음과 같이 조회한다:

1. `/root/src/folder/moduleB.ts`
2. `/root/src/folder/moduleB.d.ts`

**비-상대적 모듈 import**에서는, 컴파일러가 가져온 파일을 갖고 있는 디렉터리부터 시작해서 디렉터리 트리를 거슬러 올라가 맞는 정의 파일의 위치를 찾으려고 한다.

예를 들어:

소스 파일 `/root/src/folder/A.ts`안에 `import { b } from "moduleB"`처럼 moduleB의 비-상대적 import은 "moduleB"의 위치를 찾기 위해 다음과 같은 위치를 찾는다.

- `/root/src/folder/moduleB.ts`
- `/root/src/folder/moduleB.d.ts`
- `/root/src/moduleB.ts`
- `/root/src/moduleB.d.ts`
- `/root/moduleB.ts`
- `/root/moduleB.d.ts`
- `/moduleB.ts`
- `/moduleB.d.ts`

## 노드 (Node)

이 해석 전략은 런타임에 Node.js의 모듈 해석 메커니즘을 모방하려고 시도한다. 전체 Node.js 해석 알고리즘은 [Node.js 모듈 문서](https://nodejs.org/api/modules.html#modules_all_together)에 요약되어 있다.

### `Node.js`가 모듈을 해석하는 방법 (How Node.js resolves modules)

TS 컴파일러가 어떤 과정을 따를지 이해하기 위해서는, Node.js 모듈을 이해하는 것이 중요하다. 전통적으로, `Node.js`의 `import`는 `require 함수`를 호출해 수행한다. **`Node.js`의 동작은 `require`에 상대적 경로 혹은 비-상대적 경로가 주어지는지에 따라 달라진다.**

### 상대적 경로

상대적 경로는 아주 간단하다. `var x = require("./moduleB");`라는 import 문을 포함한 `/root/src/moduleA.js`에 위치한 파일을 생각해보자.

1. `/root/src/moduleB.js`라는 파일이 존재하는지 확인.
2. 만약 `"main"` 모듈을 지정하는 `package.json`라는 파일을 포함하고 있으면, `/root/src/moduleB` 폴더 확인하기. 
  ```json
  // /root/src/moduleB/package.json
  { "main": "lib/mainModule.js" } 
  // 해당 package.json을 찾으면 ==> /root/src/moduleB/lib/mainModule.js 를 참조한다.
  ```
3. `index.js` 라는 파일을 포함하고 있으면, `/root/src/moduleB` 확인하기. (이 파일은 폴더의 "main" 모듈임을 암시적으로 나타낸다.)

### 비-상대적 경로

비-상대적 모듈 이름에 대한 해석은 다르게 수행한다. Node는 디렉터리 체인을 올라가, 로드하려는 모듈을 찾을 때까지 각 node_modules을 찾는다.

`/root/src/moduleA.js`가 대신 비-상대적 경로를 사용하고 `var x = require("moduleB");` import를 가지고 있다고 하자.

Node는 하나가 일치할 때까지 각 위치에서 moduleB를 해석하려고 시도한다.

- 1. `/root/src/node_modules/moduleB.js`
- 2. `/root/src/node_modules/moduleB/package.json ("main" 항목을 지정했다면)`
- 3. `/root/src/node_modules/moduleB/index.js`

- 4. `/root/node_modules/moduleB.js`
- 5. `/root/node_modules/moduleB/package.json ("main" 항목을 지정했다면)`
- 6. `/root/node_modules/moduleB/index.js`

- 7. `/node_modules/moduleB.js`
- 8. `/node_modules/moduleB/package.json ("main" 항목을 지정했다면)`
- 9. `/node_modules/moduleB/index.js`

## TypeScript가 모듈을 해석하는 방법 (How TypeScript resolves modules)

`TypeScript`는 컴파일-타임에 모듈의 정의 파일 위치를 찾기 위해 `Node.js`의 런타임 해석 전략을 모방한다. 이를 달성하기 위해, `TypeScript`는 `TypeScript` 소스 파일 확장자 (`.ts, .tsx 와 .d.ts)`를 Node의 해석 로직 위에 씌운다. `Typescript`는 `"main"`의 목적을 반영하기 위해 `"types"`라는 필드를 `package.json`에서 사용한다.

`/root/src/moduleA.ts`안에 `import { b } from "./moduleB"` 같은 import 문은 `"./moduleB"`의 위치를 찾기 위해 다음과 같이 위치를 찾는다.

- `/root/src/moduleB.ts`
- `/root/src/moduleB.tsx`
- `/root/src/moduleB.d.ts`
- `/root/src/moduleB/package.json ("types" 항목을 지정했다면)`
- `/root/src/moduleB/index.ts`
- `/root/src/moduleB/index.tsx`
- `/root/src/moduleB/index.d.ts`

비슷하게, 비-상대적 import는 Node.js 해석 로직을 따른다.

- `/root/src/node_modules/moduleB.ts`
- `/root/src/node_modules/moduleB.tsx`
- `/root/src/node_modules/moduleB.d.ts`
- `/root/src/node_modules/moduleB/package.json ("types" 프로퍼티를 지정했다면)`
- `/root/src/node_modules/@types/moduleB.d.ts`
- `/root/src/node_modules/moduleB/index.ts`
- `/root/src/node_modules/moduleB/index.tsx`
- `/root/src/node_modules/moduleB/index.d.ts`

- `/root/node_modules/moduleB.ts`
- `/root/node_modules/moduleB.tsx`
- `/root/node_modules/moduleB.d.ts`
- `/root/node_modules/moduleB/package.json ("types" 항목을 지정했다면)`
- `/root/node_modules/@types/moduleB.d.ts`
- `/root/node_modules/moduleB/index.ts`
- `/root/node_modules/moduleB/index.tsx`
- `/root/node_modules/moduleB/index.d.ts`

- `/node_modules/moduleB.ts`
- `/node_modules/moduleB.tsx`
- `/node_modules/moduleB.d.ts`
- `/node_modules/moduleB/package.json ("types" 항목을 지정했다면)`
- `/node_modules/@types/moduleB.d.ts`
- `/node_modules/moduleB/index.ts`
- `/node_modules/moduleB/index.tsx`
- `/node_modules/moduleB/index.d.ts`

## 추가 모듈 해석 플래그 (Additional module resolution flags)

프로젝트 소스 레이아웃이 출력과 일치하지 않을 때도 있다. 예를들어, `ts`파일을 `.js`로 컴파일하고, 다른 소스 위치에서 하나의 출력 위치로 의존성을 복사하는 것이 있다. 

**최종 결과는 런타임의 모듈이 해당 정의를 포함하는 소스 파일과 다른 이름을 가질 수 있다. 또 최종 출력의 모듈 경로가 컴파일 타임에 해당하는 소스 파일 경로와 일치하지 않을 수 있다.**

TypeScript 컴파일러는 최종 출력을 생성하기위해 소스에 발생할 것으로 예상되는 변환을 컴파일러에게 알리기 위한 추가 플래그 세트가 있다.

### 기본 URL (Base URL)

`baseUrl`을 사용하는 것은 모듈들이 런타임에 단일 폴더로 "배포"되는 `AMD` 모듈 로더를 사용하는 애플리케이션에서 일반적인 방법이다. 이 모듈들의 소스는 다른 디렉터리 안에 있을 수 있지만, 빌드 스크립트가 모두 하나로 만든다.

`baseUrl`의 값은 다음 중 하나로 결정된다.

- `baseUrl` 명령 줄 인수 값 (만약 주어진 경로가 상대적이면, 현재 디렉터리를 기준으로 계산됨)
- `'tsconfig.json'`안에 `baseUrl` 프로퍼티 값 (만약 주어진 경로가 상대적이면, `'tsconfig.json'`의 위치를 기준으로 계산됨)

### 경로 매핑 (Path mapping)

가끔 모듈이 `baseUrl` 아래에 위치하지 않는 경우가 있다. 예를 들어, `"jquery"` 모듈의 `import`는 런타임에 `"node_modules/jquery/dist/jquery.slim.min.js"`로 번역된다.

`TypeScript` 컴파일러는 `tsconfig.json` 파일 안에 `"paths"` 프로퍼티를 사용한 매핑의 선언을 지원한다.

아래는 `jquery`를 위한 `"paths"` 프로퍼티를 지정하는 방법에 대한 예제이다.

```json
{
  "compilerOptions": {
    "baseUrl": ".", // "paths"가 있는 경우 반드시 지정되어야함.
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery"] // 이 매핑은 "baseUrl"에 상대적임.
    }
  }
}
```

"paths"가 "baseUrl"에 상대적으로 해석된다.

```
projectRoot
├── folder1
│   ├── file1.ts (imports 'folder1/file2' and 'folder2/file3')
│   └── file2.ts
├── generated
│   ├── folder1
│   └── folder2
│       └── file3.ts
└── tsconfig.json
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "*": [
        "*",
        "generated/*"
      ]
    }
  }
}
```

### rootDirs 가상 디렉터리 (Virtual Directories with rootDirs)

`'rootDirs'`를 사용하면, 컴파일러에게 이 "가상" 디렉터리를 구성하는 roots를 알릴 수 있다. 따라서 컴파일러는 이러한 "가상" 디렉터리 내에서 상대적 모듈 `import`를 마치 하나의 디렉터리에 같이 병합 한 것처럼 해석할 수 있다.

```
 src
 └── views
     └── view1.ts (imports './template1')
     └── view2.ts

 generated
 └── templates
         └── views
             └── template1.ts (imports './view2')
```

src/views 안의 파일들은 UI 컨트롤을 위한 유저 코드이다. generated/templated 안의 파일들은, 빌드의 일부로써 템플릿 생성기에 의해 자동-생성된 UI 템플릿 바인딩 코드이다. 빌드 스텝은 `/src/view`와 `/generated/templates/views`를 출력에서 같은 디렉터리로 복사한다고 할 때, `"rootDirs"`를 사용해서 런타임에 병합할 것으로 예상되는 roots 의 목록을 지정할 수 있다.

```json
// tsconfig.json
{
  "compilerOptions": {
    "rootDirs": [
      "src/views",
      "generated/templates/views"
    ]
  }
}
```

컴파일러가 `rootDirs` 중 하나의 하위 폴더에서 상대적 모듈 `import`를 볼 때마다, 각 `rootDirs`의 엔트리에서 이 `import`를 찾으려고 할 것이다.

또, `./#{locale}/messages`와 같은 상대 모듈 경로의 일부로 `#{locale}`와 같은 특수 경로 토큰을 보간하여 빌드 툴이 로케일 전용 번들을 자동으로 생성할 수 있다.

```js
export default [
    "您好吗",
    "很高兴认识你"
];
```

```js
{
  "compilerOptions": {
    "rootDirs": [
      "src/zh",
      "src/de",
      "src/#{locale}"
    ]
  }
}
```

컴파일러는 이제 './#{locale}/messages'를 './zh/messages'로 해석해서 로케일에 관계없는 방법으로 개발할 수 있다.

## 모듈 해석 추적 (Tracing module resolution)

앞에서 논의한 바와 같이 컴파일러는 모듈을 해석할 때 현재 폴더 외부의 파일을 방문할 수 있다. 이는 모듈이 해석되지 않거나 잘못된 정의로 해석된 이유를 진단할 때 어려울 수 있다. **`'--traceResolution'`을 사용하여 컴파일러 모듈 해석 추적을 활성화하면 모듈 해석 과정 중에 발생한 작업에 대한 인사이트를 얻을 수 있다.**

`app.ts`는 `import * as ts from "typescript"` 같은 `import`가 있다고 하자.

```
│   tsconfig.json
├───node_modules
│   └───typescript
│       └───lib
│               typescript.d.ts
└───src
        app.ts
```

`--traceResolution`으로 컴파일러를 호출

```sh
tsc --traceResolution
```

다음과 같은 출력이 발생:

```bash
======== Resolving module 'typescript' from 'src/app.ts'. ========
Module resolution kind is not specified, using 'NodeJs'.
Loading module 'typescript' from 'node_modules' folder.
File 'src/node_modules/typescript.ts' does not exist.
File 'src/node_modules/typescript.tsx' does not exist.
File 'src/node_modules/typescript.d.ts' does not exist.
File 'src/node_modules/typescript/package.json' does not exist.
File 'node_modules/typescript.ts' does not exist.
File 'node_modules/typescript.tsx' does not exist.
File 'node_modules/typescript.d.ts' does not exist.
Found 'package.json' at 'node_modules/typescript/package.json'.
'package.json' has 'types' field './lib/typescript.d.ts' that references 'node_modules/typescript/lib/typescript.d.ts'.
File 'node_modules/typescript/lib/typescript.d.ts' exist - use it as a module resolution result.
======== Module name 'typescript' was successfully resolved to 'node_modules/typescript/lib/typescript.d.ts'. ========
```

### 주의사항 (Things to look out for)

- import의 이름과 위치
  > ======== 'src/app.ts' 에서 'typesciprt' 모듈 해석. ========
- 컴파일러가 따르는 전략
  > 모듈 해석 종류가 지정되지 않으면, 'NodeJs 사용.
- npm 패키지에서 types 로딩
  > 'package.json'은 'node_modules/typescript/lib/typescript.d.ts'를 참조하는 'types' 필드 './lib/typescript.d.ts'가 있습니다.
- 최종 결과
  > ======== 모듈 이름 'typescript'는 'node_modules/typescript/lib/typescript.d.ts'로 성공적으로 해석 되었습니다. ========

## --noResolve 사용하기 (Using --noResolve)

일반적으로 컴파일러는 컴파일 과정을 시작하기 전에 모든 모듈 `import`를 해석하려고 한다. 파일의 `import`를 성공적으로 해석할 때마다, 파일은 나중에 컴파일러가 처리할 파일 세트에 추가된다.

`--noResolve` 컴파일러 옵션은 명령 줄에 전달하지 않은 파일은 컴파일에 "추가" 하지 않도록 지시한다.

### app.ts

```ts
import * as A from "moduleA" // 성공, 'moduleA'가 명령줄로 전달됨
import * as B from "moduleB" // Error TS2307: Cannot find module 'moduleB.
```

```sh
tsc app.ts moduleA.ts --noResolve
```

`--noResolve`를 사용한 `app.ts`의 컴파일은 다음과 같은 결과가 나옵니다:

- 명령 줄로 전달했기 때문에 `moduleA`는 정확하게 찾음.
- 전달하지 않았기 때문에 `moduleB`를 찾는데 실패함.

## 공통 질문 (Common Questions)

### 제외 목록에 있는 모듈을 여전히 컴파일러가 선택하는 이유는 무엇인가? (Why does a module in the exclude list still get picked up by the compiler?)

`tsconfig.json`은 폴더를 "프로젝트"로 바꾼다. `"exclude"` 나 `"files"` 엔트리를 지정하지 않으면, `tsconfig.json`를 포함하는 폴더 안의 모든 파일과 모든 하위-디렉터리가 컴파일에 포함된다. 일부 파일을 제외하고 싶으면 `"exclude"`를 사용하고, 컴파일러가 찾도록 하게 하는 대신 모든 파일을 지정하고 싶으면, `"files"`를 사용하자.

위에서 논의한 내장 모듈 해석에 대한 내용이아니다. 컴파일러는 파일을 모듈 `import` 대상으로 식별한 경우, 이전 단계에서 제외되었는지에 관계없이 컴파일에 포함하게 된다.

그래서 컴파일에 파일은 제외하기 위해서는, 그 파일을 제외하고 그 파일에 `import`나 `/// <reference path="..."" />` 지시문이 있는 모든 파일을 제외해야 한다.