# 서론: 목표와 전체 아키텍처

> 본 프로젝트는 Electron 환경에서 웹사이트(React 기반)를 직접 loadURL()하는 구조입니다. 즉, 웹 관련 변경 사항은 웹 배포만으로도 바로 반영되지만, 데스크탑 앱 자체의 배포는 별개로 관리해야 했습니다.

**문제**는 다음과 같았습니다.

매번 `공증 → 산출물 빌드 → 스토리지(S3, R2)에 업로드 → 앱 반영` 과 같은 과정을 수동으로 처리해야 했고, 이는 시간 효율을 저하시킬 뿐더러 사람이 반복할수록 실수할 가능성이 높아지는 위험이 있었습니다.

이를 개선하기 위해, 아래와 같은 **CI 파이프라인을 구축해 전면 자동화했습니다.**
**`빌드·서명·공증·스테이플 → 아티팩트 업로드 → 앱에서 백그라운드 다운로드 & 재시작 시 교체`**

### 전체 아키텍처 다이어그램

![image.png](../img/electron_down_arch.png)

결과적으로 **한 번의 태그 푸시(tag push)**로 앱의 공증, 스테이플, 아티팩트 업로드, 자동 업데이트까지 프로덕션/개발용으로 나누어 이어집니다.

추가로 데스크탑 앱 업데이트를 위해서는 `latest-mac.yml`, `*.zip`, `*.zip.blockmap`조합이 필요한데 용량 문제로 중간에 Cloudflare의 R2를 사용한 파일 배포를 포함했습니다.

이외에도 유저의 환경에서의 작업을 확인하기 위해 초기 테스트 배포용/다운로드용 의 채널과 tag를 나누어서, 특정 채널별 tag를 push한 경우에만 업데이트 되는 구조로 설계했습니다.

# 사전 준비(macOS 기준)

## 공증(Notarization) 관련 필수 정보

macOS에서 Electron 앱을 자동 배포하려면 다음 값이 필요합니다.

- Apple ID 및 Apple Team ID
- Developer ID Application 인증서 (.p12)
- App Specific Password
- Hardened Runtime + entitlements 설정(보안 정책상 필수로, 없는 경우에는 공증이 거절됨)
- appId 고정 필요(macOS 식별자이므로 배포 후에는 변경하면 X)

# electron-builder 설정 핵심

```jsx
// package.json
"build": {
    "publish": [
      {
        "provider": "generic",
        "url": "R2 Public URL/production"
      }
    ],
  ...
}

// electron-beta.json
{
	"publish": [
    {
      "provider": "generic",
      "url": "R2 Public URL/beta",
      "channel": "beta"
    }
  ]
}
```

generic provider를 사용하면 publish.url이 앱의 업데이트 파일(latest-mac.yml, _.zip, _.blockmap)이 배포되는 베이스 경로가 됩니다.

추가로 예시 코드처럼 beta와 production을 분리하면 CI에서 채널별 업로드를 간단히 제어할 수 있습니다.

> ⚠️ latest-mac.yml은 자동 업데이트의 핵심 메타데이터 파일입니다. 해당 파일의 URL 경로와 앱 설정이 일치하지 않으면 업데이트가 동작하지 않으므로 확인을 꼭 해주세요!

# Cloudflare R2를 선택한 이유

처음에는 Vercel을 통해 배포를 관리했으나, 직접 테스트했을 때 업데이트 시 필요한 파일들의 용량이 **150MB 이상이면 502 에러**가 자주 발생했습니다.

GitHub Releases는 private 레포 접근 이슈와 토큰 노출 위험이 있어 제외하고, 비용 효율성과 운영 편의성을 고려해 Cloudflare R2를 최종적으로 선택했습니다.

R2는 AWS S3호환 명령어로 업로드가 가능하며, **300MB 이상**의 파일은 Cloudflare CLI보다 S3 호환 AWS CLI가 더 안정적이었습니다.

# 앱 내 자동 업데이트 코드

Electron 앱에서는 electron-updater를 통해 자동 업데이트를 구현할 수 있습니다.

```jsx
import { app, dialog } from "electron";
import { autoUpdater } from "electron-updater";
import log from "electron-log";

autoUpdater.logger = log;
log.transports.console.level = "info";
log.transports.file.level = "info";

// 베타 채널 허용 여부(버전 접미사 기준)
autoUpdater.allowPrerelease = app.getVersion().includes("-beta");

autoUpdater.autoDownload = true; // 백그라운드 다운로드
autoUpdater.autoInstallOnAppQuit = true; // 종료/재시작 시 자동 설치

app.whenReady().then(() => {
  if (app.isPackaged) {
    log.info("업데이트 확인 시작");
    autoUpdater.checkForUpdates();
    setInterval(() => autoUpdater.checkForUpdates(), 2 * 60 * 60 * 1000);
    // 2시간마다 업데이트 감지
  }
});

autoUpdater.on("checking-for-update", () => log.info("업데이트 확인 중"));
autoUpdater.on("update-available", (info) =>
  log.info(`업데이트 발견: ${info.version}`)
);
autoUpdater.on("update-not-available", () => log.info("현재 최신 버전"));
autoUpdater.on("error", (err) => log.error("업데이트 오류:", err));
autoUpdater.on("update-downloaded", (info) => {
  log.info(`업데이트 다운로드 완료: ${info.version}`);
  const choice = dialog.showMessageBoxSync({
    type: "question",
    buttons: ["지금 재시작", "나중에"],
    defaultId: 0,
    message: "업데이트가 준비되었습니다. 지금 재시작할까요?",
  });
  if (choice === 0) autoUpdater.quitAndInstall();
});
```

checkForUpdates() + dialog를 활용하면 사용자 경험을 직접 통제할 수 있습니다. 베타/프로덕션 채널을 구분하려면 `allowPrerelease` 옵션을 함께 사용이 가능합니다.

# CI/CD 구현 코드 (GitHub Actions)

아래 워크플로우는 **`공증 + 스테이플 + 빌드 + 업로드`**를 모두 자동화합니다.

태그를 푸시하면(v0.1.0, v0.1.0-beta.1 등) 해당 채널로 자동 배포됩니다.

```jsx
# .github/workflows/ci.yml
name: Build, Notarize and Publish (macOS)

on:
  push:
    tags:
      - 'v*'
      - 'v*-beta*'

jobs:
  mac-build:
    runs-on: macos-latest
    permissions:
      contents: write
    env:
      APPLE_ID: ${{ secrets.APPLE_ID }}
      APPLE_APP_SPECIFIC_PASSWORD: ${{ secrets.APPLE_APP_SPECIFIC_PASSWORD }}
      APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: yarn install --frozen-lockfile
      - run: yarn build:prod

      - name: Setup certificates
        env:
          BUILD_CERTIFICATE_BASE64: ${{ secrets.BUILD_CERTIFICATE_BASE64 }}
          P12_PASSWORD: ${{ secrets.P12_PASSWORD }}
        run: |
          echo "$BUILD_CERTIFICATE_BASE64" | base64 --decode > certificate.p12
          security create-keychain -p "" build.keychain
          security import certificate.p12 -k build.keychain -P "$P12_PASSWORD" -T /usr/bin/codesign
          security unlock-keychain -p "" build.keychain
          security set-key-partition-list -S apple-tool:,apple: -s -k "" build.keychain

      - name: Build & publish (beta)
        if: contains(github.ref_name, '-beta')
        env: { GH_TOKEN: ${{ secrets.GITHUB_TOKEN }} }
        run: npx electron-builder --config electron-builder.beta.json --mac dmg zip --publish always

      - name: Build (stable, no publish)
        if: startsWith(github.ref_name, 'v') && !contains(github.ref_name, '-beta')
        env: { GH_TOKEN: ${{ secrets.GITHUB_TOKEN }} }
        run: npx electron-builder --mac dmg zip --publish never

      - name: Install AWS CLI
        run: brew install awscli

      - name: Upload artifacts to R2 (beta)
        if: contains(github.ref_name, '-beta')
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
          R2_ENDPOINT: ${{ secrets.R2_ENDPOINT }}
          R2_BUCKET: ${{ secrets.R2_BUCKET }}
        run: |
          ZIP=release-beta/App-Beta.zip
          MAP=release-beta/App-Beta.zip.blockmap
          YML=release-beta/beta-mac.yml

          aws s3 cp "$ZIP" "s3://$R2_BUCKET/beta/App-Beta.zip" \
            --endpoint-url="$R2_ENDPOINT" --content-type "application/zip" --acl public-read
          aws s3 cp "$MAP" "s3://$R2_BUCKET/beta/App-Beta.zip.blockmap" \
            --endpoint-url="$R2_ENDPOINT" --content-type "application/octet-stream" --acl public-read
          aws s3 cp "$YML" "s3://$R2_BUCKET/beta/beta-mac.yml" \
            --endpoint-url="$R2_ENDPOINT" --content-type "text/yaml" \
            --cache-control "max-age=0, no-cache" --acl public-read

      - name: Upload artifacts to R2 (stable)
        if: startsWith(github.ref_name, 'v') && !contains(github.ref_name, '-beta')
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.R2_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.R2_SECRET_ACCESS_KEY }}
          R2_ENDPOINT: ${{ secrets.R2_ENDPOINT }}
          R2_BUCKET: ${{ secrets.R2_BUCKET }}
        run: |
          ZIP=release/App.zip
          MAP=release/App.zip.blockmap
          YML=release/latest-mac.yml

          aws s3 cp "$ZIP" "s3://$R2_BUCKET/production/App.zip" \
            --endpoint-url="$R2_ENDPOINT" --content-type "application/zip" --acl public-read
          aws s3 cp "$MAP" "s3://$R2_BUCKET/production/App.zip.blockmap" \
            --endpoint-url="$R2_ENDPOINT" --content-type "application/octet-stream" --acl public-read
          aws s3 cp "$YML" "s3://$R2_BUCKET/production/latest-mac.yml" \
            --endpoint-url="$R2_ENDPOINT" --content-type "text/yaml" \
            --cache-control "max-age=0, no-cache" --acl public-read
```

```jsx
S3/R2 구조 예시
/beta/      → beta-mac.yml, App-Beta.zip, App-Beta.zip.blockmap
/production/    → latest-mac.yml, App.zip, App.zip.blockmap
/downloads/         → App.dmg (최초 설치용)
```

# 결과 및 회고

해당 파이프라인을 구축한 이후, 더 이상 사람이 직접 공증과 배포를 반복할 필요가 없어졌으며, 평균적으로 약 8분에서 3분으로 약 62.5% 개선할 수 있었습니다.

태그를 채널에 맞게 푸시하면 CI가 알아서 **`빌드 -> 공증 -> 스테이플 -> 업로드 -> 앱 자동 업데이트 반영`** 까지 전 과정을 수행합니다.

테스트 채널과 프로덕션 채널을 분리하면서 베타용 테스트 버전 피드백도 빠르게 수집할 수 있었고 배포 실수 또한 줄일 수 있었습니다.

CI/CD로 공증,스테이플부터 업로드까지 자동화한 덕분에 이제는 **태그 한 번 푸시로 배포 전체가 끝나는** **환경**을 만들 수 있었습니다.

# 참고 문헌

https://www.electronjs.org/docs/latest/api/auto-updater

https://www.electron.build/

https://developers.cloudflare.com/r2/

https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution
