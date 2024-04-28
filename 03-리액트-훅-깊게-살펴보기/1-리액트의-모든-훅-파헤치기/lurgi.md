# 🚀 리액트의 모든 훅 파헤치기

## 1️⃣ useState

1. `useEffect`와 함께 상태 관리를 할 수 있게 해주는 훅.
2. 기본 사용법

   ```tsx
   import { useState } from "react";

   const [state, setState] = useState(initValue);
   ```

3. 아무런 값도 넘겨주지 않으면 초깃값은 undefined 이다.

   리액트의 렌더링은 함수 컴포넌트에서 반환한 결과물인 return의 값을 비교해 실행되기 때문에 다음과 같은 상황에서는 렌더링이 되지 않는다.

   ```tsx
   function Component() {
     const [_, triggerRender] = useState();
     let state = "hello";

     function handleButtonClick() {
       state = "hi";
       triggerRender();
     }

     return (
       <>
         <h1>{state}</h1>
         <button onClick={handleButtonClick}>Click</button>
       </>
     );
   }
   ```

4. 리액트는 클로저를 활용하였다. 어떤 함수(`useState`) 내부에 선언된 함수(`setState`)가 실행이 종료된 이후에도 지역변수인 state를 계속 참조할 수 있다는 것을 의미한다.
5. 간략한 버전의 코드는 다음과 같다. (실제 코드는 내부적으로 `useReducer` 훅을 사용해 구현돼있다)

   ```tsx
   const global = {};
   let index = 0;

   function useState(initValue) {
     if (!global.states) {
       global.states = [];
     }

     const currentState = global.states[index] || initialState;
     global.states[index] = currentState;

     const setState = (function () {
       //클로저로 감싸 계속 동일한 index에 접근할 수 있다.
       let currentIndex = index;
       return function (value) {
         global.states[currentIndex] = value;
       };
     })();

     index = index + 1;

     return [currentState, setState];
   }
   ```

### ❗️게으른 초기화

1. `useState()`의 인수로 대부분 원싯값을 사용하겠지만, 특정한 함수를 인수로 넣어줄 수도 있다.이를 게으른 초기화(lazy initialization)라고 한다.

   ```tsx
   // 일반적인 사용방법
   const [count, setCount] = useState(Number.parseInt(window.localStorage.getItem(cacheKey)));

   // 게으른 초기화
   const [count, setCount] = useState(() => Number.parseInt(window.localStorage.getItem(cacheKey)));
   ```

2. 게으른 초기화는 `useState()`의 초깃값이 복잡하거나 무거운 연산일 때 사용하면 된다.
3. 만약 리렌더링이 발생한다면 이후 내부 함수의 실행은 무시된다.

## 2️⃣ useEffect

1. 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 메커니즘이다.
2. 리액트 공식문서에서는 다음과 같이 정의한다.

   `useEffect`는 [외부 시스템과 컴포넌트를 동기화](https://ko.react.dev/learn/synchronizing-with-effects)하는 React Hook입니다.

3. 일반적인 형태는 다음과 같습니다.

   ```tsx
   function Component() {
     //...
     useEffect(() => {
       // do something
     }, [props, state]);
   }
   ```

4. 첫 번째 인자로 함수를, 두 번째 인자로 의존성 배열을 받는다. 의존성 배열의 경우 아무런 값이 없는 빈 배열일 수도 있고, 두 번째 인자 자체를 넣지 않아도 된다.
5. 렌더링할 때마다 의존성에 있는 값을 비교하며 이전과 다른 게 하나라도 있으면 부수 효과를 실행하는 평범한 함수라고 할 수 있다.

### ❗️ 주의 사항!

기본적으로, Effect는 _모든_ 렌더링 후에 실행됩니다. 이러한 이유로 다음과 같은 코드는 **무한 루프를 만들어낼** 것입니다.

```tsx
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(count + 1);
});
```

Effect는 렌더링의 *결과*로 실행됩니다. state를 설정하면 렌더링이 *트리거*됩니다. Effect 안에서 즉시 상태를 설정하는 것은 기계의 전원 플러그를 기계 그 자체에 연결하는 것과 비슷합니다. Effect가 실행되고 상태가 설정되면 재렌더링이 발생하고, Effect가 다시 실행되고 상태가 설정되면 또 다른 재렌더링이 발생하며, 이런 식으로 계속됩니다.

### ❗️ 클린업 함수의 목적

1. 클린업 함수는 이벤트를 등록하고 지울 때 사용해야 하는 것으로 알려져 있다.
2. 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지한다.
3. 클린업 함수는 함수가 정의되었을 당시에 선언되었던 이전 값을 보고 실행된다.

```tsx
const [counter, setCounter] = useState(0);

useEffect(() => {
  function addMouseEvent() {
    //이벤트 로직
    setCounter((prev) => prev + 1);
    console.log(counter); // 함수가 정의되었을 때의 값 +1이 찍힌다.
  }
  window.addEventListener("click", addMouseEvent);

  return () => {
    console.log(counter); // 함수가 정의되었을 때의 값이 찍힌다.
    window.removeEventListener("click", addMouseEvent);
  };
}, [counter]);
```

1. 언마운트는 특정 컴포넌트가 DOM에서 사라진다는 것을 의미하는 클래스 컴포넌트의 용어로, 클린업 함수는 언마운트 라기보다는, 함수 컴포넌트가 리렌더링 되었을 때 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행하는, ‘청소’ 함수이다.

### ❗️ 의존성 배열

1. 빈 배열을 둔다면 최초 렌더링 직후에 실행된 다음 더 이상 실행되지 않는다.
2. 아무런 값도 넘겨주지 않는다면, 렌더링할 때마다 실행된다.
3. 의존성 배열이 없는 `useEffect`가 매 렌더링시 실행된다면, 없애면 되는게 아닐까?

   ```tsx
   function Component{
   	console.log('렌더링')
   	//...
   }

   function Component{
   	useEffect(()=>{
   		console.log('렌더링')
   	})
   }
   ```

   → 이 둘은 명백한 차이를 두고 있다.

   1. 서버 사이드 렌더링 관점에서, `useEffect`는 클라이언트 사이드에서 실행되는 것을 보장한다.
   2. `useEffect`는 컴포넌트의 렌더링이 완료된 후 실행되지만, 함수 내부 직접 실행할 경우 컴포넌트가 렌더링되는 도중에 실행된다. 이 작업은 함수 컴포넌트의 반환을 지연시키는 행위다.

### ❗️useEffect의 구현

1. 코드를 대략적으로 구현한다면 다음과 같을 것이다.

```tsx
const global = {};
let index = 0;

function useEffect(callback, dependencies){
	const hooks = global.hooks;

	let prevDependencies = hooks[index];

	let isDependenciesChanged = prevDependencies ? dependencies.some(
		(value, idx) => !Object.is(value, prevDependencies[index])
		// Object.is를 기반으로 하는 얕은 비교.
	);

	if(isDependenciesChanged) {
		callback();

		index += 1;

		hooks[index] = dependencies;
	}

	return { useEffect }
}
```

### ❗️ useEffect를 사용할 때 주의할 점

1. 의존성 배열을 빈 배열로 사용하는 경우.

   ```tsx
   useEffect(() => {
     //...
   }, []);
   ```

   1. 이는 클래스 컴포넌트의 생명주기 메서드인 `componentDidMount`에 기반한 접근법으로 가급적 사용해선 안된다.
   2. `useEffect`는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행돼야 하는 훅이다.
   3. 정말로 의존성 빈 배열이 필요하다면, **최초의 함수 컴포넌트가 마운트 됐을 시점에만 콜백 함수 실행이 필요한지**를 다시 한번 되물어 봐야한다.
   4. 만약 ‘그렇다’라고 했다면, `useEffect`의 부수효과 함수가 실행될 위치가 잘못 되었을 가능성이 크다.

      ```tsx
      function Component({ log }) {
        useEffect(() => {
          logging(log);
        }, []);
      }
      ```

   5. `useEffect`의 흐름과 컴포넌트의 흐름이 맞지 않는 버그를 안은 위 코드는 잘못되었을 가능성이 크다.
   6. 위의 부수 효과 함수는 log를 전달하는 부모 컴포넌트에서 실행되는 것이 옳을지도 모른다.

2. useEffect의 콜백 인수로 비동기 함수를 바로 넣을 수 없다.

   1. useEffect의 인수로 비동기 함수가 사용 가능하다면 비동기 함수의 응답속도에 따라 결과가 이상하게 나타날 수 있다.
   2. 극단적 예제로 처음 state기반의 응답이 10초 걸렸고, 두번째 응답이 1초가 걸렸다면, 결과가 뒤봐뀌는 불상사가 생길 수 있다. 이러한 문제를 `useEffect`의 경쟁상태(race condition)라고 한다.

      ```tsx
      useEffect(async () => {
        const response = await fetch(URL);
        const result = await response.json();
        setData(result);
      }, []);
      ```

   3. 비동기 함수 실행 자체가 문제가 되는 것은 아니기 때문에 다음과 같이 비동기 함수를 만들어 사용할 수 있다.

      ```tsx
      useEffect(() => {
        let shouldIgnore = false;

        async function fetchData() {
          const response = await fetch(URL);
          const result = await response.json();
          if (!shouldIgnore) {
            setData(result);
          }
        }

        fetchData();

        return () => {
          shouldIgnore = true;
        };
      }, []);
      ```

### 3️⃣ useMemo

1. `useMemo`는 비용이 큰 연산에 대한 결과를 저장(메모이제이션)하고 반환하는 훅이다.
2. 첫 번째 인수로 어떠한 값을 반환하는 생성함수를, 두 번째 인수로 해당함수가 의존하는 값의 배열을 전달한다. `useMemo`는 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 함수를 재실행 하지 않고 기억해둔 값을 반환한다.

   ```tsx
   const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b]);
   ```

3. 단순한 값 뿐만 아니라 컴포넌트도 가능하다. 하지만 컴포넌트는 `React.memo`를 사용하는 것이 현명하다.

   → 이유를 못찾았습니다. 결론은 memo랑 useMemo를 분리할 이유가 있을까? 로 결론내렸습니다.

   ```tsx
   const Component = () => {
     //...
     return <Children />;
   };

   export default React.memo(Component);
   ```

   ```tsx
   const Component = () => {
     //...
     return useMemo(() => <Children />, []);
   };

   export default Component;
   ```

   둘은 같지 않나?

### 4️⃣ useCallback

1. `useMemo`가 값을 기억했다면, `useCallback`은 인수로 넘겨받은 콜백 자체를 기억한다.

   ```tsx
   const Input = () => {
     /*...*/
   }; //useMemo를 사용한 컴포넌트

   function App() {
     const [state1, setState1] = useState();
     const [state2, setState2] = useState();

     const toggle1 = (e) => setState1(e.target.value);
     const toggle2 = (e) => setState2(e.target.value);

     return (
       <>
         <Input value={state1} onChange={(e) => toggle1(e)} />
         <Input value={state2} onChange={(e) => toggle2(e)} />
       </>
     );
   }
   ```

2. 위 코드는 메모이제이션 했지만, 한 버튼을 클릭했을 때 클릭한 컴포넌트 이외에도 다른 컴포넌트가 리렌더링 되는 것을 확인할 수 있다.
3. 이는 state가 바뀌면서 App컴포넌트가 리렌더링 되면서 `toggle` 함수가 재생성 되고 있기 때문이다.
4. `useCallback` 을 이용하여 함수를 재생성을 막을 수 있다.

   ```tsx
   const Input = () => {
     /*...*/
   }; //useMemo를 사용한 컴포넌트

   function App() {
     const [state1, setState1] = useState();
     const [state2, setState2] = useState();

     const toggle1 = (e) => setState1(e.target.value);
     const toggle2 = (e) => setState2(e.target.value);

     return (
       <>
         <Input value={state1} onChange={(e) => toggle1(e)} />
         <Input value={state2} onChange={(e) => toggle2(e)} />
       </>
     );
   }
   ```

5. 기본적으로 useCallback은 useMemo를 이용하여 구현할 수 있다. Preact의 코드는 다음과 같다.

   ```tsx
   export function useCallback(callback, args) {
     currentHook = 8;
     return useMemo(() => callback, args);
   }
   ```

   useMemo를 따로 제공하는 이유로는 **불필요하게 코드가 길어짐을 방지하는 것** 이유일 수 있다.

### 5️⃣ useRef

1. useState와 마찬가지로, 렌더링이 일어나도 변경 가능한 상태값을 저장한다는 공통점이 있다. useState와 구별되는 큰 차이점은 다음과 같다.
   1. useRef는 반환값 자체 있는 current로 값에 접근, 변경할 수 있다.
   2. useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.
2. 렌더링에 영향을 미치지 않는 고정된 값을 관리하기 위해서 다음과 같이 사용하면 안될까? 굳이 useRef를 사용하는 이유가 뭘까?

   ```tsx
   let value = 0;

   function Component() {
     function handleClick() {
       value += 1;
     }
     //...
   }
   ```

   위 방식에는 단점이 있다.

   1. 컴포넌트가 실행되어 렌더링 되지 않았을 때도 value라는 값이 기본적으로 존재한다. 이는 메모리에 불필요한 값을 저장한다는 뜻이다.
   2. 컴포넌트가 여러개 생성되었을 경우에도, 바라보는 값이 value로 동일하다.
   3. useRef는 위 단점을 해결해줄 리액트적 접근법이다.

3. 이 useRef를 활용하여 usePrevious훅을 구현할 수 있다.

   ```tsx
   function usePrevious (value) {
   	const ref = useRef();
   	useEffect(()=>{
   		ref.current = value
   	},[value])
   	return ref.current
   }

   function Component(){
   	const [counter, setCounter] = useState(0);
   	const previousCounter = usePrevious(counter);


   ```

4. Preact에서 useRef는 다음과 같이 구현되어 있다.

   ```tsx
   export function useRef(initValue) {
     currentHook = 5;
     return useMemo(() => ({ current: initValue }), []);
   }
   ```

## 6️⃣ useContext

1. 부모가 가진 데이터를 자식에게 넘겨주기 위해서 props를 사용하는 것이 일반적이다.
2. 컴포넌트의 거리가 멀어질수록 코드가 복잡해진다.
3. props 내려주기를 극복하기 위해서 등장한 개념이 Context이고, 함수형 컴포넌트에서 이 Context를 사용하기 위한 훅이 useContext이다.

```tsx
const Context = createContext(undefined);

function ParentComponent() {
  return (
    <Context.Provider value={{ hello: "hi" }}>
      <Context.Provider value={{ hello: "hello" }}>
        <ChildComponent />
      </Context.Provider>
    </Context.Provider>
  );
}

function ChildComponent() {
  const value = useContext(Context);

  return <>{value ? value.hello : ""}</>; // hello가 출력된다.
}
```

1. 가까운 컨텍스트 Provider의 값을 반환한다.
2. useContext 내부에 해당 컨텍스트가 존재하는 환경인지 다음과 같이 확인할 수 있다.

```tsx
import { useContext } from "react";

const useContextWrapper = <T,>(contextValue: React.Context<T>) => {
  const context = useContext(contextValue);

  if (!context) {
    throw new Error("[ERROR] ContextProvider 내부에서만 사용할 수 있습니다.");
  }
  return context;
};

export default useContextWrapper;
```

### ❗️useContext를 사용할 때 주의할 점❗️

1. Provider에 의존성을 가지고 있기 때문에 아무데서나 재활용할 수 없다.

   **위와 같은 문제를 해결하기 위해서 최상위 루트 컴포넌트에 넣는 것은 어떨까?**

   이는 해당props를 다수의 컴포넌트에서 사용할 수 있게끔 해야 하므로 불필요하게 리소스가 낭비된다.

2. useContext를 사용한다고 해서 렌더링이 최적화되지 않는다. 즉 내부의 자식 요소들은 모두 리렌더링된다. 그러니 context를 사용하는 환경에서 리렌더링을 막기 위해선, React.memo를 사용해야 한다.

## 7️⃣ useReducer

1. useState와 비슷한 기능을 가지지만, 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다.
2. 형태는 다음과 같다.

   ```tsx
   const [state, dispatcher] = useReducer(reducer, initialState, init);
   ```

   1. dispatcher는 state를 업데이트하는 함수이다. setState는 단순히 값을 넘겨주지만, 여기서는 action을 넘겨준다.
   2. action을 정의하는 함수다.
   3. initialState는 useReducer의 초깃값을 설정한다.
   4. init은 필수값은 아니고, 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수이다. 즉 state의 게으른 초기화라고 생각하면 된다.

   ```tsx
   function reducer(state, action) {
     switch (action.type) {
       case "up":
         return { count: state.count + 1 };
       case "down":
         return { count: state.count - 1 > 0 ? state.count - 1 : 0 };
       case "reset":
         return { count: 0 };
       default:
         throw new Error("Unexpected action type");
     }
   }

   export default function App() {
     const [count, dispatcher] = useReducer(reducer, { count: 0 });

     function handleUp() {
       dispatcher({ type: "up" });
     }

     function handleDown() {
       dispatcher({ type: "down" });
     }

     function handleReset() {
       dispatcher({ type: "reset" });
     }

     // ...
   }
   ```

## 8️⃣ useImperativeHandle

1. 잘 사용되지 않는 훅이지만, forwardRef를 사용할 때 유용하다.

### ❗️ forwardRef 살펴보기

1. ref를 자식 컴포넌트에 넘겨주기 위해서는 이 forwardRef가 필요하다.
2. ref는 props로 넘겨줄 수 없다. 넘겨주기 위해선 ref를 parentRef라는 ref가 아닌 다른 이름으로 넘겨줄 수 있다.

   ```tsx
   function Child({ parentRef }) {
     useEffect(() => {
       console.log(parentRef);
     }, [parentRef]);

     return; //...
   }

   function Parent() {
     const inputRef = useRef();

     return (
       <>
         <input ref={inputRef} />
         <Child parentRef={inputRef} />
       </>
     );
   }
   ```

3. 위 처럼 넘겨줄 수 있지만, ref를 전달하는 데 있어서 일관성을 제공하기 위해 forwardRef를 사용한다.

   ```tsx
   const Child = forwardRef((_, ref) => {
     useEffect(() => {
       console.log(ref);
     }, [ref]);

     return; //...
   });

   function Parent() {
     const inputRef = useRef();

     return (
       <>
         <input ref={inputRef} />
         <Child ref={inputRef} />
       </>
     );
   }
   ```

### ❗️ 그래서 useImperativeHandle이 뭔데?

useImperativeHandle는 부모요소에서 받은 ref를 마음대로 수정할 수 있는 훅이다.

```tsx
const Child = forwardRef((props, ref) => {
  useImperativeHandle(
    ref,
    () => ({
      alert: () => alert(props.value),
    }),
    [props, value]
  );

  return; //...
});

function Parent() {
  const inputRef = useRef();

  function handleClick() {
    inputRef.current.alert(); // alert를 사용할 수 있다.
  }

  return (
    <>
      <input ref={inputRef} />
      <Child ref={inputRef} />
    </>
  );
}
```

## 9️⃣ useLayoutEffect

1. `이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.`
2. 실행 순서는 다음과 같다.
   1. 리액트가 DOM을 업데이트
   2. useLayoutEffect를 실행
   3. 브라우저에 변경 사항을 반영
   4. useEffect를 실행
3. 즉 useEffect가 먼저 실행되어 있어도, useLayoutEffect가 항상 먼저 실행된다.
4. 그리고 동기적으로 발생한다는 것은, useLayoutEffect가 모두 실행이 끝난 이후 화면을 그린다는 것을 의미한다.

### ❓ 언제쓰면 좋을까?

1. DOM요소를 기반으로 한 애니메이션, 스크롤 위치를 제어하는 등 화면에 반영되기 전에 하고싶은 작업들은 이 useLayoutEffect을 사용하면 좋다. 자연스러운 사용자 경험을 제공할 수 있다.

## 🔟 훅의 규칙

1. 최상위에서만 훅을 호출해야 한다. 반복문, 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다.
2. 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트 혹은 사용자 정의 훅 두 가지 경우 뿐이다.
