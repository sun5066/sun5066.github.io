---
layout: post
title: "ffmpeg, mediaRecorder, mediaCodec, VideoCapture 중 무엇을 사용해야할까?"
date: 2021-04-18 17:20:00 -0400
categories: android
tags: android, video, media, record
comments: 1
---

# [ffmpeg][ffmpeg]
```
1. 자체 SO를 구축하면 사용 사례에 따라 크기를 약 2~3MB로 줄일 수 있다. (6000중 빌드 스크립트를 편집하려면 시간과 노력이 필요하지만..)
2. 다양한 포맷 지원(거의 모든 포맷 지원)
3. 모든 장치에서 결과가 동일함
4. 모든 해상도가 지원됨
5. 디코딩으로 인한 높은 에너지 소비는 속도저하의 원인이됨, lib-stagefright를 지원하는 플러그인이 있지만, 많은 기기에서 작동하지 않는다고함
6. 라이센스에 예민함
```
---
# [MediaRecorder][MediaRecorder]
```
1. 가장 구현하기가 쉬움(mediaCodec, lib-stagefright에 대한 단순화된 액세스) 원시 데이터가 엔코더에 직접 전달되므로 문제 발생 방지
2. 대부분의 장치에서 빠르고 에너지 절약이 가능함
3. 장치마다 결과가 다를 수 있음
4. 라이센스 문제 없음
5. 전면 카메라 녹화시 좌우 반전되서 찍히는데 못 고침
(외부 라이브러리 사용해서 가능하지만, 용량이 2배가 되어버림)
```
---
# [MediaCodec][MediaCodec]
```
1. 대부분 MediaRecorder도 여기에 적용됨(사용 편의성 말고)
2. codec(encoding/decoding)에 대한 가장 유연한 액세스
3. 사용하기 까다로움
```
---
# [VideoCapture][VideoCapture]
```
1. CameraX에 종속됨
2. 간단한 코드 몇줄로 구현가능
3. MediaRecorder 와 마찬가지로 좌우반전 설정 불가
4. 장점은 CameraX의 기능을 사용할수 있음
```
>**[MediaRecorder][MediaRecorder]가 가장 구현하기 쉽고, 모든 장치에서 지원되며, 고품질/크기 옵션을 제공하기 때문에**
>**[MediaRecorder][MediaRecorder]를 사용하는걸 권장한다고합니다.**
>**[ffmpeg][ffmpeg]는 동일한 크기에 더 나은 결과를 만들지만 투자된 시간과 노력에 비해 [MediaRecorder][mediaRecorder]가 노동적으로 이득이지만 위에 설명대로 미리보기와 다르게 좌우반전되어 녹화되며, 이것을 커스텀 할순 없다. mp4로 만들어진 파일을 다시한번 수정해야 하며, 용량또한 두배로 늘어난다.(안그래도 용량 큰데 두배로 커져버림)**
>**단, 중대형 서비스에는 [ffmpeg][ffmpeg] or [MediaCodec][mediaCodec] 사용 추천**
>**세가지 다 아직 깊게는 사용 안해봤지만,**
>**[ffmpeg][ffmpeg] 사용하는게 가장 스트레스 덜 받을꺼같다.**

>**[아는 개발자][아는 개발자](위 세가지에 대해서 자세하게 설명이 되어있습니다.)**
> 
>**[할류열풍][할류열풍]([MediaCodec][mediaCodec]을 사용해야 한다면 이분 블로그 참조하시길 권장드립니다.)**

[ffmpeg]: https://github.com/bravobit/FFmpeg-Android
[MediaRecorder]: https://developer.android.com/reference/android/media/MediaRecorder
[MediaCodec]: https://developer.android.com/reference/android/media/MediaCodec
[VideoCapture]: https://developer.android.com/jetpack/androidx/releases/camera?hl=ko#camera2-core-1.0.0-alpha06
[아는 개발자]: https://selfish-developer.com/entry/MediaCodec-Getting-Started
[할류열풍]: https://m.blog.naver.com/PostView.nhn?blogId=sandyhallyu&logNo=220760330226&proxyReferer=https:%2F%2Fwww.google.com%2F
