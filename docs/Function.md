# 함수 

## 함수 (Function)

```ts
// 기명 함수
fucntion add(x, y) {
  return x + y;
}

// 익명 함수
let myAdd = function(x, y) { return x + y };
```

JavaScript에서처럼, 함수는 함수 외부의 변수를 참조할 수 있다. 이런 경우를, 변수를 캡처(capture) 한다고 한다.

```ts
let z = 100;

function addToZ(x, y) {
  return x + y + z;
}
```

## 함수 타입 (Function Types)

### 함수의 타이핑 (Typing the function)

```ts
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y };
```

TypeScript는 반환 문을 보고 반환 타입을 파악할 수 있으므로 반환 타입을 생략할 수 있다.

### 함수 타입 작성하기 (Writing the function type)

```ts
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

위의 코드 대신 이렇게 쓸 수도 있다:

```ts
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```

만약 함수가 값을 반환하지 않는다면 비워두는 대신 void를 써서 표시한다.

매개변수 타입과 반환 타입만이 함수 타입을 구성한다. 캡처된 변수는 타입에 반영되지 않는다. 사실상 캡처된 변수는 함수의 "숨겨진 상태"의 일부이고 API를 구성하지 않는다.

### 타입의 추론 (Inferring the types)

TypeScript 컴파일러가 방정식의 한쪽에만 타입이 있더라도 타입을 알아낼 수 있다.

```ts
// myAdd는 전체 함수 타입을 가집니다
let myAdd = function(x: number, y: number): number { return  x + y; };

// 매개변수 x 와 y는 number 타입을 가집니다
let myAdd: (baseValue: number, increment: number) => number =
    function(x, y) { return x + y; };
```

이러한 타입 추론 형태를 `contextual typing` 이라 부른다.

## 선택적 매개변수와 기본 매개변수 (Optional and Default Parameter)

TypeScript에서는 모든 매개변수가 함수에 필요하다고 가정한다. **함수가 호출될 때, 컴파일러는 각 매개변수에 대해 사용자가 값을 제공했는지를 검사한다.**

함수에 주어진 인자의 수는 함수가 기대하는 매개변수의 수와 일치해야한다.

```ts
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 오류, 너무 적은 매개변수
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 정확함
```

JavaScript에서는 모든 매개변수가 선택적이고, 사용자는 적합하다고 생각하면 그대로 둘 수 있다. 그렇게 둔다면 그 값은 undefined가 된다. TypeScript에서도 선택적 매개변수를 원한다면 매개변수 이름 끝에 ? 를 붙임으로써 해결할 수 있다.

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");                  // 지금은 바르게 동작
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 정확함
```

TypeScript에서는 유저가 값을 제공하지 않거나 undefined로 했을 때에 할당될 매개변수의 값을 정해 놓을 수도 있다. 이것을 기본-초기화 매개변수라고 한다.

```ts
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 올바르게 동작, "Bob Smith" 반환
let result2 = buildName("Bob", undefined);       // 여전히 동작, 역시 "Bob Smith" 반환
let result3 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result4 = buildName("Bob", "Adams");         // 정확함
```


순수한 선택적 매개변수와는 다르게 기본-초기화 매개변수는 필수 매개변수 뒤에 오는 것이 강요되지 않는다.

```ts
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 오류, 너무 적은 매개변수
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 성공, "Bob Adams" 반환
let result4 = buildName(undefined, "Adams");     // 성공, "Will Adams" 반환
```

## 나머지 매개변수 (Rest Parameters)

TypeScript에서는 인자들을 하나의 변수로 모을 수 있다:

```ts
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

// employeeName 은 "Joseph Samuel Lucas MacKinzie" 가 될것입니다.
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

## this

TypeScript는 몇 가지 기술들로 잘못된 this 사용을 잡아낼 수 있다. Yehuda Katz의 글 [JavaScript 함수 호출과 "this" 이해하기](https://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)를 먼저 읽어보자.

### this와 화살표 함수 (this and arrow functions)

```ts
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

createCardPicker에 의해 생성된 함수에서 사용 중인 this가 deck 객체가 아닌 window에 설정된다.

cardPicker()의 자체적인 호출 때문에 생긴 일이다. 최상위 레벨에서의 비-메서드 문법의 호출은 this를 window로 한다. (Note: strict mode에서는 `this`가 `window` 대신 `undefined` 가 된다.)

### this 매개변수 (this parameter)

```ts
interface Card {
    suit: string;
    card: number;
}
interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}
let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: 아래 함수는 이제 callee가 반드시 Deck 타입이어야 함을 명시적으로 지정합니다.
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

이제 TypeScript는 createCardPicker가 Deck 객체에서 호출된다는 것을 알게 됐다. 이것은 this가 any 타입이 아니라 Deck 타입이며 따라서 --noImplicitThis 플래그가 어떤 오류도 일으키지 않는다는 것을 의미한다.

## 오버로드 (Overloads)

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // 인자가 배열 또는 객체인지 확인
    // 만약 그렇다면, deck이 주어지고 card를 선택합니다.
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // 그렇지 않다면 그냥 card를 선택합니다.
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

오버로드 목록에서 첫 번째 오버로드를 진행하면서 제공된 매개변수를 사용하여 함수를 호출하려고 시도한다. 만약 일치하게 된다면 해당 오버로드를 알맞은 오버로드로 선택하여 작업을 수행한다. 이러한 이유로 가장 구체적인 것부터 오버로드 리스팅을 하는 것이 일반적이다.

위 예제에서 function pickCard(x): any는 오버로드 목록에 해당되지 않음을 유의하자. 그래서 두 가지 오버로드만을 가집니다: 객체를 받는것 하나와 숫자를 받는 것 하나. 다른 매개변수 타입으로 pickCard를 호출하는 것은 오류가 발생한다.

