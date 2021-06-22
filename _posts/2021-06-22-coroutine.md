---
title: "코틀린 코루틴 동시성 프로그래밍"
date: 2021-06-22 17:20:00 -0400
categories: android
---

# 프로세스, 스레드, 코루틴
애플리케이션을 시작할 때 운영체제는 프로세스를 생성하고 여기에 스레드를 연결한 다음, 메인스레드로 알려진 해당 스레드를 시작한다. 프로세스, 스레드 및 코루틴 간의 관계를 자세히 설명할 텐데 동시성을 이해하고 구현하는 데 반드시 알아둬야 할 요소들이다.

## 프로세스
프로세스는 실행 중인 애플리케이션의 인스턴스다. 애플리케이션이 시작될 때마다 애플리케이션의 프로세스가 시작된다. 프로세스는 상태를 갖고 있다. 리소스는 여는 핸들, 프로세스 ID, 데이터, 네트워크 연결 등은 프로세스 상태의 일부이며 해당 프로세스 내부의 스레드가 액세스를 할 수 있다.

애플리케이션은 여러 프로세스로 구성될 수 있다. 인터넷 브라우저같은 경우도 여러 프로세스로 구성된다. 그러나 다중 프로세스 애플리케이션을 구현하는 데는 이 책의 범위를 벗어나는 다양한 문제가 있다. 단일 프로세스에서 실행되지만 하나 이상의 스레드를 실행하는 애플리케이션의 구현에 한해서만 다룬다.

## 스레드
실행 스레드는 프로세스가 실행할 일련의 명령을 포함한다. 따라서 프로세스는 최소한 하나의 스레드를 포함하며 이 스레드는 애플리케이션의 진입점을 실행하기 위해 생성된다. 보통 진입점은 애플리케이션의 main() 함수이며 메인 스레드가 하는데 프로세스의 라이프 사이클과 밀접하게 연관된다. 스레드가 끝나면 프로세스의 다른 스레드와 상관없이 프로세스가 종료된다. 그 예를 살펴보자.

```
fun main(args: Array<String>) {
    doWork()
}
```
기본적인 애플리케이션이 실행되면 main() 함수의 명령 집합이 포함된 메인 스레드가 생성된다. doWork()은 메인 스레드에서 실행되므로 doWork()이 종료되면 애플리케이션의 실행이 종료된다.

각 스레드는 스레드가 속한. 프로세스에 포함된 리소스를 액세스하고 수정할 수 있지만 스레드 로컬 스토리지라는 자체 저장소도 갖고 있다.

스레드 안에서 명령은 한 번에 하나씩 실행돼 스레드가 블록되면 블록이 끝날 때까지 같은 스레드에서 같은 명령을 실행할 수 없다. 그러나 많은 스레드가 같은프로세스에서 생성될 수 있으며 서로 통신할 수 있다. 따라서 애플리케이션이 사용자 경험에 부정적인 영향을 미칠 수 있는 스레드는 블로킹하지 않아야 한다. 블로킹할 때는 블로킹 작업을 별도의 전용 스레드에 할당해야 한다.

그래픽 사용자 인터페이스 애플리케이션에는 UI 스레드가 있다. UI 스레드는 사용자 인터페이스를 업데이트하고 사용자와 애플리케이션 간의 상호작용을 리스닝하는 일을 한다. 스레드를 블록하면 애플리케이션이 UI를 업데이트하거나 사용자로부터 상호작용을 리스닝하는 일을 한다. 스레드를 블록하면 애플리케이션이 UI를 업데이트 하거나 사용자로부터 상호작용을 수신하지 못하도록  방해한다. GUI 애플리케이션은 애플리케이션의 응답성을 항상 유짛기 위해서 UI 스레드를 블록하지 않는다.

예컨데 안드로이드 3.0 이상에서는 UI 스레드에서 네트워킹 작업을 하면 애플리케이션이 중단된다. 네트워킹 작업이 스레드를 블로킹한다는 점을 감안해서 개발자가 이를 수행하지 못하도록 하기 위함이다.

```
한마디로 안드로이드에서는 activity, fragment(UI 스레드)에서 네트워킹 작업하면 앱 중지됨
```

이 책에는 GUI 애플리케이션의 메인 스레드를 UI 스레드나 메인 스레드(안드로이드는 메인스레드 == UI 스레드), 커맨드 라인 애플리케이션의 경우는 메인 스레드라고만 언급할 것이다. 이 두 스레드 외게 다른 스레드는 백그라운드 스레드로 설명하고 백그라운드 스레드 간의 구분이 필요할 때 명확히 하기 위해서 고유한 식별자를 사용하겠다.

코틀린이 동시성을 구현항 방식을 보면 여러분이 직접 스레드를 시작하거나 중지할 필요가 없다는 것을 알게 된다. 한두 줄의 코드로 코틀린이 특정 스레드나 스레드 풀을 생성해서 코루틴을 싱행하도록 지시하기만 실행하도록 지시하기만 하면 된다. 스레드와 관련된 나머지 처리는 프레임워크에 의해 수행된다.

## 코루틴
코틀린 문서에서는 코루틴을 경량 스레드라고도 한다. 대부분의 스레드와 마찬가지로 코루틴이 프로세서가 실행할 명령어 집합의 실행을 정의하기 때문이다. 또한 코루틴은 스레드와 비슷한 라이프 사이클을 갖고 있다.

코루틴은 스레드 안에서 실행된다. 스레드 하나에 많은 코루틴이 있을 수 있지만 주어진 시간에 하나의 스레드에서 하나의 명령만이 실행될 수 있다. 즉 같은 스레드에 10개의 코루틴이 있다면 해당 시점에는 하나의 코루틴만 실행된다.

스레드와 코루틴의 가장 큰 차이점은 코루틴이 빠르고 적은 비용으로 생성할 수 있다는 것이다. 수천 개의 코루틴도 쉽게 생성할 수 있으며, 수천 개의 스레드를 생성하는 것보다 빠르고 자원도 훨씬 적게 사용한다.

다음 코드를 살펴보자. 아직 이해가 되지 않는 부분은 걱정하지 않아도 된다.
```
suspend fun createCoroutines(amount: Int) {
    val jobs = ArrayList<Job>()
    for (i in 1..amount) {
        jobs += launch {
            delay(1000)
        }
    }
    jobs.forEach {
        it.join()
    }
}
```

함수는 파라메터 amount에 지정된 수만큼 코루틴을 생성해 각 코루틴을 1초 간 지연시킨후 모든 코루틴이 종료될 때까지 기다렸다가 반환한다. 예를 들어 이 함수는 amount 를 10,000으로 설정해서 호출될 수 있다.
```
fun main(args: Array<String>) = runBlocking {
    val time = measureTimeMillis {
        createCoroutines(10_000)
    }
    println("Took $time ms")
}

suspend fun createCoroutines(amount: Int) {
    val jobs = ArrayList<Job>()
    for (i in 1..amount) {
        jobs += launch {
            delay(1000)
        }
    }
    jobs.forEach {
        it.join()
    }
}
```

> measureTimeMillis()는 코드 블록을 갖는 인라인 함수이며 실행 시간을 밀리초로 반환한다.
> measureTimeMillis()에는 자매 함수(sibling function)인 measureNanoTime()이 있으며, 시간을 나노초 단위로 반환한다. 두 함수 모두 코드의 실행 시간을 대략적으로 예측할 때 매우 유용하다.

테스트 환경에서 amount 를 10,000으로 실행할 때 약 1,160ms 가 걸리는 반해 100,000으로 실행하는데 1,649ms가 소요됐다. 코틀린은 고정된 크기의 스레드 풀을 사용하고 코루틴을 스레드들에 배포가히기 때문에 실행 시간이 매우 적게 증가한다. 따라서 수천 개의 코루틴을 추가하는 것은 거의 영량이 없다. 코루틴이 일시 중단되는 동안 실행 중인 스레드는 다른 코루틴을 실행하는 데 사용되며 코루틴은 시작 또는 재개될 준비 상태가 된다.

Thread 클래스의 activeCount() 메소드를 활용하면 활성화된 스레드 수를 알 수 있다. 예를 들어 main() 함수를 업데이트해 다음 작업을 수행한다.
```
fun main(args: Array<String>) = runBlocking {
    println("${Thread.activeCount()} threads active at the start")
    
    val time = measureTimeMillis {
        createCoroutines(10_000)
    }
    
    println("${Thread.activeCount()} threads active at the end")
    println("Took $time ms")
}

suspend fun createCoroutines(amount: Int) {
    val jobs = ArrayList<Job>()
    for (i in 1..amount) {
        jobs += launch {
            delay(1000)
        }
    }
    jobs.forEach {
        it.join()
    }
}
```


이전과 같은 테스트 환경에서 10,000갸의 코루틴을 생성하기 위해서 4개의 스레드만 생성하면 된다.

그러나 createCoroutines() 에 amount 값을 1로 낮추면 두개의 스레드만 생성된다.
> 인텔리제이에서는 Monitor Control+Break 라는 스레드 때문에 애플리케이션이 시작될때 두개의 스레드가 있다.

코루틴이 특정 스레드 안에서 실행되더라도 스레드와 묶이지 않는다는 점을 이해해야한다. 코루틴의 일부를 특정 스레드에서 실행하고, 실행을 중지한 다음 나중에 다른 스레드에서 계속 실행하는 것이 가능하다. 이전 예제에서도 일어났던 것으로 코틀린이 실행 가능한 스레드로 코루틴을 이동시키기 때문에다. 가령 createCoroutines()에 amount 를 3으로 하고, launch() 블록을 다음과 같이 변경해서 현재 실행 중인 스레드를 출력시키면 실제 내부에서 일어나는 일을 볼 수 있다.

```
suspend fun createCoroutines(amount: Int) {
    val jobs = ArrayList<Job>()
    for (i in 1..amount) {
        jobs += launch {
            println("started $i in ${Thread.currentThread().name}")
            delay(1000)
            println("finished $i in ${Thread.currentThread().name}")
        }
    }
    jobs.forEach {
        it.join()
    }
}
```


다른 스레드에서 다시 시작하는 경우가 많음을 알게 될 것이다.

스레드는 한 번에 하나의 코루틴만 실행할 수 있기 때문에 프레임워크가 필요에 따라 코루틴을 스레드들 사이에 옮기는 역할을 한다. 코틀린은 개발자가 코루틴을 실행할 스레드를 지정하거나 코루틴을 해당 스레드로 제한할지 여부를 지정할 수 있을 만큼 충분히 유연하다.

## 내용 정리
지금까지 애플리케이션이 하나 이상의 프로세스로 구성돼 있고 각 프로세스가 하나 이상의 스레드를 갖고 있음을 배웠다. 스레드를 블록한다는 것은 그 스레드에서 코드의 실행을 중지한다는 의미인데, 사용자와 상호작용하는 스레드는 블록되지 않아야 한다. 코루틴이 기본적으로 스레드 안에 존재하지만 스레드에 얽매이지 않은 가벼운 스레드 라는 것을 알게 됐다.

동시성은 애플리케이션이 동시에 한 개 이상의 스레드에서 실행될 때 발생한다. 동시성이 발행하려면 두 개 이상의 스레드가 생성돼야 하며, 애플리케이션이 제대로 작동하려면 이런 스레드 간의 통신과 동기화가 필요하다.

---

---

## 동시성에 대해
올바른 동시성 코드는 결정론적인 결과를 갖지만 실행 순서에는 약간의 가변성을 허용하는 코드다. 그러려면 코드의 서로 다른 부분이 어느 정도 독립성이 있어야 하며 약간의 조정도 필요하다. 동시성을 이해하는 가장 좋은 방법은 순차적인 코드를 동시성 코드와 비교하는 것이다. 먼저 비동시성 코드를 살펴본다.

```
fun getProfile(id: Int): Profile {
    val basicUserInfo = getUserInfo(id)     // 사용자 정보
    val contactInfo = getContactInfo(id)    // 연락처 정보
    
    return createProfile(basicUserInfo, contactInfo)
}
```

만약 여러분에게 사용자 정보와 연락처 정보중 어느 정보를 먼저 얻게 될것인지를 묻는다면, 대부분 사용자 정보가 먼저 검색될 거라고 답하리라 생각한다. 여기서 가장 중요한 것은 사용자 정보가 반환되기 전까지 연락처 정보를 요청하지 않는다는 사실이다.

```getProfile { getUserInfo() > getContactInfo() > createProfile() }```

이것이 순차 코드의 장점이다. 정확한 실행 순서를 쉽게 알 수 있어서 예측하지 못한 일이 벌어지지는 않을 것이다. 그러나 순차 코드에는 두 가지의 큰 문제점이 있다.

- 동시성 코드에 비해 성능이 저하될 수 있음
- 코드가 실행되는 하드웨어를 제대로 활용하지 못할 수 있음

getUserInfo 와 getContactInfo 둘 다 웹 서비스를 호출하고, 각 서비스는 응답을 반환하는 데 1초 정도 소요된다고 하자, 즉 getProfile 은 항상 2초 이상 걸릴 것이다. getContactInfo 는 getUserInfo 에 의존하지 않아 보이므로 이들을 동시에 호출해서 getProfile 의 실행 시간을 절반으로 줄일 수 있을 것이다.

getProfile 의 동시성 구현에 관해 살펴보자.
```
suspend fun getProfile(id: Int) {
    val basicUserInfo = asyncGetUserInfo(id)
    val contactInfo = asyncGetContactInfo(id)
    
    createProfile(basicUserInfo.await(), contactInfo.await())
}
```

변경된 버전에서는 getProfile()은 일시 중단 함수(suspend function) 로서 정의에 suspend 수식어가 있음을 알 수 있으며, asyncGetUserInfo() 및 asyncGetContactInfo()의 구현은 비동기다. 두 함수는 예제를 단순화하기 위해서 별도로 표시하지는 않았다.

asyncGetUserInfo()와 asyncGetContactInfo()는 **서로 다른 스레드에서 실행되도록 작성 됐기 때문에 동시성**이라고 한다. 지금은 두 함수가 동시에 실행된다고 생각해보자. 나중에 꼭 그렇지만은 않음을 알게 되겠지만 지금은 동시에 실행된다고 가정하자.

asyncGetContactInfo()의 실행이 asyncGetUserInfo()의 완료에 좌우되지 않으므로 웹 서비스에 대한 요청이 동시에 이룰 수 있다. 각 서비스가 응답하는데 약 1초가 걸린다고 하면, 한상 최소한 2초가 걸리는 순차 코드 버전보다 동시성 버전이 빨리 호출될 것이다. 순차적 코드 버전에서는 getProfile()이 시작된 후 약 1초 후에 createProfile()이 호출된다.
```getProfile { asyncGetUserInfo() -> createProfile <- asyncGetContactInfo() }```

그러나 이 업데이트된 버전의 코드에서는 사용자 정보를 연락처 정보보다 먼저 얻게될지는 실제로 알 수 없다. 앞에서 각각의 웹 서비스가 약 1초 정도 걸린다고 했고, 두 요청이 거의 동시에 시작될 것이라고 말했다.

asyncGetContactInfo()가 asyncGetUserInfo()보다 빠르게 응답하면 연락처 정보를 먼저 얻게 되고, asyncGetUserInfo()가 먼저 반환되면 사용자 정보를 먼저 얻을 수 있으며, 이 둘이 동시에 정보를 반환할 수도 있다. 곧 getProfile()의 동시성 구현 버전은 순차적 구현보다 두 배 빠르게 수행될 수 있지만 실행할 때 약간의 가변성이 있다.

그것이 createProfile()을 호출할 때 두 개의 wait() 호출이 있는 이유다. 이것(wait())이 하는일은 asyncGetUserInfo()와 asyncGetContactInfo()가 모두 완료될 때까지 getProfile()의 실행을 일시 중단하는 것이다. 둘 다 완료됐을 때만 createProfile()이 실행된다. 어떤 동시성 호출이 먼저 종료되는지에 관계없이 getProfile()의 결과가 결정론적임을 보장한다.

그것이 동시성의 까다로운 부분이다. 코드의 준독립적인 부분이 완성되는 순서에 관계없이 결과가 결정적이어야 함을 보장해야 한다. 예제에서는 모든 부분이 완성될 때까지 코드의 일부를 정지시키는 부분만 작업했지만, 책의 뒷부분에서는 동시성 코드를 코루틴 간에 통신하게 하도록 조정할 수 있게 하려고 한다.

## 동시성은 병렬성이 아니다
흔히 동시성과 병렬성을 혼동하곤 한다. 어쨋든 두 개의 코드가 동시에 실행된다는 점에서 둘 다 상당히 비슷해 보이긴 한다. 이 둘을 나눌 명확한 선을 규정할 것이다.

첫 번째 절에서 다룬 비동시성 예제로 돌아가 보자.
```
suspend fun getProfile(id: Int) {
    val basicUserInfo = asyncGetUserInfo(id)
    val contactInfo = asyncGetContactInfo(id)
    
    createProfile(basicUserInfo.await(), contactInfo.await())
}
```

코드의 getProfile()의 실행 타임라인으로 돌아가 보면 getUserInfo()와 getContactInfo()의 실행 시간이 겹치지 않는다. getContactInfo()의 실행은 항상 getUserInfo()가 종료된 후 수행된다.
```getProfile { getUserInfo(1) > getContactInfo(2) > createProfile(finish) }```

이제 동시성 구현을 다시 살펴보자
```
suspend fun getProfile(id: Int) {
    val basicUserInfo = asyncGetUserInfo(id)
    val contactInfo = asyncGetContactInfo(id)
    
    createProfile(basicUserInfo.await(), contactInfo.await())
}
```

코드를 동시적으로 실행(concurrent execution)하면 다음의 그림과 같다. asyncGetUserInfo()와 asyncGetContactInfo()이 중복 실행되는 반면, createProfile()은 둘 중 어느 하나와도 중복되지 않는다.
```당연한거 아닌가?...```

병렬적 실행을 위한 타임라인은 위의 동시성 타임라인과 정확히 같아 보일 것이다. 동시성과 병렬성 타임라인이 모두 똑같아 보이는 이유는 이 타임라인이 하위 레벨에서 일어나고 있는 일을 보여줄 만큼 세분화돼 있지 않기 때문이다. 둘의 차이점은 같은 프로세스 안에서 서로 다른 명령 집합의 타임라인이 겹칠 때 동시성이 발생한다는 점이다. 동시성은 정확히 같은 시점에 실행되는지 여부와는 상관이 없다. 이것을 살펴볼 수 있는 가장 좋은 방법은 getProfile()의 코드를 코어가 하나만 있는 기계에서 실행한다고 상상해보는 것이다, 단일 코어는 두 스레드를 동시에 실행할 수 없기 때문에 asyncGetUserInfo()와 asyncGetContactInfo() 간에 교차 배치돼 스케쥴이 겹치지만 동시에 실행되지는 않는다.

다음 다이어그램은 단일 코어에서 발생한 동시성을 나타낸다. 동시성이지만 병렬은 아니다. 단일 처리 장치는 X와 Y 스레드 사이에 교차 배치되며, 두 개의 전체 일정이 겹치지만 지정된 시점에 둘 중 하나만 실행되고 있다.