# 도입 배경

채팅과 관련한 서비스를 멀티플랫폼으로 확장하는 과정에서 웹보다는 데스크탑 앱이 더 부드러운 사용자 경험을 제공할 수 있다는 결론에 도달했습니다.

특히 넓은 화면에서 여러 채팅방을 동시에 띄워 대화하는 등 멀티창 기반 interaction이 핵심 경험이라 생각되어 **Electron**을 통해 독립된 창을 효율적으로 관리할 수 있는 환경을 구축하기로 결정했습니다.

무엇보다 기존 서비스는 앱 전용 플랫폼으로만 제공되고 있었고, 직장인이나 노트북 사용자들 사이에서 “데스크탑 버전이 있었으면 좋겠다”는 요청이 꾸준히 늘고 있었습니다.

이러한 배경이 **Electron 도입의 직접적인 계기**가 되었습니다.

# Electron 프로젝트 구조 이해하기

Electron 프로젝트는 크게 세 가지 파일로 구성됩니다.

- `main.js`: 애플리케이션의 진입점으로 앱의 생명주기와 창 생성 등을 담당하는 **메인 프로세스**
- `preload.js`: 메인(Node.js)과 render(브라우저) 간의 브리지 역할
- `renderer`: 실제 사용자 인터페이스(UI)를 렌더링

## main.js - 앱의 진입점

package.json의 `main`필드가 가리키는 파일이 곧 Electron 앱의 **메인 프로세스 진입점**입니다. 이곳에서 앱 실행, 창 생성, 이벤트 관리 등 전반적인 동작을 제어합니다.

> The [**`main`**](https://docs.npmjs.com/cli/v7/configuring-npm/package-json#main) script you defined in package.json is the entry point of any Electron application. This script controls the **main process**, which runs in a Node.js environment and is responsible for controlling your app's lifecycle, displaying native interfaces, performing privileged operations, and managing renderer processes (more on that later).

공식 문서에서도 위와 같이 정의하고 있습니다.

## preload.js - 보안 브리지

Electron은 보안상의 이유로 메인 프로세스(Node.js)와 render(웹)가 분리되어 있습니다.

renderer는 Node.js API에 직접적으로 접근할 수 없기 때문에 **preload 스크립트**를 통해 필요한 기능만 노출합니다.

> To bridge Electron's different process types together, we will need to use a special script called a **preload**.

### 작성 예시

`process.version`를 전역 변수로 노출시켜주고, 렌더러가 사용할 Electron API를 등록하는 예시입니다.

```jsx
const { contextBridge, ipcRenderer } = require("electron");

contextBridge.exposeInMainWorld("versions", {
  node: () => process.versions.node,
  chrome: () => process.versions.chrome,
  electron: () => process.versions.electron,
});

contextBridge.exposeInMainWorld("**electronAPI**", {
  createMainWindow: () => ipcRenderer.invoke("create-main-window"),
  openChatWindow: (chatId, chatbotName) =>
    ipcRenderer.invoke("open-chat-window", chatId, chatbotName),
});
```

> 💡 `contextIsolation: true` 설정을 활성화하면 렌더러의 전역 스코프와 분리되어 보안이 강화됩니다. 이때는 반드시 contextBridge를 통해 필요한 기능만 노출해줘야 합니다

# **BrowserWindow 생성하기**

Electron에서 새 창을 생성할 때는 BrowserWindow 객체를 사용합니다.

기본적인 사용 예시는 다음과 같습니다.

```jsx
const { BrowserWindow } = require("electron");

const win = new BrowserWindow({ width: 800, height: 600 });

win.loadURL("https://github.com");

win.loadFile("index.html");
```

프로덕션 환경에서는 경로 설정, preload 연결, 보안 설정 등을 함께 지정해줘야 합니다.

```jsx
function createWindow() {
  const __filename = fileURLToPath(import.meta.url);
  const __dirname = path.dirname(__filename);

  win = new BrowserWindow({
    width: windowWidth,
    height: windowHeight,
    icon: path.join(__dirname, "../public/icons/Home.png"),
    resizable: true,
    webPreferences: {
      preload:
        process.env.NODE_ENV === "development"
          ? path.join(__dirname, "../electron/preload.cjs")
          : path.join(__dirname, "preload.cjs"),
      devTools: true,
      nodeIntegration: false,
      contextIsolation: true,
      webSecurity: true,
    },
  });

  win.loadFile("index.html");
}
```

> 💡 `nodeIntegration: false`로 설정하면 renderer는 Node.js API에 직접 접근이 불가능해집니다. 대신 preload를 통해 필요한 기능만 안전하게 연결할 수 있습니다.

# 실행 흐름

Electron 앱이 실행되는 흐름은 다음과 같습니다.

1. main.js에서 윈도우 생성 시 `webPreferences.preload` 옵션을 통해 preload 스크립트 경로를 지정
2. 브라우저 창이 초기화 되면서 렌더러 환경이 준비되지만 HTML 로드 이전에 preload.js가 먼저 실행
3. preload 내부에서 `contextBridge.exposeInMainWorld` 등을 통해 renderer가 사용할 수 있는 함수나 객체를 정의
4. render에서는 window.electronAPI를 통해 메인 프로세스 기능 호출 가능
5. 메인 프로세스는 ipcMain.handle() 또는 IpcMain.on()등으로 요청을 처리

# IPC 통신으로 데이터 주고받기

메이 프로세스와 renderer간 통신은 **IPC(Inter-Process Communication)**으로 이루어집니다.

이 방식은 요청-응답 구조 또는 이벤트 기반 구조로 사용할 수 있습니다.

| **항목**       | ipcMain.on                                | ipcMain.handle                              |
| -------------- | ----------------------------------------- | ------------------------------------------- |
| 사용 패턴      | 이벤트(event) 기반                        | 요청-응답(request-response) 기반 (Promise)  |
| 렌더러 측 함수 | ipcRenderer.send() / ipcRenderer.on()     | ipcRenderer.invoke() / ipcRenderer.handle() |
| 응답 방식      | 수동으로 event.reply() 해야 함            | 자동으로 Promise resolve/reject 반환        |
| 비동기 처리    | 직접 구현해야 함                          | 기본적으로 async/await 기반                 |
| 주 용도        | 지속적인 스트림, 상태 알림, 단방향 메시지 | 데이터 요청, 함수 호출, 단일 응답 패턴      |
| 예시           | 실시간 알림, progress 이벤트              | 설정 값 요청, 파일 읽기 등                  |

# Electron에서 자주 쓰이는 이벤트

Electron 앱의 생명주기 이벤트는 app 객체를 통해 제어합니다.

```jsx
const { app } = require("electron");

app.on("ready", () => {
  createWindow();
});

app.on("window-all-closed", () => {
  if (process.platform !== "darwin") app.quit();
});
```

- `app.on(’ready’)`: 초기화가 완료되면 창을 생성합니다.
- `window-all-closed`: 모든 창이 닫히면 앱을 종료합니다.
- `activate`: macOS에서 Dock 아이콘 클릭 시 다시 창을 생성할 때 사용됩니다.

# 빌드 및 배포하기

배포 시에는 `electron-builder`와 같은 툴을 사용해 OS별 설치 파일을 생성합니다.

아래는 예시 설정입니다.

```jsx
"scripts": {
  "start": "electron .",
  "dist": "electron-builder"
},
"build": {
  "appId": "com.example.app",
  "productName": "AIChat",
  "directories": {
    "output": "dist"
  },
  "files": ["dist/**/*"]
}
```

# 마무리 및 도입 후기

Electron을 도입한 이후, 브라우저 탭 여러 개를 띄우며 느리게 작업하던 부분을 **하나의 독립된 앱 환경으로 통합**할 수 있었습니다.

덕분에 사용자들은 동시에 여러 채팅을 자연스럽게 대화할 수 있게 되었고, 베타 테스트를 통한 피드백에서 서비스 완성도와 몰입감이 향상됨을 확인할 수 있었습니다.

Electron은 웹 기술 기반으로 개발할 수 있다는 강력한 장점과 동시에 네이티브 수준의 UX를 구현할 수 있다는 점에서 하이브리드 서비스에 매우 유용한 선택이었습니다 👍🏻

# 참고 문헌

https://www.electronjs.org/docs/latest

https://www.electronjs.org/docs/latest/api/browser-window

https://www.electronjs.org/docs/latest/tutorial/security
