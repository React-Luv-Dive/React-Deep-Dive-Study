### 3.1.1 useState

**useState**

**: 함수형 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 하는 훅**

**구현 살펴보기**

```jsx
import { useState } from 'react';

const [state, setState] = useState(initialState);
```

인수 : state 초기값, 없을 시 undefined

반환 값 : 배열

배열의 첫 번째 원소 state를 값 자체로 사용하고.

배열의 두번째 원소 setState 함수를 사용해 해당 state 를 변경할 수 있다.

**Lazy initialization**

일반적으로른 기본값을 선언할 때 원시값을 넣지만, 특정한 값을 넘기는 함수를 인수로 넣어줄 수 도 있다.

useState에 변수 대신 함수를 넘기는 것 == 게으른 초기화

사용하는 상황

: 초기값이 복잡하거나 무거운 연산을 포함하고 있을 때

( localStorage 나 sessionStorage에 대한 접근, map, filter, find 같은 배열에 대한 접근, 혹은 초기값 계산을 위해 함수 호출이 필요할 때와 같이 무거운 연산이 포함된 경우)

게으른 초기화 함수는 오직 state가 처음 만들어질 때만 사용된다. 리랜더링이 발생해도 함수 실행은 무시됨

```jsx
const [state, setState] = useState(() => {
  console.log('복잡한 연산...');
  // 이 컴포넌트가 처음 구동될 때만 실행되고, 리랜더링 시에는 실행되지 않는다.
});
```

만약 Number.parseInt(window.localStorage.getItem(cacheKey)) 와 같이 한번 실행되는데 어느정도 비용이 드는 값이 있을 때 이를 인수 그 자체로 사용하면 초기값이 필요없는 리랜더링 시에도 동일하게 계쏙 값에 접근해서 낭비가 발생한다. 이 경우에는 함수 형태로 인수를 넘겨주는 편이 훨씬 경제적일 것이다.
