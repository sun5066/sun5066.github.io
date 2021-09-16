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
반면에 병령 실행은 두 스레드가 정확히 같은 시점에 실행될 때만 발생한다. 두 개의 코어가 있는 컴퓨터에서 getProfile()이 실행되고 있는 경우 코어 하나는 asyncGetUserInfo()의 명령을 실행하고 다른 하나의 코어에서 asyncGetContactInfo()의 명령을 실행한다.

다음 다이어 그램은 두개의 프로세스 유닛을 사용해 각각 독립적인 스레드를 실행하는 동시성 코드를 병렬로 실행한 것이다. 이때 스레드 X와 Y의 타임라인이 겹칠 뿐만 아니라, 정확히 같은 시점에 실행되고 있다.

요약하자면 다음과 같다.

- 동시성은 두 개 이상의 알고리즘의 실행 시간이 겹쳐질때 발생한다. 중첩이 발생하려면 두 개 이상의 실행 스레드가 필요하다. 이런 스레드들이 단일 코어에서 실행되면 병렬이 아니라 동시에 실행되는데, 단일 코어가 서로 다른 스레드의 인스트럭션을 교차 배치해서, 스레드들의 실행을 효율적으로 겹쳐서 실행한다.
- 병렬은 두 개의 알고리즘이 정확히 같은 시점에 실행될 때 발생한다. 이것이 가능하려면 2개 이상의 코어와 2개 이상의 스레드가 있어야 각 코어가 동시에 스레드의 인스트럭션을 실행할 수 있다. 병렬은 동시성을 의미하지만 동시성은 병렬성이 없이도 발생할 수 있다는 점에 유의하자.

다음 절에서 보겠지만, 이 차이는 단순히 사소한 기술적인 세부사항이 아니다. 차이를 이해해야 제대로 작동하는 코드를 작성하는 데 많은 도움이 될 것이다.
```
병렬성의 필요저건인 두 개 이상의 코어가 꼭 로컬 컴퓨터로 국한되지 않는다는 것을 말하고 싶다. 그 예로 애플리케이션이 네트워크의 여러 다른 컴퓨터에서 분산 작업을 실행하도록 할 수 있다. 이러한 구현을 분산 컴퓨팅이라고 하며, 병렬성의 한 형태이다.
```

## CPU 바운드와 I/O 바운드
병목 현상은 다양한 유형의 성능저하가 발생하는 지점을 나타낸다. 애플리케이션의 성능을 최적화할 때 가장 중요한 사항이다. 동시성과 병렬성이 CPU 나 I/O 연산에 바인딩됐는지 여부에 따라 알고리즘의 성능에 어떻게 영향을 미칠 수 있는지를 알아본다.
```
동시성 코드가 항상 필요한 것도 아니고 이를 통해 꼭 이득을 얻는 것도 아니다. 코드의 병목과 스레드 및 코루틴의 작동방식과 동시성 및 병렬성 간의 차이를 이해해 두면 동시성 소프트웨어를 언제 어떻게 구현해야 하는지를 제대로 판단할 수 있다.
```

## CPU 바운드
CPU 만 완료하면 되는 작업을 중심으로 구현되는 알고리즘이 많다. 알고리즘의 성능은 실행 중인 CPU 의 성능에 좌우되며 CPU 만 업그레이드해도 성능이 향상된다.

단어를 가져와서 좌우가 같은 단어인지를 판별하는 간단한 알고리즘을 살펴보자.
```
fun isPalindrome(word: String): Boolean {
    val lcWord = word.toLowerCase()
    return lcWord == lcWord.reversed()
}
```

단어 목록을 가져와서 좌우가 같은 단어를 반환하는 filterPalindromes()에서 위의 함수를 호출한다고 생각해보자
```
fun filterPalindromes(words: List<String>): List<String> {
    return words.filter { isPalindrome(it) }
}
```

마지막으로 단어 목록이 이미 정의된 애플리케이션의 main 메소드에서 filterPalindromes()를 호출한다.
```
val words = listOf("level", "pope", "needle", "Anna", "Pete", "noon", "status")

fun main(args: Array<String>) {
    filterPalindromes(words).forEach { println(it) }
}
```

예제에서는 실행의 모든 부분이 CPU 성능에 의존적이다. 수십 만 개의 단어를 보내도록 코드를 바꾸면 filterPalindromes()는 더 오래 걸릴 것이다. 코드를 더 빠른 CPU 에서 실행하면 코드의 변경 없이도 성능이 향상된다.

## I/O 바운드
I/O 바운드는 입출력 장치에 의존하는 알고리즘이다. 실행 시간은 입출력 장치의 속도에 따라 달라지는데, 예컨데 문서를 읽어서 문서의 각 단어를 filterPalindromes()에 전달해 좌우가 같은 단어를 출력하는 알고리즘이 I/O 바운드다. 방금 예제에서 몇 줄을 변경하면 다음과 같은 효과를 얻을 수 있다.
```
fun main(args: Array<String>) {
    val words = readWordsFromJson("resources/words.json")
    filterPalindromes(words).forEach { println(it) }
}
```

readWordsFromJson() 함수는 파일 시스템에서 파일을 읽는다. 파일을 읽는 속도에 따라 성능이 달라지는 I/O 작업이다. 예를 들어 파일을 하드 드라이브에 저장하면 SSD 에 저장하는 경우보다 애플리케이션 성능이 더 나빠진다.

네트워킹이나 컴퓨터 주변 기기로부터 입력을 받는 작업들도 I/O 작업이다. I/O 바운드 알고리즘은 I/O 작업을 기준으로 성능에 대한 병목 현상을 일으키는데, 최적화가 외부 시스템이나 장치에 의존한다는 것을 의미한다.
```
많은 I/O 바운드를 갖는 데이터베이스와 같은 고성능 애플리케이션은 결국 실행 중인 기기의 스토리지 액세스 속도에 따라 성능이 좌우된다. 스마트폰 애플리케이션과 같은 네트워킹 기반 애플리케이션도 마찬가지로 인터넷 연결 속도에 따라 성능이 좌우된다.
```

## CPU 바운드 알고리즘에서의 동시성과 병렬성
CPU 바운드 알고리즘의 경우 다중 코어에서 병렬성을 활용하면 성능을 향상시킬 수 있지만 단일 코어에서 동시성을 구현하면 성능이 저하되기도 한다. 예를 들어 단어 목록을 가져와 좌우가 같은 단어를 필터링하는 이전의 알고리즘을 1,000개의 단어당 하나의 스레드가 생성되도록 수정할 수 있다. isPalindrome()이 단어 3,000개를 입력 받는다면 실행은 다음과 같을 것이다.

```
Thread-X {"Anna", "Pete", "Noon", ...}
Thread-Y {"level", "status", "music", ...}
Thread-Z {"needle", "pointer", "epic", ...}

isPalindrome은 1,000개 단어당 하나의 스레드를 생성
```

# 단일 코어에서 실행
단일 코어에서 실행된다면 하나의 코어가 3개의 스레드 사이에서 교차배치 되며 매번 일정량의 단어를 필터링하고 다음 스레드로 전환된다. 전환 프로세스를 *컨텍스트 스위칭* 이라고 한다.

컨텐스트 스위칭은 현재 스레드의 상태를 저장한 후 다음 스레드의 상태를 적재해야 하기 때문에 전체 프로세스에 오버헤드가 발생한다. 오버헤드로 인해 다중 스레드로 구현한 isPalindrome()은 앞에서 본 순차적 구현에 비해 단일 코어 머신에서는 더 오래 걸릴 가능성이 있다. 순차적 구현에서는 단일 코어가 모든 작업을 수행하지만 컨텍스트 스위칭이 발생하지 않기 때문이다.

## 병렬 실행
병렬 실행의 경우 각 스레드가 하나의 전용 코어에서 실행된다고 가정하면 isPalindrome()의 실행은 순차적 실행의 약 3분의 1이 될 것이다. 각 코어는 1,000개의 단어를 중단 없이 필터링해서 작업을 완료하는 데 필요한 총 시간을 줄일 것이다.

CPU 바운드 알고리즘을 위해서는 현재 사용 중인 장치의 코어 수를 기준으로 적절한 스레드 수를 생성하도록 고려해야만 한다. 이렇게하면 CPU 바운드 알고리즘을 실행하기 위해 생성된 스레드 풀인 코틀린의 CommonPool 을 활용할 수 있다.
```
CommonPool 의 크기는 머신의 코어 수에서 1을 뺀 값이다. 4개의 코어가 있는 머신에서는 크기가 3이 될 것이다.
```

## I/O 바운드 알고리즘에서의 동시성 대 병렬성
앞에서 봤듯이 I/O 바운드 알고리즘은 끊임없이 무언가를 기다린다. 지속적인 대기는 단일 코어 기기에서 대기하는 중에 다른 유용한 작업에 프로세스를 사용할 수 있도록한다.따라서 I/O 바운드인 동시성 알고리즘은 병렬이거나 단일 코어에 상관없이 유사하게 수행될 것이다.

I/O 바운드 알고리즘은 순차적인 알고리즘보다 동시성 구현에서 항상 더 나은 성능을 발휘할 것으로 예상돼 I/O 작업은 늘 동시성으로 실행하는 편이 좋다. 앞서 설명한 바와 같이 GUI 애플리케이션에서는 UI 스레드를 블록하지 않는 것이 무엇보다 중요하다.

## 동시성이 어려운 이유
동시성 코드를 제대로 작성하기란 좀처럼 쉽지 않다. 동시성 프로그램 자체가 어렵기도 하지만 많은 프로그래밍 언어가 이것을 더 어렵게 만들기 때문이기도 하다. 어떤 언어들은 너무 번거롭게 동시성 코드를 만들기도 하고, 융통성 없이 만들어 사용을 떨어뜨리는 언어도 있다. 코틀린 팀은 이점을 염두에 두고 동시성 프로그래밍을 가능한 단순하면서도 여러 가지 사용 사례에 맞춰 조절할 수 있도록 충분히 유연하게 만들려고 노력했다. 이 책의 뒷부분에서 코틀린 팀이 만들어 낸 기본형을 사용해 다양한 사용 사례들을 다루게 될 텐데, 지금은 동시성 코드를 프로그래밍할 때 제시되는 공통된 문제점을 살펴본다.

코틀린이 우리가 만든 동시성 코드를 동기화하고 통신할 수 있게 만들기 때문에 실행 흐름이 바뀌어도 애플리케이션의 작동에는 영향이 없다.

## 레이스 컨디션
동시성 코드를 작성할 때 가장 흔한 오류인 레이스 컨디션은 코드를 동시성으로 작성했지만 순차적 코드처럼 동작할 것이라고 예샹할 때 발생한다. 좀더 구체적으로는 동시성 코드가 항상 특정한 순서로 실행될 것이라 가정하고 오해할 때 생기는 문제다.

예를 들어, 데이터베이스에서 데이터를 가져오고 웹 서비스를 호출하는 기능을 동시에 수행하는 코드를 작성 중이라고 가정하자. 이 두 작업이 모두 끝나면 약간의 연산을 수행해야 한다. 많은 사람들이 가장 흔히 하는 실수는 데이터베이스가 더 빠를 것으로 가정하고 웹 서비스 작업이 끝나자마자 데이터베이스 작업의 결과에 접근하려고 하는 것이다. 이때쯤이면 데이터베이스에서 나온 정보가 항상 준비되리라 생각한다. DB 작업이 웹 서비스 호출보다 오래 걸릴 때 마다 애플리케이션 중단이 되거나 일관되지 않은 상태에 빠진다.

*레이스 컨디션은 동시성 코드 일부가 제대로 작동하기 위해 일정한 순서로 완료돼야 할때 발생한다.* 이것은 동시성 코드를 구현하는 방법이 아니다.

간단한 예를 살펴보자.
```
data class UserInfo(val name: String, val lastName: String, val id: Int)

lateinit var user: UserInfo

fun main(args: Array<String>) {

    asyncGetUserInfo(1)
    // Do some other operations
    delay(1000)
    
    println("User ${user.id} is ${user.name}")
}

fun asyncGetUserInfo(id: Int) = async {
    user = UserInfo(id = id, name = "Susan", lastName = "Calvin")
}
```

main() 함수는 백그라운드 코루틴으로 사용자 정보를 얻으며 1초를 지연한 후에는 사용자 이름을 출력한다. 이 코드는 1초를 지연하기 때문에 잘 작동할 것이다. 지연을 없애거나 asyncGetUserInfo() 안에서 지연 시간을 크게 하면 애플리케이션이 중단된다. asyncGetUserInfo()를 다음과 같이 바꿔보자

```
fun asyncGetUserInfo(id: Int) = async {
    delay(1100)
    user = UserInfo(id = id, name = "Susan", lastName = "Calvin")
}
```

실행하면 user 에 정보를 출력하는 동안 초기화 되지 않아서 main()이 중단된다. 레이스 컨디션을 고치려면 정보에 접근하려고 하기 전에 정보를 얻을 때까지 명시적으로 기다려야만 한다.

## 원자성 위반
원자성 작업이란 작업이 사용하는 데이터를 간섭 없이 접근할 수 있음을 말한다. 단일 스레드 애플리케이션에서는 모든 코드가 순차적으로 실행되기 때문에 모든 작업이 모두 원자일 것이다. 스레드가 하나만으로 실행되므로 간섭이 있을 수 없다.

원자성은 객체의 상태가 동시에 수정될 수 있을 때 필요하며 그 상태의 수정이 겹치지 않도록 보장해야 한다. 수정이 겹칠 수 있다는 것은 데이터 손실이 발생할 수 있다는 뜻인데, 가령 코루틴이 다른 코루틴이 수정하고 있는 데이터를 바꿀 수 있다는 것이다. 실제 사례를 보자.
```
var counter = 0
fun main(args: Array<String>) = runBlocking {
    val workerA = asyncIncrement(2000)
    val workerB = asyncIncrement(100)
    workerA.await()
    workerB.await()
    print("counter [$counter]")
}

fun asyncIncrement(by: Int) = GlobalScope.async {
    for (i in 0 until by) counter++
}
```

위 코드는 원자성 위반의 간단한 예시코드 입니다. 위 코드에서 asyncIncrement() 코루틴을 두 번 동시에 실행한다. 한 호출에서는 counter 를 2,000번 증가시키는 반면 다른 호출에서는 100번 증가시킬 것이다. 문제는 asyncIncrement()의 두 실행이 서로 간섭할 수 있으며, 서로 다른 코루틴 인스턴스가 값을 재정의할 수 있다는 것이다. main()을 실행하면 대부분 counter == 2100 이겠지만, 꽤 많은 실행에서 2,100 보다 적은 값을 인쇄한다는 것을 의미한다.
```
  10
  10                      10
  10           10
  11                      11
  11           11
--------------------------------
counter     workerA     workerB
```

위 결과 에서는 counter++의 원자성이 부족해서 workerA 와 workerB 에서 각각 한 번씩 counter 값을 증가시켜야 하지만 한 번만 증가시켰다. 이런 현상이 발생할 때마다 예상 값이 2,100에서 1씩 줄어들게 된다.

코루틴에서 명령이 중쳡되는 것은 counter++작업이 원자적이지 않기 때문이다. 실제로 이 작업은 counter 의 현재 값을 읽은 다름 그 값을 1씩 증가시킨 후에 그 결과를 counter 에 다시 저장하는 세 가지 명령어로 나눌 수 있다. counter++ 에서 원자성이 없기 때문에 두 코루틴이 다른 코루틴이 하는 조작을 무시하고 값을 읽고 수정할 수 있다.

## 교착상태
동시성 코드가 올바르게 동기화되려면 다른 스레드에서 작업이 완료되는 동안 실행을 일시 중단하거나 차단할 필요가 있다. 이러한 상황의 복잡성 때문에, 즉 순환적 의존성으로 인해 전체 애플리케이션의 실행이 중단되는 상황이 드물지 않게 일어난다.
```
lateinit var jobA: Job
lateinit var jobB: Job

fun main(args: Array<String>) = runBlocking {
    jobA = GlobalScope.launch {
        delay(1000)
        // wait for jobB to finish
        jobB.join()
    }
    
    jobB = GlobalScope.launch {
        // wait for jobB to finish
        jobA.join()
    }
    
    // wait for jobA to finish
    jobA.join()
    println("Finished")
}
```

위 코드를 설명하자면, jobA 는 jobB 가 끝나기를 기다리고 있고,
반대로 jobB 도 jobA 가 끝나기를 기다리고 있다.
둘 다 서로를 기다리고 있기 때문에 누구도 끝나지 않는다. 따라서 Finished 메시지는 출력되지 않는다.

단순화하려고 의도된 예시라서 실제 시나리오에서 교착 상태를 발견하고 수정하기란 이보다 훨씬 어렵다. 일반적으로 복잡한 잠금 연관관계에 의해 발생하며 레이스 컨디션과 자주 같이 발생한다. 레이스 컨디션은 교착 상태가 발생할 수 있는 예기치 않은 상태를 만들기도 한다.

## 라이브 락
라이브 락은 애플리케이션이 올바르게 실행을 계속할 수 없을 때 발생하는 교착상태와 유사하다. 라이브 락이 진행될 때 애플리케이션의 상태는 지속적으로 변하지만 애플리케이션이 정상 실행으로 돌아오지 못하게 하는 방향으로 변한다는 점이 다르다.

> '엘리야' 와 '수잔' 이라는 두 사람이 좁은 복도에서 마주보고 걸어오는 모습으로 라이브락을 설명한다. 두 사람은 각각 한쪽 방향으로 이동해서 서로를 피하려 한다. 엘리야가 왼쪽으로 이동할 때 수잔은 오른쪽으로 이동하지만 마주보고 있기 때문에 서로의 길을 막고 있다. 이제 엘리야는 오른쪽으로 움직이지만, 동시에 수잔이 왼쪽으로 움직이면서 다시 한번 서로 길을 막게된다. 그들은 계속 이렇게 움직이며 서로의 길을 막게 된다.

위의 예시에서 엘리야와 수잔 모두 교착 상태에서 벗어나는 방법을 알고 있다. 그러나 회복하려는 시점이 그들의 진행을 방해한다.

교착 상태를 복구하도록 설계된 알고리즘에서 라이브 락이 발생하는 경우가 많다. 교착 상태에서 복구하려는 시도가 라이브 락을 만들어 낼 수도 있다.

# 코틀린에서의 동시성
동시성의 기본 사항을 쭉 살펴봤고 코틀린에서 동시성의 구체적인 애용을 알아본다. 이제 동시성 프로그램에서 코틀린의 가장 차별화된 특징을 보여줄 것이며 철하걱인 주제와 기술적인 주제를 모두 다루려고 한다.

## 넌 블로킹
스레드는 무겁고 생성하는 데 비용이 많ㄹ이 들며 제한된 수의 스레드만 생성할 수 있다. 스레드가 블로킹 되면 어떻게 보면 자원이 낭비되는 셈이어서 코틀린은 중단 가능한 연산이라는 기능을 제공한다. 스레드의 실행을 블로킹하지 않으면서 실행을 잠시 중단하는 것이다. 예를 들어 스레드 Y에서 작업이 끝나기를 기다리려면 스레드 X를 블로킬 하는대신, 대기해야 하는 코드를 일시 중단하고 그동안 스레드 X를 다른 연산 작업에 사용하기도 한다.

코틀린은 채널, 액터, 상호 배제와 같은 훌륭한 기본형도 제공해 스레드를 블록하지 않고 동시성 코드를 효과적으로 통신하고 동기화하는 메커니즘을 제공한다.

## 명시적인 선언
동시성은 깊은 고민과 설계가 필요해, 연산이 동시에 실행돼야 하는 시점을 명시적으로 만드는 것이 중요하다. 일시 중단 가능한 연산은 기본적으로 순차적으로 실행된다. 연산은 일시 중단될 때 스레드를 블로킹하지 않기 때문에 직접적인 단점은 아니다.
```
fun main(args: Array<String>) = runBlocking {
    val time = measureTimeMillis {
        vval name = getName()
        val lastName = getLastName()
        println("Hello, $name $lastName")
    }
    println("Execution took $time ms")
}

suspend fun getName(): String { // 일시 중단 가능한 함수
    delay(1000)
    return "Susan"
}

suspen fun getLastName(): String { // 일시 중단 가능한 함수
    delay(1000)
    return "Calvin"
}
```

위 코드에서 main()은 현재 스레드에서 일시 중단 가능한 연산 getName()과 getLastName()을 순차적으로 실행한다.

main()을 실행하면 다음 내용을 출력한다.
> Hello, Susan $Calvin
>
> Execution took 2016 ms

실행 스레드를 블록하지 않는 비 동시성 코드를 작성할 수 있어서 편리하다. 그러나 어느 정도 시간이 지나 분석하고 나면, getLastName()과 getName() 간에 서로 의존성이 없기 때문에, getLastName()이 getName()이 실행될 때까지 기다려야 할 필요가 없음을 알게 된다. 대충 동시에 수행하는 편이 더 낫다는 말이다.(await)

```
fun main(args: Array<String>) = runBlocking {
    val time = measureTimeMillis {
        val name = async { getName() }
        val lastName = async { getLastName() }
        
        println("Hello, ${name.await()} ${lastName.await()}")
    }
    println("Execute took $time ms")
}
```

이제는 async {...}를 호출해 두 함수를 동시에 실행해야 하며 await()를 호출해 두 연산에 모두 결과가 나타날 때까지 main()이 일시중단되도록 요청한다.
> Hello, Susan $Calvin
> 
> Execution took 1019 ms

> 비 동시성 코드의 took = 2016ms
> 
> 동시성 코드의 took = 1019ms
> 
> 동시성 > 비 동시성

## 가독성
코틀린의 동시성 코드는 순차적 코드만큼 읽기 쉽다. 자바를 비롯해 다른 언어에서는 동시성 코드를 이해하고 디버깅하는 것이 어렵다는 문제점이 있다. 코틀린의 접근법은 관용구적인 동시성 코드를 허용한다. (대충 코틀린에서는 동시성 프로그래밍이 다른 언어들보다는 쉽다는 말)

```
suspend fun getProfile(id: Int) {
    val basicUserInfo = asyncGetUserInfo(id)
    val contractInfo = asyncGetContractInfo(id)
    
    createProfile(basicUserInfo.await(), contractInfo.await())
}
```

> 보통 동시성 함수의 네이밍은 asyncFunction 혹은 functionAsync 처럼
> 
> async 로 시작하거나 Async 로 끝나도록 이름을 짓자.

suspend 메소드는 백그라운드 스레드에서 실행될 두 메소드를 호출하고 정보를 처리하기 전에 완료를 기다린다. 순차 코드처럼 간단하게 읽고 디버깅하기가 쉬운 코드가 됐다.

> 비동기 함수를 작성하는 대신 suspend 함수를 작성해 async {} 또는 launch {} 블록 안에서 호출하는 것이 좋다. suspend 함수를 갖게 되면 함수의 호출자에게 더 많은 유연성을 제공하기 때문이다. 가령 호출자가 언제 동시적으로 실행할 것인지를 결정할 수 있다. 그 밖에는 동시적 함수와 일시 중단 함수를 모두 작성하길 원할 때 유용하다.

## 기본형 활용
스레드를 만들고 관리하는 것은 여러 프로그래밍 언어에서 동시성 코드를 작성할 때 가장 어려운 부분 중 하나다. 언제 스레드를 만들 것인가를 아는 것 못지 않게 얼마나 많은 스레드를 만드는지를 아는 것도 중요하다. 또한 I/O 작업 전용 스레드와 CPU 바운드 작업을 처리하는 스레드가 있어야 하는데, 스레드를 통신/동기화하는 것은 그 자체로 어려운 일이다.
코틀린은 동시성 코드를 쉽게 구현할 수 있는 고급 함수와 기본형을 제공한다.
- 스레드는 스레드 이름을 파라미터로 하는 newSingleThreadContext()를 호출하면 생성된다. 일단 생성되면 필요한 만큼 많은 코루틴을 수행하는 데 사용할 수 있다.
- 스레드 풀은 크기와 이름을 파라미터로 하는 newFixedThreadPoolContext()를 호출하면 쉽게 생성할 수 있다.
- CommonPool은 CPU 바운드 작업에 최적인 스레드 풀이다. 최대 크기는 시스템의 코어에서 1을 뺀 값이다.
- 코루틴을 다른 스레드로 이동시키는 역할은 런타임이 담당한다.
- 채털, 뮤텍스 및 스레드 한정과 같은 코루틴의 통신과 동기화를 위해 필요한 많은 기본형과 기술이 제공된다.

## 유연성
코틀린은 간단하면서도 유연하게 동시성을 사용하게 해주는 기본형을 많이 제공한다.
코틀린에서 동시성 프로그래밍을 수행하는 방법이 많음을 알게 될 것이다. 다음은 책 전체에서 살표볼 주제들이다.
- 채널(channels): 코루틴 간에 데이터를 안전하게 보내고 받는 데 사용할 수 있는 파이프다.
- 작업자풀(worker pools): 많은 스레드에서 연산 집합의 처리를 나눌 수 있는 코루틴의 풀이다.
- 액터(actors): 채널과 코루틴을 사용하는 상태를 감싼 래퍼로 여러 쓰레드에서 상태를 안전하게 수정하는 메커니즘을 제공한다.
- 뮤텍스(mutexes): 크리티컬 존 영역을 정의해 한 번에 하나의 스레드만 실행할 수 있도록 하는 동기화 메커니즘. 크리티컬 존에 액세스하려는 코루틴은 이전 코루틴이 크리티컬 존을 빠져나올 때까지 일시 정지된다.
- 스레드 한정(Thread confinement): 코루틴의 실행을 제한해서 지정된 스레드에서만 실행하도록 하는 기능이다.
- 생성자(반복자 및 시퀀스: 필요에 따라 정보를 생성할 수 있고 새로운 정보가 필요하지 않을 때 일시 중단될 수 있는 데이터 소스다.)

모든 것은 코틀린에서 동시성 코드를 작성할 때 수시로 사용할 수 있는 도구이며, 그 범위가 용례는 동시성 코드를 구현할 때 올바른 선택을 하는 데 도움이 될 것이다.