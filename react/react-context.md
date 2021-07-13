# React Context

* Context로 부모-자식이 아닌 컴포넌트 간 통신을 구현할 수 있다

## When to use Context

* 컨텍스트는 전역 데이터를 공유해야 할 때 사용한다
    (ex. 인증된 사용자, 테마 등등)

* 아래 예시에선 ThemedButton에 theme을 전달하기 위해 App -> Toolbar -> ThemeButton으로 전달하고 있다 
```
class App extends React.Component {
    render() {
        return <Toolbar theme="dark"/>;
    }
}

function Toolbar(props) {
    return (
        <div>
            <ThemedButton theme={props.theme} /> 
        </div>
    )
}

class ThemedButton extends React.Component {
    render() {
        return <Button theme={this.props.theme} />
    }
}
```

* 컨텍스트를 사용하면 위와 같이 번거로운 작업을 피할 수 있다
```
const themeContext = React.createContext('light');

class App extends React.Component {
    render() {
        return (
            <ThemeContext.Provider value="dark">
                <Toolbar />
            </ThemeContext.Provider>
        );
    }
}

function Toolbar() {
    return (
        <div>
            <ThemedButton />
        </div>
    )
}

class ThemedButton extends React.Component {
    static contextType = ThemeContext;
    render() {
        return <Button theme={this.context} />
    }
}
```

## Before you use Context

* Context is primarily used when some data needs to be accessible by many components at different nesting levels.
* Apply it sparingly because it makes components reuse more difficult

**If you only want to avoid passing some props through any levels, component composition is often a simpler solution than context**

* 예를 들어 user, avatarSize prop을 Link, Avatar 컴포넌트가 읽을 수 있게 자식 컴포넌트들에게 전달한다고 해보자
```
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout user={user} avatarSize={avatarSize} />
// ... which renders ...
<NavigationBar user={user} avatarSize={avatarSize} />
// ... which renders ...
<Link href={user.permalink}>
  <Avatar user={user} size={avatarSize} />
</Link>
```

* user와 avatarSize를 여러 레벨의 컴포넌트에 전달하는 것이 불현할 수 있다
    * Avatar의 상위 레벨 컴포넌트들이 모두 user, avatarSize prop을 받아야 하기 때문이다

* context 없이 이를 해결하는 방법은 Avatar 컴포넌트에게 직접 user, avatarSize를 전달하는 것이다
```
function Page(props) {
    const user = props.user;
    const userLink = (
        <Link href={user.permalink}>
            <Avatar user={user} size={props.avatarSize} />
        </Link>
    )
    return <PageLayout userLink={userLink} />
}
```

* 위와 같이 바꾸면 오직 최상위 컴포넌트만 Link와 Avatar가 user, avatarSize를 사용한다는 사실을 안다
* The inversion of control can make your code cleaner in many cases by reducing the amount of props you need to pass through your application and giving more control to the root compoennets
* Such inversion, however, isn't the right choice in every case
* moving more complexity hight in thre tree make those higher-level components more complicated and forces the lower-level components to be more flexible than you may want

* 아래와 같이 여러 컴포넌트드를 전달할 수 있으며, 여러 slot들로 분리하는 것도 가능하다
```
function Page(props) {
    const user = props.user;
    const content = <Feed user={user} />;
    const topBar = (
        <NavigationBar>
            <Link href={user.permaLink}>
                <Avatar user={user} size={props.avatarSize} />
            </Link>
        </NavigationBar>
    );

    return <PageLayout
        topBar={topBar}
        content={content}
    />
}
```

* intermideate parent를 없애고 싶은거라면, 대부분의 경우 위처럼 래픽토링하는 것만으로도 충분하
* 하지만 같은 데이터가 여러 컴포넌트들에 공유돼야 하는상황이 있을 수 있다
* Context lets you braodcast such data, and changes to it, to all components below
* Common examples where using context might be simpler than the alternatives include managing the current locale, theme, data cache

## API

### React.createContext

```
const MyContet = React.createContext(defaultValue);
```

* Context 오브젝트 생성
* 이 오브젝트를 구독하는 컴포넌트를 렌더링할 때 상위 계층에서 가장 가까운 Provider에서 값을 가져온다

* defaultValue는 컴포넌트가 Provider를 찾지 못했을 때만 사용된다

### Context.Provider

```
<MyContext.Provider value={...}>
```

* 모든 Context 오브젝트는 Provider 컴포넌트를 가지고 있다
* Provider React Component allows consuming components to subscribe to context changes

* Provider의 value prop은 Context 오브젝트를 구독하는 자식 컴포넌트들에게 전달된다
* 한 Provider는 여러 Consumer와 연결될 ㅜㅅ 이다
* Provider를 중첩시켜 value를 오버라이딩 하는 것도 가능하다

### Claass.contextType

```
class MyClass extends React.Component {
    componentDidMount() {
        let value = this.context;
    }

    componentDidUpdate() {
        let value = this.context;
    }

    componentWillUnmount() {
        let value = this.context;
    }

    render() {
        let value = this.context;
    }
}

MyClass.contextType = MyContext;
```

* 클래스가 Context 오브젝트에 구독할 수 있게한다
* this.context로 Context 오브젝트의 값을 가져올 수 있다
* render 함수를 포함해 모든 라이프사이클 단계에서 사용할 수 있다

### Context.Consumer
```
<MyContext.Consumer>
    {value => /* render */}
</MyContext.Consumer>
```

* 함수형 컴포넌트가 Context 오브젝트를 구독할 수 있게한다
* Requires a function as a child
    * The function receives the current context value and returns a React node

### Context.displayName

* React DevTools에서 표시되는 겂을 설정할 수 있다

```
const myContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" in DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" in DevTools
```