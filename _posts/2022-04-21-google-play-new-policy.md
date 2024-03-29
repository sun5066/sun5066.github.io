---
layout: post
title: "2022년 Google Play 정책 바이트"
date: 2022-04-21 17:20:00 +0900
categories: policy
tags: android google policy
comments: 1
---

이 글은 YouTube `Android Developers` 채널에 올라온 `Google Play 정책 바이트 - 2022년 4월 정책 업데이트` 영상을 정리한 내용입니다.

## 대상 API 수준

모든 신규 앱과 앱 업데이트는 최신 주요 Android OS 버전으로부터 1년 이내의 Android API 수준을 타겟팅해야 합니다.
- 최신의 주요 Android OS 버전으로부터 2년 이내의 API 수준을 타겟팅하도록 업데이트 되지 않은 앱은 신규 Android OS 버전의 기기를 사용하는 새로운 사용자에게 더 이상 표시되거나 검색되지 않을 것입니다.
- 제한이 적용되는 날짜는 2022년 11월 중이며, 기간이 더 필요한 경우 `Play Console`에서 6개월 기간 연장이 가능하다.
    
## 패키지 설치 요청

패키지 설치 요청 권한은 애플리케이션이 앱 패키지 설치를 요청하도록 허용합니다. 이 권한을 사용하려면 앱의 핵심 기능에 다음이 포함되어야 합니다.
- 앱 패키지 전송 또는 수신
- 사용자에 의한 앱 패키지 설치 지원

허용되는 기능들의 예시는 다음과 같습니다.
- 웹 브라우징 또는 검색
- 첨부파일을 지원하는 커뮤니케이션 서비스
- 파일 공유, 전송, 관리
- 엔터프라이즈 기기 관리

## 정책 업데이트

- 뉴스
  - 이제 '뉴스 및 잡지' 카테고리에 표시되어 있으며 앱 등록정보에 뉴스라고 설명된 앱이 뉴스 정책의 범위에 포함됩니다.
  - 뉴스 앱에 연락처 정보를 기재하는 방식이 두 가지가 아닌 한 가지만 허용됩니다.
  - 최초 게시자가 모든 기사에 저자를 기재하지 않아도 됩니다.
- 사용자 제작 콘텐츠
  - 우발적인 `성적 콘텐츠`를 허용하는 개발자는 다음 요구사항을 충족해야 합니다.
    - 이러한 콘텐츠는 기본적으로 숨겨져야 합니다.
    - 아동의 앱 액세스는 금지되어야 합니다.
    - 콘텐츠 등급 설문지를 정확하게 작성해야합니다.
- 접근성
  - 사용자를 보호하고 추가적인 보안을 강화하기 위해 `Accessibility` API는 원격 통화 오디오 녹음을 위해 만들어지지 않았으며, 이 목적으로 해당 API를 요청할 수 없습니다. 또한 해당 API를 사용하려면 앱 등록 정보에 앱이 접근성 도구임을 공개적으로 선언 해야합니다.
    
## 데이터 보안 섹션
  - 2022년 4월 말: Google Play 스토어에서 이 기능이 사용자에게 표시됩니다.
    - 정보가 아직 승인되지 않은 경우 `관련 정보가 없습니다.` 와 같은 형태로 표시됩니다.
  - 2022년 7월 20일: 양식 제출 및 개인정보처리방침 승인 기한입니다.
    - 7월 20일 부터 모든 신규앱 제출 및 앱 업데이트를 위해서는 데이터 보안 양식을 작성하여 제출해야한다.
    
- 알림
  - 모든 앱은 개인정보처리방침을 제공해야 합니다.
  - SDK와 서드 파티 라이브러리에서 데이터를 수집 또는 처리하는지 확인하시기 바랍니다.
  - 오프라인으로 양식을 작성하거나, 여러앱의 양식을 작성하기 위해 CSV 파일 기능을 사용할 수 있습니다. [양식 보러가기](https://support.google.com/googleplay/android-developer/answer/10787469?hl=ko)
    
## 기타 업데이트

위 내용들 외에 기타 업데이트가 있으므로 밑의 링크에 자세한 내용이 있습니다.

[업데이트 내용 보러가기](https://goo.gle/playupdates)
  - 어린이 및 가족
  - 금융 서비스
  - Android 이모티콘
  - 증오심 표현
  - 사기 행위: 왜곡된 주장

## 2022년 정책 시행

- 5월
  - `K&F: 인증 광고 SDK 요구사항`
- 7월
  - `패키지 설치 요청`
  - `데이터 보안 섹션`
- 8월
  - `뉴스`
- 10월
  - `사용자 제작 콘텐츠`
- 11월
  - `대상 API 수준`(2023년 5월로 선택적 연장 가능)
    
## Google의 무료 정책 과정

구글에서 정책과정 교육을 무료로 제공합니다.

[무료 정책 과정 보러가기](https://g.co/playacademy/policy)

# 참조사항

- [공식 Developers 유튜브 영상](https://www.youtube.com/watch?v=p9c9EpJljWM)