# [DAY10] Dagger (Qualifier Named BinsInstance)

지난 문서에서 생성했던 코드를 가지고 좀 더 심화된 내용을 살펴보자

```kotlin
interface Person { 
	fun name(): String 
    fun skill(): String 
} 

interface Weapon { 
    fun type(): String 
} 

class IronMan: Person { 
    override fun name() = "토니 스타크" 
    override fun skill() = "수트 변형" 
} 

class Suit : Weapon { 
    override fun type() = "수트" 
} 


class Hero @Inject constructor(private val person: Person, private val weapon: Weapon) { 
    fun info() { 
        Log.d("doo", "name: ${person.name()} skill: ${person.skill()} | 			weapon:${weapon.type()}") 
    } 
}
```

```kotlin
@Module
class HeroModule {
    @Provides
    fun providePerson(): Person = IronMan()
    
    @Provides
    fun provideWeapon(): Weapon = Suit()
    
    @Provides
    fun provideHero(person: Person, weapon:Weapon) = Hero(person, weapon)
}

@Component(modules = [HeroModule::class])
interface HeroComponent {
	fun callHero(): Hero
}

fun main(args: Array) { 
    val hero = DaggerHeroComponent.create().callHero()
    val hero2 = DaggerHeroComponent.builder().build().callHero()
}
```

</br>

### Qualifiers

이번에는, Hero 객체를 아이언맨과 더불어 캡틴아메리카를 생성하고자 한다.

그럼 다음과 같이 추가할 수 있다.

```kotlin
class CaptinAmerica: Person {
	override fun name() = "스티브 로저스" 
    override fun skill() = "방패 투척" 
}

class Shield: Weapon {
    override fun type() = "방패"
}
```

따라서, 지난번 만든 HeroModule에 다음을 더 추가해야 한다.

```kotlin
@Module
class HeroModule {
    @Provides
    fun provideIronman(): Person = IronMan()
    
    @Provides
    fun provideSuit(): Weapon = Suit()
    
    @Provides
    fun provideCaptinAmerica(): Person = CaptainAmerica()
    
    @Provides
    fun provideShield(): Weapon = Shield()
    
    @Provides
    fun provideHero(person: Person, weapon:Weapon) = Hero(person, weapon)
}
```

하지만 main() 함수에서 이 모듈을 보고, provider의 return type만 보고 주입할 객체를 선택하게 되는데 Hero 객체의 인자로 어떤 클래스를 넣어야 할지를 모르게 된다.

왜냐하면, IronMan과 Captain 모두 Person interface를 return 하게 되어있으며

Suit와 Shield 모두 Weapon interface를 반환하기 때문이다.

</br>

따라서 @Named 어노테이션을 이용해 각각의 provider에 이름을 부여하자.

```kotlin
@Module
class HeroModule {
    @Provides
    @Named("ironMan")
    fun provideIronman(): Person = IronMan()
    
    @Provides
    @Named("suit")
    fun provideSuit(): Weapon = Suit()
    
    @Provides
    @Named("captainAmerica")
    fun provideCaptinAmerica(): Person = CaptainAmerica()
    
    @Provides
    @Named("shield")
    fun provideShield(): Weapon = Shield()
    
    @Provides
    @Named("heroIronMan")
    fun provideHeroIronMan(@Named("ironMan") person: Person, @Named("suit") weapon:Weapon) = Hero(person, weapon)
 	@Named("heroCaptainAmerica")
    fun provideHeroCaptainAmerica(@Named("captainAmerica") person: Person, @Named("shield") weapon:Weapon) = Hero(person, weapon)
}
```

이렇게 변경을 하고 나면, Component도 변경을 해야 한다.

</br>

```kotlin
interface HeroComponent {
	@Named("heroIronMan")
    fun callIronMan(): Hero
    
    @Named("heroCaptainAmerica")
    fun callCaptainAmerica(): Hero
}
```

</br>

그럼 이제 메인문에서 이렇게 호출을 할 수 있게 된다.

```kotlin
fun main() {
    val heroIronMan = DaggerHeroComponent.create().callIronMan()
    val heroCaptain = DaggerHeroComponent.create().callCaptainAmerica()
}
```

이 방법 역시, 지난번 문서처럼 생성자 주입 대신 Field Injection을 사용하고 싶다면

Hero class를 다음과 같이 변경하여 사용할 수 있다. ( 물론 모듈과 컴포넌트도 같은 방식으로 변경해야 함 )

```kotlin
class Hero {
    @Inject
    @Named("ironMan")
    lateinit var person: Person
    
    @Named("suit")
    lateinit var weapon: Weapon
    
    fun info() {
        Log.d("doo", "name: ${person.name()} skill: ${person.skill()} | 			weapon:${weapon.type()}") 
    }
}
```

</br>

### Binding Instances

Module을 작성하다 보면 객체 생성시 매개변수를 외부에서 전달받아야 하는 경우도 있다.

위의 예제들은 모두 객체생성시 필요한 매개변수를 Module 내부에서 정의했지만

생성 시점에 따라 가변적으로 바뀌어야 하는 경우나 생성을 못하는 경우도 있기 때문이다.

</br>

이런 경우에는 이미 생성된 객체를 Dagger에게 넘겨주어야 한다.

그 예를 한번 살펴보자.

이번에는 Hulk 클래스를 추가해보자.

```kotlin
class Hulk: Person {
    override fun name() = "브루스 배너"
    override fun skill() = "육탄전"
}
```

여기서, Hero 클래스는 생성자로 Person객체와 Weapon 객체를 받아야 하지만

Hulk는 무기가 없다.

하지만 외부로부터 헐크 수트를 넘겨받을 수 있다고 가정해보자.

즉, Dagger가 직접 Weapon 객체를 생성하지는 못하지만 외부에서 생성된 Weapon 객체를 넘겨받도록 해보자.

```kotlin
class HulkBuster: Weapon {
	override fun type() = "헐크버스터"
}
```

이렇게 헐크가 넘겨받을 무기를 나타내는 HulkBuster 클래스를 생성하였다.

하지만 이 클래스는 Module에서 정의하지 않고, 외부에서 주입을 받도록 할 것이다.

</br>

```kotlin
class HeroModule {
    ...
    @Provides
    @Named("hulk")
    fun provideHulk(): Person = Hulk()
    ...
    @Provides
    @Named("heroHulkGetWeapon")
    fun provideHeroHulkGetWeapon(@Named("hulk") person: Person, weapon: Weapon) = Hero(person, weapon)
}
```

코드를 보면 알 수 있듯이 providerHeroHulkGetWeapon()에서 weapon 인자에는 아무런 어노테이션을 넣지 않았다.

그럼 어떻게 받아올 수 있을까?

그 답을 Component에서 살펴보자

```kotlin
interface HeroComponent {
	@Named("heroIronMan")
    fun callIronMan(): Hero
    
    @Named("heroCaptainAmerica")
    fun callCaptainAmerica(): Hero
    
    @Named("heroHulkGetWeapon")
    fun callHulkWithWeapon(): Hero
    
    @Component.Builder
    interface Builder {
        fun setHulkWeapon(@BindsInstance hulkWeaponProvider: Weapon): Builder
        fun build(): HeroComponent
    }
}
```

기본적으로 Dagger는 빌더에 대해서 아무것도 언급하지 않으면 기본 Builder를 자동 생성한다.

하지만, 여기서는 필요한 객체를 넘겨야 하기 때문에 Dagger 호출 시 Builder에 객체를 넘겨줄 수 있도록 Builder를 재구성 하였다.

</br>

그리고 @BindsInstance 어노테이션을 사용하여 setHulkWeapon() 이란 함수로 Weapon을 헐크에게 넘겨줄 것이라는 것을 Dagger에게 알려주었고

Builder가 재생성 되어야 함을 알려주기 위해 @Component.Builder를 사용하였다.

</br>
그럼 이제 실제로 객체를 호출하는 부분은 다음과 같이 작성될 것이다.

```kotlin
val hulkBuster = HulkBuster()
val hulkWithWeapon = DaggerHeroComponent.builder().setHulkWeapon(hulkBuster).build().callHulkWithWeapon()
```



그럼 다음 문서에서는 Dagger의 Provider Injection과 Lazy Injection을 살펴보도록 하자.

