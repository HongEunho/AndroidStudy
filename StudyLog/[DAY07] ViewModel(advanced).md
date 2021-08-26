# ViewModel (advanced)

[지난 ViewModel문서에 이어서 추가적인 내용을 이어가고자 한다](https://oss.navercorp.com/ghdwns315/AndroidGoogleCalendar/blob/master/StudyLog/%5BDAY04%5D%20ViewModel%2C%20ListAdapter%2BDiffUtil.md)

ViewModel은 View를 참조하지 않는 것이 최선이지만

Context가 꼭 필요하다면 Application 객체를 생성자로 받아 사용할 수 있다.

</br>

SavedStateHandle

SavedStateHandle을 이용하지 않으면 

SavedInstanceState로 Activity가 재시작 될 때 UI 데이터 유지가 가능하다.

이를 위해 SavedInstanceState와 관련한 많은 코드 및 커스텀 ViewModelFactory를 구현해야한다.

</br>

하지만 SavedStateHandle을 이용하면 이러한 보일러플레이트 코드를 줄여준다.

SavedStateHandle을 ViewModel 생성자의 인자로 추가하면

커스텀 ViewModelFactory를 사용하지 않아도 된다.

</br>

이는 데이터를 맵으로 관리하며 LiveDate도 맵으로 관리하기 때문에 LiveData를 사용할 때 편리하다.

</br>

**ViewModel에서 코루틴 사용하기**

ViewModel 클래스 안에 다음과 같은 코드가 있다고 가정해보자.

```kotlin
init {
    CoroutineScope(Dispatchers.Main.immediate).launch {
        delay(3000)
        while(true){
            delay(1000)
            increaseCount()
        }
    }
}
```

이 코드의 결과는 앱 시작 3초 후에 1초마다 숫자 카운트를 +1 하게 되는데,

앱을 finish해서 viewModel이 cleared 된 상황에서도 코루틴은 계속 실행이 되면서 숫자 카운트가 계속 +1이 된다.

이는 메모리 누수로 이어질 수 있게 된다.

</br>

이를 해결하기 위한 첫 번째 방법은 **Job**을 이용하는 것이다.

```kotlin
init {
        job = CoroutineScope(Dispatchers.Main.immediate).launch {
            delay(3000)
            while(true){
                delay(1000)
                increaseCount()
            }
        }
    }
...
override fun onCleared() {
    job.cancel()
}
```

이렇게 job을 이용하여 viewModel이 cleared될 때 cancel을 해주면 코루틴을 멈출 수 있다.

</br>

하지만, 이 방법은 viewModel이 조금만 복잡해져도 비동기 코드가 많아지면서 cancel() 처리를 해주어야 하는 job이 굉장히 많아질 것이기 때문에 비추천 하는 방법이다.

</br>

그래서 이용할 두 번째 방법이 **ViewModelScope** 이다.

```kotlin
init {
    viewModelScope.launch {
        delay(3000)
        while(true){
            delay(1000)
            increaseCount()
        }
    }
}
```

이렇게 viewModelScope를 이용하게 되면 job 관리 없이

viewModel이 Cleared 되더라도 알아서 코루틴을 종료시킨다.

</br>

viewModelScope를 통해 얻는 장점은?

만약, viewModel이 아닌 Activity나 Fragment에서 LifecycleScope를 통해

```kotlin
lifecycleScope.launch {
	delay(3000)
	while(true){
		delay(1000)
		viewModel.increaseCount()
	}
}
```

이 코드를 실행했다면, 코루틴이 activity의 Lifecycle에 맞춰 실행되기 때문에

Activity가 재생성 되는 경우에는 그만큼 delay가 생기게 된다.

예를 들어, 위 코드를 실행하고 빠른 시간동안 계속 화면회전을 하게 되면

3초 이상의 시간이 지난 후에 코루틴이 실행되게 될 것이다.

</br>

하지만 ViewModelScope를 이용하게 되면, 아무리 화면회전을 해서 Activity가 재생성되더라도

3초 후에 코루틴이 실행되게 된다.

</br>

그래서 만약, 액티비티 lifecyclescope에서 API를 호출하여 결과값을 받는 상황에서

사용자가 화면회전 등을 하게 되어 Activity를 재생성하게 된다면

다시 실행되기 때문에 그만큼 결과값을 받는 시간이 지연되게 된다.

</br>

하지만, ViewModelScope에서 API를 호출하여 결과값을 받는다면

사용자가 화면회전을 하더라도 기존에 시작된 비동기 코드가 종료되지 않기 때문에 정해진 시간에 결과값을 받을 수 있게 된다.

