---
title: Kotlin - 고차 함수와 inline
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

일급 함수와 고차 함수에 해당된다는 이점 덕분에 코틀린 함수는 코드 재사용성과 가독성을 높이고 좀 더 추상적인 구현을 지원합니다. 그렇다면 코틀린 함수는 어떻게 고차 함수이고 어떤 활용 방법이 있을까요? 또한 거기서 나오는 장점과 단점은 무엇이며 이에 대한 보완책은 무엇일까요?

이번 포스팅은 코틀린의 고차 함수와 여기에서 발생되는 문제점을 막기 위한 `inline` 키워드에 대해 알아보고자 합니다. 이를 알기 위해서 먼저 코틀린에서 지원하는 유연한 함수 처리 방법에 대해 알아야 할 필요가 있습니다.

​		

## Function types



## Lambda & Anonymous

### Function literals

함수 리터럴은 **선언되지 않고 표현식으로 즉시 전달되는 함수**입니다. 코틀린이 제공하는 **람다 표현식**과 **익명 함수**가 함수 리터럴에 해당합니다. 유연한 함수 활용의 시작은 여기서 시작됩니다.

### Lambda expression

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```

- 람다 표현식은 항상 중괄호로 둘러싸여 있습니다.
- 매개변수 선언은 중괄호 안에서, 매개변수 타입은 작성한 문법에 따라 생략이 가능합니다.
- 함수 본문은 중괄호 내 매개변수 뒤에 위치한  `->` 다음에 정의합니다.
- 람다의 유추된 반환 유형이 `Unit`이 아닌 경우, 함수 본문 내 마지막 표현식이 반환값으로 처리됩니다.



