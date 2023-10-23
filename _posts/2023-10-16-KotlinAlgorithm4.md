---
title: (프로그래머스 | C++) - 숫자 블록
author: yoonmin
date: 2023-10-16 20:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

<span style="color: #30aaa0">**이 문제의 핵심은 가장 큰 약수를 구하는 데 있다.**</span>  문제에서 블록의 번호가 `n` 이고 해당 블록은 `n*2` `n*3` ... 이렇게 설치를 한다. 이것은 곧 약수가 됨을 의미한다. 예를 들어서 구간 `10` 에 설치될 블록의 번호 무엇일까? 바로  `5` 다. 왜 이렇게 나오는 것일까? <span style="color: #30aaa0">**구간이 가지고 있는 약수들 중에서 본인을 제외한 가장 큰 숫자가 설치될 블록 번호**</span>이기 때문이다.

`10` 의 약수는 `1, 2, 5, 10` 인데 약수 `1` 부터 살펴보면 `1*10 = 10` 인 것을 알 수 있고 `n=1` 로 표현이 가능하다. 그래서 처음에는 구간 `10` 에 `1` 블록이 설치된다.  그 다음 블록 번호는 `2` 다. 마찬가지로 `2*5=10` 인 것을 알 수 있고 `n=2` 로 표현이 가능하다. 따라서 이미 설치된 `1` 블록에 `2` 블록을 덮어씌운다. 그 다음 블록 번호 `3` `4` 는 특정 숫자를 곱해서 `10` 을 만들 수 없으므로 패스한다. 이런 방식으로 진행을 하면 `5` 가 `5*2=10` 이므로 구간 `10` 에 마지막으로 설치되는 블록 번호는 `5` 임을 알 수 있다. 결국 구간이 가진 본인 제외, 가장 큰 약수가 해당 구간에 설치될 블록의 번호다.

1. 특정 숫자에 대해서 가장 큰 약수를 반환하는 메서드를 정의한다.

   ```c++
   int GetMostDivisor(int num) {
       if(num == 1) return 0;
       vector<int> divisor(1, 1);
       
       for(int i=2; i<=sqrt(num); i++) {
           if(num%i == 0) {
               if(num/i <= MAX_BLOCK_NUM) return num/i;
               divisor.push_back(i);
           }   
       }
       return divisor.back();
   }
   ```

   

2. `begin` ~ `end` 범위의 반복문을 통해 본인 제외 가장 큰 약수를 저장한다.

   ```c++
   for(int i=begin; i<end+1; i++) {
       answer[i-begin] = GetMostDivisor(i);
   }
   ```

   

## 설명

#### 순서 1.

가장 큰 약수를 구하려면 가장 작은 수부터 나눠서 확인하면 된다. 이때 문제 조건인 블록 최대값 `10,000,000` 이 넘어가지 않는다면 해당 결과값이 된다. 아니라면 따로 약수 후보들을 배열에 저장해놓고 반복문이 끝나면 해당 배열에서 최대값을 리턴하면 된다.

#### 순서 2.

각 구간의 시작과 끝을 반복문으로 돌면서 `answer` 에 저장하면 간단하게 끝난다.



## 전체 코드

```c++
#include <string>
#include <vector>
#include <math.h>
#include <iostream>

using namespace std;

const int MAX_BLOCK_NUM = 10000000;

int GetMostDivisor(int num) {
    if(num == 1) return 0;
    vector<int> divisor(1, 1);
    
    for(int i=2; i<=sqrt(num); i++) {
        if(num%i == 0) {
            if(num/i <= MAX_BLOCK_NUM) return num/i;
            divisor.push_back(i);
        }   
    }
    return divisor.back();
}

vector<int> solution(long long begin, long long end) {
    vector<int> answer(end-begin+1, 0);
    
    for(int i=begin; i<end+1; i++) {
        answer[i-begin] = GetMostDivisor(i);
    }
    return answer;
}
```



