---
title: GitHub Blog 간편하고 빠르게 꾸며보자 - 깃허브 블로그 커스터마이징 팁
author: yoonmin
date: 2024-08-23 00:00:00 +0900
categories: [블로그, Github 블로그 만들기]
tags: [GitHub, Blog, 깃허브 블로그, 커스텀, 블로그 꾸미기]
render_with_liquid: true
---

![krishdiphong-prayoonwongkasem-IU_jPI1aCDY-unsplash](https://gist.github.com/user-attachments/assets/58e56709-2cb9-4732-a6da-b4157bf60bd5)_[Unsplash](https://unsplash.com/ko/사진/테이블-위에-앉아-있는-컴퓨터-키보드-IU_jPI1aCDY?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Krishdiphong Prayoonwongkasem](https://unsplash.com/ko/@beamerzz?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

## Intro

깃허브 블로그의 장점중 하나는 블로그 테마를 본인 입맛대로 커스터마이징할 수 있다는 것입니다. 깃허브 블로그를 제작하는 과정은 본인이 마음에 드는 템플릿(테마) 파일을 받아서 초기화를 진행한 후에 깃허브를 통해 배포하는 것이 일반적이라 봅니다.

블로그를 오픈하는 과정하는 설명대로 따라가면 수월하게 할 수 있습니다만 이후의 블로그 관리는 조금 어려울 수 있습니다. 아이러니하게도 제가 블로그 관리에서 가장 어려움을 느꼈던 부분은 깃허브 블로그의 최고 장점이라 할 수 있는 커스터마이징 부분이었습니다.

아무래도 모바일 개발을 주력으로 하다 보니 웹 개발과 관련된 개념에 무지했고 이 때문에 깃허브 블로그 테마 폴더 구조 파악이 쉽지 않았습니다. 이쁘게 블로그를 내 입맛대로 꾸며보고 싶은데 CSS 문법과 개념을 모르니 테마 폴더를 어디서부터 건드리고 어떻게 코드를 작성해야 하는지 고민이 많았습니다.

그래서 이번 포스팅은 이렇게 웹 프로그래밍 지식이 부족한 제가 어떻게 블로그 테마 내 이것저것들을 수정을 할 수 있었는지 조그만한 팁을 공유하고자 합니다. 이 글을 읽고 블로그 테마 커스터마이징에 많은 도움이 되길 바라겠습니다!

​		

## 크롬 개발자 도구를 활용하자

![스크린샷 2024-08-23 오후 6 50 55](https://gist.github.com/user-attachments/assets/5288b131-5e6b-43a3-9df7-ed2623acf36c)

블로그 디자인에 사용되는 테마 템플릿의 폴더 구조를 완벽하게 파악한다는 것은 어렵기 때문에 크롬 개발자 도구를 이용해서 커스텀이 필요한 요소가 정의된 위치를 빠르게 파악해야 합니다. 이를 위해 본인의 깃허브 블로그를 크롬 브라우저로 접속한 다음에 개발자 도구 페이지(_**Ctrl(Cmd) + Shift + C**_)를 부릅니다.

 		

​		

## 개발자 도구를 통한 커스터마이징

### 디자인 수정이 필요한 요소 클릭하기

![스크린샷 2024-08-23 오후 4 54 25](https://gist.github.com/user-attachments/assets/eb437eac-1e67-4e45-a7a3-dd493cd9a121)_인라인 코드 블럭(inline code block)을 디자인 수정 요소 예시로 사용합니다._

개발자 도구를 열어서 디자인 수정이 필요한 요소에 커서를 올리면 해당 요소에 대한 정보가 간략하게 메시지 박스로 볼 수 있습니다. 그리고 요소를 클릭하면 오른쪽 개발자 도구 페이지에서 해당 요소와 관련된 코드와 코드 위치 정보가 나타납니다.

​		

### 디자인 수정이 필요한 요소와 관련된 정보 확인하기

![Group 69](https://gist.github.com/user-attachments/assets/803e2ab0-6fb6-42b0-8540-695258916177)

_**청록색 영역**_ <span style="color: #55C2BC">**==**</span>

: 디자인 수정이 필요한 요소와 관련된 코드가 있는 영역입니다. 이곳에서 해당 요소가 어떻게 정의되었는지(모양, 색상, 크기, 효과 등) 확인할 수 있습니다.

_**초록색 영역**_ _<span style="color: #71DF55">**==**</span>_

: 청록색 영역에서 확인했던 디자인 정의 코드가 어느 위치에 있는지 표시하는 영역입니다. 어떤 코드 파일에 정의되어 있는지 확인할 수 있습니다. 그리고 정의된 코드가 위치한 라인까지 확인가능합니다.

​		

### 원하는 디자인 속성이 정의되어 있는 코드 찾기

