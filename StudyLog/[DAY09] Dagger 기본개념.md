# [DAY09] Dagger 기본개념

Dagger에는 5가지의 필수 개념이 있다.

1. Inject
2. Component
3. Subcomponent
4. Module
5. Scope



### Inject

의존성 주입을 요청한다. @Inject 어노테이션으로 주입을 요청하면 연결된 Component가 Module로 부터 객체를 생성하여 넘겨준다.



### Component

연결된 **Module**을 이용하여 의존성 객체를 생성하고, **Inject**로 요청받은 인스턴스에 생성한 객체를 주입한다. 의존성을 요청받고 주입하는 Dagger의 주된 역할을 담당한다.



### Subcomponent

Component는 계층 관계를 만들 수 있다. Subcomponent는 inner class 방식의 하위계층 component이다.

Subcomponent의 Subcomponent도 만들 수 있다.

Subcomponent는 Dagger의 중요한 개념이라 할 수 있는 그래프를 형성한다.

Inject로 주입을 요청받으면 Subcomponent에서 먼저 의존성을 검색하고, 없으면 부모로 올라가면서 검색을 하게 된다.



### Module

Component에 연결되어 의존성 객체를 형성한다.

생성 후 Scope에 따라 관리도 한다.



### Scope

생성된 객체의 LifeCycle 범위이다. 안드로이드에서는 주로 PerActivity, PerFragment 등으로 화면의 생명주기와 맞추어 사용한다.

Module에서 Scope를 보고 객체를 관리한다.



위의 5가지 개념을 따라 Dagger가 의존성을 주입하는 flow는 다음과 같다.



@Inject => Subcomponent => Module => Scope에 있으면 return. 없으면 생성

</br>

만약 Subcomponent Module에서 맞는 타입을 못찾으면

상위 Component => Module => Scope에 있으면 return. 없으면 생성


<img width="539" alt="flow" src="https://user-images.githubusercontent.com/46186664/125820683-6f0001ec-09fc-4ca2-a512-a8b5e14b94ac.png">




## Dagger 사용하기

그럼 Application Component부터 의존성을 요청하는 Fragment까지 순서대로 예제를 살펴보자.

위의 Flow를 참고해서 본다면, Dagger의 구조를 이해하는데 많은 도움이 될 것 같다.

</br>


### Application Component

```java
@Singleton  // Scope
// 연결할 Module을 정의
@Component(modules = {AndroidSupportInjectionModule.class,
        ActivityBindingModule.class,
        ApplicationModule.class})
// Application과의 연결을 도울 AndroidInjector를 상속받고, 제네릭으로 BaseApplication 클래스를 정의합니다.
public interface AppComponent extends AndroidInjector<BaseApplication> {

    // Application과의 연결을 도울 Builder를 정의합니다.
    @Component.Builder
    interface Builder {
        @BindsInstance
        AppComponent.Builder application(Application application);
        AppComponent build();
    }
}
```

</br>

### Application Module

```java
@Module
public class ApplicationModule {
    // Context 타입의 의존성 객체를 생성합니다.
    @Provides
    Context providesContext(Application application){
        return application;
    }
}
```

@Provides 어노테이션으로 의존성 객체를 생성할 메소드를 정의한다.

반환 타입을 따라 Component가 검색하여 활용한다.

</br>

### BaseApplication에서 Component 연동하기

```java
// DaggerApplication를 상속받고, ApplicationComponent에서 정의한 Builder를 활용하여 Component와 연결합니다.
public class BaseApplication extends DaggerApplication {
    @Override
    protected AndroidInjector<? extends DaggerApplication> applicationInjector() {
        return DaggerAppComponent.builder().application(this).build();
    }
}
```

</br>

### ActivitySubcomponent 생성하기

```java
// ActivityBindingModule은 위 ApplicationComponent에 연결되어 있습니다.
@Module
public abstract class ActivityBindingModule {
    // ContributesAndroidInjector 어노테이션을 달고 반환타입을 통해 해당 Activity의 Subcomponent를 생성합니다.
    // modules에 Subcomponent와 연결할 Module을 정의합니다. 이 Module들이 실제 의존성 객체를 생성합니다.
    @ActivityScoped  // Scope
    @ContributesAndroidInjector(modules = TasksModule.class)
    abstract TasksActivity tasksActivity();

    @ActivityScoped
    @ContributesAndroidInjector(modules = TaskDetailPresenterModule.class)
    abstract TaskDetailActivity taskDetailActivity();
}
```

원래 Subcomponent는 어노테이션을 활용하여 직접 만들었어야 했지만, 

이제는 ContributesAndroidInjector를 활용하여 Module에서 자동으로 생성할 수 있다.

</br>

### ActivitySubcomponent의 Module

```java
@Module
public abstract class TasksModule {
    // ContributesAndroidInjector로 FragmentSubcomponent를 생성합니다.
    @FragmentScoped  // Scope
    @ContributesAndroidInjector
    abstract TasksFragment tasksFragment();

    // TasksPresenter 타입의 의존성 객체를 생성합니다.
    @ActivityScoped  // Scope
    @Provides
    static TasksPresenter taskPresenter(){
        return new TasksPresenter();
    }
}
```

ApplicationModule과 마찬가지로 Provides으로 의존성 객체를 생성할 메소드를 정의한다.

그리고 ContributesAndroidInjector로 하위 Fragment의 Subcomponent를 생성한다.

</br>

### Activity에서 Component 연동하기

```java
// DaggerAppCompatActivity를 상속받아 Component를 연결합니다.
public class TasksActivity extends DaggerAppCompatActivity { 
    // TasksPresenter 타입의 의존성 주입을 요청합니다.
    @Inject
    TasksPresenter mTasksPresenter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.tasks_act);
    }
}
```

TasksPresenter 의존성을 요청하면 TasksActivity와 연결된 TasksModule에서 생성하고

Subcomponent에서 주입한다.

TasksModule은 ActivityBindingModule에서 Subcomponent를 생성하면서 modules로 연결한다.

</br>

### Fragment에서 Component 연동하기

```java
// DaggerFragment를 상속받아 Component를 연결합니다.
@ActivityScoped  // Fragment는 Activity에 속하므로 Activity Scope를 정의하였습니다.
public class TasksFragment extends DaggerFragment {
    // TasksContract.Presenter 타입의 의존성 주입을 요청합니다.
    @Inject
    TasksContract.Presenter mPresenter;
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
}
```

위 Activity와 비슷한 흐름이다.

여기서 요청받은 TasksContract.Presenter 타입의 의존성은 FragmentModule이 없으므로

부모인 ActivitySubcomponent를 검색하여 ActivityModule에서 생성한다.

이렇게 Subcomponent를 활용하여 Dagger 그래프를 끊지 않고 하위 화면들을 연결할 수 있다.

</br>

### Dagger의 주요 개념

Dagger는 **"그래프"** 가 핵심 개념이다. 의존성을 요청받으면 Subcomponent, Component, Inject 생성자 순으로 검색하여 주입한다.

</br>

Dagger의 개념이 어려운 만큼 다음 문서에서 또 다른 예시를 통해 설명을 이어가려고 한다.
