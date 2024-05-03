# 🚀 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

## 1️⃣ 사용자 정의 훅

1. 서로 다른 컴포넌트 내부에서 같은 로직을 공유하고자 할 때 주로 사용되는 것이 바로 사용자 정의 훅이다.
2. 사용자 정의 훅은 반드시 use로 시작하는 함수를 만들어야 한다. react-hooks/rules-of-hook의 도움을 받기 위해서.
3. 다음은 fetch를 수행하는 사용자 정의 훅이다.

   ```tsx
   import { uesEffect, useState } from "react";

   function useFetch<T>(url: string, { method, body }: { method: string; body?: XMLHttpRequestBodyInit }) {
     const [result, setResult] = useState<T | undefined>();
     const [isLoading, setIsLoading] = useState<boolean>(false);
     const [ok, setOk] = useState<boolean | undefined>();
     const [status, setStatus] = useState<number | undefined>();

     useEffect(() => {
       const abortController = new AbortController();

       (async () => {
         //IIFE
         setIsLoading(true);

         const res = await fetch(url, { method, body, signal: abortController });

         setOk(response.ok);
         setStatus(response.status);

         if (response.ok) {
           const apiResult = await res.json();
           setResult(apiResult);
         }

         setIsLoading(false);
       })(); // IIFE 실행

       return () => {
         abortController.abort();
       };
     }, [url, method, body]);

     return { ok, result, isLoading, status };
   }
   ```

## 2️⃣ 고차 컴포넌트

1. HOC, Higher Order Component. 컴포넌트 자체의 로직을 재사용하기 위한 방법이다.
2. 자바스크립트의 일급 객체, 함수의 특징을 이용하므로 리액트가 아니더라도 js환경에서 사용할 수 있다.
3. 리액트에서의 유명한 고차 컴포넌트로 React.memo가 있다.

### ❓React.memo?

1. 리액트 컴포넌트는 부모 컴포넌트가 새로 렌더링 될 때마다 자식 컴포넌트는 새로 렌더링된다.
2. 자식 컴포넌트의 props가 변경되지 않았음에도, 렌더링이 일어나는데, 이를 방지하기 위해 만들어진 리액트의 고차 컴포넌트가 React.memo이다.
3. 그렇다면, 컴포넌트도 값이라는 관점에서 본다면 React.memo대신 useMemo를 사용해도 되지 않을까?

   ```tsx
   function MemoizedChild = () => {
   		return useMemo(() => <ChildComponent value="hello"/>)
   	})

   function ParentComponent() {
   	const [state, setState] = useState(1);

   	const handleChange = (e: ChangeEv두t<HTMLInputElement>) => {
   		setState(Number(e.target.value))
   	}

   	return (
   		<>
   			<input type="number" value={state} onChange={handleChange} />
   			<MemoizedChild/>
   		</>
   	)
   }
   ```

### ❓ 고차함수란 무엇인가?

1. 고차함수의 정의는 함수를 인자로 받거나 결과로 반환하는 함수이다.
2. 예시로, useState의 경우, 클로저를 사용한다. 즉시실행함수를 반환하는 고차함수이다.
3. 고차함수를 활용하면 함수를 인수로 받거나 새로운 함수를 반환해 완전히 새로운 결과를 만들어 낼 수 있다.

### ❗️고차함수를 활용하여 리액트 고차 컴포넌트 만들기

```tsx
function withLoginComponent<T>(Component: ComponentType<T>) {
  return function (props: T & LoginProps) {
    const { loginRequired, ...restProps } = props;

    if (loginRequired) {
      return <>로그인이 필요합니다</>;
    }

    return <Component {...(restProps as T)} />;
  };
}
```

1. 물론 이런 인증 처리는 서버단에서 처리하는 것이 훨씬 효율적일 것이다.
2. 리액트의 고차 컴포넌트도 마찬가지로 with로 시작하는 이름을 사용해야한다.
3. 고차 컴포넌트는 부수 효과를 최소화해야 한다. 즉 props를 임의로 수정, 추가, 삭제하는 일은 없어야 한다.

```tsx
const Component = withHigherOrderComponent(
	withHigherOrderComponent1(
		withHigherOrderComponent2(
			withHigherOrderComponent3(
				withHigherOrderComponent4(
					withHigherOrderComponent5(
						return <div>Hello</div>
	)))))

```

1. 여러개의 고차 컴포넌트를 반복적으로 사용하는 경우, 복잡성이 커지므로 고차 컴포넌트는 최소한으로 사용하는 것이 좋다.

## 3️⃣ 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### ❗️ 사용자 정의 훅이 필요한 경우

1. 단순히 useState, useEffect와 같이 리액트에서 제공하는 훅으로 공통 로직을 격리할 수 있다면, 사용자 정의 훅을 사용하는 것이 좋다.

### ❗️ 고차 컴포넌트를 사용해야 하는 경우

1. 애플리케이션 관점에서 컴포넌트를 감추고 로그인을 요구하는 공통 컴포넌트를 노출하는 것이 좋을 경우.

```tsx
function Hook() {
  const { loggedIn } = useLogin();

  if (!loggedIn) {
    return <LoginComponent />;
  }

  return <div>Hi</div>;
}
```

```tsx
const HOC = withLoginComponent(() => {
  return <div>Hi</div>;
});
```

1. 사용자 정의 훅은 해당 컴포넌트가 반환하는 렌더링 결과물에 영향을 미치기 어렵기 때문에, 이러한 경우 고차 컴포넌트를 사용하면 좋다.
