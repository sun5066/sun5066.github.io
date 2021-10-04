---
title: "안드로이드 코루틴 기초 요약본"
date: 2021-10-04 17:20:00 -0400
categories: android
---

# 코루틴
```
1. 경량(가벼운) 스레드, suspend 지원으로 단일 스레드에서 많은 코루틴을 실행할수 있음
2. android developer 의 권장과 짧은 러닝타임이 장점
3. rxJava와 유사한 flow 지원 
```

# Coroutine Builder
```
기본적으로 suspend 함수에서 실행되어야합니다.

1. launch: Job 을 반환하며, 비동기 작업의 완료를 기다리거나, 중단 가능
2. async/await: Deferred<T> 를 반환하며, 반환할때까지 기다리거나, 중단가능
3. runBlocking: 안드로이드에선 사용 X
4. withContext: async/await 처럼 사용가능하지만,
    차이점은 <T> 타입으로 코드블록 가장 마지막줄의 타입으로 반환됨
```

# Coroutine Scope
```
1. CoroutineContext를 정의
2. Coroutine의 LifeCycle을 관리
3. viewModelScope, lifeCycleScope, mainScope 등 coroutine-android 라이브러리에서 지원되는 scope가 있음
4. 기본적으로 Dispatcher.Main 으로 실행되며, 
    내부 코루틴인 경우 디스패처를 안달아줄경우 부모 코루틴의 디스패처를 따라간다.
```

# GlobalScope
```
(사용은 안해보고 이론만 공부했습니다.)

1. 사용 권장X
2. 보통 앱이 실행되고 종료될때까지 백그라운드로 실행되는 작업에 사용함
3. 관리하는게 보통이 아니라고한다.
```

# Coroutine Dispatcher
```
설정된 디스패처에 따라 어떤 스레드에서 실행될지 결정되며, 
Android Develop에서는 Coroutine Builder에 하드코딩하지 말라고 나와있음

1. Default: CPU를 많이 사용하는 스레드에서 실행되며 리스트의 정렬, JSON 파싱등 에서 효율적이다.
2. Main: 메인스레드(UI 스레드)에서 실행되며 UI 업데이트, LiveData 업데이트를 위해서만 사용해야함
3. IO: 디스크, 네트워크 작업에 최적화 된 스레드에서 실행되며, 당연히 메모리를 많이 사용하는 스레드
4. Unconfined: suspend 후 다시 실행되었을때 생각지도 못한 스레드에서 실행될수 있기 때문에 권장 X
```

# Channel
```
(채널은 레퍼런스가 꽤나 많기 때문에 최대한 간략하게 써봅니다..;;)

1. 채널은 데이터의 스트림(데이터의 흐름)을 반환
2. 여러 코루틴에서 원자성을 보장하기 위해 사용함
3. Iterable 패턴을 사용하여 FIFO 순서가 보장됨
4. capacity 이라는 채널의 buffer size 를 정할수있는 옵셔널을 제공하며 RENDEZVOUS가 기본 값이다. 
    4-1. UNLIMITED: 버퍼 제한이 Int.MAX_VALUE
    4-2. RENDEZVOUS: 버퍼가 없는 상태
    4-3. CONFLATED: 항상 최신의 값 하나만 가지고 있으며, 새로운 데이터가 들어오면 이전 데이터는 소실된다.
    4-4. BUFFERED: 시스템이 정한 버퍼값(64)를 가지고 있음
```

# Mutex
```
레이스 컨디션이 발생하지 않도록 데이터의 원자성을 보장
```

# Flow
```
RxJava 와 유사하며 데이터 비동기로 Stream(생산자, 중개자, 소비자)을 관리할수 있으며,
마찬가지로 Cold Flow, Hot Flow가 지원됨
```


## 후기
```
actor, produce 등 설명 안나온것들이 있지만,
필요할때 공부해서 업데이트 할 예정
```

## 참고
- [two22 님 블로그](https://two22.tistory.com/23)
- [공식 홈페이지](https://developer.android.com/kotlin/coroutines/coroutines-adv?hl=ko)