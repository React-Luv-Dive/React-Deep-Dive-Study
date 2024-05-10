## 5.2 리액트 훅으로 시작하는 상태 관리

### 5.2.1 가장 기본적인 방법: useState와 useReducer

useState혹은 useReducer를 통해 useCounter라는 커스텀 훅을 만들 수 있다. 해당 커스텀 훅은 각 컴포넌트의 counter 상태를 관리한다.  
하지만, 이 counter는 지역 상태로 여러 컴포넌트에서 동시에 사용할 수 없다. 지역 상태인 counter를 전역 상태로 만들기 위해서 상태를 컴포넌트 밖으로 한 단계 끌어 올리면 된다.

useCounter라는 커스텀 훅을 각 컴포넌트에서 사용하지 않고 Parent라는 상위 컴포넌트에서만 사용한다. 해당 훅의 반환값을 하위 컴포넌트의 props로 제공한다.  
지역 상태였던 useCounter를 부모 컴포넌트로 한 단계 끌어 올리고 해당 값을 하위 컴포넌트에 참조하게 사용함으로 counter를 전역 상태로 관리할 수 있게 한다.

하지만 이런 props 형태로 필요한 컴포넌트에 제공해야 한다는 것은 불편하다.

### 5.2.2 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기

useState와 useEffect를 통해 useStoreSelector, useStore 등의 커스텀 훅을 만들어 리액트 외부에서 관리되는 값에 대한 변경을 추적하고 리렌더링 하도록 할 수 있다.
이런 것을 페이스북팀에서 만든 useSubscription을 통해 수행할 수 있다.
