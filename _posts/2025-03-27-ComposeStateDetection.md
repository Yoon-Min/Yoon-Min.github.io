---
title: Android Compose UI State - MutableState로 분석해 보는 상태 자동 업데이트
author: yoonmin
date: 2025-03-27 00:00:00 +0900
categories: [Android, Compose]
tags: [Android, Compose, UI, State, Snapshot, MutableState]
render_with_liquid: true
---

![Image](https://github.com/user-attachments/assets/4fe074d6-c7af-4f72-bd7a-4c60cb4ee0d9)_사진: [Unsplash](https://unsplash.com/ko/사진/검은색-평면-스크린-컴퓨터-모니터-근처에서-켜진-검은색-노트북-컴퓨터-컴퓨터-2836IR07ocg?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Frederic Köberl](https://unsplash.com/ko/@internetztube?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

​		

**Compose UI State 포스팅 현황**

+ [x] <span style="color: #898989">Android Compose UI State - Sanpshot 시스템</span>
+ [x] <span style="color: #e05069">**Android Compose UI State - 어떻게 MutableState의 변경사항은 자동으로 처리될 수 있을까?**  </span>

​		

# Intro

지난 포스팅에서 스냅샷 개념을 이야기했습니다. 스냅샷을 간단하게 복습해 보면 특정 상태의 시점을 저장하여 독립적으로 해당 시점을 활용할 수 있는 객체입니다. 이러한 특징으로 깃(Git)과 유사하다는 언급도 했었는데요, 이번 글에서는 스냅샷 기반으로 동작하는 가변 상태 객체에 대해서 정리해 보겠습니다.

시작하기 앞서 가볍게 `MutableState` 개념에 대해서 짚고 넘어가겠습니다. 가변 상태 객체(MutableState)는 이름 그대로 상태 값을 보유하는 홀더 객체입니다. 상태 값을 읽거나 수정할 수 있으며, 수정 작업이 발생하면 컴포즈 시스템이 이를 인식하여 리컴지션(recomposition) 작업을 진행합니다.

​		

# 상속 관계

컴포저블 함수에서 가변 상태 객체를 사용한다면 보통 `MutableState` 혹은 `mutableStateOf` 일 것입니다. 이 둘의 내부 상속 클래스를 타고 가다 보면 위 클래스를 마주할 것입니다. 해당 클래스는 가변 상태 객체 구조를 잘 나타내고 있습니다.

​		

## 상속 및 구현 관계

![Image](https://github.com/user-attachments/assets/84674ece-2216-4232-af69-7863b4dc9a6b)



스냅샷 가변 상태 구현체는 스냅샷 가변 상태 인터페이스와 상태 객체 구현 추상 클래스를 구현한 형태입니다. 이 두 개를 합쳐서 상태를 여러 버전으로 관리하고 쓰기와 읽기 작업을 처리합니다.

### State

```kotlin
@Stable
interface State<out T> {
    val value: T
}
```

`State` 인터페이스는 아주 단순한 구조로 값만 가집니다.

​		

### MutableState

```kotlin
@Stable
interface MutableState<T> : State<T> {
    override var value: T
    operator fun component1(): T
    operator fun component2(): (T) -> Unit
}
```

`MutableState` 인터페이스는 `State` 인터페이스 내용을 가져오고 컴포넌트 연산 함수를 추가한 형태입니다. 해당 연산 함수는 변수 두 개 선언을 통해 상태 값 읽기와 쓰기 작업을 좀 더 단순한 형태로 지원합니다.

```kotlin
var (foo, setFoo) = remember { mutableStateOf(0) } 
// set -> setFoo(123)
// get -> foo == 123 
```

​		

### SnapshotMutableState

```kotlin
interface SnapshotMutableState<T> : MutableState<T> {
    /**
     * A policy to control how changes are handled in a mutable snapshot.
     */
    val policy: SnapshotMutationPolicy<T>
}
```

스냅샷 가변 상태 객체 인터페이스는 가변 상태 인터페이스 내용에 정책을 추가한 형태입니다. 여기서 정의하는 정책이란 가변 상태 객체가 상태 수정이 가능함에 따라 변경 사항을 처리하는 방법을 의미합니다.

상태를 수정한다는 것은 이전 상태와 수정을 거치고 난 후의 현재 상태를 비교했을 때 서로 다른 것을 의미합니다. 이를 판별하기 위해서는 비교 기준을 무엇으로 잡을 것인지 정해야 합니다. mutableStateOf 생성코드를 보면 알 수 있듯이 구조적 동등이 기본 비교 기준으로 설정되어 있습니다.

```kotlin
@StateFactoryMarker
fun <T> mutableStateOf(
    value: T,
    policy: SnapshotMutationPolicy<T> = structuralEqualityPolicy()
): MutableState<T> = createSnapshotMutableState(value, policy)
```

````kotlin
@Suppress("UNCHECKED_CAST")
fun <T> structuralEqualityPolicy(): SnapshotMutationPolicy<T> =
    StructuralEqualityPolicy as SnapshotMutationPolicy<T>

private object StructuralEqualityPolicy : SnapshotMutationPolicy<Any?> {
    override fun equivalent(a: Any?, b: Any?) = a == b

    override fun toString() = "StructuralEqualityPolicy"
}
````

구조적 동등을 비교 기준으로 삼는 정책은 상태의 값이 구조적으로(==) 동등한지 아닌지 검사합니다. 만약 같다면 이전 상태와 수정한 상태를 동일한 것으로 취급합니다. 

그래서 정책 객체를 통해 값의 변화가 없다고(이전 상태와 수정한 상태가 동일)판단하면 수정 작업을 진행하지 않습니다. 반대로 수정이 발생(이전 상태와 수정한 상태가 다름)했다고 판단하면 수정 작업을 진행합니다.

가변 상태 생성코드를 보면 알 수 있지만, 가변 상태 객체를 생성할 때 정책을 직접 설정할 수 있습니다. 내부적으로 구조적 동등을 기본으로 사용하지만 필요에 따라 직접 상태 변경 기준을 정의한 정책으로 적용가능합니다.

​		

### StateObject

````kotlin
interface StateObject {
    val firstStateRecord: StateRecord
    fun prependStateRecord(value: StateRecord)
    fun mergeRecords(
        previous: StateRecord,
        current: StateRecord,
        applied: StateRecord
    ): StateRecord? = null
}
````

가변 상태 객체는 스냅샷 기반으로 동작하기 때문에 상태를 여러 버전으로 관리합니다. 특정 시점의 상태 값을 하나의 노드처럼 관리하여 여러 노드들을 서로 연결하는 연결 리스트 구조를 사용합니다.

노드의 역할을 하는 객체가 바로 StateObject 입니다. 이 객체는 노드로서 연결 리스트를 구성하는 요소기 때문에 노드 추가나 병합과 같은 작업 및 연결된 다음 노드를 가리키는 포인터가 존재합니다.

firstStateRecord

: 연결 리스트의 첫 번째 노드

prependStateRecord

: 연결 리스트 시작 부분에 새 노드를 추가합니다. 새로 추가되어서 첫 번째 노드가 됐기 때문에 반드시 firstStateRecord 로 지정해야 합니다.

merge

: 깃(Git)에서 사용하는 병합과 동일합니다. 정의한 방식으로 충돌 상태를 병합합니다. 병합 방법은 정책(policy)에 정의합니다.

![Image](https://github.com/user-attachments/assets/43374222-4602-4157-bb51-6cde804d1095)

​		

### StateObjectImpl

````kotlin
internal abstract class StateObjectImpl internal constructor() : StateObject {
    private val readerKind = AtomicInt(0)
    internal fun recordReadIn(reader: ReaderKind) {
        do {
            val old = ReaderKind(readerKind.get())
            if (old.isReadIn(reader)) return

            val new = old.withReadIn(reader)
        } while (!readerKind.compareAndSet(old.mask, new.mask))
    }

    internal fun isReadIn(reader: ReaderKind): Boolean =
        ReaderKind(readerKind.get()).isReadIn(reader)
}
````

상태 객체 인터페이스 구현체는 변경 사항 기록을 최적화하는 동작이 정의되어 있습니다.

​		

# 내부 구조 분석

<script src="https://gist.github.com/Yoon-Min/a81377e74bd70f0bfd24e64a081da7ab.js"></script>

스냅샷 가변 상태 객체는 스냅샷 가변 상태 인터페이스에 상태 객체가 합쳐진 구현체기 때문에 위에서 설명한 인터페이스의 추상 멤버들을 구현합니다.









