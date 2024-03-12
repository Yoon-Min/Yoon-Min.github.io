---
title: Kotlin - 고차 함수와 inline (1/2)
author: yoonmin
date: 2024-03-11 00:00:00 +0900
categories: [CS, 프로그래밍 언어]
tags: [Kotlin, Android, 함수형 프로그래밍, 람다]
render_with_liquid: true
---

![christian-lue-b4kKyX0BQvc-unsplash 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/3278a245-59a1-4b63-abd4-f70758bb333a)

## Intro

> *"Kotlin functions are first-class, which means they can be stored in variables and data structures, and can be passed as arguments to and returned from other higher-order functions. You can perform any operations on functions that are possible for other non-function values."*
>
> **Kotlin docs**

Kotlin 함수는 `first-class` - 일급 함수로, 변수와 데이터 구조에 저장할 수 있으며 다른 상위 함수에 인자로 전달하거나 반환값으로 활용할 수 있습니다. 즉, 함수에 대한 다양한 연산을 수행할 수 있다는 뜻입니다.

이는 자연스럽게 코틀린 함수는 일급 함수뿐만 아니라 함수를 파라미터로 활용하거나 반환할 수 있는 고차 함수(Higher-order functions)에도 해당된다는 것을 알 수 있습니다. 

```kotlin
/* 함수를 파라미터로 사용  */
fun <T, R> Collection<T>.fold(
    initial: R,
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}

/* 함수를 변수로 사용  */
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }

/* 함수를 반환값으로 사용  */
fun adderInitializer(x: Int): (Int) -> Int {
    return { it + x }
}
val adder = adderInitializer(10)
println(adder(2))
```



일급 함수와 고차 함수에 해당된다는 이점 덕분에 코틀린 함수는 코드 재사용성과 가독성을 높이고 좀 더 추상적인 구현을 지원합니다. 그렇다면 코틀린 함수는 어떻게 고차 함수이고 어떤 활용 방법이 있을까요? 또한 거기서 나오는 장점과 단점은 무엇이며 이에 대한 보완책은 무엇일까요?

이번 포스팅은 코틀린의 고차 함수와 여기에서 발생되는 문제점을 막기 위한 `inline` 키워드에 대해 알아보고자 합니다. 이를 알기 위해서 먼저 코틀린에서 지원하는 유연한 함수 처리 방법에 대해 알아야 할 필요가 있습니다.

​		

## 코틀린 함수 타입

### Common type

코틀린은 함수를 변수, 인자, 반환값으로 활용할 수 있기 때문에 정수는 `Int` , 문자열은 `String` 인 것처럼 함수도 코드로 명시할 타입이 필요합니다. 코틀린이 지정한 모든 함수의 공통적인 타입 표기는 다음과 같습니다.

```kotlin
(A, B) -> C
```

소괄호 내에 파라미터 목록을 표기하고 파라미터가 없다면 빈 소괄호로 표기합니다. 그리고 `->` 을 추가하고 소괄호 내 파라미터 목록을 활용하여 `C` 타입의 값을 반환합니다. 

이 방식으로 다음 예제와 같이 함수에 대한 타입을 표기합니다.

```kotlin
// ex1
(Int) -> String
// ex2
val onClick: () -> Unit = { /* ... */ }
```

​		

### Option1. Receiver type

수신자가 있는 함수 타입으로 정의를 하는 방법도 있습니다. 수신자가 있는 함수 타입 정의는 다음과 같이 수신자와 `.` 을 앞에 표기합니다. 이렇게 하면 수신자 객체 `A` 에서 매개변수 `B` 를 호출하고 `C` 를 반환하는 동작을 합니다.

```kotlin
A.(B) -> C
```

이 방식은 수신자가 있는 함수 리터럴로 인스턴스화할 수 있습니다. 즉, [람다 표현식](#lambda-expression)이나 [익명 함수](#anonymous-function)와 같은 함수 리터럴 본문 내에서 수신자 객체를 암시적 `this` 로 접근해서 수신자 객체나 멤버에 접근할 수 있습니다.

```kotlin
/* Lambda expression */
val add: Int.(Int) -> Int = { this.plus(it) }
println(2.add(10))
println(add(2,10))

/* Anonymous function */
val add = fun Int.(other: Int): Int = this + other
println(2.add(3))
println(add(2,3))
```

코드에서 어디서 본 듯한 익숙함을 느끼지 않았나요? 이 동작은 함수 본문 내에서 수신자 객체의 멤버에 접근할 수 있는 **확장 함수의 동작과 유사**한 것을 알 수 있습니다.

​		

### Option2. Suspend type

중단 함수에 대한 타입을 정의하는 방법도 있습니다. 함수 타입 맨 앞에 `suspend` 키워드를 추가합니다.

```kotlin
suspend A.(B) -> C
```

​		

## 함수 리터럴 기반의 람다 표현식과 익명 함수

### Function literals

함수 리터럴은 **선언되지 않고 표현식으로 즉시 전달되는 함수**입니다. 코틀린이 제공하는 **람다 표현식**과 **익명 함수**가 함수 리터럴에 해당합니다.

### Lambda expression

```kotlin
/* 람다 표현식 전체 구문 */
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
/* 람다로 함수 타입 유추가 가능하므로 함수 타입 생략 가능 */
val sum = { x: Int, y: Int -> x + y }
/* 함수 타입으로 타입 유추가 가능하므로 파라미터 이름만 만들어서 사용 */
val sum: (Int, Int) -> Int = { x, y -> x + y }
/* 함수 타입에서 파라미터명 정의해서 사용 */
val sum: (x: Int, y: Int) -> Int = { x, y -> x + y }
/* 파라미터가 없는 경우는 생략 가능 */
val temp: () -> Unit = { println("temp") }
```

- 람다 표현식은 항상 중괄호로 둘러싸여 있습니다.
- 매개변수 선언은 중괄호 안에서, 매개변수 타입은 작성한 문법에 따라 생략이 가능합니다.
- 함수 본문은 중괄호 내 매개변수 뒤에 위치한  `->` 다음에 정의합니다.
- 람다의 유추된 반환 유형이 `Unit`이 아닌 경우, 함수 본문 내 마지막 표현식이 반환값으로 처리됩니다.

​		

### Anonymous function

```kotlin
fun(x: Int, y: Int): Int = x + y
fun(x: Int, y: Int): Int { return x + y }
```

람다 표현식은 파라미터와 반환식을 통해 반환값 타입 유추가 가능하기 때문에 반환 타입을 명시하지 않습니다. 하지만 반환 타입 명시가 필요한 경우 람다와 같은 함수 리터럴인 익명 함수를 사용할 수 있습니다.

익명 함수는 파라미터와 반환 타입은 일반 함수와 동일한 방식으로 지정되지만, 문맥에서 유추할 수 있는 경우 파라미터 타입은 생략할 수 있습니다.

```kotlin
/* forEach */ 
@kotlin.internal.HidesMembers
public inline fun <T> Iterable<T>.forEach(action: (T) -> Unit): Unit { /* T -> Unit 확인 */
    for (element in this) action(element)
}
/* (Int) -> Unit 형태의 함수 리터럴이 필요 */
listOf(1,2,3,4).forEach(fun(item) {println(item)})
```

```kotlin
/* filter */
public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> { /* T -> Boolean 확인 */
    return filterTo(ArrayList<T>(), predicate)
}
/* (Int) -> Boolean 형태의 함수 리터럴이 필요 */
listOf(1,2,3,4).filter(fun(item) = item > 0)
```

또한 익명 함수는 표현식을 사용했을 때 반환 타입 추론이 가능하지만 함수 블록을 사용하는 경우에는 반환 타입을 명시해야 합니다. 단, 반환형이  `Unit` 인 경우는 블록을 사용해도 생략 가능합니다.

```kotlin
val l = listOf(1,2,3,4)
l.forEach(fun(item) { println(item) }) /* Unit 반환형은 함수 블록에서도 생략 가능 */
l.filter(fun(item): Boolean { return item > 2 }) /* 함수 블록 사용시 반환형 명시 */
l.filter(fun(item) = item > 2) /* 표현식 사용시 반환형 추론 가능 */
```

​		

## 람다 함수와 익명 함수의 차이점

### Non-local return

코틀린에서 `fun` 키워드를 사용하는 이름이 있는 명명된 함수와 익명 함수는 레이블이 없는 반환이 가능합니다. 이 덕분에 흔히 사용하는 `return` 을 반환될 `fun` 을 명시하는 레이블 없이 사용할 수 있습니다.

```kotlin
fun(x: Int, y: Int): Int {
  return x + y 
}
```

반면에  람다 표현식은 `fun` 키워드 없이 사용되기 때문에 레이블 없는 축약된 반환이 불가능합니다. 그래서 기본적으로 람다 표현식 내부에서 레이블 없는 반환은 금지되어 있습니다.

```kotlin
fun foo() {
    ordinaryFunction {
        return // ERROR: cannot make `foo` return here
    }
}
```

람다 표현식에서 람다 자체를 반환하고 싶다면 다음과 같이 람다가 실행되는 함수 영역 대한 레이블을 추가해야 합니다.

```kotlin
fun func(lambda: () -> Unit) {
    lambda()
}

fun main() {
  func {
      return@func
  }
}
```

그런데 람다 표현식이 `inlin` 된 경우라면 람다 표현식을 둘러싼 함수 영역의 반환이 가능합니다. 





