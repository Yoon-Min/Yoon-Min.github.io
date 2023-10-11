---
title: (프로그래머스 | Kotlin) - 억억단을 외우자
author: yoonmin
date: 2023-10-10 16:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

<span style="color: #30aaa0">**이 문제의 핵심은 약수의 개수를 구하는 데 있다.**</span>  특정 숫자의 약수 개수를 알아야 등장 빈도를 알 수 있기 때문이다. 예를 들어서 숫자 `4` 를 예시로 들어보겠다. 억억단 예시 사진을 보면 해당 숫자가 세 번 등장하는데 그 이유가 구구단, 1단부터 `e` 단까지를 행렬로 표현해서 `1X4` `4X1` `2X2` 이렇게 나오기 때문이다. 

그럼 여기서 대충 약수와 관련이 있다는 것을 눈치챌 수 있다. `4` 의 약수는 총 세 개, `1, 2, 4` 이다. 가운데를 기준으로 특정 위치에 있는 숫자는 자신의 반대쪽에 있는 숫자와 곱을 하면 `4` 가 된다. 그래서 원래는 `4` 가 두 개가 나오는 것이 맞지만 억억단은 행과 열 둘 다 구구단이 존재하기 때문에 동일 숫자 곱(`2X2`)을 제외하고 각각 2배를 해줘야 한다. 따라서  `1`, `4` 는 `1X4` `4X1` 이렇게 두 가지가 된다.

이런 특징을 이용해서 숫자  `e` 까지의 등장 빈도를 배열에 저장하고 이 배열을 이용해 `result` 를 구하면 되는데 문제는 <span style="color: #30aaa0">**등장 빈도를 구하는 속도가 빨라야 한다.**</span> 나같은 경우에는 처음에  제곱근을 이용한 약수 개수를 구하는 알고리즘을 이용해서 시도했었는데 시간 초과가 됐었다. 그래서 <span style="color: #30aaa0">**배수를 이용한 약수 개수 구하는 알고리즘**</span>으로 변경하여 제출했더니 통과가 됐다. 따라서 이 문제는 약수 개수를 구하는 알고리즘만 잘 짜면 그 뒤는 무난하게 풀 수 있다. 해결 순서는 다음과 같다.

1. `e` 까지 각 숫자의 등장 빈도를 배열에 저장한다.

   ```kotlin
   for (i in 1..e) {
       for (j in 1..e / i) {
           counter[i*j]++
       }
   }
   ```

   

2. `starts` 배열 내 원소들을 하나씩 방문하여 `result` 를 구한다.

   `getResult` 는 `i` ~ `j` 사이에서 빈도가 가장 높은 수와 빈도 값을 배열에 담아 리턴하는 메서드이다.

   ```kotlin
   private fun getResult(i: Int, j: Int, counter: Array<Int>): IntArray {
       var maxCnt = 0
       var numOfMaxCnt = 0
       for(num in i .. j) {
           if(counter[num] > maxCnt) {
               maxCnt = counter[num]
               numOfMaxCnt = num
           }
       }
       return intArrayOf(numOfMaxCnt, maxCnt)
   }
   ```

   ```kotlin
   var prev = getResult(sortedStarts.last()[0], e, counter)
   answer[sortedStarts.last()[1]] = prev[0]
   
   for(i in sortedStarts.lastIndex-1 downTo 0) {
       val curr = getResult(sortedStarts[i][0], sortedStarts[i + 1][0], counter)
       answer[sortedStarts[i][1]] = if (curr[1] >= prev[1]) curr[0] else prev[0]
   
       if(curr[1] > prev[1]) {
           answer[sortedStarts[i][1]] = curr[0]
           prev = curr
       }
       else if(curr[1] == prev[1]) {
           answer[sortedStarts[i][1]] = min(curr[0], prev[0])
           prev = if(curr[0] < prev[0]) curr else prev
       }
       else {
           answer[sortedStarts[i][1]] = prev[0]
       }
   }
   answer.toIntArray()
   ```

   

   ## 설명

   #### 순서 1.

   이 부분은 위에서 설명했듯이 배수를 이용한 약수 개수 구하는 알고리즘을 사용해서 `e` 까지 빈도를 배열에 저장하면 된다.

   #### 순서 2.

   이 부분은 풀이 방법이 사람마다 다를 수 있는데 나같은 경우에는 `starts` 를 정렬해놓고 내림차순으로 각 원소의 `result` 를 구하는 방법을 사용했다. 예를 들어서 문제 예시에서 준 `starts` 는 `[1, 3, 7]` 인데 이를 먼저 정렬한다. (근데 이미 정렬이 되어 있으므로 패스) 그리고  `e` 는 `8` 이다.

   이제 `starts` 를 내림차순으로 반복문을 돌려서 `7~8` -> `3~8` -> `1~8` 이런 식으로 `result` 를 구한다. 그런데 `7~8` 을 먼저 계산을 하면 `3~8` 을 계산할 때 `3~7` 까지만 계산하면 된다. `7~8` 부분은 이미 앞서 계산을 끝마쳤기 때문이다. 서로 두 결과값을 비교해서 빈도가 더 많은쪽을 설정해주면 된다. 만약 빈도가 서로 같다면 문제에서 내건 조건대로 숫자가 더 작은쪽을 설정해주면 된다. 이런 방식으로 계산을 하면은 마지막인 `1~8` 계산은 `1~3` 까지만 계산하고 앞서 계산한 `3~8` 값과 비교해서 값을 업데이트하면 된다.

   

   ## 전체 코드

   ```kotlin
   import kotlin.math.*
   
   class Solution {
       fun solution(e: Int, starts: IntArray): IntArray {
           val answer = Array(starts.size) {0}
           val counter = Array(e+1) {0}
           val sortedStarts = Array(starts.size) { Array(2) {0} }
   
   
           for((i, num) in starts.withIndex()) {
               sortedStarts[i][0] = num
               sortedStarts[i][1] = i
           }
           sortedStarts.sortBy {it[0]}
           
           for (i in 1..e) {
               for (j in 1..e / i) {
                   counter[i*j]++
               }
           }
   
           return if(starts.size < 2) {
               intArrayOf(getResult(starts[0], e, counter)[0])
           } else {
               var prev = getResult(sortedStarts.last()[0], e, counter)
               answer[sortedStarts.last()[1]] = prev[0]
               
               for(i in sortedStarts.lastIndex-1 downTo 0) {
                   val curr = getResult(sortedStarts[i][0], sortedStarts[i + 1][0], counter)
                   answer[sortedStarts[i][1]] = if (curr[1] >= prev[1]) curr[0] else prev[0]
   
                   if(curr[1] > prev[1]) {
                       answer[sortedStarts[i][1]] = curr[0]
                       prev = curr
                   }
                   else if(curr[1] == prev[1]) {
                       answer[sortedStarts[i][1]] = min(curr[0], prev[0])
                       prev = if(curr[0] < prev[0]) curr else prev
                   }
                   else {
                       answer[sortedStarts[i][1]] = prev[0]
                   }
               }
               answer.toIntArray()
           }
   
       }
   
       private fun getResult(i: Int, j: Int, counter: Array<Int>): IntArray {
           var maxCnt = 0
           var numOfMaxCnt = 0
           for(num in i .. j) {
               if(counter[num] > maxCnt) {
                   maxCnt = counter[num]
                   numOfMaxCnt = num
               }
           }
           return intArrayOf(numOfMaxCnt, maxCnt)
       }
   }
   ```

   

   

   