# MVP 샘플코드의 Data패키지

전 문서에 이어서 샘플 코드의 Data패키지를 살펴보자

```kotlin
interface TasksDataSource {

    interface LoadTasksCallback {

        fun onTasksLoaded(tasks: List<Task>)

        fun onDataNotAvailable()
    }

    interface GetTaskCallback {

        fun onTaskLoaded(task: Task)

        fun onDataNotAvailable()
    }

    fun getTasks(callback: LoadTasksCallback)

    fun getTask(taskId: String, callback: GetTaskCallback)

    fun saveTask(task: Task)

    fun completeTask(task: Task)

    fun completeTask(taskId: String)

    fun activateTask(task: Task)

    fun activateTask(taskId: String)

    fun clearCompletedTasks()

    fun refreshTasks()

    fun deleteAllTasks()

    fun deleteTask(taskId: String)
}
```

TaskDataSource에는 Data에 관련된 비즈니스 로직이 선언되어 있다.

여기에는 또 **LoadTasksCallback** 인터페이스와 **GetTaskCallback** 인터페이스가 있는데

LoadTask가 필요한 부분은 LoadTasksCallback을 상속받아 구현하도록 하도록 설계되었고

GetTask가 필요한 부분은 GetTasksCallback을 상속받아 구현하도록 설계가 되어있다.

</br>

즉, 각각의 TaskCallback에서 필요한 부분들에 대한 함수들을 **틀을 정해 명시**해 놓은 것이다.

그래서 예를 들어, GetTasksCallback을 Presenter에서 구현한다고 하면,

직접 다른 함수들을 만들어서 구현할 필요 없이

여기에서 명시한 함수들을 구현하면 되는 것이다.

</br>

GetTaskCallback 인터페이스 아래로는 getTasks부터 deleteTask까지 명시가 되어있는데

TaskData를 처리하는데 필요한 필수 함수들을 모두 명시해 놓은 것이다.

마찬가지로, TaskDataSource를 상속받는 구현체들에서 이 함수들을 **각각의 목적에 맞게**

구현을 하면 되는것이다.

</br>

실제로 TasksLocalDataSource나 TasksRemoteDataSource, TasksRepository 가 이를 상속받아서

각각의 목적에 맞게 구현이 되어있다.

</br>

> 여기서 내가 처음에 한가지 의문이 들었던 거는 TasksRepository에서 이를 상속받아서 구현이 되어있음에도 불구하고, 왜 TasksLocalDataSource도 이를 상속받아 따로 구현하고 TasksRemoteDataSource도 이를 상속받아 구현했다는 점이다.

</br>

어차피 Presenter을 보면, TaskRepository에 접근해서 사용하는 구조인데, 

그냥 Repository에 한 번만 상속받아 구현을 하고

매개변수로 remote인지 local인지 받아서, 그에 맞게 처리하면 되지 않나?

라는 의문을 갖게했다.

</br>

이 의문을 해결해준 답안은 이렇다.

TasksLocalDataSource는 LocalData에 맞게 구현하였고, 

TasksRemoteDataSource는 RemoteData에 맞게 구현하였고,

TasksRepository는 이 둘의 객체를 가지고 적절하게 사용가능하도록 또 다르게 구현이 되어있다.

</br>

즉 Presenter에서는 TasksRepository에 접근해 사용할지 몰라도,

TasksRepository는 또 TasksLocalDataSource와 TasksRemoteDataSource를 이용해 구현을 하기 때문에 각각 상속받아 다르게 구현을 한 것이다.



그리고 Presenter에서는

```kotlin
val tasksRepository: TasksDataSource,
```

이렇게 변수명을 tasksRepository로 지었지만 TasksDataSource 객체이다.

그래서, tasksRepository.getTask()라는 함수를 호출하게 되면

원래는 TasksRepository에 구현된 getTask()를 호출하는 것이 아니라 TasksDataSource에 정의된 getTask()를 호출하게 되겠지만,

이 때, Presenter 생성자에서 넘겨받은 TasksDataSource 객체는 TasksRepository 인스턴스 이기 때문에 TasksRepository에서 구현된 getTask()를 호출하게 되며

이 TaskRepository의 getTask()에서는 상황에 맞게 TasksLocalDataSource와 TasksRemoteDataSource를 이용하게 된다.

</br>

이러한 설계는 객체지향 원칙 **SOLID** 단일 책임 원칙에 의해 하나의 클래스는 하나의 책임만 가져야 한다는 원칙을 잘 지킨다고 볼 수 있다.

만약, 내가 처음에 생각했던 대로 하나의 클래스에서 이 세 기능을 모두 하도록 구현하도록 했다면

코드의 길이도 굉장히 길어지며 원칙을 위반하므로 좋은 코드라고 할 수 없다.

</br>

또한, 예를 들어 기존의 getTask()함수에서 특정 부분을 변경하여 특정 클래스에 반영을 하고 싶을 때,

이렇게 잘 설계된 구조라면, 변경하고 싶은 부분만 변경한 새로운 Repository를 생성하여 새로운 Repository에서는 getTask()가 다르게 작동하도록 하여

그 클래스의 Repository만 새걸로 갈아끼워 넣어주면 되는데

하나의 클래스에서 이를 모두 해결했다면, 기존 Repository에 추가 함수를 만들어야 할 것이고 해당하는 Presenter나 기타 클래스로 가서 하나씩 함수명을 모두 바꿔주어야 한다.

</br>

따라서 이렇게 SOLID의 원칙에 따라 구조를 잘 설계한다면 추후의 유지보수에도 굉장히 용이하다.

</br>

이 샘플 코드를 통해 **추상화의 중요성**에 대해서도 다시 한 번 깨닫게 되었다.