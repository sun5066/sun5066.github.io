---
layout: post
title: "Android init Process에 대해서(짧게) 알아보자"
date: 2023-07-27 09:20:00 -0400 
categories: android
tags: android init-process
comments: 1
---

### init Process 란?

안드로이드 init 프로세스는 안드로이드 운영체제의 초기화 단계에서 실행되는 무려 첫 번째 프로세스이다.

init 프로세스는 리눅스 기반 시스템의 init 프로세스와 비슷한 역할을 수행하지만, 안드로이드의 특정한 기능과 운영체제 초기화를 담당하는 데 있어서 고유한 역할을 한다.

- 부팅 시 초기화: 안드로이드 디바이스가 부팅될 때 init 프로세스가 가장 먼저 실행된다.(닉값) init 프로세스는 기본적인 시스템 리소스를 초기화하고, 필요한 서비스와 데몬 프로세스를 실행한다.
- init.rc 파일 로드: init 프로세스는 시스템 설정과 서비스 시작에 대한 정보를 담고 있는 init.rc라는 특별한 스크립트 파일을 읽는다. 이 init.rc 파일에는 안드로이드 시스템 구성, 서비스, 권한, 소켓, 디렉토리 및 파일 권한 등의 정보가 기록되어있다.
- 서비스 시작: init.rc 파일에 정의된 서비스들을 실행한다. 이러한 서비스들은 안드로이드 시스템의 핵심 기능과 백그라운드 데몬 등을 포함한다.(Zygote도 여기에 포함됨)
- 속성 설정: init 프로세스는 안드로이드 시스템에서 사용되는 속성(property)을 설정하고, 이를 다른 프로세스들이 참조할 수 있도록 한다. 이 속성들은 안드로이드 시스템의 설정 값이나 환경 변수와 관련되며, 프로세스 간의 통신에 사용된다.
- 셀 매칭 및 스크립트 실행: init.rc 파일에는 조건문과 스크립트가 포함되어 있어, 특정 조건에 따라 다른 동작을 수행할 수 있다. 예를 들어, 특정 디바이스 모델에 따라 서비스를 조건부로 실행하거나, 시스템 속성에 따라 환경 설정을 변경하는 등의 작업이 가능하다.
- 다른 프로세스 실행: init 프로세스가 서비스를 실행하고 리소스를 초기화한 후에는 리눅스 시스템과 마찬가지로 다른 프로세스들을 생성하고, 안드로이드 시스템이 정상적으로 동작할 수 있도록 한다.

> *.rc 파일이란? 리눅스와 안드로이드에서 사용되는 설정 파일의 일종이다. "rc"는 "Run Command"의 약어로, 실행 시점에 사용되는 초기화 스크립트 파일을 의미한다.
> 
> init.rc 외에도 ueventd.rc, service.rc 등이 존재한다.

안드로이드의 init 프로세스는 시스템의 핵심이며, 안드로이드 운영체제를 초기화하고 안정적으로 동작하도록 하는 역할을 담당한다. init 프로세스가 아플경우 부팅이 안될 수 있다고 한다.

Chat GPT를 통해 짧게 알아봤으며, 더 깊은 내용이 필요하다면 [붉은 코딱지님 블로그](https://m.blog.naver.com/bl2019/10186383283)를 참고하는걸 추천드리고 싶다.