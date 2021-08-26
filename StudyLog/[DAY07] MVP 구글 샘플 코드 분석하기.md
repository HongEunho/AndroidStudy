# [DAY07]MVP 구글 샘플 코드 분석하기

구글 샘플코드의 todo-mvp 코드중 addedittask 패키지부터 분석해 보았다.

패키지 안에는 **Activity**, **Contract**, **Fragment**, **Presenter**가 있다.

</br>

먼저 `Contract`에는 **View**와 **Presenter**에서 필요한 필수적인 기능들을 명시함으로써 틀을 잡아주었고 

**View**와 **Presenter**를 하나의 인터페이스에 각각 정의하여 한눈에 볼 수 있도록 하여 가독성을 높여주었다.

```kotlin
interface AddEditTaskContract {

    interface View : BaseView<Presenter> {
        var isActive: Boolean

        fun showEmptyTaskError()

        fun showTasksList()

        fun setTitle(title: String)

        fun setDescription(description: String)

    }

    interface Presenter : BasePresenter {
        var isDataMissing: Boolean

        fun saveTask(title: String, description: String)

        fun populateTask()
    }
}
```

</br>

다음으로 `AddEditTaskPresenter`을 살펴보았다.

AddEditTaskPresenter생성자 부분을 살펴보면,

```kotlin
class AddEditTaskPresenter(
        private val taskId: String?,
        val tasksRepository: TasksDataSource,
        val addTaskView: AddEditTaskContract.View,
        override var isDataMissing: Boolean
) : AddEditTaskContract.Presenter, TasksDataSource.GetTaskCallback
```

- **Contract**의 **Presenter**와 **TasksDataSource**의 **GetTaskCallback**을 상속받는다.

- 즉, Contract의 Presenter에서 정의한 interface내의 함수들을 모두 구현해야한다.

  => 이는, Contract에서 Presenter에서 **꼭 구현해야 할 비즈니스 로직**에 대해 안내를 함으로써 틀을  짜준다고 볼 수 있다.

- 생성자로 입력하는 AddEditTaskContract.View를 멤버 변수 addTaskView와 매치시킨다.

- 매치시킨 View의 presenter로 현재 presenter를 set 한다. ( init { 여기에서.. })

</br>

이를 활용해  만약, **View**에서 비즈니스 로직을 통해 UI가 갱신되어야 한다면, 

이와 관련된 부분을 **Contract**의 View Interface안에 메소드를 추가해주고 

이를 Activity나 Fragment등의 View에서 오버라이딩 하여 구현하면 된다.

</br>

그럼 대표 함수 하나를 예로 들어 동작 과정을 살펴보면 다음과 같다.

Contract의 Presenter인터페이스의 saveTask를 오버라이딩 한 AddEditTaskPresenter의 saveTask를 살펴보면,

```kotlin
override fun saveTask(title: String, description: String) {
    if (taskId == null) {
        createTask(title, description)
    } else {
        updateTask(title, description)
    }
}
```

이 함수는 View에서 저장버튼의 요청에 따라 View의 title과 description을 인자로 받은 후에, 

taskId 값에 따라 createTask와 updateTask을 수행하는 로직을 가지고 있는 함수이다.

</br>

이 예시코드 앱이 todo 앱이다 보니 createTask는 새로운 할일을 생성하는 것인데,

createTask는 addTaskView의 showTasksList()을 수행하며

또, tasksRepository의 saveTask를 수행하게 된다.

```kotlin
private fun createTask(title: String, description: String) {
    val newTask = Task(title, description)
    if (newTask.isEmpty) {
        addTaskView.showEmptyTaskError()
    } else {
        tasksRepository.saveTask(newTask)
        addTaskView.showTasksList()
    }
}
```

</br>

즉, view에서 할일 리스트를 보여주고, repository에 현재 변경된 task를 저장하는 것이다.

</br>

할일 리스트를 보여주는 것(showTasksList())은 UI와 관련된 작업이고

할일 데이터를 저장하는 것(saveTask)은 Model과 관련된 작업이기 때문에

presenter에서는 이들을 직접 처리할 수 없으므로, 각각의 컴포넌트가 처리하도록 하는 것이다.

</br>

그럼 다음으로, AddEditTaskActivity를 살펴보자.

```kotlin
val addEditTaskFragment =
        supportFragmentManager.findFragmentById(R.id.contentFrame) as AddEditTaskFragment?
                ?: AddEditTaskFragment.newInstance(taskId).also {
            replaceFragmentInActivity(it, R.id.contentFrame)
        }
```

먼저 addEditTaskFragment 를 newInstance(taskId) 를 통해 fragment 객체를 생성한다.

```kotlin
addEditTaskPresenter = AddEditTaskPresenter(taskId,
        Injection.provideTasksRepository(applicationContext), addEditTaskFragment,
        shouldLoadDataFromRepo)
```

그리고 addEditTaskPresenter를 생성하면서 생성자의 인자로 Fragment 객체를 넘겨주기 때문에 Presenter의 View와 이 Fragment가 연결되게 된다.

</br>

Addedittask 패키지 외에 task와 관련된 패키지들은 이와 비슷한 구조로 되어 있다.

</br>

다음 페이지에서 이어서 Data 패키지를 분석해보자.
