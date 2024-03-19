---
title: Kotlin - 고차 함수와 inline (2/2)
author: yoonmin
date: 2024-03-19 00:00:00 +0900
categories: [CS, 프로그래밍 언어]
tags: [Kotlin, Android, 함수형 프로그래밍, 람다, 익명 함수, inline]
render_with_liquid: true
---

![slava-auchynnikau-N1u1b-YKXN0-unsplash 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/0f11549b-1681-4805-993f-744a6d804670)

## Intro

저번 고차 함수와 인라인 1편에서 코틀린 함수가 왜 고차 함수인지 함수 타입과 함수 리터럴을 통해 알아봤습니다. 코틀린은 함수에 대한 타입을 제공하고 이를 통해서 함수를 인자로 활용하거나 반환값으로 활용할 수 있습니다. 특히 인자로 활용하는 과정에서 함수 리터럴을 활용하면 가독성이 뛰어난 코드 구현이 가능합니다. 

이런 뛰어난 장점을 가진 고차 함수도 단점은 존재합니다. 이 때문에 코틀린은 고차 함수의 단점을 보완하기 위해 `inline` 이라는 키워드를 제공하여 고차 함수의 단점을 보완하는 데 도움을 제공합니다.

따라서 코틀린이 제공하는 함수형 프로그래밍을 제대로 활용하려면 고차 함수의 장점과 단점이 무엇인지 파악하고 사용해야 합니다. 이번 글에서는 고차 함수가 가지는 장점과 단점이 무엇이며 단점을 해결하기 위한 `inline` 키워드에 대해 알아보겠습니다.

​		

## 고차 함수를 활용해야 하는 이유

#### **1. 코드 재사용성 및 모듈성**

특정 기능을 개별 함수로 분리하여 전역의 여러 부분에서 재사용이 가능합니다. 애플리케이션 전역에서 재사용이 가능하다는 점 덕분에 베이스 코드의 유지 관리가 용이하고 중복성을 줄일 수 있습니다.

전역의 여러 부분에서 사용하는 확장 함수를 보면 이해가 쉽습니다. 우리가 자주 사용하는 고차 함수 특징을 활용한 `forEach` , `map` 등의 확장 함수 베이스 코드는 `Collections.kt` 파일에서 관리합니다.

```kotlin
/* _Collections.kt */

public inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    return mapTo(ArrayList<R>(collectionSizeOrDefault(10)), transform)
}

public inline fun <T> Iterable<T>.forEach(action: (T) -> Unit): Unit {
    for (element in this) action(element)
}
```

`forEach` , `map` 함수는 둘 다 모든 원소에 대해서 파라미터로 받는 함수 내 정의된 동작을 적용합니다. 따라서 `forEach` , `map` 을 호출하는 지점에서 우리는 각 원소에 적용할 동작 코드만 정의하면 됩니다.

```kotlin
listOf(1,2,3,4,5,6).map { it+1 } // ex) 각 원소에 1을 더하기 위한 map 사용
listOf(1,2,3,4,5,6).forEach { println(it) } // ex) 각 원소를 출력하기 위한 forEach 사용
```

이런 식으로 공통 코드(베이스 코드)를 정의해놓고 필요한 동작만 함수 파라미터로 우리가 작성해서 전달하면 베이스 코드와 필요한 동작 코드가 분리되어 유지 관리가 편리하고 재사용하기 좋습니다.

​		

#### **2. 추상화와 캡슐화를 통한 분리**

함수의 구현 세부사항을 숨기고 필요한 기능만 노출하는 추상화를 가능하게 합니다. 이러한 캡슐화는 보안을 강화하고 복잡한 기능을 단순화합니다. 그리고 관심사를 분리하여 쉬운 유지보수가 가능합니다.

`forEach` , `map` 을 사용하기 위해 필요한 것은 오직 각 원소에 적용할 동작을 정의하는 것입니다. `_Collections.kt` 파일에 구현 세부사항을 정의하고 사용자 정의가 필요한 부분만 함수 파라미터로 받기 때문에 `forEach` , `map` 을 사용하는 우리는 필요한 동작만 정의해서 전달하면 됩니다.

```kotlin
/* _Collections.kt

public inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    return mapTo(ArrayList<R>(collectionSizeOrDefault(10)), transform)
}

*/

fun main() {
    listOf(1,2,3,4,5,6).map { it+1 }
}
```

​		

#### **3. 가독성 향상**

고차 함수의 람다 표현식을 통해서 보다 표현력이 풍부하고 간결한 코드를 작성할 수 있습니다. 특히 코틀린이 주장하는 코틀린의 강력한 장점중 하나인 함수형 프로그래밍을 위해서 고차 함수의 좋은 가독성은 필수입니다.

코틀린 공식 홈페이지를 보면 코틀린의 장점 다섯 가지를 확인할 수 있는데 그 중 하나가 함수형 프로그래밍 구현입니다. 고차 함수를 활용하면 다음과 같은 간결하고 명료한 코드 작성이 가능합니다.

```kotlin
fun main() {
    // Who sent the most messages?
    val frequentSender = messages
        .groupBy(Message::sender)
        .maxByOrNull { (_, messages) -> messages.size }
        ?.key                                                 // Get their names
    println(frequentSender) // [Ma]

    // Who are the senders?
    val senders = messages
        .asSequence()                                         // Make operations lazy (for a long call chain)
        .filter { it.body.isNotBlank() && !it.isRead }        // Use lambdas...
        .map(Message::sender)                                 // ...or member references
        .distinct()
        .sorted()
        .toList()                                             // Convert sequence back to a list to get a result
    println(senders) // [Adam, Ma]
}

data class Message(                                           // Create a data class
    val sender: String,
    val body: String,
    val isRead: Boolean = false,                              // Provide a default value for the argument
)

val messages = listOf(                                        // Create a list
    Message("Ma", "Hey! Where are you?"),
    Message("Adam", "Everything going according to plan today?"),
    Message("Ma", "Please reply. I've lost you!"),
)
```

실제로 위와 같은 고차 함수를 활용한 메서드 체이닝 기법을 사용하면 무슨 동작을 할 것인지에 대해서만 코드를 작성할 수 있기 때문에 코드 가독성 개선에 좋습니다.

​		

## 고차 함수 활용시 주의사항

> *"Using higher-order functions imposes certain runtime penalties: each function is an object, and it captures a closure. A closure is a scope of variables that can be accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead."*
>
> **Kotlin docs**

공식 문서에서 말하는 고차 함수 사용시 발생할 수 있는 부작용입니다. 고차 함수를 사용하면 **특정 런타임 페널티**가 부과되는데,  고차 함수가 객체로 전환되며 **클로저**를 이용합니다. 클로저는 함수 본문에서 접근할 수 있는 변수의 범위입니다. **메모리 할당**(함수 객체와 클래스 모두)과 가상 호출은 런타임 오버헤드를 유발합니다.

​		

### **코틀린 클로저?**

> *"A lambda expression or anonymous function (as well as a local function and an object expression can access its closure, which includes the variables declared in the outer scope. The variables captured in the closure can be modified in the lambda"*
>
> **Kotlin docs**
>
> *"A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function's scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time."*
>
> **MDN - JavaScript**

코틀린이 설명하는 클로저는 람다 표현식 또는 익명 함수(로컬 함수 및 객체 표현식도 마찬가지)를 사용할 때 외부 범위에서 선언된 변수에 액세스할 수 있는 것을 말합니다.

자바스크립트에서 설명하는 클로저는 함수를 주변 상태(어휘 환경)에 대한 참조와 함께 묶음 결합한 것입니다. 다시 말해, 클로저를 사용하면 내부 함수에서 외부 함수의 범위에 접근할 수 있습니다.

즉, 종합하면 클로저는 외부 범위에 접근할 수 있는 기능이라고 할 수 있습니다. 한 번 생각해 봅시다. 코틀린 반복문인 `forEach` 을 사용하기 위해  함수 리터럴로 동작 코드를 정의할 때 자연스럽게 외부 변수를 가져다 사용한 경험이 있지 않나요? 다음 예시처럼요.

```kotlin
var sum = 0
listOf(1,2,3,4).forEach {
    sum += it
}
println(sum)
```

알고보면 우리는 코틀린을 사용할 때 클로저라는 기능을 지금까지 자연스럽게 사용해 왔습니다. 그렇다면 우린 다음과 같은 생각을 할 수 있습니다.

{: .prompt-tip}

> "그럼 코틀린과 밀접한 자바도 외부 변수에 접근하고 수정도 가능하겠네?"

​		

### **자바 클로저는 코틀린과 같을까?**

> *"In Android Java world, you know that when you declare a lambda in a function, you can access the parameters of that function as well as the local variables declared before the lambda. These variables are said to be captured by the lambda, but the variables must be final or effectively final to be captured by lambda. In Kotlin, this constraint has been removed."*
>
> *"안드로이드 자바 세계에서는 함수에서 람다를 선언하면 해당 함수의 매개변수와 람다 앞에 선언된 로컬 변수에 액세스할 수 있다는 것을 알고 계실 겁니다. 이러한 변수는 람다에 의해 캡처된다고 하지만, 람다가 캡처하려면 변수가 `final`이거나 `effectively final`이어야 합니다. Kotlin에서는 이 제약 조건이 제거되었습니다."*
>
> [**How Kotlin lambda capture variable**](https://medium.com/@yangweigbh/how-kotlin-lambda-capture-variable-ef90e11e531d)

자바 클로저는 외부 변수에 접근하려면 외부 변수가 `final` 혹은 `effectively final` 이어야 하는 조건이 있습니다. 



​		

## 참조

[**Android Interview Questions: 7 - What are the High-order functions in Kotlin?**](https://medium.com/@dawinderapps/android-interview-questions-7-what-are-the-high-order-functions-in-kotlin-695f7c8c3dee)

[**How Kotlin lambda capture variable**](https://medium.com/@yangweigbh/how-kotlin-lambda-capture-variable-ef90e11e531d)

[**Higher-order functions and lambdas**](https://kotlinlang.org/docs/lambdas.html)

[**Inline functions**](https://kotlinlang.org/docs/inline-functions.html)

[**First-Class and Higher-Order Functions**](https://medium.com/@rabailzaheer/first-class-and-higher-order-functions-86d14e40c688)

[**클로저(Closure): Java와 Kotlin 비교**](https://jaeyeong951.medium.com/java-%ED%81%B4%EB%A1%9C%EC%A0%80-vs-kotlin-%ED%81%B4%EB%A1%9C%EC%A0%80-c6c12da97f94)

