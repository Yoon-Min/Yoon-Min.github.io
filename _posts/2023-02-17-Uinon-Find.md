---
title: Union-Find 유니온 파인드
author: yoonmin
date: 2023-02-17 12:00:00 +0900
categories: [CS, 알고리즘]
tags: [자료구조, 유니온 파인드]
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

연결 리스트와 트리 구조를 코드로 구현해보겠습니다. 연결 리스트와 트리 구조의 노드는 구조체나 클래스 등을 통해서 만들 수 있지만 배열로도 구현이 가능합니다.

배열의 인덱스가 각 원소의 번호이며 인덱스에 해당하는 값은 해당 인덱스(원소 번호) 가 가리키고 있는 노드의 원소 번호입니다. 따라서 집합 내에서 대표 원소는 최상위에 위치하므로 가리킬 대상이 없고 자기자신을 가리키게 됩니다. 여기서는 배열의 이름를 `parent` 라고 하겠습니다.



### MakeSet

``` c++
void MakeSet(int X) {
  parent[x] = x;
}
```



### Union

```c++
void Union(int x, int y) {
    x = Find(parent[x]);
    y = Find(parent[y]);
    parent[x] = y;
}
```



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

## 하지만 문제점이 존재한다.

지금까지 설명드린 방법은 가장 간단한 방법입니다.  여기서 문제점이 존재합니다. 바로 `Find` 연산입니다. 연결 리스트와 트리 구조 둘 다 깊이가 커지면 대표 원소를 찾는 시간이 그만큼 오래 걸리게 됩니다. 원소가 `n`개라면 최악의 경우에는 `Find`연산이 `O(n)` 이 됩니다. 연산 시간을 단축시키기 위해서는 어떻게 해야 할까요?



### 연결 리스트는 각 노드에 헤드를 가리키는 포인터를 추가시킨다.





