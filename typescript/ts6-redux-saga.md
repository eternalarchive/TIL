# 타입스크립트로 redux-saga 사용하기

- 기존 action에 GET_USER_PROFILE은 페이로드가 없어 undefined를 썼지만, redux-saga는 필요

```tsx
export const getUserProfileAsync = createAsyncAction(
  GET_USER_PROFILE,
  GET_USER_PROFILE_SUCCESS,
  GET_USER_PROFILE_ERROR
)<string, GithubProfile, AxiosError>();
```

- sagas.ts 생성

  - 제너레이터 함수 생성 후 action을 파라미터로 받아오는데 타입을 ReturnType<typeof getUserProfileAsync.request>로 지정
  - 함수 내부에 action.을 칠 경우 payload가 자동완성 됨을 알 수 있음
  - call은 특정 함수 호출, put은 특정 액션 디스패치, takeEvery는 특정 액션을 모니터링하다가 원하는 액션이 들어오면 사전에 정한 사가 함수를 호출해주는 작업
  - 여기서 userProfile은 타입이 any로 나오므로 타입을 지정해주도록 하자(GithubProfile) - 추후 개선될 것으로 예상

  ```javascript
  const userProfile = yield call(getUserProfile, action.payload);
  ```

  - 깃헙사가 내부에서 takeEvery를 사용해서 GET_USER_PROFILE 액션이 발생하면 getUserProfileSaga를 실행하도록 설정

```tsx
// sagas.ts

import { getUserProfileAsync, GET_USER_PROFILE } from "./actions";
import { call, put, takeEvery } from 'redux-saga/effects'
import { getUserProfile, GithubProfile } from "../../api/github";

function* getUserProfileSaga(action: ReturnType<typeof getUserProfileAsync.request>) {
  try {
    const userProfile: GithubProfile = yield call(getUserProfile, action.payload);
    yield put(getUserProfileAsync.success(userProfile));
  } catch (e) {
    yield put(getUserProfileAsync.failure(e));
  }
}

export function* githubSaga() {
  yield takeEvery(GET_USER_PROFILE, getUserProfileSaga)
}
```

- modules/github/index.ts에 `export * from './sagas';`추가
- modules/index.ts에 rootSaga를 만들자
  - rootSaga함수를 만들어서 `yield all([githubSaga()]);`를 추가해준다.

```tsx
import github, { githubSaga } from './github';
import { all } from 'redux-saga/effects';

...

export function* rootSaga() {
  yield all([
    githubSaga()
  ]);
```

- 제일 바깥의 index.tsx

  - 상단에서 createSagaMiddleware를 불러온 뒤 사가 미들웨어 생성

  ```tsx
  import React from 'react';
  import ReactDOM from 'react-dom';
  import './index.css';
  import App from './App';
  import * as serviceWorker from './serviceWorker';
  import { Provider } from 'react-redux';
  import { createStore, applyMiddleware } from 'redux';
  import rootReducer, { rootSaga } from './modules';
  import createSagaMiddleware from 'redux-saga';
  
  const sagaMiddleware = createSagaMiddleware();
  
  const store = createStore(rootReducer, applyMiddleware(sagaMiddleware));
  
  sagaMiddleware.run(rootSaga);
  
  ReactDOM.render(
    <Provider store={store}>
      <React.StrictMode>
        <App />
      </React.StrictMode>
    </Provider>,
    document.getElementById('root')
  );
  
  // If you want your app to work offline and load faster, you can change
  // unregister() to register() below. Note this comes with some pitfalls.
  // Learn more about service workers: https://bit.ly/CRA-PWA
  serviceWorker.unregister();
  
  ```

- containers에 GithubPRofileLoader에서 Thunk로 디스패치한 것을 지우고 아래와 같이 사용함

```tsx
  const onSubmitUsername = (username: string) => {
dispatch(getUserProfileAsync.request(username));
  };
```

