---
title: Android Compose UI State - MutableState로 분석해 보는 상태 자동 업데이트
author: yoonmin
date: 2025-03-28 00:00:00 +0900
categories: [Android, Compose]
tags: [Android, Compose, UI, State, Snapshot, MutableState]
render_with_liquid: true
---

![test]({{ site.url }}/assets/img/post/branch4/thumbnail.jpg)

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

![test]({{ site.url }}/assets/img/post/branch4/1.png)

컴포저블 함수에서 가변 상태 객체를 사용한다면 보통 `MutableState` 혹은 `mutableStateOf` 일 것입니다. 이 둘의 내부 상속 클래스를 타고 가다 보면 위 클래스를 마주할 것입니다. 해당 클래스는 가변 상태 객체 구조를 잘 나타내고 있습니다.

​		

## 상속 및 구현 관계



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

![test]({{ site.url }}/assets/img/post/branch4/2.png)

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

## parameter

```kotlin
internal open class SnapshotMutableStateImpl<T>(
    value: T,
    override val policy: SnapshotMutationPolicy<T>
)
```

파라미터부터 보면 초기 값과 정책이 정의되어 있습니다. 가변 상태 객체 생성시 개발자가 반드시 전달해야 하는 값은 `value` 이고 정책은 위에서 설명했듯이 내부적으로 기본 값을 지정해 놨기 때문에 선택사항입니다.

`MutableState` 의 경우, 현재는 일부 타입들로 강제하는 가변 상태 객체가 존재합니다. 이 글에서 설명하고 있는 가변 상태는 제네릭을 사용하는 형태입니다. 안드로이드 스튜디오에서도 알려주지만, 현재는 타입 처리에 대한 성능 이슈로 특정 타입에 대한 가변 상태 객체 사용을 권장하고 있습니다.

```kotlin
internal open class SnapshotMutableIntStateImpl(
    value: Int // 제네릭이 아닌 Int 사용
)
```

​		

## StateStateRecord

```kotlin
private class StateStateRecord<T>(myValue: T) : StateRecord() {
    override fun assign(value: StateRecord) {
        @Suppress("UNCHECKED_CAST")
        this.value = (value as StateStateRecord<T>).value
    }
```

`StateObject` 는 `StateRecord` 을 노드로 사용하여 연결리스트 형태로 상태를 관리하는 객체임을 설명했습니다. 가변 상태 객체 역시 `StateObject` 구현체기 때문에 상태를 `StateRecord` 로 관리합니다. 그래서 상태 레코드 추상 클래스를 구현해서 사용하는 것입니다.

### StateRecord

```kotlin
abstract class StateRecord {
    internal var snapshotId: Int = currentSnapshot().id
    internal var next: StateRecord? = null
    abstract fun assign(value: StateRecord)
    abstract fun create(): StateRecord
}
```

상태 레코드는 노드 역할을 하기 때문에 다음 노드를 가리키는 next 변수와 해당 레코드가 생성된 컨텍스트(스냅샷) 아이디를 가집니다. 또한 값을 할당하거나 새 레코드를 생성할 수도 있습니다. 

```kotlin
internal fun currentSnapshot(): Snapshot =
    threadSnapshot.get() ?: currentGlobalSnapshot.get()
```

스냅샷 아이디는 현재 스레드가 활동하는 스냅샷의 아이디를 의미합니다. 현재 스레드 스냅샷이 존재하지 않는다면 글로벌 스냅샷으로 대체합니다.

​		

## next

```kotlin
private var next: StateStateRecord<T> = StateStateRecord(value).also {
    if (Snapshot.isInSnapshot) {
        it.next = StateStateRecord(value).also { next ->
            next.snapshotId = Snapshot.PreexistingSnapshotId
        }
    }
}
```

가변 상태 객체는 `StateObject` 의 구현체이기 때문에 상태 레코드를 노드로 활용하여 연결리스트로 상태 관리를 합니다. 연결리스트 내 상태를 하나의 노드로 관리하기 위해서는 포인터 역할의 변수가 필요합니다. next 변수가 이에 해당합니다.

![img]({{ site.url }}/assets/img/post/branch4/3.png)

가변 상태 객체를 생성하면 `next` 변수가 생성되는데, 잘 보면 상태 레코드를 하나가 아닌 **두 개를 생성**하고 있습니다. 상태는 시간이 지남에 따라 값이 변화하기 때문에 '**상태**'라 할 수 있습니다. 특정 시점에 초기화 시점의 순수 상태 컨텍스트 접근이 필요할 수 있으므로 미리 초기 상태 백업용으로 하나 더 추가하는 것입니다.

```kotlin
override val firstStateRecord: StateRecord
    get() = next
```

그리고 `next` 변수는 가장 첫 번째 노드가 됩니다.

​		

## value

```kotlin
override var value: T
    get() = next.readable(this).value
    set(value) = next.withCurrent {
        if (!policy.equivalent(it.value, value)) {
            next.overwritable(this, it) { this.value = value }
        }
    }
```

`value` 는 상태 값이 저장되는 변수입니다. 해당 변수에 초기 상태 값을 저장해 놓고 수정과 읽기를 여러 번 작업하면 상태의 여러 버전이 연결리스트의 노드로 쌓일 겁니다. 만약 상태 노드가 여러 개라면, get 요청시 유효하면서 가장 최신 버전의 노드를 반환해야 할 것입니다.

### get()

```kotlin
fun <T : StateRecord> T.readable(state: StateObject): T {
    val snapshot = Snapshot.current
    snapshot.readObserver?.invoke(state)
    return readable(this, snapshot.id, snapshot.invalid) ?: sync {
        val syncSnapshot = Snapshot.current
        @Suppress("UNCHECKED_CAST")
        readable(state.firstStateRecord as T, syncSnapshot.id, syncSnapshot.invalid) ?: readError()
    }
}
```

가변 상태 객체 게터의 핵심은 현재 첫 번째 노드부터 끝 노드까지 검사하면서 유효한 가장 최신의 노드를 반환하는 것입니다. `readable` 함수가 이 역할을 맡습니다. 그런데 자세히 보면 만약 유효한 최신 노드가 존재하지 않으면 한 번 더 탐색을 시도합니다.

스냅샷도 멀티스레딩 환경에서 사용되기 때문에 다른 영역에서 다른 스레드에 의해 전역 스냅샷이 `advenced(적용)`하여 상태가 덮여 쓰여질 수 있습니다. 이때는 `null`이 반환되기도 합니다. 그래서 이러한 상황을 대비하여 한 번더 탐색을 시도하는 겁니다.

```kotlin
private fun <T : StateRecord> readable(r: T, id: Int, invalid: SnapshotIdSet): T? {
    var current: StateRecord? = r
    var candidate: StateRecord? = null
    while (current != null) {
        if (valid(current, id, invalid)) {
            candidate = if (candidate == null) current
            else if (candidate.snapshotId < current.snapshotId) current else candidate
        }
        current = current.next
    }
    if (candidate != null) {
        @Suppress("UNCHECKED_CAST")
        return candidate as T
    }
    return null
}
```

`readable` 함수의 내부 구조를 보면 가변 상태 객체의 첫 번째 노드인 next 변수부터 시작하여 끝 노드까지 돌면서 `valid` 하고 최신인 후보 노드를 찾아 반환합니다. 

현재 탐색중인 노드의 아이디가 후보 노드의 아이디보다 높다는 것은 탐색중인 노드가 후보 노드보다 최근에 생성됐다는 것을 의미합니다. 스냅의 아이디 값은 순차적으로 증가하기 때문입니다. 그래서 아이디 값이 높을수록 최신 상태입니다.

```kotlin
if (valid(current, id, invalid)) {
    candidate = if (candidate == null) current
    else if (candidate.snapshotId < current.snapshotId) current else candidate
}
```

그래서 위와 같은 코드가 실행되면 후보 변수에는 유효하면서 가장 최신의 상태 노드가 저장됩니다.

​		

### valid

```kotlin
private fun valid(currentSnapshot: Int, candidateSnapshot: Int, invalid: SnapshotIdSet): Boolean {
    return candidateSnapshot != INVALID_SNAPSHOT && candidateSnapshot <= currentSnapshot &&
        !invalid.get(candidateSnapshot)
}
```

최신 노드라는 조건과 더불어 유효한 노드를 기준으로 삼는데, 그렇다면 무엇이 유효한 걸까요? 유효하다고 판단하는 기준은 해당 함수 내부코드를 보면 알 수 있습니다.

```
1. 후보 스냅샷의 아이디 값은 유효한 스냅샷이어야 한다. 즉, 아이디 값이 invalid를 의미하는 0이 아니어야 한다.
2. 후보 스냅샷의 아이디가 현재 스냅샷 아이디보다 같거나 작아야 한다. 즉, 현재 스냅샷보다 과거에 만들어 졌거나 같은 시기에 만들어 진 것이어야 한다.
3. 유효하지 않은 스냅샷 리스트에서 후보 스냅샷이 해당되면 안 된다.
```

​		

### set()

```kotlin
set(value) = next.withCurrent {
    if (!policy.equivalent(it.value, value)) {
        next.overwritable(this, it) { this.value = value }
    }
}
```

가변 상태 객체 세터의 핵심은 유효하면서 가장 최신 상태 레코드와 요청 수정 값을 비교하여 기존 값에 수정 값을 덮어씌우는 것입니다. 

```kotlin
internal fun <T : StateRecord> current(r: T) =
    Snapshot.current.let { snapshot ->
        readable(r, snapshot.id, snapshot.invalid) ?: sync {
            Snapshot.current.let { syncSnapshot ->
                readable(r, syncSnapshot.id, syncSnapshot.invalid)
            }
        } ?: readError()
    }
```

상태 값을 수정하려면 현재 유효한 최신 상태의 값과 비교할 필요가 있습니다. 바로 덮어쓰지 않고 굳이 비교를 하는 이유는 동일한 값을 굳이 시스템 자원을 소모하면서 덮어쓰기 작업을 할 필요는 없기 때문에 먼저 검증하는 것입니다. `readable` 함수를 활용하여 가장 최신이면서 유효한 상태 레코드를 꺼냅니다. 

```kotlin
if (!policy.equivalent(it.value, value)) {
    next.overwritable(this, it) { this.value = value }
}
```

그리고 정책에 정의한 비교 기준으로 값을 서로 비교합니다. 만약 값이 같다면 덮어쓰기 작업 없이 종료하고, 값이 다르다면 덮어 쓰는 작업을 진행합니다. 

```kotlin
internal inline fun <T : StateRecord, R> T.overwritable(
    state: StateObject,
    candidate: T,
    block: T.() -> R
): R {
    var snapshot: Snapshot = snapshotInitializer
    return sync {
        snapshot = Snapshot.current
        this.overwritableRecord(state, snapshot, candidate).block()
    }.also {
        notifyWrite(snapshot, state)
    }
}

internal fun <T : StateRecord> T.overwritableRecord(
    state: StateObject,
    snapshot: Snapshot,
    candidate: T
): T {
    if (snapshot.readOnly) {
        snapshot.recordModified(state)
    }
    val id = snapshot.id

    if (candidate.snapshotId == id) return candidate

    val newData = sync { newOverwritableRecordLocked(state) }
    newData.snapshotId = id

    if (candidate.snapshotId != Snapshot.PreexistingSnapshotId) snapshot.recordModified(state)

    return newData
}
```

해당 작업은 현재 가변 스냅샷에서 생성된 경우 가능합니다. 그래서 현재 스냅샷 아이디와 후보 노드(유효한 최신 노드)의 아이디가 같은지 비교하는 작업을 진행합니다. 만약 같다면 바로 후보 노드의 값을 수정 값으로 적용하고, 같지 않다면 새 레코드를 생성해서 현재 스냅샷 아이디 값과 맞추는 작업을 한 후에 수정 값을 적용합니다.

```kotlin
internal fun <T : StateRecord> T.newOverwritableRecordLocked(state: StateObject): T {
    @Suppress("UNCHECKED_CAST")
    return (usedLocked(state) as T?)?.apply {
        snapshotId = Int.MAX_VALUE
    } ?: create().apply {
        snapshotId = Int.MAX_VALUE
        this.next = state.firstStateRecord
        state.prependStateRecord(this as T)
    } as T
}
```

만약 새 레코드를 생성한다면, 레코드를 생성하고 가져오는 과정동안 해당 레코드의 아이디를 정수형 최대 값으로 설정합니다. 이는 멀티스레딩 환경에서 새 레코드 대상의 동시다발적 참조에 의한 문제가 발생할 수 있으므로 보호 차원에서 다른 스레드가 접근하지 못하게 블락을 거는 것입니다.

그런데 아이디 값을 매우 큰 값을 잡는 것으로 어떻게 블락 효과를 내는 걸까요? 정답은 노드 유효성 검사에 있습니다. 분명 상태를 읽기 위해서는 유효성 검사를 통과하고 최신인 레코드만 가져와야 한다고 위에서 언급했습니다. 

유효성 판단 조건중 하나가 후보 스냅샷 아이디 값이 현재 스냅샷 아이디보다 같거나 작아야 한다고 했습니다. 그런데 후보 스냅샷이 `Int.MAX_VALUE` 라면 이 조건을 충족할 수 없습니다. 그래서 유효성 검사에서 아이디 값이 정수형 최대 값인 레코드는 자연스럽게 후보에서 탈락되고 어떤 스레드도 읽을 수 없는 상황이 만들어지게 됩니다.

마치 스레드 `blocking` 효과와 닮지 않았나요? 그래서 레코드 아이디 값을 매우 높게 잡으면 블락 효과를 가진다고 설명하는 것입니다. 이 덕분에 덮어쓰기 위한 레코드를 가져오는 과정을 좀 더 안전하게 처리할 수 있습니다.

```kotlin
private fun usedLocked(state: StateObject): StateRecord? {
    var current: StateRecord? = state.firstStateRecord
    var validRecord: StateRecord? = null
    val reuseLimit = pinningTable.lowestOrDefault(nextSnapshotId) - 1
    val invalid = SnapshotIdSet.EMPTY
    while (current != null) {
        val currentId = current.snapshotId
        if (currentId == INVALID_SNAPSHOT) {
            return current
        }
        if (valid(current, reuseLimit, invalid)) {
            if (validRecord == null) {
                validRecord = current
            } else {
                return if (current.snapshotId < validRecord.snapshotId) current else validRecord
            }
        }
        current = current.next
    }
    return null
}
```

이제 다시 새 레코드를 만드는 방법에 집중해 봅시다. 새 레코드는 덮어쓰기를 위한 레코드를 가져온다는 것을 의미하는데 두 가지 방법이 있습니다.

```
1. 유효하지 않은 레코드 꺼내서 재사용(재개발)
2. 유효한 레코드중 오래된(낡은(?)) 것 꺼내서 재사용(재개발)
3. 그냥 새로 레코드 하나 만들기
```

우선순위는 1번이 우선이고 1번에 해당하는 레코드가 없다면 차선책으로 2번 방식을 사용합니다. 1번과 2번 모두 해당하는 레코드가 없다면 최후의 방법으로 3번 방식을 사용합니다. 유효하지 않은 레코드를 우선으로 뽑는 이유는 재사용으로 불필요한 자원낭비를 막기 위함입니다. 

상태 레코드들 중에 오래됐거나, 덜 중요하거나, 잘 사용되지 않거나, 유효하지 않은 일종의 낙후된 레코드가 있을 수 있습니다. 반대로 중요하게 사용되는 레코드도 있을 수 있습니다. 그렇다면 어차피 잘 사용하지도 않는 조금 낡은(?) 레코드를 새로 업데이트해주는 것이 자원 소모도 막고 낙후된 레코드도 줄일 수 있습니다.

도시 재개발, 게임 캐릭터의 리메이크, 운영체제의 페이지 교체 알고리즘 등을 생각해 보면 좀 더 이해가 빠를 겁니다. 

```kotlin
if (currentId == INVALID_SNAPSHOT) {
    return current
}
```

이제는 왜 유효하지 않은 스냅샷을 최우선으로 반환하는지 알게 됐습니다. 하지만 모든 레코드가 유효하지 않은 레코드이기는 힘듭니다.

```kotlin
if (valid(current, reuseLimit, invalid)) {
    if (validRecord == null) {
        validRecord = current
    } else {
        return if (current.snapshotId < validRecord.snapshotId) current else validRecord
    }
}
```

만약 유효하지 않은 레코드가 보이지 않으면 차선책으로 유효 레코드를 우선 임시 등록합니다. 그 다음, 만약 다음 탐색 대상으로 다시 유효한 레코드가 나왔다면 이중 아이디가 더 낮은(더 오래된) 레코드를 반환합니다. 굳이 오래된 레코드를 지나치고 비교적 최신인 레코드를 업데이트할 필요는 없으니까요.

```kotlin
return (usedLocked(state) as T?)?.apply {
    snapshotId = Int.MAX_VALUE
} ?: create().apply {
    snapshotId = Int.MAX_VALUE
    this.next = state.firstStateRecord
    state.prependStateRecord(this as T)
} as T
```

하지만 이러한 노력에도 불구하고 재사용 가능한 레코드를 가져오지 못했다면 최후의 방법으로 새로 레코드를 생성합니다. 새 레코드는 가장 최신 레코드로 현재 가변 상태 객체의 첫 번째 노드(상태 레코드)가 됩니다.

```kotlin
var snapshot: Snapshot = snapshotInitializer
return sync {
    snapshot = Snapshot.current
    this.overwritableRecord(state, snapshot, candidate).block()
}.also {
    notifyWrite(snapshot, state)
}
```

이제 준비한 레코드에 요청한 값을 적용(block())하고 적용한 사실을 `notifty` 합니다. 이렇게 해서 가변 상태 객체의 값 수정 작업이 끝나게 됩니다.

​		

# 정리

가변 상태 객체 `MutableState` 는 스냅샷 기반에서 `StateObject` 로 활동합니다. 스냅샷 덕분에 단일 상태 값을 관리하는 것이 아닌 연결 리스트 형태로 다중 버전 상태 값을 관리합니다. 그래서 값을 읽을 때나 쓰기 작업을 할 때 항상 연결 리스트의 노드들을 모두 탐색하여 유효하거나 혹은 유효하지 않은 레코드를 선별하는 작업을 진행합니다.

이러한 과정을 통해 값을 읽거나 쓸 수 있는 겁니다. 그리고 쓰기 작업을 통해 값을 수정하면 알림(notify)을 전달하여 최종적으로 리컴포지션을 유도합니다.

​		

# 참조

[**Kotlin Internals**](https://leanpub.com/composeinternalskor)











