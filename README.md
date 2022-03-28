# Kotlin Flow API

```
22.03.23 스터디 시작

22.03.26 스터디 내용 공유
```

### 관련 문서
https://developer.android.com/kotlin/flow?hl=ko

### 정리자료 
https://ivy-hedgehog-78c.notion.site/Flow-API-Kotlin-8e14ebbd020247cfb1dfd1831335d181

### StateFlow

- StateFlow 는 값이 업데이트 된 경우에만 반환하고 동일한 값을 반환하지 않는다.
- Flow 는 일반적으로 Cold Stream / StateFlow 는 Hot Stream 이다. 마지막 값의 개념이 있으며 생성하자마자 활성화 된다.
- 즉 , 항상 활성 상태이고 메모리 내에 있으며 GC 루트에서 달리 참조가 없는 경우에만 GC에 사용할 수 있다.
- StateFlow 는 constructor 로 초기값을 필요로 한다.

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        // Start a coroutine in the lifecycle scope
        lifecycleScope.launch {
            // repeatOnLifecycle launches the block in a new coroutine every time the
            // lifecycle is in the STARTED state (or above) and cancels it when it's STOPPED.
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                // Trigger the flow and start listening for values.
                // Note that this happens when lifecycle is STARTED and stops
                // collecting when the lifecycle is STOPPED
                latestNewsViewModel.uiState.collect { uiState ->
                    // New value received
                    when (uiState) {
                        is LatestNewsUiState.Success -> showFavoriteNews(uiState.news)
                        is LatestNewsUiState.Error -> showError(uiState.exception)
                    }
                }
            }
        }
    }

```

- UI 업데이트를 해야 하는 경우 launch 또는 launchIn 확장 함수로 UI 에서 직접 흐름을 수집하면 안된다. 이러한 함수는 뷰가 표시되지 않는 경우에도 이벤트를 처리한다. 이 동작으로 인해 앱이 다운될 수 있다. 이를 방지하려면 repeatOnLifecycle API 를 사용해야한다.

- 뷰가 STOPPED 되면 LiveData 는 자동으로 소비자를 등록취소
- StateFlow 는 흐름에서 수집하는 경우 자동으로 수집을 중지하지 않는다. 하지만 LiveData 와 동일한 동작을 실행하려면 Lifecycle.repeatOnLifecycle 블록에서 흐름을 수집해야함

왜 StateFlow ?

- LiveData 는 Android Lifecycle 에 따라 달라지기 때문에 View / ViewModel 외부에서 사용하기 적합하지 않다.
- StateFlow 는 LiveData 와 같은 동작을 하지만 다양한 연산자와 더 좋은 성능을 가지고 있다고함
- StateFlow API 는 LiveData 와 거의 동일하며 SharedFlow 는 고급 사용 사례를 위한 유연성을 제공한다.
- StateFlow 는 앱이 백그라운드로 이동하면 자동 구독 / 구독 취소 동선에서 모든 이점을 가져갈 수 있다.
