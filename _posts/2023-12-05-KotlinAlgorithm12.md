---
title: (프로그래머스 | Kotlin) - 메뉴 리뉴얼
author: yoonmin
date: 2023-12-05 12:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스, 카카오 코딩테스트]
render_with_liquid: false
---

## 해결 방법

가장 간단한 해결 방법은 `orders` 의 모든 원소들을 `course` 내의 희망 개수에 맞춰 단품메뉴 조합을 만드는 것이다. 예를 들어서 1번 예시를 보면 `orders` 의 첫 번째 원소는 `ABCFG`, `course` 의 첫 번째 원소는 `2` 다.

이를 토대로 `ABCFG` 에서 두 개로 구성된 조합을 모두 꺼내서 `map` 에 카운트를 증가시킨다. 마찬가지로 `orders` 의 나머지 원소들도 두 개로 구성된 조합을 모두 만들어서 `map` 에 카운트를 증가시킨다. 이렇게 해서 `course` 에서 `2` 에 대한 작업이 끝나면 나머지 `3` `4` 도 동일하게 처리한다.

이제 `map` 에는 `course` 에서 희망한 단품메뉴 조합 수를 토대로 제작한 조합들이 `key` 값으로, 해당 조합이 등장한 횟수를 카운트한 값이 `value` 로 존재한다. `2` 개로 구성된 조합도 많을테고, `3` 개, `4` 개의 조합 수도 많을 것이다.

이제 그중에서 등장 횟수가 가장 많은 조합만 꺼내면 된다.  `2` 개 조합에서 가장 많은 등장을 한 조합은 `AC` , `3` 개 조합에서 가장 많은 등장을 한 조합은 `CDE` , `4` 개 조합은 최대 등장 횟수가 동일한 `ACDE` `BCFG` .

​		

### 정렬된 결과값이 필요하기 때문에 `orders` 의 모든 원소들을 정렬

`result` 도 정렬되어야 하고, `result` 내 원소들도 정렬되어야 한다. 따라서 시작할 때 `orders` 의 모든 원소들을 정렬시키는 것이 좋다. 그래야 부분 조합을 구할 때도 정렬된 조합으로 만들어져서 정렬과 관련된 작업을 하지 않아도 된다.

```kotlin
orders.map { it.toSortedSet().joinToString("") }.forEach { order ->
    course.forEach { menuCnt ->
        rec(0, menuCnt, order)
    }
}
```

여기서 배열이나 조합과 같은 컬렉션의 원소들을 모두 합친 형태의 문자열로 만들고 싶으면 `jotinToString` 을 사용하자. 해당 메서드를 사용하면 컬렉션의 각 원소들을 더하기 연산으로 붙이는 작업을 할 필요가 없다.

​		

### `course` 별 등장 빈도가 가장 높은 조합을 꺼내서 `result` 에 추가

1. 원하는 조합 수에 해당되는 `key` 값들만 필터링
2. 그중에서 최대 등장값 구하기 (없으면 쓰레기값 넣기)
3. 주문 횟수가 최소 2개 이상이면서 최대 등장값인 조합 필터링

```kotlin
course.forEach { menuCnt ->
    val filtered = m.filter { it.key.length == menuCnt }
    val max = filtered.maxByOrNull { it.value }?.value ?: Int.MAX_VALUE
    answer.addAll(filtered.filter { it.value > 1 && it.value == max }.keys)
}
```

​		

### 부분 조합을 구하기 위해 재귀 함수 이용

백트래킹을 응용해서 원하는 개수의 부분 조합을 구한다. 조합이 완성될 때마다 `map` 에 해당 조합을 새로 등록하거나 카운트를 증가시킨다.

```kotlin
private fun rec(
    start: Int,
    targetLength: Int,
    order: String,
    tmp: MutableList<Char> = mutableListOf(),
) {
    if(tmp.size == targetLength) {
        with(tmp.joinToString("")) {
            if(!m.containsKey(this)) { m[this] = 0 }
            m[this] = m[this]!! + 1
        }
        return
    }
    for(i in start..order.lastIndex) {
        tmp.add(order[i])
        rec(i+1, targetLength, order, tmp)
        tmp.removeLast()
    }
}
```

​		



## 전체 코드

```kotlin
class Solution {
    private val m = mutableMapOf<String, Int>()

    fun solution(orders: Array<String>, course: IntArray): Array<String> {
        val answer = ArrayList<String>()
        orders.map { it.toSortedSet().joinToString("") }.forEach { order ->
            course.forEach { menuCnt ->
                rec(0, menuCnt, order)
            }
        }
        course.forEach { menuCnt ->
            val filtered = m.filter { it.key.length == menuCnt }
            val max = filtered.maxByOrNull { it.value }?.value ?: Int.MAX_VALUE
            answer.addAll(filtered.filter { it.value > 1 && it.value == max }.keys)
        }
        return answer.sorted().toTypedArray()
    }

    private fun rec(
        start: Int,
        targetLength: Int,
        order: String,
        tmp: MutableList<Char> = mutableListOf(),
    ) {
        if(tmp.size == targetLength) {
            with(tmp.joinToString("")) {
                if(!m.containsKey(this)) { m[this] = 0 }
                m[this] = m[this]!! + 1
            }
            return
        }
        for(i in start..order.lastIndex) {
            tmp.add(order[i])
            rec(i+1, targetLength, order, tmp)
            tmp.removeLast()
        }
    }
}
```

