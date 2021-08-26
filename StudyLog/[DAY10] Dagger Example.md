# [DAY10] Dagger Example

지난번 문서에서 학습한 Dagger의 필수 5가지 개념을 이용해 직접 예시를 작성해보자.

1. Inject
2. Component
3. Subcomponent
4. Module
5. Scope

이렇게 5가지가 있었고. 그 중 가장 중요한 것은 **Module**과 **Component**라고 할 수 있다.

</br>

**Module**은 Component에 연결되어 의존성 객체를 생성하고,

**Component**는 Module과 연결되어, 연결된 Module을 이용해 의존성 객체를 생성한다.

그리고 Inject로 요청받은 인스턴스에, 생성한 객체를 주입한다는 개념을 기억하자.



### Example

</br>

**MyModule.java**

```java
@Module // 의존성을 제공하는 클래스에 붙인다
public class MyModule {
	@Provides // 의존성을 제공하는 메소드에 붙인다
	String provideHelloWorld() {
		return "Hello World : 의존성 주입!!"
	}
}
```

의존성을 제공하는 의존성 객체를 생성하는 과정이다.

이 객체의 provideHelloWorld() 의존성 메소드는 String 형으로 반환한다.

</br>

**MyComponent.java**

```java
//MyComponent 인터페이스 내에는 제공할 의존성들을 메서드로 정의해야 한다

//@Component 에 참조된 모듈 클래스로부터 의존성을 제공받음
//여기서는 MyModule.class 가 참조된 모듈 클래스이다.

@Component(modules = MyModule.class)
//Component 메서드의 반환형을 보고 모듈과 관계를 맺으므로,
//바인드된 모듈로부터 해당 반환형을 갖는 메서드를 찾지 못하면 컴파일 타임 에러 발생
//여기서는 MyModule 의 String provideHelloWorld() 함수의 return "Hello World"; 에서 String 반환형을 찾게 된다.

public interface MyComponent {
    String getString(); //프로비전 메서드, 바인드된 모듈로부터 의존성을 제공
}
```

</br>

**MainActivity.java**

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        MyComponent myComponent = DaggerMyComponent.create();
        TextView tv1 = findViewById(R.id.tv1);
        tv1.setText(myComponent.getString());

    }
}
```

여기서는 이렇게 myComponent에서 getString을 호출하게 되는데

위에서 만든 MyComponent를 보게 되면, 컴포넌트 내부에 String을 반환하는 getString이라는 메소드가 있다.

이때, 컴포넌트는 MyModule과 연결되어 있으므로

MyModule로 거슬러 올라가서 String 타입을 반환하는 String provideHelloWorld()을 호출하게 되므로 결국 textView는 "Hello World : 의존성 주입!!" 으로 setText 된다.

</br>

여기까지가 Dagger의 flow를 이해하는 간단한 예제이다.

이 예제가 이해 되었다면 코틀린으로 좀 더 확장된 예제를 살펴보자.

</br>

먼저, 다음과 같은 클래스 두개를 만들자.

```kotlin
public class Cat {
	private val catName = "RussianBlue"
	fun getCatName() : String {
		return catName
	}
}
```

</br>

```kotlin
public class Dog {
	private val dogName = "Puddle"
	fun getDogName() : String {
		return dogName
	}
}
```

</br>

그 다음 위에서 만든 Cat 클래스를 제공할 모듈을 만들자.

```kotlin
@Module
object CatModule {
	
	@Provides
	fun provideCat() : Cat {
		return Cat()
        // 자바는 return new Cat()
	}
}
```

이 코드는 SingleTon으로 CatModule을 만들고, @Module 어노테이션을 붙임으로써 의존성을 제공하는 객체임을 나타낸다.

그리고 Cat객체를 생성하여 return 하며, @Provides를 붙임으로써  의존성을 제공하는 메소드임을 나타낸다.

</br>

마찬가지로 Dog 클래스를 제공할 모듈을 만들자.

```kotlin
object DogModule {
	
	@Provides
	fun provideDog() : Dog {
		return Dog()
	}
}
```

</br>

그리고 이제 모듈을 사용할 인터페이스 (Component) 를 만들자.

여기서, 모듈을 연결하여 의존성 객체를 생성하고 @Inject로 요청받은 인스턴스에 의존성 객체를 주입하게 된다. ( 이게 Component의 역할이다. )

```kotlin
@Singleton
@Component(modules = [DogModule::class, CatModule::class])
interface PetComponent {
	fun inject(mainActivity: MainActivity)
    
    @Component.Builder
    interface Builder {
        fun build() : PetComponent
        
        fun dogModule(dogModule: DogModule) : Builder
        fun catModule(catModule: CatModule) : Builder
    }
}
```

위 코드는 먼저, DogModule과 CatModule을 PetComponent에 연결시킨다.

그리고 연결된 모듈을 사용할 인스턴스(MainActivity)에 주입하는 Inject 함수를 만든다.

</br>

그리고 추가적으로, @Component.Builder 부분은

MainActivity와의 연결을 도울 Builder이다. 

</br>

이제 MainActivity에 의존성을 주입시켜 의존성 객체를 사용해보자.

```kotlin
class MainActivity: AppCompatActivity() {
	@Inject
	lateinit var cat: Cat
	lateinit var dog: Dog
	
	override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        injectComponent()

        setGetCatNameButton()
        setGetDogNameButton()
    }
    
    private fun injectComponent() {
        DaggerPetComponent.builder()
                .catModule(CatModule)
                .dogModule(DogModule)
                .build()
                .inject(this)
    }

    private fun setGetCatNameButton() {
        catNameButton.setOnClickListener {
            catNameText.text = cat.getCatName()
        }
    }
    private fun setGetDogNameButton() {
        dogNameButton.setOnClickListener {
            dogNameText.text = dog.getDogName()
        }
    }
}
```

위의 코드를 살펴보면 다음과 같다.

1. @Inject 어노테이션을 사용해 의존성 객체를 주입한다.
2. MainActivity 생성 시 Component를 연결한다. ( 연결 전 빌드를 한 번 해야 연결 된다 )
3. Component 연결 시 사용할 Module을 지정한다.
4. inject(this)를 통해 의존성 주입을 요청한다.
5. 그럼 위에 @Inject 어노테이션이 정의되어 있는 객체 ( cat, dog)가 주입된다.
6. button 이벤트를 발생하면 의존성 객체의 메소드를 사용하여 이름을 불러온다.

즉, MainActivity에서 Dog와 Cat 객체를 직접 생성하지 않고도 메소드를 사용할 수 있게 된다.
