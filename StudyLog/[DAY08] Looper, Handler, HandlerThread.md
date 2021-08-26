# [DAY08] Android Main Thread

# Looper, Handler, HandlerThread

### Main Thread

- 안드로이드 시스템이 특수하게 취급하는 가장 중요한 스레드
- 앱의 실행과 함께 생성
- 가장 높은 우선권을 지님
- 앱의 핵심 메소드는 대부분 메인 스레드에서 실행됨
  - 액티비티의 라이프사이클 콜백
  - 뷰의 이벤트 리스너
- 메인 스레드가 반응하지 않으면 ANR 발생

</br>

### Main Thread 에서만 할 수 있는 일

- UI (뷰)를 다루는 일
- 다른 스레드에서는 할 수 없음(Exception 발생!)

</br>

### Main Thread에서 해서는 안되는 일

- 네트워크 I/O

  절대 할 수 없음(Exception 발생)

- 파일 I/O

  DB 조회/생성/수정/삭제

  SharedPreference

- 그 외 오래 걸리는 모든 작업

  JSON 파싱

  비트맵 디코딩

</br>

### View.post() & Activity.runOnUiThread()

- 만약 메인 스레드에게 "이 Runnable을 실행해라" 라는 메시지를 전달 한다면
- 메인 스레드는 어떻게 메시지를 받아서 처리할까?
  - Runnable은 메시지 객체로 래핑됨
  - 래핑된 메시지는 핸들러를 통해 메인 스레드의 메시지 큐에 삽입됨
  - 메인 스레드는 루퍼를 통해 메시지 큐에서 메시지를 읽어들여 처리함
- Thread : Looper : Message Queue = 1 : 1 : 1

</br>

### Message

- 데이터 또는 작업을 담는 객체
- Message Object Pool 이 존재
  - Message.obtain()을 통해서 생성
  - new Message()도 가능하지만 권장X
  - 객체 풀을 통한 재사용이 불가능하기 때문
- Message Queue
  - 핸들러와 루퍼 사이에 있는 존재로써
  - 메시지들의 Singly Linked List
  - 메시지가 처리되어야 하는 순서대로 연결(Message의 TimeStamp 순)
  - 응용 프로그램에서 직접 사용할 일은 거의 없음

</br>

### Looper

- 메시지 큐로부터 메시지를 하나씩 읽어들여 처리

- 어떻게 하나씩 읽어들일까?

  - 무한 루프를 돌면서 한 루프에 하나의 메시지를 가져옴
  - 메시지 큐가 텅 비면 스레드 Sleep

- 무한 루프에 빠져 있기 때문에 루퍼가 동작하는 스레드는 루퍼를 돌리는 것 외에 다른 일을 할 수 없음

- Looper 만들기

  ```java
  public static void prepare(); // loop()을 호출하기 전 반드시 호출해야함
  public static void loop();
  ```

- Looper 가져오기

  ```java
  public static Looper getMainLooper(); // main thread looper
  public static Looper myLooper(); // 현재 thread의 looper
  ```

- Looper 종료하기

  ```java
  public void quit();	// message queue의 다음 message를 null로 만듬
  public void quitSafely();	// 남아있는 작업을 완료한 다음 quit
  ```

- Looper.prepare()을 호출하면 현재 스레드에 대한 Looper가 만들어지며

  호출하지 않으면 Looper.myLooper()는 null

- Looper.loop()을 호출하면 루퍼가 동작하면서 무한 루프에 빠짐

  Looper 객체를 통해 시작하는 것이 아님

- Looper.getMainLooper()는 항상 존재함

</br>

### Handler

- 응용 프로그램에서 주로 다루는 것
- 메시지를 보내는 쪽(producer) & 받는 쪽(consumer) 을 위한 기능 제공
  - 보낼 때 : 메시지 큐에 메시지 삽입
  - 받을 때 : 메시지 큐로부터 꺼낸 메시지에 대한 콜백 호출
- 핸들러를 통해 보낸 메시지는 그 핸들러에서만 받을 수 있음
  - 따라서 보내는 쪽과 받는 쪽이 핸들러 객체를 공유 해야 함

</br>

### Handler 생성자와 Looper

```java
public Handler(); // 해당 thread의 looper 사용
public Handler(Callback callback); // 해당 thread의 looper 사용
public Handler(Looper looper);
public Handler(Looper looper, Callback callback);
```

- Looper를 지정하지 않으면 Looper.myLooper()가 사용됨
- 받는 쪽의 현재 스레드는 반드시 Looper.prepare()된 상태여야 함
- 특히 이 Handler는 메인 스레드에서 생성되어야 정상 동작

</br>

### Handler를 통한 메시지 전송 : 유형별

- 작업(Runnable)

  ```java
  public boolean post(Runnable r);
  public boolean postDelayed(Runnable r, long delayMillis);
  public boolean postAtTime(Runnable r, long uptimeMillis);
  public boolean postAtTime(Runnable r, Object token, long uptimeMillis);
  public boolean postAtFrontOfQueue(Runnable r);
  ```

- 데이터 (message with data)

  ```java
  public boolean sendMessage(Message msg);
  public boolean sendMessageDelayed(Message msg, long delayMillis);
  public boolean sendMessageAtTime(Message msg, long uptimeMillis);
  public boolean sendMessageAtFrontOfQueue(Message msg);
  ```

- 빈(What만 있는) 데이터 (Message with what)

  ```java
  public boolean sendEmptyMessage(int what);
  public boolean sendEmptyMessageDelayed(int what, long delayMillis);
  public boolean sendEmptyMessageAtTime(int what, long uptimeMillis);
  ```

- 전송 취소

  ```java
  public void removeCallbacks(Runnable r);
  public void removeCallbacks(Runnable r, Object token);
  public void removeMessages(int what);
  public void removeMessages(int what, Object object);
  public void removeCallbacksAndMessages(Object token);
  ```

  - token이 null일 경우 모든 메시지가 삭제됨

    전체 삭제 : removeCallbacksAndMessage(null)

  - 해당 Handler에서 보낸 메시지만 삭제 됨

    다른 Handler와 Looper를 공유하더라도 영향을 미치지 않음

  - 주로 delayed와 엮어서 유용한 처리 가능

    사용자가 5초 이상 아무것도 하지 않으면 자동으로 다음 화면 이동

    다음 화면 이동 작업을 5000ms 지연해서 postDelayed()

    화면 터치시 removeCallbacks()로 작업 취소

</br>

### 백그라운드 스레드에서의 메시지 수신

- 앞의 내용은 백그라운드 스레드에서 메인 스레드로 메시지를 보내는 경우

- 백그라운드 스레드가 메시지를 받으려면?

  => 핸들러가 백그라운드 스레드의 looper와 연결되어야 함!
