---
title: (프로그래머스 | Kotlin) - 이모티콘 할인행사
author: yoonmin
date: 2023-10-11 20:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결방법

<span style="color: #30aaa0">**이 문제의 핵심은 `emoticons` 배열에 담겨 있는 이모티콘들의 할인율을 배정하는 것이다.**</span> 각 이모티콘은 각각 10, 20, 30, 40 퍼센트로 네 가지 할인율을 가질 수 있다. 따라서 각 이모티콘은 네 가지 중에서 하나의 할인율을 가진다. `emoticons` 의 최대 길이가 `7` 이므로 나올 수 있는 이모티콘들의 할인율의 경우의 수는 최대 `4^7` 개다.

나올 수 있는 최대 경우의 수가 4의 7승이라면 모든 경우를 탐색해도 지장이 없는 크기다. 따라서 <span style="color: #30aaa0">**완전 탐색을 통해 문제에서 요구하는 최대 효율을 가진 경우의 수(최대 효율을 내는 할인율 조합)를 찾을 수 있다.**</span> 완전 탐색을 위해서 모든 경우의 수를 구해야 하므로 백트래킹을 이용하여 나올 수 있는 모든 할인율 조합을 배열에 저장한다.

모든 할인율 조합을 구했다면 하나씩 꺼내서 `users` 를 통해 결과값을 도출해본다. 결과값은 문제 조건대로 이모티콘 구독자 수가 많은 게 우선이고 그 다음은 이모티콘 판매액이다. 따라서 현재 계산 시점에서 나온 결과값과 이전에 계산했던 결과값 중에서 최대 효율을 내는 값과 비교하여 둘 중에 더 좋은 효율을 가진 결과값으로 업데이트하는 방식으로 해결한다. 해결 순서는 다음과 같다.

1. 주어진 이모티콘 배열의 사이즈에 맞는 구할 수 있는 모든 할인율 조합을 구한다.

   `max` 는 주어진 이모티콘 배열의 사이즈를 의미한다.

   ```kotlin
   private fun setRatePermutation(count: Int){
       if(count == max){
           ratePermutation.add(tempPermutation.copyOf().toIntArray())
           return
       }
       for(i in rate.indices){
           tempPermutation[count] = rate[i]
           setRatePermutation(count + 1)
       }
   }
   ```

   `main` 함수에서 먼저 할인율 조합을 저장한다.

   ```kotlin
   this.max = emoticons.size
   setRatePermutation(0)
   ```

2. 할인율 조합을 반복문으로 돌려 각 할인율 조합에 대한 `users` 배열을 계산해서 `result` 를 갱신한다.

   ```kotlin
   for(p in ratePermutation) {
       var selling = 0
       var subscriberCounter = 0
       for(user in users) {
           var boughtSum = 0
           for((i,price) in emoticons.withIndex()) {
               if(p[i] >= user[0]) {
                   boughtSum += ((100-p[i])*0.01*price).toInt()
               }
           }
           if(boughtSum >= user[1]) { subscriberCounter++ }
           else{ selling += boughtSum }
       }
       if(answer[0] < subscriberCounter || (answer[0]==subscriberCounter) && answer[1] < selling) {
           answer[0] = subscriberCounter
           answer[1] = selling
       }
   }
   ```



## 설명

#### 순서 1.

백트래킹을 이용한다. 여기서 할인율은 각 이모티콘마다 서로 다른 값으로 배정돼야 하는 것이 아니므로 중복값을 허용하도록 설계해야 한다. 이모티콘이 4개 있으면 할인율이  `10, 20, 30, 40` 일수도 있고, `10, 10, 10, 10` 일수도 있다. 이렇게 구한 경우의 수가 최대(`emoticons` 사이즈가 7일 때) 약 16000 ~ 17000개다. 굉장히 값이 작아서 충분히 완전 탐색할 수 있는 크기다.

#### 순서 2.

`users` 배열에 대한 계산은 그냥 문제에서 하라는 것만 하면 문제없이 결과값을 구할 수 있다. 결과값이 나왔다면 현재 최대값(1. 최대한 많은 이모티콘 구독자 수 2. 높은 판매액)과 비교해서 최대값을 갱신하면 된다. 최대값을 새로 갱신해야 하는 경우는 다음과 같다.

{: .prompt-tip }

> 1. 결과값(`[이모티콘 구독자 수, 이모티콘 판매액]`)의 이모티콘 구독자 수가 현재 최대값의 이모티콘 구독자 수보다 많을 경우
> 2. 이모티콘 구독자 수가 결과값과 현재 최대값이 서로 같은데 판매액의 결과값이 더 많은 경우

## 전체 코드

```kotlin
class Solution {
    private var max = 7
    private val tempPermutation = Array(max) {0}
    private val ratePermutation = ArrayList<IntArray>()
    private val rate = listOf(10, 20, 30, 40)

    fun solution(users: Array<IntArray>, emoticons: IntArray): IntArray {
        val answer = Array(2) {0}
        this.max = emoticons.size
        setRatePermutation(0)

        for(p in ratePermutation) {
            var selling = 0
            var subscriberCounter = 0
            for(user in users) {
                var boughtSum = 0
                for((i,price) in emoticons.withIndex()) {
                    if(p[i] >= user[0]) {
                        boughtSum += ((100-p[i])*0.01*price).toInt()
                    }
                }
                if(boughtSum >= user[1]) { subscriberCounter++ }
                else{ selling += boughtSum }
            }
            if(answer[0] < subscriberCounter || (answer[0]==subscriberCounter) && answer[1] < selling) {
                answer[0] = subscriberCounter
                answer[1] = selling
            }
        }
        return answer.toIntArray()
    }
    
    private fun setRatePermutation(count: Int){
        if(count == max){
            ratePermutation.add(tempPermutation.copyOf().toIntArray())
            return
        }
        for(i in rate.indices){
            tempPermutation[count] = rate[i]
            setRatePermutation(count + 1)
        }
    }
}
```





