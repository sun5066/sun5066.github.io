---
title: "Android Fastlane & Firebase Distribute Setup(Feat Windows)"
date: 2023-03-13 09:20:00 -0400 
categories: android
---

# 윈도우 환경에서 FastLane을 사용하여 Firebase Distribute 테스트 자동배포 환경 세팅하기

최신 Release 버전의 Android Studio가 설치된 가정하에, 아래 목록들을 설치한다.

- [Ruby Install](https://rubyinstaller.org/downloads/)
  - FastLane은 Ruby 기반이기 때문에 필요함
- [Node.js Install](https://nodejs.org/en/)
  - Firebase Cli 인증을 위해 Node.js가 필요함

설치가 완료되었다면, 먼저 환경변수 설정을 해줘야한다.

### 환경 변수 설정
- [FastLane 환경변수 동영상 가이드](https://www.youtube.com/watch?v=zYBYegeTNwY)

### FireBase Cli 인증
- Android Studio terminal에서 아래의 명령어를 실행
- npm install -g firebase-tools
- firebase login --interactive

### FastLane Firebase Distribute 환경 설정 및 테스트 배포
1. 관리자 권한으로 cmd 실행
2. gem install bundler
3. cd 명령어를 사용하여 fastlane 을 적용할 프로젝트 경로로 가준다.
4. bundle init
5. bundle add fastlane
6. bundle exec fastlane init
7. app package 입력
8. 플레이콘솔 서비스 파일 경로 입력
9. fastlane add_plugin firebase_app_distribution
10. Fastfile에 아래의 명령어를 추가해주자.
11. fastlane distribute

```
  desc "build_android_app"
  lane :build_android_app do
    gradle(task: "clean assembleRelease")
    gradle(
      task: "bundle",
      build_type: "Release",
      print_command: true,
      properties: {
        "android.injected.signing.store.file" => "store 인증 파일",
        "android.injected.signing.store.password" => "",
        "android.injected.signing.key.alias" => "",
        "android.injected.signing.key.password" => "",
      }
    )
  end

  platform :android do
      desc "My awesome app"
      lane :distribute do
          build_android_app()

          firebase_app_distribution(
              app: "",
              groups: "android-developers",
              android_artifact_type: "AAB",
              release_notes: "빌드 테스트"
          )

      end
  end
```

