# [DAY10] Dagger Example2

지난 예제에 이어, 다른 예제를 한 번 더 적용해보면서 Dagger에 좀 더 익숙해지자.

```kotlin
interface Person { 
	fun name(): String 
    fun skill(): String 
} 

interface Weapon { 
    fun type(): String 
} 

class Hero(private val person: Person, private val weapon: Weapon) { 
    fun info() { 
        Log.d("doo", "name: ${person.name()} skill: ${person.skill()} | 			weapon:${weapon.type()}") 
    } 
}
```

여기서 Hero라는 클래스는 생성자로 Person과 Weapon을 인자로 받는다.

</br>

위의 코드로, 만약 아이언맨을 생성하고 싶다면 아래와 같이 할 수 있다.

```kotlin
class IronMan: Person { 
    override fun name() = "토니 스타크" 
    override fun skill() = "수트 변형" 
} 

class Suit : Weapon { 
    override fun type() = "수트" 
} 

fun main(args: Array) { 
    val person = IronMan() 
    val weapon = Suit() 
    val hero = Hero(person, weapon) 
}
```

main()에서 Hero라는 클래스를 만들기 위해 직접 아이언맨과 수트를 생성하여 hero에 넘겨주었다.

</br>

여기서, 아이언맨과 수트를 직접 생성하는 부분을 Dagger에게 위임하여 Hero 클래스를 생성하거나,

아예 Hero 클래스 생성 자체까지도 Dagger에게 위임할 수 있다.

그렇게 함으로써 hero는 person과 weapon이 어떻게 생성되는지 알 필요가 없게되고

hero까지 Dagger가 생성해준다면, hero에 어떤 인자가 들어가야 하는지 조차 알 필요가 없게 된다.

</br>

그럼 위의 코드를 가지고 Dagger를 실습해보자.

</br>

먼저 의존성을 제공하는 의존성 객체(Module)를 생성하자.

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
```

앞선 예제에서 설명했던 것 처럼, 컴포넌트에서 이 모듈을 이용할 때 @Provides로 제공된 함수들의 반환 타입을 보고 알아서 해당 함수를 찾아 호출하게 된다.

</br>

이어서 Component를 생성해보자.

```kotlin
@Component(modules = [HeroModule::class])
interface HeroComponent {
	fun callHero(): Hero
}
```

여기서, Dagger가 알아서 Hero객체를 생성하게 하려면

Hero를 생성할 때 필요한 생성자의 인자들을 주입시켜야 한다.

</br>

그래서 위의 Hero 클래스를 다음과 같이 수정하자.

앞선 예제에서는, 생성자의 인자들이 없었지만, 이번엔 생성자의 인자가 필요한 경우의 예시이다.

```kotlin
class Hero @Inject constructor(val person: Person, val weapon: Weapon) {
	fun info() {
        Log.d("doo", "name: ${person.name()} skill: ${person.skill()} | 			weapon:${weapon.type()}") 
    }
}
```

이렇게 Hero의 생성자에 @Inject 어노테이션을 붙이면 된다.

</br>

그럼 이제 의존성 객체가 필요한 곳에서 다음과 같이 사용하면 된다.

```kotlin
val hero = DaggerHeroComponent.create().callHero()
```

</br>

------

그럼 이번에는 위의 예제중 Hero 클래스를 생성자에 주입하는 방식이 아닌,

멤버 변수에 주입하는 방식으로 수정해보려고 한다. ( 앞선 Dagger Example 문서도 이 방식을 이용했다.)

</br>

```kotlin
class Hero {
    @Inject
    lateinit var person: Person
    lateinit var weapon: Weapon
    
    fun info() {
        Log.d("doo", "name: ${person.name()} skill: ${person.skill()} | 			weapon:${weapon.type()}") 
    }
}
```

이렇게 생성자가 아닌 멤버변수에 @Inject를 달고 주입을 받는다.

</br>

그리고, HeroModule은 다음과 같이 변경해야 한다.

```kotlin
@Module
class HeroModule {
    @Provides
    fun providePerson(): Person = IronMan()
    
    @Provides
    fun provideWeapon(): Weapon = Suit()
    
//    @Provides
//    fun provideHero(person: Person, weapon:Weapon) = Hero(person, weapon)
}
```

여기서 @Provides를 주석처리하여 Hero 객체를 직접 생성하는 provider를 제거한다.

Hero클래스에서 생성자 주입 방식을 제거하였고, 멤버변수에 @Inject를 해놓았기 때문이다.

</br>

그럼, HeroComponent도 다음과 같이 변경되어야 할 것이다.

```kotlin
@Component(modules = [HeroModule::class])
interface HeroComponent {
	//fun callHero(): Hero
    fun inject(hero: Hero)
}
```

</br>

그리고 이렇게 사용한다.

```kotlin
val hero = Hero()
DaggerHeroComponent.create().inject(hero)
```

</br>

그럼 다음 문서에서 Qualifier, instance binding, @Named, @BinsInstance 등

Dagger의 좀 더 심화 기능에 대해 다뤄보고자 한다.