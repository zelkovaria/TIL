# index signature와 Record type

리팩토링을 하던 도중 아래의 사진처럼 `~형식의 매개 변수가 포함된 인덱스 시그니처를 찾을 수 없습니다`라는 에러가 떴었다. 즉, 인덱스 형식의 사용이 불가능 하다는 소리였다

![image](https://github.com/user-attachments/assets/02b6aaa5-573a-4e42-b9e8-4d5a60ad10c5)

## 오류 원인

```tsx
// HomeworkBox.tsx
export const HOMEWORK_DATA = {
  WEB: {
    title: "자기소개 페이지를 첨부해주세요 (웹파트)",
    taskHelperText: "※ 학번.zip 파일 형식으로 첨부해주세요.",
  },
  SERVER: {
    title: "지원 과제 Repo 링크를 남겨주세요 (서버 파트)",
    taskHelperText: "※ 지원 기간이 지난 후의 commit 기록은 반영되지 않습니다.",
    LinkHelperText:
      "※ 다른 주소가 아닌 GitHub repository 링크 혹은 구글 드라이브 링크(액세스 권한-링크가 있는 모든 사용자)를 첨부해주세요.",
  },
};
```

`HomeworkBox.tsx`에서 `HOMEWORK_DATA`는 **key가 WEB, SERVER**에 따라 title, taskHelperText 그리고 SERVER의 경우에는 추가로 LinkHelperText를 갖고있었다. 때문에 `WEB: { title: string; taskHelperText: string; }`과 같은 type을 지정해주하면 되지 않을까?라는 생각으로 처음에 코드를 작성했었다. 그렇다면 여기서 말하는 **<mark>인덱스 시그니처(index signature)</mark>란 무엇일까?**

## index signature

- 인덱스 시그니처는 `{[key: T] : U }` 형식으로
  - 객체가 여러개의 key를 가질 수 있으며 key와 매핑되는 value를 가지는 경우 사용한다
  - 특정 value에 점근하고 싶을 때, 해당 value의 key를 obj에 문자열로 인덱싱해 참조하는 방법
    - 객체의 특정 value에 접근하고 싶을 때
    - 객체의 속성들(key, value)의 모든 이름이나 type을 명확히 알지 못할 때, 속성의 type만 우선 지정해주어 객체의 정보들에 접근하기 위해 사용하는데 이때 **반드시 key의 type은 string 혹은 number여야만 한다. 때문에 내 코드에서 오류가 발생했었다 🥲**
    - 속성의 type을 알고있는 상황이라면, 정확한 type을 정의하여 실수를 방지할 수 있다는 장점이 있다
- 인덱스 시그니처(index signature)는 대괄호로 객체를 접근하는 방법으로 사용 예시는 아래와 같다!

  ```tsx
  // index signature
  type menuInfo = {
    [name: string]: number;
  };

  let food: menuInfo = {
    짜글이: 12000,
    볶음밥: 13000,
    카레: 14000,
  };
  ```

- 인덱스 시그니처의 단점으로 내가 작성했던 코드처럼 문자열 리터럴을 key로 사용하는 경우 오류가 발생한다는 것이다. 그러면 이러한 문제는 어떻게 해결할 수 있을까? 🤔

## Record Type

Record Type은 key로 문자열 리터럴을 허용한다!

이전의 index signature 코드를 Record Type을 사용하면 아래와 같다

```tsx
// Record Type
type menuInfo = Record<string, number>;

let food: menuInfo = {
  짜글이: 12000,
  볶음밥: 13000,
  카레: 14000,
};
```

추가로 문자열 리터럴을 사용하여 아래와 같이도 적용할 수 있다!

```tsx
type foods = "짜글이" | "볶음밥" | "카레";

type menuInfo = Record<foods, number>;

let human: menuInfo = {
  짜글이: 12000,
  볶음밥: 13000,
  카레: 14000,
};
```

- `Record<key, value>` 와 같이 작성하며 키가 key 타입, value는 value타입인 `객체 타입`을 생성한다
- 결론적으로 인덱스 시그니처와 유사한 기능을 수행하지만 차이점은 **key로 문자열 리터럴을 사용**할 수 있다는 것이다!

  ```tsx
  // 아래와같은 인덱스 시그니처는 key에 문자열 리터럴 사용을 못함
  // 틀린 예시
  type Zelkovaria = {
    [name: "느티나무" | "hyein"]: number;
  };
  // Record를 사용한 올바른 예시
  type Names = "느티나무" | "hyein";
  type Zelkovaria = Record<Names, number>;

  let scores: Zelkovaria = {
    느티나무: 100,
    hyein: 200,
  };
  ```

## 오류 해결

마찬가지로 오류 해결을 위해 IPartKey라는 type을 사용하여 IHOMEWORK_DATA의 key type을 지정하고, 이를 HOMEWORK_DATA에 적용해줌으로써 오류를 해결할 수 있었다!

```tsx
export type IPartKey = "WEB" | "SERVER";
export type IHOMEWORK_DATA = Record<
  IPartKey,
  { title: string; taskHelperText: string; LinkHelperText?: string }
>;

export const HOMEWORK_DATA: IHOMEWORK_DATA = {
  WEB: {
    title: "자기소개 페이지를 첨부해주세요 (웹파트)",
    taskHelperText: "※ 학번.zip 파일 형식으로 첨부해주세요.",
  },
  SERVER: {
    title: "지원 과제 Repo 링크를 남겨주세요 (서버 파트)",
    taskHelperText: "※ 지원 기간이 지난 후의 commit 기록은 반영되지 않습니다.",
    LinkHelperText:
      "※ 다른 주소가 아닌 GitHub repository 링크 혹은 구글 드라이브 링크(액세스 권한-링크가 있는 모든 사용자)를 첨부해주세요.",
  },
};
```

### 그렇다면 이유는? type narrowing

관련 개념상 index signature는 특정 키의 타입 자체를 string 혹은 number로 상대적으로 넓은 범위를 지니고 있다. 즉, 상위개념인 string이라는 그 자체의 타입은 좁은 범위로 지정해둔 값의 하위개념으로 들어갈 수 없기 때문이라는 것을 알 수 있었다.

정리하자면, index signature와 달리 Record type은 키가 특정 값으로 제한 될 수 있기 때문에 타입을 좁힐 수 있다

## 이외의 ReactNode 할당 불가

ReactNode 할당 불가이슈 해결은 생각보다 더 간단했다,,ㅎ 현재 오류가 뜨는 부분인 errors[’link’]?.message는 string | ~ | undefined 타입을 반환해서 생긴 오류이다. 이를 해결하기 위해 해당 부분을 <></>로 감싸면 표현식이 ReactNode의 일부로 간주되어 오류없이 처리할 수 있다!

![image](https://github.com/user-attachments/assets/686f4400-b11b-4c08-9bb1-05824c0c7753)

```tsx
{errors['link']?.message}를

<>{errors['link']?.message}</>로
```

참고 자료

https://reactnative.dev/docs/react-node<br/>
https://soopdop.github.io/2020/12/01/index-signatures-in-typescript/<br/>
https://cheeseb.github.io/typescript/typescript-record/<br/>
[https://velog.io/@ddowoo/Typescript-유틸리티-타입-Record-Type이란](https://velog.io/@ddowoo/Typescript-%EC%9C%A0%ED%8B%B8%EB%A6%AC%ED%8B%B0-%ED%83%80%EC%9E%85-Record-Type%EC%9D%B4%EB%9E%80)
