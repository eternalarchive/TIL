#타입스크립트로 투두리스트 만들기(with redux)

- todos.ts

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
```

- 다음으로 액션타입, Todo, TodosState의 타입을 써줌

```tsx
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
```

- todos 리듀서를 작성함

```tsx
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

- index.ts rootReducer에 todos를 추가해줌
- 각각 TodoInsert, TodoItem, TodoList 컴포넌트를 생성

```tsx
// TodoInsert.tsx
import React, { useState, FormEvent } from 'react';

type TodoInsertProps = {
  onInsert: (text: string) => void;
}

function TodoInsert({ onInsert }: TodoInsertProps) {
  const [value, setValue] = useState('');
  const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };
  const onSubmit = (e: FormEvent) => {
    e.preventDefault();
    onInsert(value);
    setValue('');
  };

  return (
    <form onSubmit={onSubmit}>
      <input
        placeholder="할 일을 입력하세요"
        value={value}
        onChange={onChange}
      />
      <button type="submit">등록</button>
    </form>
  );
};

export default TodoInsert;
```

```tsx
// todoItem.tsx
import React, { CSSProperties } from 'react';
import { Todo } from '../../modules/todos';

type TodoItemProps = {
  todo: Todo;
  onToggle: (id: number) => void;
  onRemove: (id: number) => void;
}

function TodoItem({ todo, onToggle, onRemove }: TodoItemProps) {
  const handleToggle = () => onToggle(todo.id);
  const handleRemove = () => onRemove(todo.id);

  const textStyle: CSSProperties = {
    textDecoration: todo.done ? 'line-through' : 'none'
  };

  const removeStyle: CSSProperties = {
    color: 'red',
    marginLeft: 8,
  };

  return (
    <li>
      <span onClick={handleToggle} style={textStyle}>{todo.text}</span>
      <span onClick={handleRemove} style={removeStyle}>(X)</span>
    </li>
  );
};

export default TodoItem;
```

```tsx
// TodoList.tsx

import React from 'react';
import { Todo } from '../../modules/todos';
import TodoItem from './TodoItem';

type TodoListProps = {
  todos: Todo[];
  onToggle: (id: number) => void;
  onRemove: (id: number) => void;
}

function TodoList({ todos, onToggle, onRemove }: TodoListProps) {
  if (todos.length === 0) return <p>등록된 항목이 없습니다.</p>
  return (
    <ul>
      {
        todos.map(todo => (
        <TodoItem
          todo={todo}
          onToggle={onToggle}
          onRemove={onRemove}
          key={todo.id}
        />))
      }
    </ul>
  );
};

export default TodoList;
```

- 컨테이너를 작성함

```tsx
// TodoApp.tsx

import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { RootState } from '../modules';
import { addTodo, toggleTodo, removeTodo } from '../modules/todos';
import TodoInsert from '../components/todos/TodoInsert';
import TodoList from '../components/todos/TodoList';

function TodoApp() {
  const todos = useSelector((state: RootState) => state.todos);
  const dispatch = useDispatch();

  const onInsert = (text: string) => {
    dispatch(addTodo(text));
  };
  const onToggle = (id: number) => {
    dispatch(toggleTodo(id));
  };
  const onRemove = (id: number) => {
    dispatch(removeTodo(id));
  };

  return (
    <>
      <TodoInsert onInsert={onInsert}/>
      <TodoList todos={todos} onToggle={onToggle} onRemove={onRemove}/>
    </>
  );
};

export default TodoApp;
```

