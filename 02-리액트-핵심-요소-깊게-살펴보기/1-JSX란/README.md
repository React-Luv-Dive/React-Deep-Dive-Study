## 2.1 JSX란?

- JSX는 리액트가 등장하면서 페이스북(현 메타)에서 소개한 새로운 구문이다. 하지만, 반드시 리액트에서만 사용하라는 법은 아니다.
- JSX는 XML과 유사한 내장형 구문이며, 리액트에 종속적이지 않은 독자적인 문법이다.
- 반드시 트랜스파일러를 거쳐야 자바스크립트 런타임이 이해할 수 있는 의미 있는 자바스크립트 코드로 변환된다.
- JSX의 설계 목적은 JSX 내부에 트리 구조로 표현하고 싶은 다양한 것들을 작성해두고, 이 JSX를 트랜스파일이라는 과정을 거쳐 자바스크립트가 이해할 수 있는 코드로 변경하는 것이 목표이다.
- 요약, JSX는 자바스크립트 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는 데 많은 도움을 주는 새로운 문법이라 볼 수 있다.

### 2.1.1 JSX의 정의

JSX는 기본적으로 JSXElement, JSXAttributes, JSXChildren, JSXStrings 4가지 컴포넌트를 기반으로 구성돼 있다.

- **JSXElement**

  JSX를 구성하는 가장 기본 요소이다.  
  HTML의 요소와 비슷한 역할을 한다.

> **JSXFragment**: 아무런 요소가 없는 형태. `<></>`

> **요소명은 대문자로 시작**  
> 컴포넌트를 만들어 사용할 때에는 반드시 대문자로 시작하는 컴포넌트.  
> HTML 태그명과 사용자가 만든 컴포넌트 태그명을 구분짓기 위해서.

- **JSXAttributes**

  JSXElement에 부여할 수 있는 속성을 의미한다.  
  단순한 속성을 의미하기 때문에 모든 경우에서 필수값이 아니고, 존재하지 않아도 에러가 발생하지 않는다.

- **JSXChildren**

  JSXElement의 자식 값을 나타낸다.  
  JSX로 부모와 자식 관계를 나타낼 수 있으며, 그 자식을 JSXChildren이라고 한다.

- **JSXStrings**

  HTML에서 사용 가능한 문자열은 모두 가능하다.  
  _문자열: 큰따옴표, 작은 따옴표로 구성된 문자열 혹은 JSXText를 의미한다._

  > **JSXText**: {,<,>,}를 제외한 문자열

### 2.1.3 JSX는 어떻게 자바스크립트로 변환될까?

경우에 따라 다른 JSXElements를 렌더링 할 때, 요소 전체를 감싸지 않더라도 처리할 수 있다.  
이는 JSXElement만 다르고, JSXAttributes, JSXChildren이 완전히 동일한 상황에서 중복 코드를 최소화할 수 있어 유용하다.

- props 여부에 따라 children 요소만 달라지는 경우

  번거롭게 전체 내용을 삼항 연산자로 처리할 필요 없다.  
  불필요한 코드 중복이 발생한다.

  ```jsx
  function TextOrHeading({ isHeading, children }: PropsWithChildren<{ isHeading: boolean }>) {
  	return isHeading ? <h1 calssName="text">{children}</h1> : <span className="text">{children}</span>;
  }
  ```

- 간결하게 처리

  JSX가 변환되는 특성을 활용하여, 다음과 같이 간결하게 처리할 수 있다.  
  JSX 반환값이 React.createElement로 귀결되기에 이런 방식으로 쉽게 리팩토링 할 수 있다.

  ```jsx
  function TextOrHeading({
  	isHeading,
  	children,
  }:PropsWithChildren<{isHeading: boolean}>){
  	return createElement(
  		isHeading ? 'h1' : 'span',
  		{className: 'text},
  		children
  	)
  }
  ```
