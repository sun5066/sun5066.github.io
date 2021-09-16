---
title: "코루틴 launch vs async/await"
date: 2021-04-20 23:18:28 -0400
categories: android
---

## **launch vs async/await** 
```
웹 API 요청과 UI 변경에 사용한 async와 launch 함수를 **코틀린 빌더함 수(coroutine builder function)**라고 하며, 이 함수들은 특정 방법으로 작업을 수행하도록 코루틴을 설정합니다. launch는 우리가 지정한 작업을 올바르게 수행하는 코루틴을 빌드합니다.
launch와 다르게 async 코루틴 빌더는 지연된 작업을 나타내는 Deferred를 반환하는 코루틴을 빌드합니다. 즉, 해당 작업이 바로 시작되어 끝나는 것이 아니고, 향후 언젠가 완료되는 것이 약속된 작업을 나타냅니다.
Deferred 타입은 await 함수를 제공합니다. 이 함수는 우리가 원하는 작업 수행 시점에 호출합니다. await는 또한 지연된 작업이 완료될 때까지 다음에 할 작업을 보류합니다. Deferred는 자바의 Future와 유사한 방법으로 동작합니다.

웹 API 요청이 백그라운드에서 실행되더라도 Deferred의 await 함수가 있어서 코드 작성은 어렵지 않아요. 즉, await의 결과를 기다리는 코드 다음에 UI 변경 함수를 호출하는 로직을 구현하면 됩니다. 이것을 종전의 방법(콜백)과 비교하면 코드 작성이 훨씬 쉽습니다. ㅎㅎ
마치 동기화 방식으로 웹 API를 사용하는 것처럼 코드를 작성할 수 있기 때문입니다. 스레드의 중단 없이 코드의 실행을 보류하고 향후에 실행시킬 수 있게 코루틴이 해주기 때문입니다.
```

## **실행 보류 함수**
```
안드로이드 스튜디오는 await 함수를 호출하는 코드 줄의 왼쪽에 마우스 커서를 갖다 대보면 "Suspend function call"이라는 메시지를 보여줍니다. 이것의 의미는
종전의 스레드는 중단(block)되지만, 코루틴은 보류(suspend)된다는 뜻입니다. 또한, 코루틴은 스레드(안드로이드 UI Thread, CommonPool의 스레드)에 의해 실행됩니다. 하지만 코루틴을 실행하는 스레드를 중단시키지는 않고, 보류된 함수를 실행하는 스레드는 다른 코루틴을 실행하는 데 사용될 수 있습니다.
코루틴이 종전 스레드보다 더 좋은 이유가 이것 때문이지요.
```