# [DAY10] Multibinding @IntoSet @IntoMap

지난 문서에 이어 Dagger의 심화 기능에 대해 알아보려고 한다.

### Set multibinding

```kotlin
class HeroFriends {
    @Inject
    lateinit var heroFriends: Set<@JvmSuppressWildcards Person>
    
    @Inject
    lateinit var heroWeapons: Set<@JvmSuppressWildcards Weapon>

}
```

이 HeroFriends 클래스는 Heros들의 person 객체를 주입받고 weapon 객체도 주입받는다.

즉, 단순 클래스가 아니라 collection을 주입받는 형태이다.

참고로, @JvmSuppressWildcards는 kotlin에서 명확한 제네릭 타입을 입력하지 않으면 발생하는 오류이기 때문에 붙여주었다. 자바에서는 붙이지 않아도 된다.

</br>
그럼 이들을 채워줄 수 있는 모듈을 작성해보자.

```kotlin
@Module 
class HerosSetModule { 
    
    @Provides 
    @IntoSet 
    fun provideIronMan(): Person = IronMan() 
    
    @Provides 
    @IntoSet 
    fun provideCaptainAmerica(): Person = CaptainAmerica() 
    
    @Provides 
    @IntoSet 
    fun provideHulk(): Person = Hulk() 
    
    @Provides 
    @ElementsIntoSet 
    fun provideWeapons() = setOf(Suit(), Shield(), HulkBuster()) }

```

@IntoSet 어노테이션을 이용해 Person객체를 담는 set에 해당 객체를 담는다고 선언하였다.

또, 한 번에 여러개의 인자를 담으려면

@ElementsIntoSet 어노테이션을 이용하면 된다.

위의 코드에서는 Person set은 @IntoSet을 이용하였고 Weapon set은 @ElementsIntoSet을 이용하였다.

</br>

그럼 Component를 다음과 같이 작성할 수 있다.

```kotlin
@Component (modules = [HerosSetModule::class])
interface HerosSetComponent {
    
    fun heroFriends(): Set<Person> 
    fun heroWeapons(): Set<Weapon> 
    fun inject (heroFriends: HeroFriends)
    
}
```

이 컴포넌트에서, HeroFriends 객체에 주입시키기 위해 inject 함수를 정의했다.

여기서, Set 자체를 반환하는 heroFriends()와 heroWeapons()는 호출 부분에서 직접 set을 꺼내오기 위해 추가로 정의한 것이지 inject하는데는 필요하지 않다.

</br>
이제 main문에서 위의 자료들로 실험해보자.

```kotlin
val setComponent = DaggerHerosSetComponent.create() 
val heroFriends = HeroFriends() 
setComponent.inject(heroFriends)

heroFriends.heroFriends.forEach { println("name: ${it.name}") } heroFriends.heroWeapons.forEach { println("type: ${it.type}") } 

println("${setComponent.heroFriends().size},
        ${setComponent.heroWeapons().size}") 
```

</br>

### Map multibinding

이번에는 key, value가 존재하는 map형태로 모듈을 작성해보자.

```kotlin
@Module 
class HerosMapModule { 
    @Provides 
    @IntoMap 
    @StringKey("ironMan") 
    fun provideWeaponsForIronMan() : Weapon { 
        return Suit() 
    } 
    
    @Provides 
    @IntoMap 
    @StringKey("captainAmerica") 
    fun provideWeaponsForCaptain() : Weapon { 
        return Shield() 
    } 
    
    @Provides 
    @IntoMap 
    @ClassKey(Shield::class) 
    fun providePersonForShield() : Person { 
        return CaptainAmerica() 
    } 
    
    @Provides 
    @IntoMap 
    @ClassKey(Suit::class) 
    fun providePersonForSuit() : Person { 
        return IronMan() 
    } 
}
```

@IntoMap으로 어노테이션을 표기함으로써 Map에 넣을 정보라는 것을 명시해주었다.

또 @StringKey를 통해 키값을 정해 주었다.

이 때, 만약 Key를 Class형태로 사용하려면 @ClassKey를 사용하면 된다.

</br>

그럼 이어서 Component를 작성해보자.

```kotlin
@Component (modules = [HerosMapModule::class])
interface HerosMapComponent {
    
    fun personMap(): Map<String, Weapon>
    fun weaponMap(): Map(Class<*>, Person)
    
}
```

</br>
그리고 다음과 같이 호출하면 된다.

```kotlin
val mapComponent = DaggerHerosMapComponent.create()

println(mapComponent.personMap()["ironMan"]?.type()) println(mapComponent.personMap()["captainAmerica"]?.type()) println(mapComponent.weaponMap()[Suit::class.java]?.name()) println(mapComponent.weaponMap()[Shield::class.java]?.name())
```

