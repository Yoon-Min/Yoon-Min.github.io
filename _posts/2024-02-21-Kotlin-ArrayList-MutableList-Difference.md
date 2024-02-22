---
title: Kotlin - ArrayList와 MutableList, 무엇을 써야 할까?
author: yoonmin
date: 2024-02-20 12:00:00 +0900
categories: [CS, 프로그래밍 언어]
tags: [Kotlin, Collection, ArrayList, MutableList]
render_with_liquid: false
---

![tobias-keller-HzSLh5zV42s-unsplash 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/067632ff-649a-49e4-b204-c58b95d6d4a7)

## Collection

### Kotlin Collection

> "The Kotlin Standard Library provides a comprehensive set of tools for managing *collections* – groups of a variable number of items (possibly zero) that are significant to the problem being solved and are commonly operated on."
>
> **Kotlin docs -**

다양한 프로그래밍 언어에서 문제 해결을 위해 동일하거나 비슷한 의미를 가진 `item` 들이 하나의 그룹으로 제어된 방식으로 함께 동작하는 `Collection` 을 사용합니다. 코틀린에서 컬렉션은 4개의 메서드와  `Iterable` 의 `iterator` 을 상속받은 구조입니다.

```kotlin
/**
 * A generic collection of elements. Methods in this interface support only read-only access to the collection;
 * read/write access is supported through the [MutableCollection] interface.
 * @param E the type of elements contained in the collection. The collection is covariant in its element type.
 */
public interface Collection<out E> : Iterable<E> {
    public val size: Int
    public fun isEmpty(): Boolean
    public operator fun contains(element: @UnsafeVariance E): Boolean
    override fun iterator(): Iterator<E>
    public fun containsAll(elements: Collection<@UnsafeVariance E>): Boolean
}
```

​		

### Collection 기반의 List

![image](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/9e4b0f3f-6231-4fa0-8f20-9e9d6c020ce1)

​		

> [`List`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/index.html) stores elements in a specified order and provides indexed access to them.
>
> **Kotlin docs-**

`List` 는 지정된 순서로 요소를 저장하고 인덱스를 통한 요소 접근을 제공하는 `Collection` 기반의 인터페이스입니다. `Collection` 자체가 읽기만 가능(`read-only`)하기 때문에 이를 상속받은 `List` 도 `read-only` 입니다. 메서드 구성을 보면 읽기 관련만 있는 것을 알 수 있습니다.

```kotlin
/**
 * A generic ordered collection of elements. Methods in this interface support only read-only access to the list;
 * read/write access is supported through the [MutableList] interface.
 * @param E the type of elements contained in the list. The list is covariant in its element type.
 */
public interface List<out E> : Collection<E> {
    override val size: Int
    override fun isEmpty(): Boolean
    override fun contains(element: @UnsafeVariance E): Boolean
    override fun iterator(): Iterator<E>
    override fun containsAll(elements: Collection<@UnsafeVariance E>): Boolean
  
    public operator fun get(index: Int): E
    public fun indexOf(element: @UnsafeVariance E): Int
    public fun lastIndexOf(element: @UnsafeVariance E): Int
    public fun listIterator(): ListIterator<E>
    public fun listIterator(index: Int): ListIterator<E>
    public fun subList(fromIndex: Int, toIndex: Int): List<E>
}
```

### 읽기와 쓰기 작업이 모두 가능한 List - MutableList

쓰기 작업도 가능한 컬렉션을 만들기 위해서 코틀린은 `Collection` 이외에도 `MutableCollection` 도 제공합니다. 그리고 `List` 도 요소에 읽기뿐만 아니라 쓰기 작업도 가능한 `MutableList` 가 존재합니다. 이는 `MutableCollection` , `List` 인터페이스 기반입니다.

```kotlin
public interface MutableList<E> : List<E>, MutableCollection<E> {
    // Modification Operations
    /**
     * Adds the specified element to the end of this list.
     *
     * @return `true` because the list is always modified as the result of this operation.
     */
    override fun add(element: E): Boolean

    override fun remove(element: E): Boolean

    // Bulk Modification Operations
    /**
     * Adds all of the elements of the specified collection to the end of this list.
     *
     * The elements are appended in the order they appear in the [elements] collection.
     *
     * @return `true` if the list was changed as the result of the operation.
     */
    override fun addAll(elements: Collection<E>): Boolean

    /**
     * Inserts all of the elements of the specified collection [elements] into this list at the specified [index].
     *
     * @return `true` if the list was changed as the result of the operation.
     */
    public fun addAll(index: Int, elements: Collection<E>): Boolean

    override fun removeAll(elements: Collection<E>): Boolean
    override fun retainAll(elements: Collection<E>): Boolean
    override fun clear(): Unit

    // Positional Access Operations
    /**
     * Replaces the element at the specified position in this list with the specified element.
     *
     * @return the element previously at the specified position.
     */
    public operator fun set(index: Int, element: E): E

    /**
     * Inserts an element into the list at the specified [index].
     */
    public fun add(index: Int, element: E): Unit

    /**
     * Removes an element at the specified [index] from the list.
     *
     * @return the element that has been removed.
     */
    public fun removeAt(index: Int): E

    // List Iterators
    override fun listIterator(): MutableListIterator<E>

    override fun listIterator(index: Int): MutableListIterator<E>

    // View
    override fun subList(fromIndex: Int, toIndex: Int): MutableList<E>
}
```

### MutableList의 구현체 - ArrayList

`ArrayList` 는 `MutableList` 인터페이스의 구현체입니다. 즉, `interface` 가 아닌 `class` 입니다. `MutableList` 의 성질을 가지고 있기 때문에 요소의 읽기, 쓰기가 모두 가능하고 동적 배열로 동작합니다.

```kotlin
expect class ArrayList<E> : MutableList<E>, RandomAccess {
    constructor()
    constructor(initialCapacity: Int)
    constructor(elements: Collection<E>)

    fun trimToSize()
    fun ensureCapacity(minCapacity: Int)

    // From List

    override val size: Int
    override fun isEmpty(): Boolean
    override fun contains(element: @UnsafeVariance E): Boolean
    override fun containsAll(elements: Collection<@UnsafeVariance E>): Boolean
    override operator fun get(index: Int): E
    override fun indexOf(element: @UnsafeVariance E): Int
    override fun lastIndexOf(element: @UnsafeVariance E): Int

    // From MutableCollection

    override fun iterator(): MutableIterator<E>

    // From MutableList

    override fun add(element: E): Boolean
    override fun remove(element: E): Boolean
    override fun addAll(elements: Collection<E>): Boolean
    override fun addAll(index: Int, elements: Collection<E>): Boolean
    override fun removeAll(elements: Collection<E>): Boolean
    override fun retainAll(elements: Collection<E>): Boolean
    override fun clear()
    override operator fun set(index: Int, element: E): E
    override fun add(index: Int, element: E)
    override fun removeAt(index: Int): E
    override fun listIterator(): MutableListIterator<E>
    override fun listIterator(index: Int): MutableListIterator<E>
    override fun subList(fromIndex: Int, toIndex: Int): MutableList<E>
}
```

`ArrayList` 에 대한 `actual` 클래스를 보면 내부적으로 백업 저장소 역할을 하는 `backingArray: Array<E>` 를 이용해서 멤버 메서드 동작을 처리하는 것을 알 수 있습니다.

``` kotlin
/*
 * Copyright 2010-2023 JetBrains s.r.o. Use of this source code is governed by the Apache 2.0 license
 * that can be found in the LICENSE file.
 */

package kotlin.collections

actual class ArrayList<E> private constructor(
    private var backingArray: Array<E>,
    private var offset: Int,
    private var length: Int,
    private var isReadOnly: Boolean,
    private val backingList: ArrayList<E>?,
    private val root: ArrayList<E>?
) : MutableList<E>, RandomAccess, AbstractMutableList<E>() {
    private companion object {
        private val Empty = ArrayList<Nothing>(0).also { it.isReadOnly = true }
    }

    init {
        if (backingList != null) this.modCount = backingList.modCount
    }

    actual constructor() : this(10)

    actual constructor(initialCapacity: Int) : this(
            arrayOfUninitializedElements(initialCapacity), 0, 0, false, null, null)

    actual constructor(elements: Collection<E>) : this(elements.size) {
        addAll(elements)
    }

    @PublishedApi
    internal fun build(): List<E> {
        if (backingList != null) throw IllegalStateException() // just in case somebody casts subList to ArrayList
        checkIsMutable()
        isReadOnly = true
        return if (length > 0) this else Empty
    }

    override actual val size: Int
        get() {
            checkForComodification()
            return length
        }

    override actual fun isEmpty(): Boolean {
        checkForComodification()
        return length == 0
    }

    override actual fun get(index: Int): E {
        checkForComodification()
        AbstractList.checkElementIndex(index, length)
        return backingArray[offset + index]
    }

    override actual operator fun set(index: Int, element: E): E {
        checkIsMutable()
        checkForComodification()
        AbstractList.checkElementIndex(index, length)
        val old = backingArray[offset + index]
        backingArray[offset + index] = element
        return old
    }

    override actual fun indexOf(element: E): Int {
        checkForComodification()
        var i = 0
        while (i < length) {
            if (backingArray[offset + i] == element) return i
            i++
        }
        return -1
    }

    override actual fun lastIndexOf(element: E): Int {
        checkForComodification()
        var i = length - 1
        while (i >= 0) {
            if (backingArray[offset + i] == element) return i
            i--
        }
        return -1
    }

    override actual fun iterator(): MutableIterator<E> = listIterator(0)
    override actual fun listIterator(): MutableListIterator<E> = listIterator(0)

    override actual fun listIterator(index: Int): MutableListIterator<E> {
        checkForComodification()
        AbstractList.checkPositionIndex(index, length)
        return Itr(this, index)
    }

    override actual fun add(element: E): Boolean {
        checkIsMutable()
        checkForComodification()
        addAtInternal(offset + length, element)
        return true
    }

    override actual fun add(index: Int, element: E) {
        checkIsMutable()
        checkForComodification()
        AbstractList.checkPositionIndex(index, length)
        addAtInternal(offset + index, element)
    }

    override actual fun addAll(elements: Collection<E>): Boolean {
        checkIsMutable()
        checkForComodification()
        val n = elements.size
        addAllInternal(offset + length, elements, n)
        return n > 0
    }

    override actual fun addAll(index: Int, elements: Collection<E>): Boolean {
        checkIsMutable()
        checkForComodification()
        AbstractList.checkPositionIndex(index, length)
        val n = elements.size
        addAllInternal(offset + index, elements, n)
        return n > 0
    }

    override actual fun clear() {
        checkIsMutable()
        checkForComodification()
        removeRangeInternal(offset, length)
    }

    override actual fun removeAt(index: Int): E {
        checkIsMutable()
        checkForComodification()
        AbstractList.checkElementIndex(index, length)
        return removeAtInternal(offset + index)
    }

    override actual fun remove(element: E): Boolean {
        checkIsMutable()
        checkForComodification()
        val i = indexOf(element)
        if (i >= 0) removeAt(i)
        return i >= 0
    }

    override actual fun removeAll(elements: Collection<E>): Boolean {
        checkIsMutable()
        checkForComodification()
        return retainOrRemoveAllInternal(offset, length, elements, false) > 0
    }

    override actual fun retainAll(elements: Collection<E>): Boolean {
        checkIsMutable()
        checkForComodification()
        return retainOrRemoveAllInternal(offset, length, elements, true) > 0
    }

    override actual fun subList(fromIndex: Int, toIndex: Int): MutableList<E> {
        AbstractList.checkRangeIndexes(fromIndex, toIndex, length)
        return ArrayList(backingArray, offset + fromIndex, toIndex - fromIndex, isReadOnly, this, root ?: this)
    }

    actual fun trimToSize() {
        if (backingList != null) throw IllegalStateException() // just in case somebody casts subList to ArrayList
        registerModification()
        if (length < backingArray.size)
            backingArray = backingArray.copyOfUninitializedElements(length)
    }

    final actual fun ensureCapacity(minCapacity: Int) {
        if (backingList != null) throw IllegalStateException() // just in case somebody casts subList to ArrayList
        if (minCapacity <= backingArray.size) return
        registerModification()
        ensureCapacityInternal(minCapacity)
    }
  
  	/* ... */
}
```

특히 `add` 함수 구현 내용을 보면 요소 추가가 가능한지 먼저 체크를 한 후에 요소 추가 작업을 진행하는데 배열 용량이 부족하면 용량을 늘리는 것을 보아 동적 배열인 것을 확인할 수 있습니다.

```kotlin
private fun insertAtInternal(i: Int, n: Int) {
    ensureExtraCapacity(n)
    backingArray.copyInto(backingArray, startIndex = i, endIndex = offset + length, destinationOffset = i + n)
    length += n
}
```



## ArrayList VS MutableList

### 객체 생성시 내부 동작

코틀린에서 `ArrayList` 와 `MutableList` 둘 다 객체를 생성하는 방식은 `ArrayList` 클래스를 이용합니다. 생성 방식에 대한 코드는 코틀린 `Collections.kt` 에서 확인할 수 있습니다.

`MutableList` 을 생성할 때, 내부에서 `ArrayList` 객체를 생성하여 반환하는 것을 알 수 있습니다. `ArrayList` 생성은 당연히 `ArrayList` 객체를 이용합니다.

```kotlin
@SinceKotlin("1.1")
@kotlin.internal.InlineOnly
public inline fun <T> mutableListOf(): MutableList<T> = ArrayList()

@SinceKotlin("1.1")
@kotlin.internal.InlineOnly
public inline fun <T> MutableList(size: Int, init: (index: Int) -> T): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
}

@SinceKotlin("1.1")
@kotlin.internal.InlineOnly
public inline fun <T> arrayListOf(): ArrayList<T> = ArrayList()
```

이를 통해 알 수 있는 것은 결국 `ArrayList` 와 `MutableList` 는 이름만 다를 뿐이지 내부에서는 둘 다 `ArrayList` 을 이용한다는 것입니다. 그래서 이 부분만 봐서는 둘의 차이를 느끼기 힘들기 때문에 다른 부분을 봐야 합니다.

### 사용 목적에 따른 분류

객체 생성은 둘 다 `ArrayLIst` 인스턴스를 생성하므로 동작은 동일합니다. 그렇다면 개발 철학적인 관점에서 사용 이유를 생각해 봐야 합니다.

인터페이스인 `MutableList` 는 객체의 <span style="color: #30aaa0">**동작**</span>만 보면 됩니다. 어떻게 동작하는지는 알 필요가 없습니다. <span style="color: #30aaa0">**그냥 인터페이스를 통해 원하는 리스트 동작만 처리해주면 그만**</span>입니다. 

반면에 `ArrayList` 는 `MutableList` 를 구현한 클래스입니다. 여기서는 각 동작에 대한 설계 <span style="color: #30aaa0">**구조**</span>를 파악할 수 있고 직접적인 컨트롤이 가능합니다. 따라서 인터페이스처럼 <span style="color: #30aaa0">**단순 동작에 집중하는 것이 아닌 동작의 원리(구조)에 집중**</span>하게 됩니다. `ArrayList` 의 독자적인 메서드 두 개만 봐도 객체의 메모리 관리에 직접 관여가 가능한 것을 알 수 있습니다.

```kotlin
/*
*ArrayList 의 capacity 을 리스트의 현재 사이즈만큼 줄이는 최소화 작업을 하여 메모리 관리를 합니다.
*/
fun trimToSize()

/*
*필요한 경우 ArrayList 의 용량을 늘려 minCapacity 크기의 요소 수 이상을 유지할 수 있도록 합니다. 만약 공간이 충분할 경우, 추가 작업 없이 return 합니다.
*/
fun ensureCapacity(minCapacity: Int)
```

## 결론

정리하면 내부 동작에는 관심이 없고 단순히 리스트의 기능이 필요하다면 `MutableList` ,  `ArrayList` 의 동작 구조를 파악하여 문제 해결에 적합하다고 판단이 되면 `ArrayList` 의 사용을 추천하고 있습니다.

그리고 `MutableList` 의 구현체가 지금은 `ArrayList` 이지만 미래에 이보다 더 나은 다양한 구현 클래스가 제공될 수 있으므로 인터페이스 리스트를 이용하는 것이 미래 변동 사항에 유연하게 대처할 수 있다는 의견도 있습니다. 결국 동작 수행은 똑같으므로 본인의 개발 가치관에 따라서 자유롭게 선택하시면 될 것 같습니다.

​		

> The only difference between the two is communicating your intent.
>
> When you write `val a = mutableListOf()`, you're saying "I want a mutable list, and I don't particularly care about the implementation". When you write, instead, `val a = ArrayList()`, you're saying "I specifically want an `ArrayList`".
>
> In practice, in the current implementation of Kotlin compiling to the JVM, calling `mutableListOf` will produce an `ArrayList`, and there's no difference in behaviour: once the list is built, everything will behave the same.
>
> Now, let's say that a future version of Kotlin changes `mutableListOf` to return a different type of list.
>
> Likelier than not, the Kotlin team would only make that change if they figure the new implementation works better for most use cases. `mutableListOf` would then have you using that new list implementation transparently, and you'd get that better behaviour for free. Go with `mutableListOf` if that sounds like your case.
>
> On the other hand, maybe you spent a bunch of time thinking about your problem, and figured that `ArrayList` *really* is the best fit for your problem, and you don't want to risk getting moved to something sub-optimal. Then you probably want to either use `ArrayList` directly, or use the `arrayListOf` factory function (an `ArrayList`-specific analogue to `mutableListOf`).
>
> **stackoverflow -** [Difference between ArrayList() and mutableListOf() in Kotlin](https://stackoverflow.com/questions/43114367/difference-between-arrayliststring-and-mutablelistofstring-in-kotlin)

​		

> Under the covers, both mutableListOf() and arrayListOf() create an instance of ArrayList. ArrayList is a class that happens to implement the MutableList interface.
>
> The only difference is that arrayListOf() returns the ArrayList as an actual ArrayList. mutableListOf() returns a MutableList, so the actual ArrayList is "disguised" as just the parts that are described by the MutableList interface.
>
> The difference, in practice, is that ArrayList has a few methods that are not part of the MutableList interface (trimToSize and ensureCapacity).
>
> The difference, philosophically, is that the MutableList only cares about the behaviour of the object being returned. It just returns "something that acts like a MutableList". The ArrayList cares about the "structure" of the object. It allows you to directly manipulate the memory allocated by the object (trimToSize).
>
> The rule of thumb is that you should prefer the interface version of things (mutableListOf()) unless you actually have a reason to care about the exact details of the underlying structure.
>
> Or, in other words, if you don't know which one you want, choose mutableListOf first.
>
> **stackoverflow -** [Difference between ArrayList() and mutableListOf() in Kotlin](https://stackoverflow.com/questions/43114367/difference-between-arrayliststring-and-mutablelistofstring-in-kotlin)

## 참조

[**stackoverflow - Difference between ArrayList() and mutableListOf() in Kotlin**](https://stackoverflow.com/questions/43114367/difference-between-arrayliststring-and-mutablelistofstring-in-kotlin)

[**Kotlin docs - ArrayList**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-array-list/#arraylist)

[**Kotlin docs - MutableList**](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/)

[**Kotlin docs - Collection**](https://kotlinlang.org/docs/collections-overview.html)
