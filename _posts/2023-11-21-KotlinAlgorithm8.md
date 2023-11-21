---
title: (프로그래머스 | Kotlin) - 호텔 대실
author: yoonmin
date: 2023-11-21 12:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

<span style="color: #30aaa0">**이 문제의 핵심은 그리디 알고리즘을 이용하는 것**</span>이다. 우선 문제에서 예약 시작 시간과 종료 시간이 여러 개 담긴 배열을 줬으므로 해당 <span style="color: #30aaa0">**예약 시작 시간을 기준으로 배열을 정렬**</span>시킨다. 그리고 정렬된 배열을 가지고 반복문을 돌려서 현재 원소의 예약 시작 시간보다 10분 더 적은 종료 시간이 있는 방 번호를 찾는다.

예를 들어서 현재 예약이 잡힌 방의 갯수를 `n` 개라고 가정하자. 그럼 방의 번호는 현재 `1` 번부터 `n` 번까지 존재한다. 정렬된 배열의 반복문을 돌렸을 때 가장 좋은 예약 방법은 기존에 존재하는 방에 예약을 거는 것이다. 그렇게 하기 위해선 현존하는 방들 중에서 마지막으로 예약된 시간대의 종료 시간이 현재 원소의 시작 시간보다 10분 더 적어야 한다.

그래서 모든 방을 탐색해서 예약이 가능한 방을 찾으면 해당 방의 예약 현황을 업데이트(마지막 예약의 종료 시간)한다. 업데이트가 완료되면 다음 원소로 넘어가서 이전과 똑같은 작업을 한다.

만약 예약 가능한 방이 존재하지 않는다면 현존하는 방으로는 예약을 할 수 없다는 뜻이므로 이때는 방을 새로 생성한다. 현존하는 방의 갯수가 `n` 개라면 이때 방을 새로 생성해서 `n + 1` 개가 된다.

​		

## 설명

### 왜 종료 시간이 아닌 시작 시간을 기준으로 배열을 정렬시켜야 하는가?

종료 시간을 기준으로 정렬을 했다 가정해보자. 다음 예시는 임의로 만들어본 예시다. 해당 그림은 예약 종료 시간을 기준으로 위에서 아래로 오름차순 정렬을 한 상태를 나타낸다.

![Group 7](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/ea8d0140-0af2-4790-8e45-e38c455e5da9)

여기서 설명했던 해결 방법대로 예약 시간들을 배치하면 다음과 같다. 해당 결과로 방이 세 개 생성된다. 뭔가 이상하지 않은가? 뭔가 듬성듬성 배치되어 방이 세 개나 사용됐다. 

![Group 7](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/843c0785-088c-48d8-a521-a63d32d4c4ea)

`12:14 ~ 12:34` 시간대를 첫 번째 방이 아닌 두 번째 방에 배치하고 `11:20 ~ 14:20` 시간대를 첫 번째 방에 배치하면 방을 세 개로 사용하지 않고 두 개만 가지고 모든 예약 처리를 할 수 있다. 이런 반례가 있어서 종료 시간을 기준으로 정렬하는 것은 옳지 않다.

![Group 7](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/bb566e1e-c49a-4df6-a8f4-62ca74defb7c)

​		

## 전체 코드

```kotlin
class Solution {
    fun solution(bookTime: Array<Array<String>>): Int {
        val bookTimeMinute = bookTime
            .map { arrayOf(it[0].toMinute(), it[1].toMinute()) }
            .sortedBy { it[0] }

        val availableTime = ArrayList<Int>()
        for(t in bookTimeMinute) {
            var renewedRoom: Int? = null
            for((roomNumber, lastTime) in availableTime.withIndex()) {
                if(lastTime <= t[0]) {
                    availableTime[roomNumber] = t[1]+10
                    renewedRoom = roomNumber
                    break
                }
            }
            renewedRoom ?: availableTime.add(t[1]+10)
        }
        return availableTime.size
    }

    private fun String.toMinute(): Int {
        val sum: Int
        this
            .split(':')
            .map { it.toInt() }
            .let { sum = it[0] * 60 + it[1] }
        return sum
    }
}
```

