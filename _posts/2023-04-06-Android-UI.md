---
title: Android의 UI, 뷰(View) 그리고 레이아웃(ViewGroup)
author: yoonmin
date: 2023-04-06 12:00:00 +0900
categories: [Android, 기본]
tags: [Android, UI, 레이아웃, 뷰]
render_with_liquid: false
---

## 모바일 UI는 어떻게 구성되어 있는가?

예시로 우리가 스마트폰을 이용해서 배달의민족 앱을 이용할 때를 생각해 봅시다. 해당 앱에 접속을 해보면 여러 카테고리 버튼이 있고 밑으로 스크롤을 하면 여러 음식점들의 간략한 정보가 리스트 형식, 혹은 카드 형태로 제공됩니다. 

특정 음식점을 클릭하면 해당 음식점에 대한 상세 페이지로 이동하게 되고 여기서는 가게에 대한 텍스트 정보, 음식 사진에 대한 이미지 정보, 메뉴들을 선택할 수 있는 체크박스, 해당 가게 정보를 저장할 수 있는 찜 버튼과 같은 다양한 상호작용 요소들을 볼 수 있습니다. 

{:.prompt-tip}

> 이렇듯 모바일 UI는 하나의 화면 안에 사용자와 상호작용이 가능한 다양한 요소들이 존재합니다.

​		

## Android는 UI를 어떻게 제작하는가?

그렇다면 Android는 이러한 모바일 UI를 어떤 방식으로 구현할까요? 가장 먼저 사용자에게 앱과 상호작용할 수 있는 화면으로 안내하는 것이 우선입니다.

### **상호작용을 위한 화면**

안드로이드는 사용자가 앱과 상호작용할 수 있도록 진입점을 제공하는데 이것이 `Activity` 입니다. 액티비티는 안드로이드 앱을 구성하는 4대 구성 요소 중에 하나이며 <span style="color: #30aaa0">**사용자에게 UI를 제공하여 상호작용을 하기 위한 진입점입니다.**</span> 안드로이드 앱 내에 여러 화면을 구성할 수 있는 것도 액티비티가 있기 때문입니다.

### **화면 내에 존재하는 여러 상호작용 요소들**

사용자가 `Activity` 에 도달했다면 이제는 그곳에 배치된 여러 요소들을 통해서 본격적으로 상호작용을 할 수 있게 됩니다. 이 요소들은 뷰(View) 와 뷰그룹(ViewGroup) 이라는 것들의 계층 구조로 이루어져 있습니다. 배달의민족을 통해 음식점 상세 페이지에 갔을 때 볼 수 있는 <span style="color: #30aaa0">텍스트, 이미지, 체크박스, 이미지 버튼 등의 요소들을 **뷰(View)** 라고 하고 이런 여러 뷰들을 담는 컨테이너를 **뷰그룹(ViewGroup)** 이라고 합니다.</span>

​		

## 뷰(View)

안드로이드는 사용자와 상호작용할 수 있는 요소 제작과 배치를 위해서 `View` 객체를 제공합니다. 뷰는 <span style="color: #30aaa0">**사용자 인터페이스의 기본적인 구성 요소이며 그리기 및 이벤트 처리를 담당합니다.**</span> 뷰를 다른 말로 위젯이라고도 합니다.

뷰 객체는 여러 속성과 메서드를 가지고 있습니다. 대표적으로  `draw()` 라는 메서드가 있는데 이를 통해서 화면에 뷰를 그릴 수 있습니다. 이벤트 처리 같은 경우에는 대표적인 예시로 클릭에 대한 이벤트가 있는데 사용자가 뷰를 클릭하면 이를 감지하여 어떤 동작을 처리할 수 있게 지원합니다.

### **여러 뷰들은 View 객체로부터 파생된 것(View Hierarchy)**

뷰 객체가 가지고 있는 속성과 메서드의 상속을 통해서 특정 기능을 가진 뷰로 확장시키는 방식으로 다양한 뷰들을 만들 수 있는데 안드로이드에서 기본적으로 제공하는 뷰들이 이에 해당됩니다. 따라서  많이 사용되는  `TextView`, `Button`, `ImageView` 등의 뷰들은 전부 `View` 객체에서 파생된 뷰입니다. 

기본 제공되는 뷰들을 사용해서 UI 구성을 진행하고 만약에 기본적으로 제공되는 뷰의 기능이 부족하다면 직접 뷰 객체를 커스텀 해서 만들어야 합니다. 이를 커스텀 뷰라고 하는데 이 부분은 나중에 따로 포스팅해보겠습니다.

### **뷰의 상속 관계(View Hierarchy)**

![viewHier](https://user-images.githubusercontent.com/80873132/230453320-9e42e1d1-a06f-40ce-8a87-3629e4bb2d68.png)

​		

​		

## 뷰그룹(ViewGroup)

`ViewGroup` 은 상속 관계도에서 확인할 수 있듯이  `View` 를 상속받은 객체입니다. 뷰그룹은 다른 말로  <span style="color: #30aaa0">**레이아웃**</span>이라고 하는데 여러 `View` 들을 담을 수 있고 이런 여러 뷰들의 배치를 도와줍니다. 뷰그룹은 여러 뷰도 담을 수 있지만 다른 뷰그룹도 담을 수 있습니다. 그래서 중첩으로 레이아웃을 만들어 좀 더 복잡한 배치를 할 수 있습니다. 레이아웃의 종류는 크게 6개로 볼 수 있습니다.

### LinearLayout

{:.prompt-tip}

> <span style="color: #30aaa0">**뷰를 일렬로 배치하는 레이아웃**</span>

- 배치 방식을 위한 `orientation` 속성을 제공

  - `orientation = vertical` : 세로로 배치

    ![vee](https://user-images.githubusercontent.com/80873132/230606358-8d1928c8-81ee-4e67-932f-78e6aa255e52.png){: .w-75 .rounded-10 w='200' h='300' .normal }

  - `orientation = horizontal` : 가로로 배치

    ![hoo](https://user-images.githubusercontent.com/80873132/230608289-cccad59e-0f57-4e08-b2f0-e06c46ef00ed.png){:. .w-75 .rounded-10 w='200' h='300' .normal }

​		

### RelativeLayout

{: .prompt-tip}

> <span style="color: #30aaa0">**뷰의 상대적인 위치에 따라 배치하는 레이아웃**</span>

- 상대적인 위치 설정을 위한 속성들을 제공 `RelativeLayout.LayoutParams`

- 속성 몇 가지 예시

  - `layout_alignParentTop`
  - `layout_centerVertical`
  - `layout_below`
  - `layout_toRightOf`

- 배치 예시

  ![Group 15](https://user-images.githubusercontent.com/80873132/230609395-1d5ec1bc-6326-410a-ac7c-2ac145bac110.png){:. .w-75 .rounded-10 w='200' h='300' .normal }

​		

### FrameLayout

{:.prompt-tip}

> <span style="color: #30aaa0">**뷰를 중첩해서 배치하는 레이아웃**</span>

- `visibility`  속성을 이용해서 상황에 따라 특정 뷰만 보이게 하고 나머지 뷰는 보이지 않게 설정

- 배치 예시

  ![Group 17](https://user-images.githubusercontent.com/80873132/230611326-e16aefcb-971c-47e2-a668-c76064ab831a.png){:. .w-75 .rounded-10 w='200' h='300' .normal }

​		

### TableLayout

{:.prompt-tip}

> <span style="color: #30aaa0">**행과 열을 통해 표 형태로 뷰를 배치하는 레이아웃**</span>

- `TableRow`  를 이용해 한 행을 생성하고 이 안에 여러 뷰들을 만들어서 열을 채우는 방식으로 배치

- `TableRow` 내에는 `View` 혹은 `ViewGroup`  를 넣을 수 있음

- 배치 예시

  ![Group 18](https://user-images.githubusercontent.com/80873132/230612551-78967a69-c44c-418b-915e-f178136c0341.png){:. .w-75 .rounded-10 w='200' h='300' .normal }

​		

### GridLayout

{:.prompt-tip}

> 격자 배치 레이아웃

- `TableLayout` 와 마찬가지로 행과 열을 이용해 배치, 하지만 그리드레이아웃이 더 유연한 배치가 가능

- 행과 열에 대한 속성을 제공해서 유연한 배치 가능

- 배치 예시

  ![Group 20](https://user-images.githubusercontent.com/80873132/230627253-2804363d-eb7e-4f72-b51c-9c0a662dfb86.png){:. .w-75 .rounded-10 w='200' h='300' .normal }

​		

### ContraintLayout

{:.prompt-tip}

> 제약 조건을 통한 배치 레이아웃

- 부모 레이아웃을 기반으로 배치하는 것이 아닌 동일한 뷰를 기반으로 유연한 배치가 가능

- 뷰와 뷰그룹의 계층 구조로 이루어진 Android의 UI 에서 제약레이아웃을 통해 플랫한 계층 구조 설계가 가능

- 사용 가능한 제약 조건들

  - Relative positioning
  - Margins
  - Centering positioning
  - Circular positioning
  - Visibility behavior
  - Dimension constraints
  - Chains
  - Virtual Helpers objects
  - Optimizer

- 배치 예시

  ![asdc](https://user-images.githubusercontent.com/80873132/230628600-2c531b5d-f4e5-4bc3-964f-b904464b7a2c.png){:. .w-75 .rounded-10 w='200' h='300' .normal }

​		

## Android의 UI는 뷰와 뷰그룹의 계층 구조

지금까지의 이야기를 종합해 보면 사용자와 상호작용을 위한 기본적인 구성 요소인 뷰들을 레이아웃에 담아서 배치하는 식으로 UI를 제작합니다. UI가 단순하다면 단순히 하나의 레이아웃 안에 여러 뷰들을 넣어서 배치할 것이고 UI가 좀 더 복잡하다면 트리 구조처럼 레이아웃 안에 레이아웃을 더 만들어 배치할 것입니다.

따라서 <span style="color: #30aaa0">**안드로이드의 UI는 뷰와 뷰그룹의 계층 구조로 이루어져 있습니다.**</span> 레이아웃 XML 파일을 들여다보면 루트에 레이아웃이 있고 그 안에 뷰나 레이아웃을 만드는 식의 계층 구조 형태로 제작하게 돼있습니다.

![viewgroup_2x](https://user-images.githubusercontent.com/80873132/230632001-51b5f93e-ce17-4ff8-be9b-9a926f33e9a1.png) 

​		

## 계층 구조 레이아웃의 최적화

### 트리 구조를 생각해보자.

트리 구조의 깊이가 증가할수록 어떤 단점이 발생할까요? 노드 탐색이 느려질 수 있다는 단점이 생깁니다. 만약에 찾고 싶은 노드가 가장 밑단에 위치해 있다면 트리의 깊이가 깊을수록 탐색 시간도 늘어나게 됩니다.

트리를 이용한 [ <span style="color: #30aaa0">**Union-Find**</span>](https://yoon-min.github.io/posts/Uinon-Find/)도 탐색 시간 최적화를 위해서 모든 자식 노드의 부모 노드를 루트 노드로 설정해서 깊이를 최소로 줄이는 방법을 사용합니다.

### Android UI 계층 구초는 가능한 한 얕게 유지해야 한다.

뷰와 뷰그룹의 계층 구조의 안드로이드 UI도 가능하면 <span style="color: #30aaa0">**깊이를 얕게 유지해야 합니다.** </span>중첩 레이아웃이 레이아웃 계층 구조의 깊이를 증가시키는 원인이므로 되도록이면 중첩 레이아웃을 사용하지 않고 배치하는 노력을 하는 것이 좋습니다. 얕게 유지하면 유지할수록 레이아웃이 더 빠르게 화면에 그려지게 됩니다.

{:.prompt-tip}

> 가로로 넓은 뷰 계층 구조가 세로로 깊은 뷰 계층 구조보다 낫습니다.



### ConstraintLayout을 주로 사용하는 이유

 `ConstraintLayout` 은 이것만으로 다른 대부분의 레이아웃들의 배치 특성을 구현할 수 있고 뷰에 대한 포지셔닝이 굉장히 유연해서 가로로 넓은 뷰 계층 구조를 만들 수 있습니다. 그래서 저는 제약레이아웃을 주로 사용하고 있습니다.

​		

## 마무리

결국에는 여러 뷰와 뷰그룹을 통해서 UI를 구상하고 완성된 레이아웃 XML 파일은 `Activity` 에 연결되어 사용자에게 제공됩니다. 그래서 액티비티에서 뷰를 참조하여 특정한 UI 처리를 할 수 있습니다. 요즘에는 XML 방식이 아닌 최신 UI 제작 도구인 Jetpack compose를 도입하려는 움직임이 많습니다. 저도 나중에 기회가 되면 꼭 배워보려고 합니다.

마지막으로 혹시나 틀린 부분이 있다면 댓글로 남겨주세요. 바로 수정하도록 하겠습니다. 긴 글 읽어주셔서 감사하고 이 글을 읽는 여러분 모두 좋은 하루 보내시길 바랍니다!

​		

## 참조

[<span style="color: #30aaa0">**안드로이드 공식문서 - 레이아웃**</span>](https://developer.android.com/guide/topics/ui/declaring-layout?hl=ko)

[<span style="color: #30aaa0">**안드로이드 공식문서 - 성능 및 뷰 계층 구조**</span>](https://developer.android.com/topic/performance/rendering/optimizing-view-hierarchies?hl=ko)

[<span style="color: #30aaa0">**찰스의 안드로이드**</span>](https://www.charlezz.com/?p=792)
