2020-01-31 (금)

Q. 클래스와 생성자 함수의 차이는?

- 결론: 둘은 유사한 역할을 하지만! 생성자 함수보다 클래스가 더 엄격 -> 클래스를 사용하는게 나음
  - 클래스는 event처리가 어려움 - constructor 내부에서 addEventListener를 붙이거나 onclick붙이는데 이 때 this가 꼬임

- 객체를 만드는 방법: 객체 리터럴, Object.create함수, 생성자함수, 클래스

- 클래스는 문법적 설탕이 아니다!

  - 생성자 함수보다 클래스가 더 엄격하고 근본적으로 Method가 다름

  - 메소드 생성 방법

    - 프로퍼티 키: 무명함수 리터럴(엄밀히 따지면 그냥 함수)

    ```javascript
    var o ={
    	x: f
    };
    var f = o.x(); // 이렇게 할 필요 없이 x();도 가능
    ```

    - ES5시절에는 메소드가 없었음(그냥 그렇게 불렀을 뿐)
    - ES6 넘어오면서 메소드가 생겨남(그냥 호출할 수 없고, 생성자 함수로 사용할 수 없음(new못붙임))

    ```javascript
    var o = {
    	x() {} // 이게 메소드
    };
    ```

    - 메소드 함수 내부에는 constructor를 포함한 메소드만 올 수 있음

- 전통적 클래스 기반 언어는 클래스 바디 내에 메소드, 클래스 멤버(변수 같은거)가 올 수 있음! 자바스크립트는 메소드만

  - 자바스크립트에도 클래스 멤버가 추가될 예정
  - 클래스 멤버에는 결국 인스턴스화되면 프로퍼티가 될 애들인데 프로퍼티의 값에는 무엇이 올 수 있나? 자바스크립트에서 사용하는 모든 값
  - 클래스 멤버에 할당한 함수는 인스턴스 함수(프로토타입이 아님) -> 인스턴스가 만들어질때마다 함수가 생김
  - 클래스로 리액트 컴포넌트를 만든다는 가정하에 - 클래스 멤버에 화살표 함수가 나을까 bind(this)가 나을까?

## 타입스크립트 2 - 클래스

- **타입스크립트란?** 자바스크립트(ES5)의 Superset
  - 그러므로 자바스크립트의 class는 사용하지 못함(ES6)

```typescript
// person.js
class Person {
  constructor(name) {
    // 클래스 프로퍼티의 선언과 초기화
    this.name = name;
  }

  walk() {
    console.log(`${this.name} is walking.`);
  }
}

// person.ts(4,10): error TS2339: Property 'name' does not exist on type 'Person'.
// person.ts(8,25): error TS2339: Property 'name' does not exist on type 'Person'.

// person.ts 이렇게 써야 에러가 나지 않음
class Person {
  // 클래스 프로퍼티를 사전 선언하여야 한다
  name: string;

  constructor(name: string) {
    // 클래스 프로퍼티에 값을 할당
    this.name = name; // 왜 또 name을? 나중에 알게됨
  }

  walk() {
    console.log(`${this.name} is walking.`);
  }
}

const person = new Person('Lee');
person.walk(); // Lee is walking
```

- 클래스 외부에서도 볼 수 있으면 퍼블릭, 외부에서 못보면 프라이빗
- 상속 관계에서도 private은 못봄
- 자바스크립트에서는 접근 제한자를 지원하지 않아 클로저로 구현했었으나 타입 스크립트에는 접근 제한자를 사용할 수 있음

### 접근 제한자

- 접근 제한자를 선언한 프로퍼티와 메소드에 대한 접근 가능성

| 접근 가능성      | public | protected | private |
| :--------------- | :----: | :-------: | :-----: |
| 클래스 내부      |   ◯    |     ◯     |    ◯    |
| 자식 클래스 내부 |   ◯    |     ◯     |    ✕    |
| 클래스 인스턴스  |   ◯    |     ✕     |    ✕    |

```typescript
class Foo {
  public x: string;
  protected y: string;
  private z: string;

  constructor(x: string, y: string, z: string) {
    // public, protected, private 접근 제한자 모두 클래스 내부에서 참조 가능하다.
    this.x = x;
    this.y = y;
    this.z = z;
  }
}

const foo = new Foo('x', 'y', 'z');

// public 접근 제한자는 클래스 인스턴스를 통해 클래스 외부에서 참조 가능하다.
console.log(foo.x);

// protected 접근 제한자는 클래스 인스턴스를 통해 클래스 외부에서 참조할 수 없다.
console.log(foo.y);
// error TS2445: Property 'y' is protected and only accessible within class 'Foo' and its subclasses.

// private 접근 제한자는 클래스 인스턴스를 통해 클래스 외부에서 참조할 수 없다.
console.log(foo.z);
// error TS2341: Property 'z' is private and only accessible within class 'Foo'.

class Bar extends Foo {
  constructor(x: string, y: string, z: string) {
    super(x, y, z);

    // public 접근 제한자는 자식 클래스 내부에서 참조 가능하다.
    console.log(this.x);

    // protected 접근 제한자는 자식 클래스 내부에서 참조 가능하다.
    console.log(this.y);

    // private 접근 제한자는 자식 클래스 내부에서 참조할 수 없다.
    console.log(this.z);
    // error TS2341: Property 'z' is private and only accessible within class 'Foo'.
  }
}
```

- 외부에서 본다는 건 엄밀히 말하면 -> 인스턴스를 통해서 본다
- x앞에 const를 붙여도되는가? 안됨-> 변수가 되어버림
- constructor 내부에는 const를 붙여 변수 선언문도 쓸 수 있고 뭐든지 할 수 있음

### 생성자 파라미터에 접근 제한자 선언

- 매개변수를 받았으면 this.x = x 해줘야하는거 아닌가?
- 왜 몸체에 x: string을 선언하지 않았는가?
- -> 축약 표현으로 사용 가능!

```typescript
class Foo {
  public x: string;

  constructor(x: string, y: string, z: string) {
    this.x = x;
  }
}

// 다음과 같이 축약 가능
class Foo {
  /*
  접근 제한자가 선언된 생성자 파라미터 x는 클래스 프로퍼티로 선언되고 지동으로 초기화된다.
  public이 선언되었으므로 x는 클래스 외부에서도 참조가 가능하다.
  */
  constructor(public x: string) { }
}
```

- 매개변수는 지역변수이므로 만약 public생략시 그냥 x는 지역 변수가 되어버림

```typescript
class Foo {
  // x는 생성자 내부에서만 유효한 지역 변수이다.
  constructor(x: string) {
    console.log(x);
  }
}
```

- 멤버가 아니라 그냥 프로퍼티!

### readonly 키워드

- 자바스크립트에선 사용 불가능함
- 읽기만 가능한 상태가 되므로 상수 선언에 사용

```typescript
class Foo {
  private readonly MAX_LEN: number = 5;
  private readonly MSG: string;
...
```

- 이 멤버변수는 인스턴스 프로퍼티일까 프로토타입 프로퍼티일까 클래스 프로퍼티일까?
  - 인스턴스 프로퍼티

```typescript
class Foo {
  public x: string;

  constructor(x: string, y: string, z: string) {
    this.x = x;
  }
}
```

- 여기서 constructor가 없을 경우 초기화가 되지 않을 뿐 프로퍼티는 생성이 됨(undefined상태)

### static 키워드

- 인스턴스로 접근하는게 아니라 클래스 이름으로 접근
- 상수와 같은 경우 클래스가 가지고 있는게 나음
  - 인스턴스마다 다 상수를 갖고 있는건 불필요

### 추상 클래스 (abstract class)

- 추상 클래스 내부에는 자바스크립트 내부랑 거의 유사한데, 추상 메소드도 올 수 있음
  - 추상 메소드는 바디가 없음
  - 식별자이름, 매개변수정보, 리턴정보만 있음
- 추상 메소드를 가지고 인스턴스를 만들고 싶어도(new사용) 만들 수 없음 : 바디가 없고 껍데기 정보만 있으므로
  - 상속(정확한 표현은 확장)용으로 만듦
  - 추상 클래스를 상속한 클래스는 추상 클래스의 추상 메소드를 반드시 구현해야 함

```typescript
abstract class Animal {
  // 추상 메소드
  abstract makeSound(): void;
  // 일반 메소드
  move(): void {
    console.log('roaming the earth...');
  }
}

// 직접 인스턴스를 생성할 수 없다.
// new Animal();
// error TS2511: Cannot create an instance of the abstract class 'Animal'.

class Dog extends Animal {
  // 추상 클래스를 상속한 클래스는 추상 클래스의 추상 메소드를 반드시 구현하여야 한다
  makeSound() {
    console.log('bowwow~~');
  }
}

const myDog = new Dog();
myDog.makeSound();
myDog.move();
```

<br />

## 인터페이스

- 인터페이스는 다 추상 메소드
- 인터페이스의 원래 목적은 abstract랑 거의 유사하게 상속을 위한 것
  - extends가 아니라 implements로 상속
- 타입스크립트에서 인터페이스는 타입으로 많이 쓰임

```typescript
// 인터페이스의 정의
interface Todo {
  id: number;
  content: string;
  completed: boolean;
}

// 이렇게도 쓸 수 있음
type Todo = {
  id: number;
  content: string;
  completed: boolean;
};

// 변수 todo의 타입으로 Todo 인터페이스를 선언하였다.
let todo: Todo;

// 이렇게도 쓸 수 있음
let todo: {
  id: number;
  content: string;
  completed: boolean;
};

// 변수 todo는 Todo 인터페이스를 준수하여야 한다.
todo = { id: 1, content: 'typescript', completed: false };
```

- Todo타입이라는게 새로 생겨난 것
- 모든 객체는 Object.prototype의 프로토타입 체인에 있음
  - 예) new Date로 생성하면 Date타입 -> object, Object 둘 다 됨
  - Array 타입도 되고, Class 타입도 됨

<hr />

- 타입 스크립트 프로젝트 폴더 구조는 어떻게 만드는게 좋을까?
  - src에 코딩해서 컴파일하면 dist폴더로 가는게 보통
  - src폴더를 만들어서 ts파일 넣고, json 열어서 `    "outDir": "./dist",   `추가

<hr />

- 타입을 재사용하기 위해 interface를 사용
- interface와 type중 무엇을 써야할까?
  - 상관 없지만 일관성 있게 쓸 것
- interface는 객체에 상속할 수 있으므로 상속이 필요할 경우 interface를 쓸 것

### 유니온 타입

```typescript
// string이 되어야 할때도 있고 null이 되어야 할 때도 있음
// 숫자 범위 제한이 있을 때
// 유니온 타입
let foo: string | null;
let num: 1 | 2 | 3 | 4;
```

- 개수 제한 없음

```typescript
type Foo = 1 | 2 | 3 | 4;
let foo: Foo;
```

- 재사용을 하려면 이렇게 사용하면 됨

### 함수

```typescript
// 함수(매개변수, 리턴타입)
type Fun = (a: string, b: number) => void;
let fun: Fun;
fun = function () {};
fun('', 1);
```

- 함수의 경우는 위와 같이 씀
- void는 리턴 값 상관 없음 뜻

### 클래스

```typescript
// 인터페이스의 정의
interface ITodo {
  id: number;
  content: string;
  completed: boolean;
}

// Todo 클래스는 ITodo 인터페이스를 구현하여야 한다.
class Todo implements ITodo {
  constructor (
    public id: number,
    public content: string,
    public completed: boolean
  ) { }
}

const todo = new Todo(1, 'Typescript', false);
```

- implement로 상속

### 선택적 프로퍼티

```typescript
interface UserInfo {
  username: string;
  password: string;
  age?    : number;
  address?: string;
}

const userInfo: UserInfo = {
  username: 'ungmo2@gmail.com',
  password: '123456'
}
```

- ?는 있어도 되고 없어도 된다는 뜻

### 인터페이스 상속

- 인터페이스는 인터페이스를 상속할 수 있음

### 다양한 예제

```typescript
// 문자열 리터럴로 타입 지정
type Str = 'Lee';

// 유니온 타입으로 타입 지정
type Union = string | null;

// 문자열 유니온 타입으로 타입 지정
type Name = 'Lee' | 'Kim';

// 숫자 리터럴 유니온 타입으로 타입 지정
type Num = 1 | 2 | 3 | 4 | 5;

// 객체 리터럴 유니온 타입으로 타입 지정
type Obj = {a: 1} | {b: 2};

// 함수 유니온 타입으로 타입 지정
type Func = (() => string) | (() => void);

// 인터페이스 유니온 타입으로 타입 지정
type Shape = Square | Rectangle | Circle;

// 튜플로 타입 지정
type Tuple = [string, boolean];
const t: Tuple = ['', '']; // Error
```

- 튜플은 어디에? useState!
  - 첫번째는 any, 두번째는 function
- 인터페이스는 extends 또는 implements될 수 있지만 타입 앨리어스는 extends 또는 implements될 수 없음
  - 상속을 통해 확장이 필요하다면 타입 앨리어스보다는 인터페이스가 유리
  - 하지만 인터페이스로 표현할 수 없거나 유니온 또는 튜플을 사용해야한다면 타입 앨리어스를 사용하는 것이 나음

## 제네릭

- 막간 자바스크립트: 자바스크립트에도 프래그먼트가 있음
  - 사용 예) 10개 짜리 DOM 묶음을 한 번에 append하고 싶을 때 프래그먼트 사용해서 넣을 수 있음

```typescript
class Queue {
  protected data = []; // data: any[]

  push(item) {
    this.data.push(item);
  }

  pop() {
    return this.data.shift();
  }
}

const queue = new Queue();

queue.push(0);
queue.push('1'); // 의도하지 않은 실수!

console.log(queue.pop().toFixed()); // 0
console.log(queue.pop().toFixed()); // Runtime error
```

- 여기서 문제 해결을 위해 만약 item에 타입 한가지를 줘버리면 범용성을 잃게됨
- 그렇다면 상속을 받아서?

```tsx
// Queue 클래스를 상속하여 number 타입 전용 NumberQueue 클래스를 정의
class NumberQueue extends Queue {
  // number 타입의 요소만을 push한다.
  push(item: number) {
    super.push(item);
  }

  pop(): number {
    return super.pop();
  }
}

const queue = new NumberQueue();

queue.push(0);
// 의도하지 않은 실수를 사전 검출 가능
// [ts] Argument of type '"1"' is not assignable to parameter of type 'number'.
// queue.push('1');
queue.push(+'1'); // 실수를 사전 인지하고 수정할 수 있다

console.log(queue.pop().toFixed()); // 0
console.log(queue.pop().toFixed()); // 1
```

- constructor가 없고 setter가 오직 push뿐, pop은 getter
- 타입 추론에 의해 data가 number[]가 됨
- 그런데 만약, string[]도 만들고, 계속 다양한 타입의 배열이 추가된다면? 너무 많은 클래스가 생겨남
- 해결해보자.

```typescript
class Queue<T> {
  protected data: Array<T> = [];
  push(item: T) {
    this.data.push(item);
  }
  pop(): T {
    return this.data.shift();
  }
}

// number 전용 Queue
const numberQueue = new Queue<number>();

numberQueue.push(0);
// numberQueue.push('1'); // 의도하지 않은 실수를 사전 검출 가능
numberQueue.push(+'1');   // 실수를 사전 인지하고 수정할 수 있다

console.log(numberQueue.pop().toFixed()); // 0
console.log(numberQueue.pop().toFixed()); // 1

// string 전용 Queue
const stringQueue = new Queue<string>();

stringQueue.push('Hello');
stringQueue.push('World');

console.log(stringQueue.pop().toUpperCase()); // HELLO
console.log(stringQueue.pop().toUpperCase()); // WORLD

// 커스텀 객체 전용 Queue
const myQueue = new Queue<{name: string, age: number}>();
myQueue.push({name: 'Lee', age: 10});
myQueue.push({name: 'Kim', age: 20});

console.log(myQueue.pop()); // { name: 'Lee', age: 10 }
console.log(myQueue.pop()); // { name: 'Kim', age: 20 }
```

- 타입을 매개변수처럼 사용한 것(외부의 값에 따라 결정됨)
- 제네릭은 선언 시점이 아니라 생성 시점에 타입을 명시하여 하나의 타입만이 아닌 다양한 타입을 사용할 수 있도록 하는 기법
  - 한 번의 선언으로 다양한 타입에 재사용이 가능하다는 장점
  - T는 관용적인 표현일뿐 필수는 아님
  - 한 번만 사용할 수 있는게 아니라, 두 개 이상도 사용 가능

```typescript
function reverse<T>(items: T[]): T[] {
  return items.reverse();
}
```

- 함수에는 다음과 같이 사용할 수 있음
- reverse옆의 T는 문법, 클래스도 식별자 옆에 써줌