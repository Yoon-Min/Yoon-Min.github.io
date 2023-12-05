---
title: (프로그래머스 | Kotlin) - 2차원 동전 뒤집기
author: yoonmin
date: 2023-11-23 12:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

이 문제는 특정 행, 혹은 열을 뒤집어서 원하는 형태를 만드는 게 핵심이다. 어떤 행, 혹은 열을 뒤집는 순서가 중요한 것이 아니다. 어떤 행, 혹은 열의 상태를 어떻게 할 것인지(뒤집거나 그대로 두거나)가 중요하다.  `M X N`  형태에서 특정 열과 행의 상태를 결정하는 방식으로 로직을 짜면 된다.

​		

### 뒷면, 앞면의 상태값을 비트로 표현해보자

문제 마지막에 보면 `0` 은 동전의 앞면, `1` 은 동전의 뒷면을 의미한다. 그렇다면  `0` `1` 을 비트 연산으로 처리해보자. 예를 들어서 `2 X 2` 형태라고 가정해보자. 행과 열은 각각 2개고 다음과 같이 표현할 수 있다.

두 개면 모두 앞면인 경우와 모두 뒷면 경우, 한쪽은 앞면 한쪽은 뒷면인 경우가 존재한다. 이를 비트 형식으로 표현하면 다음과 같이 표현할 수 있다.  예를 들어서 행에서 `00` 은 첫 번째 행, 두 번째 행 모두 앞면인 상태를 의미한다.

```
행 ->00 01 10 11 
열 -> 00 01 10 11
```

​		

### 시프트 연산을 응용하여 모든 경우의 수(완전 탐색)를 꺼내 처리해보자

비트 `00` 은 십진수로  `0` 을 의미한다. 비트 `01` 은 `1` 이다. 비트 `10` 은 `2` 이다. 비트 `11` 은 `3` 이다. 즉, `0` 부터 `3` 까지 반복문을 통해서 나올 수 있는 동전판의 모든 경우를 꺼내서 타겟과 일치하는지 검사할 수 있다. 

```kotlin
for(i in 0 until (1 shl 2))
// 1 shl 2 => 2^2를 의미한다.
```

여기서 행과 열의 비트를 합쳐서 하나의 반복문으로 처리한다. 이렇게 하면 `2X2` 형태에서 나올 수 있는 경우의 수를 하나의 비트식으로 표현할 수 있다.

예를 들어서 `2X2` 형태에서 모든 행은 뒷면이고 모든 열은 앞면인 경우라면 `1100`, 반대로 모든 행은 앞면이고 모든 열이 뒷면이라면 `0011` 이다. 맨 앞의 두 개가 열의 상태값이고 뒤의 두 개가 행의 상태값이다.

이렇게 `0000` 부터 `1111` 까지 모든 조합을 꺼내서 원본과 비교하여 뒤집거나 그대로 두는 코드를 작성하면 된다.

```kotlin
for( i in 0 until (1 shl col+row))
// col => 행의 크기
// row => 열의 크기
// 2X2형태라면 (1 shl 4) => 2^4가 된다.

// 0000 ~ 1111 까지 루프
```

​		

### 비트 연산을 이용하여 처리하자

```kotlin
for( i in 0 until (1 shl col+row))
// 2X2크기라고 가정
```

해당 반복문을 동작시키면 `0000` 부터 `1111` 까지 나오는데 현재 `1010` 차례라고 가정해보자. 그리고 원본이 `1100` 라고 하자. 앞의 두 개는 열의 상태값을 의미하고 뒤의 두 개는 행의 상태값을 의미한다. 

행부터 살펴보면 `2^1` 부분의 값이 다르다. 반면에  `2^0` 부분은 서로 `0` 이기 때문에 뒤집을 필요가 없다. 따라서 값이 다른  `2^1` 부분은 뒤집어야 함을 의미하므로 이 부분은 뒤집어주면 된다. 

첫 번째 행의 인덱스는 `0`, 두 번째 행의 인덱스는 `1` 이다. 첫 번째 행의 상태값을 의미하는 비트 위치는 `2^0` , 두 번째 행은 `2^1` 이다. 따라서 두 번째 행을 뒤집으면 된다. 이 개념을 코드로 옮기면 다음과 같다.

{:.prompt-tip}

> `XOR` 비트 연산을 사용하면 뒤집는 동작을 쉽게 처리할 수 있다.

​		

#### 행을 처리하는 코드

```kotlin
for(c in 0 until col) {
    if(b and (1 shl c) != 0) {
        flipCounter++
        tmp[c] = tmp[c].map { it xor 1 }.toIntArray()
    }
}
```

#### 열을 처리하는 코드

```kotlin
for(r in 0 until row) {
    if(b and (1 shl r + col) != 0) {
        flipCounter++
        for(i in 0..tmp.lastIndex) { tmp[i][r] = tmp[i][r] xor 1 }
    }
}
```

{:.prompt-warning}

> 열의 경우에는 `2^2` 부터 검사를 해야 하기 때문에 `(1 shl r+col)` 로 `AND` 처리를 해줘야 한다.

​		

### 마지막은 `target` 과 비교하여 동일한지 검사한다.

```kotlin
if(tmp.contentDeepEquals(target)) { answer = min(answer, flipCounter) }
```

​		

## 전체 코드

```kotlin
import kotlin.math.*

class Solution {
    fun solution(beginning: Array<IntArray>, target: Array<IntArray>): Int {
        val col = beginning.size
        val row = beginning[0].size
        var answer = Int.MAX_VALUE

        for(b in 0 until (1 shl col+row)) {
            val tmp = Array(col) { i -> IntArray(row) { j -> beginning[i][j] } }
            var flipCounter = 0

            for(c in 0 until col) {
                if(b and (1 shl c) != 0) {
                    flipCounter++
                    tmp[c] = tmp[c].map { it xor 1 }.toIntArray()
                }
            }
            for(r in 0 until row) {
                if(b and (1 shl r + col) != 0) {
                    flipCounter++
                    for(i in 0..tmp.lastIndex) { tmp[i][r] = tmp[i][r] xor 1 }
                }
            }
            if(tmp.contentDeepEquals(target)) { answer = min(answer, flipCounter) }
        }
        return if(answer == Int.MAX_VALUE) -1 else answer
    }
    
}
```

