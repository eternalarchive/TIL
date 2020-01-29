2020-01-29 (수)

## 타입스크립트 1

### 개요

- 자바스크립트는 동적 타입 언어 <-> 정적 타입 언어
  - 자바스크립트 변수에는 타입이 없고, 값에 타입이 있음

```javascript
var x = 1;
```

- =는 동치의 개념, 타입 추론으로 = 1에 의해 x의 타입이 추론되어 짐
  - 재할당이 가능한 경우 재할당하면 타입 추론을 다시 하게 됨

```c
char a; // 1바이트 문자
int x; // 4바이트 정수
float g; // 8바이트 실수
```

- 타입을 정해주는 이유는? 메모리에 값을 집어넣기 위해 얼마만큼의 공간이 필요한가를 알기 위함 -> 메모리 절약
  - 정수타입은 2의 32승 만큼 표현이 가능한데 음수 양수가 있으므로 절반씩
- **자바스크립트는 몇바이트**? 무조건 8바이트 부동소수점
  - 자바스크립트의 숫자타입은 정수가 없음
- 자바스크립트에서는 문자열을 원시값이라고 배웠는데, 실제로 문자열은 원시값일 수가 없음
  - 문자열이란? 실질적으로는 한 개 이상의 문자로 이루어진 배열
  - 한글자 타입 char가 1바이트인 이유는? 아스키를 표현할 것으로 생각
  - 모던에서는 1바이트로 한계가 있으므로 -> 유니코드 사용(2바이트)
  - 이모지는 2~4바이트

- 현대 자바스크립트는 애플리케이션 개발 언어가 되었음
  - 자바스크립트는 암묵적 처리가 많아 대규모 작업시 오히려 이해하기가 어려워짐

### 타입은 왜 존재하는가 ?

- 값을 할당할 때 몇 바이트를 확보해야 하는지 알기 위해 -> 메모리 공간의 절약
  - 가져올 때도(참조할 때도) 바이트 수를 알고 통째로 가져오도록(깨지지 않게)
  - 모든 값은 내부적으로 전부 2진수: 참조할 때 해석을 해야하는데 문자열인지, 숫자인지, 이미지인지 알기 위해 타입을 알아야 함
    - 타입 없이는 값을 저장할 수 없음
  - 자바스크립트 초기에는 간단한 용도로 사용했지만 대규모 프로젝트에 사용되며 타입의 필요성이 생김
- 타입이 없는 경우 값의 추적이 어려워짐(예측 떨어짐 -> 안정성 저하)
- 에디터에서도 타입이 있는 경우는 사용하기 편리하도록 알려주고, 린트 작동도 더 잘됨
  - 예를들어 function foo (arr) {}에서 arr이 배열일 때 arr.을 할 경우 배열함수가 나오면 좋겠지만, 에디터가 arr타입을 모르므로 나오지 않음
- **에러도 컴파일(자바스크립트로 변환할 때를 컴파일이라고 치자) 단계에서 남** -> 런타임에서 에러가 나지 않는다!

### 자바스크립트의 객체 지향과 타언어의 객체지향

- 자바스크립트는 프로토타입 베이스, 타언어는 클래스기반
- 클래스기반 언어를 아는 경우 타입스크립트에 금방 적응할 수 있음

<br />

### Introduction

- 자바스크립트 언어의 특징
  - Prototype-based Object Oriented Language
  - Scope와 this
  - 동적 타입(dynamic typed) 언어 혹은 느슨한 타입(loosely typed) 언어
- 코드가 복잡해질 수 있고 디버그와 테스트 공수가 증가하는 등의 문제를 일으킬 수 있어 특히 규모가 큰 프로젝트에서는 주의
- **타입스크립트란**? 자바스크립트의 Superset(상위확장)
  - TypeScript 또한 자바스크립트 대체 언어의 하나로써 **자바스크립트(ES5)의 Superset(상위확장)**

<hr />

**막간, 옵셔널 체이닝이란?**

```javascript
const o = {};
o.user // undefined
o.user.name // undefined에 . 불가능
o && o.user && o.user.name // 단축평가를 활용했었음
```

- 단축평가를 활용해서 값이 있는지 확인해가며 참조했지만 옵셔널 체이닝은 다음과 같이 사용할 수 있음 `o?.user?.name`
- 정식 문법으로 아직 도입되지는 않았으나 높은 확률로 도입될 것
- 모던 브라우저에서 지원중

- **웹팩이란?**
  - 모듈 번들러, 순서에 의존하므로 파일 하나로 만들어주는 것
  - 번들하면서 모듈화를 지원(ES6모듈화가 제대로 동작하지 않음)

<hr />

- 타입 스크립트를 사용하면 웹팩을 사용하지 않아도 됨
  - 현실적으로는 여러가지 이유에서 웹팩을 사용함
- 타입 스크립트 확장자는 .ts인데, 이 파일 안에 자바스크립트 언어를 써도 무방함(의미는 없음) - ES5의 superset이므로
  - **단! 클래스는 제외**함(자바스크립트의 클래스가 다른 일반적인 클래스 언어와 문법이 다르기 때문)

```typescript
class Person {
  name: string; // 이 줄을 추가해줘야 에러가 발생하지 않음
  constructor(name: string) {
    this.name = name;
  }
}
```

<br />

### 개발 환경 구축, 실습

- 설치 `npm i -g typescript`
- 버전 확인 `tsc -v`
- index.ts에 작성하고 `tsc index`를 입력하면 index.js파일 생성(ES5문법으로)
- type추가 후 tsc index를 입력해도 index.js파일에는 type이 존재하지 않음 타입은 컴파일 때까지만 유효함

```typescript
const x: number = '';
// index.ts:1:7 - error TS2322: Type '""' is not assignable to type 'number'.
```

- tsc index를 입력하면 에러발생
- **매번 tsc index 입력하기 귀찮음**
  - `tsc --init`를 입력하면 tsconfig.json이 만들어짐
  - target: 컴파일을 할 때 버전 몇으로 할 것이냐
  - 이제는 그냥 tsc만 입력해도 컴파일이 됨(모든 ts파일을 컴파일)
  - tsconfig파일이 존재하는데, tsc 파일명을 쓰면 config파일을 무시함(디폴트 값으로 진행됨)
- tsc입력조차 귀찮아! `tsc -w`를 입력해주면 파일 변경될 때마다 알아서 컴파일

**export defalut에서 default의 뜻**

- 하나만 export
- import 할 때 이름을 그대로 받아야 함(as 사용시 변경 가능)

<hr />

```typescript
// Person.ts
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  sayHi() {
    console.log(`Hi! ${this.name}.`);
  }
}

export default Person;

const person = new Person('Lee');

// index.ts
import Person from './Person';

const person = new Person('Lee');

person.sayHi();
```

- `node index` 실행시 Hi! Lee. 출력 확인

### commonjs란?

- ES6에서 모듈(import, export)문법이 도입됐는데, 스크립트 태그에 타입 모듈을 지정해주면 자바스크립트 파일을 불러올 때 모듈로 동작함
  - 문제는 아직까지 번들링을 해야할 필요가 있고 사용하기에 미흡한 부분들이 있음
- 이를 해결하기 위해 모듈을 지원하는 라이브러리가 두 진영이 있음
  - commonjs -> node.js에서 주로 사용(require로 가져오고 export하는 문법이 따로 있음) (모듈을 동기로 가져옴)
  - amd (모듈을 비동기로 가져옴)
- tsconfig.json에 module: commonjs로 설정되어 있음

<br />

## 정적 타이핑

### 타입 선언과 정적 타이핑

- undefined는 타입이자 유일한 값, null도 마찬가지
- 배열은?

```typescript
// 배열 안의 요소들이 숫자
const arr: number[];
```

- 자바스크립트에서 배열을 getOwnPropertyDescriptor로 찍어보면 내부 어트리뷰트를 볼 수 있는데 객체임, 프로퍼티 이름이 숫자일 뿐(유사배열)
  - 원론적인 의미의 배열과 자바스크립트의 배열은 다름
  - 원론적으로 요소의 타입이 통일되어 있어야 함(메모리 공간이 연속되어 있도록-접근이 빠름)
  - 자바스크립트에서는 단지 프로퍼티의 value이므로 [1, 'x', true]와 같이 모든 값이 올 수 있음(엄밀히 말하면 배열이 없음)
  - 꼭 다른 형태가 필요할 경우 any[]

- tuple: 만드는 배열의 요소 개수를 정하고, 요소의 타입을 fix하는것
- enum: todolist에서 nav all, active, completed나 예를들어 조이스틱 위=1 아래=2 좌=3 우=4가 있다고 할 때 숫자보다는 이름이 의미가 있으므로 하나로 묶어놓는 것
  - `const D={UP, DOWN, RIGNT, LEFT}`
- never: 함수에 리턴 값이 없을 때(중간에 에러가 발생한다던가) - 생략해도 무관
- 변수는 여러번 재할당이 가능, 상수는 재할당 불가능
  - 상수를 사용하는 이유는? 고정된 값올 돌려쓰기 위함

- **타입 지정해줄 때 첫글자가 대문자인 경우?**
  - 문자열 타입이다가 아니고 스트링 객체 타입이라는 뜻이 됨
  - 타입으로 써야할 경우는 다 소문자로 써줄 것

```typescript
// String: String 생성자 함수로 생성된 String 래퍼 객체 타입
let objectStr: String;
objectStr = 'hello'; // OK
objectStr = new String('hello'); // OK
```

- String생성자 함수로 만들어진 인스턴스는 전부 String
- Date생성자 함수로 날짜를 생성하면 다 Date타입(이자 Object 타입)
- instanceof에 의한 것

<hr />

### 과거 복습 - 이걸 알아야 타입 반환 제대로

- document.querySelectAll - nodelist 반환
- document.getElementsByname - htmlcollection 객체 반환
- documentquerySelect('input') - HTMLInputElement 객체 반환 -> 상위 HTML Element -> Element -> Node

```typescript
// HTMLElement 타입
const elem: HTMLElement = document.getElementById('myId');
```

- 정확히 태그명이 뭔지 모르는 상황에(어쨌든 요소는 요소이므로)
- 이렇게 쓰면 elem. 했을 때 HTMLElement가 가지고 있는 것들을 사용할 수 있음

<hr />

- 함수에는 다음과 같이 사용

```typescript
function add(x: number, y: number): number {
  return x + y;
}

console.log(add(10, 10)); // 20
console.log(add('10', '10'));
// error TS2345: Argument of type '"10"' is not assignable to parameter of type 'number'.
```

- 매개변수 괄호 뒤 : number는 return 타입
- 정적 타이핑의 장점: **코드 가독성, 예측성, 안정성의 향상**

<br />

### 타입 추론

- 타입 추론(Type Inference) : 타입 선언을 생략하면 값이 할당되는 과정에서 동적으로 타입이 결정

- TypeScript는 정적 타입 언어이므로 타입 추론으로 타입이 결정된 이후, 다른 타입의 값을 할당하면 에러가 발생

```typescript
let foo = 123; // foo는 number 타입
foo = 'hi';    // error: Type '"hi"' is not assignable to type 'number'.
```

- 타입 선언을 생략하고 값도 할당하지 않아서 타입을 추론할 수 없으면 `any` 타입
  -  `any` 타입의 변수는 자바스크립트의 var 키워드로 선언된 변수처럼 어떤 타입의 값도 재할당이 가능
  - 이는 TypeScript를 사용하는 장점을 없애기 때문에 사용하지 않는게 좋음

```typescript
let foo; // let foo: any와 동치

foo = 'Hello';
console.log(typeof foo); // string

foo = true;
console.log(typeof foo); // boolean
```

