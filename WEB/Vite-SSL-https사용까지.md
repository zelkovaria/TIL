# Vite, SSL, 로컬에서 https 사용하기 레츠고 🏃‍♀️

### 살짝 짚고 넘어가는 HTTPS

> **💡 HTTPS란?**<br/>
> HyperText Transfer Protocol Secure의 약자로, HTTP의 보안 버전을 의미한다<br/>
> → HTTPS는 대칭키 암호화와 공개키 암호화를 조합하여 작동하며 SSL 프로토콜 위에서 HTTPS 프로토콜이 동작한다!<br/><br/>
> HTTPS의 장단점은?
>
> - 웹사이트의 무결성을 보호
> - SEO 관련 내용을 HTTPS 웹사이트에 대해 적용중이므로, 키워드 검색 시 상위 노출이 되는 기준 중 하나인 보안요소에 해당
> - 신뢰할 수 없는 CA 기업을 통해서도 인증서를 발급받을 수 있기때문에 무조건 안전한 것은 아님!

## SSL과 사용 이유 🤔

- SSL(Secure Socket Layer)는 보안 프로토콜을 통해 클라이언트와 서버가 보안이 향상된 통신을 하는 것을 의미! [출처](https://blog.naver.com/skinfosec2000/222135874222)
  - 즉, https를 이용한 인터넷은 SSL(TLS)를 이용한 것이라고 할 수 있음!

정리하자면 SSL 통신을 통해 데이터를 암호화해주는 것이라고 할 수 있다!

→ 이전에는 `MITM attack`이 많았다고 한다 (http는 어쨌거나 통신인데 서버한테 내가 클라이언트인 척, 클라이언트는 서버인 척 하면서 중간에서 내용을 다 까볼 수 있기 때문에,,) 이를 위해 나온 https는 중간에서 못 까보게 방어하는 것과 같다

⇒ 락 걸어주는 것중 하나가 바로 `SSL 보안 인증서`

- 인증 받은 인증서가 맞는지 어케 아냐면 컴퓨터별로 싸인한 인증서가 각 컴퓨터에 존재한다고 한다,,!<br/>
  → 싸인을 하면서 어떤 회사가 믿을만한지도 확인해주고, 그 회사들 밑으로 마찬가지로 또 확인을 해줌<br/>
  ⇒ 하청을 쭉 내려가다가 내 상위 기관, 내 본인의 싸인이 있으면 그걸 상위로 쭉 올라가서 쓸 수 있는 인증서가 맞는지 인증을 받으면 바로 그게 중요한 ssl 인증서이다. 인증서는 도메인창에 있는 주소를 가지고 싸인한다<br/>
  _참고로 ip로는 https를 못건다고 한다(https는 도메인이 맞는거라고 인증을 해주는 것이기 때문!!)_<br/>
  ⇒ 서버가 ec2로 배포해서 ip를 주면 우리가 배포할 땐 https걸어달라 할거면 도메인 필요하다는 흐름으로 쭉감! → https는 인증서로 거는데 내것도 같이 끼워서 넣음<br/>
  ⇒ 즉, **https로 만들기 위해 반드시 필요한 애임!**

## SSL이 사용되는 과정을 확인해보자 🕵️‍♀️

1. 사이트(서버)는 공개키와 개인키를 만든 후, 신뢰할 수 있는 인증 기관인 CA에게 자신의 사이트 정보와 공개키를 인증 요청
2. 검증을 거친 후 사이트 정보와 공개키를 인증기관의 개인키로 암호화하여 사이트 인증서를 제작
3. 사이트로 인증서 발급
4. 인증 기관은 웹 브라우저에게 자신의 공개키를 제공하며, 브라우저에 내장됨
5. 사용자가 사이트에 접속을 요청
6. 사이트는 발급받은 인증서를 사용자에게 전달
7. 인증 기관의 공개키로 인증서 검증
8. 사이트 정보와 서버의 공개키 획득
9. 획득한 사이트 공개키로 대칭키를 암호화여 전송
10. 사이트 개인키로 해독하여 대칭키 획득
11. 안전하게 전달된 대칭키를 이용하여 암호문을 주고받음

## 오잉, 그럼 대체 왜 쓰냐!

![image](https://github.com/user-attachments/assets/2b27a5ee-0004-4310-a11f-3d59f70e4ea5)

커스텀 도메인을 쓰는 이유는 결국 아래와 같다 <Br/>
→ https 사용(걸기 위해)<br/>
→ 쿠키나 로컬스토리지 같은 애들이 같은 주소면 공유가 됨 ⇒ 원하는대로 작동이 안되는 이슈가 있을 수 있음(해당 이유라면 [http://local.hyein-zelkovaria.org](http://local.hyein-zelkovaria.org/) 로도 해결이 가능하긴함)

### 그럼 https는?

- 보통 SSL을 통해 효과가 극적으로 달라지진 않는다(중간에 하이재킹을 제외하고서는 그닥 뭐가 없다,,!)
- 그러면 왜 쓰는가? 🤔 <Br/>⇒ **`서버에서 https 아니면 cors를 안 풀어주는 경우`**
  - [local | www | dev].hyein-zelkovaria.org이렇게만 cors를 풀어줬다하면 해커는 뭐가 풀려있는지 모르기 때문이다(해커 귀찮게 만들기)

때문에 어차피 사람마다 cert 폴더 따로 만들어야해서 굳이 깃허브에 안 올리는 거라고 한다!!

## 그렇다면 로컬에서 https를 사용하는 방법은?

<< 맥 기준>>

1. mkcert를 한 번만 설치

```tsx
brew install mkcert
brew install nss # if you use Firefox
```

1.  로컬 루트 CA에 mkcert를 추가하기
    `mkcert -install`
    그러면 로컬 인증 기관 (CA)이 생성이 되는데, mkcert로 생성된 로컬 CA는 기기의 **로컬**로만 신뢰할 수 있다
2.  mkcert로 서명된 사이트 인증서를 생성하기

    터미널에서 사이트의 루트 디렉터리 또는 인증서를 보관할 디렉터리로 이동 후 실행

```
mkcert localhost
```

1. `mysite.example`과 같은 커스텀 호스트 이름을 사용하는 경우 다음을 실행

```
mkcert mysite.example
```

이 명령어는 다음 두 가지 작업을 수행하기

- 지정한 호스트 이름의 인증서를 생성
- mkcert가 인증서에 서명할 수 있도록 한다

이제 인증서가 준비되고 브라우저가 로컬로 신뢰하는 인증 기관에 의해 서명이 된다!

1.  **React 개발 서버 사용 시:**

다음과 같이 `package.json`를 수정하고 `{PATH/TO/CERTIFICATE...}`를 바꾸면 된다

```tsx
"start:ssl": "HTTPS=true SSL_CRT_FILE=./cert/localhost.pem SSL_KEY_FILE=./cert/localhost-key.pem npm run start"
```

인증서를 만든 결과는 아래처럼 뿅

![image](https://github.com/user-attachments/assets/a28a415a-a831-4981-80b4-ad24c0f5756b)

## 그렇다면 vite를 사용하면서 관련 파일은 어떻게 수정해야할까? 💦

참고로 dev용과 production용을 배포한 상태이다(develop에 머지하면 `dev.도메인`으로 배포가 되고 main에 머지하면 `도메인`으로 바로 배포가 된다)

→ 현재 진행중인 프로젝트는 실사용유저가 확보되어있는 이유로, develop에 merge 후 확인을 한 이후 main에 merge하여 더 안정적인 서비스 개발을 위해 다음과 같이 진행하였다

```tsx
// vite.config.dev.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import fs from "fs";
import path from "path";
import { fileURLToPath } from "url";

const fileName = fileURLToPath(import.meta.url);
const dirName = path.dirname(fileName);

const https = {
  key: fs.readFileSync(path.resolve(dirName, "cert/localhost-key.pem")),
  cert: fs.readFileSync(path.resolve(dirName, "cert/localhost.pem")),
};

export default defineConfig({
  plugins: [react()],
  server: {
    https,
  },
});
```

- Vite를 HTTPS로 실행하기 위해 필요한 인증서 `localhost.pem`와 개인 키 `localhost-key.pem`를 읽어온다
- `fileName`: 현재 파일의 **절대 경로**의 값을 갖는다(import.meta.url로부터 파일 시스템 경로를 갖고온다)
- `path.dirname`: 파일 경로를 갖고있는 fileName에서 **디렉토리 경로**를 반환한다
- `server: { https }`: Vite dev 서버를 https로 설정

```tsx
// tsconfig.json
"types": ["node", "vite/client"],
"include": [
    "src/**/*",
    "vite.config.dev.ts",
    "vite.config.prod.ts",
    "vite.config.ts"
  ]
}
```

- TS 컴파일러 옵션을 설정하는 동시에 Vite 설정 파일들이 타입 체크에 포함되도록 설정하였으며, https와 관련하여 설정을 수정한 일부이다
- include에서 `vite.config.dev.ts`, `vite.config.prod.ts`, `vite.config.ts` 파일들을 포함시켜 설정 파일에 오류가 없는지를 확인할 수 있게끔 하였다

```tsx
// package.json
"scripts": {
    "dev": "vite --config vite.config.dev.ts",
    "build": "tsc -b && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"
  },
```

- **“dev”**: `vite.confgi.dev.ts` 파일을 통해 https 서버를 실행할 수 있다
- **“build”**: 기본적으로 `vite.lconfig.ts`파일을 사용하여 production build를 수행한다

<br/><br/>

참고
https://blog.naver.com/skinfosec2000/222135874222<br/>
https://web.dev/articles/how-to-use-local-https?hl=ko<br/>
[https://velog.io/@ajm0718/HTTPS란-무엇인가](https://velog.io/@ajm0718/HTTPS%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)
