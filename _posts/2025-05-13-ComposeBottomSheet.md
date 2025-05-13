---
title: Android Compose - Material Design을 이용한 Bottom Sheet 구현하기
author: yoonmin
date: 2025-05-13 00:00:00 +0900
categories: [Android, Compose]
tags: [Android, Compose, UI, Bottom Sheet]
render_with_liquid: true
---

![1](/assets/img/post/branch10/thumbnail.jpg)_사진: [Unsplash](https://unsplash.com/ko/사진/창문이-많은-화려한-고층-건물이-보입니다-663m8HsG95I?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[rotekirsche20](https://unsplash.com/ko/@rotekirsche20?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

# Intro

이전 포스팅에서 Material Design에서 제공하는 `TextField` 활용 방법을 다뤘습니다. 이번에는 텍스트 필드가 아닌 `Bottom Sheet` 구현 과정을 다뤄보고자 합니다. Material Design에서 제공하는 바텀 시트는 두 가지가 있습니다. 

![1](/assets/img/post/branch10/1.png)

스탠다드 버전(1)

: 화면의 메인 UI 영역과 공존하며 두 영역 모두를 동시에 보고 상호 작용할 수 있습니다. 메인 UI 영역의 콘텐츠가 자주 스크롤되거나 팬될 때 화면에 기능이나 보조 콘텐츠가 표시되도록 하는 데 일반적으로 사용됩니다. 컴포즈에서는 `BottomSheetScaffold`로 제공됩니다.

모달 버전(2)

: 인라인 메뉴나 간단한 대화의 대안으로 사용됩니다. 특히 긴 목록의 액션 아이템을 제공하거나 아이템에 긴 설명과 아이콘이 필요할 때 그렇습니다. 컴포즈에서는 `ModalBottomSheet` 로 제공됩니다.

​		

# 프로젝트 기능 요구사항

![1](/assets/img/post/branch10/2.png){: width="300" .normal}![1](/assets/img/post/branch10/3.png){: width="300" .normal}

첫 번째 사진은 지도상의 여러 마커들과 상호작용할 수 있는 바텀 시트를 요구합니다. 두 번째 사진은 유저가 참여하고 있는 맛집 지도의 부가 기능을 제공하는 바텀 시트를 요구합니다.

## Compose 요구사항

첫 번째 요구사항은 배경 영역의 UI와 바텀 시트가 상호작용해야 하기 때문에 스탠다드 버전의 `BottomSheetScaffold` 를 사용합니다. 두 번째 요구사항은 배경 영역과 상호작용이 없고 해당 영역을 대체하는 시트지가 필요하므로 모달 버전의 `ModalBottomSheet` 를 사용합니다.

​		

# BottomSheetScaffold









