---
title: (프로그래머스 | Kotlin) - 전력망을 둘로 나누기
author: yoonmin
date: 2023-11-21 12:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

특정 전선이 끊어지면 하나의 트리는 두 개의 그룹으로 나뉘게 된다. 따라서 두 개의 그룹 각각, `DFS` 혹은 `BFS` 를 이용해서 그룹 내 노드의 개수를 카운트한다. 나는 `BFS` 를 이용해서 노드의 개수를 구했다.

그리고 탐색 시작점은 제거한 전선과 연결된 노드로 지정했다. 첫 번째 입출력 예제 그림에서 `4` 와  `7` 이 연결된 전선을 끊은 경우를 보여주는데 이때 생긴 두 개의 그룹에서 탐색 시작점은 `4`,  `7` 이 된다. `4` 에서 탐색을 시작하면 6개, `7` 에서 탐색을 시작하면 3개가 카운드된다.

따라서 모든 전선에 대해서 위 방법을 적용시켜 비교해야 한다. 예제에서 전선 정보가 담긴 `wires` 을 줬기 때문에 이 배열의 각 원소(전선)에 대해서 트리를 두 개로 나누고 각 트리마다 송전탑 개수를 `BFS` 로 구한다.

​		

## 전체 코드

```kotlin
import kotlin.math.*

class Solution {
    private val node = Array(101) { arrayListOf<Int>() }

    fun solution(n: Int, wires: Array<IntArray>): Int {
        var answer = Int.MAX_VALUE
        val isChecked = Array(101) { Array(101) { false } }
        wires.forEach {
            node[it[0]].add(it[1])
            node[it[1]].add(it[0])
        }
        for(a in 1..n) {
            node[a].forEach { b ->
                if(!isChecked[a][b] && !isChecked[b][a]) {
                    answer = min(answer, abs(bfs(a, Pair(a, b)) - bfs(b, Pair(a, b))))
                    isChecked[a][b] = true
                    isChecked[b][a] = true
                }
            }
        }
        return answer
    }

    private fun bfs(s: Int, blockSet: Pair<Int, Int>): Int {
        var towerCounter = 0
        val isVisited = Array(101) { false }
        val q = ArrayDeque<Int>()
        q.add(s)
        isVisited[s] = true

        while(!q.isEmpty()) {
            val cur = q.first()
            q.removeFirst()
            towerCounter++

            node[cur].forEach { n ->
                if(!isVisited[n] && blockSet != Pair(cur, n) && blockSet != Pair(n, cur) ) {
                    isVisited[n] = true
                    q.add(n)
                }
            }
        }
        return towerCounter
    }
    
}
```

