# Literal Types

> https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types

리터럴 타입은 집합 타입의 보다 구체적인 하위 타입이다. 이것이 의미하는 바는 타입 시스템 안에서 "Hello World"는 string이지만, string은 "Hello World"가 아니란 것이다.

오늘날 TypeScript에는 문자열과 숫자, 두 가지 리터럴 타입이 있는데 이를 사용하면 문자열이나 숫자에 정확한 값을 지정할 수 있다.

## 리터럴 타입 좁히기 (Literal Narrowing)

`var` 또는 `let`으로 변수를 선언할 경우 이 변수의 값이 변경될 가능성이 있음을 컴파일러에게 알린다. 반면, `const`로 변수를 선언하게 되면 TypeScript에게 이 객체는 절대 변경되지 않음을 알린다.

```ts
// const를 사용하여 변수 helloWorld가
// 절대 변경되지 않음을 보장합니다.

// 따라서, TypeScript는 문자열이 아닌 "Hello World"로 타입을 정합니다.
const helloWorld = "Hello World";
// >> const helloWorld: "Hello World"

// 반면, let은 변경될 수 있으므로 컴파일러는 문자열이라고 선언할 것입니다.
let hiWorld = "Hi World";
// >> let hiWorld: string
```

무한한 수의 잠재적 케이스들 (문자열 값은 경우의 수가 무한대)을 유한한 수의 잠재적 케이스 (helloWorld의 경우: 1개)로 줄여나가는 것을 타입 좁히기 (narrowing)라 한다.

리터럴 유형은 그 자체로는 가치가 없다.

```ts
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
Type '"howdy"' is not assignable to type '"hello"'.
```

문자열 리터럴 타입은 유니언 타입, 타입 가드 그리고 타입 별칭과 잘 결합된다.

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

숫자 리터럴 유형은 동일한 방식으로 작동한다.

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

물론, 이것들을 리터럴이 아닌 유형과 결합할 수 있다.

```ts
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

부울 리터럴이라는 리터럴 유형이 한 가지 더 있다. 부울 리터럴 유형은 두 가지뿐이며 짐작할 수 있듯이 `true`와 `false` 두가지 유형이다.

## 문자적 추론

```ts
const req = { url: "https://example.com", method: "GET" };
const handleRequest = (url: string, method:"GET" | "POST") => {
  return [url, method]
}
handleRequest(req.url, req.method);
// Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.(2345)
```

req.method은 "GET"이 아닌 string으로 추론된다.

해결 방법은 두 가지가 있다.

1. 타입 단언(type assertion)을 추가한다. `Change 1` or `Change 2` 선택

```ts
// Change 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
handleRequest(req.url, req.method as "GET");
```

2. `as const`를 이용하여 객체의 타입 전체를 리터럴 타입으로 변경시켜준다.

```ts
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

