## 3.2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### 3.2.1 사용자 정의 훅

서로 다른 컴포넌트 내부에서 같은 로직을 공유하고자 할 때 주로 사용된다. 이는 리액트에서만 사용할 수 있다.  
리액트 훅 뿐만 아니라 사용자 정의 훅 또한 use로 시작하는 함수를 만들어야 한다.

사용자 정의 훅은 내부에서 useState와 useEffect 등을 가지고 자신만의 훅을 만드는 기법이다. 내부에서 useState와 같은 리액트 훅을 사용하고 있기 때문에 리액트 훅의 규칙을 따라야 한다.  
이 리액트 훅의 규칙을 따르고 react-hooks/rules-of-hooks의 도움을 받기 위해서는 use로 시작하는 이름을 가져야 한다. 아니면 에러가 발생한다.

```jsx
function fetch<T>(
	url:string,
	{ method, body }: { method: string; body?: XMLHttpRequestBodyInit },
	){
		// React Hook "useState" is called in function "fetch" that is neither
		// a React function component nor a custom React function. React component
		// names must start with an uppercase letter. (react-hooks/rules-of-hooks)
		const [result, setResult] = useState<T | undefined>()
		// ...
	}
```

react-hooks/rules-of-hooks에서 훅은 함수 컴포넌트 내부 또는 사용자 정의 훅 내부에서만 사용할 수 있다고 에러가 발생한다.  
해당 경고는 함수 이름 fetch 앞부분을 대문자로 바꿔 함수 컴포넌트임을 알리는 것, 혹은 use를 앞에 붙여 사용자 정의 훅임을 알리는 것이다.

### 3.2.2 고차 컴포넌트

고차 컴포넌트(HOC, Higher Order Component)는 컴포넌트 자체의 로직을 재사용하기 위한 방법이다.  
사용자 정의 훅은 리액트에서만 사용할 수 있다면, 고차 컴포넌트는 고차함수(Higher Order Function)의 일종이다. 자바스크립트의 일급 객체, 함수의 특징을 이용하므로 자바스크립트 환경이라면 사용할 수 있다.

리액트에서 고차 컴포넌트의 기법으로 다양한 최적화나 중복 로직 관리를 할 수 있다.  
가장 유명한 리액트 고차 컴포넌트는 리액트에서 제공하는 API중 하나인 React.memo이다.

#### React.memo

```jsx
const ChildComponent = ({ value }: { value: string }) => {
	useEffect(() => {
		console.log('렌더링!');
	});

	return <>Hello! {value}</>;
};

function ParentComponent() {
	const [state, setState] = useState();

	function handleChange(e: ChangeEvent<HTMLInputElement>) {
		setState(Number(e.target.value));
	}

	return (
		<>
			<input type="number" value={state} onChange={handleChange} />
			<ChildComponent value="안녕하세요" />
		</>
	);
}
```

위 예제에서 ChildComponent는 props인 value="안녕하세요"가 변경되지 않았음에도 handleChange로 인해 setState를 실행해 state를 변경하므로 리렌더링이 발생한다.  
이처럼 props의 변화가 없음에도 컴포넌트의 렌더링을 방지하기 위해 만들어진 리액트의 고차 컴포넌트가 바로 **React.memo**이다.

React.memo는 렌더링하기에 앞서 props를 비교해서 이전과 props가 같다면 렌더링 자체를 생략하고 이전에 기억해 둔(memoization) 컴포넌트를 반환한다.

memo로 감싸서 다시 실행하게 된다면 ParentComponent에서 아무리 state가 변동되어도 ChildComponent는 다시 렌더링되지 않는다.  
props가 변경되지 않았고, 변경되지 않았다는 것을 memo가 확인하고 이전에 기억한 컴포넌트를 그대로 반환한 것이다.  
따라서 불필요한 렌더링 작업을 생략할 수 있다.

**React.memo는 컴포넌트도 값이라는 관점에서 본 것이므로 useMemo를 사용해도 동일하게 메모이제이션 할 수 있지 않을까?**

useMemo는 값을 반환하기 때문에 JSX 함수 방식이 아닌 {}을 사용한 할당식을 사용한다는 차이점이 있다.  
그렇기에 목적과 용도가 뚜렷한 memo를 사용하는 것이 좋다.

#### 고차 함수

고차함수? 함수를 인수로 받거나 결과로 반환하는 함수  
대표적인 고차함수: Array.prototype.map, forEach, reduce 등  
고차 함수를 활용하여 함수를 인수로 받거나 새로운 함수를 반환해 완전히 새로운 결과를 만들어 낼 수 있다.

#### 고차 컴포넌트

고차 함수의 특징에 따라 개발자가 만든 또 다른 함수를 반환할 수 있다는 점에서 고차 컴포넌트는 유용하다.

**규칙**

- with로 시작하는 이름을 사용해야 한다.

  강제되는 사항은 아니지미나 리액트 커뮤니티에 퍼진 관습이다.

- 부수효과를 최소화해야 한다.

  컴포넌트를 인수로 받게 되는데, 반드시 컴포넌트의 props를 임의로 수정, 추가, 삭제하는 일은 없어야 한다.  
  고차 컴포넌트의 props를 추가 했을 뿐, 기존 컴포넌트의 props를 건드리는 것은 안된다.

- 여러 개의 고차 컴포넌트로 컴포넌트를 감쌀 경우 복잡성이 커진다.

  고차 컴포넌트는 최소한으로 사용하는 것이 좋다.

### 3.2.3 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

사용자 정의 훅과 고차 컴포넌트 모두 리액트 코드에서 어떤 로직을 공통화해 별도로 관리할 수 없다는 특징이 있다. 애플리케이션 전반에 필요한 중복 로직을 분리해서 컴포넌트의 크기를 줄이고 가독성을 향상 시킬 수 있다.

#### 사용자 정의 훅이 필요한 경우

리액트에서 제공하는 훅으로만 공통 로직을 분리할 수 있다면 사용자 정의 훅을 사용하는 것이 좋다.  
렌더링에 영향을 미치지 못하기 때문에 사용이 제한적이다. 그렇기에 컴포넌트 내부에 미치는 영향을 최소화해 개발자가 훅을 원하는 방향으로만 사용할 수 있다는 장점이 있다.

```jsx
// 사용자 정의 훅을 사용하는 경우
function HookComponent() {
	const { loggedIn } = useLogin();

	useEffect(() => {
		if (!loggedIn) {
			// do something
		}
	}, [loggedIn]);
}

// 고차 컴포넌트를 사용하는 경우
const HOCComponent = withLoginComponent(() => {
	// do something
});
```

useLogin은 단순히 loggedIn에 대한 값만 제공할 뿐, 이에 대한 처리는 컴포넌트를 사용하는 쪽에서 원하는 대로 사용 가능하다. 그렇기에 부수 효과가 비교적 제한적이다.

withLoginComponent는 고차 컴포넌트가 어떤 일과 결과물을 반환하는지는 고차 컴포넌트를 직접 보거나 실행해야 알 수 있다.  
대부분의 고차 컴포넌트는 렌더링에 영향을 미치는 로직이 존재하여 사용자 정의 훅에 비해 예측하기 어렵다.

단순히 컴포넌트 전반에 걸쳐 동일한 로직으로 값을 제공하거나 특정한 훅의 작동을 취하게 하고 싶을때 사용자 정의 훅을 사용하는 것이 좋다.

#### 고차 컴포넌트를 사용해야 하는 경우

컴포넌트가 반환하는 렌더링 결과물에 영향을 끼치고 싶은 경우  
이런 중복 처리가 해당 사용자 정의 훅을 사용하는 애플리케이션 전반에 걸쳐 나타날 때, 고차 컴포넌트를 사용해 처리하는 것이 좋다.

**EX)**

애플리케이션 관점에서 컴포넌트를 감추고 로그인을 요구하는 공통 컴포넌트를 노출하고 싶을 때.  
어떤 특정 에러가 발생했을 때 현재 컴포넌트 대신 에러가 발생했음을 알리는 컴포넌트를 노출하고 싶을 때.

함수 컴포넌트의 반환값, 즉 렌더링의 결과물에 영향을 미치는 공통 로직이라면 고차 컴포넌트가 좋다.  
단, 고차 컴포넌트가 많을수록 복잡성이 급증하므로 신중하게 사용해야 한다.
