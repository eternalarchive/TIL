# 라액트에서 타입스크립트 사용하기

## 컴포넌트 함수 작성시

- 화살표 함수대신 function 키워드를 사용하는 편
- 화살표 함수 사용시 React.FC 키워드와 함께 사용하는데!
  - 단점: defaultProps 설정이 되지 않음
  - 장점이자 단점? children을 자동으로 받아옴
  - 예를들어 optional?: string으로 설정시 undefined | string으로 설정됨
- e.preventDefualt()는 새로고침 방지



## useReducer에서 사용하기

```tsx
import React, { useReducer } from 'react';

type Color = 'red' | 'orange' | 'yellow';

type State = {
  count: number;
  text: string;
  color: Color;
  isGood: boolean;
};

type Action =
  | { type: 'SET_COUNT'; count: number }
  | { type: 'SET_TEXT'; text: string }
  | { type: 'SET_COLOR'; color: Color }
  | { type: 'TOGGLE_GOOD' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_COUNT':
      return {
        ...state,
        count: action.count
      };
    case 'SET_TEXT':
      return {
        ...state,
        text: action.text
      };
    case 'SET_COLOR':
      return {
        ...state,
        color: action.color
      };
    case 'TOGGLE_GOOD':
      return {
        ...state,
        isGood: !state.isGood
      };
    default:
      throw new Error('Error');
  }
}

function ReducerSample() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    text: 'hello',
    color: 'red',
    isGood: true,
  });

  const setCount = () => dispatch({ type: 'SET_COUNT', count: 5 });
  const setText = () => dispatch({ type: 'SET_TEXT', text: 'bye' });
  const setColor = () => dispatch({ type: 'SET_COLOR', color: 'orange'});
  const toggleGood = () => dispatch({ type: 'TOGGLE_GOOD' });

  return (
    <div>
      <p>
        <code>count: </code> {state.count}
      </p>
      <p>
        <code>text: </code> {state.text}
      </p>
      <p>
        <code>color: </code> {state.color}
      </p>
      <p>
        <code>isGood: </code> {state.isGood ? 'true' : 'false'}
      </p>
      <div>
        <button onClick={setCount}>SET COUNT</button>
        <button onClick={setText}>SET_TEXT</button>
        <button onClick={setColor}>SET_COLOR</button>
        <button onClick={toggleGood}>TOGGLE_GOOD</button>
      </div>
    </div>
  );
};

export default ReducerSample;
```



## Context API에서 사용하기

```tsx
import React, { createContext } from 'react';

type FooValue = {
  foo: number;
};

const FooContext = createContext<FooValue | null>(null); 
```

- createContext에는 기본값을 넣어야 함
  - null로 넣고 싶은 경우 | null을 써줄것
- useReducer했을 때 쓴 type엘리어스들과 리듀서를 복사해옴

```tsx
// 상태를 다루는 context와 dispatch를 관리하는 context를 따로 만들 것
// 먼저 상태만 다루는 컨텍스트
const SampleStateContext = createContext<State | null>(null);

// 디스패치함수의 타입은 리액트가 가지고 있으므로 Dispatch로 써주고 제너릭을 넣어줘야함(Action)
// const SampleDispatchContext = createContext<Dispatch<Action> | null>(null);
// 위와 같은 방식이 보기 불편하다면 이렇게 분리 해서 사용할 수 있음
type SampleDispatch = Dispatch<Action>;
const SampleDispatchContext = createContext<SampleDispatch | null>(null);

// children을 사용하기 위해 다음과 같이 써준다
type SampleProviderProps = {
  children: React.ReactNode;
}
export function SampleProvider({ children }: SampleProviderProps) {
    const [state, dispatch] = useReducer(reducer, {
    count: 0,
    text: 'hello',
    color: 'red',
    isGood: true,
  });
}
```

- SampleProvider 내부에는 기존에 썼던 useReducer 내용을 가져다 넣었음

```tsx
  return (
    <SampleStateContext.Provider value={state}>
      <SampleDispatchContext.Provider value={dispatch}>
        {children}
      </SampleDispatchContext.Provider>
    </SampleStateContext.Provider>
  )
```

- 컨텍스트를 통해 관리하는 value값들을 다른 컴포넌트에서 사용하려면 useContext를 사용해야 하는데, 그 대신 커스텀 훅을 통해 사용해보겠음

```tsx
export function useSampleState() {
  const state = useContext(SampleStateContext);
  // state가 null도 들어올 수 있게 되어있는데 나중에 null체크하기 불편하므로
  if (!state) throw new Error('Cannot find SampleProvider');
  return state;
}

export function useSampleDispatch() {
  const dispatch = useContext(SampleDispatchContext);
  if (!dispatch) throw new Error('Cannot find SampleProvider');
  return dispatch;
}
```

- 기존 ReducerSample에서 이 값들을 사용할 수 있게 하려면 다음과 같이 사용할 수 있음

```tsx
// ReducerSample.tsx파일
const state = useSampleState();
const dispatch = useSampleDispatch();
```

- 여기까지만 하면 오류가 발생하는데 App.tsx에서 ReducerSample파일을 SampleProvider로 감싸주어야 함

```tsx
import { SampleProvider } from './components/SampleContext';
...
return (
  <SampleProvider>
    <ReducerSample />
  </SampleProvider>
)
```

- 총 코드

```tsx
// ReducerSample.tsx
import React, from 'react';
import { useSampleDispatch, useSampleState } from './SampleContext';

function ReducerSample() {
  const state = useSampleState();
  const dispatch = useSampleDispatch();

  const setCount = () => dispatch({ type: 'SET_COUNT', count: 5 });
  const setText = () => dispatch({ type: 'SET_TEXT', text: 'bye' });
  const setColor = () => dispatch({ type: 'SET_COLOR', color: 'orange'});
  const toggleGood = () => dispatch({ type: 'TOGGLE_GOOD' });

  return (
    <div>
      <p>
        <code>count: </code> {state.count}
      </p>
      <p>
        <code>text: </code> {state.text}
      </p>
      <p>
        <code>color: </code> {state.color}
      </p>
      <p>
        <code>isGood: </code> {state.isGood ? 'true' : 'false'}
      </p>
      <div>
        <button onClick={setCount}>SET COUNT</button>
        <button onClick={setText}>SET_TEXT</button>
        <button onClick={setColor}>SET_COLOR</button>
        <button onClick={toggleGood}>TOGGLE_GOOD</button>
      </div>
    </div>
  );
};

export default ReducerSample;
```

```tsx
// SampleContext.tsx
import React, { createContext, Dispatch, useReducer, useContext } from 'react';

type FooValue = {
  foo: number;
};

const FooContext = createContext<FooValue | null>(null);

type Color = 'red' | 'orange' | 'yellow';

type State = {
  count: number;
  text: string;
  color: Color;
  isGood: boolean;
};

type Action =
  | { type: 'SET_COUNT'; count: number }
  | { type: 'SET_TEXT'; text: string }
  | { type: 'SET_COLOR'; color: Color }
  | { type: 'TOGGLE_GOOD' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_COUNT':
      return {
        ...state,
        count: action.count
      };
    case 'SET_TEXT':
      return {
        ...state,
        text: action.text
      };
    case 'SET_COLOR':
      return {
        ...state,
        color: action.color
      };
    case 'TOGGLE_GOOD':
      return {
        ...state,
        isGood: !state.isGood
      };
    default:
      throw new Error('Error');
  }
}

const SampleStateContext = createContext<State | null>(null);

type SampleDispatch = Dispatch<Action>;
const SampleDispatchContext = createContext<SampleDispatch | null>(null);

type SampleProviderProps = {
  children: React.ReactNode;
}
export function SampleProvider({ children }: SampleProviderProps) {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    text: 'hello',
    color: 'red',
    isGood: true,
  });

  return (
    <SampleStateContext.Provider value={state}>
      <SampleDispatchContext.Provider value={dispatch}>
        {children}
      </SampleDispatchContext.Provider>
    </SampleStateContext.Provider>
  )
}


export function useSampleState() {
  const state = useContext(SampleStateContext);
  if (!state) throw new Error('Cannot find SampleProvider');
  return state;
}

export function useSampleDispatch() {
  const dispatch = useContext(SampleDispatchContext);
  if (!dispatch) throw new Error('Cannot find SampleProvider');
  return dispatch;
}
```

```tsx
// App.tsx
import React from 'react';
import ReducerSample from './components/ReducerSample';
import { SampleProvider } from './components/SampleContext';

function App() {
  return (
    <SampleProvider>
      <ReducerSample />
    </SampleProvider>
  );
}

export default App;

```

