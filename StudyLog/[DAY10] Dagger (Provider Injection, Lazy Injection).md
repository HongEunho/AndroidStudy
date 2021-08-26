# [DAY10] Dagger (Provider Injection, Lazy Injection)

### Provider Injections

지금까지의 예제들은 객체를 하나씩 생성하여 채워주는 형태로 작성해보았다.

또, 멤버 변수의 객체를 채워주는 작업은 하나의 instance만 생성해서 dagger가 채워주도록 하였다.

</br>

하지만, dagger를 통해 생성되는 객체가 여러개가 필요하다면 객체를 생성해주는 Provider 자체를 받아올 수도 있다.

먼저, 다음의 예제를 살펴보자

</br>

```kotlin
class StrongHero @Inject constructor(private val person: Person) {
	val weapons = mutableListOf()
}

interface Weapon {
    fun type(): String
}

class Suit: Weapon {
    override fun type() = "수트"
}
```

먼저 StrongHero라는 클래스를 생성하였다. 지난번 Hero 클래스와는 다르게 무기를 여러개 가질 수 있는 Hero를 생성해보려고 한다.

</br>

그럼 이제 Module을 작성해보자.

```kotlin
@Module
class StringHeroModule {
    @Provides
    fun provideStrongHero(person: Person) = StrongHero(person)
    
    @Provides
    fun provideSuit(): Weapon = Suit()
    
}
```

StringHero 객체 자체를 생성해주는 provider와 Suit 객체 자체를 생성해주는 provider를 만들었다.

</br>

그럼 이를 이용해 Component를 작성해보자.

```kotlin
@Component(modules = [StrongHeroModule::class])
interface StrongHeroComponent {
	
    fun callStrongHero(): StrongHero
    
    fun getWeapon(): Weapon
    
    fun inject(main: Main)
    
    @Component.Factory
    interface Builder {
        fun create(@BindsInstance person: Person): StrongHeroComponent
    }
}
```

StrongHero 객체 자체를 생성해주며, 무기를 생성해준다.

그리고 이를 사용할 곳(main)에서 주입시킬 inject 함수를 만들어 주었다.

그리고 StrongHero객체를 생성할 때 Person 인자는 직접 생성하여 넘겨주도록 하기 위해서

**@Component.Factory**를 사용해 create() 함수를 재정의 하였다.

</br>

그럼 이제 이들을 사용해볼 main 클래스를 작성해보자.

```kotlin
class Main { 
    @Inject 
    lateinit var weaponProvider: Provider<Weapon> 
    
    fun getStrongHero() { 
        val person = IronMan() 
        val StrongHeroComponent = DaggerStrongHeroComponent.factory().create(person) StrongHeroComponent.inject(this)
        
        val StrongHero = StrongHeroComponent.callStrongHero() 
        
        (1..10).forEach { _ -> 
            val weapon = weaponProvider.get()
            StrongHero.weapons.add(weapon) 
         } 
    } 
}
```

DaggerStrongHeroComponent를 create() 하면서 생성된 IronMan 객체를 인자로 넘겨준다.

그리고 DaggerStrongHeroComponent.inject(this) 함수를 호출하여 ( 여기서 this == main )

@Inject 필드를 채우도록 한다.

따라서 Dagger는 weapon을 생성하는 **Provider 자체**를 weaponProvider 멤버변수에 inject 시켜준 것이다.

즉, 여기에서 weapon 객체를 생성한 것이 아니라

Provider에서 weapon객체를 생성하게 함으로써 Provider을 통해 weapon객체를 생성하게 된다.

</br>
그래서 weaponProvider.get() 을 통해 계속해서 새로 생성된 weapon 객체를 전달 받게 된다.

</br>

### Lazy Injection

이번에는 Provider Injection과 다르게 특정 object를 injection할 때 필요에 따라 lazy하게 생성해야 하는 경우를 살펴보자.

</br>

위의 main클래스를 lazy Injection을 사용하여 수정한 코드는 다음과 같다.

```kotlin
class Main { 
    @Inject 
    lateinit var lazyWeapon: Lazy<Weapon> 
    
    fun getStrongHero() { 
        val person = IronMan() 
        val StrongHeroComponent =
        DaggerStrongHeroComponent.factory().create(person)
        StrongHeroComponent.inject(this)
        
        val StrongHero = StrongHeroComponent.callStrongHero() 
        
        (1..10).forEach { _ -> 
            val weapon = lazyWeapon.get()
            StrongHero.weapons.add(weapon) 
         } 
    } 
}
```

위의 코드와 다른점은 Provider 대신 Lazy를 썼다는 점이다.

Lazy<T> 를 사용하게 되면 객체의 생성을 get()함수의 호출까지 지연시킬 수 있다.

첫 번째 get() 시점에 객체가 생성되며, 이후에는 get()을 통해 싱글톤 처럼 동일한 객체가 반환되게 된다.

