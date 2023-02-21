---
title: Union-Find 유니온 파인드
author: yoonmin
date: 2023-02-21 12:00:00 +0900
categories: [CS, 알고리즘]
tags: [알고리즘, 자료구조, 유니온 파인드]
render_with_liquid: false
---

## 서로소 집합(disjoint-set)을 관리하는 자료 구조

다양한 원소들이 서로소 집합을 이루는 경우가 있습니다. 필요에 따라서 이 집합들을 이용하여 서로 합치거나 특정 원소를 통해 특정 집합을 탐색할 수도 있습니다. 유니온 파인드(union-find)는 <span style="color: #30aaa0">**그래프 알고리즘으로 서로소 집합을 관리하며 서로 다른 두 노드가 동일한 집합에 있는지 판별**</span>합니다.

유니온 파인드는 <span style="color: #30aaa0">**서로 다른 두 노드가 같은 집합에 있는지 판별하기 위해서 각 집합의 대표 원소를 참고**</span>합니다. 여기서 대표 원소라 함은 연결 리스트에서는 헤드가 되며 트리에서는 루트 노드가 해당됩니다. 두 노드가 속한 집합의 대표 원소를 찾아서 대표 원소가 서로 동일하면 같은 집합에 있는 걸로 판단하게 됩니다. 이러한 판단 과정에서 여러 집합들이 합쳐지거나(Union) 특정 원소가 속한 대표 원소를 찾기 위해 집합 탐색(Find) 을 진행하기도 합니다. 그래서 유니온 파인드에서는 다음과 같은 연산들이 존재합니다.

​		

## 연산의 종류

| 연산                                            | 특징                                          |
| :---------------------------------------------- | :-------------------------------------------- |
| <span style="color: #30aaa0">**MakeSet**</span> | 특정 한 원소만 존재하는 집합을 만든다.        |
| <span style="color: #30aaa0">**Union**</span>   | 두 개의 집합을 하나의 집합으로 합친다.        |
| <span style="color: #30aaa0">**Find**</span>    | 특정 원소가 속한 집합의 대표 원소를 반환한다. |

​			

## 구현과 관리를 위해 사용되는 방법

유니온 파인드를 관리하는 방식은 두 가지가 있습니다. 첫 번째는 <span style="color: #30aaa0">**연결 리스트**</span>를 통한 관리고 두 번째는 <span style="color: #30aaa0">**트리 구조**</span>를 이용한 관리입니다. 각 방식들이 유니온 파인드의 연산을 어떻게 처리하는지 알아보도록 합시다.

### 연결 리스트

| 연산                                            | 방법                                                         |
| :---------------------------------------------- | :----------------------------------------------------------- |
| <span style="color: #30aaa0">**MakeSet**</span> | 원소 하나만 가지고 노드를 생성한다. 헤드가 해당 집합의 대표 원소다. |
| <span style="color: #30aaa0">**Union**</span>   | 한 집합의 헤드 부분이 다른 집합의 꼬리(Tail) 부분을 가리키는 방식으로 연결한다. |
| <span style="color: #30aaa0">**Find**</span>    | 지목한 원소의 집합에 존재하는 헤드 원소를 찾아 반환한다.     |

​		

### 트리 구조

| 연산                                            | 방법                                                         |
| :---------------------------------------------- | :----------------------------------------------------------- |
| <span style="color: #30aaa0">**MakeSet**</span> | 원소 하나만 가지고 노드를 생성한다. 루트 노드가 해당 집합의 대표 원소다. |
| <span style="color: #30aaa0">**Union**</span>   | 한 집합의 루트 노드를 다른 집합의 자식 노드로 연결한다.      |
| <span style="color: #30aaa0">**Find**</span>    | 특정 원소가 속한 집합의 루트 노드를 찾아 반환한다.           |

​		

## 연결 리스트를 통한 관리

### MakeSet

![1](https://user-images.githubusercontent.com/80873132/219954028-34fb73a6-02bc-4a97-bdff-befbb69993ce.png){: .normal}

​		

### Union

![Component 5](https://user-images.githubusercontent.com/80873132/220040318-f78ca869-950e-4288-8596-3d03ed0f22f9.png){: .normal}

​		

### Find

![Component 3](https://user-images.githubusercontent.com/80873132/219956460-f67ccd5d-a9fc-4418-be61-497c13981f7d.png){: .normal}

​		

## 트리 구조를 이용한 관리

### MakeSet

![Component 4](https://user-images.githubusercontent.com/80873132/220027222-16a7a7f2-44df-4343-834c-fd19270cdfc6.png){: .normal}

​		

### Union

![Component 6](https://user-images.githubusercontent.com/80873132/220040777-2e25970a-0596-4624-b23f-5d6996190e7f.png){: .normal}



​		

---

​		



![Component 7](https://user-images.githubusercontent.com/80873132/220042073-09112d5c-ddae-4b6c-9e3d-33548618a99e.png){: .normal}

​		

### Find

![Component 11](https://user-images.githubusercontent.com/80873132/220044423-a7be895c-7cf0-4628-a9ae-09a22508c76f.png){:.normal}

​		

## 코드로 구현

연결 리스트와 트리 구조를 코드로 구현해 보겠습니다. 연결 리스트와 트리 구조의 노드는 구조체나 클래스 등을 통해서 만들 수 있지만 배열로도 구현이 가능합니다.

배열의 인덱스가 각 원소의 번호이며 인덱스에 해당하는 값은 해당 인덱스(원소 번호) 가 가리키고 있는 노드의 원소 번호입니다. 따라서 집합 내에서 대표 원소는 최상위에 위치하므로 가리킬 대상이 없고 자기 자신을 가리키게 됩니다. 여기서는 배열의 이름을 `parent` 라고 하겠습니다. <span style="color: #30aaa0">( 예시로 원소는 8개, 1번 ~ 8번까지 있다고 가정하겠습니다. )</span>



### MakeSet

``` c++
void MakeSet(int X) {
  parent[x] = x;
}
```

|   index    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    |
| :--------: | :--- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| **parent** | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    |

​		



### Union

```c++
void Union(int x, int y) {
    x = Find(parent[x]);
    y = Find(parent[y]);
    parent[x] = y;
}
```

#### ex) Union(1, 2)

|   index    | 1                                         | 2    | 3    | 4    | 5    | 6    | 7    | 8    |
| :--------: | :---------------------------------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| **parent** | <span style="color: #30aaa0">**2**</span> | 2    | 3    | 4    | 5    | 6    | 7    | 8    |

#### ex) Union(2, 3)

|   index    | 1    | 2                                         | 3    | 4    | 5    | 6    | 7    | 8    |
| :--------: | :--- | ----------------------------------------- | ---- | ---- | ---- | ---- | ---- | ---- |
| **parent** | 2    | <span style="color: #30aaa0">**3**</span> | 3    | 4    | 5    | 6    | 7    | 8    |

​		

### Find

```c++
int Find(int x) {
    if(parent[x] == x) return x;
    return Find(parent[x]);
}
```

{: .prompt-tip}

> **재귀 방식**을 통해서 루트 노드 혹은 헤드 노드까지 이동해서 해당 원소를 반환합니다.

​		

## 문제점과 해결 방안 : 연결 리스트 방식

지금까지 설명드린 방법은 가장 간단한 방법입니다.  여기서 문제점이 존재합니다. 바로 `Find` 연산입니다.  <span style="color: #30aaa0">**연결 리스트와 트리 구조 둘 다 깊이가 커지면 대표 원소를 찾는 시간이 그만큼 오래 걸리게 됩니다.**</span> 원소가 `n`개라면 최악의 경우에는 `Find`연산이 `O(n)` 이 됩니다. 연산 시간을 단축시키기 위해서는 어떻게 해야 할까요?



### 각 노드에 헤드를 가리키는 포인터를 추가시킨다.

이전에 설명드린 것처럼 연결 리스트는 꼬리(Tail) 부분에서 헤드를 찾게 되면 해당 집합의 모든 원소들을 거치게 됩니다. 운 좋게 원소 2를 지목하면 한 번 이동하고 바로 헤드를 찾게 되지만 그림처럼 꼬리 부분의 원소를 지목하게 될 경우에는 전체를 거쳐서 탐색 시간이 오래 걸리게 됩니다.

![Component 12](https://user-images.githubusercontent.com/80873132/220244759-82a6d8e4-08cc-4c24-8827-cc61d6c7a747.png){:.normal}

​		

그래서 나온 해결책이  <span style="color: #30aaa0">**모든 노드에 헤드를 가리키는 포인터를 추가하는 것**</span>입니다. 그렇게 되면 어느 원소를 지목해도 상수 시간에 바로 헤드에 위치한 원소를 알 수 있습니다.

​	![Component 15](https://user-images.githubusercontent.com/80873132/220248338-ae76d241-4ff6-4a10-ad89-1f9a0284f579.png){:.normal}



​		

{: .prompt-danger}

> 하지만 만약 Union을 진행하면 합쳐지는 집합의 모든 원소들이 가리키는 헤드 업데이트가 필요하다.

​		

![Component 19](https://user-images.githubusercontent.com/80873132/220250740-47709344-08c3-4b37-8104-3fa0fcddb974.png){:.normal}

​		

![Component 20](https://user-images.githubusercontent.com/80873132/220251260-c688b676-1597-4ff7-a1c5-76b29375edff.png)

​		

​	![Component 18](https://user-images.githubusercontent.com/80873132/220250361-f9f9febf-7db2-4add-97d6-517ac0a76d05.png){:.normal}

​		

## 문제점과 해결 방안 : 트리 구조 방식

### 유니온 바이 랭크(Union by rank)

<span style="color: #30aaa0">**두 집합을 Union 할 때 트리의 깊이가 작은 집합을 트리의 깊이가 큰 집합의 루트 노드에 붙이는 방식입니다.**</span> 이러한 방식으로 유니온을 하면 두 트리의 깊이가 동일한 경우에만 깊이가 증가하게 됩니다. 따라서 깊이가 증가하는 것을 최대한 방지해서 `Find` 연산 시간을 단축시킵니다.

​		

![Component 22](https://user-images.githubusercontent.com/80873132/220267345-3aad9ce0-0d41-43f2-92dd-28f440e7f986.png){:.normal}

​		

{:.prompt-danger}

> 만약 유니온 바이 랭크를 적용하지 않는다면 유니온 했을 때 깊이가 증가할 수 있습니다.

​		

![Component 7](https://user-images.githubusercontent.com/80873132/220266872-3e92fb2b-71c8-438b-9a24-e9f50c7eac2c.png){:.normal}

​			

### 유니온 바이 랭크 구현

```
 function MakeSet(x)
     x.parent := x
     x.rank   := 0
```

```
 function Union(x, y)
     xRoot := Find(x)
     yRoot := Find(y)
     // if x and y are already in the same set (i.e., have the same root or representative)
     if xRoot == yRoot
         return
```

```
     // x and y are not in same set, so we merge them
     if xRoot.rank < yRoot.rank
         xRoot.parent := yRoot
     else if xRoot.rank > yRoot.rank
         yRoot.parent := xRoot
     else
         yRoot.parent := xRoot
         xRoot.rank := xRoot.rank + 1
```

{:.prompt-info}

> 해당 코드는 **wekipedia 서로소 집합 자료 구조**에서 참조했음을 알려드립니다.
>
> [**wekipedia 서로소 집합 자료 구조**](https://ko.wikipedia.org/wiki/%EC%84%9C%EB%A1%9C%EC%86%8C_%EC%A7%91%ED%95%A9_%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0)

​		

### 경로 압축(path compression)

 `Find(x)` 연산은 원소 `x` 가 속한 집합의 대표 원소를 찾아줍니다. 가장 간단한 방법은 대표 원소를 찾아 반환하고 끝내는 방식이지만 경로 압축에서는 <span style="color: #30aaa0">**대표 원소를 반환하기 전에 `x` 를 루트 노드(대표 원소)에 연결**</span>하고 대표 원소를 반환합니다. 이렇게 하면 모든 원소들이 대표 원소를 의미하는 루트 노드를 가리키게 됩니다. 이런 방식으로 모든 원소들을 루트 노드(대표 원소)로 연결시키면 연산 시간을 단축시킬 수 있습니다.

​				

![Component 23](https://user-images.githubusercontent.com/80873132/220275279-9033baba-f8ec-41fe-95b6-cda76afca752.png){:.normal}

​		

### 경로 압축 구현

```
 function Find(x)
     if x.parent != x
        x.parent := Find(x.parent)
     return x.parent
```

{:.prompt-info}

> 해당 코드는 **wekipedia 서로소 집합 자료 구조**에서 참조했음을 알려드립니다.
>
> [**wekipedia 서로소 집합 자료 구조**](https://ko.wikipedia.org/wiki/%EC%84%9C%EB%A1%9C%EC%86%8C_%EC%A7%91%ED%95%A9_%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0)

​		

## 참조

[**wekipedia 서로소 집합 자료 구조**](https://ko.wikipedia.org/wiki/%EC%84%9C%EB%A1%9C%EC%86%8C_%EC%A7%91%ED%95%A9_%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0)

[**신찬수 교수님 : 자료구조 - union-find 자료구조 1/2**](https://www.youtube.com/watch?v=GaET9oHzC3Q)
