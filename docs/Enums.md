# Enums

> https://typescript-kr.github.io/pages/enums.html

열거형으로 이름이 있는 상수들의 집합을 정의할 수 있다. TypeScript는 숫자와 문자열-기반 열거형을 제공한다.

## 숫자 열거형 (Numeric enums)

열거형은 `enum` 키워드를 사용해 정의할 수 있다.

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right,
}
```

위 코드에서 Up이 1 로 초기화된 숫자 열거형을 선언했다. 그 지점부터 뒤따르는 멤버들은 자동으로-증가된 값을 갖는다. 

원한다면, 전부 초기화 하지 않을 수도 있다. 이 경우 초기화 값이 0 부터 진행된다.

```ts
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

열거형 사용은 하기와 같다.

```ts
enum Response {
    No = 0,
    Yes = 1,
}

function respond(recipient: string, message: Response): void {
    // ...
}

respond("Princess Caroline", Response.Yes)
```

초기화되지 않은 열거형이 먼저 나오거나, 숫자 상수 혹은 다른 상수 열거형 멤버와 함께 초기화된 숫자 열거형 이후에 와야한다.

```ts
enum E {
    A = getSomeValue(),
    B, // 오류! 앞에 나온 A가 계산된 멤버이므로 초기화가 필요합니다.
}
```

## 문자열 열거형 (String enums)

문자열 열거형에서 각 멤버들은 문자열 리터럴 또는 다른 문자열 열거형의 멤버로 상수 초기화 해야한다.

```ts
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

## 이종 열거형 (Heterogeneous enums)

기술적으로 열거형은 숫자와 문자를 섞어서 사용할 수 있지만 굳이 그렇게 할 이유는 없다.

```ts
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

**확실하게 JavaScript 런타임에서 장점을 취하려는 것이 아니라면, 사용하지 않는 것을 권장한다.**

## 계산된 멤버와 상수 멤버 (Computed and constant members)

각 열거형의 멤버는 상수이거나 계산된 값일 수 있다. 열거형의 멤버는 아래의 경우 상수로 간주한다.

- 열거형의 첫 번째 데이터이며 초기화 값이 없는 경우, 0으로 값이 할당된다.

```TS
// E.X는 상수입니다:
enum E { X }
```

- 초기화 값이 없으며 숫자 상수로 초기화된 열거형 멤버 뒤에 따라 나오는 경우. 앞에 나온 상수 값에 1씩 증가한 값을 상수로 갖습니다.

```ts
// 'E1' 과 'E2' 의 모든 열거형 멤버는 상수입니다.

enum E1 { X, Y, Z }

enum E2 {
    A = 1, B, C
}
```

열거형 멤버는 상수 열거형 표현식으로 초기화된다. 상수 열거형 표현식은 컴파일 시 알아낼 수 있는 TypeScript 표현식의 일부이다. 아래의 경우 상수 열거형 표현식이라고 한다.

- 리터럴 열거형 표현식 (기본적으로 문자 리터럴 또는 숫자 리터럴)
- 이전에 정의된 다른 상수 열거형 멤버에 대한 참조 (다른 열거형에서 시작될 수 있음)
- 괄호로 묶인 상수 열거형 표현식
- 상수 열거형 표현식에 단항 연산자 `+`, `-`, `~` 를 사용한 경우
- 상수 열거형 표현식을 이중 연산자 `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^` 의 피연산자로 사용할 경우
- 상수 열거형 표현식 값이 NaN 이거나 Infinity 이면 컴파일 시점에 오류가 납니다.

이외 다른 모든 경우 열거형 멤버는 계산된 것으로 간주한다.

```ts
enum FileAccess {
    // 상수 멤버
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // 계산된 멤버
    G = "123".length
}
```

## 유니언 열거형과 열거형 멤버 타입 (Union enums and enum member types)

열거형의 모든 멤버가 리터럴 열거형 값을 가지면 특별한 의미로 쓰이게된다.

첫째로 열거형 멤버를 타입처럼 사용할 수 있다.

```ts
enum ShapeKind {
    Circle,
    Square,
}

interface Circle {
    kind: ShapeKind.Circle;
    radius: number;
}

interface Square {
    kind: ShapeKind.Square;
    sideLength: number;
}

let c: Circle = {
    kind: ShapeKind.Square, // 오류! 'ShapeKind.Circle' 타입에 'ShapeKind.Square' 타입을 할당할 수 없습니다.
    radius: 100,
}
```

또 다른 점은 열거형 타입 자체가 효율적으로 각각의 열거형 멤버의 유니언이 돼서, TypeScript는 값을 잘못 비교하는 어리석은 버그를 잡을 수 있다.

```ts
enum E {
    Foo,
    Bar,
}

function f(x: E) {
    if (x !== E.Foo || x !== E.Bar) {
        //             ~~~~~~~~~~~
        // 에러! E 타입은 Foo, Bar 둘 중 하나이기 때문에 이 조건은 항상 true를 반환합니다.
    }
}
```

## 런타임에서 열거형 (Enums at runtime)

열거형은 런타임에 존재하는 실제 객체이다. 예를 들어 아래와 같은 열거형은

```ts
enum E {
    X, Y, Z
}
```

실제로 아래와 같이 함수로 전달될 수 있다.

```ts
function f(obj: { X: number }) {
    return obj.X;
}

// E가 X라는 숫자 프로퍼티를 가지고 있기 때문에 동작하는 코드입니다.
f(E);
```

## 컴파일 시점에서 열거형 (Enums at compile time)

`keyof typeof` 를 사용하면 모든 열거형의 키를 문자열로 나타내는 타입을 가져온다.

```ts
enum LogLevel {
    ERROR, WARN, INFO, DEBUG
}

/**
 * 이것은 아래와 동일합니다. :
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel;

function printImportant(key: LogLevelStrings, message: string) {
    const num = LogLevel[key];
    if (num <= LogLevel.WARN) {
       console.log('Log level key is: ', key);
       console.log('Log level value is: ', num);
       console.log('Log level message is: ', message);
    }
}
printImportant('ERROR', 'This is a message');
```

## 역 매핑 (Reverse mappings)

```ts
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

TypeScript는 아래와 같은 JavaScript 코드로 컴파일한다.

```ts
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[a]; // "A"
```

이렇게 생성된 코드에서, 열거형은 정방향 (`name` -> `value`) 매핑과 역방향 (`value` -> `name`) 매핑 두 정보를 모두 저장하는 객체로 컴파일된다.

**문자열 열거형은 역 매핑을 생성하지 않는다.**

## const 열거형 (const enums)

열거형 값에 접근할 때, 추가로 생성된 코드 및 추가적인 간접 참조에 대한 비용을 피하기 위해 const 열거형을 사용할 수 있다.

```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

const 열거형은 상수 열거형 표현식만 사용될 수 있으며 일반적인 열거형과 달리 컴파일 과정에서 완전히 제거된다. 

```ts
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

위 코드는 아래와 같이 컴파일된다.

```ts
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

## Ambient 열거형 (Ambient enums)

Ambient 열거형은 이미 존재하는 열거형 타입의 모습을 묘사하기 위해 사용된다. TypeScript에서는 컴파일러가 알아야 할 사항을 "선언"할 수 있지만 실제로 코드를 내보내지는 않는다.

```ts
declare enum Enum {
    A = 1,
    B,// 초기화가 되어 있으면 상수, 그렇지 않으면 계산된 멤버로 간주!
    C = 2
}
```