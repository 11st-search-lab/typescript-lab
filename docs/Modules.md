# 모듈

> https://typescript-kr.github.io/pages/modules.html



ECMAScript 2015부터 JavaScript에는 모듈 개념이 있으며, TypeScript는 또한 이 개념을 공유한다.



### 모듈은 전역 스코프가 아닌 자체 스코프 내에서 실행된다.

 모듈 내에서 선언된 변수, 함수, 클래스 등은 [`export` 양식](https://typescript-kr.github.io/pages/modules.html#export) 중 하나를 사용하여 명시적으로 export 하지 않는 한 모듈 외부에서 보이지 않는다. 반대로 다른 모듈에서 export 한 변수, 함수, 클래스, 인터페이스 등을 사용하기 위해서는 [`import` 양식](https://typescript-kr.github.io/pages/modules.html#import) 중 하나를 사용하여 import 해야 한다.



### 모듈은 선언형이다.

모듈 간의 관계는 파일 수준의 imports 및 exports 관점에서 지정된다.



### 모듈은 모듈 로더를 사용하여 다른 모듈을 import 한다. 

런타임 시 모듈 로더는 모듈을 실행하기 전에 모듈의 모든 의존성을 찾고 실행해야 한다. 

**<JavaScript에서 사용하는 유명한 모듈 로더>**

- [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 모듈 용 Node.js의 로더

- 웹 애플리케이션의 [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) 모듈 용 [RequireJS](https://requirejs.org/) 로더



## Export

### 선언 export 하기 (Exporting a declaration)

`export` 키워드를 추가하여 모든 선언 (변수, 함수, 클래스, 타입 별칭, 인터페이스)를 export 할 수 있다.



##### StringValidator.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

##### ZipCodeValidator.ts

```ts
import { StringValidator } from "./StringValidator";
//...
```



### Export 문 (Export statements)

Export 문은 export 할 이름을 바꿔야 할 때 편리하다. 

```ts
class ZipCodeValidator implements StringValidator {
//...
export { ZipCodeValidator as mainValidator };
```



### Re-export 하기 (Re-exports)

종종 모듈은 다른 모듈을 확장하고 일부 기능을 부분적으로 노출한다. Re-export 하기는 지역적으로 import 하거나, 지역 변수를 도입하지 않는다. (entry 사용에도 유용)



##### ParseIntBasedZipCodeValidator.ts

```ts
// 기존 validator의 이름을 변경 후 export
export {ZipCodeValidator as RegExpBasedZipCodeValidator} from "./ZipCodeValidator";
```

선택적으로, 하나의 모듈은 하나 혹은 여러 개의 모듈을 감쌀 수 있고, `export * from "module"` 구문을 사용해 export 하는 것을 모두 결합할 수 있다.

##### AllValidators.ts

```ts
export * from "./StringValidator"; // 'StringValidator' 인터페이스를 내보냄
export * from "./ZipCodeValidator";  // 'ZipCodeValidator' 와 const 'numberRegexp' 클래스를 내보냄
```



## Import

export 한 선언은 아래의 `import` 양식 중 하나를 사용하여 import 한다.



### 모듈에서 단일 export를 import 하기 (Import a single export from a module)

```ts
import { ZipCodeValidator } from "./ZipCodeValidator";

let myValidator = new ZipCodeValidator();
```



이름을 수정해서 import 할 수 있다.

```ts
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```



### 전체 모듈을 단일 변수로 import 해서, 모듈 exports 접근에 사용하기 (Import the entire module into a single variable, and use it to access the module exports)

```ts
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```



### 부수효과만을 위해 모듈 import 하기 (Import a module for side-effects only)

권장되지는 않지만, 일부 모듈은 다른 모듈에서 사용할 수 있도록 일부 전역 상태로 설정한다. 이러한 모듈은 어떤 exports도 없거나, 사용자가 exports에 관심이 없다.

```ts
import "./my-module.js"
```



### 타입 import 하기 (Importing Types)

TypeScript 3.8에서는 `import` 문 혹은 `import type`을 사용하여 타입을 import 할 수 있다.

```ts
// 동일한 import를 재사용하기
import {APIResponseType} from "./api";

// 명시적으로 import type을 사용하기
import type {APIResponseType} from "./api";
```

`import type`은 항상 JavaScript에서 제거되며, 바벨 같은 도구는 `isolatedModules` 컴파일러 플래그를 통해 코드에 대해 더 나은 가정을 할 수 있다. [3.8 릴리즈 정보](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#type-only-imports-exports)참고



### Default exports

각 모듈은 선택적으로 `default` export를 export 할 수 있다. default export는 `default` 키워드로 표시된다.

**모듈당 하나의 `default` export만 가능**합니다. `default` export는 다른 import 양식을 사용하여 import 합니다.

`default` exports는 정말 편리합니다. 예를 들어 jQuery와 같은 라이브러리는 `jQuery` 혹은 `$`와 같은 default export를 가질 수 있으며, `$`나 `jQuery`와 같은 이름으로 import할 수 있다.



##### [JQuery.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/jquery/JQuery.d.ts)

```ts
declare let $: JQuery;
export default $;
```

##### App.ts

```ts
import $ from "jquery";

$("button.continue").html( "Next Step..." );
```



**클래스 및 함수 선언은 default exports**로 직접 작성될 수 있다. default export 클래스 및 함수 선언 이름은 선택사항이다.

##### ZipCodeValidator.ts

```ts
export default class ZipCodeValidator {
 //...
}
```



## x로 모두 export 하기 (Export all as x)

TypeScript 3.8에서는 다음 이름이 다른 모듈로 re-export 될 때 단축어처럼 `export * as ns`를 사용할 수 있다.

```ts
export * as utilities from "./utilities";
```

모듈에서 모든 의존성을 가져와 export한 필드로 만들면, 다음과 같이 import할 수 있다.

```ts
import { utilities } from "./index";
```



## `export =`와 `import = require()` (`export =` and `import = require()`)

CommonJS와 AMD 둘 다 일반적으로 모듈의 모든 exports를 포함하는 `exports` 객체의 개념을 가지고 있다.

TypeScript는 기존의 CommonJS와 AMD 워크플로우를 모델링 하기 위해 `export =`를 지원한다.

- `export =` 구문은 모듈에서 export되는 단일 객체를 지정합니다. 클래스, 인터페이스, 네임스페이스, 함수 혹은 열거형이 될 수 있다.

- `export =`를 사용하여 모듈을 export할 때, TypeScript에 특정한 `import module = require("module")`를 사용하여 모듈을 가져와야 한다.

##### ZipCodeValidator.ts

```ts
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```





## 모듈을 위한 코드 생성 (Code Generation for Modules)

컴파일 중에는 지정된 모듈 대상에 따라 컴파일러는 Node.js ([CommonJS](http://wiki.commonjs.org/wiki/CommonJS)), require.js ([AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)), [UMD](https://github.com/umdjs/umd), [SystemJS](https://github.com/systemjs/systemjs), 또는 [ECMAScript 2015 native modules](http://www.ecma-international.org/ecma-262/6.0/#sec-modules) (ES6) 모듈-로딩 시스템에 적합한 코드를 생성한다. 

아래 예제는 import 및 export 하기 중에 사용된 이름이 모듈 로딩 코드로 변환되는 방법을 보여준다.

##### SimpleModule.ts

```ts
import m = require("mod");
export let t = m.something + 1;
```

##### AMD / RequireJS SimpleModule.js

```js
define(["require", "exports", "./mod"], function (require, exports, mod_1) {
    exports.t = mod_1.something + 1;
});
```

##### CommonJS / Node SimpleModule.js

```js
var mod_1 = require("./mod");
exports.t = mod_1.something + 1;
```

##### UMD SimpleModule.js

```js
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports); if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./mod"], factory);
    }
})(function (require, exports) {
    var mod_1 = require("./mod");
    exports.t = mod_1.something + 1;
});
```

##### System SimpleModule.js

```js
System.register(["./mod"], function(exports_1) {
    var mod_1;
    var t;
    return {
        setters:[
            function (mod_1_1) {
                mod_1 = mod_1_1;
            }],
        execute: function() {
            exports_1("t", t = mod_1.something + 1);
        }
    }
});
```

##### Native ECMAScript 2015 modules SimpleModule.js

```js
import { something } from "./mod";
export var t = something + 1;
```



### 간단한 예제 (Simple Example)

아래에서는 각 모듈에서 단일 이름으로 export 하기 위해 이전 예제에서 사용한 Validator 구현을 통합한다.

컴파일 하려면, 명령 줄에서 모듈 대상을 지정해야 한다. 

- Node.js의 경우, `--module commonjs`를 사용
- require.js의 경우 `--module amd`를 사용

```Shell
tsc --module commonjs Test.ts
```

컴파일이 되면, 각 모듈은 별도의 `.js`파일이 된다. 참조 태그와 마찬가지로, 컴파일러는 `import`문을 따라 의존적인 파일들을 컴파일 한다.



### 선택적 모듈 로딩과 기타 고급 로딩 시나리오 (Optional Module Loading and Other Advanced Loading Scenarios)

상황에 따라 특정 조건에서만 모듈을 로드하도록 만들 수 있다. TypeScript에서는 아래에 있는 패턴을 사용하여 이 시나리오와 다른 고급 로딩 시나리오를 구현하여 타입의 안전성을 잃지 않고 모듈 로더를 직접 호출할 수 있다.

- 컴파일러는 노출된 JavaScript 안에서 각 모듈의 사용 여부를 감지
- 모듈 식별자가 표현식이 아닌 타입 표시로만 사용된다면 그 모듈에 대한 `require` 호출은 발생X 
- 사용하지 않는 참조를 제거하면 성능을 최적화할 수 있으며, 해당 모듈을 선택적으로 로딩 가능

이 패턴의 핵심 아이디어는 `import id = require("...")` 문을 통해 **모듈로 노출된 타입에 접근이 가능하다는 것이다**.

 아래 `if` 블록에 보이는 것처럼, 모듈 로더는 (`require`을 통해) 동적으로 호출된다. 이 기능은 참조-제거 최적화를 활용하므로 필요할 때만 모듈을 로드할 수 있다. 해당 패턴이 동작하려면 `import`를 통해 정의된 심벌은 오직 타입 위치(즉, JavaScript로 방출되는 위치에서는 사용 안 함)에서만 사용되는 것이 중요하다.

타입 안전성을 유지하기 위해, `typeof` 키워드를 사용할 수 있다. `typeof` 키워드는 타입 위치에서 사용될 때는 값의 타입, 이 경우에는 모듈의 타입을 생성한다.

##### Node.js에서 동적 모듈 로딩 (Dynamic Module Loading in Node.js)

```ts
declare function require(moduleName: string): any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
    let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator");
    let validator = new ZipCodeValidator();
    if (validator.isAcceptable("...")) { /* ... */ }
}
```



### 다른 JavaScript 라이브러리와 함께 사용하기 (Working with Other JavaScript Libraries)

TypeScript로 작성되지 않은 라이브러리의 형태를 설명하려면, 라이브러리를 노출하는 API를 선언해야 한다.

구현을 정의하지 않은 선언을 "ambient"라 한다. 이 선언들은 일반적으로 `.d.ts` 파일에 정의되어 있다.



### Ambient 모듈 (Ambient Modules)

Node.js에서는 대부분의 작업은 하나 이상의 모듈을 로드하여 수행한다. 최상위-레벨의 내보내기 선언으로 각 모듈을 `.d.ts` 파일로 정의할 수 있지만, 더 큰 하나의 `.d.ts` 파일로 모듈들을 작성하는 것이 더 편리하다. 

##### node.d.ts (간단한 발췌)

```ts
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export var sep: string;
}
```

이제 `/// <reference>` `node.d.ts`를 수행한 다음, `import url = require("url");` 또는 `import * as URL from "url"`을 사용하여 모듈을 로드할 수 있다.

```ts
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");
```



### 속기 ambient 모듈 (Shorthand ambient modules)

새로운 모듈을 사용하기 전에 선언을 작성하지 않는 경우, 속기 선언(shorthand declaration)을 사용하여 빠르게 시작할 수 있다.

##### declarations.d.ts

```ts
declare module "hot-new-module";
```

속기 모듈로부터 모든 imports는 `any` 타입을 가진다.

```ts
import x, {y} from "hot-new-module";
x(y);
```



### 와일드카드 모듈 선언 (Wildcard module declarations)

[SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/overview.md#plugin-syntax)나 [AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md)와 같은 모듈 로더는 **비-JavaScirpt 내용을 import** 할 수 있다. 이 둘은 일반적으로 접두사 또는 접미사를 사용하여 특수한 로딩 의미를 표시한다. 이러한 경우를 다루기 위해 와일드카드 모듈 선언을 사용할 수 있다.

```ts
declare module "*!text" {
    const content: string;
    export default content;
}
// 일부는 다른 방법으로 사용합니다.
declare module "json!*" {
    const value: any;
    export default value;
}
```

이제 `"*!text"` 나 `"json!*"`와 일치하는 것들을 import 할 수 있다.

```ts
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```



### UMD 모듈 (UMD modules)

일부 라이브러리는 많은 모듈 로더에서 사용되거나, 모듈 로딩 (전역 변수) 없이 사용되도록 설계되었다. 이를 [UMD](https://github.com/umdjs/umd) 모듈이라고 합니다. 이러한 라이브러리는 import나 전역 변수를 통해 접근할 수 있다. 

##### math-lib.d.ts

```ts
export function isPrime(x: number): boolean;
export as namespace mathLib;
```



라이브러리는 모듈 내에서 import로 사용할 수 있다:

```ts
import { isPrime } from "math-lib";
isPrime(2);
mathLib.isPrime(2); // 오류: 모듈 내부에서 전역 정의를 사용할 수 없습니다.
```



전역 변수로도 사용할 수 있지만, 스크립트 내에서만 사용할 수 있다. (스크립트는 imports나 exports가 없는 파일)

```ts
mathLib.isPrime(2);
```



## 모듈 구조화에 대한 지침 (Guidance for structuring modules)

- **가능한 최상위-레벨에 가깝게 export 하기**
  - 중첩 수준을 과도하게 추가하면 다루기 힘들어지는 경향이 있으므로, 어떻게 구조를 구성할지 신중하게 생각해야 한다.
  - 모듈에서 네임스페이스를 export 하는 것은 너무 많은 중첩 레이어를 추가하는 예이다. 
  - 네임스페이스는 때때로 용도가 있지만, 모듈을 사용할 때 추가적인 레벨의 간접 참조를 추가한다. 

- **단일 `class`나 `function`을 export 할 경우, `export default`를 사용하기**

- **여러 객체를 export 하는 경우, 최상위-레벨에 두기** 

  MyThings.ts

  ```ts
  export class SomeType { /* ... */ }
  export function someFunc() { /* ... */ }
  ```

- 모듈

  > https://typescript-kr.github.io/pages/modules.html

  

  ECMAScript 2015부터 JavaScript에는 모듈 개념이 있으며, TypeScript는 또한 이 개념을 공유한다.

  

  ### 모듈은 전역 스코프가 아닌 자체 스코프 내에서 실행된다.

   모듈 내에서 선언된 변수, 함수, 클래스 등은 [`export` 양식](https://typescript-kr.github.io/pages/modules.html#export) 중 하나를 사용하여 명시적으로 export 하지 않는 한 모듈 외부에서 보이지 않는다. 반대로 다른 모듈에서 export 한 변수, 함수, 클래스, 인터페이스 등을 사용하기 위해서는 [`import` 양식](https://typescript-kr.github.io/pages/modules.html#import) 중 하나를 사용하여 import 해야 한다.

  

  ### 모듈은 선언형이다.

  모듈 간의 관계는 파일 수준의 imports 및 exports 관점에서 지정된다.

  

  ### 모듈은 모듈 로더를 사용하여 다른 모듈을 import 한다. 

  런타임 시 모듈 로더는 모듈을 실행하기 전에 모듈의 모든 의존성을 찾고 실행해야 한다. 

  **<JavaScript에서 사용하는 유명한 모듈 로더>**

  - [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 모듈 용 Node.js의 로더

  - 웹 애플리케이션의 [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) 모듈 용 [RequireJS](https://requirejs.org/) 로더

  

  ## Export

  ### 선언 export 하기 (Exporting a declaration)

  `export` 키워드를 추가하여 모든 선언 (변수, 함수, 클래스, 타입 별칭, 인터페이스)를 export 할 수 있다.

  

  ##### StringValidator.ts

  ```ts
  export interface StringValidator {
      isAcceptable(s: string): boolean;
  }
  ```

  ##### ZipCodeValidator.ts

  ```ts
  import { StringValidator } from "./StringValidator";
  //...
  ```

  

  ### Export 문 (Export statements)

  Export 문은 export 할 이름을 바꿔야 할 때 편리하다. 

  ```ts
  class ZipCodeValidator implements StringValidator {
  //...
  export { ZipCodeValidator as mainValidator };
  ```

  

  ### Re-export 하기 (Re-exports)

  종종 모듈은 다른 모듈을 확장하고 일부 기능을 부분적으로 노출한다. Re-export 하기는 지역적으로 import 하거나, 지역 변수를 도입하지 않는다. (entry 사용에도 유용)

  

  ##### ParseIntBasedZipCodeValidator.ts

  ```ts
  // 기존 validator의 이름을 변경 후 export
  export {ZipCodeValidator as RegExpBasedZipCodeValidator} from "./ZipCodeValidator";
  ```

  선택적으로, 하나의 모듈은 하나 혹은 여러 개의 모듈을 감쌀 수 있고, `export * from "module"` 구문을 사용해 export 하는 것을 모두 결합할 수 있다.

  ##### AllValidators.ts

  ```ts
  export * from "./StringValidator"; // 'StringValidator' 인터페이스를 내보냄
  export * from "./ZipCodeValidator";  // 'ZipCodeValidator' 와 const 'numberRegexp' 클래스를 내보냄
  ```

  

  ## Import

  export 한 선언은 아래의 `import` 양식 중 하나를 사용하여 import 한다.

  

  ### 모듈에서 단일 export를 import 하기 (Import a single export from a module)

  ```ts
  import { ZipCodeValidator } from "./ZipCodeValidator";
  
  let myValidator = new ZipCodeValidator();
  ```

  

  이름을 수정해서 import 할 수 있다.

  ```ts
  import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
  let myValidator = new ZCV();
  ```

  

  ### 전체 모듈을 단일 변수로 import 해서, 모듈 exports 접근에 사용하기 (Import the entire module into a single variable, and use it to access the module exports)

  ```ts
  import * as validator from "./ZipCodeValidator";
  let myValidator = new validator.ZipCodeValidator();
  ```

  

  ### 부수효과만을 위해 모듈 import 하기 (Import a module for side-effects only)

  권장되지는 않지만, 일부 모듈은 다른 모듈에서 사용할 수 있도록 일부 전역 상태로 설정한다. 이러한 모듈은 어떤 exports도 없거나, 사용자가 exports에 관심이 없다.

  ```ts
  import "./my-module.js"
  ```

  

  ### 타입 import 하기 (Importing Types)

  TypeScript 3.8에서는 `import` 문 혹은 `import type`을 사용하여 타입을 import 할 수 있다.

  ```ts
  // 동일한 import를 재사용하기
  import {APIResponseType} from "./api";
  
  // 명시적으로 import type을 사용하기
  import type {APIResponseType} from "./api";
  ```

  `import type`은 항상 JavaScript에서 제거되며, 바벨 같은 도구는 `isolatedModules` 컴파일러 플래그를 통해 코드에 대해 더 나은 가정을 할 수 있다. [3.8 릴리즈 정보](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#type-only-imports-exports)참고

  

  ### Default exports

  각 모듈은 선택적으로 `default` export를 export 할 수 있다. default export는 `default` 키워드로 표시된다.

  **모듈당 하나의 `default` export만 가능**합니다. `default` export는 다른 import 양식을 사용하여 import 합니다.

  `default` exports는 정말 편리합니다. 예를 들어 jQuery와 같은 라이브러리는 `jQuery` 혹은 `$`와 같은 default export를 가질 수 있으며, `$`나 `jQuery`와 같은 이름으로 import할 수 있다.

  

  ##### [JQuery.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/jquery/JQuery.d.ts)

  ```ts
  declare let $: JQuery;
  export default $;
  ```

  ##### App.ts

  ```ts
  import $ from "jquery";
  
  $("button.continue").html( "Next Step..." );
  ```

  

  **클래스 및 함수 선언은 default exports**로 직접 작성될 수 있다. default export 클래스 및 함수 선언 이름은 선택사항이다.

  ##### ZipCodeValidator.ts

  ```ts
  export default class ZipCodeValidator {
   //...
  }
  ```

  

  ## x로 모두 export 하기 (Export all as x)

  TypeScript 3.8에서는 다음 이름이 다른 모듈로 re-export 될 때 단축어처럼 `export * as ns`를 사용할 수 있다.

  ```ts
  export * as utilities from "./utilities";
  ```

  모듈에서 모든 의존성을 가져와 export한 필드로 만들면, 다음과 같이 import할 수 있다.

  ```ts
  import { utilities } from "./index";
  ```

  

  ## `export =`와 `import = require()` (`export =` and `import = require()`)

  CommonJS와 AMD 둘 다 일반적으로 모듈의 모든 exports를 포함하는 `exports` 객체의 개념을 가지고 있다.

  TypeScript는 기존의 CommonJS와 AMD 워크플로우를 모델링 하기 위해 `export =`를 지원한다.

  - `export =` 구문은 모듈에서 export되는 단일 객체를 지정합니다. 클래스, 인터페이스, 네임스페이스, 함수 혹은 열거형이 될 수 있다.

  - `export =`를 사용하여 모듈을 export할 때, TypeScript에 특정한 `import module = require("module")`를 사용하여 모듈을 가져와야 한다.

  ##### ZipCodeValidator.ts

  ```ts
  let numberRegexp = /^[0-9]+$/;
  class ZipCodeValidator {
      isAcceptable(s: string) {
          return s.length === 5 && numberRegexp.test(s);
      }
  }
  export = ZipCodeValidator;
  ```

  

  

  ## 모듈을 위한 코드 생성 (Code Generation for Modules)

  컴파일 중에는 지정된 모듈 대상에 따라 컴파일러는 Node.js ([CommonJS](http://wiki.commonjs.org/wiki/CommonJS)), require.js ([AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)), [UMD](https://github.com/umdjs/umd), [SystemJS](https://github.com/systemjs/systemjs), 또는 [ECMAScript 2015 native modules](http://www.ecma-international.org/ecma-262/6.0/#sec-modules) (ES6) 모듈-로딩 시스템에 적합한 코드를 생성한다. 

  아래 예제는 import 및 export 하기 중에 사용된 이름이 모듈 로딩 코드로 변환되는 방법을 보여준다.

  ##### SimpleModule.ts

  ```ts
  import m = require("mod");
  export let t = m.something + 1;
  ```

  ##### AMD / RequireJS SimpleModule.js

  ```js
  define(["require", "exports", "./mod"], function (require, exports, mod_1) {
      exports.t = mod_1.something + 1;
  });
  ```

  ##### CommonJS / Node SimpleModule.js

  ```js
  var mod_1 = require("./mod");
  exports.t = mod_1.something + 1;
  ```

  ##### UMD SimpleModule.js

  ```js
  (function (factory) {
      if (typeof module === "object" && typeof module.exports === "object") {
          var v = factory(require, exports); if (v !== undefined) module.exports = v;
      }
      else if (typeof define === "function" && define.amd) {
          define(["require", "exports", "./mod"], factory);
      }
  })(function (require, exports) {
      var mod_1 = require("./mod");
      exports.t = mod_1.something + 1;
  });
  ```

  ##### System SimpleModule.js

  ```js
  System.register(["./mod"], function(exports_1) {
      var mod_1;
      var t;
      return {
          setters:[
              function (mod_1_1) {
                  mod_1 = mod_1_1;
              }],
          execute: function() {
              exports_1("t", t = mod_1.something + 1);
          }
      }
  });
  ```

  ##### Native ECMAScript 2015 modules SimpleModule.js

  ```js
  import { something } from "./mod";
  export var t = something + 1;
  ```

  

  ### 간단한 예제 (Simple Example)

  아래에서는 각 모듈에서 단일 이름으로 export 하기 위해 이전 예제에서 사용한 Validator 구현을 통합한다.

  컴파일 하려면, 명령 줄에서 모듈 대상을 지정해야 한다. 

  - Node.js의 경우, `--module commonjs`를 사용
  - require.js의 경우 `--module amd`를 사용

  ```Shell
  tsc --module commonjs Test.ts
  ```

  컴파일이 되면, 각 모듈은 별도의 `.js`파일이 된다. 참조 태그와 마찬가지로, 컴파일러는 `import`문을 따라 의존적인 파일들을 컴파일 한다.

  

  ### 선택적 모듈 로딩과 기타 고급 로딩 시나리오 (Optional Module Loading and Other Advanced Loading Scenarios)

  상황에 따라 특정 조건에서만 모듈을 로드하도록 만들 수 있다. TypeScript에서는 아래에 있는 패턴을 사용하여 이 시나리오와 다른 고급 로딩 시나리오를 구현하여 타입의 안전성을 잃지 않고 모듈 로더를 직접 호출할 수 있다.

  - 컴파일러는 노출된 JavaScript 안에서 각 모듈의 사용 여부를 감지
  - 모듈 식별자가 표현식이 아닌 타입 표시로만 사용된다면 그 모듈에 대한 `require` 호출은 발생X 
  - 사용하지 않는 참조를 제거하면 성능을 최적화할 수 있으며, 해당 모듈을 선택적으로 로딩 가능

  이 패턴의 핵심 아이디어는 `import id = require("...")` 문을 통해 **모듈로 노출된 타입에 접근이 가능하다는 것이다**.

   아래 `if` 블록에 보이는 것처럼, 모듈 로더는 (`require`을 통해) 동적으로 호출된다. 이 기능은 참조-제거 최적화를 활용하므로 필요할 때만 모듈을 로드할 수 있다. 해당 패턴이 동작하려면 `import`를 통해 정의된 심벌은 오직 타입 위치(즉, JavaScript로 방출되는 위치에서는 사용 안 함)에서만 사용되는 것이 중요하다.

  타입 안전성을 유지하기 위해, `typeof` 키워드를 사용할 수 있다. `typeof` 키워드는 타입 위치에서 사용될 때는 값의 타입, 이 경우에는 모듈의 타입을 생성한다.

  ##### Node.js에서 동적 모듈 로딩 (Dynamic Module Loading in Node.js)

  ```ts
  declare function require(moduleName: string): any;
  
  import { ZipCodeValidator as Zip } from "./ZipCodeValidator";
  
  if (needZipValidation) {
      let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator");
      let validator = new ZipCodeValidator();
      if (validator.isAcceptable("...")) { /* ... */ }
  }
  ```

  

  ### 다른 JavaScript 라이브러리와 함께 사용하기 (Working with Other JavaScript Libraries)

  TypeScript로 작성되지 않은 라이브러리의 형태를 설명하려면, 라이브러리를 노출하는 API를 선언해야 한다.

  구현을 정의하지 않은 선언을 "ambient"라 한다. 이 선언들은 일반적으로 `.d.ts` 파일에 정의되어 있다.

  

  ### Ambient 모듈 (Ambient Modules)

  Node.js에서는 대부분의 작업은 하나 이상의 모듈을 로드하여 수행한다. 최상위-레벨의 내보내기 선언으로 각 모듈을 `.d.ts` 파일로 정의할 수 있지만, 더 큰 하나의 `.d.ts` 파일로 모듈들을 작성하는 것이 더 편리하다. 

  ##### node.d.ts (간단한 발췌)

  ```ts
  declare module "url" {
      export interface Url {
          protocol?: string;
          hostname?: string;
          pathname?: string;
      }
  
      export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
  }
  
  declare module "path" {
      export function normalize(p: string): string;
      export function join(...paths: any[]): string;
      export var sep: string;
  }
  ```

  이제 `/// <reference>` `node.d.ts`를 수행한 다음, `import url = require("url");` 또는 `import * as URL from "url"`을 사용하여 모듈을 로드할 수 있다.

  ```ts
  /// <reference path="node.d.ts"/>
  import * as URL from "url";
  let myUrl = URL.parse("http://www.typescriptlang.org");
  ```

  

  ### 속기 ambient 모듈 (Shorthand ambient modules)

  새로운 모듈을 사용하기 전에 선언을 작성하지 않는 경우, 속기 선언(shorthand declaration)을 사용하여 빠르게 시작할 수 있다.

  ##### declarations.d.ts

  ```ts
  declare module "hot-new-module";
  ```

  속기 모듈로부터 모든 imports는 `any` 타입을 가진다.

  ```ts
  import x, {y} from "hot-new-module";
  x(y);
  ```

  

  ### 와일드카드 모듈 선언 (Wildcard module declarations)

  [SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/overview.md#plugin-syntax)나 [AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md)와 같은 모듈 로더는 **비-JavaScirpt 내용을 import** 할 수 있다. 이 둘은 일반적으로 접두사 또는 접미사를 사용하여 특수한 로딩 의미를 표시한다. 이러한 경우를 다루기 위해 와일드카드 모듈 선언을 사용할 수 있다.

  ```ts
  declare module "*!text" {
      const content: string;
      export default content;
  }
  // 일부는 다른 방법으로 사용합니다.
  declare module "json!*" {
      const value: any;
      export default value;
  }
  ```

  이제 `"*!text"` 나 `"json!*"`와 일치하는 것들을 import 할 수 있다.

  ```ts
  import fileContent from "./xyz.txt!text";
  import data from "json!http://example.com/data.json";
  console.log(data, fileContent);
  ```

  

  ### UMD 모듈 (UMD modules)

  일부 라이브러리는 많은 모듈 로더에서 사용되거나, 모듈 로딩 (전역 변수) 없이 사용되도록 설계되었다. 이를 [UMD](https://github.com/umdjs/umd) 모듈이라고 합니다. 이러한 라이브러리는 import나 전역 변수를 통해 접근할 수 있다. 

  ##### math-lib.d.ts

  ```ts
  export function isPrime(x: number): boolean;
  export as namespace mathLib;
  ```

  

  라이브러리는 모듈 내에서 import로 사용할 수 있다:

  ```ts
  import { isPrime } from "math-lib";
  isPrime(2);
  mathLib.isPrime(2); // 오류: 모듈 내부에서 전역 정의를 사용할 수 없습니다.
  ```

  

  전역 변수로도 사용할 수 있지만, 스크립트 내에서만 사용할 수 있다. (스크립트는 imports나 exports가 없는 파일)

  ```ts
  mathLib.isPrime(2);
  ```

  

  ## 모듈 구조화에 대한 지침 (Guidance for structuring modules)

  - **가능한 최상위-레벨에 가깝게 export 하기**
    - 중첩 수준을 과도하게 추가하면 다루기 힘들어지는 경향이 있으므로, 어떻게 구조를 구성할지 신중하게 생각해야 한다.
    - 모듈에서 네임스페이스를 export 하는 것은 너무 많은 중첩 레이어를 추가하는 예이다. 
    - 네임스페이스는 때때로 용도가 있지만, 모듈을 사용할 때 추가적인 레벨의 간접 참조를 추가한다. 

  - **단일 `class`나 `function`을 export 할 경우, `export default`를 사용하기**

  - **여러 객체를 export 하는 경우, 최상위-레벨에 두기** 

    MyThings.ts

    ```ts
    export class SomeType { /* ... */ }
    export function someFunc() { /* ... */ }
    ```

  - **import 한 이름을 명시적으로 나열**

    Consumer.ts

    ```ts
    import { SomeType, someFunc } from "./MyThings";
    let x = new SomeType();
    let y = someFunc();
    ```

  - **많은 것을 import 하는 경우, 네임스페이스 import 패턴을 사용하세요**

    MyLargeModule.ts

    ```ts
    export class Dog { ... }
    export class Cat { ... }
    export class Tree { ... }
    export class Flower { ... }
    ```

    Consumer.ts

    ```ts
    import * as myLargeModule from "./MyLargeModule.ts";
    let x = new myLargeModule.Dog();
    ```

  - **상속을 위한 re-export 하기 (Re-export to extend)**
    - 기존의 객체를 *변형하지 않고* 새로운 기능을 제공하는 개체를 export 하라.

  -  **네임스페이스를 사용하지 마세요 (Do not use namespaces in modules)**
    - 네임스페이스는 전역 스코프에서 네이밍 충돌을 피하기 위해 중요하다. 예를 들어, `My.Application.Customer.AddForm`과 `My.Application.Order.AddForm` -- 두 타입의 이름은 같지만 다른 네임스페이스를 가지고 있다. 그러나 모듈 내에서 두 개의 객체가 같은 이름을 가질만한 이유는 없기 때문에 문제가 되지 않기에 네임스페이스를 사용하지 않는 것이 좋다. 
    - 모듈과 네임스페이스에 대한 자세한 내용은 [Namespaces and Modules](https://typescript-kr.github.io/pages/namespaces-and-modules.html)를 참고

  ### 

- **import 한 이름을 명시적으로 나열**

  Consumer.ts

  ```ts
  import { SomeType, someFunc } from "./MyThings";
  let x = new SomeType();
  let y = someFunc();
  ```

- **많은 것을 import 하는 경우, 네임스페이스 import 패턴을 사용하세요**

  MyLargeModule.ts

  ```ts
  export class Dog { ... }
  export class Cat { ... }
  export class Tree { ... }
  export class Flower { ... }
  ```

  Consumer.ts

  ```ts
  import * as myLargeModule from "./MyLargeModule.ts";
  let x = new myLargeModule.Dog();
  ```

- **상속을 위한 re-export 하기 (Re-export to extend)**
  - 기존의 객체를 *변형하지 않고* 새로운 기능을 제공하는 개체를 export 하라.

-  **네임스페이스를 사용하지 마세요 (Do not use namespaces in modules)**
  - 네임스페이스는 전역 스코프에서 네이밍 충돌을 피하기 위해 중요하다. 예를 들어, `My.Application.Customer.AddForm`과 `My.Application.Order.AddForm` -- 두 타입의 이름은 같지만 다른 네임스페이스를 가지고 있다. 그러나 모듈 내에서 두 개의 객체가 같은 이름을 가질만한 이유는 없기 때문에 문제가 되지 않기에 네임스페이스를 사용하지 않는 것이 좋다. 
  - 모듈과 네임스페이스에 대한 자세한 내용은 [Namespaces and Modules](https://typescript-kr.github.io/pages/namespaces-and-modules.html)를 참고

### 