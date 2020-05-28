# 타입스트립트로 리덕스 미들웨어 사용하기(redux-thunk)

- index.ts에 import Thunk를 한 뒤 applyMiddleware를 불러옴

```tsx
const store = createStore(rootReducer);

const store = createStore(rootReducer, applyMiddleware(Thunk));
```

- 깃헙 api를 이용해보자

```
GET https://api.github.com/users/:usename
```

- https://quicktype.io/
  - json파일을 복사해서 붙여넣으면 바로 타입을 알 수 있음

- src/api 디렉토리를 만든 후 github.ts파일을 만들고 axios로 불러온 값의 반환 타입을 지정해줌

```tsx
import axios from 'axios';

export async function getUserProfile(usename: string) {
  const response = await axios.get<GithubProfile>(`https://api.github.com/users/:usename`);
  return response.data;
}

export interface GithubProfile {
...
}
```

- modules에 github디렉토리 생성 후 actions.ts에 액션들을 만듦
  - error의 반환값은 AxiosError

```tsx
import { createAction } from "typesafe-actions";
import { GithubProfile } from "../../api/github";
import { AxiosError } from "axios";

export const GET_USER_PROFILE = 'github/GET_USER_PROFILE';
export const GET_USER_PROFILE_SUCCESS = 'github/GET_USER_PROFILE_SUCCESS';
export const GET_USER_PROFILE_ERROR = 'github/GET_USER_PROFILE_ERROR';

export const getUserProfile = createAction(GET_USER_PROFILE)();
export const getUserProfileSuccess = createAction(GET_USER_PROFILE_SUCCESS)<GithubProfile>();
export const getUserProfileError = createAction(GET_USER_PROFILE_ERROR)<AxiosError>();
```

- typesafe-actions에는 요청 비동기에 관한 액션을 선언할 때 더 쉽게 만들 수 있도록 해주는 함수 존재
  - createAsyncAction
  - 불러올 때, 성공했을 때, 실패했을 때
    - getUserProfileAsync. 까지 쳤을 때 차례대로 failure, request, success
  - GET_USER_PROFILE은 반환값이 없으므로 undefined

```tsx
export const getUserProfileAsync = createAsyncAction(
  GET_USER_PROFILE,
  GET_USER_PROFILE_SUCCESS,
  GET_USER_PROFILE_ERROR
)<undefined, GithubProfile, AxiosError>();
```

- types.ts파일 생성

```tsx
import * as actions from './actions';
import { ActionType } from 'typesafe-actions';
import { GithubProfile } from '../../api/github';

export type GithubAction = ActionType<typeof actions>;
export type GithubState = {
  userProfile: {
    loading: boolean;
    data: GithubProfile | null;
    error: GithubProfile | null;
  };
}
```

- thunk함수 생성할 것. thunk.ts파일 생성

```tsx
export function getUserProfileThunk(username: string) {
  return async dispatch => {};
}
```

- 여기서 dispatch의 타입을 지정할 때

  - 간편한 방법(추천)

  ```tsx
  import { Dispatch } from "redux";
  
  export function getUserProfileThunk(username: string) {
    return async (dispatch: Dispatch) => {};
  }
  ```

  - 제대로 하는 방법
  - ThunkAction으로 한 뒤 리턴타입을 정해주어야 함
    - 첫번째(반환값): 특별히 반환값이 없으므로 void
    - 두번째(상태): 루트상태
    - 세번째(extraType): 없으므로 null
    - 네번째(액션들의 타입): GithubAction
  - 이 때 state를 사용하면 알아서 값이 잘 들어가 있음

  ```tsx
   import { ThunkAction } from 'redux-thunk';
   import { RootState } from '..';
   import { GithubAction } from './types';
  
   export function getUserProfileThunk(username: string): ThunkAction<void, RootState, null, GithubAction> {
     return async (dispatch, state) => {};
   }
  ```

- 간편한 방법 사용시 state

```tsx
import { Dispatch } from "redux";
import { RootState } from "..";

export function getUserProfileThunk(username: string) {
  return async (dispatch: Dispatch, getState: () => RootState) => {
    const state = getState();
  };
}
```

- 최종 thunks.ts

```tsx
import { Dispatch } from "redux";
import { RootState } from "..";
import { getUserProfileAsync } from "./actions";
import { getUserProfile } from "../../api/github";

export function getUserProfileThunk(username: string) {
  return async (dispatch: Dispatch, getState: () => RootState) => {
    const { request, success, failure } = getUserProfileAsync;
    dispatch(request()); // 요청 시작 알림
    try {
      const userProfile = await getUserProfile(username);
      dispatch(success(userProfile));
    } catch(e) {
      dispatch(failure(e));
    }
  };
}
```

- 리듀서를 만들자

```tsx
import { createReducer } from 'typesafe-actions';
import { GithubState, GithubAction } from './types';
import { GET_USER_PROFILE, GET_USER_PROFILE_SUCCESS, GET_USER_PROFILE_ERROR } from './actions';

const initialState: GithubState = {
  userProfile: {
    loading: false,
    error: null,
    data: null
  }
};

const github = createReducer<GithubState, GithubAction>(initialState, {
  [GET_USER_PROFILE]: (state) => ({
    ...state,
    userProfile: {
      loading: true,
      error: null,
      data: null,
    }
  }),
  [GET_USER_PROFILE_SUCCESS]: (state, action) => ({
    ...state,
    loading: false,
    error: null,
    data: action.payload,
  }),
  [GET_USER_PROFILE_ERROR]: (state, action) => ({
    ...state,
    loading: false,
    error: action.payload,
    data: null,
  }),
});

export default github;
```

- index.ts

```tsx
export { default } from './reducer';
export * from './actions';
export * from './types';
export * from './thunks';
```

- 루트 리듀서에 github추가

정리

- api를 선언할때는 결과물 타입을 지정해주고 axios로 불러올때 제너릭으로 지정
- thunk를 만들때는 ThunkAction대신 Dispatch를 이용하여 편리하게 구현가능
- action생성함수 만들 때 createAsyncAction을 이용해 편리하게 생성가능