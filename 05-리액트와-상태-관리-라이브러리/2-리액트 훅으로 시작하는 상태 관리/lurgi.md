# 🚀 리액트 훅으로 시작하는 상태 관리

## 1️⃣ 가장 기본적인 방법: useState와 useReducer

```tsx
function useCounter(initCount: number = 0) {
  const [counter, setCounter] = useState(initCount);

  function init() {
    setCounter((prev) => prev + 1);
  }

  return { count, inc };
}
```

```tsx
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

function useStateWithUseReducer<T>(initialState: T) {
  const [state, dispatch] = useReducer(
    (prev: T, action: Initializer<T>) => (typeof action === "function" ? action(prev) : action),
    initialState
  );
  return [state, dispatch];
}
```

1. useCounter을 사용하여 여러 컴포넌트에서 손쉽게 재사용할 수 있는 훅을 만들었다.
2. useState와 useReducer모두 지역 상태 관리를 위해 만들어졌다.
3. 만약 지역 상태인 counter를 전역 상태로 만들어 컴포넌트가 사용하는 모든 훅이 동일한 값을 참조하게 하려면 어떻게 해야할까?

## 2️⃣ 지역 상태의 한계를 벗어나보자

```tsx
const state = {
  counter: 0,
};

function Counter1() {
  const [count, setCount] = useState(state);

  function handleClick() {
    set((prev) => {
      const newState = { counter: prev.counter + 1 };
      setCount(newState);
      return newState;
    });
  }
  //...
}

function Counter2() {
  const [count, setCount] = useState(state);

  function handleClick() {
    set((prev) => {
      const newState = { counter: prev.counter + 1 };
      setCount(newState);
      return newState;
    });
  }
  //...
}
```

1. 예제 코드는 억지로 전역에 있는 상태를 참조하게 만들었다.
2. 그러나 외부에서 상태를 관리함에도, 각각의 컴포넌트를 눌렀을 때 이상하게 작동하는 것을 볼 수 있다.
3. 같은 상태를 바롸봐야 하는 반대쪽 컴포넌트에서는 리렌더링이 일어나지 않는다는 것.
4. 반대쪽 컴포넌트를 눌러야 그제서야 렌더링이되어 최신값을 불러온다.

---

### → 외부에서 상태를 참조하고, 이를 통해 렌더링까지 자연스럽게 일어나려면 다음과 같은 조건을 만족해야 한다.

1. 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트가 사용할 수 있어야 한다.
2. 외부에 있는 상태를 사용하는 컴포넌트는 상태의 변화를 알아챌 수 있어야 하고, 상태가 변할 때 마다 리렌더링이 발생해야 한다.
3. 상태가 원시값이 아닌 객체인 경우엔 감지하지 않는 값 이외의 객체 값이 변한다 해도 리렌더링이 발생되어선 안된다.

```tsx
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

type Store<State> = {
  get: () => State;
  set: (action: Initializer) => State;
  subscribe: (callback: () => void) => () => void;
};

export const createStore = <State extends unknown>(initialState: Initializer<State>): Store<State> => {
  let State = typeof initialState !== "function" ? initialState : initialState();

  const callbacks = new Set<() => void>();
  const get = () => state;
  const set = (nextState: State | ((prev: State) => State)) => {
    state = typeof nextState === "function" ? (nextState as (prev: State) => State)(state) : nextState;

    callbacks.forEach((callback) => callback());
    return state;
  };

  const subscribe = (callback: () => void) => {
    callbacks.add(callback);

    return () => {
      callback.delete(callback);
    };
  };

  return { get, set, subscribe };
};
```

1. createStore는 자신이 관리해야 하는 상태를 내부 변수로 가진 다음, get 함수로 해당 변수의 최신값을 제공한다.
2. 또 set함수로 내부 변수를 최신화 하면서, 등록된 콜백을 모조리 실행한다.
3. createStore로 만들어진 store의 값을 참조하고, 컴포넌트의 렌더링을 유도할 사용자 정의 훅이 필요하다. useStore라는 훅은 다음과 같다.

```tsx
type Store<State extends unknown> = {
  get: () => State;
  set: (action: Initializer) => State;
  subscribe: (callback: () => void) => () => void;
};

export const useStore = <State extends unknown>(store: Store<State>) => {
  const [state, setState] = useState<State>(() => store.get());

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.get());
    });

    return unsubscribe;
  }, [store]);

  return [state, store.set] as const;
};
```

1. 이렇게 만든 useStore도 완벽한 것은 아니다. 만약 스토어의 구조가 객체인 경우 해당 객체의 일부값만 변경한다해도 해당 값 이외의 모든 것들이 리렌더링이 일어날 것이다
2. useStore훅을 변경하여, 변경 감지가 필요한 값에서만 setState를 호출해 객체 상태에 대한 불필요한 리렌더링을 막을 수 있다.

```tsx
type Store<State extends unknown> = {
  get: () => State;
  set: (action: Initializer) => State;
  subscribe: (callback: () => void) => () => void;
};

export const useStoreSelector = <State extends unknown, Value extends unknown>(
  store: Store<State>,
  selector: (state: State) => Value
) => {
  const [state, setState] = useState(() => selector(store.get()));

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      const value = selector(store.get());
      setState(value);
    });

    return unsubscribe;
  }, [store, selector]);

  return state;
};
```

1. 위와 같은 useStoreSelector훅을 이용해, 객체에서 원하는 값만 불러와 렌더링 효율성을 증가시킬 수 있다.

```tsx
const store = createStore({ count: 0, text: "hi" });

function Counter() {
  const counter = useStoreSelector(
    store,
    useCallback((state) => state.count, [])
  );

  //...
}
```

### 3️⃣ useSubscription

1. 리액트 외부에서 관리되는 값에 대한 변경을 추적하고, 이를 리렌더링까지 할 수 있는 훅은 미리 존재한다. 바로 useSubscription 훅이다.

```tsx
const store = createStore({ count: 0, text: "hi" });

function NewCounter() {
  const subscription = useMemo(
    () => ({
      getCurrentValue: () => store.get(),
      subscribe: (callback: () => void) => {
        const unsubscribe = store.subscribe(callback);
        return () => unsubscribe();
      },
    }),
    []
  );

  const value = useSubscription(subscription);

  //...
}
```

1. useSubscription을 타입스크립트로 흉내낸 코드는 다음과 같다.

```tsx
export function useSubscription<Value>({
  getCurrentValue,
  subscribe,
}: {
  getCurrentValue: () => Value;
  subscribe: (callback: Function) => () => void;
}): Value {
  const [state, setState] = useState(() => ({
    getCurrentValue,
    subscribe,
    value: getCurrentValue(),
  }));

  let valueToReturn = state.value;

  if (state.getCurrentValue !== getCurrentValue || state.subscribe !== subscribe) {
    valueToReturn = getCurrentValue();

    setState({
      getCurrentValue,
      subscribe,
      value: valueToReturn,
    });
  }

  useDebugValue(valueToReturn);

  useEffect(() => {
    let didUnsubscribe = false;

    const checkForUpdates = () => {
      // 이미 구독이 취소되었다면 아무것도 하지 않는다.
      if (didUnsubscribe) return;

      const value = getCurrentValue();

      setState((prevState) => {
        // 단순히 함수의 차이만 있다면,
        // 이전 값과 현재 값이 같다면 이전 값을 반환한다.
        if (
          prevState.getCurrentValue !== getCurrentValue ||
          prevState.subscribe !== subscribe ||
          prevState.value === value
        ) {
          return prevState;
        }

        return { ...prevState, value };
      });
    };

    const unsubscribe = subscribe(checkForUpdates);

    checkForUpdates();

    return () => {
      didUnsubscribe = true;
      unsubscribe();
    };
  }, [getCurrentValue, subscribe]);

  return valueToReturn;
}
```

1. 리액트 18버전의 useSubscription을 살펴보면 useSyncExternalStore로 재작성되었다.

### 4️⃣ useState와 Context를 동시에 쓰기

1. 훅과 스토어를 사용하는 구조는, 반드시 하나의 스토어만 가지게 된다.
2. 즉 동일한 형태의 어러 개의 스토어를 가지려면 아래와 같이 번거로운 작업을 통해 구현해야만 한다.

```tsx
const store1 = createStore({ count: 0 });
const store2 = createStore({ count: 0 });
const store3 = createStore({ count: 0 });
```

1. 이런 부분을 Context를 이용하여 해당 스토어를 하위 컴포넌트에 주입할 수 있고, 이를 통해 문제를 해결할 수 있다.

```tsx
export const CounterStoreContext = createContext<Store<CounterStore>>(
	createStore<CounterStore>({ count: 0, text: "hello" }),
)

export const CounterStoreProvider = ({
	initialState,
	children,
}): PropsWithChildren<{
	initialState: CounterState
}>) => {
	const storeRef = useRef<Store<CounterStore>>()

	if(!storeRef.current){
		storeRef.current = createStore(initialState)
	}

	return (
		<CounterStoreContext.Provider value={storeRef.current}>
			{children}
		</CounterStoreContext.Provider>
	)
}
```

1. 위 코드에서 storeRef를 사용하는데, 그 이유는 Provider로 넘기는 props가 불필요하게 변경돼서 리렌더링되는 것을 막기 위해서다.

```tsx
export const useCounterContextSelector = <State extends unknown>(selector: (state: CounterStore) => State) => {
  const store = useContext(CounterStoreContext);

  const subscription = useSubscription(
    useMemo(
      () => ({
        getCurrentValue: () => selector(store.get()),
        subscribe: store.subscribe,
      }),
      [store, selector]
    )
  );

  return [subscription, store.get] as const;
};
```

1. 위 새로운 훅과 Context를 사용하는 방법은 다음과 같다

```tsx
//...
const [counter, setStore] = useCounterContextSelector(useCallback((state: CounterStore) => state.count, []));

const [text, setStore] = useCounterContextSelector(useCallback((state: CounterStore) => state.text, []));
```
