---
title: Kotlin Generic 제네릭
author: yoonmin
date: 2023-11-01 12:00:00 +0900
categories: [CS, 프로그래밍 언어, Kotlin]
tags: [Kotlin, Java, Generic, 제네릭]
render_with_liquid: false
---

## Kotlin도 Java의 제네릭 기능을 제공한다.

> **"Classes in Kotlin can have type parameters, just like in Java:"** - Kotlin 공식 문서

코틀린의 클래스는 자바와 마찬가지로 타입 파라미터를 가질 수 있습니다. 제네릭을 사용함으로써 <span style="color: #30aaa0">**컴파일 타임에 타입 안전성을 관리할 수 있고 여러 타입에 대응하여 코드를 재사용하는 이점**</span>을 얻을 수 있습니다.

코틀린 역시  `<>` 기호를 이용한 제네릭 기능을 제공합니다. 다음 `Box` 클래스는 제네릭을 사용하여 만들어졌기 때문에  `Int` , `String`, `Float` 등의 다양한 타입을 가질 수 있습니다. 

```kotlin
class Box<T>(t: T) {
    var value = t
}
```

```kotlin
val boxInt: Box<Int> = Box<Int>(1)
val boxString: Box<String> = Box<String>("empty")
val boxFloat: Box<Float> = Box<Float>(2.0F)
```

위의 예시를 통해서 제네릭(Generic)은 단순히 하나의 타입으로 고정하는 것이 아닌 다양한 타입을 수용할 수 있는 하나의 일반화(Generalization)된 타입 파라미터라고 할 수 있습니다. 

위의 예제에서는 타입 파라미터 정의를 `T` 로 했는데 이는 제네릭을 사용할 때 무조건 `T` 를 사용해야 한다는 뜻이 아닙니다. 제네릭에서 사용되는 기호는 정해진 것이 아닌 사용자가 직접 정의합니다. 그래서 상황에 따라 네이밍을 하면 됩니다.

| 유형  |  의미   |
| :---: | :-----: |
| `<T>` |  Type   |
| `<E>` | Element |
| `<K>` |   Key   |
| `<V>` |  Value  |
| `<N>` | Number  |

​		

## 타입 안전성의 증가

`Animal` 추상 클래스가 있고, 해당 추상 클래스의 구현체인 `Tiger`,  `Lion` 이렇게 두 개의 클래스가 존재하는 상황이라고 가정하겠습니다. 동물의 정보와 관련된 클래스를 정의했으므로 이제 동물들을 관리하는 동물원, `Zoo` 클래스를 정의합니다.

```kotlin
abstract class Animal
abstract class Mammalia: Animal()
class Tiger: Mammalia()
class Lion: Mammalia()
```

```kotlin
class Zoo {
    private val animals = mutableListOf<Animal>()

    fun getLast(): Animal {
        return animals.last()
    }

    fun getFirst(): Animal {
        return animals.first()
    }

    fun add(animal: Animal) {
        animals.add(animal)
    }
}
```

메인 함수에서 호랑이를 동물원에 추가하고 다시 꺼내오는 작업을 했을 때 `Tiger`로 받으려면 다운캐스팅이기 때문에 변환 타입을 명시해야 합니다.

```kotlin
val zoo = Zoo()
zoo.add(Tiger())
val tiger: Tiger = zoo.getLast() as Tiger
```

그런데 동물 리스트에서 데이터를 꺼내올 때 해당 데이터가 무조건 `Tiger` 라는 보장이 없습니다. 데이터를 삽입할 때 `Tiger` 가 아닌 `Lion` 을 넣을수도 있기 때문입니다. 그래서 `as?` 를 이용하거나 엘비스 연산자 `?:` 를 이용해서 예외에 대응하는 방법이 있습니다만...

### 제네릭 클래스로 수정하자

제네릭을 이용한다면 타입 미스매치 발생을 방지하고 코드 가독성도 좋게 만들 수 있습니다. 동물들을 관리하는 `Zoo` 클래스에 타입 파라미터 `T` 를 정의하여 수정하면 클래스 생성부터  `Tiger` 타입 지정이 가능합니다.

```kotlin
class Zoo<T> {
    private val animals = mutableListOf<T>()

    fun getLast(): T {
        return animals.last()
    }

    fun getFirst(): T {
        return animals.first()
    }

    fun add(animal: T) {
        animals.add(animal)
    }
}
```

```kotlin
val zoo = Zoo<Tiger>()
zoo.add(Tiger())
val tiger: Tiger = zoo.getLast()
```

이렇게 제네릭 클래스로 만들면 컴파일 타임에 타입 오류를 찾아낼 수 있고 `Tiger` 를 관리하는 동물원 `Zoo` 로 관리가 가능합니다.
