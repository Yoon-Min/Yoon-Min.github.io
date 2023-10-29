---
title: (프로그래머스 | Kotlin) - 카드 짝 맞추기
author: yoonmin
date: 2023-10-28 22:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

이 문제의 핵심은 그냥 구현, 또 구현, 그리고 또 구현이다. 막 엄청 수준이 높은 알고리즘을 요구하는 것이 아닌 기본적인 알고리즘 여러 개를 가지고 복잡한 구현을 요구한다. 그래서 이 문제를 풀면서 굉장한 피로감을 느꼈다... 나같은 경우에는 백트래킹, 너비 우선 탐색, 그리디 이 세 가지를 중심으로 구현을 진행했다. 해당 알고리즘을 사용한 이유는 다음과 같다.

#### 1. 최소 탐색을 위해서는 가능한 뒤집는 순서 조합을 모두 구해서 계산해야 한다.

만약 보드판 위에 `1` `2` `3` 이렇게 세 종류의 카드가 존재한다면 가능한 뒤집는 순서의 갯수는 6가지다. 이 6가지 순서쌍에서 최소 탐색을 할 수 있는 순서쌍이 존재하기 때문에 모든 순서쌍을 계산해서 최소값을 찾는 방식으로 진행한다.

1. `1 -> 2 -> 3`
2. `1 -> 3 -> 2`
3. `2 -> 1 -> 3`
4. `2 -> 3 -> 1`
5. `3 -> 1 -> 2`
6. `3 -> 2 -> 1`

```kotlin
  private fun setPermutation(p: ArrayList<Int>, isVisited: Array<Boolean>) {
      if(p.size == image.size) {
          permutation.add(p.toIntArray())
          return
      }

      image.forEach { imageCode ->
          if(!isVisited[imageCode]) {
              isVisited[imageCode] = true
              p.add(imageCode)
              setPermutation(p, isVisited)
              p.removeLast()
              isVisited[imageCode] = false
          }
      }
  }
```



#### 2. 같은 종류의 카드가 2개씩 있는데 이 2개의 뒤집는 순서도 고려해야 한다.

카드가 `1` `2` `3` 이렇게 있으면 한 개씩 세 장을 의미하는 게 아니라 실제로는 각 종류별로 2개씩 총 6장의 카드가 보드판에 존재한다. 따라서 각 종류별로 2장을 어떤 순서로 뒤집을 건지도 고려해야 한다. 이 부분은 예를 들어서 특정  `n`번 카드 두 장을 각각 `a` `b` 라고 가정하자. 그럼 `a -> b` `b -> a` 이렇게 두 가지 경우에서 더 작은 이동값을 가지는 경우를 택한다.

```kotlin
  private fun getMinimumMove(cursor: Pair<Int,Int>, target: Int, board:  Array<IntArray>): Point {
      val a = position[target][0]
      val b =  position[target][1]
      val moveFromAtoB = bfs(cursor, a, board) + bfs(a, b, board) + 2
      val moveFromBtoA = bfs(cursor, b, board) + bfs(b, a, board) + 2

      return when(min(moveFromAtoB, moveFromBtoA)) {
          moveFromAtoB -> Point(b.first, b.second, moveFromAtoB)
          else -> Point(a.first, a.second, moveFromBtoA)
      }
  }
```





#### 3. 보드판 위에서 특정 위치의 카드를 찾는 방법은 BFS를 이용한다.

카드를 뒤집기 위해서는 현재 위치에 있는 커서가 카드가 있는 곳으로 최소의 움직임으로 이동해야 한다. 따라서 이를 위해 BFS를 이용하여 구현했다. 대충 함수 헤더가 어떤지 설명하자면 `fun bfs(curr: Pair<Int,Int>, target: Pair<Int,Int>)` 이런 느낌이다.

```kotlin
private fun bfs(cursor: Pair<Int,Int>, target: Pair<Int,Int>, board:  Array<IntArray>): Int {
    val q = ArrayDeque<Point>()
    val col = listOf(0,1,0,-1)
    val row = listOf(1,0,-1,0)
    val isVisited = Array(4) { Array(4) {false} }

    q.add(Point(cursor.first, cursor.second, 0))
    isVisited[cursor.first][cursor.second] = true

    while(!q.isEmpty()) {
        val curr = q.first()
        q.removeFirst()

        if(curr.x == target.first && curr.y == target.second) {
            return curr.cnt
        }

        for(i in 0..3) {
            val nextCol = curr.x+ col[i]
            val nextRow = curr.y + row[i]
            if(-1 < nextCol && nextCol < 4 && -1 < nextRow && nextRow < 4 && !isVisited[nextCol][nextRow]) {
                q.add(Point(nextCol, nextRow, curr.cnt+1))
                isVisited[nextCol][nextRow] = true
            }
        }

        for(i in 0..3) {
            var nextCol = curr.x
            var nextRow = curr.y
            while(true) {
                nextCol += col[i]
                nextRow += row[i]
                if(3 < nextCol || nextCol < 0 || 3 < nextRow || nextRow < 0) {
                    nextCol -= col[i]
                    nextRow -= row[i]
                    break
                }
                if(board[nextCol][nextRow] > 0) { break }
            }
            if(isVisited[nextCol][nextRow]) { continue }
            q.add(Point(nextCol, nextRow, curr.cnt+1))
            isVisited[nextCol][nextRow] = true
        }
    }
    return 0
}
```

## 설명

#### 순서 1.

백트래킹을 이용해서 순열을 구한다. 중복 순열이 필요한 것은 아니므로 방문한 번호에 대해서는 재방문을 허용하지 않도록 `isVisited` 와 같은 논리형 변수를 만들어서 처리한다. 순열이 한쌍 완성될 때마다 만들어둔 `permutation` 배열에 저장한다.

#### 순서 2.

`n` 번 카드 2장(`a` `b` )이 있고 `a -> b` `b -> a`  를 각각 구해야 한다. 이를 `bfs` 로 구하려면 `a -> b` 를 `현재 커서 -> a ` `a -> b` 이렇게 나눠서 계산해야 하고 `b -> a` 도 마찬가지다. 그리고 두 장의 카드에 도달했을 때 뒤집는 행동도 카운트해야 하니까 `a` 뒤집을 때 한 번, `b` 뒤집을 때 한 번 해서 총 2를 더하는 것도 잊지 말아야 한다. 이렇게 계산을 끝내고 값이 더 적은 결과값을 반환하면 된다.

#### 순서 3.

보통 `bfs` 문제들은 동, 서, 남, 북 이렇게 네 방향으로 한 칸씩 움직이면서 목표 지점을 향한다. 그런데 이 문제는 컨트롤을 누른채로 움직이는 것도 있어서 이것도 추가로 고려를 해줘야 한다. 따라서 해당 부분에 대한 로직도 같이 작성한다.

```kotlin
for(i in 0..3) {
    var nextCol = curr.x
    var nextRow = curr.y
    while(true) {
        nextCol += col[i]
        nextRow += row[i]
        if(3 < nextCol || nextCol < 0 || 3 < nextRow || nextRow < 0) {
            nextCol -= col[i]
            nextRow -= row[i]
            break
        }
        if(board[nextCol][nextRow] > 0) { break }
    }
    if(isVisited[nextCol][nextRow]) { continue }
    q.add(Point(nextCol, nextRow, curr.cnt+1))
    isVisited[nextCol][nextRow] = true
}
```

## 전체 코드

```kotlin
class Solution {
    private val position = Array(7) { arrayListOf<Pair<Int,Int>>() }
    private val permutation = ArrayList<IntArray>()
    private val image = arrayListOf<Int>()

    fun solution(board: Array<IntArray>, r: Int, c: Int): Int {
        board.forEachIndexed { i, l ->
            l.forEachIndexed { j, value ->
                if (value != 0 && position[value].size == 0) { image.add(value) }
                position[value].add(Pair(i, j))
            }
        }
        var answer: Int = 4*4*2*image.size

        setPermutation(arrayListOf(), Array(7){false})
        permutation.forEach { l ->
            var cursor = Pair(r,c)
            var sum = 0
            val tmpBoard = Array(4) { IntArray(4) }
            for(i in 0..3) { for(j in 0..3) { tmpBoard[i][j] = board[i][j] } }
            l.forEachIndexed { i, imageCode ->
                val result =  getMinimumMove(cursor, l[i], tmpBoard)
                sum += result.cnt
                cursor = Pair(result.x, result.y)
                tmpBoard[position[imageCode][0].first][position[imageCode][0].second] = 0
                tmpBoard[position[imageCode][1].first][position[imageCode][1].second] = 0
            }
            answer = min(answer, sum)
        }
        return answer
    }

    private fun getMinimumMove(cursor: Pair<Int,Int>, target: Int, board:  Array<IntArray>): Point {
        val a = position[target][0]
        val b =  position[target][1]
        val moveFromAtoB = bfs(cursor, a, board) + bfs(a, b, board) + 2
        val moveFromBtoA = bfs(cursor, b, board) + bfs(b, a, board) + 2

        return when(min(moveFromAtoB, moveFromBtoA)) {
            moveFromAtoB -> Point(b.first, b.second, moveFromAtoB)
            else -> Point(a.first, a.second, moveFromBtoA)
        }
    }

    private fun bfs(cursor: Pair<Int,Int>, target: Pair<Int,Int>, board:  Array<IntArray>): Int {
        val q = ArrayDeque<Point>()
        val col = listOf(0,1,0,-1)
        val row = listOf(1,0,-1,0)
        val isVisited = Array(4) { Array(4) {false} }

        q.add(Point(cursor.first, cursor.second, 0))
        isVisited[cursor.first][cursor.second] = true

        while(!q.isEmpty()) {
            val curr = q.first()
            q.removeFirst()

            if(curr.x == target.first && curr.y == target.second) {
                return curr.cnt
            }

            for(i in 0..3) {
                val nextCol = curr.x+ col[i]
                val nextRow = curr.y + row[i]
                if(-1 < nextCol && nextCol < 4 && -1 < nextRow && nextRow < 4 && !isVisited[nextCol][nextRow]) {
                    q.add(Point(nextCol, nextRow, curr.cnt+1))
                    isVisited[nextCol][nextRow] = true
                }
            }

            for(i in 0..3) {
                var nextCol = curr.x
                var nextRow = curr.y
                while(true) {
                    nextCol += col[i]
                    nextRow += row[i]
                    if(3 < nextCol || nextCol < 0 || 3 < nextRow || nextRow < 0) {
                        nextCol -= col[i]
                        nextRow -= row[i]
                        break
                    }
                    if(board[nextCol][nextRow] > 0) { break }
                }
                if(isVisited[nextCol][nextRow]) { continue }
                q.add(Point(nextCol, nextRow, curr.cnt+1))
                isVisited[nextCol][nextRow] = true
            }
        }
        return 0
    }

    private fun setPermutation(p: ArrayList<Int>, isVisited: Array<Boolean>) {
        if(p.size == image.size) {
            permutation.add(p.toIntArray())
            return
        }

        image.forEach { imageCode ->
            if(!isVisited[imageCode]) {
                isVisited[imageCode] = true
                p.add(imageCode)
                setPermutation(p, isVisited)
                p.removeLast()
                isVisited[imageCode] = false
            }
        }
    }

    data class Point(
        val x: Int,
        val y: Int,
        val cnt: Int
    )
}
```





 