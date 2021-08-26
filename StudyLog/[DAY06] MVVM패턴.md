# [DAY06] MVVM패턴

## MVVM 패턴

<img width="539" alt="KakaoTalk_Photo_2021-07-12-23-37-32" src="https://media.oss.navercorp.com/user/25503/files/0502af80-e36b-11eb-8a4a-39a2a4468e5f">

MVVM패턴은 MVP 패턴의 Presenter의 역할을 ViewModel이 대신하게 된다.

MVP패턴에서는 View와 Presenter간의 관계가 1:1이었지만

MVVM패턴에서 View와 ViewModel의 관계는 독립적이다.

</br>

따라서, MVP패턴의 View와 Presenter간의 의존성 문제를 어느정도 해소한 패턴이라고 볼 수 있다.

</br>

MVVM패턴 자체만 놓고 보면, LiveData를 이용하여 View에서 ViewModel을 Observer 함으로써 ViewModel의 데이터 값에 따라 View(UI)가 자동으로 갱신되게 하지만

여전히 ViewModel의 값에 따라 View에서 setText를 하여 View를 변경하기 때문에

MVVM패턴에서도 View와 ViewModel의 의존성이 남아있다.

그래서 이 의존성 해결을 위해 Databinding을 이용하게 된다.

</br>

// ToDo MVVM 패턴의 동작과정, 장단점, 코드 뜯어보기

