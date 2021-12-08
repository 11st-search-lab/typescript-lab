# 이터러블 (Iterables)

> https://typescript-kr.github.io/pages/iterators-and-generators.html



객체가 [`Symbol.iterator`](https://www.typescriptlang.org/docs/handbook/symbols.html#symboliterator)프로퍼티에 대한 구현을 가지고 있다면 이터러블로 간주한다.

`Array`, `Map`, `Set`, `String`, `Int32Array`, `Uint32Array` 등과 같은 일부 내장 타입에는 이미 `Symbol.iterator` 프로퍼티가 구현되어 있다.  객체의 `Symbol.iterator` 함수는 반복할 값 목록을 반환한다.

> 자바스크립트에서 **반복자(Iterator)**는 시퀀스를 정의하고 종료시의 반환값을 잠재적으로 정의하는 객체입니다. 더 구체적으로 말하자면, 반복자는 두 개의 속성( `value`, `done`)을 반환하는 next() 메소드 사용하여 객체의 [Iterator protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterator_protocol)을 구현합니다. 시퀀스의 마지막 값이 이미 산출되었다면 `done` 값은 true 가 됩니다. 만약 `value`값이 `done` 과 함께 존재한다면, 그것은 반복자의 반환값이 됩니다.
>
> (출처: [MDN Iterators and generators](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Iterators_and_Generators)) 
>
> 
>
> 이터레이션 프로토콜은 다양한 데이터 공급자가 하나의 순회 방식을 갖도록 규정하여 데이터 소비자가 효율적으로 다양한 데이터 공급자를 사용할 수 있도록 데이터소비자와 데이터 공급자를 연결하는 인터페이스의 역할을 한다. 
>
> - 데이터 소비자: for...of, spread operator, desctructuring, Map, Set 생성자
> - 인터페이스: iteration protocol
> - 데이터 공급자: Array, String, Map, Set, DOM Collection
>
> ![Introduction to javascript iterables, iterators, and generators | by  Mahmoud Felfel | dubizzle Engineering | Medium](https://miro.medium.com/max/1374/1*aw0ODP5vE6ajNe2A3xH8ig.png)
>
> (출처: Modern Javascript Deep Dive, 이터레이션 프로토콜의 필요성, [Introduction to javascript iterables, iterators, and generators](https://medium.com/dubizzletechblog/introduction-to-javascript-iterables-iterators-and-generators-a26be413dfd9))



### 코드 생성 (Code generation)

#### ES5 및 ES3 타게팅 (Targeting ES5 and ES3)

ES5 또는 ES3-호환 엔진을 대상으로 하는 경우, 반복자는 `Array` 유형의 값만 허용한다. 배열이 아닌 값이 `Symbol.iterator` 프로퍼티를 구현하더라도  `for..of` 루프를 사용하면 오류가 발생한다.

- 컴파일러는 `for..of` 루프에 대한 간단한 `for` 루프를 생성한다.

```ts
let numbers = [1, 2, 3];
for (let num of numbers){
 console.log(num);
}
```

- 컴파일 결과 

```ts
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++){
 var num = numbers[_i];
 console.log(num);
}
```



#### ECMAScript 2015 및 상위 버전 타케팅 (Targeting ECMAScript 2015 and higher)

ECMAScipt 2015-호환 엔진을 타케팅하는 경우, 컴파일러는 엔진의 내장 반복자 구현을 대상으로 하는 `for..of` 루프를 생성한다.