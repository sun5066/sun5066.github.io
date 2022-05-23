---
title: "안드로이드 아고라 SDK 공식문서 파헤치기-1"
date: 2022-05-16 17:20:00 +0900
categories: agora
---

해당 글은 android agora-sdk-v3.7.0 기준으로 작성되었습니다.

- [RtcEngine](https://docs.agora.io/en/Voice/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html) 애플리케이션에서 호출할 수 있는 기본 기능을 제공.
- [IRtcEngineEventHandler](https://docs.agora.io/en/Voice/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_i_rtc_engine_event_handler.html) 애플리케이션에서 `RtcEngine`의 기능이 동작의 콜백을 제공.
- [RtcChannel](https://docs.agora.io/en/Voice/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_channel.html) 지정된 채널에서 실시간 통신을 가능하게 하는 방법을 제공한다. 해당 인스턴스 하나당 한개의 채널에 가입이 가능하며, n개 만큼의 인스턴스 생성시 n개 만큼의 채널에 가입할 수 있다.
- [IRtcChannelEventHandler](https://docs.agora.io/en/Voice/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_i_rtc_channel_event_handler.html) 현재 채널의 이벤트와 통계의 콜백을 제공.
- [IAudioEffectManager](https://docs.agora.io/en/Voice/API%20Reference/java/interfaceio_1_1agora_1_1rtc_1_1_i_audio_effect_manager.html) 오디오 효과 파일을 관리하는 기능을 제공.
- [AgoraMediaRecorder](https://docs.agora.io/en/Voice/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_agora_media_recorder.html) 로컬 오디오/비디오를 녹화하는 기능 및 콜백을 제공.

