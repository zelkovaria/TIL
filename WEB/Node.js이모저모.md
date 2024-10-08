## Node란?

js는 인터프리터가 웹 브라우저 안에만 있었어서 ‘라이언 달’이 만들었다
<br/>(초기에는 웹 페이지의 동적인 기능을 위해 웹 브라우저에서 주로 사용이 되었다가 Node.js와 같은 서버 측 환경에서도 JavaScript를 실행이 가능해짐) <br/>
이때, 인터프리터는 코드를 한줄씩 읽으므로 컴파일러와는 좀 다르다. 추가로 런타임은 모듈이 추가로 있어 포괄적인 무언가를 제공하는 프로그램이 가능하다 <br />

> `💡 Node.js 는 브라우저 바깥에서 크롬의 코어인 V8 자바스크립트 엔진을 구동!`<br/>
> 웹 브라우저 환경이 아닌 곳에서도 자바스크립트를 사용하여 실행 가능
> <br/>
> 리액트가 SSR 같은 Node.js 를 필수로 필요로 하는 기술을 사용하지 않는다는 전제하에, 리액트 애플리케이션은 웹 브라우저에서 실행되는 코드이므로 Node.js와 직접적인 연관은 없지만, 프로젝트 개발하는데 **필요한 주요 도구들이 Node.js를 사용**하기 때문에 설치
> Node.js를 설치하면 Node.js 패키지 매니저 도구인 npm이 설치된다!  
> `→ npm으로 수많은 개발자가 만든 패키지 (재사용 가능한 코드)를 설치하고 설치한 패키지의 버전 관리 용이`

<details>
  <summary>그렇다면, nvm이란?</summary>
  <div markdonw="1">
    <ul>
      <li>노드 버전 매니저(다양한 노드 버전 설치 가능)로, npm의 패키지 매니저이다</li>
      <li>패키지 매니저는 언어에만 종속적이지 않다</li>
      <li>패키지 매니저는 패키지 설치, 리스트업 및 저장 등의 관리 + @를 한다</li>
    </ul>
  </div>
</details>

## 환경 변수(feat. Node.js)

- build를 하기 전에 `yarn start`를 하면 HTTP request를 했을 때 HTTP response를 할 수 있게 해주는 서버를 열어주는 역할이 필요하다!
  <details>
  <summary>CRA를 사용한 React는?</summary>
  React에서는 CRA를 사용하는 경우 해당 역할을 해주는데, <strong>REACT_APP_</strong>라는 명으로 시작되지 않는 환경변수는 무시되므로 보안이 필요한 환경변수 유출을 위할 때는 조심해야한다!<br/>
  이때 기준으로 환경변수가 있으면 <mark>process.env</mark>를 읽게 해준다
  </details>
- 즉, 환경 변수는 프로그램이 실행될 때 프로그램 밖에서 값을 전달받을 수 있는 방법중 하나이며, 가장 흔히 사용되는 방법이기도 하다(환경변수가 꼭 환경만을 의미하지는 않는다고 한다)

> ## 그래서 환경변수가 뭔데?
>
> > **`💡 운영체제에서 프로그램이나 프로세스가 실행될 때 참조할 수 있는 동적 값을 저장하는 변수를 의미한다! `<br/>
> > 즉, 시스템의 속성을 기록하고 있는 변수와 같음
> > 추가로 GitHub에 대놓고 표시를 하고싶지 않을 때 .env를 통해 올리듯 보안상 문제를 어느정도 완화해주는 역할을 한다**  
> > 예를들어, NODE_ENV의 역할은?<br/>
> > 내가 어떤 production 환경인지를 알려주는 변수와 같음<br/>
> > Node.js 애플리케이션의 동작을 제어하기 위해 설정할 수 있는 환경 변수
> > ⇒ 개발(Development), 테스트(Testing), 생산(Production) 등 다양한 환경에서 애플리케이션의 동작을 조절함

> ### 그렇다면 리액트는 프론트엔드 프레임워크인데 왜 Node.js가 필요할까?
>
> 1.  프로젝트 개발하는데 필요한 주요 도구들이 Node.js를 사용하기 때문에 설치
> 2.  Node.js를 설치하면 Node.js 패키지 매니저 도구인 npm이 설치되기 때문
>     → npm으로 수많은 개발자가 만든 패키지 (재사용 가능한 코드)를 설치하고 설치한 패키지의 버전 관리 용이
> 3.  build를 하기 위해 필요

<br/>

> ### Node.js에서 환경변수의 역할은 무엇일까?
>
> 환경에 따라 다른 변수를 쓰고싶을 수 있음 <br/>→ Vercel만 가도 적어도 3가지 환경(Production, Development, Preview)을 줌 <br/>→ 그 각각일 때 어떤 값을 쓸건지(실행할 때 주입해주고 싶은 가변적인 값일 때 사용)
