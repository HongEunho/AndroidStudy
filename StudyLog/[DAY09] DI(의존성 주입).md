# [DAY09] DI(의존성 주입)

### DI(의존성 주입) 이란?

- 컴포넌트 간의 의존관계를 소스코드 내부가 아닌 외부 설정 파일등을 통해 정의되게 하는 디자인 패턴 중 하나
- 객체를 직접 생성하지 않고 외부에서 주입한 객체를 사용하는 방식
- 인스턴스 간 디커플링을 만들어 줌 => 유닛테스트 용이성 증대
- ex) Dagget, Hilt

</br>

### Service Locator 이란?

- 중앙 등록자 'Service Locator'를 통해 요청이 들어왔을 때 특정 인스턴스 반환
- apk 크기, 빌드 속도, 메소드 수 등 복잡한 제약이 있는 경우 사용하기 편함
- ex) Koin ( 경량화 된 DI라고 알려져 있지만, 내부 동작은 Service Locator로 봐도 무방)

</br>

|           | DI (Dependency Injection)              | Service Locator                      |
| --------- | -------------------------------------- | ------------------------------------ |
| 종속성    | 일부 핵심 클래스에 종속성을 주입       | 모든 클래스가 서비스 로케이터에 종속 |
| 호출 방법 | 처음 한번만 호출(명시적인 호출이 없음) | 인젝터를 직접 호출(명시적인 호출)    |
| 의존 관계 | 의존 관계 파악이 쉬움                  | 의존 관계 파악이 어려움              |

</br>



## DI의 기본 및 예시

먼저, DI에 관한 내용은 [안드로이드 공식 문서](https://developer.android.com/training/dependency-injection?hl=ko)를 참고하여 정리하였다.

클래스에는 흔히 다른 클래스의 참조가 필요하다.

다음의 코드를 한 번 살펴보자.

```kotlin
class Car {
    private val engine = Engine()

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.start()
}
```

먼저, 위의 코드처럼 Car 클래스가 Engine 클래스의 참조가 필요한 경우가 있다고 하자.

이처럼 필요한 클래스를 종속 항목이라고 하는데, 위의 코드에서는 Car 클래스가 실행되기 위해서는 Engine 클래스의 인스턴스가 있어야 한다.

</br>

클래스가 필요한 객체를 얻는 세 가지 방법은 다음과 같다.

1. 클래스가 필요한 종속 항목을 구성한다. 위의 코드 처럼 Car는 자체 Engine 인스턴스를 생성한다.
2. 다른 곳에서 객체를 가져온다. Context getter 혹은 getSystemService()와 같은 방식으로.
3. 객체를 매개변수로 제공받는다. 앱은 클래스가 구성될 때 이러한 종속 항목을 제공하거나 각 종속 항목이 필요한 함수에 전달할 수 있다.

3번 설명이 DI (종속성 주입)에 관한 설명이다.

이 방법을 이용하면 클래스 인스턴스가 자체적으로 종속 항목을 얻는 대신 클래스의 종속 항목을 받아서 사용하게 된다.

</br>

하지만 위의 예시 코드는 1번 설명 처럼 Car 클래스가 자체 Engine을 구성하기 때문에 종속성 주입을 한 코드가 아니다.



그렇다면, 다음 코드를 한 번 살펴보자.

```kotlin
class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val engine = Engine()
    val car = Car(engine)
    car.start()
}
```

코드를 보면 알 수 있듯이, Car의 각 인스턴스는 초기화 시 자체 Engine을 구성하는 대신 Engine객체를 생성자의 매개변수로 받는다.

</br>

그래서 메인함수에서 Car을 사용할 때, Car는 Engine에 종속되므로 앱은 Engine 인스턴스를 생성한 후 이를 사용해 Car 인스턴스를 구성한다.

</br>

이러한 DI 기반의 접근방식의 이점은 다음과 같다.

- Car의 재사용 가능. Engine의 다양한 구현을 Car에 전달할 수 있다.

  예를 들어, Car에서 사용할 ElectronicEngine이라는 새로운 Engine 서브 클래스를 정의할 수 있다.

  DI를 사용하게 되면, 업데이트 된 ElectronicEngine 이라는 서브클래스의 인스턴스를 전달하기만 하면 Car는 추가 변경 없이도 계속 작동한다.

- Car의 테스트 편이성. 테스트 더블을 전달하여 다양한 시나리오를 테스트할 수 있다.

  예를 들어 FakeEngine이라는 Engine의 테스트 더블을 생성하여 다양한 테스트에 맞게 구성할 수 있다.

</br>

안드로이드에서 DI를 실행하는 두 가지 주요 방법은 다음과 같다.

- 생성자 삽입 : 위의 코드 예시 처럼. 클래스의 종속 항목을 생성자에 전달한다.

- 필드 삽입(또는 setter 삽입) : Activity나 Fragment처럼 특정 안드로이드 프레임워크 클래스는 시스템에서 인스턴스화 하므로 생성자 삽입이 불가능하다.

  필드 삽입을 하게 되면 종속 항목은 클래스가 생성된 후 인스턴스화 되는데, 아래의 코드가 예시이다.

  ```kotlin
  class Car {
      lateinit var engine: Engine
  
      fun start() {
          engine.start()
      }
  }
  
  fun main(args: Array) {
      val car = Car()
      car.engine = Engine()
      car.start()
  }
  ```

이렇게 라이브러리를 사용하지 않고 다양한 클래스의 종속 항목을 직접 생성, 제공 및 관리하는 방법을 **직접(수동) 종속 항목 삽입** 이라고 한다.

이렇게 수동으로 하게 되면, 종속 항목과 클래스가 많아지면 관리가 복잡해 지게 된다.



그래서, 이러한 프로세스를 자동화하여 해결하는 다음과 같은 라이브러리가 존재한다.

- 런타임 시 종속 항목을 연결하는 리플렉션 기반 솔루션
- 컴파일 타임에 종속 항목을 연결하는 코드를 생성하는 정적 솔루션



우리가 공부할 Dagger는 두 번째에 해당하는 라이브러리 이며, Kotlin 및 안드로이드 용으로 널리 사용되는 DI 라이브러리이다.



## DI의 이점

- 클래스 재사용 가능 및 종속 항목 분리 : 종속 항목 구현을 쉽게 교체할 수 있다. 컨트롤 반전으로 인해 코드 재사용이 개선 되었으며 더 이상 종속 항목 생성 방법을 제어하지는 않지만 모든 구성에서 작동한다.
- 리팩토링 편의성 : 종속 항목은 API 노출 영역의 검증 가능한 요소가 되므로 구현 세부정보로 숨겨지지 않고 객체 생성 타임 또는 컴파일 타임에 확인할 수 있다.
- 테스트 편의성 : 클래스는 종속 항목을 관리하지 않으므로 테스트시 다양한 구현을 전달하여 다양한 모든 사례를 테스트 할 수 있다.



<img width="539" alt="DI" src="https://user-images.githubusercontent.com/46186664/125788706-d65843fa-c6ca-4ca2-a780-9601de3d7428.png">

클래스 간의 종속성은 그래프로 표시할 수 있고 그래프에서 각 클래스는 종속된 클래스에 연결된다.

모든 클래스와 서로의 종속성을 표시하면 어플리케이션 그래프가 구성되는데,

위 그림은 어플리케이션 그래프의 추상적 개념을 보여주는 그림이다.

ViewModel(클래스 A)이 Repository(클래스 B)에 종속하면 A에서 B까지 가리키는 선이 종속성을 나타낸다.

</br>

종속성 삽입을 사용하게 되면 클래스를 쉽게 연결할 수 있고, 테스트를 위해 구현을 교체할 수 있다.

예를 들어 Repository에 종속된 ViewModel을 테스트 할 때, face나 Mock으로 Repository의 다른 구현을 전달하여 다른 사례를 테스트할 수 있다. 



## 수동 종속성 삽입(수동 DI)

안드로이드 설계 샘플 코드를 살펴보면, Repository는 LocalDataSource와 RemoteDataSource에 종속하는 것을 알 수 있다.

실제 코드의 Repository 클래스를 보게 되면 다음과 같은 구조로 설계 되어 있다.

```kotlin
    class UserRepository(
        private val localDataSource: UserLocalDataSource,
        private val remoteDataSource: UserRemoteDataSource
    ) { ... }

    class UserLocalDataSource { ... }
    class UserRemoteDataSource(
        private val loginService: LoginRetrofitService
    ) { ... }
```

LoginActivity 클래스는 다음과 같은 구조로 되어 있다고 하자.

```kotlin
    class LoginActivity: Activity() {

        private lateinit var loginViewModel: LoginViewModel

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)

            // In order to satisfy the dependencies of LoginViewModel, you have to also
            // satisfy the dependencies of all of its dependencies recursively.
            // First, create retrofit which is the dependency of UserRemoteDataSource
            val retrofit = Retrofit.Builder()
                .baseUrl("https://example.com")
                .build()
                .create(LoginService::class.java)

            // Then, satisfy the dependencies of UserRepository
            val remoteDataSource = UserRemoteDataSource(retrofit)
            val localDataSource = UserLocalDataSource()

            // Now you can create an instance of UserRepository that LoginViewModel needs
            val userRepository = UserRepository(localDataSource, remoteDataSource)

            // Lastly, create an instance of LoginViewModel with userRepository
            loginViewModel = LoginViewModel(userRepository)
        }
    }
```

하지만, 이러한 접근 방식은 다음과 같은 문제가 발생한다.

- **상용구 코드가 많다**. 코드의 다른 부분에서 LoginViewModel의 다른 인스턴스를 만드려면 코드가 중복될 수 있다.
- **종속성은 순서대로 선언해야 한다**. UserRepository를 만드려면 LoginViewModel 전에 인스턴스화 해야 한다.
- **객체를 재사용하기가 어렵다**. 여러 기능에 걸쳐 UserRepository를 재사용하려면 싱글톤 패턴을 따르게 해야한다.  하지만 모든 테스트가 동일한 싱글톤 인스턴스를 공유하므로 테스트 하기가 더 어려워진다.

</br>

여기서 객체 재사용 문제를 해결하려면 다음과 같이 종속 항목을 가져오는데 사용하는 **종속 항목 컨테이너 클래스**를 만들면 된다.

```kotlin
    // Container of objects shared across the whole app
    class AppContainer {

        // Since you want to expose userRepository out of the container, you need to satisfy
        // its dependencies as you did before
        private val retrofit = Retrofit.Builder()
                                .baseUrl("https://example.com")
                                .build()
                                .create(LoginService::class.java)

        private val remoteDataSource = UserRemoteDataSource(retrofit)
        private val localDataSource = UserLocalDataSource()

        // userRepository is not private; it'll be exposed
        val userRepository = UserRepository(localDataSource, remoteDataSource)
    }
```

이러한 종속 항목은 전체 어플리케이션에 걸쳐 사용되므로 

다음과 같이모든 활동에서 사용할 수 있는 일반적인 위치, 즉 어플리케이션 클래스에 배치해야 한다.

```kotlin
    // Custom Application class that needs to be specified
    // in the AndroidManifest.xml file
    class MyApplication : Application() {

        // Instance of AppContainer that will be used by all the Activities of the app
        val appContainer = AppContainer()
    }
```

이렇게 만들어놓으면, 다음과 같이 어플리케이션에서 AppContainer 인스턴스를 가져와서 공유 UserRepository 인스턴스를 획득할 수 있다.

```kotlin
    class LoginActivity: Activity() {

        private lateinit var loginViewModel: LoginViewModel

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)

            // Gets userRepository from the instance of AppContainer in Application
            val appContainer = (application as MyApplication).appContainer
            loginViewModel = LoginViewModel(appContainer.userRepository)
        }
    }
    
```

</br>

하지만 이러한 방식으로는 싱글톤 UserRepository를 얻지 못한다. 

대신 위 그림의 그래프의 객체가 포함된 모든 활동에서 공유된 AppContainer가 있고 

다른 클래스에서 사용할 수 있는 이러한 객체의 인스턴스를 만든다.

</br>

LoginViewModel이 더 많은 곳에서 필요하면 한 곳에서 LoginViewModel의 인스턴스를 만드는 것이 좋다. 만든 LoginViewModel을 컨테이너로 이동시키고 그 object의 새 객체에 팩토리를 제공할 수 있다.

LoginViewModelFactory의 코드는 다음과 같다.

```kotlin
    // Definition of a Factory interface with a function to create objects of a type
    interface Factory {
        fun create(): T
    }

    // Factory for LoginViewModel.
    // Since LoginViewModel depends on UserRepository, in order to create instances of
    // LoginViewModel, you need an instance of UserRepository that you pass as a parameter.
    class LoginViewModelFactory(private val userRepository: UserRepository) : Factory {
        override fun create(): LoginViewModel {
            return LoginViewModel(userRepository)
        }
    }
```

이렇게 만든 Factory를 AppContainer에 포함하여 LoginActivity에서 사용하게 할 수 있다.

```kotlin
    // AppContainer can now provide instances of LoginViewModel with LoginViewModelFactory
    class AppContainer {
        ...
        val userRepository = UserRepository(localDataSource, remoteDataSource)

        val loginViewModelFactory = LoginViewModelFactory(userRepository)
    }

    class LoginActivity: Activity() {

        private lateinit var loginViewModel: LoginViewModel

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)

            // Gets LoginViewModelFactory from the application instance of AppContainer
            // to create a new LoginViewModel instance
            val appContainer = (application as MyApplication).appContainer
            loginViewModel = appContainer.loginViewModelFactory.create()
        }
    }
```



이 방식은 아까 위에서 설명한 방식보다는 좋지만, 여전히 다음과 같은 문제를 고려해야 한다.

- AppContainer를 직접 관리하여 모든 종속 항목의 인스턴스를 수동으로 만들어야 한다
- 여전히 상용구 코드가 많다. 객체를 재사용할지에 따라 수동으로 팩토리나 매개변수를 만들어야 한다.



## 어플리케이션 흐름에서의 종속성 관리

프로젝트에 기능을 더 많이 포함하려고 한다면 AppContainer는 더욱 더 복잡해진다.

그러면 다음과 같은 상황이 발생할 수도 있다.

- Flow가 다양하면 객체가 Flow 범위 안에만 있기를 원할 수 있다.

  예를 들어, 로그인 Flow에서만 사용되는 사용자 이름과 비밀번호로 구성될 수 있는 LoginUserData를 만들 때, 개발자는 다른 사용자의 이전 로그인 Flow의 데이터를 유지하지 않으려고 한다 ( 즉, 모든 새로운 Flow에 새 Instance를 원한다. )

  이 문제에 대한 해답은 아래의 코드에서 살펴보자. 

  (AppContainer 내에 FlowContainer 객체를 생성하면 된다.)

- 어플리케이션 그래프와 Flow 컨테이너를 최적화 하는 것도 어려울 수 있다.

  흐름에 따라 필요하지 않은 인스턴스를 삭제해야 한다.

</br>

그럼, 액티비티 하나(LoginActivity)와 여러 프레그먼트(LoginUsernameFragment, LoginPasswordFragment)로 구성된 로그인 Flow를 가정해보자.

</br>

이 뷰들을 다음과 같이 만드려고 한다.

1. 로그인 flow가 완료될 때 까지 공유해야 하는 동일한 LoginUserData 인스턴스에 액세스 한다.
2. flow가 다시 시작되면 LoginUserData의 새 인스턴스를 만든다.

이렇게 만들기 위해, LoginContainer를 이용해보자.

이 때, 이 LoginContainer는 로그인 flow가 시작될 때 만들어지고 flow가 끝날 때 메모리에서 삭제되어야 한다.

</br>

그럼 이제 위의 코드 예시에서, LoginContainer를 추가해보자.

```kotlin
    class LoginContainer(val userRepository: UserRepository) {

        val loginData = LoginUserData()

        val loginViewModelFactory = LoginViewModelFactory(userRepository)
    }

    // AppContainer contains LoginContainer now
    class AppContainer {
        ...
        val userRepository = UserRepository(localDataSource, remoteDataSource)

        // LoginContainer will be null when the user is NOT in the login flow
        var loginContainer: LoginContainer? = null
    }
```

이렇게 Flow 관련 컨테이너가 있으면 Flow 컨테이너를 언제 만들고 삭제할 지를 판단해야 한다.

LoginActivity는 onCreate()에서 인스턴스를 만들고 onDestroy()에서 삭제할 수 있다.

</br>

그럼, 다음과 같이 LoginActivity를 구성할 수 있겠다.

```kotlin
    class LoginActivity: Activity() {

        private lateinit var loginViewModel: LoginViewModel
        private lateinit var loginData: LoginUserData
        private lateinit var appContainer: AppContainer

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            appContainer = (application as MyApplication).appContainer

            // Login flow has started. Populate loginContainer in AppContainer
            appContainer.loginContainer = LoginContainer(appContainer.userRepository)

            loginViewModel = appContainer.loginContainer.loginViewModelFactory.create()
            loginData = appContainer.loginContainer.loginData
        }

        override fun onDestroy() {
            // Login flow is finishing
            // Removing the instance of loginContainer in the AppContainer
            appContainer.loginContainer = null
            super.onDestroy()
        }
    }
    
```

이렇게 한다면, LoginActivity와 마찬가지로 LoginFragment도 AppContainer에서 LoginContainer에 액세스 하여 공유 LoginUserData 인스턴스를 사용할 수 있다.

</br>

지금까지 DI에 대한 전체적인 내용을 살펴봤는데, 이렇게 수동으로 DI를 구성하게 되면 오류가 발생할 수도 있고 앱의 크기가 커질수록 처리해줘야 할 것들이 많아진다.

따라서, 다음 문서에서는 이를 자동화해주는 도구인 Dagger에 대해 알아보자.
