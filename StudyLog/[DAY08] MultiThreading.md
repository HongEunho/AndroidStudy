# [DAY08] MultiThreading

## MultiThreading

### 멀티스레딩이란?

- 하나의 프로세스에서 여러 스레드를 실행하여 많은 작업을 동시에 처리하는 것

- 멀티 프로세싱, 멀티 태스킹과는 구별됨

- 스레드들은 해당 프로세스의 여러 자원을 공유할 수 있음
- 안드로이드에서는 시간이 걸리는 작업을 처리하면서도 UI가 계속 반응하게 하기 위해서는 필수적

</br>

#### 자바의 멀티스레딩

- 자바는 Thread, Runnable등의 요소를 통해 멀티스레딩을 지원
- 안드로이드에서도 자바의 멀티스레딩 도구를 활용할 수 있음

</br>

#### Thread

- 하나의 스레드를 나타내는 클래스
- 시작/중지/대기 등 스레드 자체에 대한 동작 관리

</br>

#### Runnable

- 하나의 작업(Task)를 나타내는 클래스

</br>

#### MultiThreading 이슈

병렬 처리에 따른 이슈 ( Race Condition )

- 여러 스레드를 동시에 실행할 경우 실행 순서가 보장되지 않음

  => 실행 순서를 예상하거나 단정지을 수 없음

- 하나의 데이터에 동시에 접근하는 경우 실행 순서에 따라 결과가 바뀔 수 있음

  => 서로 다른 공간의 변수에 접근하거나 변경할 수 있기 때문

  => 값을 읽고 변경하고 다시 쓰는 동작이 thread safe하게 발생하지 않기 때문

</br>

해결방법

- Synchronized 제어자 or 블록 사용

- Lock객체를 직접 사용
- AtomicInteger 사용

</br>

하지만, Java Concurrency in Practice, Ch. 6 에는 다음과 같은 문구가 있다.

" 프로그램 어디에서든 간에 new Thread(runnable).start)() 와 같은 코드가 남아있다면

Executor를 사용해 구현하는 방안을 심각하게 고려해봐야 한다. "

</br>

실제로, 어제 살펴본 구글에서 제공한 MVP 샘플코드의 스레드 관련된 코드들이 Executor를 사용해 구현되었음을 알 수 있다.

그럼 Executor란 무엇일까?

</br>

#### Executor 프레임워크

- 자바에서 제공하는 고수준 비동기 작업 실행 프레임워크

  Thread 는 저수준 API

- 작업의 등록과 실행을 분리하고 "실행 정책"을 추상화

  - 어느 스레드에서?
  - 어떤 순서로?
  - 동시에 몇 개를?
  - 최대 몇 개 까지 대기?
  - 작업을 거절해야 할 경우 어떤 작업을?
  - 작업의 실행 전후에 어떤 동작을?

- Executor Interface

  - 어떤 작업(Runnable)을 실행하는 기능을 나타내는 인터페이스

    ```kotlin
    public interface Executor {
        void execute(Runnable var1);
    }
    ```

  - 이를 활용한 구현 예제

    ```kotlin
    public class DirectExecutor : Executor {
    	override fun execute(command: Runnable) {
    		command.run()
    	}
    }
    ```

    ```kotlin
    public class ThreadPerTaskExecutor : Executor {
    	override fun execute(command: Runnable) {
    		Thread(command).start()
    	}
    }
    ```

    ```kotlin
    class DiskIOThreadExecutor : Executor {
        private val diskIO = Executors.newSingleThreadExecutor()
        override fun execute(command: Runnable) { diskIO.execute(command) }
    }
    ```



Executor를 통해 얻고 싶은 이득은 => Thread Pool

#### Thread Pool

- 작업을 실행하는 스레드를 "풀"로 관리하는것

- 여러 스레드에 작업을 분배해야 하므로 동기화된 큐 필요

- 이미 생성된 스레드를 재사용

  작업마다 매번 스레드를 생성할 필요가 없으므로, 소모되는 시스템 자원 및 작업 시작딜레이가 매우 줄어듦

- 생성되는 스레드의 개수를 통제

  CPU 코어를 최대한 활용할 수 있으며, 과도하게 많은 스레드가 시스템 자원을 두고 경쟁하는 상황을 방지할 수 있음

- 작업자 스레드가 관리됨

  생명주기 관리, 모니터링, 오류 처리 등이 용이함

</br>

#### ExecutorService

- 스레드 풀에 대한 생명주기를 관리하는 기능이 추가된 Executor

- 큐에 남은 작업을 정리하거나 스레드를 종료시키는 등의 매커니즘을 제공

- Executors 클래스의 팩토리 메소드를 통해 미리 정의된 스레드 풀을 사용하는  ExecutorService를 생성할 수 있음

- ex )

  Executors.newFixedThreadPool(int nThreads)

  최대 크기가 고정된 스레드 풀

  Executors.newCachedThreadPool()

  크기가 유동적인 스레드 풀

  Executors.newSingleThreadExecutor()

  단일 스레드에서 동작하는 스레드 풀

