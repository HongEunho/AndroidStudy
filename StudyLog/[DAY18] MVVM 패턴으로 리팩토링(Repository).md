# MVVM 패턴으로 리팩토링

### DataSource와 Repository의 구현

- 먼저, 액티비티에서 바로 객체를 만들어서 사용했던 API 통신 작업과 LocalDB 작업 관련 함수를

  DataSource 인터페이스와 이를 상속받은 Repository에서 구현하였다.

<img width="643" alt="스크린샷 2021-08-01 오후 10 57 06" src="https://user-images.githubusercontent.com/46186664/127774786-29d45a18-e6a4-4f5b-8808-9afddf5400a9.png">
<img width="920" alt="스크린샷 2021-08-01 오후 10 57 30" src="https://user-images.githubusercontent.com/46186664/127774792-fbf62d14-7f8f-4783-b901-7cc9dd952dc0.png">

이렇게 구조를 잡음으로써, 앞으로 데이터 관련 작업(비즈니스 로직)의 틀을 잡아주었다.

레포지토리를 구현함으로써 ViewModel과 Model의 상호작용에 대한 틀을 잡아주었고,

현재 구현된 부분 이외에 더 구현해야 할 기능이 생기면 DataSource에 함수명을 기입하고 Repository에서 이를 구현하게 함으로써 깔끔하게 코드관리를 할 수 있다.

또한, 지금은 1인 프로젝트를 진행하고 있지만 만약 팀원들과 함께하는 프로젝트라면

이 틀을 통해 원활한 의사소통과 기능구현이 가능하게 한다.

</br>

또, 테스트 과정에서 현재 Repository를 TestRepository로 변경해야 한다면, 

현재의 CalendarRepository는 코드수정 없이 그대로 두고 

새로 만든 TestRepository를 테스트 과정에서 주입받게 함으로써 간단하게 Repository만 바꿔치기 하여

원활한 테스트가 가능하게 한다.



</br>

### View, ViewModel, Model의 분리 및 Fragment 구현

지금까지 하나의 액티비티에서 모든 기능을 구현했다면,

이젠 이렇게 각각의 패키지마다 Activity와 Fragment, ViewModel을 설정하여

각각의 클래스들이 각자의 역할에 충실할 수 있도록 구현하였다.

</br>

<img width="404" alt="스크린샷 2021-08-01 오후 10 57 56" src="https://user-images.githubusercontent.com/46186664/127774807-ca7b5339-e2c5-4727-8268-d90e23e412cb.png">

</br>

또한, Activity에 모두 구현했던 코드들을 Fragment로 분산시킴으로써

CalendarActivity 안에서, Calendar관련 화면들이 Fragment를 통해 자유자재로 전환될 수 있도록 하였고

Fragment 안에서는 비즈니스 로직의 결과를 토대로 Ui 관련 작업만 할 수 있도록 하였고

직접적인 데이터 관련 작업은 ViewModel을 통해서만 할 수 있도록 구현하였다.

<img width="796" alt="스크린샷 2021-08-01 오후 10 58 02" src="https://user-images.githubusercontent.com/46186664/127774813-7c2a2c35-20e1-45d6-b220-7be09ab5c6a8.png">
</br>
