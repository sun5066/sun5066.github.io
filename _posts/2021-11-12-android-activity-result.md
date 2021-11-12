---
title: "안드로이드 Deprecated된 onActivityResult() 함수의 로직을 변경없이 마이그레이션 하기"
date: 2021-11-12 9:00:00 -0400
categories: android
---

조금은 늦었지만, 괜찮은 방법이라 생각하여 공유하고자 합니다.


## ActivityResultLauncher 콜백 객체 만들기
```
class ResultObserver(private val registry: ActivityResultRegistry) : LifecycleObserver {

    private lateinit var getIntentContent: ActivityResultLauncher<Intent>
    private var intentCallback: ((ActivityResult) -> Unit)? = null

    @OnLifecycleEvent(value = Lifecycle.Event.ON_CREATE)
    private fun onCreate(owner: LifecycleOwner) {
        getIntentContent = registry.register("intentKey", owner, ActivityResultContracts.StartActivityForResult(), { result ->
                intentCallback?.invoke(result)
            })
    }

    operator fun invoke(intent: Intent?, callback: (ActivityResult) -> Unit) {
        intentCallback = callback
        getIntentContent.launch(intent)
    }
}
```

## 생명주기 옵저버 등록
> BaseActivity 가 있다면 그곳에서 정의해주세요.

```
class BaseActivity: Activity() {
    lateinit var mResultObserver: ResultObserver
    
    /** 
     * ActivityResult 콜백 함수
     * 기존에 사용했던 onActivityResult() 와 동일한 기능을 수행합니다.
     * */
    open fun onLauncherResult(requestCode: Int, resultCode: Int, data: Intent?) {}
    
    /**
     * startActivityForResult 와 동일한 기능을 수행
     * activityResult 콜백을 받고 onLauncherResult 함수를 호출함
     * */ 
    protected fun activityStartForResult(requestCode: Int, intent: Intent) {
        mResultObserver(intent) { result ->
            onLauncherResult(requestCode, result.resultCode, result.data)
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) { 
        mResultObserver = ResultObserver(activityResultRegistry)
        lifecycle.addObserver(mResultObserver)
    }
}
```

## 사용법
```
class MainActivity: BaseActivity() {
    ...
    
    /**
     * 기존의 onActivityResult()에서 이름만 변경하면 됩니다.
     */
    override fun onLauncherResult(requestCode: Int, resultCode: Int, data: Intent?) {
        if (resultCode != RESULT_OK) return

        when (requestCode) {
            1223 -> Log.d(TAG, "내 생일")
        }
    }
    
    private fun start(intent: Intent) {
        activityStartForResult(1223, intent) // 실행
    }
}
```