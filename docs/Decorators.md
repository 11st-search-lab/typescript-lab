# Decorators

> https://typescript-kr.github.io/pages/decorators.html

`TypeScript` 및 `ES6`에 클래스가 도입됨에 따라, 클래스 및 클래스 멤버에 어노테이션을 달거나 수정하기 위해 추가 기능이 필요한 특정 시나리오가있다.

데코레이터는 클래스 선언과 멤버에 어노테이션과 메타-프로그래밍 구문을 추가할 수 있는 방법을 제공한다.

데코레이터는 `JavaScript`에 대한 2단계 제안이며 `TypeScript`의 실험적 기능으로 이용 가능하다.

데코레이터에 대한 실험적 지원을 활성화하려면 명령줄 또는 `tsconfig.json`에서 `experimentDecorators` 컴파일러 옵션을 활성화해야한다.

```json
// tsconfig.json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

## 데코레이터 (Decorators)

데코레이터는 클래스 선언, 메서드, 접근자, 프로퍼티 또는 매개 변수에 첨부할 수 있는 특수한 종류의 선언이다. 데코레이터는 `@expression` 형식을 사용한다. `expression`은 데코레이팅된 선언에 대한 정보와 함께 런타임에 호출되는 함수여야 한다.

예를 들어, 데코레이터 @sealed를 사용하면 다음과 같이 sealed 함수를 작성할 수 있다.

```js
function sealed(target) {
    // 'target' 변수와 함께 무언가를 수행합니다.
}
```

##  데코레이터 팩토리 (Decorator Factories)

데코레이터가 선언에 적용되는 방식을 원하는 대로 바꾸고 싶다면 데코레이터 팩토리를 작성할 수 있다. 데코레이터 팩토리는 단순히 데코레이터가 런타임에 호출할 표현식을 반환하는 함수이다.

```js
function color(value: string) { // 데코레이터 팩토리
    return function (target) { // 데코레이터
        // 'target'과 'value' 변수를 가지고 무언가를 수행합니다.
    }
}
```

## 데코레이터 합성 (Decorator Composition)

다음 예제와 같이 선언에 여러 데코레이터를 적용할 수 있다.

- 단일 행일 경우:

```ts
@f @g x
```

- 여러 행일 경우:

```ts
@f
@g
x
```

여러 데코레이터가 단일 선언에 적용되는 경우는 수학의 합성 함수와 유사하다. 이 모델에서 함수 f와 g을 합성할 때 (f∘g)(x)의 합성 결과는 f(g(x))와 같다.

TypeScript에서 단일 선언에서 여러 데코레이터를 사용할 때 다음 단계가 수행된다.

1. 각 데코레이터의 표현은 위에서 아래로 평가
2. 그런 다음 결과는 아래에서 위로 함수로 호출

Example:

```ts
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}
```

Output:

```sh
f(): evaluated
g(): evaluated
g(): called
f(): called
```

## 데코레이터 평가 (Decorator Evaluation)

- 메서드, 접근자 또는 프로퍼티 데코레이터가 다음에 오는 매개 변수 데코레이터는 각 인스턴스 멤버에 적용
- 메서드, 접근자 또는 프로퍼티 데코레이터가 다음에 오는 매개 변수 데코레이터는 각 정적 멤버에 적용
- 매개 변수 데코레이터는 생성자에 적용
- 클래스 데코레이터는 클래스에 적용

## 클래스 데코레이터 (Class Decorators)

클래스 데코레이터는 클래스 선언 직전에 선언된다.

클래스 데코레이터의 표현식은 데코레이팅된 클래스의 생성자를 유일한 인수로 런타임에 함수로 호출된다.

클래스 데코레이터가 값을 반환하면 클래스가 선언을 제공하는 생성자 함수로 바꾼다.

다음은Greeter 클래스에 적용된 클래스 데코레이터 (@sealed)의 예이다.

```ts
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

@sealed
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

생성자를 재정의하는 방법에 대한 예제는 다음과 같다.

```ts
function classDecorator<T extends {new(...args:any[]):{}}>(constructor:T) {
    return class extends constructor {
        newProperty = "new property";
        hello = "override";
    }
}

@classDecorator
class Greeter {
    property = "property";
    hello: string;
    constructor(m: string) {
        this.hello = m;
    }
}

console.log(new Greeter("world"));
```

Output:

```ts
{
  "property": "property",
  "hello": "override",
  "newProperty": "new property"
} 
```

## 메서드 데코레이터 (Method Decorators)

메서드 데코레이터는 메서드 선언 직전에 선언된다. 데코레이터는 메서드의 프로퍼티 설명자(Property Descriptor) 에 적용되며 메서드 정의를 관찰, 수정 또는 대체하는 데 사용할 수 있다.

메서드 데코레이터의 표현식은 런타임에 다음 세 개의 인수와 함께 함수로 호출된다.

1. 정적 멤버에 대한 클래스의 생성자 함수 또는 인스턴스 멤버에 대한 클래스의 프로토타입입니다.
2. 멤버의 이름
3. 멤버의 프로퍼티 설명자

다음은 Greeter 클래스의 메서드에 적용된 메서드 데코레이터 (@ enumerable)의 예이다.

```ts
function enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.enumerable = value;
    };
}

class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

## 접근자 데코레이터 (Accessor Decorators)

접근자 데코레이터는 접근자 선언 바로 전에 선언된다.

> 참고: TypeScript는 단일 멤버에 대해 get 및 set 접근자를 데코레이팅 할 수 없다. 대신 멤버의 모든 데코레이터를 문서 순서대로 지정된 첫 번째 접근자에 적용해야 한다. 왜냐하면, 데코레이터는 각각의 선언이 아닌 get과 set 접근자를 결합한 프로퍼티 설명자에 적용되기 때문이다.

다음은 Point 클래스의 멤버에 적용되는 접근자 데코레이터 (@configurable)의 예이다.

```ts
function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.configurable = value;
    };
}

class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}
```

## 프로퍼티 데코레이터 (Property Decorators)

이 정보를 사용하여 다음 예와 같이 프로퍼티에 대한 메타데이터를 기록할 수 있다.

```ts
import "reflect-metadata";

const formatMetadataKey = Symbol("format");

function format(formatString: string) {
    return Reflect.metadata(formatMetadataKey, formatString);
}

function getFormat(target: any, propertyKey: string) {
    return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}

class Greeter {
    @format("Hello, %s")
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        let formatString = getFormat(this, "greeting");
        return formatString.replace("%s", this.greeting);
    }
}
```

`@format("Hello, %s")`가 호출되면 `reflect-metadata` 라이브러리의 `Reflect.metadata` 함수를 사용하여 프로퍼티에 대한 메타데이터 항목을 추가한다.

## 매개변수 데코레이터 (Parameter Decorators)

다음은 Greeter 클래스 멤버의 매개 변수에 적용되는 매개 변수 데코레이터 (@required)의 예이다.

```ts
class Greeter {
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }

    @validate
    greet(@required name: string) {
        return "Hello " + name + ", " + this.greeting;
    }
}
```

다음 함수 선언을 사용하여 @required 및 @validate 데코레이터를 정의할 수 있다.

```ts
import "reflect-metadata";

const requiredMetadataKey = Symbol("required");

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
    let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
    existingRequiredParameters.push(parameterIndex);
    Reflect.defineMetadata(requiredMetadataKey, existingRequiredParameters, target, propertyKey);
}

function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {
    let method = descriptor.value;
    descriptor.value = function () {
        let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName);
        if (requiredParameters) {
            for (let parameterIndex of requiredParameters) {
                if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
                    throw new Error("Missing required argument.");
                }
            }
        }

        return method.apply(this, arguments);
    }
}
```

@required 데코레이터는 필요에 따라 매개변수를 표시하는 메타데이터 항목을 추가한다. 그런 다음 @validate 데코레이터는 원래 메서드를 호출하기 전에 인수 유효성 검증하는 함수로 기존의 greet 메서드를 감싼다.

## 메타데이터 (Metadata)

일부 예제는 [실험적 메타데이터 API](https://github.com/rbuckton/reflect-metadata)에 대한 폴리필(polyfill)을 추가하는 reflect-metadata 라이브러리를 사용한다.

```json
// tsconfig.json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

```ts
import "reflect-metadata";

class Point {
    x: number;
    y: number;
}

class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}

function validate<T>(target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<T>) {
    let set = descriptor.set;
    descriptor.set = function (value: T) {
        let type = Reflect.getMetadata("design:type", target, propertyKey);
        if (!(value instanceof type)) {
            throw new TypeError("Invalid type.");
        }
        set.call(target, value);
    }
}
```

다음 TypeScript와 동일하다고 생각할 수 있다.

```ts
class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    @Reflect.metadata("design:type", Point)
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    @Reflect.metadata("design:type", Point)
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}
```