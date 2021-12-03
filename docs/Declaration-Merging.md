# 선언 병합

> https://typescript-kr.github.io/pages/declaration-merging.html



## 소개 

TypeScript의 독특한 개념들 중 일부는 타입 레벨에서 JavaScript 객체의 형태를 설명한다. 이 중 '선언 병합'은 고급 추상화 개념이다. 



### 선언 병합

- 컴파일러가 같은 이름으로 선언된 두 개의 개별적인 선언을 하나의 정의로 병합하는 것 
- 병합된 정의는 원래 두 선언 각각의 기능을 모두 가짐
- 병합할 선언의 개수는 상관 없음



## 기본 사용법 

TypeScript에서 선언은 네임스페이스, 타입, 값 중 한 종류 이상의 엔티티를 생성한다. 

- **네임스페이스-생성 선언**: 점으로 구분된 표기법을 사용하여 접근할 이름을 가진 네임스페이스를 생성
- **타입-생성 선언**: 선언된 형태로 표시되며 지정된 이름에 바인딩 된 타입을 생성
-  **값-생성 선언**:  출력된 JavaScript에서 확인할 수 있는 값을 생성

|  선언 타입   | 네임스페이스 | 타입 |  값  |
| :----------: | :----------: | :--: | :--: |
| 네임스페이스 |      X       |      |  X   |
|    클래스    |              |  X   |  X   |
|    열거형    |              |  X   |  X   |
|  인터페이스  |              |  X   |      |
|  타입 별칭   |              |  X   |      |
|     함수     |              |      |  X   |
|     변수     |              |      |  X   |



## 인터페이스 병합(Merging Interfaces)

- 인터페이스 병합: 가장 일반적이고 간단한 선언 병합 유형.가장 기본적인 수준에서, 병합은 각 선언의 멤버를 같은 이름의 인터페이스에 기계적으로 결합한다.

```ts
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

let box: Box = {height: 5, width: 6, scale: 10};
```

- 각 인터페이스의 비-함수 멤버는 고유해야 한다.

  만약 고유하지 않다면, 모두 같은 타입이어야 한다. 인터페이스가 같은 이름이지만 다른 타입을 가진 비-함수 멤버가 있을 경우, 컴파일러는 오류를 일으키기 때문이다.

  ```tsx
  interface Test {
      A: string
  }
  interface Test{
      A :number; //오류
      B: string;
  }
  ```

  

- 함수 멤버의 경우, 이름이 같을 경우 오버로드한다.

  > 1. **개념**
  >    - method의 이름은 같지만 매개변수의 개수는 동일하게 type은 다르게 정의하여 사용하는 방법
  >    - 다른 언어의 Overloading 개념은 method명만 같으면 되지만 typescript는 Overloading을 사용하기 위해서는  함수명과 매개변수의 개수가 같아야 함
  >
  > 2. **사용 이유**
  >
  >    매개변수의 type만 다르고 동작은 동일할 때 코드를 줄이기 위함
  >
  > 3) **형태**
  >
  > ![img](https://blog.kakaocdn.net/dn/4HcAG/btqu0LAJUpm/ViryfEhOMyxZdYEc7r7LwK/img.png)
  >
  > 참고: [오버 라이딩(Overriding)과 오버 로딩(Overloading)](https://doitnow-man.tistory.com/198)

  

- `A` 인터페이스와 나중의 `A` 인터페이스를 병합하는 경우, **두 번째 인터페이스가 더 높은 우선순위**를 가진다.

이 말은, 예를 들어:

```ts
interface Cloner {
    clone(animal: Animal): Animal; //---4
}

interface Cloner {
    clone(animal: Sheep): Sheep; //---3
}

interface Cloner {
    clone(animal: Dog): Dog;  //---1
    clone(animal: Cat): Cat;  //---2
}
```

위의 세 인터페이스는 병합되어 다음과 같이 하나의 새로운 선언을 생성한다.

```ts
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
}
```

- 그룹의 요소는 동일한 순서를 유지하지만,**나중에 병합되어 오버로드 될수록 첫 번째에 위치**한다.
- 예외) 특수 시그니처: 시그니처에 *단일* 문자열 **리터럴 타입**(예: 문자열 리터럴의 유니언이 아닌 경우)인 매개변수가 있을 경우, 시그니처는 병합된 오버로드 목록의 **맨 위**로 올라오게 된다.

<예시>

```ts
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
}
interface Document {
    createElement(tagName: string): HTMLElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

`Document`의 병합된 선언은 다음과 같이 생성한다.

```ts
interface Document {
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```



## 네임스페이스 병합 (Merging Namespaces)

인터페이스와 마찬가지로, 같은 이름을 가진 **네임스페이스도 멤버를 병합**한다. 네임스페이스는 **네임스페이스와 값을 둘 다 생성**한다.

#### 참고

namespace는 전역 네임스페이스에서 이름이 지정된 JavaScript 객체이다. 하나의 독립된 이름 공간을 만들고 여러 파일에 걸쳐 하나의 이름 공간을 공유할 수 있는 **선언 병합** 개념이다. namespace와 비슷한 역할을 하는 것중에는 module이 있는데, ts가 module을 쓰다가 ES2015부터 namespace라는 이름을 표준으로 쓰면서 module은 deprecated 이고, namespace를 사용하게 되었다. 네임스페이스는 모듈과 달리 여러 파일에 걸쳐 있을 수 있으며 이를 `--outFile` 사용하여 연결할 수 있다.([https://typescript-kr.github.io/pages/namespaces-and-modules.html](네임스페이스와 모듈)에서 계속..)

> 타입스크립트 1.5에서 기록해둘 만큼 중요한 명명법 변경이 있었습니다. “내부 모듈(Internal modules)”은 “네임스페이스”가 되었습니다. “외부 모듈(External modules)”은 이제 간단하게 “모듈(modules)”이 되어 [ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/)의 용어와 맞췄습니다. (`module X {`는 `namespace X {`와 동일하며 후자가 선호됩니다.)
>
> 참고) [Namespaces and Modules](https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html)



네임스페이스를 병합하기 위해 

- 각 네임스페이스에 선언된 내보낸 인터페이스로부터 타입 정의가 병합된다.
- 내부에 병합된 인터페이스 정의들이 있는 단일 네임스페이스를 형성된다.



**< `Animals`의 선언 병합>**

```ts
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}
```

<결과>

```ts
namespace Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
}
```

export 되지 않은 멤버는 원래의(병합되지 않은) 네임스페이스에서만 볼 수 있다. 이는 병합된 후에 다른 선언으로부터 합쳐진 멤버는 export 되지 않은 멤버를 볼 수 없다는 것을 의미한다.



<export 되지 않은 멤버 예제> 

```ts
namespace Animal {
    let haveMuscles = true; //다른 네임스페이스에서 엑세스 불가

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

namespace Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // 오류, haveMuscles는 이곳에서 액세스 할 수 없습니다.
    }
}
```

`haveMuscles`이 export 되지 않아서, 동일하게 병합되지 않은 네임스페이스를 공유하는 `animalsHaveMuscles` 함수만 이 심벌을 볼 수 있다. `doAnimalsHaveMuscles` 함수가 병합된 `Animal`의 멤버일지라도, 내보내지 않은 멤버는 볼 수 없다.



## 클래스, 함수, 열거형과 네임스페이스 병합 (Merging Namespaces with Classes, Functions, and Enums)

네임스페이스는 다른 타입의 선언과 병합할 수 있을 정도로 유연하다. 이를 위해선, 네임스페이스의 선언은 병합할 선언을 따라야 합니다. 결과 선언에는 두 타입의 프로퍼티를 모두 가진다.



### 네임스페이스와 클래스 병합 (Merging Namespaces with Classes)

이 부분에서는 내부 클래스를 설명하는 방법을 제공한다.

```ts
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}
```

병합된 멤버의 가시성 규칙은 '네임스페이스 병합' 섹션에서 설명한 것과 같으므로,

-  `AlbumLabel` 클래스를 export 해야 병합된 클래스를 볼 수 있다. 
- 최종 결과물은 다른 클래스 내에서 관리되는 클래스이다.
- 네임스페이스를 사용하여 기존 클래스에 더 많은 정적 멤버를 추가할 수도 있다.



내부 클래스 패턴 이외에도, JavaScript에서 함수를 생성한 후 프로퍼티를 추가하여 함수를 확장하는 것처럼, TypeScript는 선언 병합을 통해 타입을 안전하게 보존하며 정의할 수 있다.

```ts
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

console.log(buildLabel("Sam Smith"));
```

마찬가지로, 네임스페이스를 정적 멤버의 열거형을 확장할 수 있다.

```ts
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```



## 허용되지 않는 병합 (Disallowed Merges)

 **클래스**는 다른 클래스 혹은 변수와 병합할 수 없다. 클래스 병합을 대체하려면, [TypeScript의 믹스인](https://typescript-kr.github.io/pages/mixins.html) 섹션을 참고하자.



## 모듈 보강 (Module Augmentation)

JavaScript는 모듈 병합을 지원하지 않지만, 기존 객체를 가져와 업데이트하여 패치 할 수 있다.

 

<장난감 Observable 예시>

```ts
// observable.ts
export class Observable<T> {
    // ... 구현은 여러분의 몫으로 남겨놓겠습니다 ...
}

// map.ts
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
    // ... 여러분을 위한 또 다른 숙제
}
```

이는 TypeScript에서 잘 동작하지만, 컴파일러는 `Observable.prototype.map`에 대해 모른다. 모듈 보강을 통해 컴파일러에게 정보를 알려줄 수 있다.

```ts
// observable.ts
export class Observable<T> {
    // ... 구현은 여러분의 몫으로 남겨놓겠습니다 ...
}

// map.ts
import { Observable } from "./observable";
declare module "./observable" {
    interface Observable<T> {
        map<U>(f: (x: T) => U): Observable<U>;
    }
}
Observable.prototype.map = function (f) {
    // ... 여러분을 위한 또 다른 숙제
}


// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number>;
o.map(x => x.toFixed());
```

모듈 이름은 `import`/`export`의 모듈 지정자와 같은 방법으로 해석된다. (자세한 내용은 [모듈](https://typescript-kr.github.io/pages/modules.html)을 참고) 그 다음 보강된 선언은 마치 원본과 같은 파일에서 선언된 것처럼 병합된다.



1. 보강에 새로운 최상위 선언을 할 수 없습니다 -- 기존 선언에 대한 패치만 가능하다.
2. default export 는 보강할 수 없으며, 이름을 갖는 export 만 보강할 수 있다.(해당 이름으로 확장시켜야 하며, `default`는 예약어입니다 - 자세한 내용은 [#14080](https://github.com/Microsoft/TypeScript/issues/14080)을 참고)



## 전역 보강 (Global augmentation)

모듈 내부에서 전역 범위에 선언을 추가할 수도 있다:

```ts
// observable.ts
export class Observable<T> {
    // ... 여전히 구현해놓지 않았습니다 ...
}

declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}

Array.prototype.toObservable = function () {
    // ...
}
```

전역 보강 또한 모듈 보강과 동일한 제한 사항을 가진다.



### 참고할 만한 자료



> [TypeScript #6 모듈(Modules)](https://devowen.com/243#%EC%--%B-%EB%B-%--%--%EB%AA%A-%EB%--%--)
>
> **내부 모듈**
>
> **내부 모듈인 네임스페이스는 전역 이름 공간과 분리된 네임스페이스 단위의 이름공간**이다. 따라서 같은 네임스페이스의 이름 공간이라면 파일 B가 파일 A에 선언된 모듈을 참조(reference)할 수 있는데 참조할 때는 별도의 참조문을 선언할 필요가 없다. 같은 네임스페이스 안에서는 이름을 중복해서 클래스, 함수, 변수 등을 선언할 수 없다. 하지만 다른 네임스페이스 간에는 이름이 같아도 충돌이 없다.
>
> **외부 모듈**
>
> export로 선언한 모듈을 외부 모듈이라고 한다. 일반적으로 말하는 모듈은 외부 모듈을 가리킨다. 외부 모듈의 이름 공간과 관련된 특징을 살펴보면 외부 모듈의 이름 공간은 파일 내부로 제한되는 특징이 있다. 예를 들어 export function hello() {}를 hello1.ts과 hello2.ts에서 동시에 선언하는 건 가능하다. 하지만 두 파일 안에서 function hello() {}를 선언하는 건 외부 모듈로 선언하는 것이 아니게 되고, 전역 스코프의 이름 공간을 공유해 이름 충돌이 발생하게 된다.

