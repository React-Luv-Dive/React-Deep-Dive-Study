## 10.2 리액트 18 버전 살펴보기

### 10.2.1 새로 추가된 훅 살펴보기

#### useId

컴포넌트별로 유니크한 값을 생성하는 새로운 훅이다.  
컴포넌트 내부에서 사용할 유니크한 값을 생성하는 것은 쉽지 않다. 컴포넌트가 여러 군데에서 재사용 되는 경우를 고려하고, 리액트 컴포넌트 트리에서 컴포넌트가 가지는 모든 값이 겹치지 않고 다 달라야 한다는 제약이 존재하기 때문이다.  
또한, 서버 사이드 렌더링 환경에서 하이드레이션이 일어날 때도 서버와 클라이언트에서 동일한 값을 가져야 에러가 발생하지 않으므로 이런 점도 고려해야 한다.

새로운 훅인 useId를 사용하면 클라이언트와 서버에서 불일치를 피하면서 컴포넌트 내부의 고유한 값을 생성할 수 있게 된다.

> **하이드레이션**  
> 서버에서 렌더링 한 페이지에 스크립트 코드를 집어넣어 웹 페이지를 동적으로 처리하는 것  
> 하이드레이션을 활용해 CSR과 SSR의 장점을 모두 가진 웹 페이지를 개발할 수 있습니다.

#### useTransition

UI 변경을 가로막지 않고 상태를 업데이트할 수 있는 리액트 훅이다.  
상태 업데이트를 긴급하지 않은 것으로 간주해 무거운 렌더링 작업을 조금 미룰 수 있다. 그리고 사용자에게 더 나은 사용자 환경을 제공할 수 있다.

상태 변경으로 인해 무거운 작업이 발생하고, 이로 인해 렌더링이 가로막힐 여지가 있는 경우에 useTransition을 사용하면 이런 문제를 해결할 수 있다.

useTransition은 리액트 18의 변경 사항의 핵심 중 하나인 '동시성(concurrency)'을 다룰 수 있는 새로운 훅이다. 과거 리액트의 모든 렌더링은 동기적으로 작동해 느린 렌더링 작업이 있을 경우 애플리케이션 전체적으로 영향을 끼쳤다. 그러나 useTransition과 같은 동시성을 지원하는 기능을 사용하면 느린 렌더링 과정에서 로딩 화면을 보여주거나 혹은 지금 진행 중인 렌더링을 버리고 새로운 상태값으로 다시 렌더링하는 등의 작업을 할 수 있게 된다.

useTransition은 컴포넌트에서만 사용이 가능한 훅이다. 훅을 사용할 수 없는 상황이라면 단순히 startTransition을 바로 import 할 수도 있다.

**useTransition을 사용할 때 주의할 점**

- startTransition 내부는 반드시 setState와 같은 상태를 업데이트하는 함수와 관련된 작업만 넘길 수 있다.
- startTransition으로 넘겨주는 상태 업데이트는 다른 모든 동기 상태 업데이트로 인해 실행이 지연될 수 있다.
- startTransition으로 넘겨주는 함수는 반드시 동기 함수여야 한다.

#### useDeferredValue

useDeferredValue는 리액트 컴포넌트 트리에서 리렌더링이 급하지 않은 부분을 지연할 수 있도록 도와주는 훅이다.  
디바운스와 비슷하지만 디바운스 대비 useDeferredValue만이 가진 장점이 몇가지 있다.

> **디바운스**  
> 특정 시간 동안 발생하는 이벤트를 하나로 인식해 한 번만 실행하게 해준다.

**useDeferredValue 만의 장점**

- 고정된 지연 시간 없이 첫번째 렌더링이 완료된 이후에 이 useDeferredValue로 지연된 렌더링을 수행한다.  
  디바운스는 고정된 지연 시간을 필요로 한다. 그러나 useDeferredValue는 고정된 지연 시간이 없어서 지연된 렌더링을 중단할 수 있다. 또한 사용자 인터렉션을 차단하지 않는다.
- list를 생성하는 기준을 text가 아닌 deferredText로 설정함으로써 잦은 변경이 있는 text를 먼저 업데이트해 렌더링하고, 이후 여유가 있을 때 지연된 deferredText를 활용해 list를 새로 생성하게 된다.  
  list에 있는 작업이 더 무겁고 오래 걸릴수록 useDeferredValue를 사용하는 이점을 더욱 누릴 수 있다.

**useDeferredValue와 useTransition의 차이점**

- useTransition은 state 값을 업데이트 하는 함수를 감싸서 사용한다. useDeferredValue는 state 값 자체만을 감싸서 사용한다.  
  방식의 차이가 존재할 뿐 지연된 렌더링을 한다는 점에서 모두 동일한 역할을 한다. 그렇기에 두 가지를 모두 동시에 사용할 필요 없이, 상황에 맞는 방법을 선택하면 된다.
- 낮은 우선순위로 처리해야 할 작업에 대해 직접적으로 상태를 업데이트할 수 있는 코드에 접근할 수 있다면, useTransition이 좋다.
- 컴포넌트의 props와 같이 상태 업데이트에 관여할 수 없고 오로지 값만 받아야 하는 상황이라면 useDeferredValue를 사용하는 것이 좋다.

#### useSyncExternalStore

useSyncExternalStore는 일반적인 애플리케이션 코드를 작성할 때는 사용할 일이 별로 없는 훅이다.

> **티어링 (tearing)**  
> 영어로 '찢어진다'라는 뜻으로, 리액트에서는 하나의 state 값이 있음에도 서로 다른 값(보통 state나 props의 이전과 이후)을 기준으로 렌더링되는 현상을 말한다.

리액트 17에서는 이런 현상이 일어날 여지가 없었으나, 리액트 18에서는 앞서 useTransition, useDeferredValue의 훅처럼 렌더링을 일시 중지하거나 뒤로 미루는 등의 최적화가 가능해지면서 동시성 이슈가 발생할 수 있게 됐다.

그런데 이런 티어링 현상을 해결할 수 있는 훅이 바로 useSyncExternalStore이다.  
외부에 상태가 있는 데이터에는 반드시 useSyncExternalStore를 사용해 값을 가져와야 startTransition 등으로 인한 티어링 현상이 발생하지 않을 수 있다.

useSyncExternalStore는 애플리케이션 코드에서 직접적으로 사용할 일은 많지 않지만, 사용 중인 관리 라이브러리가 외부에서 상태를 관리하고 있다면 이 useSyncExternalStore를 통해 외부 데이터 소스의 변경을 추적하고 있는지 반드시 확인해야 한다. 만약 해당 라이브러리가 이 훅을 사용하고 있지 않다면 렌더링 중간에 발생하는 값 업데이트를 적절하게 처리하지 못하고 티어링 현상이 발생할 것이다.

#### useInsertionEffect

useSyncExternalStore가 상태 관리 라이브러리를 위한 훅이라면 useInsertionEffect는 CSS-in-js 라이브러리를 위한 훅이다.

CSS의 추가및 수정은 브라우저에서 렌더링하는 작업 대부분을 다시 계산에 작업해야 하는데, 이는 리액트 관점으로 본다면 모든 리액트 컴포넌트에 영향을 미친 수도 있는 매우 무거운 작업이다.  
따라서 리액트 17과 styled-components에서는 클라이언트 렌더링 시에 이러한 작업이 ㅂ라생하지 않도록 서버 사이드에서 스타일 코드를 삽입했다. 그러나 이 작업을 훅으로 처리하는 것이 쉽지 않았는데, 훅에서 이런 작업을 할 수 있도록 도와주는 새로운 훅이 바로 useInsertionEffect다.

useInsertionEffect의 기본적인 훅 구조는 useEffect와 동일하다. 다만 useEffect와 다른 차이점은 바로 실행 시점이다.
useInsertionEffect는 DOM이 실제로 변경되기 전에 동기적으로 실행된다. 이 훅 내부에 스타일을 삽입하는 코드를 집어 넣음으로써 브라우저가 레이아웃을 계산하기 전에 실행될 수 있게끔 해서 좀 더 자연스러운 스타일 삽입이 가능해진다.

**useEffect, useLayoutEffect, useInsertionEffect의 실행 순서**

```jsx
function Index() {
	useEffect(() => {
		console.log('useEffect'); //3
	});
	useLayoutEffect(() => {
		console.log('useLayoutEffect'); // 2
	});
	useInsertionEffect(() => {
		console.log('useInsertionEffect'); // 1
	});
}
```

useLayoutEffect와 비교했을 때 실행되는 타이밍이 미묘하게 다르다. 두 훅 모두 브라우저에 DOM이 렌더링되기 전에 실행된다는 공통점이 존재한다.  
그러나 useLayoutEffect는 모든 DOM의 변경 작업이 다 끝난 이후에 실행되는 반면 useInsertionEffect는 이러한 DOM 변경 작업 이전에 실행된다.  
이런 차이는 브라우저가 다시금 스타일을 입혀서 DOM을 재계산하지 않아도 된다는 점에서 매우 크다.

리액트에서 권고하는 것처럼 useSyncExternalStore와 마찬가지로 useInsertionEffect는 실제 애플리케이션 코드를 작성할 때는 사용될 일이 거의 없으므로 라이브러리를 작성하는 경우가 아니라면 참고만 하고 실제 애플리케이션 코드에서는 가급적 사용하지 않는 것이 좋다.

### 10.2.2 react-dom/client

#### createRoot

기존의 react-dom에 있던 render 메서드를 대체할 새로운 메서드이다.  
리액트 18의 기능을 사용하고 싶다면 createRoot와 render를 함께 사용해야 한다.

```jsx
// before
import ReactDom from 'react-dom';
import App from 'App';

const container = document.getElementById('root');

ReactDOM.render(<App />, container);

// after
import ReactDom from 'react-dom';
import App from 'App';

const container = document.getElementById('root');

const root = ReactDOM.createRoot(container);
root.render(<App />);
```

#### hydrateRoot

서브 사이드 렌더링 애플리케이션에서 하이드레이션을 하기 위한 새로운 메서드다.

```jsx
// before
import ReactDom from 'react-dom';
import App from 'App';

const container = document.getElementById('root');

ReactDOM.hydrate(<App />, container);

// after
import ReactDom from 'react-dom';
import App from 'App';

const container = document.getElementById('root');

const root = ReactDOM.hydrateRoot(container, <App />);
```

대부분의 서버 사이드 렌더링은 프레임워크에 의존하고 있을 것이므로 사용하는 쪽에서 수정할 일은 거의 없는 코드다. 다만 자체적으로 서버 사이드 렌더링을 구현해서 사용하고 있다면 해당 부분 역시 수정해야 한다.

API가 변경된 것 외에도 추가로 두 API는 새로운 옵션인 onRecoverableError를 인수로 받는다. 해당 옵션은 리액트가 렌더링 또는 하이드레이션 과정에서 에러가 발생했을 때 실행하는 콜백 함수이다.  
기본 값으로 reportError 또는 console.error를 사용하지만 필요하다면 원하는 내용을 추가해도 된다.

### 10.2.3 react-dom/server

#### renderToPipeableStream

리액트 컴포넌트를 HTML로 렌더링하는 메서드이다.
해당 메서드는 스트림을 지원하는 메서드로, HTML을 점직적으로 렌더링하고 클라이언트에서는 중간에 script를 삽입하는 등의 작업을 할 수 있다.  
이를 통해 서버에서는 Suspense를 사용해 빠르게 렌더링이 필요한 부분을 먼저 렌더링 할 수 있다. 또, 값비싼 연산으로 구성된 부분은 이후에 렌더링되게끔 할 수 있다.

추가로 hydrateRoot를 호출하면 서버에서는 HTML을 렌더링하고, 클라이언트의 리액트에서는 여기에 이벤트만 추가함으로써 첫번째 로딩을 매우 빠르게 수행할 수 있다.

기존 renderToNodeStream의 문제는 무조건 렌더링을 순서대로 해야 하고, 그리고 그 순서에 의존적이기 때문에 이전 렌더링이 완료되지 않는다면 이후 렌더링도 끝나지 않는다는 것이다. 따라서 렌더링 중간에 오래 걸리는 작업이 있다면 그 작업 때문에 나머지 렌더링도 덩달아 지연된다는 문제가 있다.  
그러나 renderToPipeableStream을 활용하면 순서나 오래 걸리는 렌더링에 영향받을 필요 없이 빠르게 렌더링을 수행할 수 있게 된다.

### 10.2.4 자동 배치 (Automatic Batching)

리액트가 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 성능을 향상시키는 방법을 의미한다.  
예를 들어, 버튼 클릭 한 번에 두개 이상의 state를 동시에 업데이트한다. 자동 배치에서는 이를 하나의 리렌더링으로 묶어서 수행할 수 있다.

리액트 17에서는 자동 배치가 되지 않아 두번의 리렌더링이 일어났지만 리액트 18에서는 자동 배치 덕분에 리렌더링이 단 한번만 일어남을 볼 수 있다.

그리고 Promise를 사용해 고의로 실행을 지연시키는 sleep 함수를 호출하지 않으면 버전과 상관없이 동일하게 한 번만 렌더링된다는 것이다. 리액트 17 이하의 과거 버전의 경우 이벤트 핸들러 내부에서는 이러한 자동 배치 작업이 이뤄지고 있었지만 Promise, setTimeout 같은 비동기 이벤트에서는 자동 배치가 이뤄지고 있지 않았다. 즉, 동기와 비동기 배치 작업에 일관성이 없었다.

이를 보완하기 위해서 리액트 18부터는 루트 컴포넌트를 createRoot를 사용해서 만들면 모든 업데이트가 배치 작업으로 최적화할 수 있게 됐다.

```jsx
import ReactDOM from 'react-dom/client'
import App from './App'

const rootElement = document.getElementById('root')
const root = ReactDOM.createRoot(rootElement)

root.render(
	<React.StritcMode>
		<App />
	</React.StrictMode>
)
```

이 코드에서는 루트 요소를 `document.getElementById('root')`로 가져온 다음 `ReactDOM.createRoot`를 활용해 렌더링하도록 설정해뒀다. 이렇게 하면 자동 배치가 활성화되어 리액트가 동기, 비동기, 이벤트 핸들러 등에 관계 없이 렌더링을 배치로 수행하게 된다.

만약 이런 자동 배치를 리액트 18에서 하고 싶지 않다면 flushSync를 사용하면 된다. (react-dom에서 제공한다.)

### 10.2.5 더욱 엄격해진 엄격 모드

#### 리액트의 엄격 모드

리액트의 엄격 모드는 리액트에서 제공하는 컴포넌트 중 하나로, 리액트 애플리케이션에서 발생할 수도 있는 잠재적인 버그를 찾는데 도움이 되는 컴포넌트다.  
이 엄격 모드에서 수행하는 모드는 모두 개발자 모드에서만 작동하고, 프로덕션 모드에서는 작동하지 않는다.

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));

root.render(
	<StrictMode>
		<App />
	</StrictMode>
);
```

**엄격 모드의 작업 내용**

- 더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고
- 문자열 ref 사용 금지
- findDOMNode에 대한 경고 출력
- 구 Context API 사용시 발생하는 경고
- 예상치 못한 부작용(side-effects) 검사
  두번씩 실행하여 사이드 이펙트를 검사한다.
  리액트 18에서는 두 번씩 기록하되, 두번째 기록, 즉 엄격 모드에 의해 실행된 console.log는 회색으로 표시하게끔 바꿨다. 엄격 모드에서도 한번씩 기록되게 하고 싶다면 리액트 개발자 도구를 설치한 뒤 리액트 개발자 모드에서 설정 -> debugging -> hide logs during second render in Strict Mode를 체크하면 된다.

#### 리액트 18에서 추가된 엄격 모드

향후 리액트에서는 컴포넌트가 마운트 해제된 상태에서도 컴포넌트 내부의 상태값을 유지할 수 있는 기능을 제공할 예정이라 밝혔다. 예를 들어, 사용자가 뒤로가기를 했다가 다시 현재 화면으로 돌아왔을 때, 리액트가 즉시 이전의 상태를 그대로 유지해 표시할 준비를 하는 기능을 말이다.
이러한 기능을 향후에 지원하기 위해 엄격 모드의 개발자 모드에 새로운 기능을 도입했다.

컴포넌트가 최초에 마운트될 때 자동으로 모든 컴포넌트를 마운트 해제하고 두번째 마운트에서 이전 상태를 복원한다.
즉, 이후에 있을 변경을 위해 StrictMode에서 고의로 useEffect를 두번 작동시키는 내용을 추가한 것이다.

미래에 있을 리액트 업데이트에 효과적으로 대비하려면 이 두번의 useEffect 호출에도 자유로운 컴포넌트를 작성하는 것이 좋다. 이를 위해서 useEffect를 사용할 때 반드시 적절한 cleanup 함수를 배치해서 반복 실행될 수 있는 useEffect로부터 최대한 자유로운 컴포넌트를 만드는 것이 좋다.

### 10.2.8 그 밖에 알아두면 좋은 변경사항

- 컴포넌트에서 undefined를 반환해도 에러가 발생하지 않는다. undefined 반환은 null 반환과 동일하게 처리된다.
- 마찬가지로 `<Suspense fallback={undefined}>`도 null과 동일하게 처리된다.
- renderToNodeStream 지원이 중단됐다. 대신 renderToPipeableStream을 사용하는 것이 권장된다.
