---
title: (프로그래머스 | Kotlin) - 무인도 여행
author: yoonmin
date: 2023-12-07 12:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

그래프 탐색을 이용해서 `X` 가 아닌 영역들을 찾아서 합산한다. `BFS` 혹은 `DFS` 를 사용하면 되는데 나는 `DFS` 를 사용해서 풀었다. `maps` 의 모든 원소를 반복문으로 돌면서 `X` 가 아니면서 방문하지 않은 곳을 시작점으로 두고 `DFS` 를 돌린다. 

그리고 `DFS` 를 돌면서 방문한 점들은 모두 방문 처리(`true`)를 한다. 전형적인 `DFS` `BFS` 문제라서 그래프 탐색 알고리즘만 작성할 수 있으면 어렵지 않게 풀 수 있는 문제다.

​		

## 전체 코드

```kotlin
class Solution {
    fun solution(maps: Array<String>): IntArray {
        val answer = ArrayList<Int>()
        val isVisited = Array(100) { BooleanArray(100) }
        for(i in 0..maps.lastIndex) {
            for(j in 0..maps[0].lastIndex) {
                if(maps[i][j] != 'X' && !isVisited[i][j]) {
                    isVisited[i][j] = true
                    answer.add(rec(Point(i, j, maps[i][j] - '0'), maps, isVisited))
                }
            }
        }
        return if(answer.isEmpty()) intArrayOf(-1) else answer.sorted().toIntArray()
    }

    private fun rec(
        s: Point,
        maps: Array<String>,
        isVisited: Array<BooleanArray>
    ): Int {
        var answer = s.days
        val dirX = listOf(0, 1, 0, -1)
        val dirY = listOf(1, 0, -1, 0)
        for(i in 0..3) {
            val nxtX = s.x + dirX[i]
            val nxtY = s.y + dirY[i]
            if(nxtX in maps.indices && nxtY in maps[0].indices && maps[nxtX][nxtY] != 'X' && !isVisited[nxtX][nxtY]) {
                isVisited[nxtX][nxtY] = true
                answer += rec(Point(nxtX, nxtY, maps[nxtX][nxtY] - '0'), maps, isVisited)
            }
        }
        return answer
    }

    data class Point(
        val x: Int,
        val y: Int,
        val days: Int
    )
}
```



