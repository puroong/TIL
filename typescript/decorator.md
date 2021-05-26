# Decorator

* 아직 typescript가 제공하는 experimental feature임
    * 사용하기 위해선 experimentalDecorators 옵션을 사용해야함

* Decorator는 클래스, 메소드, 점근자, 프로퍼티, 파라미터와 같이 사용할 수 있음

* Decorators use the form `@expression`, where `expression` must evaluate to a function that will be called at runtime with information about the decorated declaration 


## Decorator Factories

```typescript
function color(value: string) {
    return function(target: any) {
        // use target and value
    }
}
```

## Decorator Composition

* decorator 함성도 가능함

```typescript
@f @g x
```

```typescript
@f
@g
x
```

* As such, the following steps are performed when evaluating multiple decorators on a single declaration in Typescript:
    1. The expression for each decorator are evaluated top-to-bottom
    2. The results are then called as functions from bottom-to-top

## Decorator Evaluation

* There is a well defined order to how decorators applied to various declarations inside of a class are applied
    1. Parameter Decorators, followed by Method, Accessor, or Property Decorators are applied for each instance member
    2. Parameter Decorators, followed by Method, Accessor, or Property Decorators are applied for each static member
    3. Parameter Decorators are applied for the constructor
    4. Class Decorators are applied for the class

## Class Decorators

* class decorator는 클래스 생성자에 적용되고, 클래스 정의를 수정하는데 사용됨

* The value that class decorator return will replace class declaration with the provided constructor function

## Method Decorators

* 메소드 바로 전에 선언됨

* method decorator는 인자로 다음 3가지 값을 받음
    * static member라면 constructor function / class, instance member라면 class의 prototype
    * member 이름
    * member의 property descriptor

## Accessor Decorators

* accessor 선언 바로 전에 선언됨

* accessor decorator는 인자로 다음 3가지 값을 받음
    * static member라면 constructor function / class, instance member라면 class의 prototype
    * member 이름
    * member의 property descriptor

## Property Decorators

* property 선언 전에 선언됨

* property decorator는 인자로 다음 2가지 값을 받음
    * static member라면 constructor function / class, instance member라면 class의 prototype
    * member 이름

* descriptor가 인자로 주어지지 않은 이유는 ???

## Parameter Decorators

* 파리미터 선언 전에 선언됨

* parameter decorator는 인자로 다음 3가지 값을 받음
    * static member라면 constructor function / class, instance member라면 class의 prototype
    * member 이름
    * 함수에서 해당 parameter 순서 인덱스

* parameter decorator 반환값은 무시됨