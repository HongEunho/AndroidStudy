# [DAY12] 안드로이드와 구글 캘린더 API 셋팅 및 로그인 구현

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
```

- 사용자로부터 인터넷 권한과 네트워크 접근권한, 계정 접근권한이 필요하므로 이 세 가지 퍼미션을 manifest에 추가했습니다.

</br>

### 구글 로그인을 구현하기 위해 Credential을 받아오는 부분은

- 안드로이드에 내장된 라이브러리를 이용해 받아오는 방법과, 파이어베이스의 Authentication을 사용해 받아오는 방법이 있다.

- 파이어베이스를 이용한 방법은 전에도 해본 경험이 있기 때문에 새로운 방식으로 이용해보고자 SDK를 이용해 구현을 했다.

- 환경 셋팅을 위해 다음과 같이 build.gradle에 추가해 주었다.

  ```kotlin
  dependencies {
      compile fileTree(dir: 'libs', include: ['*.jar'])
      compile 'com.android.support:appcompat-v7:25.0.1'
      compile 'com.google.android.gms:play-services-auth:15.0.1'
      compile 'pub.devrel:easypermissions:0.3.0'
      compile('com.google.api-client:google-api-client-android:1.23.0') {
          exclude group: 'org.apache.httpcomponents'
      }
      compile('com.google.apis:google-api-services-<API>-<VERSION>') {
          exclude group: 'org.apache.httpcomponents'
      }
  }
  ```

  마지막의 <api> 와 <version>은 각각 필요한 api와 그에 맞는 버전을 삽입하면 된다.

</br>

### 구글 로그인 기능 구현

- 먼저 오늘은 구글 로그인 까지의 구현을 목표로 하였다.
- 임시로 버튼을 하나 만들어 버튼을 누르면 구글 로그인을 진행하도록 하였다.
- 로그인 전 권한요청과 그에 따른 응답으로 로그인을 진행하도록 하였으며
- 여러 구글 계정 중 하나를 선택하도록 하여 선택한 아이디로 로그인 하도록 구현 하였다.
- 내일은 이 로그인 된 계정으로 캘린더의 목록들을 가져와 뷰에 띄워볼 예정이다.

</br>

### 구글 캘린더 API 사용을 위한 셋팅

- 구글 클라우드 플랫폼에서 캘린더 API 사용을 위해 OAuth 2.0 클라이언트 ID 생성을 완료했다.
- 내일 셋팅한 ID로 구글 캘린더 API를 사용하여 일정을 가져와볼 예정이다.

</br>

### 마일스톤 작성

- 어제 작성한 FeatureList를 토대로 마일스톤을 작성할 예정이다.
- FeatureList를 개발자의 측면에서 다시 나눠 재구성하여 멘토님과 함께 MD산정을 할 예정이다.

