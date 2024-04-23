# JSX란 무엇인가?

1. 리액트의 등장과 함께 메타에서 선보인 구문이지만, 반드시 리액트에서 사용하리란 법은 없다.
2. xml과 유사한 내장형 구문이다.
3. ECMA 자바스크립트 표준은 아니다.
   1. 즉 V8이나 Deno와 같은 JS엔진에서 실행되거나 표현된다면 에러가 발생한다
4. 따라서 JSX는 반드시 트랜스파일러를 거처야 비로소 JS엔진이 이해할 수 있는 언어가 된다.

## 1️⃣ JSX의 정의

### JSXElement

1. **JSXOpeningElement** : `<Element>`
2. **JSXClosingElement** : `</Element>`
3. **JSXSelfClosingElement** : `<Element/>`
4. **JSXFragment** : `<></>`

→ 요소의 이름은 대문자로 시작해야한다. 이는 JSXElement의 표준은 아니다. html태그명과 구분하기 위한 React의 규칙이다.

### JSXElementName

1. **JSXIdentifier** : JSX내부에서 사용할 수 있는 식별자를 의미한다. 이는 자바스크립트 식별자 규칙과 동일하다. 숫자로 시작하거나 ‘$’와 ‘\_’이외의 특수문자는 사용 불가능 하다.

   `<$></$>` `<_></_>`

2. **JSXNamespaceName** : `JSXIdentifier:JSXIdentifier` 조합을 의미한다. 두 개 이상의 식별자는 취급하지 않는다.

   `<foo:bar></foo:bar>`

3. **JSXMemberExpression** : `JSXIdentifier.JSXIdentifier` 조합을 의미한다. 여러 개를 이어 사용하는 것이 가능하다. JSXNamespaceName과 이어서 사용은 불가능하다.

   `<foo.bar.baz></foo.bar.baz>`

### JSXAttribute

1. **JSXSpreadAttributes** : 자바스크립트의 전개 구문과 같은 역할을 한다.

   `{...AssignmentExpression}`

2. **JSXAttribute** : 속성을 나타내는 키와 값으로 나타낸다. JSXAttributeName과 JSXAttributeValue로 나타낸다.

   1. JSXAttributeName 은 ‘:’ 을 이용하여 키를 나타낼 수 있다.

      `<foo:bar foo:bar="baz"/>`

   2. JSXAttributeValue 값은 자바스크립트 문자열 혹은 `{AssignmentExpression}` 으로 나타낸다.
   3. JSXElement : 속성 값으로 다른 JSX요소가 들어갈 수 있다.
   4. JSXFragment : 별도 속성을 갖지 않는 형태의 JSX요소가 들어갈 수 있다. `<></>`

### JSXChildren

1. **JSXChild**

   JSXChildren을 이루는 기본 단위이다. JSXChildren는 JSXChild를 0개이상 가질 수 있다.

   JSXChild값으로 다음과 같은 요소가 올 수 있다.

   1. JSXText : 문자열
   2. JSXElement : 다른 요소값이 들어갈 수 있다.
   3. JSXFragment : 빈 JSX요소 값이 들어갈 수 있다.
   4. { JSXChildExpression (optional) } : 자바스크립트의 AssignmentExpression을 의미한다.

   ```tsx
   function App() {
     return <>{(() => "foo")()}</>;
   }
   ```

2. **JSXStrings**

   JSXAttributeValue와 JSXText는 HTML과 JSX사이에 복사와 붙여넣기를 쉽게 할 수 있도록 설계돼 있다.

   HTML에서 사용 가능한 문자열은 모두 JSXStrings에서도 가능하다. 이는 의도적으로 설계된 것이다.

   여기서 정의된 문자열은 **큰따옴표 혹은 작은따옴표로 구성된 문자열 혹은 JSXText**를 의미한다.

   **자바스크립트와 한 가지 중요한 차이점이 발생하는데, \로 시작하는 이스케이프 문자 형태소다.**

   ```tsx
   <button>/</button> // ok
   let escape1 = "\" // SyntaxError
   let escape2 = "\\" // ok
   ```

## 2️⃣ JSX는 어떻게 JS에서 변환될까?

@babel/plugin-transform-jsx 플러그인을 안다면 JSX가 변환되는 방식을 알 수 있다.

1. JSX 코드

   ```tsx
   const ComponentA = <A required={true}>Hello World</A>;
   const ComponentB = <>Hello World</>;
   const ComponentC = (
     <div>
       <span>hello world</span>
     </div>
   );
   ```

2. JSX 코드를 @babel/plugin-transform-jsx을 통해 변환한 결과

   ```tsx
   "use strict";

   var ComponentA = React.createElement(
     A,
     {
       required: true,
     },
     "Hello World"
   );
   var ComponentB = React.createElement(React.Fragment, null, "Hello World");
   var ComponentC = React.createElement("div", null, React.createElement("span", null, "hello world"));
   ```

JSX 반환값이 결국 `React.createElement`로 귀결된다는 사실을 파악한다면 쉽게 리팩터링할 수 있다.

```jsx
function TextOrHeading({ isHeading, children }) {
  return isHeading ? <h1 className="text">{children}</h1> : <span className="text">{children}</span>;
}
```

```jsx
function TextOrHeading({ isHeading, children }) {
	return React.createElement(
		isHeading ? "h1" : "span",
		{ className="text" },
		children,
	)
}
```
