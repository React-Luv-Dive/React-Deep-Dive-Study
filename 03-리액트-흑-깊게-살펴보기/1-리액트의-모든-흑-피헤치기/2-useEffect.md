### 3.1.2 useEffect

**useEffect : 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 매커니즘**

**useEffect 란?**

```jsx
useEffect(() => {
	// do something
	}. [props, state])
```

첫번째 인수로는 실행할 부수 효과가 포함된 함수를, 두번째 인수로는 의존성 배열을 전달

그리고 의존성 배열 내부 값이 변경될 때마다 첫번째 인수인 콜백을 실행한다.

**클린업 함수의 목적**

클린업 함수 : 이벤트를 등록하고 지울 때 사용하는 것

```jsx
import { useState, useEffect } from 'react'

export default function App() {
	const [counter, setCounter] = useState(0)

	function handleαick() {
		setCounter((prev) => prev + 1)
	}

	useEffect(() => {
		function addMouseEvent() {
			console.1og(counter)
		}

		window.addEventListener('click', addMouseEvent)

		// 클린업 함수
		return () => {
			console.1og('클린업함수실행!',counter)
			window.removeEventListener('click',addMouseEvent)
		}
	}, [counter])

retum (
	<>
		<h1>{counter}</h1>
		<buttononcIick={handleCIick)>+</button> </>
	)
}
```

실행 결과

```jsx
클린업 함수 실행! 0
1
클린업 함수 실행! 1
2
클린업 함수 실행! 2
3
클린업 함수 실행! 2
4
```

→ 클린업 함수는 이전 counter 값, 즉 이전 state를 참조해 실행됨

클린업 합수는 새로운 값과 함께 랜더링된 뒤에 실행되기 때문에

: 클린업 함수는 비록 새로운 값을 기반으로 랜더링 뒤에 실행되지만, 이 변경된 값을 읽는 것이 아니라 함수 정의됐을 당시 선언했던 이전 값을 보고 실행된다.

**의존성 배열**

: 빈 배열을 두거나, 아무것도 넘기지 않거나, 사용자가 원하는 값을 넣어줄 수 있다.

빈 배열 : 비교할 의존성이 없다고 판단해 최초 랜더링 직후 실행된 다음부터 더 이상 실행되지 않는다.

```jsx
useEffect(() => {
  console.log('빈 배열');
}, []);
```

아무것도 안넘김 : 의존성을 비교할 핋요 없이 랜더링할 때마다 실행이 필요하다고 판단해 랜더링이 발생할 때마다 실행된다.

보통 컴포넌트가 랜더링됐는지 확인하기 위해 사용된다.

```jsx
useEffect(() => {
  console.log('컴포넌트 랜더링됨');
});
```

그냥 console.log("컴포넌트 랜더링됨")만 적었을 때랑 차이점

1. 서버 사이드 랜더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해준다. useEffect 내부에서는 window 객체의 접근에 의존하는 코드를 사용해도 된다.

2. useEffect는 컴포넌트 랜더링의 부수 효과, 즉 컴포넌트 랜더링이 완료된 이후 실행됨. 반명 직접 실행(콘솔 예제)는 컴포넌트가 랜더링되는 도중에 실행된다. 따라서 1번과 달리 서버 사이드 랜더링의 경우에 서버에서도 실행이된다. 이 작업은 함수형 컴포넌트의 빈환을 지연시키는 행위댜 즉 무거운 작업일 경우 렌더링을 닐馮 M하므로 성능에 익영향을 미칠 수 었댜

⇒ useEffect의 effect 는 컴포넌트의 사이드 이펙트, 즉 부수 효과를 의미한다.

useEffect는 컴포넌트가 랜더링된 후에도 어떠한 부수 효과를 일으키고 싶을 때 사용된다.

**useEffect 사용할 때 주의할 점**

1. eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제하라

저 eslint 룰은 useEffect 인수 내부에서 사용하는 값 중 의존성 배열에 포함되어 있지 않는 값이 있을 때 경고를 발생시킨다.

이렇게 사용되는 때는 주로 빈 배열 []을 의존성으로 할 때, 즉 컴포넌트를 미운트하는 시점 에만 무언가를 하고 싶다라는 의도로 직성하곤 한다.

그러나 이는 클래스형 컴포넌트의 생명주기 메서드인
componentDidMount에 기반한 접근법으로, 가급적이면 사용해선 안된다.

useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행돼야 하는 혹이다. 그러나 의존성 배열 을 넘기지 않은 채 콜백 힘수 내부에서 특정 값을 사용한디는 것은, 이 부수 효과가 실제로 관칠해서 실행돼 야 히는 값괴는 별개로 직동한다는 것을 의미한다.

정말로 의존성으로 []가 필요히디면 최초에 함수형 컴포넌트가 미운트 됐을 시점에만 콜백 힘수 실행이 필요한 지를 다시 한번 되물어뵈야 한다. 만약 정말 ‘그렇다’리고 하면 useEffect 내 부수 효과가 실행될 위 치가 질못됐을 기능성이 크다

2. useEffect 의 첫번째 인수에 힘수명을 부여하라

useEffect의 인수를 익명 함수가 아닌 적절한 이름을 시용한 기명 함수로 바꾸는 것이 좋다. 우리가 변수에 적절한 이름을 붙이는 이유는 해당 변수가 왜 만들어졌는지 파악하기 위함이다. useEffect 도 미찬가지로적절한 이름을 붙이면 해당 useEffect의 목적을 파익하기 쉬워진다.

```jsx
useEffect(
	function 1ogActiveUser() {
		1ogging(user.id)
	},
	[user.id],
)
```

3. 거대한 useEffect를 만들지마라

크기가 커질수록 애플리케이션 성능에 악영힝을 미친다. 가능한 useEffect는 간결하고 가볍게 유지히는 것이 좋다. 만약 부득이하게 큰 useEffect를 만들어야 한다면 적은 의존성 배열을 시용하는 여러 개의 useEffect로 분리히는 것이 좋다.

만약 의존성 배열에 불가피하게 여러 변수가 들어가야 하는 상황이라면 최대한 useCallback과 useName 등으로 사전에 정제한 내용들만 useEffect에 담아두는것이 좋다

4. 불필요한 외부 함수를 만들지마라
