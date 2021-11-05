# 인터페이스

>  https://typescript-kr.github.io/pages/interfaces.html

TypeScript의 핵심 원칙 중 하나는 타입 검사가 값의 *형태*에 초점을 맞추고 있다는 것이다. 이를 [덕 타이핑(duck typing)](https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91) 혹은 `구조적 서브타이핑 (structural subtyping)`이라고도 한다. TypeScript에서 인터페이스는 이런 타입들의 이름을 짓는 역할을 하고 코드 안의 계약을 정의하는 것뿐만 아니라 프로젝트 외부에서 사용하는 코드의 계약을 정의하는 강력한 방법이다.

```ts
interface LabeledValue {
    label: string;
}

function printLabel(labeledObj: LabeledValue) {
    console.log(labeledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

이 인터페이스는  `문자열` 타입의 `label` 프로퍼티 하나를 가진다는 것을 의미한다. 타입 검사는 프로퍼티들의 순서를 요구하지 않는다. 이 객체가 실제로는 더 많은 프로퍼티를 갖고 있지만, 컴파일러는 *최소한* 필요한 프로퍼티가 있는지와 타입이 잘 맞는지만 검사한다.



## 선택적 프로퍼티 (Optional Properties)

선택적 프로퍼티는 선언에서 프로퍼티 이름 끝에 `?`를 붙여 표시한다. 선택적 프로퍼티의 이점은 **인터페이스에 속하지 않는 프로퍼티의 사용을 방지하면서, 사용 가능한 속성을 기술**하는 것이다. 예를 들어, `createSquare`안의 `color` 프로퍼티 이름을 잘못 입력하면, 오류 메시지로 알려준다.

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    let newSquare = {color: "white", area: 100};
    if (config.clor) {
        // Error: Property 'clor' does not exist on type 'SquareConfig'
        newSquare.color = config.clor;
    }
    //...
}

let mySquare = createSquare({color: "black"});
```



## 읽기전용 프로퍼티 (Readonly properties)

프로퍼티 이름 앞에 `readonly`를 넣어서 **객체가 처음 생성될 때만 프로퍼티를 수정할 수 있도록** 할 수 있다.

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}
```

객체 리터럴을 할당하여 `Point`를 생성한다. 할당 후에는 `x`, `y`를 수정할 수 없다.


TypeScript에서는 `Array<T>`와 동일하나 모든 변경 메서드(Mutating Methods)가 제거된 `ReadonlyArray<T>` 타입을 제공한다.  Readonly 타입을 사용하면 생성 후에 배열을 변경하지 않음을 보장할 수 있다.

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // 오류!
a = ro; // 오류!
```



### 타입 단언 (type assertion)으로 오버라이드

위 예제 마지막 줄에서 `ReadonlyArray`를 일반 배열에 재할당이 불가능한 것을 확인할 수 있다. **타입 단언(type assertion)**으로 오버라이드하는 것은 가능하다.

```ts
a = ro as number[];
```



## 초과 프로퍼티 검사 (Excess Property Checks)

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

위 예제에서 `createSquare`의 매개변수가 `color`대신 `colour`로 전달되었다. TypeScript는 이 코드에 버그가 있을 수 있다고 생각한다. 

객체 리터럴은 다른 변수에 할당할 때나 인수로 전달할 때, 특별한 처리를 받고, **초과 프로퍼티 검사** (excess property checking)를 받는다. 만약 객체 리터럴이 **'대상 타입 (target type)'이 갖고 있지 않은 프로퍼티를 갖고 있으면, 에러가 발생**한다. 

```ts
// error: Object literal may only specify known properties, but 'colour' does not exist in type 'SquareConfig'. Did you mean to write 'color'?
let mySquare = createSquare({ colour: "red", width: 100 });
```

**초과 프로퍼티 검사** 를 피하는 법

1) **타입 단언 사용하기**

   가장 간단한 방법은 타입 단언을 사용하는 것이다.

   ```js
   let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
   ```

   

2) **문자열 인덱스 서명(string index signature)을 추가**

   추가 프로퍼티가 있음을 확신한다면, **문자열 인덱스 서명(string index signature)을 추가**하는 것이 더 나은 방법이다. 만약 `SquareConfig` `color`와 `width` 프로퍼티를 위와 같은 타입으로 갖고 있고, *또한* 다른 프로퍼티를 가질 수 있다면, 다음과 같이 정의할 수 있다.

   ```tsx
   interface SquareConfig {
       color?: string;
       width?: number;
       [propName: string]: any;
   }
   ```

   하지만 여기서는 `SquareConfig`가 여러 프로퍼티를 가질 수 있고, 그 프로퍼티들이 `color`나 `width`가 아니라면, 그것들의 타입은 중요하지 않다.

   

3) **객체를 다른 변수에 할당**

   `squareOptions`가 추가 프로퍼티 검사를 받지 않기 때문에, 컴파일러는 에러를 주지 않는다.

   ```tsx
   let squareOptions = { colour: "red", width: 100 };
   let mySquare = createSquare(squareOptions);
   ```

   `squareOptions`와 `SquareConfig` 사이에 공통 프로퍼티가 있는 경우에만 위와 같은 방법을 사용할 수 있다. 이 예제에서는, `width`가 있지만 **만약에 변수가 공통 객체 프로퍼티가 없으면 에러가 발생**한다. 

   ```ts
   let squareOptions = { colour: "red" };
   let mySquare = createSquare(squareOptions);
   ```

   위처럼 간단한 코드의 경우, 이 검사를 '피하는' 방법을 시도하지 않는 것이 좋다. 메서드가 있고 상태를 가지는 등 더 복잡한 객체 리터럴에서 이 방법을 생각해볼 수 있다. 

   

   하지만 **초과 프로퍼티 에러의 대부분은 실제 버그**다. 그 말은, 만약 옵션 백 같은 곳에서 초과 프로퍼티 검사 문제가 발생하면 타입 정의를 수정해야 할 필요가 있다는 것이다. 예를 들어, 만약 `createSquare`에 `color`나 `colour` 모두 전달해도 괜찮다면, `squareConfig`가 이를 반영하도록 정의를 수정해야 한다.



## 함수 타입 (Function Types)

인터페이스는 JavaScript 객체가 가질 수 있는 넓은 범위의 형태를 기술할 수 있다. 프로퍼티로 객체를 기술하는 것 외에, 인터페이스로 함수 타입을 설명할 수 있다.

인터페이스로 함수 타입을 기술하기 위해, 인터페이스에 **호출 서명 (call signature)**를 전달한다.

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```



함수 타입 인터페이스는 다른 인터페이스처럼 사용할 수 있다.

```ts
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    let result = source.search(subString);
    return result > -1;
}
```



**매개변수의 이름이 같을 필요는 없다**. 예를 들어, 위 예제를 아래와 같이 쓸 수도 있다.

```ts
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
    let result = src.search(sub);
    return result > -1;
}
```

함수 매개변수들은 같은 위치에 대응되는 매개변수끼리 한번에 하나씩 검사한다. 만약 타입을 전혀 지정하지 않았어도 `SearchFunc` 타입의 변수로 직접 함수 값이 할당되었기 때문에 TypeScript의 **문맥상 타이핑 (contextual typing)**이 인수 타입을 추론할 수 있다. 

아래 예제에서 **함수 표현의 반환 타입이 반환하는 값으로 추론**된다. (여기서는 `false`와 `true` 이므로 boolean으로 추론)

```ts
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```



이때 함수 표현식이 숫자 나 문자열을 반환했다면, 타입 검사는 반환 타입이 `SearchFunc` 인터페이스에 정의된 반환 타입과 일치하지 않는다는 에러를 발생시킨다.

```ts
let mySearch: SearchFunc;

// error: Type '(src: string, sub: string) => string' is not assignable to type 'SearchFunc'.
// Type 'string' is not assignable to type 'boolean'.
mySearch = function(src, sub) {
  let result = src.search(sub);
  return "string";
};
```



## 인덱서블 타입 (Indexable Types)

`a[10]` 이나 `ageMap["daniel"]` 처럼 타입을 '인덱스'로 기술할 수 있다. 인덱서블 타입은 인덱싱 할때 해당 반환 유형과 함께 객체를 인덱싱하는 데 사용할 수 있는 타입을 기술하는 `인덱스 시그니처 (index signature)`를 가지고 있다.

```ts
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

위 코드에는 인덱스 서명이 있는 `StringArray` 인터페이스가 있다. 이 인덱스 서명은 `StringArray`가 `number`로 색인화(indexed)되면 `string`을 반환할 것을 나타낸다. 

- 인덱스 서명을 지원하는 타입
  - 문자열
  - 숫자



두 타입의 인덱서(indexer)를 모두 지원하는 것은 가능하지만, 숫자 인덱서에서 반환된 타입은 반드시 **문자열 인덱서에서 반환된 타입의 하위 타입(subtype)**이어야 한다.

`number`로 인덱싱 할 때, JavaScript는 실제로 객체를 인덱싱하기 전에 `string`으로 변환하기에 `100` (`number`)로 인덱싱하는 것은 `"100"` (`string`)로 인덱싱하는 것과 같기 때문이다.

```ts
class Animal {
    name: string;
}
class Dog extends Animal {
    breed: string;
}

// 오류: 숫자형 문자열로 인덱싱을 하면 완전히 다른 타입의 Animal을 얻게 될 것입니다!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}
```

문자열 인덱스 시그니처는 "사전" 패턴을 기술하는데 강력한 방법이지만, 모든 프로퍼티들이 반환 타입과 일치하도록 강제한다. 다음 예제에서, `name`의 타입은 문자열 인덱스 타입과 일치하지 않고, 타입 검사는 에러를 발생시킨다.

```ts
interface NumberDictionary {
    [index: string]: number;
    length: number;    // 성공, length는 숫자입니다
    name: string;      // 오류, `name`의 타입은 인덱서의 하위타입이 아닙니다
}
```



- 인덱스 시그니처가 프로퍼티 타입들의 합집합이라면 다른 타입의 프로퍼티들도 허용할 수 있다.

  ```ts
  interface NumberOrStringDictionary {
      [index: string]: number | string;
      length: number;    // 성공, length는 숫자입니다
      name: string;      // 성공, name은 문자열입니다
  }
  ```

  

- 인덱스의 할당을 막기 위해 인덱스 시그니처를 `읽기 전용`으로 만들 수 있다.

  ```ts
  interface ReadonlyStringArray {
      readonly [index: number]: string;
  }
  let myArray: ReadonlyStringArray = ["Alice", "Bob"];
  myArray[2] = "Mallory"; // 오류! 인덱스 시그니처가 읽기 전용이기 때문에 myArray[2]의 값 할당 불가
  ```

  

## 클래스 타입 (Class Types)

### 인터페이스 구현하기 (Implementing an interface)

아래 예제의 `setTime` 처럼 클래스에 구현된 메서드를 인터페이스 안에서도 기술할 수 있다.

```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date): void;
}

class Clock implements ClockInterface {
    currentTime: Date = new Date();
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

인터페이스는 클래스의 public과 private 모두보다는 public을 기술한다. 그래서 클래스 인스턴스의 private에서는 특정 타입이 있는지 검사할 수 없다.



### 클래스의 스태틱과 인스턴스의 차이점 (Difference between the static and instance sides of classes)

클래스와 인터페이스를 다룰 때, 클래스는 *두 가지* 타입을 가진다.

- 스태틱 타입
- 인스턴스 타입

```ts
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```



생성 시그니처 (construct signature)로 인터페이스를 생성하고, 클래스를 생성하려고 한다면, 인터페이스를 implements 할 때, 에러가 발생한다. 클래스가 인터페이스를 implements 할 때, 클래스의 인스턴스만 검사하기 때문이다. **생성자는 스태틱이기 때문에, 이 검사에 포함되지 않는다.**

따라서 클래스의 스태틱 부분을 직접적으로 다룰 필요가 있다.



1) **생성자 함수로 정의하는 방법** 

   ```ts
   interface ClockConstructor {
       new (hour: number, minute: number): ClockInterface;
   }
   interface ClockInterface {
       tick(): void;
   }
   
   function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
       return new ctor(hour, minute);
   }
   
   class DigitalClock implements ClockInterface {
       constructor(h: number, m: number) { }
       tick() {
           console.log("beep beep");
       }
   }
   class AnalogClock implements ClockInterface {
       constructor(h: number, m: number) { }
       tick() {
           console.log("tick tock");
       }
   }
   
   let digital = createClock(DigitalClock, 12, 17);
   let analog = createClock(AnalogClock, 7, 32);
   ```

   `createClock`의 첫 번째 매개변수는 `createClock(AnalogClock, 7, 32)`안에 `ClockConstructor` 타입이므로, `AnalogClock`이 올바른 생성자 시그니처를 갖고 있는지 검사한다.

   

2) **클래스 표현을 사용하는 방법**

   ```ts
   interface ClockConstructor {
     new (hour: number, minute: number);
   }
   
   interface ClockInterface {
     tick();
   }
   
   const Clock: ClockConstructor = class Clock implements ClockInterface {
     constructor(h: number, m: number) {}
     tick() {
         console.log("beep beep");
     }
   }
   ```

   

## 인터페이스 확장하기 (Extending Interfaces)

클래스처럼 인터페이스들도 확장(extend)이 가능하다. 이는 한 인터페이스의 멤버를 다른 인터페이스에 복사하는 것을 가능하게 해주는데, 인터페이스를 재사용성 높은 컴포넌트로 쪼갤 때, 유연함을 제공한다.

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = {} as Square;
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

인터페이스는 여러 인터페이스를 확장할 수 있어, 모든 인터페이스의 조합을 만들어낼 수 있다.



## 하이브리드 타입 (Hybrid Types)

JavaScript의 동적이고 유연한 특성 때문에, 위에서 설명했던 몇몇 타입의 조합으로 동작하는 객체를 가끔 마주할 수 있다.

그러한 예제 중 하나는 추가적인 프로퍼티와 함께, 함수와 객체 역할 모두 수행하는 객체이다.

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = (function (start: number) { }) as Counter;
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```



## 클래스를 확장한 인터페이스 (Interfaces Extending Classes)

인터페이스 타입이 클래스 타입을 확장하면, 클래스의 멤버는 상속받지만 구현은 상속받지 않는다. 이는 인터페이스가 구현을 제공하지 않고, 클래스의 멤버 모두를 선언한 것과 마찬가지이다. 

- 인터페이스는 기초 클래스의 private과 protected 멤버도 상속받는다. 

- 인터페이스는 private 혹은 protected 멤버를 포함한 클래스를 확장할 수 있으며, 인터페이스 타입은 그 클래스나 하위클래스에 의해서만 구현될 수 있다.

```ts
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class TextBox extends Control {
    select() { }
}

// Error: Property 'state' is missing in type 'Image'.
class Image implements SelectableControl {
    private state: any;
    select() { }
}

class Location {

}
```



위 예제에서 `SelectableControl`은 private `state` 프로퍼티를 포함하여 `Control`의 모든 멤버를 가지고 있다. `state`는 private 멤버이기 때문에, `SelectableControl`를 구현하는 것은 `Control`의 자식만 가능하다.

`Control`의 자식만 같은 선언에서 유래된 `state` private 멤버를 가질수 있기 때문이고, private 멤버들이 호환되기 위해 필요하다.

`Control` 클래스 안에서 `SelectableControl`의 인스턴스를 통해서 `state` private 멤버에 접근할 수 있다. `SelectableControl`은 `select` 메서드를 가진 `Control`과 같은 역할을 한다.

 `Button`과 `TextBox` 클래스들은 `SelectableControl`의 하위타입이지만 (`Control`을 상속받고, `select` 메서드를 가지기 때문에), `Image`와 `Location` 클래스는 아니다.