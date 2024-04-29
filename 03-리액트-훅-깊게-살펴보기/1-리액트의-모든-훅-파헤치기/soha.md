## 3.1 리액트의 모든 훅 파헤치기

### 3.1.1 useState

함수 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 해주는 훅이다.

```jsx
import { useState } from 'react';

const [state, setState] = useState(initialState);
```

useState의 인수로는 사용할 state의 초기값을 넘겨준다. 아무런 값을 넘겨주지 않으면 초기값은 undefined이다.  
useState의 혹의 반환 값은 배열이며, 배열의 첫번째 원소로 state 값 자체를 사용할 수 있다.  
두번째 원소인 setState 함수를 사용해 해당 state의 값을 변경할 수 있다.

#### 게이른 초기화

useState의 인수로 원시값이 아닌 특정 값을 넘기는 함수를 인수로 넣어줄 수 있다.
useState에 변수 대신 함수를 넘기는 것을 게으른 초기화(lazy initialization)라고 한다.

게으른 초기화 함수는 오로지 state가 처음 만들어질 때만 사용된다.  
게으른 초기화는 useState의 초기값이 복잡하거나 무거운 연산을 포함하고 있을 때 사용하는 것이 좋다.  
localStorage나 sessionStorage에 대한 접근, map, filter, find 같은 배열에 대한 접근, 혹은 초기값 계산을 위해 함수 호출이 필요할 때 등

- 일반적인 useState 사용

  바로 값을 집어 넣는다.

  ```jsx
  const [count, setCount] = useState(Number.parseInt(window.localStorage.getItem(cacheKey)));
  ```

- 게으른 초기화

  함수를 실행해 값을 반환해서 넣는다.

  ```jsx
  const [count, setCount] = useState(() => Number.parseInt(window.localStorage.getItem(cacheKey)));
  ```

### 3.1.2 useEffect

애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 매커니즘이다.  
렌더링할 때마다 의존성에 있는 값을 보면서 의존성 값이 이전과 다른 게 하나라도 있으면 부수 효과를 실행한다. 즉, state와 props의 변화 속에서 일어나는 렌더링 과정에서 실행되는 부수 효과 함수이다.

이 부수 효과가 언제 일어나는지 보다는 **어떤 상태값과 함께 실행되는지** 살펴보는 것이 중요하다.

```jsx
function Component() {
	// ...
	useEffect(() => {
		// do something
	}, [props, state]);
	// ...
}
```

useEffect의 첫번째 인수로는 실행할 부수 효과가 포함된 함수를, 두번째 인수로는 의존성 배열을 전달한다.  
의존성 배열은 어느 정도 길이를 가진 배열일 수도, 아무런 값이 없는 빈 배열일 수도, 배열 자체를 넣지 않고 생략할 수도 있다.

#### 클린업 함수의 목적

**클린업 함수**  
useEffect 내에서 반환되는 함수이며, 이벤트를 등록하고 지울 때 사용해야 한다.  
새로운 값을 기반으로 렌더링 뒤에 실행되지만 변경된 값을 읽는 것이 아니라 함수가 정의됐을 당시에 선언됐던 이전 값을 보고 실행된다. 말 그대로 이전 상태를 청소해주는 것이다.

```jsx
// 최초 실행
useEffect(() => {
	function addMouseEvent() {
		console.log(1);
	}

	window.addEventListener('click', addMouseEvent);

	// 클린업 함수
	// 해당 클린업 함수는 다음 렌더링이 끝난 뒤에 실행된다.

	return () => {
		console.log('클린업 함수 실행', 1);
		window.removeEventListener('click', addMouseEvent);
	};
}, [counter]);

// ...

// 이후 실행
useEffect(() => {
	function addMouseEvent() {
		console.log(2);
	}

	window.addEventListener('click', addMouseEvent);

	// 클린업 함수

	return () => {
		console.log('클린업 함수 실행', 2);
		window.removeEventListener('click', addMouseEvent);
	};
}, [counter]);
```

useEffect에 이벤트를 추가했을 때 클린업 함수에서 해당 이벤트를 지워야 한다.  
useEffect는 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤에 콜백을 실행한다.  
따라서, 이벤트를 추가하기 전에 이전에 등록했던 이벤트 핸들러를 삭제하는 코드를 클린업 함수에 추가하는 것이다. 이렇게 함으로 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지할 수 있다.

#### 의존성 배열

의존성 배열은 보통 빈 배열을 두거나, 아무런 값도 넘기지 않거나, 혹은 사용자가 직접 원하는 값을 넣어 줄 수 있다.

- 빈 배열을 둔다면, 리액트가 이 useEffect는 비교할 의존성이 없다고 판단해 최초 렌더링 직후에 실행된 다음부터는 더 이상 실행되지 않는다.
- 아무런 값도 넘겨주지 않는다면, 의존성을 비교할 필요 없이 렌더링할 떄마다 실행이 필요하다고 판단해 렌더링이 발생할 때마다 실행된다.
  보통 컴포넌트가 렌더링됐는지 확인하기 위한 방법으로 사용된다.

  서버 사이드 렌더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해준다.

  ```jsx
  useEffect(() => {
  	console.log('컴포넌트 렌더링됨');
  });
  ```

#### useEffect를 사용할 때 주의할 점

- **`eslint-disable-line react-hooks/exhaustive-deps` 주석은 최대한 자제하라**

  해당 주석을 사용해 ESLint의 `react-hooks/exhaustive-deps` 룰에서 발생하는 경고를 무시하게 된다. 이 룰은 useEffect 인수 내부에서 사용하는 값 중 의존성 배열에 포함돼 있지 않은 값이 있을 때 경고를 발생시킨다.

  ```jsx
  useEffect(() => {
  	console.log(props);
  }, []); // eslint-disable-line react-hooks/exhaustive-deps
  ```

  대부분의 경우에 의도치 못한 버그를 만들 가능성이 크기 때문에 가급적 사용해서는 안된다.

  > useEffect에 빈 배열을 넘기기 전에는 정말로 useEffect의 부수 효과가 컴포넌트의 상태와 별개로 작동해야만 하는지, 혹은 여기서 호출하는게 최선인지 한 번 더 검토해 봐야 한다.
  >
  > 또한, 만약 특정 값을 사용하지만 해당 값의 변경 시점을 피할 목적이라면 메모이제이션을 적절히 활용해 해당 값의 변화를 막거나 적당한 실행 위치를 다시 한번 고민하는 것이 좋다.

- **useEffect의 첫번째 인수에 함수명을 부여하라**

  useEffect를 사용하는 많은 코드에서 첫번째 인수로 익명 함수를 넘겨준다.

  ```jsx
  useEffect(() => {
  	logging(user.id);
  }, [user.id]);
  ```

  useEffect의 수가 적거나 복잡성이 낮다면 이런 익명 함수를 사용해도 큰 문제는 없다.

  그러나 useEffect의 코드가 복잡하고 많아질수록 무슨 일을 하는 useEffect 코드인지 파악하기 어려워 진다.  
  이때, useEffect의 인수를 적절한 이름을 사용한 기명함수로 사용하는 것이 좋다.  
  useEffect의 목적을 파악하기 쉬워지기 때문이다. 또한 책임을 최소한으로 좁힌다는 점에서 굉장히 유용하다.

  ```jsx
  function logActiveUser() {
  	useEffect(() => {
  		logging(user.id);
  	}, [user.id]);
  }
  ```

- **거대한 useEffect를 만들지 마라**

  부수 효과의 크기가 커질수록 애플리케이션 성능에 악영향을 미치기 때문에 useEffect는 간결하고 가볍게 유지하는 것이 좋다.  
  부득이하게 큰 useEffect를 만들어야 한다면 적은 의존성 배열을 사용하는 여러 개의 useEffect로 분리하는 것이 좋다.

  의존성 배열에 여러 변수가 들어가야 하는 상황이라면 최대한 useCallback과 useMemo 등으로 사전에 정제한 내용들만 useEffect에 담아두는 것이 좋다. 이렇게 하면 언제 useEffect가 실행되는지 좀 더 명확하게 알 수 있다.

- **불필요한 외부 함수를 만들지 마라**

  useEffect가 실행하는 콜백 또한 불필요하게 존재해서는 안된다.  
  useEffect 내에서 사용할 부수 효과라면 내부에서 만들어서 정의해서 사용하는 편이 불필요한 의존성 배열도 줄일 수 있는 등 훨씬 도움이 된다.

  > **왜 useEffet의 콜백 인수로 비동기 함수를 바로 넣을 수 없을까?**  
  > 기술적인 문제가 아닌 useEffect에서 비동기로 함수를 호출할 경우 경쟁 상태가 발생할 수 있기 때문이다.
  >
  > **비동기 함수를 어떻게 실행할 수 있을까?**  
  > 비동기 useEffect는 state의 경쟁 상태를 야기할 수 있고 cleanup 함수의 실행 순서도 보장할 수 없기 때문에 개발자의 편의를 위해 useEffect에서 비동기 함수를 인수로 받지 않는다고 볼 수 있다.
  >
  > useEffect의 인수로 비동기 함수를 지정할 수 없는 것이지, useEffect 내부에서 비동기 함수를 선언해 실행하거나 즉시 실행 비동기 함수를 만들어서 사용하는 것은 가능하다.  
  > 다만, 비동기 함수가 내부에 존재하면 useEffect 내부에서 비동기 함수가 생성되고 실행되는 것을 반복하므로 클린업 함수에서 이전 비동기 함수에 대한 처리를 추가하는 것이 좋다.  
  > fetch의 경우에는 abortController 등으로 이전 요청을 취소하는 것이 좋다.
  >
  > **abortController**: 하나 이상의 웹 요청을 취소할 수 있게 해준다. DOM 요청이 완료되기 전에 취소한다. 이를 통해 fetch 요청, 모든 응답 `Body` 소비, 스트림을 취소할 수 있다.
  >
  > **경쟁상태(race condition)**:  실행/출력 결과가 일정하지 않고, 입력의 타이밍, 실행되는 순서나 시간 등에 영향을 받게 되는 상황. 이전 state 기반으로 결과가 나오게 된다.

### 3.1.3 useMemo

useMemo는 비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅이다.  
리액트에서 최적화를 떠올릴 때 가장 먼저 업급된다.

```jsx
import { useMemo } from 'react';

const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b]);
```

첫번째 인수로는 어떠한 값을 반환하는 생성 함수를, 두번째 인수로는 해당 함수가 의존하는 값의 배열을 전달한다.

렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 함수를 재실행하지 않고 이전에 기억해 둔 해당 값을 반환한다.  
의존성 배열의 값이 변경됐다면 첫번째 인수의 함수를 실행한 후에 그 값을 반환하고 그 값을 다시 기억해 둔다.

이런 메모이제이션은 값뿐만 아니라 컴포넌트도 가능하다.

```jsx
function ExpensiveComponent({value}){
	useEffect(()=>{
	console.log('rendering')
	})
	return <span>{value + 1000}</span>
}

function App() {
	const [value,setValue] = useState(10)
	const [, triggerRendering] = useState(false)

	// 컴포넌트의 props를 기준으로 컴포넌트 자체를 메모이제이션했다.
	const MemoizedComponent = useMemo(
		() => <ExpensiveComponent value={value}>,
		[value]
	)

	function handleClick() {
		setValue(Number(e.target.value))
	}

	function handleClick() {
		triggerRendering((prev)=> !prev)
	}

	return(
		<>
			<input value={value} onChange={handleChange}>
			<button onClick={handleCLick}>렌더링 발생!</button>
			{MemoizedComponent}
		</>
	)
}
```

useMemo로 컴포넌트를 감싸는 것 보다는 React.memo를 쓰는 것이 더 현명하긴 하다.

triggerRendering으로 컴포넌트 렌더링을 강제로 발생시켰지만 MemoizedComponent는 리렌더링되지 않는다. MemoizedComponent는 의존성으로 선언된 value가 변경되지 않는 한 다시 계산되는 일이 없다.  
useMemo 등 메모이제이션을 활용하면 무거운 연산을 다시 수행하는 것을 막을 수 있다는 장점이 있다.

### 3.1.4 useCallback

useCallback은 인수로 넘겨받은 콜백 자체를 기억한다. 즉, 특정 함수를 새로 만들지 않고 다시 재사용한다는 의미이다.  
값의 메모이제이션을 위해 useMemo를 사용했다면, 함수의 메모이제이션을 위해 사용하는 것이 useCallback이다.

useCallback의 첫번째 인수로 함수를, 두번째 인수로 의존성 배열을 넣으면 의존성 배열이 변경되지 않는 한 함수를 재생성하지 않는다.

```jsx
const ChildComponent = memo(({name, value, onChange})=>{
	useEffect(()=>{
		console.log('rendering',name)
	})

	return (
		// ...
	)
})

function App() {
	const [status1,setStatus1] = useState(false)
	const [status2,setStatus2] = useState(false)

	const toggle1 = useCallback(
		function toggel1(){
			setStatus1(!status1)
		}, [status1]
	)

	const toggle2 = useCallback(
			function toggel2(){
				setStatus2(!status2)
			}, [status2]
		)

	return (
		<ChildComponent name="1" value={status1} onChange={toggle1} />
		<ChildComponent name="2" value={status2} onChange={toggle2} />
	)
}
```

ChildComponent에 memo를 사용해 name, value, onChange의 값을 모두 기억하고, 값이 변경되지 않았을 때는 렌더링되지 않는 것을 목적으로 작성한 코드이다.  
useCallback을 추가하기 전에는 어느 한 버튼을 클릭하면 클릭하지 않은 컴포넌트도 렌더링되었다.  
state 값이 바뀌면서 App 컴포넌트가 리렌더링되고, 매번 onChange로 넘기는 함수가 재생성되고 있기 때문이다.  
useCallback을 추가하면서, 해당 의존성이 변경됐을 때만 함수가 재생성되도록 즉, 클릭한 컴포넌트만 리렌더링 되도록 수정되었다.

**함수의 재생성을 막아 불필요한 리소스 또는 리렌더링을 방지하고 싶을 때** useCallback을 사용하면 된다.

> **왜 useCallback에 기명 함수를 넘겨주었나?**  
> 일반적으로는 useCallback이나 useMemo, useEffect를 익명 함수로 첫번째 인수를 넘겨준다.  
> 그러나 위 예시처럼 기명함수를 넘겨줄 수 있다. 익명함수는 이름이 없기에 해당 함수를 추적하기가 어렵다. 그러나 기명함수로 넘겨주게 되면 크롬 메모리 탭에서 디버깅을 용이하다.

useCallback과 useMemo의 유일한 차이는 메모이제이션을 하는 대상이 변수인지 함수인지 이다.  
그렇기 때문에 useMemo는 반환문으로 함수 선언문을 반환해야 한다.  
다만, useMemo로 useCallback을 구현할 수 있는데 이렇게 하게 되면 불필요한 코드가 길어지고 혼동을 야기할 수 있다.

```jsx
const handleClickCallback = useCallback(() => {
	setCounter((prev) => prev + 1);
}, []);

const handleClickMemo = useMemo(() => {
	return () => setCounter((prev) => prev + 1);
}, []);
```

### 3.1.5 useRef

useRef는 컴포넌트가 렌더링될 때만 생성되며, 컴포넌트 인스턴스가 여러개라도 각각 별개의 값을 바라본다.  
useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다.  
**그러나, useState와 구별되는 큰 차이점 2가지**

- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
- useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.
  즉, useRef는 렌더링을 발생시키지 않고 원하는 상태값을 저장할 수 있다.

useRef의 가장 일반적인 사용 예는 DOM에 접근하고 싶을 때이다.

```jsx
function RefComponent() {
	const inputRef = useRef();

	// 이때는 미처 렌더링이 실행되기 전(반환되기 전)이므로 undefined를 반환한다.
	console.log(inputRef.current); // undefined

	useEffect(() => {
		console.log(inputRef.current); // <input type="text"></input>
	}, [inputRef]);

	return <input ref={inputRef} type="text" />;
}
```

useRef는 최초에 넘겨받은 기본값을 가지고 있다.  
최초 기본값은 return 문에 정의해 둔 DOM이 아니라 useRef()로 넘겨받은 인수다.  
useRef가 선언된 당시에는 컴포넌트가 렌더링되기 전이라 return으로 컴포넌트의 DOM이 반환되기 전이므로 undefined이다.

### 3.1.6 useContext

#### Context란?

A 컴포넌트에서 제공하는 데이터를 D 컴포넌트에서 사용하려면 props를 하위 컴포넌트로 필요한 위치까지 계속해서 넘겨야 한다. 이러한 기법을 **prop 내려주기(props drilling)** 라고 한다.

```jsx
<A props={something}>
	<B props={something}>
		<C props={something}>
			<D props={something} />
		</C>
	</B>
</A>
```

prop 내려주기는 해당 데이터를 제공하고 사용하는 쪽 모두 불편하다.  
해당 값을 사용하지 않는 컴포넌트에서도 단순히 값을 전달하기 위해 props가 열려 있어야 하며, 사용하는 쪽도 prop 내려주기가 적용돼 있는지 확인해야 하는 등 번거롭기 때문이다.

이러한 prop 내려주기를 극복하기 위해서 등장한 개념이 바로 **콘텍스트(Context)** 이다.  
콘텍스트를 사용하면 명시적 props 전달 없이도 선언된 하위 컴포넌트 모두에서 자유롭게 원하는 값을 사용할 수 있다.

#### Context를 함수 컴포넌트에서 사용할 수 있게 해주는 useContext 훅

useContext는 상위 컴포넌트에서 만들어진 Context를 함수 컴포넌트에서 사용할 수 있도록 만들어진 훅이다.

```jsx
const Context = (createContext < { hello: string }) | (undefined > undefined);

function ParentComponent() {
	return (
		<>
			<Context.Provider value={{ hello: 'react' }}>
				<Context.Provider value={{ hello: 'javascript' }}>
					<ChildComponent />
				</Context.Provider>
			</Context.Provider>
		</>
	);
}

function ChildComponent() {
	const value = useContext(Context);

	// react가 아닌 javascript가 반환된다.
	return <>{value ? value.hello : ''}</>;
}
```

useContext를 사용하면 상위 컴포넌트 어딘가에서 선언된 <Context.Provider />에서 제공한 값을 사용할 수 있다. 만약, 여러개의 Provider가 존재한다면 가장 가까운 Provider의 값을 가져오게 된다.  
따라서, 예제에서는 가장 가까운 Provider인 'javascript'가 반환된다.

컴포넌트 트리가 복잡해질수록 콘텍스트를 사용하는 것이 쉽지 않다.  
useContext로 원하는 값을 얻으려 했지만, 실행될 때 해당 콘텍스트가 존재하지 않아 예상치 못하게 에러가 발생할 수 있다.  
이러한 에러를 방지하려면 useContext 내부에서 해당 콘텍스트가 존재하는 환경인지, 즉 콘텍스트가 한 번이라도 초기화되어 값을 내려주고 있는지 확인하면 된다.

```jsx
const MyContext = createContext<{hello: string} | undefined>(undefined)

function ContextProvider({
	children,text
}: PropsWithChildren<{text: string}>) {
	return (
		<MyContext.Provider value={{ hello: text}}>{children}</MyContext.Provider>
	)
}

function useMyContext() {
	const context = useContext(MyContext)
	if(context === undefined) {
	throw ...
	}
	return context
}

function ChildComponent() {
	const { hello } = useMyContext()

	return <>{hello}</>

}

function ParentComponent() {
	return (
		<>
			<Context.Provider text="react">
				<ChildComponent />
			</Context.Provider>
		</>
	)
}
```

다수의 Provider와 useContext를 사용한다면 (특히 타입스크립트 사용), 별도의 함수로 감싸서 사용하는 것이 좋다. 타입 추론에 유용하고, 상위에 Provider가 없는 경우 사전에 에러를 찾기 쉽다.

#### useContext 사용시 주의할 점

useContext를 함수 컴포넌트 내부에서 사용할 때는 항상 컴포넌트 재활용이 어려워진다.  
Provider에 의존성을 가지고 있는 셈이므로 아무 곳에서나 재활용하기 어려운 컴포넌트가 된다. 즉, useContext가 있는 컴포넌트는 사용 순간부터 눈으로 직접 보이지 않을 수도 있는 Provider와의 의존성을 갖게 되는 것이다.  
이런 상황을 방지하기 위해서는 useContext를 사용하는 컴포넌트를 최대한 작게 하거나, 재사용되지 않을 만한 컴포넌트에서 사용해야 한다.

useContext를 상태 관리를 위한 리액트 API로 오해한다.  
콘텍스트는 **상태를 주입** 해주는 API이다. 상태 관리 라이브러리가 될 수 없다.

> **상태 관리 라이브러리 조건**
>
> 1.  어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 한다.
> 2.  필요에 따라 이러한 상태 변화를 최적화할 수 있어야 한다.

useContext는 위 조건 모두에 부합하지 않는다. 단순히 props 값을 하위로 전달 해 줄 뿐, 렌더링이 최적화되지 않는다. 그렇기 때문에 Provider의 값이 변경될 때 어떤 식으로 렌더링되는지 눈여겨 봐야 한다.

### 3.1.7 useReducer

useReducer는 useState의 심화 버전으로 볼 수 있다.  
useState와 비슷한 형태를 띠지만 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.

**useReducer에서 사용되는 용어**

- 반환값은 useState와 동일하게 길이가 2인 배열이다.
  - state: 현재 useReducer가 가지고 있는 값을 의미한다. useState와 마찬가지로 배열을 반환하는데, 동일하게 첫번째 요소가 해당 값이다.
  - dispatcher: state를 업데이트하는 함수이다. useReducer가 반환하는 배열의 두번째 요소이다. setState는 단순히 값을 넘겨주지만 여기서는 action을 넘겨준다는 점이 다르다. action은 state를 변경할 수 있는 액션을 의미한다.
- useState의 인수와 달리 2개에서 3개의 인수를 필요로 한다.
  - reducer: useReducer의 기본 action을 정의하는 함수다. 이 reducer은 useReducer의 첫번째 인수로 넘겨줘야 한다.
  - initialState: 두번째 인수로, useReducer의 초기값을 의미한다.
  - init: useState의 인수로 함수를 넘겨줄 때처럼 초기값을 지연해서 생성시키고 싶을 때 사용하는 함수다. 해당 함수는 필수값이 아니며, 만약 여기에 인수를 넘겨주는 함수가 존재한다면 useState와 동일하게 게으른 초기화가 일어나며 initialState를 인수로 init 함수가 실행된다.

```jsx
// useReducer가 사용할 state를 정의
type State = {
	count: number
}

// state의 변화를 발생시킬 action의 타입과 넘겨줄 값(payload)을 정의
// 꼭 type과 payload라는 네이밍을 지킬 필요 X, 객체일 필요 X
// 다만 해당 네이밍이 가장 널리 사용됨
type Action = { type: 'up' | 'down' | 'reset'; payload?: State}

// 무거운 연산이 포함된 게으른 초기화 함수
function init(count: state): State {]
	// count: State를 받아서 초깃값을 어떻게 정의할지 연산
	return count
}

// 초깃값
const initialState: State = { count: 0 }

// 앞서 선언한 state와 action을 기반으로 state가 어떻게 변경될지 정의
function reducer(state: State, action: Action): State {
	switch (action.type){
		case 'up':
			return { count: state.count + 1 }
		case 'down':
			return { count: state.count - 1 > 0 ? state.count - 1 : 0 }
		case 'reset':
			return init(action.payload || { count: 0 })
		default:
			throw new Error(`Unexpected action type ${action.type}`)
	}
}

export default function App() {
	const [state, dispatcher] = useReducer(reducer, initialState, init)

	function handleButtonClick() {
		dispatcher({ type: 'up'})
	}

	// ...

	return (
		<div class="App">
		...
		</div>
	)
}
```

복잡한 형태의 state를 사전에 정의된 dispatcher로만 수정할 수 있게 만들어 줌으로써 state 값에 대한 접근은 컴포넌트에서만 가능하다. 이를 업데이트하는 방법에 대한 상세 정의는 컴포넌트 밖에다 둔 다음, state의 업데이트를 미리 정의해 둔 dispatcher로만 제한한다.  
**state 값을 변경하는 시나리오를 제한적으로 두고 이에 대한 변경을 빠르게 확인할 수 있게끔 하는 것이 useReducer의 목적이다.**

세번째 인수인 게이른 초기화 함수는 굳이 사용하지 않아도 된다. 해당 함수가 없다면 두번째 인수로 넘겨받은 기본값을 사용하게 될 것이다.  
다만, 게으른 초기화 함수를 넣어줌으로 useState에 함수를 넣은 것과 같은 동일한 이점을 누릴 수 있다. 추가로 state에 대한 초기화가 필요할 때 reducer에서 이를 재활용 할 수도 있다.

useReducer나 useState 둘 다 세부 작동과 쓰임에만 차이가 있을 뿐, 클로저를 활용해 값을 가둬서 state를 관리한다는 사실은 동일하다. 그렇기에 필요에 맞게 useReducer나 useState를 골라서 선택하면 된다.

### 3.1.8 useImperativeHandle

useImperativeHandle은 개발에 널리 사용되지 않지만, 일부 사례에서 유용하게 활용할 수 있다.  
useImperativeHandle을 이해하기 위해서 React.forwardRef를 먼저 알아야 한다.

#### forwardRef

ref는 useRef에서 반환한 객체로, 리액트 컴포넌트의 props ref에 넣어 HTMLElement에 접근하는 용도로 흔히 사용된다.  
이런 ref를 상위 컴포넌트에서 하위 컴포넌트로 전달하고 싶다면 어떻게 할까?

예약어로 지정된 ref 대신에 다른 props로 받으면 된다.  
예약어로 지정된 ref로 할 경우 props로 쓸 수 없다는 경고문과 undefined를 반환한다.

forwardRef는 위에 설명한 것과 동일한 작업을 하는 리액트 API이다.  
그렇다면 props로 구현할 수 있는 것을 왜 만든걸까?

이유는 ref를 전달하는데 일관성을 제공하기 위해서이다.  
네이밍의 자유가 존재하는 props보다는 forwardRef를 사용하면 좀 더 확실하게 ref를 전달함을 예측할 수 있다. 또한, 사용하는 쪽에서 안정적으로 받아서 사용할 수 있다.

```jsx
const ChildComponent = forwardRef((props, ref) => {
	useEffect(() => {
		console.log(ref);
	}, [ref]);

	return <div>Hello</div>;
});

function ParentComponent() {
	const inputRef = useRef();

	return (
		<>
			<input ref={inputRef} />
			<ChildComponent ref={inputRef} />
		</>
	);
}
```

ref를 받고자 하는 컴포넌트를 forwardRef로 감싸고, 두번째 인수로 ref를 전달받는다. 그리고 부모 컴포넌트에서 동일하게 props.ref를 통해 ref를 넘겨준다.

#### useImperativeHandle

useImperativeHandle은 부모에게서 넘겨받은 ref를 원하는대로 수정할 수 있는 훅이다.

```jsx
const Input = forwardRef((props, ref) => {
	// useImperativeHandle을 사용하면 ref의 동작을 추가로 정의할 수 있다.
	useImperativeHandle(
		ref,
		() => ({
			alter: () => alert(props.value),
		}),
		// useEffect의 deps와 같다.
		[props.value]
	);

	return <input ref={ref} {...props} />;
});

function App() {
	// input에서 사용할 ref
	const inputRef = useRef();
	// input의 value
	const [text, setText] = useState('');

	function handleClick() {
		// inputRef에 추가한 alert라는 동작을 사용할 수 있다.
		inputRef.current.alert();
	}

	function handleChange(e) {
		setText(e.target.value);
	}

	return (
		<>
			<Input ref={inputRef} value={text} onChange={handleChange} />
			<button onClick={handleClick}>Focus</button>
		</>
	);
}
```

원래 ref는 `{current: <HTMLElement>}`와 같은 형태로 HTMLElement만 주입할 수 있는 객체였다. 그러나 위 코드에서는 ref에 useImperativeHandle 훅을 사용해 추가적인 동작을 정의했다. 이로 인해, 부모는 HTMLElement 뿐만 아니라 자식 컴포넌트에서 새롭게 설정한 객체의 키와 값에 대해서도 접근할 수 있게 됐다.  
useImperativeHandle을 사용하면 ref의 값에 원하는 값이나 액션을 정의할 수 있다.

### 3.1.9 useLayoutEffect

useLayoutEffect는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.  
DOM의 변경이란, 렌더링을 뜻한다. 브라우저에서 실제로 해당 변경 사항이 반영되는 시점을 의미하는 것이 아니다.

```jsx
useEffect(() => {
	console.log('useEffect', count);
}, [count]);

useLayoutEffect(() => {
	console.log('useLayoutEffect', count);
}, [count]);
```

**실행순서**

1. 리액트가 DOM을 업데이트
2. useLayoutEffect를 실행
3. 브라우저 변경 사항을 반영
4. useEffect를 실행

useEffect가 먼저 선언돼 있지만 useLayoutEffect가 먼저 실행된다.  
동기적으로 발생한다는 것은 useLayoutEffect의 실행이 종료될 때까지 기다린 다음에 화면을 그린다는 것을 의미한다. 즉, 리액트 컴포넌트는 useLayoutEffect가 완료될 때까지 기다려 컴포넌트가 잠시 일시중지 되는 것과 같은 일이 발생하게 되어 애플리케이션 성능에 문제가 발생할 수 있다.

따라서, useLayoutEffect는 **DOM은 계산됐지만 화면에 반영되기 전에 하고 싶은 작업이 있을 때**와 같이 반드시 필요한 경우에만 사용하는 것이 좋다.

### 3.1.10 useDebugValue

useDebugValue는 사용자 정의 훅 내부의 내용에 대한 정보를 남길 수 있는 훅이다. 일반적으로 프로덕션 웹서비스에서 사용하는 훅이 아니다.  
해당 훅은 리액트 애플리케이션을 개발하는 과정에서 사용되며, 디버깅하고 싶은 정보를 해당 훅에다 사용하면 리액트 개발자 도구에서 볼 수 있다.

두번째 인수로 포매팅 함수를 전달하면 이에 대한 값이 변경됐을 때만 호출되어 포매팅된 값을 노출한다. 즉, 첫번째 인수와 두번째 인수의 값이 같으면 포매팅 함수는 호출되지 않는다.

useDebugValue는 **오직 다른 훅 내부에서만 실행할 수 있음**을 주의해야 한다. 컴포넌트 레벨에서 실행한다면 작동되지 않는다.  
그렇기에 공통 훅을 제공하는 라이브러리나 대규모 웹 애플리케이션에서 디버깅 관련 정보를 제공하고 싶을 때 유용하게 사용할 수 있다.

### 3.1.11 훅의 규칙

리액트에서 제공하는 훅을 사용하는데 몇가지 규칙이 존재하는데, 이를 **rules-of-hooks**라고 한다. 이와 관련된 ESLint 규칙인 react-hooks/rules-of-hooks도 존재한다.

#### rules-of-hooks

1. 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 이 규칙을 따라야 컴포넌트가 렌더링될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장할 수 있다.

2. 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트, 혹은 사용자 정의 훅의 두가지 경우 뿐이다. 일반 자바스크립트 함수에서는 훅을 사용할 수 없다.

**주의**  
조건이나 다른 이슈로 인해 훅의 순서가 깨지거나 보장되지 않을 경우 리액트 코드는 에러를 발생시킨다.  
그러므로 훅은 절대 조건문, 반복문 등에 의해 리액트에서 예측 불가능한 순서로 실행되어서는 안된다.  
만약 조건문이 필요하다면 반드시 훅 내부에서 수행해야 한다.
