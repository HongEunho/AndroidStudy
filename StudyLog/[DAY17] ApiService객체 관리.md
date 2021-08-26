# MVVM패턴에서 서비스 객체 관리

먼저, mvvm 아키텍쳐를 적용하기 전에는 액티비티에서 다음과 같이 서비스 객체를 생성하여 사용하였다.

```kotlin
private lateinit var service: Calendar
private lateinit var googleCredential: GoogleAccountCredential
```

```kotlin
googleCredential = GoogleAccountCredential.usingOAuth2(
            applicationContext, listOf(CalendarScopes.CALENDAR)
        ).setBackOff(ExponentialBackOff())

transport = AndroidHttp.newCompatibleTransport()
jsonFactory = JacksonFactory.getDefaultInstance()
service = Calendar
    .Builder(transport, jsonFactory, credential)
    .setApplicationName("BeforeArchitecture QuickStart")
    .build()
```

여기서, googleCredential 를 생성하는 코드쪽을 보게 되면, applicationContext를 인자로 사용하고 있다.

즉, ***context***를 인자로 사용하게 된다.

</br>

서비스는 API를 호출할 때 마다 항상 필요한 객체이기 때문에

싱글톤으로 생성하여 계속 가져다가 쓰는 방식으로 구현을 할 계획이었다.

그래서 처음에는 다음과 같이 코드를 짰는데, 인자로 context가 필요한 상황이 발생하였고

viewModel에서는 context 참조를 지양해야 했기 때문에 어떻게 코드를 수정해야 할지 고민을 해봤다.

```kotlin
object ApiProvider {

    val apiService: Calendar by lazy { getService(context) }

    fun getService(context: Context) {
       val transport = AndroidHttp.newCompatibleTransport()
       val jsonFactory = JacksonFactory.getDefaultInstance()
      
        googleCredential = GoogleAccountCredential.usingOAuth2(
            context, listOf(CalendarScopes.CALENDAR)
        ).setBackOff(ExponentialBackOff())

        val user = FirebaseAuth.getInstance().currentUser
        credential.selectedAccount = Account(user?.email, BuildConfig.APPLICATION_ID)

        return Calendar.Builder(transport, jsonFactory, credential)
            .setApplicationName("BeforeArchitecture QuickStart")
            .build()
    }


}
```

</br>

그래서 googleCredential을 액티비티나 프레그먼트에서 생성을 하고 넘겨주는 방식을 고안했다.

싱글톤과 코틀린 문법에 아직 익숙하지 않다보니 

처음에는 위의 코드처럼 apiService 객체를 이 ApiProvider 오브젝트 안의 getService를 통해 생성하고 그 결과를 받아야 한다는 생각을 갖고

코드를 짰었다.

</br>

하지만 여러가지 방법을 찾아본 결과 다음과 같이 credential를 외부에서 주입받고, createService안에서 Service객체를 생성하여 

lateinit을 통해 apiService 변수에 대입을 함으로써 해결을 할 수 있었다.

```kotlin
object ApiProvider {

    lateinit var apiService: Calendar

    fun createService(transport: HttpTransport, jsonFactory: JsonFactory, credential: GoogleAccountCredential) {

        val user = FirebaseAuth.getInstance().currentUser
        credential.selectedAccount = Account(user?.email, BuildConfig.APPLICATION_ID)

        Log.e("test", credential.selectedAccount.toString())

        apiService = Calendar.Builder(transport, jsonFactory, credential)
            .setApplicationName("BeforeArchitecture QuickStart")
            .build()
    }

    
}
```
