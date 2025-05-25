## 개요

본 포스팅에서는 Code Rabbit을 활용하여 GitHub Pull Request에 대한 자동 리뷰를 설정하고, 팀 컨벤션에 맞춘 코드 품질 피드백을 받을 수 있도록 초기 설정 방법을 안내를 위해 작성했습니다. 내용은 아래와 같습니다.

- Code Rabbit 가입 및 레포 권한 설정
- 언어 및 리뷰 스타일 설정
- `.coderabbit.yaml` 설정으로 팀 컨벤션 반영
- 실제 PR 리뷰 화면 예시 확인

## **Code Rabbit 가입 및 권한 설정**

1. [Code Rabbit 홈페이지](https://www.coderabbit.ai/)에 가입합니다.
2. GitHub 계정으로 로그인 후, 리뷰를 적용할 **조직과 레포지토리**에 권한을 부여합니다.

   ![image.png](./img/CodeRabbit1.png)

3. 권한 설정 후, 리뷰할 레포를 선택하면 기본 설정이 완료됩니다.
4. 코드래빗 홈페이지에 들어가서 해당되는 레포의 언어를 바꿔줍니다. 0개 국어인 저는 `Korean`으로 설정했습니다.

![스크린샷 2025-05-23 오후 3.33.43.png](./img/CodeRabbit2.png)

## 리뷰 언어 및 스타일 설정

- 홈페이지가 아니더라도 코드 리뷰 언어를 팀 상황에 맞게 변경할 수 있습니다. (예: 한국어(ko-KR))
- 일부 컨벤션을 추가하고싶으면 `.coderabbit.yaml` 에서 설정할 수 있으며, [공식 문서](https://docs.coderabbit.ai/getting-started/configure-coderabbit/)에서도 확인이 가능합니다.

  ```tsx
  language: "ko-KR";

  reviews: profile: "chill";
  request_changes_workflow: false;
  high_level_summary: true;
  poem: false;
  review_status: true;
  auto_review: enabled: true;
  drafts: false;
  ```

### 설정 옵션

- 설정할 수 있는 옵션의 대표적인 예시는 아래 표와 같습니다.

| 항목                     | 설명                                                                                                                               |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| profile                  | 리뷰 스타일을 지정하는 옵션이고 chill, strict, balanced 등 있어요. 제가 설정한 "**chill**"은 코멘트를 부드럽고 친근하게 작성해줘요 |
| request_changes_workflow | true면 리뷰에서 문제를 발견했을 때 자동으로 “Changes requested” 상태로 PR에 표시할 수 있어요                                       |
| high_level_summary       | true면 리뷰 마지막에 전체 요약 코멘트를 작성해줘요                                                                                 |
| poem                     | 리뷰에 재미 요소를 생성해주는데, 보통 false 권장한다 해요                                                                          |
| review_status            | 각 파일/라인에 대해 “좋음 / 개선 필요” 등의 리뷰 상태 뱃지를 표시해줘요                                                            |
| auto_review.enabled      | PR이 생성되거나 업데이트될 때 자동으로 리뷰를 시작할지 여부를 나타내요                                                             |
| auto_review.drafts       | true이면 draft PR에도 자동 리뷰해줘요. false면 draft PR은 무시해요                                                                 |

> 저는 따닷한 말투를 원해 chill하게 반영했습니다.

![image.png](./img/chillguy.png)

## 팀 컨벤션 적용하기

- 우리 팀의 코드 리뷰를 위한 컨벤션을 적용하고싶으면 `instrtuctions`에 작성할 수 있습니다.

  ```tsx
  // 예시

  language: "ko-KR"

  reviews:
    instructions: |
      우리 팀은 다음의 코드 스타일 가이드를 따릅니다:
      - 변수는 const를 우선 사용
      - Emotion 기반 스타일링 권장
      - 커스텀 훅은 `use` 접두사와 일관된 네이밍 유지
  ```

- 참고로 `path_instructions`는 해당 파일 경로에 있는 항목들에 대해 설정한 `instructions` 내용이 반영됩니다.

## 실제 사용 예시

- 설정이 완료되면 PR을 생성하거나 업데이트할 때 자동으로 리뷰가 시작됩니다
- 전반적인 요약을 설명해주는 Walkthorugh, 변경된 내용에 대한 설명이 나오는 Changes, 전반적인 다이어그램을 그림으로도 보여줍니다.

![image.png](./img/CodeRabbit5.png)

![image.png](./img/CodeRabbit3.png)

- 리뷰 예시는 아래와 같습니다.

![image.png](./img/CodeRabbit4.png)

## 마무리

협업 과정에서 코드 리뷰가 누적되면 작업 속도가 지연되고, 전체 프로젝트 진행에도 영향을 주는 경우가 많았습니다. Code Rabbit을 도입하면 리뷰 효율을 높이고 리뷰에 대한 부담을 줄일 수 있어 팀 전체의 퍼포먼스를 더욱 안정적으로 끌어올릴 수 있었습니다. 👍🏻
