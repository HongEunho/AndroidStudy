# [DAY06] MVP패턴

지금까지는 View( Activity, Fragment )에서 View의 동작과, 데이터 처리까지 모두 작성하는 방법으로 안드로이드 앱을 제작했다.



하지만 이렇게 작성하게 되면 코드 작성은 쉬울 수 있지만, 여러 사람들과 협업을 하게 되면 코드 가독성이나 관리, 로직 구현이 굉장히 까다로워진다.

또한 하나의 class에 모든 처리를 위한 수많은 코드들이 들어가게 되어 스파게티 코드로 변환될 가능성이 있다.

그리고 UI에서 모든 걸 처리하기 때문에 비즈니스 로직에 따른 UI의 변화들을 직접 개발자가 바꿔줘야 하는 까다로움이 존재한다.

이로 인해 테스트 코드의 작성이 어려워지게 되며 유지보수에 굉장히 큰 어려움이 따를 수 있다.  

</br>

이러한 단점을 극복하고자 나온 패턴이 바로 MVP 패턴과 MVVM 패턴이다.

MVP패턴 또한 장단점을 갖고 있으며, MVP패턴의 단점을 극복하고자 발전한 패턴이 MVVM 패턴이기 때문에 먼저 MVP 패턴부터 살펴보도록 하자.

</br>

## MVP 패턴에 대해

먼저, MVP패턴은 [구글에서 제공한 아키텍쳐 설계 코드](https://github.com/android/architecture-samples)를 기반으로 공부하였다.

MVC패턴과 MVP패턴의 차이점을 먼저 그림으로 보고, 정리해보고자 한다.

</br>

안드로이드 MVC 패턴

<img width="539" alt="KakaoTalk_Photo_2021-07-12-23-37-32" src="https://media.oss.navercorp.com/user/25503/files/4e063400-e36a-11eb-88ed-6d861c6da1af">

</br>

MVP패턴

<img width="539" alt="KakaoTalk_Photo_2021-07-12-23-37-32" src="https://media.oss.navercorp.com/user/25503/files/d84e9800-e36a-11eb-80bc-02d029deb8f7">

MVC패턴에서는 Model과 Activity( View + Controller )로 구성이 되어있고

MVP 패턴은 Model과 View, Presenter로 구성되어 있다.

</br>

MVC패턴의 Activity에서 UI관련 처리부터 데이터 처리까지 모든 기능을 구현했다면

MVP패턴은 이들을 View와 Presenter로 분할한 것이다.

</br>

여기서 Presenter의 역할은 View와 Model을 분리하여, 서로간에 상호작용을 Presenter가 담당함으로써 서로간의 영향을 최소화 하는 것이다.

</br>

작동과정은 다음과 같다.

1. View에서 사용자 이벤트가 발생하게 되면 이 이벤트를 Presenter로 전달하게 되고
2. Presenter는 이 이벤트를 받아 이에 필요한 데이터를 Model로 요청하게 된다.
3. Model은 Presenter의 호출에 따라 필요한 데이터를 Presenter로 전달하게 되고
4. Presenter는 Model로부터 받은 데이터를 가공하여 View로 전달하게 된다.
5. View는 이것들을 바탕으로 UI를 갱신하게 된다.

</br>

그럼 이제, 구글에서 제공한 샘플 코드를 가지고 MVP 패턴을 살펴보자.

먼저 각각의 Task마다 View, Presenter, Contract가 작성된 것을 확인할 수 있는데

여기서 Contract는 View와 Presenter에 대한 interface를 작성하는 부분이다.

그래서 View는 Contract.View을 상속받아서 구현하게 되고

Presenter는 Contract.Presenter을 상속받아서 구현하게 된다.

</br>

Presenter에는 필요한 데이터를 Model로 요청하는 함수와 받아온 데이터를 View로 전달하는 코드들이 구현되어 있고

View에는 UI의 갱신과, Presenter로 이벤트 전달을 하는 코드들이 구현되어 있음을 알 수 있다.

</br>

하지만 코드를 뜯어보면 알 수 있듯이, View와 Presenter 간의 의존성이 높음을 알 수 있다.

View와 Model은 분리가 되었지만, 앱의 크기가 거대해지면서 Model에서 받아온 대량의 데이터에 따라 각각의 다른 UI를 표시해야 한다면, 각각의 조건에 맞는 로직을 추가해주어야 하고, 그렇게 되면 추가된 만큼의 코드가 생겨나면서 로직이 굉장히 거대해지기 때문에 코드의 길이 또한 상당히 길어지게 된다.

그렇게 되면 위에서 설명한 것 처럼 코드의 가독성이 떨어지며 테스트 또한 어려워지게  된다.

또한, 각각의 조건에 맞게 UI가 올바르게 변경되었는지 확인작업도 필요할 것이다.

</br>

그래서 이러한 단점을 극복하기 위해 MVVM 패턴과 Usecase 등이 등장하게 되었다.



이어서..[MVVM](https://oss.navercorp.com/ghdwns315/AndroidGoogleCalendar/blob/master/StudyLog/%5BDAY06%5D%20MVVM패턴.md)
