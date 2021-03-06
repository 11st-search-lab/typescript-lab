# JSX

> https://typescript-kr.github.io/pages/jsx.html

JSX는 유효한 JavaScript로 변환되어야 한다. JSX는 React로 큰 인기를 얻었지만, 이후 다른 구현도 등장했다. 
TypeScript는 임베딩, 타입 검사, JSX를 JavaScript로 직접 컴파일하는 것을 지원한다.

## 기본 사용법 (Basic usage)

JSX를 사용하려면 다음 두 가지 작업을 해야한다.

- 1. 파일 이름을 .tsx 확장자로 지정합니다.
- 2. jsx 옵션을 활성화합니다.

TypeScript는 `preserve`, `react` 및 `react-native`라는 세 가지의 JSX 모드와 함께 제공된다.
`react` 모드는 `React.createElement`를 생성하여, 사용하기 전에 JSX 변환이 필요하지 않으며, 결과물은 .js 확장자를 갖게된다.

<img src="https://user-images.githubusercontent.com/22424891/145334093-7e523b4e-b36b-4199-b89f-000306c2230d.png" />

> *참고: React JSX를 생성할 때 --jsxFactory 옵션으로 사용할 JSX 팩토리(JSX factory) 함수를 지정할 수 있다. (기본값은 React.createElement)

## `as` 연산자 (The `as` operator)

```ts
var foo = <foo>bar;
```

TypeScript는 꺾쇠 괄호를 사용해 타입을 단언하기 때문에, `JSX` 구문과 함께 사용할 경우 특정 문법 해석에 문제가 될 수도 있다. TypeScript는 `.tsx` 파일에서 화살 괄호를 통한 타입 단언을 허용하지 않는다.

위의 구문은 .tsx 파일에서 사용할 수 없으므로, as라는 대체 연산자를 통해 타입 단언을 해야한다.

```ts
var foo = bar as foo;
```

## 타입 검사 (Type Checking)

TypeScript는 React와 동일한 규칙을 사용하여 구별한다. 내장 요소는 항상 소문자로 시작하고 값-기반 요소는 항상 대문자로 시작한다.

## 내장 요소 (Intrinsic elements)

내장 요소는 특수 인터페이스 `JSX.IntrinsicElements`에서 조회된다. 기본적으로 이 인터페이스가 지정되지 않으면 그대로 진행되어 내장 요소 타입은 검사되지 않는다. 그러나 이 인터페이스가 있는 경우, 내장 요소의 이름은 `JSX.IntrinsicElements` 인터페이스의 프로퍼티로 조회된다.

```ts
declare namespace JSX {
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // 성공
<bar />; // 오류
```

## 값-기반 요소 (Value-based elements)

값-기반 요소는 해당 스코프에 있는 식별자로 간단하게 조회된다.

```ts
import MyComponent from "./myComponent";

<MyComponent />; // 성공
<SomeOtherComponent />; // 오류
```

값-기반 요소를 정의하는데엔 다음의 두 가지 방법이 있다.

- 1. 함수형 컴포넌트 (FC)
- 2. 클래스형 컴포넌트

이 두 가지 유형의 값-기반 요소는 JSX 표현식에서 서로 구별할 수 없으므로, TS는 오버로드 해결을 사용하여 먼저 함수형 컴포넌트 표현식으로 해석한다. 이 과정이 성공적으로 진행되면, TS는 이 선언을 표현식으로 해석한다. 함수형 컴포넌트로 해석되지 않는다면, TS는 클래스형 컴포넌트로 해석을 시도한다. 이 과정도 실패할 경우, TS는 오류를 보고한다.

## 함수형 컴포넌트 (Function Component)

이름에서 알 수 있듯, 컴포넌트는 첫 번째 인수가 props 객체인 JavaScript 함수로 정의된다. TS는 컴포넌트의 반환 타입이 `JSX.Element`에 할당 가능하도록 요구한다.

```ts
interface FooProp {
  name: string;
  X: number;
  Y: number;
}

declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
  return <AnotherComponent name={prop.name} />;
}

const Button = (prop: {value: string}, context: { color: string }) => <button>
```

함수형 컴포넌트는 JavaScript 함수이므로, 함수 오버로드 또한 사용 가능하다.

```ts
interface ClickableProps {
  children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
  home: JSX.Element;
}

interface SideProps extends ClickableProps {
  side: JSX.Element | string;
}

function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
  ...
}
```

> 참고: 함수형 컴포넌트는 이전에 무상태 함수형 컴포넌트(SFC)로 알려져 있다. 하지만 최근 버전의 리액트에선 더 이상 함수형 컴포넌트를 무상태로 취급하지 않으며, SFC 타입과 그 별칭인 StatelessComponent은 더 이상 사용되지 않는다.

## 클래스형 컴포넌트 (Class Component)

```ts
class MyComponent {
  render() {}
}

// 생성자 시그니처 사용
var myComponent = new MyComponent();

// 요소 클래스 타입 => MyComponent
// 요소 인스턴스 타입 => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {
    }
  }
}

// 호출 시그니처 사용
var myComponent = MyFactoryFunction();

// 요소 클래스 타입 => MyFactoryFunction
// 요소 인스턴스 타입 => { render: () => void }
```

흥미롭게도 요소 인스턴스 타입은 `JSX.ElementClass`에 할당 가능해야 하며, 그렇지 않을 경우 오류를 일으킨다. 기본적으로 `JSX.ElementClass`는 `{}`이지만, 적절한 인터페이스에 적합한 타입으로만 `JSX`를 사용하도록 제한할 수 있다.

```ts
declare namespace JSX {
  interface ElementClass {
    render: any;
  }
}

class MyComponent {
  render() {}
}
function MyFactoryFunction() {
  return { render: () => {} }
}

<MyComponent />; // 성공
<MyFactoryFunction />; // 성공

class NotAValidComponent {}
function NotAValidFactoryFunction() {
  return {};
}

<NotAValidComponent />; // 오류
<NotAValidFactoryFunction />; // 오류
```

## 속성 타입 검사 (Attribute type checking)

속성 타입 검사를 위해선 첫 번째로 요소 속성 타입 을 결정해야 한다. 내장 요소의 경우, 요소 속성 타입은 `JSX.IntrinsicElements`의 프로퍼티 타입과 동일하다.

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// 'foo'의 요소 속성 타입은 '{bar?: boolean}'
<foo bar />;
```

값-기반 요소의 경우에는 이전에 요소 인스턴스 타입 의 프로퍼티 타입에 따라 결정된다. 사용할 프로퍼티는 `JSX.ElementAttributesProperty`에 따라 결정된다.

```ts
declare namespace JSX {
  interface ElementAttributesProperty {
    props; // 사용할 프로퍼티 이름을 지정
  }
}

class MyComponent {
  // 요소 인스턴스 타입의 프로퍼티를 지정
  props: {
    foo?: string;
  }
}

// 'MyComponent'의 요소 속성 타입은 '{foo?: string}'
<MyComponent foo="bar" />
```

## 자식 타입 검사 (Children Type Checking)

TypeScript 2.3부터, TS는 자식 의 타입 검사를 도입했다. `JSX.ElementChildrenAttribute`를 사용해 해당 props 내의 자식 의 이름을 결정한다. `JSX.ElementChildrenAttribute`는 단일 프로퍼티로 선언되어야한다.

```ts
declare namespace JSX {
  interface ElementChildrenAttribute {
    children: {};  // 사용할 자식의 이름을 지정
  }
}
```

```ts
<div>
  <h1>Hello</h1>
</div>;

<div>
  <h1>Hello</h1>
  World
</div>;

const CustomComp = (props) => <div>{props.children}</div>
<CustomComp>
  <div>Hello World</div>
  {"This is just a JS expression..." + 1000}
</CustomComp>
```

다른 속성처럼 자식 의 타입도 지정할 수 있다.

```ts
interface PropsType {
  children: JSX.Element
  name: string
}

class Component extends React.Component<PropsType, {}> {
  render() {
    return (
      <h2>
        {this.props.children}
      </h2>
    )
  }
}

// 성공
<Component name="foo">
  <h1>Hello World</h1>
</Component>

// 오류 : 자식은 JSX.Element의 배열이 아닌 JSX.Element 타입입니다.
<Component name="bar">
  <h1>Hello World</h1>
  <h2>Hello World</h2>
</Component>

// 오류 : 자식은 JSX.Element의 배열이나 문자열이 아닌 JSX.Element 타입입니다.
<Component name="baz">
  <h1>Hello</h1>
  World
</Component>
```

## JSX 결과 타입 (The JSX result type)

기본적으로 JSX 표현식의 결과물은 `any` 타입이다.

## 표현식 포함하기 (Embedding Expressions)

JSX는 중괄호({ })로 표현식을 감싸 태그 사이에 표현식 사용을 허용한다.

```ts
var a = <div>
  {["foo", "bar"].map(i => <span>{i / 2}</span>)}
</div>
```

위의 코드는 문자열을 숫자로 나눌 수 없으므로 오류를 일으킵니다. preserve 옵션을 사용할 때, 결과는 다음과 같다.

```ts
var a = <div>
  {["foo", "bar"].map(function (i) { return <span>{i / 2}</span>; })}
</div>
```

## 리액트와 통합하기 (React integration)

리액트에서 TSX를 사용하기 위해선 `@types/react`를 사용해야한다. 이는 리액트를 사용할 수 있도록 JSX 네임스페이스를 적절하게 정의한다.

```ts
/// <reference path="react.d.ts" />

interface Props {
  foo: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent foo="bar" />; // 성공
<MyComponent foo={0} />; // 오류
```

## 팩토리 함수 (Factory Functions)

jsx: react 컴파일러 옵션에서 사용하는 팩토리 함수는 설정이 가능하다.

주석 pragma 버전은 다음과 같이 사용할 수 있다.(TypeScript 2.8 기준):

```ts
import preact = require("preact");
/* @jsx preact.h */
const x = <div />;
```

이는 다음처럼 생성된다.

```ts
const preact = require("preact");
const x = preact.h("div", null);
```

팩토리가 `React.createElement(기본값)`로 정의되어 있다면, 컴파일러는 전역 JSX를 검사하기 전에 `React.JSX`를 먼저 검사할 것이다.
팩토리가 `h`로 정의되어 있다면, 컴파일러는 전역 JSX를 검사하기 전에 `h.JSX`를 검사할 것이다.

---

## 참고

- https://egas.tistory.com/32
