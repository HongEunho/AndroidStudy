# [Day01] Kotlin 기초 문법

## 학습내용

- Kotlin의 Scope Function
- Scope Function 이용한 자바와 Kotlin 코드의 차이점 이해

## Scope Function

- 코틀린 표준 라이브러리에서 제공하는,

  객체의 Context 내에서 코드 블럭을 실행하는 것을 목적으로 만든 함수

- let, run, with, apply, also 의 5가지가 존재

- let, run, with는 람다식의 결과를 반환하며

  apply, also는 컨텍스트 객체를 반환한다.



## let

`let()` 함수는 호출한 객체를 이어지는 함수 블록의 인자로 전달한다.

그리고 그 함수 블록의 결과를 반환한다.

그래서 주로 **객체 결과값에 함수를 호출**하는 경우에 사용한다.

```kotlin
val sumNumberStr = number?.let{
    "${sum(10, it)}"
}.orEmpty()
```

위의 코드처럼, number 객체에 sum함수를 호출하여 sum의 결과값을 반환한다.



scope함수를 이용하지 않은 자바 코드의 경우 다음과 같다.

```java
Integer number = null;
String sunNumberStr = null;
if (number != null){
    sumNumberStr = "" + sum(10, number);
} else{
    sumNumberStr = "";
}
```



두 코드를 비교해보면, 확실히 가독성 측면에서 차이가 있다.

scope함수를 이용하면 연관된 코드를 블럭안에 묶음으로써 가독성을 높일 수 있고,

유효범위를 명확히 알 수 있게 된다.

또한 nullable에 대한 처리로 인해 안전성 또한 높일 수 있다.



## run

`run()` 함수는 변수를 넣고 그 변수로 반환값을 바로 얻고싶을 때 주로 사용한다.

```kotlin
val width = run{
    val point = Point()
    point.x * 0.5
}
```

만약, scope함수를 이용하지 않고 자바 코드로 구현을 하게 되면

아래와 같은 코드가 탄생할 것이다.

```java
Point point = new Point()
int width = point.x * 0.5
```



let함수는 또한 복잡한 계산을 할 때, 블록 내에 임시변수가 필요할 때 사용하기도 한다.

```kotlin
val result = run {
    val defaultPadding = TypedValue.applyDimension(...)
    val extraPadding = TypedValue.applyDimension(...)
  
    defaultPadding + extraPadding
}
```



## with

`with()` 함수는 인자로 받는 객체를 이어지는 함수 블록의 리시버로 전달한다

즉, 함수에서 사용할 객체를 매개변수를 통해 전달받는 형태이다.

```kotlin
val person = Person()
with(person){
    work()
    sleep()
    println(age)
}
```



with함수를 사용하지 않은 자바코드는 아래와 같다.

```java
Person person = new Person();
person.work();
person.sleep();
System.out.println(person.age);
```



## apply

`apply()` 함수는 이 함수를 호출한 객체를 이어지는 함수 블록의 리시버로 전달한다.

객체를 반환하기 때문에 주로 객체 초기화에 사용한다.

```kotlin
val person = Person().apply {
    firstName = "Eun Ho"
    lastName = "Hong"
}
```



scope함수를 사용하지 않은 자바 코드는 다음과 같다.

```java
Person person = new Person();
person.firstname = "Eun Ho";
person.lastname = "Hong";
```



## also

also() 함수는 기존 객체를 변경하지 않기 때문에 

주로 데이터의 유효성을 확인하거나 디버깅을 할 때 사용한다.

```kotlin
Random.nextInt(100).also{
    print("getRandomInt() generated value $it")
}
```



자바 코드는 다음과 같다.

```java
int value = Random().nextInt(100);
System.out.print(value);
```