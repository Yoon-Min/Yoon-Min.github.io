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

## 고차 함수를 사용했을 때 얻는 이점

#### **1. 코드 재사용성 및 모듈성**

특정 기능을 개별 함수로 분리하여 전역의 여러 부분에서 재사용이 가능합니다. 애플리케이션 전역에서 재사용이 가능하다는 점 덕분에 베이스 코드의 유지 관리가 용이하고 중복성을 줄일 수 있습니다.

#### **2. 추상화와 캡슐화를 통한 분리**

함수의 구현 세부 사항을 숨기고 필요한 기능만 노출하는 추상화를 가능하게 합니다. 이러한 캡슐화는 보안을 강화하고 복잡한 기능을 단순화합니다. 그리고 관심사를 분리하여 쉬운 유지보수가 가능합니다.

#### **3. 가독성 향상**

고차 함수의 람다 표현식을 통해서 보다 표현력이 풍부하고 간결한 코드를 작성할 수 있습니다. 특히 코틀린이 주장하는 코틀린의 강력한 장점중 하나인 함수형 프로그래밍을 위해서 고차 함수의 좋은 가독성은 필수입니다.

​		

## 런타임 오버헤드

> *"Using higher-order functions imposes certain runtime penalties: each function is an object, and it captures a closure. A closure is a scope of variables that can be accessed in the body of the function. Memory allocations (both for function objects and classes) and virtual calls introduce runtime overhead."*
>
> **Kotlin docs**

공식 문서에서 말하는 고차 함수 사용시 발생할 수 있는 부작용입니다. 고차 함수를 사용하면 특정 런타임 페널티가 부과되는데,  고차 함수가 객체로 전환되며 클로저를 이용합니다. 클로저는 함수 본문에서 액세스할 수 있는 변수의 범위입니다. 메모리 할당(함수 객체와 클래스 모두)과 가상 호출은 런타임 오버헤드를 유발합니다.

​		

## 참조

[**Android Interview Questions: 7 - What are the High-order functions in Kotlin?**](https://medium.com/@dawinderapps/android-interview-questions-7-what-are-the-high-order-functions-in-kotlin-695f7c8c3dee)

[**How Kotlin lambda capture variable**](https://medium.com/@yangweigbh/how-kotlin-lambda-capture-variable-ef90e11e531d)

[**Higher-order functions and lambdas**](https://kotlinlang.org/docs/lambdas.html)

[**Inline functions**](https://kotlinlang.org/docs/inline-functions.html)

[**First-Class and Higher-Order Functions**](https://medium.com/@rabailzaheer/first-class-and-higher-order-functions-86d14e40c688)