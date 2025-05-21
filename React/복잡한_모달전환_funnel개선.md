> 본 게시물에서 등장하는 ‘길드’ 키워드는 실제 디스코드의 ‘서버’와 같은 맥락입니다.
> 개발시 백엔드 팀원들의 혼선을 방지하고자 **길드(Guild)** 로 사용중임을 감안해주시면 감사하겠습니다 :)

## 💭 단계별 modal 전환 고민 과정

- 길드 생성 시 `길드 선택` 이후 `길드 커스터마이즈`로 모달의 내용이 단계별로 전환되어야 했습니다.
- 이를 위해 `currentModal` 상태를 만들어 단계별 컴포넌트를 상태값에 따라 나눠 띄우는 방식으로 구현했습니다.

```tsx
const [currentModal, setCurrentModal] = useState<CreateGuildStep | null>(null);

useEffect(() => {
  if (currentModal === null) return;

  switch (currentModal) {
    case "initial":
      handleChangeModal(<CreateGuildModal setCurrentModal={setCurrentModal} />);
      break;
    case "customize":
      handleChangeModal(
        <CustomizeGuildModal setCurrentModal={setCurrentModal} />
      );
      break;
  }
}, [currentModal]);
```

- 첫 번째 단계에서 `다음` 버튼 클릭 시 **customize**로 상태를 변경하고, 두 번째 단계에서는 `뒤로가기` 버튼 클릭 시 다시 **initial**로 변경합니다.
- 이 방식은 간단하지만 currentModal 값이 같으면 useEffect가 다시 실행되지 않아 **모달이 뜨지 않는 버그**가 발생했습니다.

```tsx
useEffect(() => {
  if (!modal.basic) {
    setCurrentModal(null);
  }
}, [modal]);
```

- 이를 통해 모달이 닫힐 때 currentModal을 초기화하여 상태 변경을 감지하게 했지만, 전반적으로 **관리 포인트가 늘어나고 흐름이 복잡해졌습니다**.

## 문제점

- 단계가 늘어날수록 `useEffect`가 복잡해지고 관리가 어려워졌습니다.
- 단계별 입력값도 **GuildList**에서 직접 관리하게 되면서 아래와 같이 **불필요한 많은 상태**를 갖게 됩니다.

```tsx
// Guild(서버) 생성에 필요한 데이터들
const [guildName, setGuildName] = useState("");
const [guildImage, setGuildImage] = useState<File | null>(null);
const [guildType, setGuildType] = useState<"private" | "public" | null>(null);
```

- 이처럼 **일부 단계에서만 필요한 값조차도 상위 컴포넌트에서 관리**하게 되며 코드가 복잡해지고 책임 분리가 모호해집니다.
- 전역 상태나 Context로 해결할 수도 있으나 모달의 종류가 많아질수록 상태 관리 범위가 넓어지고 디버깅이 어려워지는 문제가 생길 것이라 생각했습니다.

## Funnel 구조로의 전환

이러한 문제점들을 해결하기 위해 [toss slash](https://toss.im/slash-23/session-detail/A1-3)를 참고하여 **Funnel** **구조를 도입**했습니다.

### **Funnel 구조란?**

- Funnel은 사용자가 어떤 목표 지점(ex: 가입 완료)에 도달하기까지 **여러 단계를 거치는 흐름**을 의미합니다.
- 마케팅, 제품 설계 등에서 자주 사용되며, 토스에서는 이런 흐름을 손쉽게 구성할 수 있도록 useFunnel 훅을 만들었습니다.

### Funnel 구조 적용 방식

1. Guild 생성 플로우를 **하나의 CreateGuildModalContent** 컴포넌트로 묶습니다.
2. 내부에서 `useFunnel` 훅으로 각 단계의 콘텐츠를 전환합니다.

```tsx
const { Funnel, currentStep, moveToNextStep, moveToPrevStep } = useFunnel({
  defaultStep: "서버공개여부",
  stepList: ["서버공개여부", "서버커스텀"],
});

return (
  <Funnel currentStep={currentStep}>
    <Funnel.Step name="서버공개여부">
      <CreateGuildModal onNext={moveToNextStep} />
    </Funnel.Step>
    <Funnel.Step name="서버커스텀">
      <CustomizeGuildModal onPrev={moveToPrevStep} />
    </Funnel.Step>
  </Funnel>
);
```

- 각 스텝 컴포넌트는 더 이상 상태를 직접 관리할 필요가 없어지고 **onNext, onPrev만 넘겨받아 동작**하면 됩니다.
- **GuildList**에서는 단순히 아래와 같이 하나의 모달만 열면 됩니다

```tsx
const handleChangeModal = () => {
  openModal("basic", <CreateGuildModalContent />);
};
```

## 결과

- 단계별 상태와 렌더링 분기를 한 곳에 모으면서 **관리 포인트가 줄어들고** 코드를 직관적으로 바꿀 수 있었습니다.
- 상태가 퍼져 있던 기존 구조보다 단계별 흐름이 명확해지고 유지보수도 쉬워졌습니다.
- 특히 모달 안에서 사용하는 **입력값 상태**를 각 스텝 컴포넌트 내부에서만 관리할 수 있어 **범위가 명확해졌습니다.**

## **📎 참고**

> 이 구조는 [토스의 Funnel 설계 방식](https://fe-developers.kakaoent.com/2022/220731-composition-component/)을 참고했습니다.
