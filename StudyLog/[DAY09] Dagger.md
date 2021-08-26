# [DAY09] Dagger

이전 문서에서 소개했던 대로 Dagger는 자동 DI 도구로써

반복되고 오류가 발생하기 쉬운 상용구 코드를 작성하지 않아도 된다.

그리고 다음과 같은 기능을 제공한다.

- 수동 DI 섹션에서 수동으로 구현한 AppContainer 코드를 생성한다.

- 어플리케이션 그래프에서 사용할 수 있는 클래스 팩토리를 만든다.

- Scope를 사용하여 object를 구성하는 방법에 따라 종속 항목을 재사용하거나 새 인스턴스를 만든다.

- Dagger 하위 컴포넌트를 사용하여 이전 섹션의 LoginFlow에서와 같이 특정 Flow의 컨테이너를 만든다.

  이렇게 하면 더 이상 필요하지 않은 객체를 메모리에서 해제함으로써 앱 성능이 향상된다.

</br>

개발자가 클래스의 종속 항목을 선언하고 어노테이션을 사용하여 종속 항목을 충족하는 방법을 지정하기만 하면 Dagger는 빌드 타임에 자동으로 이러한 모든 일들을 완료한다.

</br>

그럼 간단한 Dagger 사용 예시를 보면서 이해해보자.

### Dagger의 팩토리 생성
<img width="539" alt="Dagger" src="https://user-images.githubusercontent.com/46186664/125789149-5c2644b3-d595-49a0-91b0-5ab960980e6b.png">

위의 다이어그램에 나와있는 UserRepository 클래스의 간단한 팩토리를 만들어보자.

```kotlin
    class UserRepository(
        private val localDataSource: UserLocalDataSource,
        private val remoteDataSource: UserRemoteDataSource
    ) { ... }
```

이 코드를, Dagger가 UserRepository를 생성하는 방법을 알 수 있도록 @Inject 어노테이션을 UserRepository 생성자에 추가하여 다음과 같이 짜보자.

```kotlin
    // @Inject lets Dagger know how to create instances of this object
    class UserRepository @Inject constructor(
        private val localDataSource: UserLocalDataSource,
        private val remoteDataSource: UserRemoteDataSource
    ) { ... }
```

위의 코드를 분석해보면 다음과 같다.

1. `@Inject` 어노테이션이 달린 생성자를 사용하여 UserRepository 인스턴스를 만드는 방법을 알려줌
2. 종속 항목은 UserLocalDataSource 와 UserRemoteDataSource 임을 알려줌

여기까지만 살펴보면, Dagger는 이제 UserRepository의 인스턴스를 만드는 방법을 알고 있다.

</br>

하지만 아직 종속 항목을 만드는 방법은 모른다.

그럼 다음과 같이 다른 클래스에도 어노테이션을 지정하여 Dagger가 그 클래스를 만드는 방법을 알게하자.

```kotlin
    // @Inject lets Dagger know how to create instances of these objects
    class UserLocalDataSource @Inject constructor() { ... }
    class UserRemoteDataSource @Inject constructor() { ... }
```



### Dagger Component

Dagger는 종속 항목이 필요할 때 종속 항목을 가져올 위치를 찾는 데 사용할 수 있는 프로젝트의 종속 항목 그래프를 만들 수 있다.

Dagger가 이렇게 하도록 하려면, 인터페이스를 만들고 `@Component`로 어노테이션을 지정해야 한다.

그럼 Dagger는 수동 종속 항목 삽입시와 마찬가지로 컨테이너를 만든다.

</br>

`@Component` 인터페이스 내에서 필요한 클래스의 인스턴스를 반환하는 함수를 정의할 수 있다.

@`Component`는 노출하는 object을 충족하는데 필요한 모든 종속 항목이 있는 컨테이너를 생성하도록 Dagger에게 지시한다.

```kotlin
    // @Component makes Dagger create a graph of dependencies
    @Component
    interface ApplicationGraph {
        // The return type  of functions inside the component interface is
        // what can be provided from the container
        fun repository(): UserRepository
    }
```

그래서 위 코드를 빌드하게 되면, Dagger는 자동으로 ApplicationGraph 인터페이스의 구현,

즉 DaggerApplicationGraph를 생성한다.

Dagger는 어노테이션 프로세서를 사용하여 하나의 진입점(UserRepository 인스턴스 가져오기)으로 세 클래스(UserRepository, UserLocalDatasource 및 UserRemoteDataSource) 간의 관계로 구성된

종속 항목 그래프를 만든다.

</br>

그리고 다음과 같이 사용할 수 있다.

```kotlin
    // Create an instance of the application graph
    val applicationGraph: ApplicationGraph = DaggerApplicationGraph.create()
    // Grab an instance of UserRepository from the application graph
    val userRepository: UserRepository = applicationGraph.repository()
```

Dagger는 요청될 때 마다 아래와 같이 UserRepository의 새 인스턴스를 만든다.

```kotlin
    val applicationGraph: ApplicationGraph = DaggerApplicationGraph.create()

    val userRepository: UserRepository = applicationGraph.repository()
    val userRepository2: UserRepository = applicationGraph.repository()

    assert(userRepository != userRepository2)
```

</br>

때로는 다음과 같은 이유 때문에 컨테이너에 종속 객체의 고유한 인스턴스가 있어야 한다.

- 동일한 LoginUserData를 사용하는 로그인 flow의 여러 ViewModel 객체와 같이, 

  동일한 인스턴스를 공유하기 위해 이 클래스를 종속 클래스로 보유하는 다른 object를 원한다.

- 객체를 생성하는 데 비용이 많이 든다. 따라서 인스턴스를 종속 객체로 선언할 때 마다 새 인스턴스를 생성하지 않으려고 한다. ( 예를 들면, JSON Parser 처럼 )

</br>

위의 코드에서는 UserRepository를 요청할 때 마다 항상 동일한 인스턴스를 얻도록 그래프에서 사용 가능한 고유한 UserRepository 인스턴스를 가지려고 할 수 있다.

이는 더 복잡한 어플리케이션 그래프가 있는 실제 어플리케이션에서 UserRepository에 따라 여러 ViewModel 객체를 가질 수 있고, 

UserRepository를 제공해야 할 때 마다 UserLocalDataSource 혹은 UserRemoteDataSource의 새 인스턴스를 생성하지는 않으려고 하므로 유용하다.



### Scoping With Dagger ( Dagger로 범위 지정)

범위 어노테이션을 사용하여 객체의 Lifetime을 컴포넌트의 Lifetime으로 제한할 수 있다.

즉, object를 제공해야 할 때 마다 종속체의 동일한 인스턴스를 사용한다.



ApplicationGraph의 Repository를 요청할 때, UserRepository의 고유한 인스턴스를 가지려면

`@Component`인터페이스 및 UserRepository에 동일한 Scope Annotation을 사용한다.

다음과 같이 Dagger에서 사용하는 javax.inject 패키지와 함께 이미 제공된 `@Singletone` 어노테이션을 사용할 수 있다.

```kotlin
    // Scope annotations on a @Component interface informs Dagger that classes annotated
    // with this annotation (i.e. @Singleton) are bound to the life of the graph and so
    // the same instance of that type is provided every time the type is requested.
    @Singleton
    @Component
    interface ApplicationGraph {
        fun repository(): UserRepository
    }

    // Scope this class to a component using @Singleton scope (i.e. ApplicationGraph)
    @Singleton
    class UserRepository @Inject constructor(
        private val localDataSource: UserLocalDataSource,
        private val remoteDataSource: UserRemoteDataSource
    ) { ... }
```

</br>

또는 맞춤 Scope Annotation을 만들어 사용할 수 있다.

```kotlin
    // Creates MyCustomScope
    @Scope
    @MustBeDocumented
    @Retention(value = AnnotationRetention.RUNTIME)
    annotation class MyCustomScope
```

이렇게 만든 후, 위의 코드에 적용하면 다음과 같은 코드가 탄생한다.

```kotlin
    @MyCustomScope
    @Component
    interface ApplicationGraph {
        fun repository(): UserRepository
    }

    @MyCustomScope
    class UserRepository @Inject constructor(
        private val localDataSource: UserLocalDataSource,
        private val service: UserService
    ) { ... }
    
```



두 코드 모두 `@Component` 인터페이스에 주석을 지정하는데 사용된 것과 동일한 Scope가 객체에 제공된다.

따라서, applicationGraph.repository()를 호출할 때 마다 동일한 UserRepository 인스턴스를 얻게 된다.

```kotlin
    val applicationGraph: ApplicationGraph = DaggerApplicationGraph.create()

    val userRepository: UserRepository = applicationGraph.repository()
    val userRepository2: UserRepository = applicationGraph.repository()

    assert(userRepository == userRepository2)
```
