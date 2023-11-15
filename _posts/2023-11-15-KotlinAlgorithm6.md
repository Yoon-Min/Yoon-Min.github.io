---
title: (프로그래머스 | Kotlin) - 수식 최대화
author: yoonmin
date: 2023-11-15 12:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

뭔가 풀이 방법이 다양할 것 같은 문제인데 나는 <span style="color: #30aaa0">**재귀를 이용해서 해결**</span>했다. 연산 기호가 최대 세 개인데 연산 우선순위를 리스트로 표현했다. 예를 들어서 표현식에서 사용된 연산 기호가 `*` `-` , 2개라면 만들 수 있는 우선순위 조합은 2개, `[[*, -], [-, *]]`  이렇게 리스트로 표현이 가능하다. 인덱스가 클수록 연산 우선순위가 높다는 뜻이다.

이런 식으로 만들 수 있는 연산 우선순위를 모두 만들고 모든 경우에 대해 식을 계산하고 결과값을 비교해서 가장 큰 값을 리턴하면 문제 해결이 가능하다. 그런데 <span style="color: #30aaa0">**문제는 식을 계산하는 방법**</span>이다. 여기서 풀이 방법이 다양하게 갈릴 것 같은데 내가 푼 방법은 다음과 같다.

​		

### 1. 나올 수 있는 연산 우선순위 조합 모두 구하기

`"100-200*300-500+20"` 예제를 예시로 생각해보자. 사용된 연산자는 `*` `-` `+` 총 세 개, 나올 수 있는 연산 우선순위 조합은 총 여섯 개다. 우선 이 여섯 개를 구하는 알고리즘을 짜야 한다. 이는 [**순열 알고리즘**](https://notepad96.tistory.com/108)을 이용해서 구하면 된다. 

제네릭 함수, 리스트 확장함수, 연산자 함수를 응용한 깔끔한 순열 알고리즘이다. 자세한 원리는 링크를 통해서 확인하면 된다. 개인적으로 가장 깔끔한 순열 알고리즘인 것 같다.

```kotlin
private fun <T> permutation(sub: List<T>, fin: List<T> = listOf()): List<List<T>> {
    return if(sub.isEmpty()) listOf(fin)
    else sub.flatMap { permutation(sub - it, fin + it) }
}
```

```kotlin
permutation(listOf('*', '-', '+'))
// [*, +, -]
// [*, -, +]
// [+, *, -]
// [+, -, *]
// [-, *, +]
// [-, +, *]
```

​		

### 2. 재귀를 이용한 식 계산

```kotlin
permutation(existingOperator.toList()).forEach {
    answer = max(answer, abs(rec(expression, it.toList())))
}
```

`rec` : <span style="color: #30aaa0">**식을 계산한 결과값을 반환하는 재귀 함수**</span>이고 계산할 식이 담긴 문자열과 연산 우선순위 정보를 인자로 넘긴다. 그리고 계산 결과의 절대값으로 비교를 해야 하므로 `abs` 로 감싸준다. 이렇게 메인 함수에서 틀을 잡는다.

다음은 `rec` 함수 구현 부분이다. `permutation` 결과값의 첫 번째 값이 `[*, +, -]` 이므로  `rec` 함수로 넘기는 인자는 `"100-200*300-500+20"` , `[*, +, -]` 다. 연산자 우선순위 리스트의 인덱스가 높을수록 연산 우선순위가 높다고 설정했기 때문에 `- -> + -> *`  순서로 계산을 처리한다.

해당 순서로 계산을 처리하기 위해선 연산 우선순위가 낮은순으로 연산자를 기준으로 `split` 을 하고, `split` 의 결과값으로 만들어진 리스트 내의 모든 식의 결과값들을 현재 기준이 되는 연산자로 계산을 해주면 된다. 글만 봐서는 바로 이해가 되지 않으니 예시를 살펴보자.

​		

#### 1. 연산 우선순위가 가장 낮은 연산자는 `*` 이므로 이 연산자를 기준으로 `split` 을 한다

```
"100-200*300-500+20" -> ["100-200", "300-500+20"]
```

식이 두 개로 분리가 됐다. `"100-200"` 을 `A` 라 하고 `"300-500+20"` 을 `B` 라고 한다면 `A * B` 가 `rec` 함수의 반환값이 된다. 따라서 `A` `B` 를 먼저 계산을 해야 한다. 여기서 `rec` 함수를 이용해서 `A` `B` 의 결과값을 받아오면 된다. 그리고 `*`  연산자를 기준으로 분리된 식들이기 때문에 이제 처리해야 할 연산자는 `+` `-`  둘뿐이다.

​		

#### 2. `split` 으로 분리된 식들의 결과값을 `rec` 재귀호출로 구한다.

따라서 `A` = `rec("100-200", ["+", "-"])`  , `B` = `rec("300-500+20", ["+", "-"])` 이 된다. 이제 재귀호출로  `A` `B` 를 구하는 `rec` 함수를 살펴보자.  `*` 연산자를 기준으로 처리했을 때와 마찬가지로 이번에는 `+` 연산자를 기준으로 `split` 한다.

```
A -> "100-200" -> ["100-200"]
B -> "300-500+20" -> ["300-500", "20"]
```

이제 `-` 연산자에 해당되는 식만 남았다. 이것들도 `rec` 함수를 재귀호출하여 계산 결과를 구한다. 각각 `rec("100-200", ["-"])` `rec("300-500", ["-"])` `rec("20", ["-"])` 를 `C` `D` `E` 라고 하면 `A` = `C` , `B` = `D` + `E` 가 된다. 그리고 마지막에는 `A` * `B` 가 최종 반환된다.

​		

#### 정리

1. 연산 우선순위대로 식을 계산하기 위해서는 연산 우선순위가 가장 낮은 연산자부터 해당 연산자를 기준으로 식을 `split` 한다. `split` 을 하면 분리된 식들로 구성된 리스트가 반환된다.
2. 분리된 식들의 계산 결과를 `rec` 함수를 재귀호출하여 구한다.
3. 분리된 식들의 계산 결과들을 현재 `split`  기준이 되는 연산자로 계산하여 반환한다.

{:.prompt-tip}

> 결국 가장 낮은 연산 우선순위의 연산자부터 가장 큰 우선순위의 연산자 순서로 `split` 및 재귀호출을 하게 되면 결국 재귀호출의 특성 때문에 계산 순서가 연산 우선순위가 가장 높은 연산자에서 가장 낮은 연산자 순으로 처리가 된다.

​		

#### `rec` 함수 코드

```kotlin
private fun rec(expression: String, splitOperators: List<Char>): Long {
    var sum: Long
    getNextSplitOperatorIndex(expression, splitOperators)?.let { i ->
        val splitOperator = splitOperators[i]
        val splitExpression = expression.split(splitOperator)
        sum = rec(splitExpression.first(), splitOperators.subList(i + 1, splitOperators.size))

        splitExpression.subList(1, splitExpression.size).forEach {
            sum = splitOperator calculate Pair(sum, rec(it, splitOperators.subList(i + 1, splitOperators.size)))
        }
        return sum
    }
    return expression.toLong()
}

private fun getNextSplitOperatorIndex(expression: String, splitOperators: List<Char>): Int? {
    var i = 0
    while(i < splitOperators.size && !expression.contains(splitOperators[i])) { i++ }
    return if (i == splitOperators.size) null else i
}
```

​		

## 연산 우선순위를 고려한 `rec` 함수 처리 과정

![Group 6](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/8ca5c7fe-fe2c-4217-8428-3d0e55d6976e)

​		

## 전체 코드

```kotlin
fun solution(expression: String): Long {
    var answer: Long = 0
    val existingOperator = mutableSetOf<Char>()

    listOf('*', '+', '-').forEach {
        if (expression.contains(it)) { existingOperator.add(it) }
    }
    permutation(existingOperator.toList()).forEach {
        answer = max(answer, abs(rec(expression, it.toList())))
    }
    return answer
}

private fun rec(expression: String, splitOperators: List<Char>): Long {
    var sum: Long
    getNextSplitOperatorIndex(expression, splitOperators)?.let { i ->
        val splitOperator = splitOperators[i]
        val splitExpression = expression.split(splitOperator)
        sum = rec(splitExpression.first(), splitOperators.subList(i + 1, splitOperators.size))

        splitExpression.subList(1, splitExpression.size).forEach {
            sum = splitOperator calculate Pair(sum, rec(it, splitOperators.subList(i + 1, splitOperators.size)))
        }
        return sum
    }
    return expression.toLong()
}

private fun getNextSplitOperatorIndex(expression: String, splitOperators: List<Char>): Int? {
    var i = 0
    while(i < splitOperators.size && !expression.contains(splitOperators[i])) { i++ }
    return if (i == splitOperators.size) null else i
}

private fun <T> permutation(sub: List<T>, fin: List<T> = listOf()): List<List<T>> {
    return if(sub.isEmpty()) listOf(fin)
    else sub.flatMap { permutation(sub - it, fin + it) }
}

private infix fun Char.calculate(pair: Pair<Long, Long>): Long {
    return when (this) {
        '*' -> pair.first * pair.second
        '+' -> pair.first + pair.second
        '-' -> pair.first - pair.second
        else -> throw IllegalArgumentException()
    }
}
```

