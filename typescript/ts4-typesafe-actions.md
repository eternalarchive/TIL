# 타입스크립트 typesafe-actions 사용하기

## Counter 예제

- 액션 생성함수와 리듀서를 깔끔하게 작성 가능
- typesafe-actions 사용시 as const 쓰지 않아도 됨

```tsx
import { createAction, ActionType, createReducer } from 'typesafe-actions';

// 변경 전
const INCREASE = 'counter/INCREASE' as const;
const DECREASE = 'counter/DECREASE' as const;
const INCREASE_BY = 'counter/INCREASE_BY' as const;

export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseBy = (diff: number) => ({
  type: INCREASE_BY,
  payload: diff,
});

// 변경 후
const INCREASE = 'counter/INCREASE';
const DECREASE = 'counter/DECREASE';
const INCREASE_BY = 'counter/INCREASE_BY';

export const increase = createAction(INCREASE)(); // payload 없기 때문에 제너릭 생략 가능
export const decrease = createAction(DECREASE)();
export const increaseBy = createAction(INCREASE_BY)<number>();
```

```tsx
// 변경 전
type CounterAction =
  | ReturnType<typeof increase>
  | ReturnType<typeof decrease>
  | ReturnType<typeof increaseBy>
    
// 변경 후
const actions = { increase, decrease, increaseBy };
type CounterAction = ActionType<typeof actions>;
```

```tsx
// 변경 전
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
// 변경 후
const counter2 = createReducer<CounterState, CounterAction>(initialState, {
  [INCREASE]: (state) => ({ count: state.count + 1 }),
  [DECREASE]: (state) => ({ count: state.count - 1 }),
  [INCREASE_BY]: (state, action) => ({ count: state.count + action.payload })
})
```

- 메서드 체이닝 방식으로 구현 가능

```tsx
const counter = createReducer<CounterState, CounterAction>(initialState)
  .handleAction(increase, state => ({ count: state.count + 1 }))
  .handleAction(decrease, state => ({ count: state.count - 1 }))
  .handleAction(increaseBy, (state, action) => ({ count: state.count + action.payload }))
```

- 액션 타입 말고 액션 타입 생성 함수를 넣어도 작동함
  - 리듀서에서 CounterAction 생략 가능하므로 지워주고 액션 생성 역시 한번만 이루어지므로 별도 상수 선언 필요 없이 바로 넣어서 사용하도록 함

```tsx
import { createAction, ActionType, createReducer } from 'typesafe-actions';

export const increase = createAction('counter/INCREASE')();
export const decrease = createAction('counter/DECREASE')();
export const increaseBy = createAction('counter/INCREASE_BY')<number>();

const actions = { increase, decrease, increaseBy };

type CounterState = {
  count: number;
}

const initialState: CounterState = {
  count: 0,
};

// 메서드 체이닝 방식
const counter = createReducer<CounterState, CounterAction>(initialState)
  .handleAction(increase, state => ({ count: state.count + 1 }))
  .handleAction(decrease, state => ({ count: state.count - 1 }))
  .handleAction(increaseBy, (state, action) => ({ count: state.count + action.payload }))

export default counter;
```

## Todos 예제

- addTodo 같은 경우에는 받아오는 값이 아닌 nextId가 필요하므로 createAction 사용시 더 복잡할 수 있음
  - 이 때 as const를 사용해야 하는 것 잊지 말기

```tsx
export const addTodo = createAction(ADD_TODO, action => (text: string) => action({
  id: nextId++,
  text
}));

// 사용하지 않는 경우
export const addTodo = (text: string) => ({
  type: ADD_TODO,
  payload: {
    id: nextId++,
    text,
  }
});
```

- 완성코드

```tsx

import { createAction, createReducer, ActionType} from 'typesafe-actions';

let nextId = 1;

const ADD_TODO = 'todos/ADD_TODO' as const;
const TOGGLE_TODO = 'todos/TOGGLE_TODO';
const REMOVE_TODO = 'todos/REMOVE_TODO';

export const addTodo = (text: string) => ({
  type: ADD_TODO,
  payload: {
    id: nextId++,
    text,
  }
});
export const toggleTodo = createAction('todos/TOGGLE_TODO')<number>();
export const removeTodo = createAction('todos/REMOVE_TODO')<number>();

const actions = { addTodo, toggleTodo, removeTodo };

type TodosAction = ActionType<typeof actions>;

export type Todo = {
  id: number;
  text: string;
  done: boolean;
}

type TodosState = Todo[];

const initialState: TodosState = [];

const todos = createReducer<TodosState, TodosAction>(initialState, {
  [ADD_TODO]: (state, action) => state.concat({
    ...action.payload,
    done: false
  }),
  [TOGGLE_TODO]: (state, action) => state.map(todo => todo.id === action.payload ? {...todo, done: !todo.done} : todo),
  [REMOVE_TODO]: (state, action) => state.filter(todo => todo.id !== action.payload)
})

export default todos;
```

- 변경전 참고

```tsx
const ADD_TODO = 'todos/ADD_TODO' as const;
const TOGGLE_TODO = 'todos/TOGGLE_TODO' as const;
const REMOVE_TODO = 'todos/REMOVE_TODO' as const;

let nextId = 1;

export const addTodo = (text: string) => ({
  type: ADD_TODO,
  payload: {
    id: nextId++,
    text
  },
});

export const toggleTodo = (id: number) => ({
  type: TOGGLE_TODO,
  payload: id,
});

export const removeTodo = (id: number) => ({
  type: REMOVE_TODO,
  payload: id,
});

type TodosAction =
  | ReturnType<typeof addTodo>
  | ReturnType<typeof toggleTodo>
  | ReturnType<typeof removeTodo>

export type Todo = {
  id: number;
  text: string;
  done: boolean;
}

type TodosState = Todo[];

const initialState: TodosState = [];

function todos(state: TodosState = initialState, action: TodosAction): TodosState {
  switch (action.type) {
    case ADD_TODO:
      return state.concat({
        id: action.payload.id,
        text: action.payload.text,
        done: false,
      });
    case TOGGLE_TODO:
      return state.map(todo =>
          todo.id === action.payload ? { ...todo, done: !todo.done } : todo
        );
    case REMOVE_TODO:
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
}

export default todos;
```

