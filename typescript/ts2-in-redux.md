# 리덕스에서 타입스크립트 사용하기

## 리덕스 튜토리얼

- typesearch를 통해 타입 지원이 되는지 확인해볼 수 있음
- src/modules 생성하고 counter.ts생성

```ts
const INCREASE = 'counter/INCREASE';
const DECREASE = 'counter/DECREASE';
const INCREASE_BY = 'counter/INCREASE_BY';

export const increase = () => ({ type: INCREASE });
```

- 이 때 increase에 화살표를 올려보면 액션 생성함수의 결과가 string이 나오는데, 타입스크립트를 유추할 수 없게 되므로 as const를 추가하여 해결하도록 함

```ts
const INCREASE = 'counter/INCREASE' as const;
```

- 액션의 타입을 지정할 때 type: 'counter/INCREASE'와 같이 쓰면 겹치므로 typeof INCREASE라고 사용하기도 하는데 좋은 방법은 아님
  - ReturnType을 통해 반환값을 받아올 수 있음!

```tsx
type CounterAction = 
  | {type: typeof INCREASE} // 이거보단
	| ReturnType<typeof increase> // 이렇게
```

- 아래와 같이 사용할 경우 return값 형태를 잡아내지 못하므로 꼭 return값도 써주도록 하자

```tsx
function counter(state: CounterState = initialState, action: CounterAction) {
  switch (action.type) {
    case INCREASE:
      return { count: 'asdf' }
  }
}

// 리턴값도 빼먹지 말고 넣어주자
function counter(state: CounterState = initialState, action: CounterAction): CounterState {
  switch (action.type) {
    case INCREASE:
      return { count: 'asdf' } // 타입오류! 숫자를 써주자 state.count + 1
  }
}
```

```ts
// counter.ts

const INCREASE = 'counter/INCREASE' as const;
const DECREASE = 'counter/DECREASE' as const;
const INCREASE_BY = 'counter/INCREASE_BY' as const;

export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseBy = (diff: number) => ({
  type: INCREASE_BY,
  payload: diff,
});

type CounterState = {
  count: number;
}

const initialState: CounterState = {
  count: 0,
};

type CounterAction =
  | ReturnType<typeof increase>
  | ReturnType<typeof decrease>
  | ReturnType<typeof increaseBy>

function counter(state: CounterState = initialState, action: CounterAction): CounterState {
  switch (action.type) {
    case INCREASE:
      return { count: state.count + 1 };
    case DECREASE:
      return { count: state.count - 1 };
    case INCREASE_BY:
      return { count: state.count + action.payload };
    default:
      return state;
  }
}

export default counter;
```

- index.ts를 생성해주자
  - combineReducers를 통해 root를 만들어줄 것
  - 결국 함수이므로 ReturnType을 사용하여 리덕스스토어에서 관리하고 있는 상태에 대한 타입을 만들자

```tsx
import { combineReducers } from 'redux';
import counter from './counter';

const rootReducer = combineReducers({
  counter
});

export default rootReducer;
export type RootState = ReturnType<typeof rootReducer>
```

- 외부 index.tsx파일로 가서 다음과 같이 변경

```tsx
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import rootReducer from './modules';

const store = createStore(rootReducer);

ReactDOM.render(
  <Provider store={store}>
    <React.StrictMode>
      <App />
    </React.StrictMode>
  </Provider>,
  document.getElementById('root')
);
```

- 컴포넌트폴더에 Counter.tsx 생성

```tsx
import React from 'react'

type CounterProps = {
  count: number;
  onIncrease: () => void;
  onDecrease: () => void;
  onIncreaseBy: (diff: number) => void;
}

function Counter({
  count,
  onIncrease,
  onDecrease,
  onIncreaseBy
}: CounterProps) {
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
      <button onClick={() => onIncreaseBy(5)}>+5</button>
    </div>
  );
};

export default Counter;
```

- CounterContainer생성

```tsx
import React from 'react';
import Counter from '../components/Counter';
import { useSelector, useDispatch } from 'react-redux';
import { RootState } from '../modules';
import { increase, decrease, increaseBy } from '../modules/counter';

function CounterContainer() {
  const count = useSelector((state: RootState) => state.counter.count);
  const dispatch = useDispatch();

  const onIncrease = () => {
    dispatch(increase());
  };

  const onDecrease = () => {
    dispatch(decrease());
  };

  const onIncreaseBy = (diff: number) => {
    dispatch(increaseBy(diff));
  };

  return (
    <Counter
      count={count}
      onIncrease={onIncrease}
      onDecrease={onDecrease}
      onIncreaseBy={onIncreaseBy}
    />
  );
}

export default CounterContainer;
```

