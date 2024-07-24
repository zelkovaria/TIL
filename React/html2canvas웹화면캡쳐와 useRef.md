# html2canvas를 사용한 웹 화면 캡쳐하기

프로젝트를 준비하면서 유저에게 편리함을 주기 위해 특정 컴포넌트를 이미지로 저장할 수 있는 기능을 고려하게 되었다!<br />
보통 프로젝트를 진행하게 되면 SVG 요소를 많이 사용하는데, 비슷한 라이브러린 `dom-to-image`는 _img파일을 인식하지 못하는 이슈_ 가 있다고 한다. 반대로 html2canvas는 SVG를 포함하여 캡쳐할 수 있어서 해당 방법을 적용해봤다!

> 추가로 dom-to-image가 속도 측면에서는 html2canvas보다는 빠르지만 cors에러 또한 발생할 수 있다고 한다

`html2canvas`: DOM을 기반으로 하며, 사용자 브라우저에서 전체 또는 특정 영역의 스크린샷을 생성할 수 있는 라이브러리<br/>

### 사용법

라이브러리와 [관련된 사이트](https://html2canvas.hertzen.com/)이다!

- `yarn add html2canvas` 후 `import`하여 사용 <br/>
- 예시 코드

  ```tsx
  import html2canvas from "html2canvas";
  import styled from "styled-components";
  import { useRef } from "react";

  import "./App.css";

  function App() {
    const imageRef = useRef(null);

    const downLoadImage = () => {
      if (imageRef.current) {
        html2canvas(imageRef.current, { backgroundColor: null }).then(
          (canvas) => {
            const link = document.createElement("a");
            link.download = "downimage.png";
            link.href = canvas.toDataURL("image/png");
            link.click();
          }
        );
      }
    };

    return (
      <Wrapper>
        <ImageDiv ref={imageRef} />
        <button onClick={() => downLoadImage()}>다운</button>
      </Wrapper>
    );
  }
  const ImageDiv = styled.div`
    background-color: pink;
    width: 200px;
    height: 200px;
  `;

  const Wrapper = styled.div`
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
  `;
  export default App;
  ```

  - imageRef: 'useRef'훅을 사용하여 'ImageDiv' 요소에 접근 할 수 있도록 한다
  - downLoadImage: imageRef가 가리키는 요소를 캡쳐하여 PNG로 다운로드 하는 역할의 함수이다
  - return 부분에서 ImageDiv는 'imageRef'를 참조하며, 다운 버튼을 클릭하면 'downLoadImage'함수가 실행되는 방식이다!

## useRef

코드에서 사용이 된 useRef란 무엇이고 언제 사용하는 걸까?

> 예시 코드처럼 JavaScript에서 DOM을 특정해야할 때 _DOM selector_ 를 사용하는데, 마찬가지로 리액트에서도 특정 DOM을 사용해야하는 경우가 있다. 이러한 상황에서 사용하는게 React Hook 중 하나인 `useRef`이다!

- useRef는 보통 렌더링과 관련이 없는 값을 저장하는데 사용한다 (scroll 위치, 배열에 새 항목을 추가할 때 필요한 고유값인 key 등)
- 주로 DOM 요소에 대한 참조를 만들거나, 렌더링과 무관한 데이터를 저장하는 데 활용한다
- useRef로 생성된 객체는 `.current` 프로퍼티를 통해 저장된 값을 얻을 수 있으며, 이 값이 변경되어도 컴포넌트는 다시 렌더링되지 않는다!!

> 따라서, `useState`는 주로 _렌더링과 관련된 동적인 상태_ 를 관리할 때 사용되며, `useRef`는 렌더링과 무관한 값을 저장하거나 DOM 요소에 접근할 때 사용된다

### 사용 방법

1. ref 객체 선언

```tsx
import { useRef } from "react";

function MyComponent() {
  const inputRef = useRef(null);
}
```

2. 해당 ref를 사용할 DOM 노드의 속성으로 전달

```tsx
return <input ref={inputRef} />;
```

3. ref 객체의 current 프로퍼티를 DOM 노드로 설정

```tsx
function handleClick() {
  inputRef.current.focus();
}
```

노드가 화면에서 제거되면 React는 current 프로퍼티를 다시 null로 설정한다

<br/>
출처<br/>
https://ko.react.dev/reference/react/useRef#manipulating-the-dom-with-a-ref
https://yong-nyong.tistory.com/53
