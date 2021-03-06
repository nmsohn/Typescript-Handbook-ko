# JSX

## 목차 (Table of contents)

[소개 (Introduction)](#소개-Introduction)

[기본 사용법 (Basic usage)](#기본-사용법-Basic-usage)

[as 연산자 (The as operator)](#as-연산자-The-as-operator)

[타입 검사 (Type Checking)](#타입-검사-Type-Checking)

* [고유 요소 (Intrinsic elements)](#고유-요소-Intrinsic-elements)
* [값-기반 요소 (Value-based elements)](#값-기반-요소-Value-based-elements)
* [함수형 컴포넌트 (Function Component)](#함수형-컴포넌트-Function-Component)
* [클래스형 컴포넌트 (Class Component)](#클래스형-컴포넌트-Class-Component)
* [속성 타입 검사 (Attribute type checking)](#속성-타입-검사-Attribute-type-checking)
* [자식 타입 검사 (Children Type Checking)](#자식-타입-검사-Children-Type-Checking)

[JSX 결과 타입 (The JSX result type)](#JSX-결과-타입-The-JSX-result-type)

[표현식 포함하기 (Embedding Expressions)](#표현식-포함하기-Embedding-Expressions)

[리액트와 통합하기 (React integration)](#리액트와-통합하기-React-integration)

[팩토리 함수 (Factory Functions)](#팩토리-함수-Factory-Functions)

## 소개 (Introduction)

[↥ 위로](#JSX)

[JSX](https://facebook.github.io/jsx/)는 내장형 XML과 같은 문법입니다. 구현 방법에 따라 변환된 출력물의 의미가 다를 수 있지만, JSX는 유효한 JavaScript로 변환되도록 이루어져 있습니다. JSX는 [리액트](https://reactjs.org/)에 의해 큰 인기를 얻었습니다만, 이후엔 다른 구현도 등장했습니다. TypeScript는 JavaScript로의 컴파일, 타입 검사, 임베딩을 지원합니다.

## 기본 사용법 (Basic usage)

[↥ 위로](#JSX)

JSX를 사용하려면 다음의 두 작업을 해야 합니다.

1. 파일 이름을 `.tsx` 확장자로 지정합니다.
2. `jsx` 옵션을 활성화 합니다.

TypeScript엔 `preserve`, `react`, `react-native`라는 세 가지의 JSX 모드가 탑재되어 있습니다. 이러한 모드는 생성(emit) 단계에서만 영향을 미치며, 타입 검사에는 영향을 주지 않습니다. `preserve` 모드는 추후에 다른 변환 단계(예: [바벨](https://babeljs.io/))에 사용할 결과물을 위해 JSX 부분을 보존해 둡니다. 이러한 결과물은 `.jsx` 확장자를 갖게 됩니다. `react` 모드는 `React.createElement`를 생성하여 별도의 JSX 변환이 필요하지 않으며, 해당 결과물은 `.js` 확장자를 갖게 됩니다. `react-native` 모드는 JSX를 보존한다는 점에서 `preserve` 모드와 동일하지만, 결과물은 `.js` 확장자를 갖게 된다는 점에서 다릅니다.

|모드|입력|출력|출력파일 확장자|
|:---|:---|:---|:---|
|`preserve`|`<div />`|`<div />`|`.jsx`|
|`react`|`<div />`|`React.createElement("div")`|`.js`|
|`react-native`|`<div />`|`<div />`|`.js`|

이러한 모드는 `--jsx` 플래그 혹은 [tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)의 해당 옵션을 사용하여 지정할 수 있습니다.
> *참고: `--jsxFactory` 옵션으로 리액트의 JSX를 생성할 때 사용할 JSX 팩토리(JSX factory) 함수를 지정할 수 있습니다 (기본값은 `React.createElement`)

## `as` 연산자 (The `as` operator)

[↥ 위로](#JSX)

타입 단언(type assertion)을 어떻게 작성하는지 떠올려 볼까요:

```js
var foo = <foo>bar;
```

이 코드는 변수 `bar`가 `foo` 타입이라는 것을 단언합니다. TypeScript는 화살 괄호(angle bracket)를 사용해 타입을 단언하기 때문에, JSX의 구문과 함께 사용할 경우 특정 문법 해석에 문제가 될 수도 있습니다. 결과적으로 TypeScript는 `.tsx` 파일에서 화살 괄호를 통한 타입 단언을 허용하지 않습니다.

위의 구문은 `.tsx` 파일에서 사용할 수 없으므로, `as`라는 대체 연산자를 통해 타입 단언을 해야 합니다. 위의 예시는 `as` 연산자로 간단히 재작성할 수 있습니다.

```ts
var foo = bar as foo;
```

`as` 연산자는 `.ts`와 `.tsx` 파일 모두 사용할 수 있으며, 화살 괄호 형식의 단언과 동일하게 동작합니다.

## 타입 검사 (Type Checking)

[↥ 위로](#JSX)

JSX의 타입 검사를 이해하기 위해선, 먼저 고유 요소(intrinsic elements)와 값-기반 요소(value-based elements)의 다른 점에 대하여 알아야 합니다. `<expr />`라는 JSX 표현에서, `expr`은 특정 환경의 고유한 요소(예: DOM 환경의 `div` 또는 `span` 등) 혹은 여러분이 만든 커스텀 컴포넌트를 참조할 것입니다. 이는 다음과 같은 두 가지 이유로 중요합니다:

1. 리액트에서 고유 요소는 `React.createElement("div")`과 같은 문자열로 생성되는 반면, 여러분이 만든 컴포넌트는 `React.createElement("MyComponent")`로 취급되지 않습니다.
2. JSX 요소에 전달되는 속성(attribute)들의 타입은 다르게 조회되어야 합니다. 고유 요소의 속성은 *고유하게* 알려야 하는 반면, 컴포넌트는 각각 자신의 속성 집합을 지정하기를 원할 것입니다.

TypeScript는 이를 구분하기 위해 [리액트가 하는 것과 같은 규약](http://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components)을 사용합니다. 고유 요소는 항상 소문자로 시작하며, 값-기반 요소는 항상 대문자로 시작합니다.

### 고유 요소 (Intrinsic elements)

[↥ 위로](#JSX)

고유 요소는 `JSX.IntrinsicElements`라는 특별한 인터페이스에 의해 조회됩니다. 기본적으로 이 인터페이스가 지정되지 않은 경우, 모든 요소가 그대로 통과되어 고유 요소를 발견하지 않습니다. 그러나 이 인터페이스가 *있는* 경우, 고유 요소의 이름은 `JSX.IntrinsicElements` 인터페이스에 프로퍼티로 조회됩니다. 예를 들어:

```ts
declare namespace JSX {
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // 성공
<bar />; // 오류
```

위의 예에서, `<foo />`는 잘 동작하지만, `<bar />`는 `JSX.IntrinsicElements`에 지정되지 않았기 때문에 오류를 일으킵니다.

> 참고: 다음과 같이 `JSX.IntrinsicElements`에 캐치-올 문자열 인덱서를 지정할 수도 있습니다.

```ts
declare namespace JSX {
    interface IntrinsicElements {
        [elemName: string]: any;
    }
}
```

### 값-기반 요소 (Value-based elements)

[↥ 위로](#JSX)

값-기반 요소는 해당 스코프(scope)에 있는 식별자에 의해 간단히 조회됩니다.

```ts
import MyComponent from "./myComponent";

<MyComponent />; // 성공
<SomeOtherComponent />; // 오류
```

값-기반 요소를 정의하는데엔 다음의 두 가지 방법이 있습니다:

1. 함수형 컴포넌트(Function Component, FC)
2. 클래스형 컴포넌트

이 두 가지 유형의 값-기반 요소는 JSX 표현에서 구분할 수 없기 때문에, TypeScript는 이를 먼저 함수형 컴포넌트의 표현으로 해석합니다. 이 과정이 성공적으로 진행되면, TypeScript는 해당 정의에 대한 표현식 해석을 완료합니다. 함수형 컴포넌트로 해석하는 과정이 실패할 경우, TypeScript는 클래스형 컴포넌트로 해석을 시도합니다. 이 과정도 실패할 경우, TypeScript는 오류를 보고할 것입니다.

#### 함수형 컴포넌트 (Function Component)

[↥ 위로](#JSX)

이름에서 알 수 있듯, 이 컴포넌트는 `props`라는 객체를 첫 번째 인수로 받는 JavaScript의 함수로 정의됩니다. TypeScript는 이 컴포넌트가 `JSX.Element`에 할당할 수 있는 반환 타입을 사용하도록 규정합니다.

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

함수형 컴포넌트는 JavaScript 함수이므로, 함수 오버로드 또한 사용 가능합니다:

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

> 참고: 함수형 컴포넌트는 이전에 무상태 함수형 컴포넌트(Stateless Function Components, SFC)로 알려져 있었습니다. 하지만 최근 버전의 리액트에선 더 이상 함수형 컴포넌트를 무상태로 취급하지 않으며, `SFC` 타입과 그 별칭인 `StatelessComponent`은 더 이상 사용되지 않습니다.

#### 클래스형 컴포넌트 (Class Component)

[↥ 위로](#JSX)

클래스형 컴포넌트의 타입을 정의하는 것은 가능합니다만, 이를 위해선 *요소 클래스 타입(element class type)* 과 *요소 인스턴스 타입(element instance type)* 이라는 새로운 용어를 이해해야 합니다.

`<Expr />`이라는 표현에서, *요소 클래스 타입* 은 `Expr`의 타입입니다. 따라서, 위의 예에서 `MyComponent`가 ES6 클래스라면 이 클래스 타입은 클래스 생성자이고 전역 변수일 것입니다. `MyComponent`가 팩토리 함수라면, 클래스 타입은 함수일 것입니다.

클래스 타입이 결정되면, 인스턴스 타입은 클래스의 생성자 혹은 호출(있는 것 중 어느 쪽으로든)에 의한 반환 타입의 조합에 의해 결정됩니다. 따라서 ES6 클래스의 경우에 인스턴스 타입은 해당 클래스의 인스턴스의 타입이 될 것이고, 팩토리 함수의 경우엔 해당 함수로부터 반환된 값의 타입이 될 것입니다.

```ts
class MyComponent {
  render() {}
}

// 생성자 사용
var myComponent = new MyComponent();

// 요소 클래스 타입 => MyComponent
// 요소 인스턴스 타입 => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {
    }
  }
}

// 호출 사용
var myComponent = MyFactoryFunction();

// 요소 클래스 타입 => MyFactoryFunction
// 요소 인스턴스 타입 => { render: () => void }
```

흥미롭게도 요소 인스턴스 타입은 `JSX.ElementClass`에 할당 가능한 것이어야 하며, 그렇지 않을 경우 오류를 일으킵니다. 기본적으로 `JSX.ElementClass`는 `{}`이지만, 적절한 인터페이스에 적합한 유형으로만 JSX를 사용하도록 제한할 수 있습니다.

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

### 속성 타입 검사 (Attribute type checking)

[↥ 위로](#JSX)

속성 타입 검사를 위해선 첫 번째로 *요소 속성 타입(element attributes type)* 을 결정해야 합니다. 이는 고유 요소와 값-기반 요소 간에 약간 다른 점이 있습니다.

고유 요소의 경우, 요소 속성 타입은 `JSX.IntrinsicElements`의 프로퍼티 타입과 동일합니다.

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// 'foo'의 요소 속성 타입은 '{bar?: boolean}'
<foo bar />;
```

값-기반 요소의 경우엔 약간 더 복잡합니다. 이전에 `JSX.ElementAttributesProperty`를 결정하는데 사용된 *요소 인스턴스 타입* 의 프로퍼티 타입에 따라 결정됩니다. 이는 단일 프로퍼티로 선언되어야 하며, 이후 해당 이름을 사용합니다. TypeScript 2.8 기준으로 `JSX.ElementAttributesProperty`가 제공되지 않는 경우, 클래스형 컴포넌트의 생성자 또는 함수형 컴포넌트의 첫 번째 매개변수 타입을 대신 사용할 수 있습니다.

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

요소 속성 타입은 JSX에서 속성 타입을 확인하는데 사용됩니다. 선택적 혹은 필수적인 프로퍼티들이 지원됩니다.

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number }
  }
}

<foo requiredProp="bar" />; // 성공
<foo requiredProp="bar" optionalProp={0} />; // 성공
<foo />; // 오류, requiredProp이 누락됨
<foo requiredProp={0} />; // 오류, requiredProp은 문자열이어야 함
<foo requiredProp="bar" unknownProp />; // 오류, unknownProp은 존재하지 않음
<foo requiredProp="bar" some-unknown-prop />; // 성공, because 'some-unknown-prop'은 유효한 식별자가 아님
```

> 참고: 만약 속성 이름이 유효한 JavaScript 식별자(`data-*` 속성 등)가 아닌 경우, 해당 이름을 요소 속성 타입에서 찾을 수 없더라도 오류로 간주하지 않습니다.

추가적으로, `JSX.IntrinsicAttributes` 인터페이스는 일반적으로 컴포넌트의 프로퍼티나 인수로 사용되지 않는 JSX 프레임워크를 위한 추가적인 프로퍼티(리액트의 `key`와 같은)를 지정할 수 있습니다. 좀 더 전문적으로 말하자면, `JSX.IntrinsicClassAttributes<T>` 제네릭(generic) 타입을 사용하여 클래스형 컴포넌트에 대해 동일한 종류의 추가 속성을 지정할 수 있습니다(함수형 컴포넌트 제외). 이 유형에서, 제네릭의 매개변수는 클래스 인스턴스 타입에 해당합니다. 리액트의 경우, 이는 `Ref<T>` 타입의 `ref` 속성을 허용하는 데에 쓰입니다. 일반적으로는, JSX 프레임워크의 사용자가 모든 태그에 특정 속성을 제공할 필요가 없다면, 이러한 인터페이스의 모든 프로퍼티는 선택적이어야 합니다.

스프레드(spread) 연산자 또한 동작합니다:

```JSX
var props = { requiredProp: "bar" };
<foo {...props} />; // 성공

var badProps = {};
<foo {...badProps} />; // 오류
```

### 자식 타입 검사 (Children Type Checking)

[↥ 위로](#JSX)

TypeScript 2.3부터, TypeScript는 *자식(children)* 의 타입 검사를 도입했습니다. *자식* 은 요소 속성 타입에 하위 JSX 표현을 삽입할 수 있는 특수한 프로퍼티입니다. TypeScript가 `JSX.ElementAttributesProperty`를 사용해 *프롭(props)* 을 결정하는 것과 유사하게, `JSX.ElementChildrenAttribute`를 사용해 해당 프롭 내의 *자식* 의 이름을 결정합니다. `JSX.ElementChildrenAttribute`는 단일 프로퍼티로 선언되어야 합니다.

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

다른 속성처럼 *자식* 의 타입도 지정할 수 있습니다. 이는 기본 타입(예: [리액트 타이핑](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react) 사용 시)을 오버라이드 할 것입니다.

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

// 오류 : 자식이 JSX.Element의 배열이 아닌 JSX.Element 타입입니다.
<Component name="bar">
  <h1>Hello World</h1>
  <h2>Hello World</h2>
</Component>

// 오류 : 자식이 JSX.Element의 배열이나 문자열이 아닌 JSX.Element 타입입니다.
<Component name="baz">
  <h1>Hello</h1>
  World
</Component>
```

## JSX 결과 타입 (The JSX result type)

[↥ 위로](#JSX)

기본적으로 JSX 표현식의 결과물은 `any` 타입입니다. `JSX.Element` 인터페이스를 수정하여 특정한 타입을 지정할 수 있습니다. 그러나, 이 인터페이스에서는 JSX의 요소, 속성, 자식에 대한 정보를 검색할 수 없습니다. 이 인터페이스는 블랙박스(black box)입니다.

## 표현식 포함하기 (Embedding Expressions)

[↥ 위로](#JSX)

JSX는 중괄호(`{ }`)로 표현식을 감싸 태그 사이에 표현식을 사용하는 것을 허용합니다.

```JSX
var a = <div>
  {["foo", "bar"].map(i => <span>{i / 2}</span>)}
</div>
```

위의 코드는 문자열을 숫자로 나눌 수 없기 때문에 오류를 일으킵니다. `preserve` 옵션을 사용할 때, 결과는 다음과 같습니다:

```JSX
var a = <div>
  {["foo", "bar"].map(function (i) { return <span>{i / 2}</span>; })}
</div>
```

## 리액트와 통합하기 (React integration)

[↥ 위로](#JSX)

리액트에서 JSX를 사용하기 위해선 [리액트 타이핑](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)을 사용해야 합니다. 이는 리액트를 사용할 수 있도록 `JSX` 네임스페이스(namespace)를 적절하게 정의합니다.

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

[↥ 위로](#JSX)

`jsx: react` 컴파일러 옵션에서 사용하는 팩토리 함수는 설정이 가능합니다. 이는 `jsxFactory` 명령 줄 옵션을 사용하거나 인라인 `@jsx` 주석을 사용하여 파일별로 설정할 수 있습니다. 예를 들어 `jsxFactory`에 `createElement`를 설정했다면, `<div />`는 `React.createElement("div")` 대신 `createElement("div")`으로 생성될 것입니다.

주석을 이용한 버전은 다음과 같이 사용할 수 있습니다(TypeScript 2.8 기준):

```ts
import preact = require("preact");
/* @jsx preact.h */
const x = <div />;
```

이는 다음처럼 생성됩니다:

```ts
const preact = require("preact");
const x = preact.h("div", null);
```

선택된 팩토리는 전역 네임스페이스로 돌아가기 전에 `JSX` 네임스페이스(타입 검사를 위한 정보)에도 영향을 미칩니다. 팩토리가 `React.createElement`(기본값)로 정의되어 있다면, 컴파일러는 전역 `JSX`를 검사하기 전에 `React.JSX`를 먼저 검사할 것입니다. 팩토리가 `h`로 정의되어 있다면, 컴파일러는 전역 `JSX`를 검사하기 전에 `h.JSX`를 검사할 것입니다.
