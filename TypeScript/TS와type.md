# 타입스크립트(TypeScript)란?

> 💡 타입스크립트란, JavaScript를 기반으로 만들어진 언어로, JS가 가지고 있는 여러 문제를 해결하고 보안하기 위해 만들어졌다. `타입 안정성`이 있다는 특징이 있다.<br/>
> JavaScript는 **dynamically typed**로 런타임 환경 때 타입이 결정이 되는데, 이로 인해 사용자가 어플리케이션을 사용하면서 에러가 발생하는 문제점이 있다. TS는 이러한 문제를 보완하여, 개발시 실시간으로 검사를 받을 수 있어 안정적이고 확장이 쉬운 `객체 지향 프로그래밍`(Object Oriented Programming)이 가능하다

- statically typed: 컴파일러시 type이 결정되고 확인된다 ex) python, JavaScript
- dynamicall typed: 런타임 환경에서 type이 결정되고 확인된다 ex) Java, TypeScript

<br/>

# Type 종류

## any

- 보통 특정 값으로 인하옅 타입 검사를 원치 않을 때 사용하는데, 즉 타입을 새로 정의하고 싶지 않을 때 유용하다
- 타입이 지정되어 있지 않은 경우 컴파일러가 기본 동작으로 any 타입을 부여한다
- 사용은 가능한 비추천한다!

## array

- 만약 문자열 배열이라면 string[] 혹은 Array<string>과 같이 표현하는게 기본적인 규칙이다

```tsx
let arr: dessert[] = ["cookie", "bread"];
// 위는 아래와 같다
let arr: Array<string> = ["cookie", "bread"];
```

## tuple

- 튜플은 최소한의 길이를 가져야하며, 특정 위치에 특정 타입이 있어야한다는 특징이 있다
- interface, type alias로 대체해서 사용하는 걸 추천하며 튜플을 사용을 권장하지 않는다

```tsx
["cookie", true, 1500];
// 해당 튜플은 string, boolean, number 순서에 해당되는 3개의 값이 와야한다

const dessert: [string, boolean, number] = ["cookie", false, 2700];
```

## enum

- 특정 상수값을 한 곳에 모아둔 집합을 의미하며, TypeScript 자체 제공한다

```tsx
enum Dessert {
  Cookie,
  Puding,
  Waffle,
}
let food: Dessert = Dessert.puding;
```

- 인덱스 번호로도 접근이 가능하다

## void

아무것도 리턴하지 않는 경우에 붙이며 생략이 가능하다

## never

리턴하는 값이 절대 없는 경우에 사용하며, 보통 에러를 던지거나 while문으로 끝나지 않게 하는 경우에 사용한다

## function

- 기본적으로 함수 타입은 매개변수와 반환 값의 타입을 포함하여 함수를 정의할 수 있다
  - `let width: (min:number, max:number) => number;`
    - width는 인자 min, max를 받는 함수이며 두 인자는 number type을 가진다
    - 해당 함수는 number type을 갖는 값을 반환한다
  - ex) 문자 혹은 undefined를 return하는 함수는?
    ```tsx
    function test(): string | undefined {
      return undefined;
    }
    ```
    추가로, unknown은 어떤 타입의 데이터가 담길지 모를 때 사용하는데 최대한 지양하는 것이 좋다!
- 함수의 파라미터 값으로 `max = 100` 과 같이 온다면 TS가 알아서 max의 type을 number로 추론!
- `call signature`: 인자의 타입과 함수의 반환 타입을 알려주는 것

  - 즉, 함수가 어떻게 작동하는지 서술해둘 수 있음(함수의 타입을 먼저 설명하고 구현을 하게되는 순서)

  ```tsx
  type Add = (a: number, b: number) => number;

  const add: Add = (a, b) => a + b;
  // 처럼 사용이 가능함
  ```

## union

- 자바스크립트의 OR 연산자(`||`)와 같이 A이거나 B이다 라는 의미의 타입과 같으며, 특정 값만 지정하고 싶을 때 주로 사용한다
- 아래와 같은 예시를 볼 때, getValue 함수는 number 또는 string type의 값을 반환하고 있음을 확인할 수 있다

```tsx
function getValue(key: string): number | string {
  const values: { [key: string]: number | string } = {
    price: 10000,
    tag: "cake",
  };

  return values[key];
}

const value1 = getValue("price"); // number 타입
const value2 = getValue("tag"); // string 타입

console.log(value1); // 출력: 10000
console.log(value2); // 출력: cake
```

- union은 발생할 수 있는 모든 케이스 중 한가지만 선택하는 거라면, `Intersection`은 그 모든 것을 합한 성격 즉 `and`의 느낌과 유사하다 할 수 있다
- discriminated union을 사용하여 union 타입을 더 안전하게 사용할 수 있다

  ```tsx
  interface Square {
    kind: "square";
    size: number;
  }

  interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
  }

  interface Circle {
    kind: "circle";
    radius: number;
  }

  type Shape = Square | Rectangle | Circle;

  function getArea(shape: Shape): number {
    switch (shape.kind) {
      case "square":
        return shape.size * shape.size;
      case "rectangle":
        return shape.width * shape.height;
      case "circle":
        return Math.PI * shape.radius * shape.radius;
      default:
        return 0;
    }
  }

  // 사용 예시
  const mySquare: Square = { kind: "square", size: 10 };
  const myRectangle: Rectangle = { kind: "rectangle", width: 10, height: 5 };
  const myCircle: Circle = { kind: "circle", radius: 7 };

  console.log(getArea(mySquare)); // 출력: 100
  console.log(getArea(myRectangle)); // 출력: 50
  console.log(getArea(myCircle)); // 출력: 153.93804002589985
  ```

## object

- 타입 정의 방법
  ```tsx
  const student: {
    name: string;
    age?: number; //optional
  } = {
    name: "hyein",
  };
  ```

## null/undefined

- `undefined`: 무엇인가가 초기화 되지 않은경우
- `null`: 무엇인가를 초기화 후 의도적으로 null값을 할당한 경우

## Type Alias

- 특정 타입이나 인터페이스를 참조할 수 있는 타입 변수를 의미한다. 즉, 내가 원하는 새로운 type을 지정할 수 있다
  ```tsx
  type StringName = string;
  const name: StringName = "hyein";
  ```
- 새로운 타입 값을 하나 생성하는 것이 아니라 정의한 타입에 대해 나중에 쉽게 참고할 수 있게 이름을 부여하는 것과 같다고 한다!

<img src="https://github.com/user-attachments/assets/fa11014b-c41a-41d8-8a1e-bc92a927d230" width="350">

현재 interface로 선언한 ValuesType은 정의한 타입이 보이지 않지만, 별칭으로 선언한 타입은 프리뷰로 쉽게 확인할 수 있다!

<img src="https://github.com/user-attachments/assets/4fc81b03-8a57-4976-b286-58ba1033f1c5" width="400" >

<br/>

# 제네릭(generic)

> 💡 **제너릭은 타입 정보가 동적으로 결정되는 타입이다.**
> 출처: 실전 리액트 프로그래밍

- 한마디로 타입을 변수화한 것이라고 할 수 있다
- 사용 예시

  ```tsx
  function add<T>(x: T, y: T): T {
    return x + y;
  }

  add<number>(3, 4); // number type의 값인 7이 반환
  add<string>("jeong", "hyein"); // string type의 값인 'jeonghyein'이 반환
  ```

  - 타입의 이름인 T는 자유롭게 작성해도 되는데, 이는 함수나 클래스의 선언 시점이 아닌 **사용 시점에 타입을 선언**하기 때문이다
  - 즉, 선언을 할 때는 문자로만 작성했다가, 생성하는 시점에서 사용할 타입이 결정되어 다양한 사용이 가능해진다

- 제네릭에서 인수로 **배열**이 들어오는 경우 `T[]`, `Array<T>`와 같이 작성을 해줘야한다

  ```tsx
  function exampleFn<T>(arg: T[]): T[] {
  	...
  	return arg;
  }

  function exampleFn<T>(arg: Array<T>): Array<T> {
  	...
  	return arg;
  }
  ```

⇒ 제네릭 함수 `pay<T extends 인터페이스명>(변수명: T): T`는 `Employee` 인터페이스를 확장하는 어떤 타입 `T`도 받을 수 있으며, 그 타입 `T`의 인스턴스를 인수로 받아 동일한 타입의 인스턴스를 반환한다!

# 인터페이스

- 인터페이스는 상호 간에 정의한 약속 혹은 규칙을 의미하며, 보통 아래와 같은 범주에 대해 약속을 정한다고 한다!
  - 객체의 스펙(속성과 속성의 타입)
  - 함수의 파라미터
  - 함수의 스펙(파라미터, 반환 타입 등)
  - 배열과 객체를 접근하는 방식
  - 클래스<br/>
    [출처](https://joshua1988.github.io/ts/guide/operator.html)
- 즉, 인터페이스는 오로지 `오브젝트의 모양을 특정`해주기 위한 목적을 지니고 있다고 할 수 있다
  ```tsx
  // type 사용 예시
  type Age = 25 | 26 | 27;
  type Hobby = "coding" | "cooking" | "reading";
  // interface 사용 예시
  interface User {
    name: string;
    age: Age;
    hobby: Hobby;
  }
  ```
- 💡 그렇다면, type과 interface의 차이점은 뭘까?

  - 차이점: type 키워드는 interface 키워드에 비해 좀 더 활용할 수 있는게 더 많음<br/>
    → 오브젝트의 모양을 정해줄 수 있다<br/>
    → 특정 값들로만 제한할 수도 있다<br/>
    → 타입 alias를 만들 수 있다<br/>
    ⇒ 인터페이스는 객체 지향 프로그래밍의 개념을 활용해서 디자인되었고, 타입은 좀 더 유연함(타입 alias 가능, 지정한 값들로만 타입 지정이 가능 등등)

  ```tsx
  // interface
  interface User {
    name: string;
  }

  interface Student extends User {}

  const hyein: Student = {
    name: "hyein",
  };
  ```

  ```tsx
  // type
  type User = {
    name: string;
  };

  type Student = User & {};

  const hyein: Student = {
    name: "hyein",
  };
  ```

  _소프트웨어 엔터티(클래스, 모듈, 함수 등)는 확장에는 열려 있어야 하지만 수정에는 닫혀 있어야_ 한다는 객체지향프로그래밍의 원칙들에 따라 가급적 확장 가능한 인터페이스로 선언하면 좋다고 한다!👍🏻

- 옵셔널 체이닝(optional chaining)

  - 객체가 null 또는 undefined이면 undefined를 리턴하고 그렇지 않은 경우 데이터 값을 리턴하는 문법을 의미한다
  - 장점으로 if 문을 줄일 수 있다

    ```tsx
    interface User {
      name?: string;
      address?: {
        city?: string;
        zipcode?: string;
      };
    }

    // if문
    if (user && user.address && user.address.zipcode) {
      console.log(user.address.zipcode);
    }

    // 옵셔널 체이닝
    const city = user?.address?.city ?? "City not available";
    console.log(city);
    ```

참고

https://joshua1988.github.io/ts/guide/operator.html
https://www.typescripttutorial.net/typescript-tutorial/typescript-type-inference/
https://radlohead.gitbook.io/typescript-deep-dive/recap/null-undefined
https://jaeheon.kr/155#google_vignette
[https://velog.io/@mokyoungg/TS-제너릭Generic](https://velog.io/@mokyoungg/TS-%EC%A0%9C%EB%84%88%EB%A6%ADGeneric)<br/>
[https://inpa.tistory.com/entry/TS-📘-타입스크립트-Generic-타입-정복하기](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-Generic-%ED%83%80%EC%9E%85-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0)
