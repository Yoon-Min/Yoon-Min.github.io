---
title: Android Compose UI State - Sanpshot 시스템
author: yoonmin
date: 2025-03-02 00:00:00 +0900
categories: [Android, Compose]
tags: [Android, Compose, UI, State, Snapshot]
render_with_liquid: true
---

![Image](https://github.com/user-attachments/assets/a032896c-9a78-480c-876f-bc171ff2a231)_사진: [Unsplash](https://unsplash.com/ko/사진/흰색-직물-위에-여러-초상화-qqd8APhaOg4?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[sarandy westfall](https://unsplash.com/ko/@sarandywestfall_photo?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

​		

**Compose UI State 포스팅 현황**

+ [x] <span style="color: #e05069">**Android Compose UI State - Sanpshot 시스템**</span>
+ [ ] <span style="color: #898989">Android Compose UI State - 어떻게 MutableState의 변경사항은 자동으로 처리될 수 있을까?  (예정)</span>

​		

# Intro

안드로이드의 최신 UI 제작 킷인 컴포즈는 상태를 등록하면 자동으로 관찰 및 수정을 처리하는 특징을 가지고 있습니다. 덕분에 컴포즈를 사용하는 개발자는 단순히 상태를 전달하기만 하면 되고, 상태가 변경됐을 때 이를 적용하는 수고를 컴포즈 런타임이 대신합니다. 이를 가능하게 만드는 대표 객체중 하나가 `MutableState` 입니다. 해당 상태 객체에 값을 저장하고 UI 데이터로 초기 설정하면 나중에 수정됐을 때 자동으로 적용됩니다.

이것은 이전 뷰 시스템에 비해 혁신적이라 할 수 있습니다. 굳이 **상태 변경을 적용한다는 코드를 명시할 필요 없이, 컴포즈가 자체적으로 다 처리**하기 때문입니다. 그렇다면 여기서 궁금한 점이 생깁니다. 컴포즈 런타임은 어떻게 상태를 관찰하고 변경 사항을 알아서 잘 적용할 수 있는 걸까요?

이러한 자동처리 기능은 스냅샷 시스템 기반으로 작동합니다. 그래서 컴포즈의 상태 업데이트를 본격적으로 파헤치기 전에 스냅샷을 알아보는 시간을 가질까 합니다. 따라서 이번 포스트는 컴포즈의 상태 추적 및 적용 시스템을 `MutableState` 객체를 통해 추적해 보고 내부 코드와 동작을 분석하기 전에 사전지식으로 알고 가야 할 스냅샷 시스템에 대해서 알아보겠습니다.

​		

# State 처리 과정 되짚기

## 상태 값을 UI 데이터로 적용하는 과정

```kotlin
@Composable
fun StatefulCounter(modifier: Modifier = Modifier) {
    var count by rememberSaveable { mutableStateOf(0) }
    StatelessCounter(
        count = count,
        onIncrement = { count++ },
        modifier = modifier
    )
}
```

1. 특정 뷰(위젯)에 표시해야 하는 데이터를 위해 상태 홀더 클래스인 `MutableState` 을 생성합니다. 

2. 해당 객체는 `count` 변수에 저장됩니다. 

3. 하위 컴포저블의 인자를 정의하는 과정에서 상태 값을 전달합니다.

   ```kotlin
   @Composable
   fun StatelessCounter(count: Int, onIncrement: () -> Unit, modifier: Modifier = Modifier) {
       Column(modifier = modifier.padding(16.dp)) {
           if (count > 0) {
               Text("You've had $count glasses.")
           }
           Button(
               onClick = onIncrement,
               enabled = count < 10,
               modifier = Modifier.padding(top = 8.dp)
           ) {
               Text("Add one")
           }
       }
   }
   ```

   하위 컴포저블에서 버튼을 클릭하면 상태 값이 증가한다는 것을 `onClick = onIncrement` 코드로 확인할 수 있습니다. 그리고 텍스트 뷰에 표시할 데이터로 `count` 를 지정했습니다.

​		

## 상태 값이 변경되는 과정

```kotlin
@Composable
fun StatelessCounter(count: Int, onIncrement: () -> Unit, modifier: Modifier = Modifier) {
    Column(modifier = modifier.padding(16.dp)) {
        if (count > 0) {
            Text("You've had $count glasses.")
        }
        Button(
            onClick = onIncrement,
            enabled = count < 10,
            modifier = Modifier.padding(top = 8.dp)
        ) {
            Text("Add one")
        }
    }
}
```

1. 모바일 애플리케이션 사용자가 버튼을 누릅니다.
2. 클릭 이벤트가 발생합니다, - `onClick`
3. 컴포저블 함수 내용 토대로 `onIncrement` - `onIncrement = { count++ }` 가 실행됩니다.
4. `count` 값이 증가합니다.
5. 컴포즈 시스템이 해당 컴포저블 내 `count` 상태 객체가 수정된 것을 인식합니다.
6. 리컴포지션을 진행하여 최신 값을 UI 데이터로 적용합니다.

​		

## 여기서 생기는 궁금증

```
1. MutableState 내부 구조는?
2. Compose 시스템의 상태 객체 감지 방법은?
```

컴포즈 런타임이 실시간으로 상태 객체를 관찰할 수 있는 것은 `State` 객체와 연관이 있습니다. 이 두 가지의 연관성을 이해하려면 먼저 `스냅샷` 시스템을 알 필요가 있습니다. 

​		

# Snapshot

## 특정 값의 순간을 캡쳐

Compose는 스냅샷이라는 시스템을 사용하여 UI 상태 변경을 자동으로 추적하고 업데이트합니다. 스냅샷은 콘솔 게임에서 현재 진행 상황을 저장하는 것처럼 프로그램의 특정 시점 상태를 저장하고 관리합니다. 그래서 스냅샷을 사용하면 상태 값을 저장(스냅샷)하고 해당 저장한 상태를 불러오며, 원하면 다른 값으로 업데이트까지 가능합니다.

```
1. 특정 상태의 값을 캡쳐(스냅샷)
2. 해당 상태 값을 사용하기 위해 로딩
3. 상태 값 업데이트
```

​		

## 1. 특정 상태의 값을 캡쳐

```kotlin
val state1 = mutableIntStateOf(1)
val s1 = Snapshot.takeSnapshot()
```

스냅샷을 사용하는 방법은 간단합니다. 상태를 생성하고 스냅샷을 하는 코드을 명시하면 됩니다. 예시로 `mutableIntStateOf` 를 사용해서 상태를 만들고 해당 상태를 스냅샷 객체의 `takeSnapshot` 메서드로 캡처해 봤습니다. 두 번째 라인의 코드로 인해 첫 번째 라인의 상태 값은 캡처되어 저장됩니다.

​		

## 2. 해당 상태 값을 사용하기 위해 로딩

```kotlin
val state1 = mutableIntStateOf(1)
val s1 = Snapshot.takeSnapshot()
s1.enter {
    println(state1.intValue)
}
s1.dispose()
```

스냅샷한 값을 불러오기 위해서는 스냅샷 객체로 진입해야 합니다. 진입을 위한 `enter` 메서드를 호출하고 람다 영역에 해당 상태 값을 불러오면 ***스냅샷으로부터 상태 값 불러오기*** 성공입니다. 마지막으로 스냅샷 사용이 끝나면 닫아줍니다.

### 스냅샷은 독립적

```kotlin
val state1 = mutableIntStateOf(1)
val s1 = Snapshot.takeSnapshot()
state1.intValue = 2
println(state1.intValue)
s1.enter {
    println(state1.intValue)
}

/*
출력 결과
2
1
*/
```

여기서 궁금한 점이 생깁니다. 스냅샷을 만들고 나서 상태 값을 변경한다면, 이전에 만든 스냅샷에도 영향이 갈까요? 직접 코드로 확인해 보겠습니다. 위처럼 스냅샷 객체 생성 이후에 상태 값을 `2` 로 변경하고 메인 함수 내에서 출력 한 번, 스냅샷 영역 내에서 출력을 해봅니다.

메인 함수에서는 `2` 가 출력됐는데 스냅샷 영역에서는 당시 캡처값인 `1` 이 출력됐습니다. 이처럼 스냅샷은 한 번 캡처한 특정 상태를 그때의 버전으로 유지하기 때문에 중간에 외부에서 상태 값을 바꿔도 변경 사항을 다른 스냅샷들에게 알리지 않는 한, 스냅샷 영역에서 안정적인 상태 객체 사용이 가능합니다.

### 가변 스냅샷

```
Exception in thread "main" java.lang.IllegalStateException: Cannot modify a state object
```

위에서 사용한 스냅샷은 읽기 전용 스냅샷이라 해당 영역 내에서 상태 객체 수정(Write)이 불가능합니다. 해당 객체를 수정하려고 시도하면 오류가 발생합니다. 기본적으로 스냅샷은 읽기 전용입니다. 

```kotlin
val state1 = mutableIntStateOf(1)
val s1 = Snapshot.takeMutableSnapshot()
```

만약 스냅샷 영역에서 캡처한 상태 객체의 값을 변경하고 싶다면 가변 스냅샷을 이용해야 합니다. 가변 스냅샷은 생성할 때 `takeMutableSnapshot` 을 명시합니다. 이후에 스냅샷 영역에서 상태 값을 변경하면 오류 없이 정상 실행됩니다.

```kotlin
val state1 = mutableIntStateOf(1)
val s1 = Snapshot.takeMutableSnapshot()
state1.intValue = 2
println(state1.intValue)
s1.enter {
    println(state1.intValue)
    state1.intValue = 3
    println(state1.intValue)
}

/*
출력 결과
2
1
3
*/
```



​		

## 3. 상태 값 업데이트

```kotlin
val state1 = mutableIntStateOf(1)
val s1 = Snapshot.takeMutableSnapshot()
state1.intValue = 2
println(state1.intValue)

s1.enter {
    println(state1.intValue)
    state1.intValue = 3
    println(state1.intValue)
}
println(state1.intValue)

/*
출력 결과
2
1
3
2
*/
```

스냅샷 영역은 캡처한 시점의 상태를 사용할 수 있게 해줍니다. 더불어 가변 스냅샷을 이용해서 해당 영역 내에서 상태 값을 변경할 수도 있습니다. 하지만 가변 스냅샷 내에 상태 값을 변경했어도, 스냅샷은 외부와 독립적인 성질을 가지고 있기 때문에 스냅샷이 닫히면 변경 사항이 적용되지 않습니다.

### apply

```kotlin
val state1 = mutableIntStateOf(1)
val s1 = Snapshot.takeMutableSnapshot()

s1.enter {
    state1.intValue = 3
    println(state1.intValue)
}

println(state1.intValue)
s1.apply()
println(state1.intValue)

/* 
출력 결과
3
1
3
*/
```

스냅샷 영역 내 변경 사항을 적용하려면 `apply` 을 호출해야 합니다. 스냅샷은 루트 스냅샷의 역할을 하는 `GlobalSnapshot` 밑에 많은 자식 스냅샷들이 존재합니다. 따라서 적용 메서드를 호출하면 해당 스냅샷의 상위 스냅샷에 해당하는 글로벌 스냅샷이 변경 사항을 적용합니다. 이후 출력하면 스냅샷 영역에서 변경했던 `3` 이 메인 함수 영역에서도 적용된 것을 확인할 수 있습니다.

​		

# 동시성 제어 시스템에서 바라본 Snapshot

## Compose 공식문서에서 말하길

> "*Composable function은 병렬적으로 실행될 수 있다.*"
>
> "Compose가 다중 코어를 활용하고 화면에 없는 구성 가능한 함수를 낮은 우선순위로 실행할 수 있다."
>
> "이 최적화는 구성 가능한 함수가 백그라운드 스레드 풀 내에서 실행될 수 있음을 의미한다."

이 말은 즉, 컴포저블 함수는 동시에 실행될 수 있어야 하므로 **멀티스레딩 환경**을 고려해야 합니다. 멀티스레딩에서 가장 중요한 것은 하나의 상태를 여러 스레드가 공유할 때의 적절한 동기화입니다. 가장 단순하고 확실한 해결책은 상태 자체를 불변으로 만들어서 값의 변경으로 인한 동기화 문제를 차단하는 것입니다.

하지만 특정 이벤트에 의해 지속적으로 값이 바뀌는 상태를 활용해야 하는 모바일 운영체제에서는 모든 상태를 불변으로 활용하는 것은 거의 불가능합니다. 결국 Android 컴포즈 기반의 UI 레이어는 가변 상태를 여러 스레드에서 안전하게 운영해야 하며, 이를 위해 ***격리***와 ***다중 버전***이 필요합니다.

​		

## Snapshot의 다중 버전 동시성 제어(MVCC) 지원

### 격리

위의 스냅샷 기본 내용을 정리해 보면 스냅샷은 독립성 즉, 고립성(isolation) 기반으로 특정 시점의 상태를 캡처하고 불러오고 업데이트한다는 것을 알 수 있습니다. 이것을 멀티스레딩 관점에서 해석해 본다면, 특정 스레드의 스냅샷 내에서 해당 스냅샷이 `apply` 될 때까지 다른 스레드에서 상태 값 변경 사항이 표시되지 않습니다. 스냅샷은 ***고립성*** 특징을 가지고 있으니까요.

 특정 시점에서 어떤 상태를 작업하고 그동안 다른 스레드가 그 상태를 건드리지 못하도록 하려면 보통 `Mutex` 같은 것을 사용해 해당 상태로의 접근을 막습니다. 하지만 스냅샷을 사용하면 다른 스레드가 상태를 변경해도 해당 변경사항이 적용되기 전까지 다른 스냅샷에서 변경 사항을 무시한 채로 기존에 들고 있던 상태 버전을 유지한 채로 독립적인 작업이 가능합니다. 즉, 스냅샷이 적용되고 전역 스냅샷이 처리하기까지 스냅샷 내부의 상태 변경 사항이 다른 스레드에 표시되지 않습니다.

### 다중 버전

격리를 구현하기 위해서 데이터의 다중 버전이 필요합니다. 게임을 예시로 봅시다. 콘솔 게임을 하면서 특정 진행상황마다 자동저장 혹은 수동저장을 하게 됩니다. 만약에 특정 시점부터 다시 플레이하고 싶다면 해당 시점까지 저장된 파일을 불러와서 해당 시점부터 플레이할 것입니다. 

스냅샷도 이러한 데이터 다중 버전을 지원합니다. 값을 변경했다고 이전 값을 폐기하는 것이 아니라 버전으로 남겨둬서 특정 시점의 상태를 제공합니다. 이제 왜 스냅샷 명칭이 스냅샷인지 이해가 조금 됩니다. 특정 시점의 값을 다양한 버전으로 관리할 수 있게 하여 여러 스레드에서 값을 읽거나 변경해도 전역으로 바로 업데이트하는 것이 아닌, 버전으로 추가해 다른 참조 영역에 영향이 가지 않게 격리 상태를 보장하는 것, 이것이 스냅샷입니다.

![Image](https://github.com/user-attachments/assets/410c8cf1-705a-4e55-9e75-aba54147cdc0)

​		

# Snapshot 구조

```kotlin
sealed class Snapshot(
    id: Int,

    /**
     * A set of all the snapshots that should be treated as invalid.
     */
    internal open var invalid: SnapshotIdSet
) {
  /* ... */
}
```

스냅샷은 `saled class` 로 해당 클래스 내부에 여러 종류의 스냅샷이 존재합니다. 제가 예제에 사용한 스냅샷은 읽기 전용 스냅샷과 가변 스냅샷입니다.

```
GlobalSnapshot (androidx.compose.runtime.snapshots)
MutableSnapshot (androidx.compose.runtime.snapshots)
NestedMutableSnapshot (androidx.compose.runtime.snapshots)
NestedReadonlySnapshot (androidx.compose.runtime.snapshots)
ReadonlySnapshot (androidx.compose.runtime.snapshots)
TransparentObserverMutableSnapshot (androidx. compose.runtime.snapshots)
```

​		

## 계층 구조로 이루어진 Snapshot

### GlobalSnapshot

```kotlin
fun main() {
    val state1 = mutableIntStateOf(1)
    val s1 = Snapshot.takeMutableSnapshot()
    println(Snapshot.current)
    println(Snapshot.isInSnapshot)

    s1.enter {
        state1.intValue = 3
        println(state1.intValue)
        println(Snapshot.current)
        println(Snapshot.isInSnapshot)
    }
}

/*
출력 결과
androidx.compose.runtime.snapshots.GlobalSnapshot@5c3bd550
false
3
androidx.compose.runtime.snapshots.MutableSnapshot@14bf9759
true
*/
```

여러 종류의 스냅샷중 글로벌 스냅샷은 말 그대로 전역 스냅샷을 뜻합니다. 스냅샷의 종류는 다양하며 여러 스냅샷들이 서로 연결되어 하나의 트리구조를 형성합니다. 글로벌 스냅샷은 이 트리의 루트 노드를 담당합니다.

처음 예제 코드에 현재 동작중인 스냅샷 정보를 출력하는 코드를 몇 개 추가해 확인해 봅시다. 출력 결과를 보면 메인 영역에서 동작하는 스냅샷은 글로벌 스냅샷, 따로 가변 스냅샷을 만들어서 해당 스냅샷 영역에서 동작하는 스냅샷은 가변 스냅샷이라 출력됩니다. 즉, 글로벌 스냅샷은 스냅샷 계층 구조의 루트 노드로 자리하고 자식 스냅샷들로부터 활동을 보고받습니다.

스냅샷 영역에서 상태 값을 변경하고 메인 영역으로 나와 `apply` 메서드를 호출한 예제를 기억하시나요? 자식 스냅샷 객체로 적용 요청이 발생하면 이 변경사항을 글로벌 스냅샷에 적용합니다. 만약 적용 요청 영역이 중첩 스냅샷 내부라면 상위 스냅샷에 적용합니다.

```kotlin
fun main() {
    val state1 = mutableIntStateOf(1)
    val s1 = Snapshot.takeMutableSnapshot()

    s1.enter {
        state1.intValue = 3
        println(state1.intValue)

        val s2 = Snapshot.takeMutableSnapshot()
        s2.enter {
            state1.intValue = 4
            println(state1.intValue)
        }
        println(state1.intValue)
        s2.apply()
        println(state1.intValue)
    }
    println(state1.intValue)
}

/*
출력 결과
3
4
3
4
1
*/
```

코드를 보면 s1 스냅샷 내에서 s2 자식 스냅샷을 생성한 다음, 자식 스냅샷 내에서 상태를 변경합니다. 그리고 상태 적용을 s1 스냅샷 내부에서 요청하면 부모 스냅샷인 s1에도 변경사항이 적용됩니다. 물론 이 과정은 전부 s1 스냅샷 내부에서 발생한 것이기 때문에 s1 밖의 영역인 글로벌 영역에는 전혀 영향이 가지 않습니다. 마지막 메인 영역에서의 상태 값이 `1` 로 출력된 것으로 알 수 있습니다.

### 계층 구조

![Image](https://github.com/user-attachments/assets/418f0e39-cc02-460f-8619-8172c56fd70f)

​		

# 정리

지금까지 스냅샷 시스템에 대해서 간략하게 알아 봤습니다. 스냅샷 시스템은 방대해서 제가 소개해 드린 내용은 극히 일부라 할 수 있습니다. 최대한 복잡하지 않게 어떤 역할을 하는지 쉽게 설명하려고 노력했는데 괜찮으셨나요? 스냅샷 객체의 내부구조는 방대하므로 궁금하신 분들은 내부 구조를 한 번 분석해 보시는 것도 좋을 것 같습니다.

컴포즈 상태 시리즈의 첫 번째 글이 이렇게 끝났습니다. 다음 두 번째 글은 컴포즈에서 제공하는 상태 홀더 클래스인 `State` 와 스냅샷 시스템이 어떻게 연결되어 자동 상태 관리를 하는지 톺아볼 예정입니다. 만약 다음 글이 포스팅된 상태라면 이 글에 이어 다음 글까지 읽는 것을 추천드립니다. 감사합니다.

​		

# 참조

[**Introduction to the Compose Snapshot system**](https://blog.zachklipp.com/introduction-to-the-compose-snapshot-system/)

