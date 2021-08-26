# [DAY10] Dagger (Scope, Singleton, SubComponent)

### Singleton

앞선 예제의 Component는 inject 시점에 매번 새로운 객체를 생성해서 반환한다.

또, 직접 객체를 반환하는 함수의 경우에도 항상 새로운 객체를 반환하게 된다.

</br>
하지만 Singleton처럼 객체를 한 번만 생성하고 그 이후엔 동일한 객체를 반환하도록 해야 하는 경우가 있다.

예를 들면, Room을 사용해 DB를 생성하면 RoomDB에 접근해 Database객체를 받아오게 되는데

이 Database객체는 한 번만 생성하고, 그 이후에는 생성된 객체를 이용해 사용하도록 권고 된다.

</br>
이러한 경우 처럼 생성 시 한 개의 객체만 생성하는 싱글톤에 대해 알아보자.

```kotlin
class Avengers {
	lateinit var ironMan: Hero
	lateinit var captainAmerica: Hero
	lateinit var hulk: Hero
	
	fun info() {
		println("Avengers Assemble!")
	}
}
```

</br>
그리고 이전 예제에서 사용했던 Hero 관련 클래스를 그대로 가져왔다.

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

class CaptinAmerica: Person {
	override fun name() = "스티브 로저스" 
    override fun skill() = "방패 투척" 
}

class Shield: Weapon {
    override fun type() = "방패"
}

class Hulk: Person {
    override fun name() = "브루스 배너"
    override fun skill() = "육탄전"
}

class HulkBuster: Weapon {
	override fun type() = "헐크버스터"
}

class Hero @Inject constructor(private val person: Person, private val weapon: Weapon) { 
    fun info() { 
        Log.d("doo", "name: ${person.name()} skill: ${person.skill()} | 			weapon:${weapon.type()}") 
    } 
}
```

예시가 [투덜이님 블로그](https://tourspace.tistory.com/330?category=797357)에서 정말 잘 설명되어 있어 블로그를 참조했다.

어벤져스는 세 명의 Hero로 구성되며 어벤져스는 하나이다.

따라서, 어디에서 호출 하든지 동일한 어벤져스가 나와야 한다.

</br>
먼저, Avengers를 호출하는 Module를 만들어보자.

```kotlin
@Module
class AvengersModule {
    @Singleton
    @Provides
    fun provideAvengers(): Avengers {
        return Avengers()
    }
}
```

어벤져스를 생성해주는 Provides를 만들었다.

여기서 이전 예제와의 차이점은 이 함수에 @Singleton 어노테이션을 붙인 점이다.

이 어노테이션의 역할은 뒤의 Component까지의 코드를 본 후 설명을 이어가려고 한다.

</br>

```kotlin
@Singleton
@Component(modules = [AvengersModule::class])
interface AvengersComponent {
    fun getAvengers(): Avengers
}
```

Component를 이렇게 구성하여 간단하게 Avengers 객체를 반환할 수 있다.

</br>

그리고 이들을 이용해 다음과 같이 호출할 수 있다.

```kotlin
val avengers = DaggerAvengersComponent.create()
println("avengers1:${avengers.getAvengers()} avengers2:${avengers.getAvengers()}")
```

이때 출력되는 avengers의 hash정보는 다음 처럼 동일하게 출력된다.

avengers1:????Avengers@5d5c505 avengers2:????Avengers@5d5c505

즉, 동일한 객체라는 뜻이다.

</br>

하지만 만약 다음과 같이 코드를 작성했다면 어떤 결과가 나올까?

```kotlin
val avangers1 = DaggerAvengersComponent.create()
val avangers2 = DaggerAvengersComponent.create()
println("avengers1:${avengers.getAvengers()} avengers2:${avengers.getAvengers()}")
```

결과는 avengers1: ????Avengers@5d3c203 avengers2: ????Avengers@3c5d605 처럼 다르게 나오게 된다.

즉, 다른 객체라는 뜻이다.

그럼 왜 이런 결과가 나타났을까?

</br>

Dagger에서는 Scope라는 개념이 존재하는데 이를 통해 생명주기를 같이 할 객체들을 묶을 수 있다.

만약, Activity 내부에서 inject되는 변수가 있고 이 멤버 변수의 생성을 Activity의 생명주기와 같이 하고 싶을 때, Scope를 이용할 수 있다.

</br>
@Singleton이 이 Scope를 이용한 개념인데

@Singleton은 보통 Component에 붙여서 Component와 생명주기를 같이하도록 할 때 사용한다.

</br>

### SubComponent

Dagger는 SubComponent를 지원하는데, 이를 이용해 Component끼리 부모-자식 관계를 맺을 수 있다.

다음 예시 코드를 통해 어떻게 사용하는지 살펴보자.

지난 예제에서 작성했던 HeroModule을 활용한 코드이다.

```kotlin
@Module
class HeroModule2 {
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
    @Named("hulk")
    fun provideHulk(): Person = Hulk()
    
    @Provides
    @Named("hulkBuster")
    fun provideHulkBuster(): Weapon = HulkBuster()
    
    @Provides
    @Named("heroIronMan")
    fun provideHeroIronMan(
        @Named("ironMan") person: Person, 
        @Named("suit") weapon:Weapon
    ) = Hero(person, weapon)
    
    @Provides
 	@Named("heroCaptainAmerica")
    fun provideHeroCaptainAmerica(
        @Named("captainAmerica") person: Person, 
        @Named("shield") weapon:Weapon
    ) = Hero(person, weapon)
    
    @Provides
    @Named("heroHulk")
    fun provideHeroHulk(
        @Named("hulk") person: Person,
    	@Named("hulkBuster") weapon: Weapon
    ) = Hero(person, weapon)
}
```

먼저 Avengers 클래스에 Hero 멤버변수를 주입시키기 위한 모듈을 만들었다.

아이언맨, 캡틴아메리카, 헐크를 생성해주는 함수와 각각의 무기를 생성해주는 함수를 만들었고

이를 이용해 각각의 Hero들을 만드는 함수를 만들었다.

사람과 무기는 동일한 Person, Weapon 객체를 return하기 때문에 각각의 Hero 생성 함수에서 인자에 어떤 값을 넣어줄지를 명시하기 위해 @Named 어노테이션을 이용했다.

</br>

그럼 이제 Hero를 생성해주는 함수들을 모아놓는 Component를 작성해보자.

이 Component는 AvengersComponent의 SubComponent로 종속시킬 것이기 때문에

@Subcomponent를 사용해 만든다.

```kotlin
@Subcomponent(modules = [HeroModule2::class])
interface HeroSubComponent {
	@Named("heroIronMan")
    fun callIronMan(): Hero
    
    @Named("heroCaptainAmerica")
    fun callCaptainAmerica(): Hero
    
    @Named("heroHulkGetWeapon")
    fun callHulk(): Hero
    
    fun inject(avengers: Avengers)
    
    @Subcomponent.Builder
    interface Builder {
        fun build(): HeroSubComponent
    }
}
```

여기서 주의해야 할 점은 SubComponent는 Builder를 정의해 놓지 않으면 외부에서 접근을 할 수 없기 때문에 **반드시 Builder를 생성**해놔야 한다.

이렇게 SubComponent는 각각의 Hero를 생성하는 함수와 외부에서 Avengers 객체를 받았을 때, 내부를 채워줄 inject 함수를 추가로 정의하였다.

</br>

다음으로, HeroModule2보다 상위 개념인 AvengersModule을 작성해보자.

```kotlin
@Module(subcomponents = [HeroSubComponent::class])
class AvengersModule {
    @Singleton
    @Provides
    fun provideAvengers(): Avengers {
        return Avengers()
    }
}
```

부모와 자식간의 연결을 표시하기 위해 Module에 subcomponent 속성을 이용하였다.

SubComponent와 Component의 종속 관계는 Component에 표기해야 할 것 같지만

Module에 이를 정의하도록 되어있다.

</br>
Dagger에서 Module은 객체를 생성하는 역할을 하며 Component는 외부에서 Dagger를 사용하기 위한 함수를 노출해주는 interface 역할을 한다.

객체생성은 Module이 담당하기 때문에, 상위 Module이 하위 Module의 객체 생성에 대한 역할까지 가지게 되며 하위 모듈에서 객체를 생성하기 위해서는 하위 Component를 이용하게 되므로

상위 Module과 하위 Component를 연결되도록 설계된 것 같다.

</br>

다음으로 상위 컴포넌트인 AvengersComponent를 작성해보자.

```kotlin
@Singleton
@Component(modules = [AvengersModule::class])
interface AvengersComponent {
    fun getAvengers(): Avengers
    fun heroSubComponent(): HeroSubComponent.Builder
}
```

Avengers의 Component에는 하위 모듈에서 제공하는 생성 기능을 사용하기 위해 Builder를 return해주는 함수를 추가해주어야 한다.

</br>

마지막으로, 위에서 작성했던 Avengers 클래스를 하위 모듈의 inject를 이용하기 위해 다음처럼 변경하자.

```kotlin
class Avengers {
    
    @Inject
    @Named("heroIronMan")
	lateinit var ironMan: Hero
    
    @Inject
    @Named("heroCaptainAmerica")
	lateinit var captainAmerica: Hero
    
    @Inject
    @Named("heroHulk")
	lateinit var hulk: Hero
	
	fun info() {
		println("Avengers Assemble!")
	}
}
```

 이렇게 상위/하위 들을 모두 연결하였으므로 이를 호출해보자.

```kotlin
val avengersComponent = DaggerAvengersComponent.create()
val avengers = avengersComponent.getAvengers()

avengersComponent.heroSubComponent().build().inject(avengers)

println("info ${avengers.ironMan.info()} | ${avengers.captainAmerica.info()} | ${avengers.hulk.info()}")
```

그럼 정상적으로 각각의 Hero들의 정보가 출력된다.

</br>

### 상위 / 하위 모듈의 Scope 분할

위에서 상위 / 하위 모듈을 정의했으니 이제 각각의 Scope를 나누어 각 객체들의 생명주기를 확인해보자.

Avengers는 상위 개념이므로 이미 정의된 @Singleton을 유지한다.

Hero의 구성요소 중 사람은 유일하고, 무기는 변경이 가능하다.

따라서 아이언맨, 캡틴아메리카, 헐크 같은 Person 객체는 한 번만 생성되고 그 이후엔 공유되어야 한다.

반면 아이언맨 수트와 아이언맨이 만들어준 헐크 버스터는 호출할 때 마다 새로 생성되도록 해야하며 캡틴의 방패는 세상에 하나 뿐이므로 한 번만 생성되고 다음 호출시에는 동일한 객체가 반환되어야 한다.

```kotlin
@Scope
@MustBeDocumented
@Kotlin.annotation.Retention(AnnotationRetention.RUNTIME)
annotation class Heros
```

이런 생명주기를 한정하기 위해 위와 같이 Heros 라는 Scope를 하나 생성하였다.

(자바의 경우에는 기존처럼 @Singleton 을 이용하면 된다고 한다.)

</br>
그리고 생성을 한정 할 Module에 @Heros 어노테이션을 붙여주자.

```kotlin
@Module
class HeroModule2 {
    @Provides
    @Heros
    @Named("ironMan")
    fun provideIronman(): Person = IronMan()
    
    @Provides
    @Named("suit")
    fun provideSuit(): Weapon = Suit()
    
    @Provides
    @Heros
    @Named("captainAmerica")
    fun provideCaptinAmerica(): Person = CaptainAmerica()
    
    @Provides
    @Heros
    @Named("shield")
    fun provideShield(): Weapon = Shield()
    
    @Provides
    @Heros
    @Named("hulk")
    fun provideHulk(): Person = Hulk()
    
    @Provides
    @Named("hulkBuster")
    fun provideHulkBuster(): Weapon = HulkBuster()
    
    @Provides
    @Named("heroIronMan")
    fun provideHeroIronMan(
        @Named("ironMan") person: Person, 
        @Named("suit") weapon:Weapon
    ) = Hero(person, weapon)
    
    @Provides
 	@Named("heroCaptainAmerica")
    fun provideHeroCaptainAmerica(
        @Named("captainAmerica") person: Person, 
        @Named("shield") weapon:Weapon
    ) = Hero(person, weapon)
    
    @Provides
    @Named("heroHulk")
    fun provideHeroHulk(
        @Named("hulk") person: Person,
    	@Named("hulkBuster") weapon: Weapon
    ) = Hero(person, weapon)
}
```

</br>

그리고 SubComponent에서도 Class 레벨에 @Heros를 붙여준다.

```kotlin
@Heros
@Subcomponent(modules = [HeroModule2::class])
interface HeroSubComponent {
	@Named("heroIronMan")
    fun callIronMan(): Hero
    
    @Named("heroCaptainAmerica")
    fun callCaptainAmerica(): Hero
    
    @Named("heroHulk")
    fun callHulk(): Hero
    
    fun inject(avengers: Avengers)
    
    @Subcomponent.Builder
    interface Builder {
        fun build(): HeroSubComponent
    }
}
```

이렇게 함으로써, HeroSubComponent가 생성되면 해당 Component에서 생성되는 IronMan, CaptainAmerica, Hulk, Shield 는 딱 하나만 생성되어 생성 요청 시 동일한 객체가 반환되게 된다.

</br>

그럼 테스트를 진행해보자

```kotlin
val avengersComponent = DaggerAvengersComponent.create() 
val heroComponent = avengersComponent.heroSubComponent().build()

val avengers1 = avengersComponent.getAvengers() 
heroComponent.inject(avengers1) 

println("avengers1: ${avengers1} | ${avengers1.ironMan} | ${avengers1.ironMan.person} | ${avengers1.ironMan.weapon}") 



val avengers2 = avengersComponent.getAvengers() 
heroComponent.inject(avengers2) 

println("avengers2: ${avengers2} | ${avengers2.ironMan} | ${avengers2.ironMan.person} | ${avengers2.ironMan.weapon}")
```

위 코드의 테스트 결과는 아래와 같다. (참조 : [투덜이님 블로그](https://tourspace.tistory.com/330?category=797357))

```kotlin
avengers1: com.dbjt.mylittleworld.sample.Avengers@c715b9 | com.dbjt.mylittleworld.sample.Hero@378cdfe | com.dbjt.mylittleworld.sample.IronMan@9a7495f | com.dbjt.mylittleworld.sample.Suit@30f5fac

avengers2: com.dbjt.mylittleworld.sample.Avengers@c715b9 | com.dbjt.mylittleworld.sample.Hero@7ba5475 | com.dbjt.mylittleworld.sample.IronMan@9a7495f | com.dbjt.mylittleworld.sample.Suit@78fcc0a
```

결과를 보면 알 수 있듯이, AvengersComponent와 HeroComponent는 각각 하나씩만 생성하여 avengers1, avengers2를 만든다.

</br>

위의 결과 과정 정리

##### avengers1, avengers2 객체의 생성과정

- **AvengersComponent**의 **getAvengers()**는 **AvengersModule**의 **ProvideAvenger()** 함수를 통해서 생성된다.
- avengers1과 avengers2는 **@Singleton**으로 **AvengersComponent**와 생명주기를 같이 한다.
- 따라서 위 코드처럼 avengersComponent를 생성하고, 해당 component로 만드는 Avengers 객체는 전부 동일한 객체가 된다.
- 결과에서 보듯이 avengers1과 avengers2는 동일한 객체이다.

</br>

##### Avengers의 멤버 변수인 Hero 객체 생성과정

- AvengersComponent를 이용하여 subComponent(HeroSubComponent)인 heroComponent를 하나 만든다.
- 이를 이용해 Avengers 객체에 hero를 inject 시킨다.
- 각각의 Hero를 생성하는 모듈의 함수들에는 Scope Annotation이 없으므로 매번 새로운 객체가 생성된다. ( Person객체가 아닌 Hero객체를 만드는 provideHeroIronMan 함수 )

</br>

##### Hero를 구성하는 Person, Weapon 객체들의 생성

- HerosModule2에 provideIronMan()은 @Heros 어노테이션으로 지정되어 있고,

  provideSuit()는 어노테이션으로 지정하지 않았다.

- 따라서, IronMan객체는 한 번만 생성되며 이후 생성요청 시 동일한 객체를 반환한다.

- 반면, Suit()는 생성요청때 마다 매 번 새로운 객체가 생성된다.

</br>

이렇게 Component와 Module의 생명주기는 Scope 어노테이션을 이용해 묶을 수 있다.

