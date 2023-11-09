---
title: Kotlin Generic <1> 제네릭 클래스와 함수, 그리고 변성
author: yoonmin
date: 2023-11-07 12:00:00 +0900
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

## 타입 안전성

### 제네릭을 사용하지 않은 클래스의 경우는?

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

​		

### 제네릭 클래스로 수정하자

제네릭을 이용한다면 타입 미스매치 발생을 방지하고 코드 가독성도 좋게 만들 수 있습니다. 동물들을 관리하는 `Zoo` 클래스에 타입 파라미터 `T` 를 정의하여 수정하면 클래스 생성부터  `Tiger` 타입 지정이 가능합니다.

이렇게 제네릭 클래스로 만들면 컴파일 타임에 타입 오류를 찾아낼 수 있고 `Tiger` 를 관리하는 동물원 `Zoo` 로 관리가 가능합니다. 그래서 `as` 를 사용해서 캐스팅할 타입을 명시하지 않아도 깔끔하고 안전하게 데이터를 가져올 수 있습니다.

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

​		

## 변성

### 변성(Variance)

<span style="color: #30aaa0">**변성(Variance)은 제네릭 클래스끼리의 상속 관계를 나타내는 개념입니다.**</span> 변성은 크게 공변, 반공변, 무공변 이렇게 세 가지로 나눌 수 있는데 해당 세 가지의 정의는 다음과 같습니다.

|            유형            |                             의미                             |
| :------------------------: | :----------------------------------------------------------: |
|    **공변(Covariance)**    | `Tiger` 가 `Mammalia` 의 서브타입이라면 `Zoo<Tiger>` 는 `Zoo<Mammalia>` 의 서브타입이다. |
| **반공변(Contravariance)** | `Tiger` 가 `Mammalia` 의 서브타입이라면 `Zoo<Mammalia>` 는 `Zoo<Tiger>` 의 서브타입이다. |
|   **무공변(Invariance)**   |               공변도 아니고 반공변도 아닌 상태               |

​		

### 제네릭 클래스는 기본적으로 무공변

동물원 예시 코드를 조금 수정하겠습니다. 다음과 같이 `Tiger` `Lion` 의 상위타입인 `Mammalia` 를 타입 파라미터로 가지는 제네릭 클래스를 생성해서 거기에 `Tiger` `Lion` 클래스를 `add` 해보겠습니다.

```kotlin
val zooWithMammalia = Zoo<Mammalia>()
zooWithMammalia.add(Tiger())
zooWithMammalia.add(Lion())
```

 `Mammalia` 를 타입 파라미터로 가지는 `Zoo<Mammalia` 의 `add` 메서드는 파라미터가 `animal: Mammalia` 로 설정이 되고 여기에 인자로 `Tiger` , `Lion` 이 온다면 서로 상속 관계이기 때문에 컴파일 및 실행에 문제가 없습니다.

```kotlin
fun add(animal: T) { // animal: Mammalia
    animals.add(animal)
}
```

이렇게 단일 동물을 다른 동물원에 추가하는 데 문제가 발생하지 않지만 동물원 자체를 다른 동물원에 합치는 경우는 어떨까요? 다음과 같이 동물원 자체를 합치는 메서드를 추가로 작성해보겠습니다.

```kotlin
fun mergeOtherZoo(zoo: Zoo<T>) {
    this.animals.addAll(zoo.animals)
}
```

그리고 `Zoo<Mammalia>` 에 `Zoo<Tiger>` 를 병합시켜보겠습니다. 결과는 단일 동물을 추가할 때와 달리 타입 미스매치가 발생합니다.

```kotlin
val zooWithTigers = Zoo<Tiger>()
val zooWithMammalia = Zoo<Mammalia>()
zooWithTigers.add(Tiger())
zooWithMammalia.mergeOtherZoo(zooWithTigers) // Type mismatch 발생
```

분명 `Zoo<Mammalia>` 에 `Tiger()` 를 추가하는 것은 문제가 없었는데 `Zoo<Tiger>` 를 추가하는 것에는 문제가 발생합니다. 이는 제네릭 클래스는 기본적으로 무공변 상태이기 때문에 아무런 관계가 없는  `Zoo<Mammalia>` 와 `Zoo<Tiger>` 는 병합 시도시 오류가 발생할 수밖에 없는 겁니다.

​		

### 공변으로 전환

이를 해결하기 위해서는 무공변인 상태를 공변으로 전환해야 합니다. 전환하는 방법은 간단합니다. 병합하는 메서드의 타입 파라미터 왼쪽에 `out` 이라는 키워드를 추가하면 됩니다. 이 결과로 본인의 하위타입을 수용할 수 있고 타입 미스매치 오류가 사라지면서 정상 실행이 됩니다.

```kotlin
fun mergeOtherZoo(zoo: Zoo<out T>) {
    this.animals.addAll(zoo.animals)
}
```

​		

### 공변으로 전환시 주의할 점

무공변 상태에서 공변으로 전환을 하면 제네릭 클래스 간 상속 관계가 생겨서 인자로 넘길 수 있게 되지만 주의할 점이 있습니다. 만약에 `mergeOtherZoo` 메서드 내부 코드가 반대로 인자로 받은 동물원에 기존 동물원을 합치는 경우면 어떻게 될까요?

타입 미스매치가 발생하게 됩니다. `out` 을 사용한 시점에서 인자로 받는 `zoo` 는 자신을 인자로 필요로 한 클래스의 서브 타입일 수 있기 때문에 하위타입에 상위타입의 데이터를 넣는 동작은 타입 안전성을 해칠 수 있습니다.

```kotlin
fun mergeOtherZoo(zoo: Zoo<out T>) {
    zoo.animals.addAll(this.animals)		
}
```

{:.prompt-tip}

> `out` 키워드를 이용해서 공변 상태로 전환할 때는 파라미터인  `zoo`  가 본인의 데이터를 넘겨주는 쪽으로 동작을 처리하면 됩니다. 

​		

### 반공변으로 전환

`out` 을 이용해서 공변 상태로 만들었을 때는 `zoo` 가 `this` 의 하위 타입이 되는 것이 가능하므로 `zoo` 에 `this` 데이터를 가져가는 동작은 타입 안전성을 해칠 수 있습니다. 이러한 동작을 처리하려면 공변이 아닌 반공변으로 설정하는 것이 좋습니다.

반공변은 말 그대로 공변의 반대입니다. 여기서는 서브타입이었던 클래스가 반대로 상위타입이 됩니다. 그래서 공변이었을 때 불가능했던 파라미터 `zoo` 에 `this` 의 데이터를 가져가는 동작이 반공변에선 가능합니다.

```kotlin
fun mergeOtherZoo(zoo: Zoo<in T>) {
    zoo.animals.addAll(this.animals)
}
```

{:.prompt-tip}

> `in` 키워드를 이용해서 반공변 상태로 전환할 때는 파라미터인 `zoo` 가 `this` 의 데이터를 소비하는 쪽으로 동작을 처리하면 됩니다.

​		

### 변성 선언 위치

변성 선언은 파라미터에서 설정하는 것 이외에도 클래스 전체를 설정하는 것도 가능합니다. 만약에 클래스 전체를 공변하게 만들고 싶거나 반공변하게 만들고 싶다면 다음과 같이 클래스 헤더에 설정하면 됩니다. 이렇게 하면 예시로 사용했던 `mergeOtherZoo` 메서드 파라미터로 `Zoo` 클래스를 받을 필요가 없어집니다.

``` kotlin
class Zoo<out T> {}
class Zoo<in T> {}
```

```kotlin
// 클래스 자체를 공변 설정하지 않았을 때
fun mergeOtherZoo(zoo: Zoo<T>) {
    this.animals.addAll(zoo.animals)
}
// 파라미터로 클래스가 아닌 리스트로 수정
fun mergeOtherZoo(zoo: List<T>) {
    this.animals.addAll(zoo)
}
```

#### 공변 설정

만약 클래스를 `out` (공변) 설정했다면 다음 두 메서드에서 오류가 발생합니다. 

```kotlin
fun add(animal: T) {
    animals.add(animal)
}

fun mergeOtherZoo(zoo: List<T>) {
    this.animals.addAll(zoo)
}

// Type parameter T is declared as 'out' but occurs in 'in' position in type T
```

다음과 같이 메서드 파라미터에 공변을 설정했을 때를 보면 `zoo` 가 데이터를 넘겨주는 역할을 해야 한다고 언급했습니다. 그래서 클래스 자체를 공변으로 설정했다면 내부 메서드도 전부 데이터를 넘기는(`getter`) 동작으로 설정해야 합니다. 그래서 `getLast()` `getFirst()` 는 별다른 오류가 없는 겁니다.

```kotlin
fun mergeOtherZoo(zoo: Zoo<out T>) {
    zoo.animals.addAll(this.animals)		
}
```

#### 반공변 설정

이번엔 클래스를 `in` (반공변) 설정했다면 다음 두 메서드에서 오류가 발생합니다.

```kotlin
fun getLast(): T {
    return animals.last()
}

fun getFirst(): T {
    return animals.first()
}
// Type parameter T is declared as 'in' but occurs in 'out' position in type T
```

이번엔 공변으로 설정했을 때와 달리 `getter` 역할을 수행하는 메서드에서 오류가 발생했습니다. 메서드 파라미터에 `in` 반공변을 설정했을 때를 보면 `zoo` 가 데이터를 넘기는 것이 아닌 오히려 데이터를 받는 입장인 것을 알 수 있습니다. 그래서 이때는 클래스 내 메서드는 데이터를 받는 쪽으로 처리하면 됩니다.

```kotlin
fun mergeOtherZoo(zoo: Zoo<in T>) {
    zoo.animals.addAll(this.animals)
}
```

​		

### 강제 변성

위의 예시를 보면 알 수 있듯이, 제네릭은 타입 안전성을 굉장히 신경을 씁니다. 그래서 `in` 혹은 `out` 으로 했을 때 안전성을 해칠 수 있는 타입 파라미터를 오류 메세지를 보여주는 것으로 막습니다. 그런데 이를 무시하고 강제로 설정을 해야 하는 경우가 있을 수 있습니다.

그래서 코틀린은 `@UnsafeVariance` 라는 어노테이션을 제공합니다. 타입 파라미터에 붙여서 사용하면 변성 설정으로 인해 발생할 수 있는 위험성을 감수하고 강제로 실행시킬 수 있습니다.

만약에 클래스 자체를 `out` 으로 설정했다면 `add` `mergeOtherZoo` 메서드의 타입 파라미터에서 오류가 발생합니다. 이때 이 어노테이션을 사용하면 강제로 실행할 수 있습니다. 물론 런타임에 발생된 오류에 대한 처리는 어노테이션을 설정한 개발자의 몫입니다.

```kotlin
fun add(animal: @UnsafeVariance T) {
    animals.add(animal)
}

fun mergeOtherZoo(zoo: List<@UnsafeVariance T>) {
    this.animals.addAll(zoo)
}
```

​		

## 범위 지정

### 타입으로 받을 수 있는 범위 제한

지금까지 동물원을 예시로 설명을 드렸습니다. 그런데 다음과 같이 동물이랑 전혀 상관없는 `Int` 를 타입으로 넘겨도 객체 생성에 문제가 없습니다. 그래서 넘기는 타입 파라미터의 범위를 지정해줘야 합니다.  

`<T: Animal>` 이런 식으로 범위를 지정하면 됩니다. 설정하면 `Animal`  범위 내의 타입들만 넘길 수 있습니다.

```kotlin
fun main() {
    val zooWithNotAnimal = Zoo<Int>() // 오류 없음
}

class Zoo<T> {}
```

```kotlin
fun main() {
    val zooWithNotAnimal = Zoo<Int>() // Type argument is not within its bounds.
}

class Zoo<T: Animal> {}
```

​		

## 제네릭 클래스뿐만 아니라 제네릭 함수도 있다.

예시로 사용한 동물원은 제네릭 클래스입니다. 제네릭 클래스는 클래스 이름 오른쪽에 타입을 지정합니다. 반면에 클래스가 아닌 함수를 제네릭으로 정의할 때는 `fun` 키워드와 함수 이름 사이에 타입 파라미터를 넣어줘야 합니다.

```kotlin
fun List<T>.isNone(): Boolean {
    return this.isEmpty()
}
// 틀린 코드: Unresolved reference: T
```

```kotlin
fun <T> List<T>.isNone(): Boolean {
    return this.isEmpty()
}
// 맞는 코드: fun 키워드 다음에 타입 파라미터를 적어줘야 함
```

제네릭 함수를 잘만 활용하면 유틸성 함수들을 마구 찍어낼 수 있습니다. 코틀린 자체에서 제공하는 여러 유용한 확장함수들도 제네릭 함수로 정의되어 제공됩니다. 대표적으로 `Collection` 과 관련된 파일을 보면 유용한 기능의 함수가 제네릭으로 정의되어 있는 것을 알 수 있습니다.

​		

## 마무리

지금까지 예시를 통해서 제네릭을 이용한 클래스와 함수가 어떻게 정의되고 어떤 특징을 가지고 있는지 설명을 해봤습니다. 단순히 하나의 클래스나 함수를 제네릭으로 정의함으로써 코드 재사용성을 높일 수 있고 변성을 통한 타입 안전성도 얻을 수 있다는 것을 알았습니다.

설명을 위해 사용된 코드는 단순히 제네릭의 특징을 직관적으로 보여주기 위해서 굉장히 단순하고 허술하게 작성을 했으니 이 점 이해해주시면 감사하겠습니다. 혹시라도 틀린 내용이 있어서 알려주신다면 바로 수정하겠습니다. 감사합니다 :)

혹시 괜찮은 코틀린 강의를 찾고 계시다면 하나 추천드리겠습니다. 개인적으로 굉장히 만족하면서 들었던 강의입니다.

[**인프런 - 코틀린 고급편**](https://www.inflearn.com/course/%EC%BD%94%ED%8B%80%EB%A6%B0-%EA%B3%A0%EA%B8%89%ED%8E%B8#reviews)

​		

## 참조

[**Wikipedia 변성**](https://ko.wikipedia.org/wiki/%EA%B3%B5%EB%B3%80%EC%84%B1%EA%B3%BC_%EB%B0%98%EA%B3%B5%EB%B3%80%EC%84%B1_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))

[**인프런 - 코틀린 고급편**](https://www.inflearn.com/course/%EC%BD%94%ED%8B%80%EB%A6%B0-%EA%B3%A0%EA%B8%89%ED%8E%B8#reviews)
