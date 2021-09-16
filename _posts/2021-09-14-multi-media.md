---
title: "안드로이드에서 영상 녹화&편집까지 해보자"
date: 2021-09-14 09:43:00 -0400
categories: android-yeoboya
---

---
## 사용된 공통 라이브러리
- [AAC]
- [Coroutine]

### VideoTrim (구간편집/업로드 모듈)
- [ExoPlayer2]
- [Retrofit2]
- [OkHttp3]
- [FFMPEG]

### Camera Record (영상 녹화 모듈)
- [CameraRecorder-Android]
- [mp4parser-muxer]

---

# 설명하기 앞서서 말씀드리자면..
우선 해당 라이브러리는 모든 Activity/Fragment는 **com.lib.multiimagechooser.MediaDialogFragment.kt** 에서 LifeCycle, BackPress를 제어하고 있기 때문에 잘 기억해놓으셔야합니다.

[AAC]: https://developer.android.com/topic/libraries/architecture?hl=ko
[Coroutine]: https://github.com/Kotlin/kotlinx.coroutines
[ExoPlayer2]: https://github.com/google/ExoPlayer
[Retrofit2]: https://github.com/square/retrofit
[OkHttp3]: https://github.com/square/okhttp
[FFMPEG]: https://github.com/tanersener/mobile-ffmpeg
[mp4parser-muxer]: https://github.com/sannies/mp4parser
[CameraRecorder-Android]: https://github.com/MasayukiSuda/CameraRecorder-android