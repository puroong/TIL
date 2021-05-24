# cheeket.js 코드 공부

(개인적으로 공부한거라 틀린 부분이 있을 수 있습니다)

## Usage

1. create container
2. bind provider
3. resolve provider

Example.

```typescript
const katanaProvider = () => new Katana();

const shurikenProvider = () => new Shuriken();

const ninjaProvider = async (context: interfaces.Context) => {
  return new Ninja(
    await context.resolve<Weapon>(Types.Weapon),
    await context.resolve<ThrowableWeapon>(Types.ThrowableWeapon)
  );
};

const container = new Container();

container.bind(Types.Weapon, inRequestScope(katanaProvider));
container.bind(Types.ThrowableWeapon, inRequestScope(shurikenProvider));
container.bind(Types.Warrior, inRequestScope(ninjaProvider));

const warrior = await container.resolve<Warrior>(Types.Warrior);
const throwableWeapon = await container.resolve<ThrowableWeapon>(
  Types.ThrowableWeapon
);
const weapon = await container.resolve<Weapon>(Types.Weapon);
```

## Container 코드 분석

bind (container.ts)

```typescript
bind<T>(token: interfaces.Token<T>, provider: interfaces.Provider<T>): void {
    this.#bindingDictionary.set(token, provider);
}
```

token, provider를 각각 key-value로 사용하여 딕셔너리에 저장함

resolve (container.ts)

```typescript
async resolve<T>(token: interfaces.Token<T>): Promise<T> {
    const provider = this.#bindingDictionary.get(token);
    if (provider !== undefined) {
        return this.resolveProvider(provider, token);
    }

    throw new CantResolveError(token, this);
}
```

딕셔너리에 token을 카로 가지고 있는 값이 있으면 resolveProvider 호출함

resolveProvider (container.ts)

```typescript
private async resolveProvider<T>(
    provider: interfaces.Provider<T>,
    token: interfaces.Token<T>
): Promise<T> {
    const context = this.createContext(token);
    const value = await provider(context);
    context.request.resolved = value;

    await this.emitAsync(EventType.Resolve, context);

    return value;
}
```

Context 생성하고 Provider 리턴값을 반환

createContext (container.ts)

```typescript
private createContext<T>(token: interfaces.Token<T>): interfaces.Context {
    const request = new Request(token);
    return new Context([this.#containerContext], request);
}
```

Request 생성하고 Context 생성한 뒤 이 값 반횐함

request.ts

```typescript
import uniqid from "uniqid";
import * as interfaces from "../interfaces";

class Request<T> implements interfaces.Request<T> {
  id = Symbol(uniqid());

  resolved?: T | T[] = undefined;

  constructor(public token: interfaces.Token<T>) {}
}

export default Request;
```

Request는 고유 id와 resovled를 프로퍼티로 가짐

**Provider를 resolve하는 행위를 Request라 하는 것 같음**

constructor (context.ts)

```typescript
class Context implements interfaces.Context {
  readonly id = Symbol(uniqid());

  readonly #containerContexts: interfaces.ContainerContext[];

  public readonly container: interfaces.ContextRequester;

  public readonly children = new Set<interfaces.Context>();

  constructor(
    containerContexts: interfaces.ContainerContext[],
    public request: interfaces.Request<unknown>,
    public parent?: interfaces.Context
  ) {
    this.#containerContexts = containerContexts;

    const current = containerContexts[0];
    if (current === undefined) {
      throw new Error("There must be at least one container.");
    }
    this.container = current.contextRequester;
  }
...
```

Container Context 목록, Request, 부모 Context를 인자로 받으며, Container Context 목록 중 가장 첫번째 값의 컨테이너를 container 프로퍼티에 저장함

Request, 부모 Context도 저장함

**Container Context를 목록으로 받는 이유? 어떻게 활요할 수 있는지?**

예시에 있는 shurikenProvider나 katanaProvider처럼 Context를 사용하지 않으면 단순히 인자로 주어진 토큰을 키로 가지고 있는 값을 딕셔너리에서 조회한 뒤 반환함

ninjaProvider처럼 컨테이너에서 바인딩된 의존성을 주입하고 싶은 경우 Context의 메소드들을 사용해야함

resolve (context.ts)

```typescript
async resolve<T>(token: interfaces.Token<T>): Promise<T> {
    const [provider, containerIndex] = this.findProviderAndContainerIndex(
        token
    );
    return this.resolveProvider(provider, token, containerIndex);
}
```

findProviderAndContainerIndex (context.ts)

```typescript
private findProviderAndContainerIndex<T>(
    token: interfaces.Token<T>
): [interfaces.Provider<T>, number] {
    // eslint-disable-next-line no-plusplus
    for (let i = 0; i < this.#containerContexts.length; i++) {
        const provider = this.#containerContexts[i].bindingDictionary.get(token);
        if (provider !== undefined) return [provider, i];
    }

    throw new CantResolveError(token, this);
}
```

resolveProvider (context.ts)

```typescript
private async resolveProvider<T>(
    provider: interfaces.Provider<T>,
    token: interfaces.Token<T>,
containerIndex: number
): Promise<T> {
    const context = this.createChild(token, containerIndex);
    const value = await provider(context);
    context.request.resolved = value;

    await this.#containerContexts[containerIndex].contextRequester.emitAsync(
        EventType.Resolve,
        context
    );

    return value;
}
```

자식 Context를 생성한 뒤 Provier 리턴 값을 반환함

createChild (context.ts)

```typescript
private createChild<T>(
    token: interfaces.Token<T>,
    containerIndex: number
): Context {
    const request = new Request(token);
    const context = new Context(
        this.#containerContexts.slice(containerIndex),
        request,
        this
    );

    this.children.add(context);

    return context;
}
```

Request, Context를 생성하고 children 프로퍼티에 새로 생성한 Context 저장함

**slice 하는 이유? Container Context 인자로 넘길 때 배열에 저장된 순서도 중요한건지?**

## 용어 정리

* Provider
    * 주입하고자 하는 인스턴스를 생성 및 반환하는 용도
* Container
    * provider를 바인딩하기 위한 용도
* Context
    * 의존성 resolve 하는 과정에서 provider가 전달 받는 context
* Request
    * context마다 request 인스턴스 하나씩 가지고 있음. resolved된 값을 저장하기 위한 용도