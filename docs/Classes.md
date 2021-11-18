# 클래스 (Classes)

> https://typescript-kr.github.io/pages/classes.html

간단한 클래스 기반 예제를 살펴보자.

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

- 이 클래스는 3개의 멤버**(greeting 프로퍼티, 생성자, greet 메서드)**를 가지고 있다.

- 클래스 안에서 클래스의 멤버를 참조 할 때 `this.`를 덧붙이는 것은 멤버에 접근한다는 의미이다.
- new를 사용하여 이전에 정의한 생성자를 호출하여 Greeter 형태의 새로운 객체를 만들고, 생성자를 실행해 초기화한다.



## 상속 (Inheritance)

TypeScript에서는, 일반적인 객체-지향 패턴을 사용할 수 있다. 클래스-기반 프로그래밍의 가장 기본적인 패턴 중 하나는 상속을 이용하여 이미 존재하는 클래스를 확장해 새로운 클래스를 만들 수 있다는 것이다.

예제를 살펴보자.

```ts
class Animal {
    move(distanceInMeters: number = 0) {
        console.log(`Animal moved ${distanceInMeters}m.`);
    }
}

class Dog extends Animal {
    bark() {
        console.log('Woof! Woof!');
    }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

상속 기능을 보여주는 가장 기본적인 예제이다. 클래스는 기초 클래스로부터 프로퍼티와 메서드를 상속받는다. 여기서, `Dog`은 `extends` 키워드를 사용하여 `Animal`이라는 *기초* 클래스로부터 파생된 *파생* 클래스이다. 파생된 클래스는 *하위클래스(subclasses)*, 기초 클래스는 *상위클래스(superclasses)* 라고 불리기도 한다.

`Dog`는 `Animal`의 기능을 확장하기 때문에, `bark()`와 `move()`를 모두 가진 `Dog` 인스턴스를 생성할 수 있다.



조금 더 복잡한 예제를 살펴보자.

```ts
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();   //Slithering...
              //Sammy the Python moved 5m.
tom.move(34); //Galloping...
              //Tommy the Palomino moved 34m.

```

1. `extends` 키워드를 사용하여 `Animal`의 하위클래스인 `Horse`와 `Snake`를 생성한다. 

2. 생성자 내에서 `this`에 있는 프로퍼티에 접근하기 전에 `super()`를 먼저 호출해야 한다. 

   이 부분은 TypeScript에서 중요한 규칙이다.

   >  `super()` 는 서브(자식) 클래스에서 상위 클래스를 호출할 때 사용하는 키워드이다.

3. 기초 클래스의 메서드를 하위클래스에 특화된 메서드로 오버라이드한다.

   - `Snake`와 `Horse`는 `Animal`의 `move`를 오버라이드해서 각각 클래스의 특성에 맞게 기능을 가진 `move`를 생성한다. 
   - `tom`은 `Animal`로 선언되었지만 `Horse`의 값을 가지므로 `tom.move(34)`는 `Horse`의 오버라이딩 메서드를 호출한다.



## Public, private 그리고 protected 지정자 (Public, private, and protected modifiers)

### 기본적으로 공개 (Public by default)

-  C#에서 처럼 명시적으로 멤버를 `public`으로 표시할 수도 있으나, TypeScript에서는 기본적으로 각 멤버는 `public` 이다.



### ECMAScript 비공개 필드 (ECMAScript Private Fields)

TypeScript 3.8에서 비공개 필드를 위해 지원된 문법이다. 

```ts
class Animal {
    #name: string;
    constructor(theName: string) { this.#name = theName; }
}

new Animal("Cat").#name; 
// 프로퍼티 '#name'은 비공개 식별자이기 때문에 'Animal' 클래스 외부에선 접근할 수 없다.
```

이 문법은 JavaScript 런타임에 내장되어 있으며, 각각의 비공개 필드의 격리를 더 잘 보장할 수 있다. 현재 TypeScript 3.8 [릴리즈 노트](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#type-only-imports-exports)에 비공개 필드에 대해 자세히 나와있다.



### TypeScript의 `private` 이해하기 (Understanding TypeScript’s `private`)

멤버를 `private`으로 표시하여 멤버를 포함하는 클래스 외부에서 이 멤버에 접근하지 못하도록 할 수 있다. 

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // 오류: 'name'은 비공개로 선언되어 있다.
```

TypeScript는 구조적인 타입 시스템이다. 두개의 다른 타입을 비교할 때 어디서 왔는지 상관없이 모든 멤버의 타입이 호환 된다면, 그 타입들 자체가 호환 가능하다고 말한다. 



하지만 `private` 및 `protected` 멤버가 있는 타입들을 비교할 때는 타입을 다르게 처리한다. 

✅ 호환된다고 판단되는 두 개의 타입 중 **한 쪽에서 `private` 혹은 `protected` 멤버를 가지고 있다면, 다른 한 쪽도 무조건 동일한 선언에 `private` 혹은 `protected` 멤버를 가지고 있어야 한다**. 

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // 오류: 'Animal'과 'Employee'은 호환될 수 없음.
```

- `Animal`과 `Animal`의 하위클래스인 `Rhino`가 있다.
-  `Animal`과 `Rhino`는 `Animal`의 `private name:string`이라는 **동일한 선언으로부터 `private` 부분을 공유**하기 때문에 호환이 가능하다. 
- 하지만   `Employee`는 **`name`이라는 `private` 멤버를 가지고 있지만, `Animal`에서 선언한 것이 아니기 때문에** `Employee`를 `Animal`에 할당할 때, 타입이 호환되지 않다는 오류가 발생한다.



### `protected` 이해하기 (Understanding protected)

`protected` 지정자도 `protected`로 선언된 멤버를 파생된 클래스 내에서 접근할 수 있다는 점만 제외하면 `private`지정자와 매우 유사하게 동작한다.

> `private`과 차이점은 `private`은 프로퍼티를 포함하는 클래스에서만 접근이 가능하지만 `protected`는 그 클래스뿐만 아니라 하위 클래스에서도 접근이 가능 하다는 점이다.

```ts
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // 오류
```

`Person` 외부에서 `name`을 사용할 수 없지만, `Employee`는 `Person`에서 파생되었기 때문에 `Employee`의 인스턴스 메서드 내에서는 여전히 사용할 수 있다.

생성자 또한 `protected`로 표시될 수도 있다. 이는 클래스를 포함하는 클래스 외부에서 인스턴스화 할 수 없지만 확장 할 수 있음을 의미한다.



## 읽기전용 지정자 (Readonly modifier)

`readonly`키워드를 사용하여 프로퍼티를 읽기전용으로 만들 수 있다. 읽기전용 프로퍼티들은 선언 또는 생성자에서 초기화해야하며, 읽기 전용이므로 초기화 후 변경 불가하다.

```ts
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 오류! name은 읽기전용 입니다.
```



## 매개변수 프로퍼티 (Parameter properties)

마지막 예제의 `Octopus` 클래스 내에서 `name`이라는 읽기전용 멤버와 `theName`이라는 생성자 매개변수를 선언했다. 이는 `Octopus`의 생성자가 수행된 후에 `theName`의 값에 접근하기 위해서 필요하다. *매개변수 프로퍼티*를 사용하면 한 곳에서 멤버를 만들고 초기화할 수 있다. 

다음은 매개변수 프로퍼티를 사용한 더 개정된 `Octopus`클래스이다.

```ts
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```

생성자에 짧아진 `readonly name: string` 파라미터를 사용하여 `theName`을 제거하고 `name` 멤버를 생성하고 초기화했습니다. 즉 선언과 할당을 한 곳으로 통합했다.

- 매개변수 프로퍼티는 접근 지정자나 `readonly` 또는 둘 모두를 생성자 매개변수에 접두어로 붙여 선언한다. 
- 매개변수 프로퍼티에 `private`을 사용하면 비공개 멤버를 선언하고 초기한다. 
- 마찬가지로, `public`, `protected`, `readonly`도 동일하게 작용한다.



## 접근자 (Accessors)

getters/setters로 객체의 멤버에 접근하는 방법을 세밀하게 제어할 수 있다. 간단한 클래스를 `get`과 `set`을 사용하도록 변환해보자.

```ts
class Employee {
    fullName: string;
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```



<변환 내용>

- `newName`의 길이를 확인하는 setter를 추가
- `fullName`을 수정하지 않는 간단한 getter도 추가

```ts
const fullNameMaxLength = 10;

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (newName && newName.length > fullNameMaxLength) {
            throw new Error("fullName has a max length of " + fullNameMaxLength);
        }

        this._fullName = newName;
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```



## 전역 프로퍼티 (Static Properties)

인스턴스가 아닌 **클래스 자체에서 보이는 *전역* 멤버를 생성**할 수 있다. 

이 예제에서는 모든 grid의 일반적인 값이기 때문에 origin에 `static`을 사용한다. 인스턴스 접근 앞에 `this.`를 붙이는 것과 비슷하게 **여기선 전역 접근 앞에 `Grid.`를 붙인다.**

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```



## 추상 클래스 (Abstract Classes)

- 추상 클래스는 **다른 클래스들이 파생될 수 있는 기초 클래스**이다.

- ✅**추상 클래스는 직접 인스턴스화할 수 없다**. 추상 클래스는 인터페이스와 달리 멤버에 대한 **구현 세부 정보를 포함**할 수 있다.

- `abstract` 키워드는 추상 클래스뿐만 아니라 **추상 클래스 내에서 추상 메서드**를 정의하는데 사용된다.

  

```ts
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log("roaming the earth...");
    }
}
```



 클래스 내에서 추상으로 표시된 메서드는 구현을 포함하지 않으며 **반드시 파생된 클래스에서 구현**되어야 합니다. 

```ts
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log("Department name: " + this.name);
    }

    abstract printMeeting(): void; // 반드시 파생된 클래스에서 구현되어야 합니다.
}

class AccountingDepartment extends Department {

    constructor() {
        super("Accounting and Auditing"); // 파생된 클래스의 생성자는 반드시 super()를 호출해야 합니다.
    }

    printMeeting(): void {
        console.log("The Accounting Department meets each Monday at 10am.");
    }

    generateReports(): void {
        console.log("Generating accounting reports...");
    }
}

let department: Department; // 추상 타입의 레퍼런스를 생성합니다
department = new Department(); // 오류: 추상 클래스는 인스턴스화 할 수 없습니다
department = new AccountingDepartment(); // 추상이 아닌 하위 클래스를 생성하고 할당합니다
department.printName();
department.printMeeting();
department.generateReports(); // 오류: 선언된 추상 타입에 메서드가 존재하지 않습니다
```



## 고급 기법 (Advanced Techniques)

TypeScript에서는 클래스를 선언하면 실제로 여러 개의 선언이 동시에 생성된다. 



### 생성자 함수 (Constructor functions)

TypeScript에서는 클래스를 선언하면 실제로 여러 개의 선언이 동시에 생성됩니다. 첫 번째로 클래스의 인스턴스 타입이다.

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world""
```

-  `let greeter: Greeter`라고 할 때, `Greeter` 클래스의 인스턴스 타입으로 `Greeter`를 사용한다.
- *클래스의 인스턴스를 `new` 를 하여 생성자 함수*라고 불리는 또 다른 값을 생성하고 있다.



```ts
let Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world"
```

여기서, `let Greeter`는 생성자 함수를 할당받을 것이다. `new`를 호출하고 이 함수를 실행할 때, 클래스의 인스턴스를 얻는다.  또한 생성자 함수는 클래스의 모든 전역 변수들을 포함하고 있다. 각 클래스를 생각하는 또 다른 방법은 *인스턴스* 측면과 *정적* 측면이 있다는 것이다.

```ts
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet()); // "Hello, there"

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet()); // "Hey there!"
```

1) `greeter1`Greeter 클래스를 인스턴스화하고 이 객체를 사용한다.

2) `greeterMaker`라는 새로운 변수를 생성하여, 클래스를 직접 사용합니다. 여기서 이 변수는 클래스 자체를 유지하거나 생성자 함수를 다르게 설명한다.

   여기서 `typeof Greeter`를 사용하여 인스턴스 타입이 아닌 "`Greeter` 클래스 자체의 타입을 제공한다". 혹은 더 정확하게 생성자 함수의 타입인 "`Greeter`라는 심볼의 타입을 제공한다". 

   이 타입은 `Greeter` 클래스의 인스턴스를 만드는 생성자와 함께 Greeter의 모든 정적 멤버를 포함할 것이다. `greeterMaker`에 `new`를 사용함으로써 `Greeter`의 새로운 인스턴스를 생성하고 이전과 같이 호출한다.



## 인터페이스로써 클래스 사용하기 (Using a class as an interface)

앞서 언급한 것처럼, 클래스 선언은 클래스의 인스턴스를 나타내는 타입과 생성자 함수라는 두 가지를 생성한다. 클래스는 타입을 생성하기 때문에 인터페이스를 사용할 수 있는 동일한 위치에서 사용할 수 있다.

```ts
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```

