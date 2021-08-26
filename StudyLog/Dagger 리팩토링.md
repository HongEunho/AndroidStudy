## Dagger 리팩토링(1) - Component, SubComponent

AndroidInjector, ContributesAndroidInjector을 사용하기 전에는

각각의 Component가 모듈과 연결되어, 주입이 필요한 곳에 inject를 하였다.

</br>

예를 들어, 메인 액티비티에서 Burger라는 객체가 필요하다면

Burger 컴포넌트는 Burger모듈과 연결되어 모듈에서 객체를 생성한 후

컴포넌트에서 inject(mainActivity: MainActivity) 를 명시를 해줌으로써

MainActivity에서 주입을 받을 수 있도록 해야한다.

그리고 MainActivity에서는 여기가 주입받을 곳이라고 알려주기 위해 build()및 inject()코드를 작성해야 한다.

</br>

그리고 inject가 필요한 액티비티가 늘어날 때 마다 매번 inject 코드를 추가해주었고

늘어난 액티비티에서도 마찬가지로 build() 및 inject()를 추가해주었다.

</br>

### ContributesAndroidInjector

하지만 이 AndroidInjector를 이용함으로써 매번 해야하는 이 귀찮은 작업들을 줄일 수 있게 된다.

ActivityModule(혹은 ActivityBinder)에서

의존성을 주입받을 Activity들을 명시해놓고

ApplicationComponent에서 이  Module과 연결을 시킨다.

그럼, 이제 의존성을 주입받을 액티비티가 늘어날 때 마다 ActivityModule에만 추가를 해주면

귀찮게 해당 액티비티에 새롭게 추가하는 코드를 작성하지 않아도 된다.

그리고 이로 인해 보일러 플레이트 코드 또한 줄어들게 된다.