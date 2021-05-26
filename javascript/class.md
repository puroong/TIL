# Javascript ES6 Class

* 아래 두 코드는 하는 일이 같음

```javascript
function PrototypicalGreeting(greeting = "Hello", name = "World") {
    this.greeting = greeting;
    this.name = name;
}

PrototypicalGreeting.prototype.greet = function() {
    return `${this.greeting}, ${this.name}`;
}

const greetProto = new PrototypicalGreeting("Hey", "folks");
console.log(greetProto.greet());
```

```javascript
class ClassicalGreeting {
    constructor(greeting = "Hello", name = "World") {
        this.greeting = greeting;
        this.name = name;
    }

    greet() {
        return `${this.greeting}. ${this.name}!`
    }
}

const classyGreeting = new ClassicalGreeting("Hey", "folks");
console.log(classyGreeting.greet());
```

* 클래스와 프로토타입을 혼용할 수 있음 (클래스는 sugar syntax일 뿐임)

```javascript
function Proto() {
    this.name = 'Proto';
    return this;
}

Proto.prototype.getName = function() {
    return this.name;
}

class MyClass extends Proto {
    constructor() {
        super();
        this.name = 'MyClass';
    }
}

const instance = new MyClass();

console.log(instance.getName());

Proto.prototype.getName = function() { return 'Overridden in Proto'; }

console.log(instance.getName());

MyClass.prototype.getName = function() { return 'Overridden in MyClass'; }

console.log(instance.getName());

instance.getName = function() { return 'Overridden in instance'; }

console.log(instance.getName());
```

출력결과

> MyClass
>
> Overridden in Proto
>
> Overridden in MyClass
>
> Overridden in instance

## Prototype vs Class

* class defines type which can be instantiated at runtime, whereas a prototype is itself an object instance

