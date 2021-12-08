# Mixin

> https://typescript-kr.github.io/pages/mixins.html



## 소개(Introduction)

재사용 가능한 컴포넌트로 부터 클래스를 빌드하는 또 다른 일반적인 방법으로, 간단한 부분클래스를 결합하여 빌드하는 패턴입니다. 

만약 FullStack 클래스를 FrontEnd 클래스와 BackEnd 클래스를 상속받아 구현한다 했을 때, 아래와 같은 코드로 구현하려 할 것이다.

```tsx
class FullStack extends FrontEnd, BackEnd {
  ...
}
```

하지만 타입스크리트는 위 방식처럼 여러 개의 클래스를 한번에 확장하는 기능을 지원하지 않는다. 

이에 대한 해결책으로 mixin 함수를 작성해서 mixin 함수 내에서 클래스를 선언하고, 그 클래스를 다른 클래스로 확장한 후 리턴하도록 하는 방식이 mixin이다. 



```tsx
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}

//mixin 함수

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            Object.defineProperty(derivedCtor.prototype, name, Object.getOwnPropertyDescriptor(baseCtor.prototype, name));
        });
    });
}

//mix 예시
applyMixins(SmartObject, [Disposable, Activatable]);
```



## 참고할 만한 자료 

[TypesScript Deep Dive 믹스인(Mixin)](https://radlohead.gitbook.io/typescript-deep-dive/type-system/mixins)

[[TS]타입스크립트를 통해 여러 개의 클래스로 확장하기(extends to multiple classes, mixin class)](https://krpeppermint100.medium.com/ts-%EC%97%AC%EB%9F%AC-%EA%B0%9C%EC%9D%98-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%A1%9C-%ED%99%95%EC%9E%A5%ED%95%98%EA%B8%B0-extends-to-multiple-classes-mixin-class-fe6cf212881b)