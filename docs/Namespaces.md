# Namespaces

> https://typescript-kr.github.io/pages/namespaces.html

## 소개 (Introduction)

이 게시물에서는 TypeScript에서 네임스페이스를 사용하여 코드를 구성하는 다양한 방법을 간략하게 설명한다.

## 첫 번째 단계 (First steps)

이 페이지 전체에서 예제로 사용할 프로그램을 시작해보자.

## 단일 파일 검사기 (Validators in a single file)

```ts
interface StringValidator {
    isAcceptable(s: string): boolean;
}

let lettersRegexp = /^[A-Za-z]+$/;
let numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// 시도해 볼 샘플
let strings = ["Hello", "98052", "101"];

// 사용할 검사기
let validators: { [s: string]: StringValidator; } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// 각 문자열이 각 검사기를 통과했는지 표시
for (let s of strings) {
    for (let name in validators) {
        let isMatch = validators[name].isAcceptable(s);
        console.log(`'${ s }' ${ isMatch ? "matches" : "does not match" } '${ name }'.`);
    }
}
```

## 네임스페이스 적용하기 (Namespacing)

더 많은 검사기를 추가하게 되면, 타입을 추적하고 다른 객체와 이름 충돌을 방지하기 위해 일종의 구조 체계가 필요하다. 전역 네임스페이스에 다른 이름을 많이 넣는 대신, 객체들을 하나의 네임스페이스로 감싸자.

인터페이스 및 클래스가 네임스페이스 외부에서도 접근 가능하도록 선언부에 `export`를 붙인다. 반면, 변수 `letterRegexp`와 `numberRegexp`는 구현 세부 사항이므로 외부로 내보내지 않아 네임스페이스 외부 코드에서 접근할 수 없다.

## 네임스페이스화된 검사기 (Namespaced Validators)

```ts
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    const lettersRegexp = /^[A-Za-z]+$/;
    const numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// 시도해 볼 샘플
let strings = ["Hello", "98052", "101"];

// 사용할 검사기
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// 각 문자열이 각 검사기를 통과했는지 표시
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}
```

## 파일 간 분할 (Splitting Across Files)

애플리케이션 규모가 커지면, 코드를 여러 파일로 분할해야 유지 보수가 용이하다.

## 다중 파일 네임스페이스 (Multi-file namespaces)

여기서 Validation 네임스페이스를 여러 파일로 분할한다. 파일이 분리되어 있어도 같은 네임스페이스에 기여할 수 있고 마치 한 곳에서 정의된 것처럼 사용할 수 있다. 파일 간 의존성이 존재하므로, 참조 태그를 추가하여 컴파일러에게 파일 간의 관계를 알린다.

### Validation.ts

```ts
namespace Validation{
    export interface StringValidator{
        isAcceptable(s: string): boolean;
    }
}
```

### LettersOnlyValidators.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

### ZipCodeValidators.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
    const numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
```

### Test.ts

```ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}
```

파일이 여러 개 있으면 컴파일된 코드가 모두 로드되는지 확인해야 한다. 이를 수행하는 두 가지 방법이 있다.

먼저, 모든 입력 파일을 하나의 JavaScript 출력 파일로 컴파일하기 위해 --outFile 플래그를 사용하여 연결 출력(concatenated output)을 사용할 수 있다.

```sh
tsc --outFile sample.js Test.ts
```

컴파일러는 파일에 있는 참조 태그를 기반으로 출력 파일을 자동으로 정렬한다.

또는 파일별 컴파일 (기본값)을 사용하여 각 입력 파일을 하나의 JavaScript 파일로 생성할 수 있다. 여러 JS 파일이 생성되는 경우, 웹 페이지에서 생성된 개별 파일을 적절한 순서로 로드하기 위해 `<script>` 태그를 사용해야한다.

### MyTestPage.html (인용)

```js
    <script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```

## 별칭 (Aliases)

네임스페이스 작업을 단순화할 수 있는 또 다른 방법은 일반적으로 사용되는 객체의 이름을 더 짧게 만들기 위해 `import q = x.y.z`를 사용하는 것이다. 모듈을 로드하는 데 사용되는 `import x = require ("name")` 구문과 혼동하지 않기 위해, 이 구문은 단순히 특정 심벌에 별칭을 생성한다.

```ts
namespace Shapes {
    export namespace Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square(); // 'new Shapes.Polygons.Square()'와 동일
```

## 다른 JavaScript 라이브러리로 작업하기 (Working with Other JavaScript Libraries)

TypeScript로 작성되지 않은 라이브러리의 형태를 설명하려면, 라이브러리가 외부에 제공하는 API를 선언해야한다. 대부분의 JavaScript 라이브러리는 소수의 최상위 객체만 노출하므로 네임스페이스를 사용하는 것이 좋다.

구현을 정의하지 않은 선언을 `"ambient"`라고 부른다. 일반적으로 이것은 `.d.ts` 파일에 정의되어 있습니다. C/C++에 익숙하다면 이를 `.h` 파일로 생각할 수 있다.

## Ambient 네임스페이스 (Ambient Namespaces)

널리 사용되는 `D3` 라이브러리는 `d3`이라는 전역 객체에서 기능을 정의한다. 이 라이브러리는 `<script>` 태그를 통해 로드되므로(모듈 로더 대신) 형태를 정의하기 위해 선언할 때 네임스페이스를 사용한다. TypeScript 컴파일러는 이 형태를 보기 위해, ambient 네임스페이스 선언을 사용한다.

### D3.d.ts

```ts
declare namespace D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```