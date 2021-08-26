# [DAY05] Coroutine

## 학습내용

Coroutine의 개념과 용어, 사용법 등을 숙지하기

- Coroutine vs Thread를 통해 Coroutine을 사용하는 이유 알아보기
- Coroutine Dispatcher
- Coroutine Context
- Coroutine Scope
- Coroutine Builder + Job
- suspend(일시 중단 함수)
- Coroutine 직접 사용해보기



## Coroutine vs Thread

### Thread

Task단위 = Thread

- 각 작업에 Thread를 할당

- 각 Thread는 자체 Stack 메모리를 가지며, JVM Stack 영역 차지

Context Switching

- Blocking : Thread1이 Thread2의 결과가 나올 때 까지 기다려야 한다면 Thread1은 Blocking되어 사용하지 못함



### Coroutines

Task 단위 = Object(Coroutine)

- 각 작업에 Object(Coroutine)을 할당
- Coroutine은 객체를 담는 JVM Heap에 적재

Context Switching

- No Context Switching

- 코드를 통해 Switching 시점을 보장함

- Suspend is NonBlocking : Coroutine1이 Coroutine2의 결과가 나올 때 까지 기다려야 한다면 Coroutine1은 Suspend 되지만, Coroutine1을 수행하던 Thread는 유효함

  Coroutine2도 Coroutine1과 동일한 Thread에서 실행할 수 있음



## Coroutine Dispatcher

- 코루틴을 시작하거나 재개할 스레드를 결정하기 위한 도구
- 모든 Dispatcher는 CoroutineDispatcher 인터페이스를 구현해야 함



## Coroutine Context

Coroutine을 운동 선수에 비유하고, CoroutineContext를 경기장으로 비유해보자.

축구선수는 축구장에 배치되어야 할 것이고, 농구선수는 농구장에 배치되어야 할 것이다.

이처럼, Coroutine의 실행 목적에 맞게 실행될 특정 ThreadPool을 지정해 주어야 한다.



CoroutineContext에는 Dispatchers.Main / Dispatcher.IO / Dispatchers.Default ... 가 있는데

- Main에는 UI 관련 작업이 모여있는 쓰레드풀이며
- IO에는 데이터를 읽고 쓰는 작업이 모여있는 쓰레드풀이며
- Default는 기본 쓰레드풀로, CPU 사용량이 많은 작업에 적합한 쓰레드풀이다.

그래서 각각의 Coroutine의 목적에 맞게 CoroutineContext를 지정해 주면 된다.



## Coroutine Scope

위의 Coroutine Context로 Coroutine이 어디서 실행될지를 결정했다면,

이제 Coroutine을 제어할 수 있는 범위( 즉, Scope ) 를 지정해주어야 한다.



제어는 작업을 취소시키거나, 작업이 끝날 때 까지 기다리는 것을 의미하는데

이러한 제어를 할 범위를 정해주는 것이다.

이러한 Coroutine Scope의 종류에는 크게 두 가지가 있는데

- 사용자 지정 Coroutine Scope
- GlobalScope



먼저, 사용자 지정 Coroutine Scope는 가장 기본적인 방식의 CoroutineScope 이다.

```kotlin
val scope = CoroutineScope(CoroutineContext /* Dispatchers.Main과 같은*/ )
val job = scope.launch{
   
}
```

이렇게 특정 Coroutine이 필요할 때 마다 새로 선언하여 사용하고,

필요 없어지면 종료하도록 할 수 있다.



예를 들어, 어떤 Activity에 보여줄 데이터를 코루틴을 통해 불러오고 있다고 생각해보자.

근데 해당 Activity가 도중에 종료가 되버린다면 불러오고 있는 데이터는 필요가 없게 되므로

코루틴도 함께 종료되어야 한다.



이 때, Coroutine Scope를 Activity의 LifeCycle에 맞춰주면 Activity가 종료될 때 코루틴도 함께 종료되도록 만들 수 있다.



반면, GlobalScope는 앱이 실행될 때 부터 종료될 때 까지 코루틴을 실행시킬 수 있는 Scope이다.

```kotlin
GlobalScope.launch {
	
}
```

그래서 어떤 Activity에서 GlobleScope를 통해 실행된 Coroutine은 해당 Activity가 종료되어도

코루틴이 완료될 때 까지 동작하게 된다.



따라서, 앱이 실행되는 동안 혹은 장시간 실행되어야 하는 코루틴의 경우에는 이 GlobalScope를 사용하고 

필요할 때만 수행되어야 하는 코루틴의 경우에는 사용자 지정 Coroutine을 사용하자.



## Coroutine Builder

CoroutineBuilder에는 async{} , launch{}, runBlocking{} 이 있다.

Coroutine Scope의 확장함수로써 { 내부 코드 } 내부의 코드를 Coroutine으로 실행시켜주는 역할을 한다.



- async()
  - 결과가 예상되는 코루틴 시작에 사용(결과 반환)
  - 전역으로 예외 처리 가능
  - 결과, 예외 반환 가능한 Deferred<T> 반환
- launch()
  - 결과를 반환하지 않는 코루틴 시작에 사용(결과 반환X)
  - 자체/ 자식 코루틴 실행을 취소할 수 있는 Job 반환
- runBlocking()
  - Blocking 코드를 일시 중지(Suspend) 가능한 코드로 연결하기 위한 용도
  - main함수나 Unit Test때 많이 사용됨
  - 코루틴의 실행이 끝날 때 까지 현재 스레드를 차단함



CoroutineBuilder에서 반환된 Job, Deferred 객체를 이용하여 각각의 Coroutine을 제어할 수 있다.

예를 들어, job.join()은 코루틴이 완료될 때 까지 기다리는 함수이다.



이러한 job의 유용성 때문에 GlobalScope사용을 지양하는 편이다.

GlobalScope에는 job이 없기 때문에 구조화된 동시성이 느슨해지게 된다.



다음으로 넘어가기 전에,  **runBlocking{}**과 **async{}**을 비교해보고 가자.

먼저, runBlocking() 이다.

```kotlin
fun main() = runBlocking {
	val name = getFirstName()
	val lastName = getLastName()
	println("Hello, $name $lastName")
}

suspend fun getFirstName() : String{
	delay(1000)
	return "Hong"
}

suspend fun getLastName() : String{
	delay(1000)
	return "Eunho"
}
```

결과값은 Hello, Hong Eunho => 총 2초가 소요된다.



아래는 async() 이다.

```kotlin
fun main() = runBlocking {
	val name = Defered<String> = async { getFirstName() }
	val lastName = Defered<String> = async { getLastName() }
	println("Hello, ${name.await()} ${lastName.await()}")
}

suspend fun getFirstName() : String{
	delay(1000)
	return "Hong"
}

suspend fun getLastName() : String{
	delay(1000)
	return "Eunho"
}
```

결과값은 Hello, Hong Eunho => 총 1초가 소요됨

async()는 각각의 함수가 동시에 실행되어 반환되므로 두 과정 모두 1초의 delay를 동시에 겪지만

runblocking() 은 한 코루틴이 끝날 때 까지 현재 스레드를 차단하기 때문에 2초가 소요된다.







## Suspend(일시 중단 함수)

- Coroutine 혹은 suspend func 내부에서 사용하기 위해 만드는 함수

- 앞에 suspend 키워드를 붙여서 함수를 구성하며  

  이렇게 suspend를 붙이게 되면 하나의 코루틴으로 동작하기 위한 자격을 얻게 됨.

- 람다를 구성하여 다른 suspend 함수를 호출함 ex) runBlocking{ ... }



## Coroutine 직접 사용해보기

먼저, 아래와 같이 종속성을 추가하여 코루틴을 사용할 셋팅을 하자.

```kotlin
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.4.1"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.4.1"
```



다음과 같은 Coroutine을 만들어 보자.

```kotlin
val scope = CoroutineScope(Dispatchers.Main)
scope.launch {
	// Coroutine1
}
scope.launch(Dispatchers.Default) {
	// Coroutine2
}
```

먼저 Coroutine1쪽의 위를 보면 CoroutineContext를 Dispatchers.Main으로 지정함을 알 수 있다.

그리고 여기서 Coroutine1이 실행된다.



하지만 Coroutine2쪽을 보면 다시 scope.launch(Dispatchers.Default)를 통해 Context를 재지정해줌을 알 수 있다.



이렇게 CoroutineContext는 CoroutineScope에서 지정될 수도 있고 CoroutineBuilder에서 지정될 수도 있는데,

Coroutine2처럼 CoroutineBuilder에서 따로 CoroutineContext를 설정해준다면,

해당 CoroutineBuilder로 실행되는 코루틴은 그 ContextContext를 따라가게 된다.



즉, 코루틴2는 Dispatchers.Default를 따라가게 되는 것이다.

그래서, 코루틴1은 MainThread, 코루틴2는 DefaultThread(background)에서 실행된다.



이렇게 UI작업을 진행하는 CoroutineScope에서 하나의 코루틴만 Background 작업으로 돌리고 싶을 때 이러한 방법을 사용한다.



이번엔 **Job**을 사용해보자.

```kotlin
val job = scope.launch {
	// todo
}
```

위의 [Coroutine Builder](#coroutine-builder)에서 설명한 것 처럼 각 코루틴에 대한 Job 객체를 반환받아 각각의 코루틴을 제어할 수 있다.



근데 만약 하나의 코루틴 Scope내에 여러 자식 코루틴이 있을 때, 이 코루틴들을 한번에 제어하려면 어떻게 해야할까?

각각의 코루틴들에 job1, job2, job3 ... 으로 각각의 job을 연결해주고 마지막에 cancel을 해야할까?

아니다, 다음과 같은 방법을 사용하자.

```kotlin
suspend fun main() = coroutineScope {
	val job = Job()
	CoroutineScope(Dispatchers.Default + job).launch {
		launch {
			println("Coroutine1 start")
			delay(1000)
			println("Coroutine1 end")
		}
		launch {
			println("Coroutine2 start")
			delay(1000)
			println("Coroutine2 end")
		}
	}
	delay(1000)
	job.cancel()
	delay(2000)
	println("Finish")
}
```

위의 코드처럼 하나의 Job 객체를 생성하고 새로 선언될 CoroutineScope에서 객체를 초기화 하면

이 CoroutineScope의 자식들에게 까지 모두 영향을 주는 Job으로 활용이 가능하다.

CoroutineScope 내부의 코루틴들은 기본적으로 자신이 속한 CoroutineScope의 Context를 상속받기 때문이다.



그럼 코루틴의 **부모- 자식** 간의 관계를 살펴보고 가자.

기본적으로 부모 코루틴이 취소되면 자식 코루틴도 취소되며

부모 코루틴은 자식 코루틴이 모두 완료될 때 까지 대기한다.



하지만, 어떤 CoroutineScope안에 있다고 해서 모두 자식 Coroutine은 아니다.

다음과 같은 코드를 살펴보자.

```kotlin
suspend fun main() = coroutineScope{
	val job = Job()
    CoroutineScope(Dispatchers.Default+job).launch {
        launch{ // Child O
            println("coroutine1 start")
            delay(1000)
            println("coroutine1 end")
        }
        CoroutineScope(Dispatchers.IO).launch { // Child X
            println("coroutine2 start")
            delay(1000)
            println("coroutine2 end")
        }
        CoroutineScope(Dispatchers.IO + job).launch { // Child O
            println("coroutine2 start")
            delay(1000)
            println("coroutine2 end")
        }
    }
    delay(300)
    job.cancel()
    delay(3000)
    println("Finish")
}
```



위의 코드를 실행시켜보면 다음과 같은 결과를 얻을 수 있다.

```kotlin
Coroutine1 Start
Coroutine2 Start
Coroutine3 Start
Coroutine2 End
Finish
```



하나의 launch안에 여러개의 Coroutine이 존재하는데, Coroutine2만 end되고 끝나버렸다.

왜냐하면, Coroutine1,3 / Coroutine2 의 제어범위가 다르기 때문이다.



Coroutine2는 기존의 Scope와는 상관없이 새로운 CoroutineScope(Dispatcher.IO) 를 생성하여

코루틴을 실행시킨다.

그래서 외부 CoroutineScope.launch의 취소 여부는 Coroutine2에 어떠한 영향도 미치지 않는다.



하지만 Coroutine3도 새로운 Scope를 생성하는 방식인데 취소가 되었다.

왜냐하면 CoroutineContext + **job** 때문이다.

job으로 인해 상위 코루틴과 연결이 되면서 상위 코루틴의 영향을 받게 된다.

