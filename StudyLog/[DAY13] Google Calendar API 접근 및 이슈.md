# [DAY13] Google Calendar API 접근 및 마일스톤 작성

어제 연동한 로그인을 바탕으로 Google Calendar API에 접근하여

캘린더 데이터를 가져오는 실습을 진행하였다.

</br>

#### 하지만 이 과정에서 다음과 같은 이슈가 발생하였다.

1. 로그인 한 구글 계정의 캘린더 리스트들을 가져오려고 함

2. 다음과 같이 코드를 작성하여 리스트를 가져오려고 했지만

   **접근 권한 오류가 발생** 하였다.
   ```kotlin
   val calendarList = service.calendarList().list().setPageToken(pageToken).execute()
   ```

   

3. 에러를 출력하여 **UserRecoverableAuthIOException**이 발생하였음을 알게되었다.

   이 오류의 원인은 구글 계정 권한 뿐만 아니라 현재 **내 앱이 해당 계정의 캘린더에 접근할 권한**을 얻어와야 하는데

   권한을 얻지 않은 상태로 구글 계정의 캘린더 목록을 불러오려고 해서 발생한 오류였다.

  

4. 그래서 다음과 같이 예외처리를 통해 해당계정의 캘린더에 접근 권한을 먼저 얻어온 다음

   구글 계정의 캘린더에 접근할 수 있게 함으로써 오류를 해결하였다.
   
   ```kotlin
   try{
       val calendarList = service.calendarList().list().setPageToken(pageToken).execute()
       pageToken = calendarList.nextPageToken
   }
   catch (exception: UserRecoverableAuthIOException) {
       Log.d("error", exception.cause.toString())
       startActivityForResult(exception.intent, REQUEST_AUTHORIZATION)
   }
   ```

</br>

이 때, 한 가지 의문점이 생겼는데...

현재 구글캘린더 API의 사용자 인증방식을 oAuth2.0으로 설정해두었고, 

앱 배포가 아닌 테스트 단계에서는 테스트 사용자에 등록한 계정만

이 권한을 얻어 캘린더API를 사용할 수 있도록 되어있다.

</br>

그래서, 테스트 계정에 등록한 계정이 아닌 다른 계정으로 앱을 실행하면

권한을 얻는 부분에서 성공적으로 권한을 얻지 못하게 된다.

</br>

개발 단계에서 테스트 계정 외에 다른 계정도 권한을 얻어 캘린더 API를 사용할 수 있는지에 대해

내일 멘토님께 문의를 드려서 확인해보려고 한다.

</br>

#### 또 다른 이슈로는 캘린더를 불러오는 과정을 메인 스레드에서 처리했을 때

데드락 오류가 발생하였다.

</br>

캘린더를 불러오는 과정 또한 데이터를 가져와 저장하는 작업이므로

시간이 오래 걸리는 작업이기 때문에 UI스레드에서 처리를 하면 안되고

**IO스레드**에서 처리를 해주어야 한다.

따라서, 다음과 같이 코루틴을 사용하여 IO스레드에서 불러오도록 처리함으로써

이 오류를 해결하였다.

```kotlin
launch {
    binding.statusTextView.text = "데이터를 가져오는 중입니다"
    withContext(Dispatchers.IO){
        getCalendar()
    }
}
```

</br>

이렇게 오류들을 해결하여 구글 계정의 캘린더를 불러오는데 까지 성공하였고

이를 바탕으로 마일스톤을 좀 더 보완하여 작성해나가고 있다.
