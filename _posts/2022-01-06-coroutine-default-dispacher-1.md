---
layout: post
title: "Coroutine Dispatchers.Default에 대해 알아보자"
date: 2022-01-06 14:15:00 -0400
categories: coroutine
tags: kotlin, coroutine
comments: 1
---

코루틴은 디스패처를 지정해 어떤 스케줄러에서 돌아갈지 선택할 수 있다.

이번 공부의 목표는 코루틴의 디스패처가 어떻게 스케쥴러를 지정하고
생성된 스레드풀은 어떤 옵션을 가지는지와 Default, IO 두가지 디스패처는 같은 스케쥴러이지만 어떤 방식으로 CPU 바운드, I/O 바운드가 동작하는지 확인하는것이다.

# Dispatcher

> - Default
> - IO
> - Main
> - Unconfined

먼저 디스패처의 종류는 총 4가지 이지만
안드로이드에서는 Unconfined 디스패처를 거의 사용을 안하기 때문에 이 부분은 패스


## Dispatchers.Default

먼저 디폴트 디스패처의 코드를 타고 들어가다보면 아래의 코드를 마주친다.

```
internal actual fun createDefaultDispatcher(): CoroutineDispatcher =
    if (useCoroutinesScheduler) DefaultScheduler else CommonPool
```

`useCoroutinesScheduler` 이 플래그는 `System.getProperty("kotlinx.coroutines.scheduler")`의 결과 값에 따라 다르다.

> - `null, "", "on"` -> true
> - "off" -> false
> - else -> IllegalStateException

필자의 경우 값이 `null` 이라서 플래그가 `true`였기에

`DefaultScheduler` 객체가 반환됨.

해당 클래스의 코드를 보면 object 클래스로 되어있으며, 우리가 호출했던 `Dispatchers.IO`는 `LimitingDispatcher`라는 객체를 반환하고있다.

```
internal object DefaultScheduler : ExperimentalCoroutineDispatcher() {
    val IO: CoroutineDispatcher = LimitingDispatcher(
        this,
        systemProp(IO_PARALLELISM_PROPERTY_NAME, 64.coerceAtLeast(AVAILABLE_PROCESSORS)),
        "Dispatchers.IO",
        TASK_PROBABLY_BLOCKING
    )

    override fun close() {
        throw UnsupportedOperationException("$DEFAULT_DISPATCHER_NAME cannot be closed")
    }

    override fun toString(): String = DEFAULT_DISPATCHER_NAME

    @InternalCoroutinesApi
    @Suppress("UNUSED")
    public fun toDebugString(): String = super.toString()
}
```

`LimitingDispatcher`에 대해서 알아보기전에 `DefaultScheduler` 클래스가 상속받고있는 `ExperimentalCoroutineDispatcher` 클래스에 대해서 먼저 알아봐야할꺼같다.

주석에도 나와있듯 레거시 코드이며 언젠가 없어질 수도 있는 클래스다.

> 현재 가장 최신버전인 kotlin 1.6.10 버전에서도 아직 사용중임

```
/**
 * @suppress **This is unstable API and it is subject to change.**
 */
// TODO make internal (and rename) after complete integration
@InternalCoroutinesApi
public open class ExperimentalCoroutineDispatcher(
    private val corePoolSize: Int,
    private val maxPoolSize: Int,
    private val idleWorkerKeepAliveNs: Long,
    private val schedulerName: String = "CoroutineScheduler"
) : ExecutorCoroutineDispatcher() {
    public constructor(
        corePoolSize: Int = CORE_POOL_SIZE,
        maxPoolSize: Int = MAX_POOL_SIZE,
        schedulerName: String = DEFAULT_SCHEDULER_NAME
    ) : this(corePoolSize, maxPoolSize, IDLE_WORKER_KEEP_ALIVE_NS, schedulerName)

    @Deprecated(message = "Binary compatibility for Ktor 1.0-beta", level = DeprecationLevel.HIDDEN)
    public constructor(
        corePoolSize: Int = CORE_POOL_SIZE,
        maxPoolSize: Int = MAX_POOL_SIZE
    ) : this(corePoolSize, maxPoolSize, IDLE_WORKER_KEEP_ALIVE_NS)

    override val executor: Executor
        get() = coroutineScheduler

    // This is variable for test purposes, so that we can reinitialize from clean state
    private var coroutineScheduler = createScheduler()

    override fun dispatch(context: CoroutineContext, block: Runnable): Unit =
        try {
            coroutineScheduler.dispatch(block)
        } catch (e: RejectedExecutionException) {
            // CoroutineScheduler only rejects execution when it is being closed and this behavior is reserved
            // for testing purposes, so we don't have to worry about cancelling the affected Job here.
            DefaultExecutor.dispatch(context, block)
        }

    override fun dispatchYield(context: CoroutineContext, block: Runnable): Unit =
        try {
            coroutineScheduler.dispatch(block, tailDispatch = true)
        } catch (e: RejectedExecutionException) {
            // CoroutineScheduler only rejects execution when it is being closed and this behavior is reserved
            // for testing purposes, so we don't have to worry about cancelling the affected Job here.
            DefaultExecutor.dispatchYield(context, block)
        }

    override fun close(): Unit = coroutineScheduler.close()

    override fun toString(): String {
        return "${super.toString()}[scheduler = $coroutineScheduler]"
    }

    /**
     * Creates a coroutine execution context with limited parallelism to execute tasks which may potentially block.
     * Resulting [CoroutineDispatcher] doesn't own any resources (its threads) and provides a view of the original [ExperimentalCoroutineDispatcher],
     * giving it additional hints to adjust its behaviour.
     *
     * @param parallelism parallelism level, indicating how many threads can execute tasks in the resulting dispatcher parallel.
     */
    public fun blocking(parallelism: Int = BLOCKING_DEFAULT_PARALLELISM): CoroutineDispatcher {
        require(parallelism > 0) { "Expected positive parallelism level, but have $parallelism" }
        return LimitingDispatcher(this, parallelism, null, TASK_PROBABLY_BLOCKING)
    }

    /**
     * Creates a coroutine execution context with limited parallelism to execute CPU-intensive tasks.
     * Resulting [CoroutineDispatcher] doesn't own any resources (its threads) and provides a view of the original [ExperimentalCoroutineDispatcher],
     * giving it additional hints to adjust its behaviour.
     *
     * @param parallelism parallelism level, indicating how many threads can execute tasks in the resulting dispatcher parallel.
     */
    public fun limited(parallelism: Int): CoroutineDispatcher {
        require(parallelism > 0) { "Expected positive parallelism level, but have $parallelism" }
        require(parallelism <= corePoolSize) { "Expected parallelism level lesser than core pool size ($corePoolSize), but have $parallelism" }
        return LimitingDispatcher(this, parallelism, null, TASK_NON_BLOCKING)
    }

    internal fun dispatchWithContext(block: Runnable, context: TaskContext, tailDispatch: Boolean) {
        try {
            coroutineScheduler.dispatch(block, context, tailDispatch)
        } catch (e: RejectedExecutionException) {
            // CoroutineScheduler only rejects execution when it is being closed and this behavior is reserved
            // for testing purposes, so we don't have to worry about cancelling the affected Job here.
            // TaskContext shouldn't be lost here to properly invoke before/after task
            DefaultExecutor.enqueue(coroutineScheduler.createTask(block, context))
        }
    }

    private fun createScheduler() = CoroutineScheduler(corePoolSize, maxPoolSize, idleWorkerKeepAliveNs, schedulerName)

    // fot tests only
    @Synchronized
    internal fun usePrivateScheduler() {
        coroutineScheduler.shutdown(1_000L)
        coroutineScheduler = createScheduler()
    }

    // for tests only
    @Synchronized
    internal fun shutdown(timeout: Long) {
        coroutineScheduler.shutdown(timeout)
    }

    // for tests only
    internal fun restore() = usePrivateScheduler() // recreate scheduler
}
```

> CORE_POOL_SIZE
> 
> 번역: 다중 스레드 문제를 해결하기 위해 최소 두 개의 스레드로 기본값을 강제 적용합니다.
  단일 코어 컴퓨터에서도 복제되지만 필요한 경우 1 스레드 스케줄러의 명시적 설정을 지원합니다.
> 
> - `System.getProperty("kotlinx.coroutines.scheduler.core.pool.size")`의 값이 `null` 인 경우
>   - 기본값으로 런타임에서 CPU 코어수 불러와 사용한다.(이 마저도 최소값이 2로 하드코딩 되어있다.)
> - 값이 1인 경우 `IllegalStateException` 예외를 발생시킨다.

> MAX_POOL_SIZE
> 
> 이름 그대로 스레드 풀의 사이즈 limit
> 
> - `System.getProperty("kotlinx.coroutines.scheduler.max.pool.size")`도 마찬가지로 값이 `null`인 경우 기본값으로 사용하는데 이때 보내는 기본값은
>   - (`CPU 코어 수` * 128)인 값을 계산 후 `CPU 코어 수`와 `(1 shl 21) - 2`를 비교해서 적정값을 반환한다.
> 
> > `(1 shl 21) - 2` shl는 Left Shift 연산이며 계산해보면 2,097,150이 나온다.
> 
> 개발자는 말보단 코드로 보는게 이해가 빠르니 아래의 코드를 확인해보자.
> CPU 코어 개수는 필자의 디바이스 기준(갤럭시 노트 5 - 옥타코어) 8개다.
>
> ```
> val AVAILABLE_PROCESSORS = 8
> val CORE_POOL_SIZE = 8 // AVAILABLE_PROCESSORS 와 동일
> val MAX_SUPPORTED_POOL_SIZE = (1 shl 21) - 2 // 2,097,150 맞나?
> 
> val defaultValue = (AVAILABLE_PROCESSORS * 128) // 1,024
> 
> defaultValue.coerceIn(CORE_POOL_SIZE, MAX_SUPPORTED_POOL_SIZE)
> ```
> 
> 대충 위 `defaultValue`의 값을 기준으로 `CORE_POOL_SIZE`와 `MAX_SUPPORTED_POOL_SIZE`의 값을 비교해서 적정값을 리턴하는데
> 그 기능을 수행하는게 바로 `coerceIn(min, max)` 함수다.
> 
> ```
> public fun Int.coerceIn(minimumValue: Int, maximumValue: Int): Int {
>     if (minimumValue > maximumValue) throw IllegalArgumentException("Cannot coerce value to an empty range: maximum $maximumValue is less than minimum $minimumValue.")
>     if (this < minimumValue) return minimumValue
>     if (this > maximumValue) return maximumValue
>     return this
> }
> ```
> 
> 결과는 위 조건문을 타지 않기 때문에 `defaultValue = 1024` 그대로 리턴되었다.

> IDLE_WORKER_KEEP_ALIVE_NS
> - `System.getProperty("kotlinx.coroutines.scheduler.keep.alive.sec")` 값이 `null`인 경우 `60L`을 기본값으로 사용한다.
> - 필자의 디바이스에서는 기본값으로 사용됨

> DEFAULT_SCHEDULER_NAME
> - 단순히 String 상수로 "DefaultDispatcher"를 갖고있다.

`ExecutorCoroutineDispatcher`의 생생자에서 만든 값을 그대로 `CoroutineScheduler(corePoolSize, maxPoolSize, idleWorkerKeepAliveNs, schedulerName)`를 만들어주고있다.

`CoroutineScheduler` 부터는 2편에서...