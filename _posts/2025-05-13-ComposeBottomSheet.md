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

```kotlin
@Composable
@ExperimentalMaterial3Api
fun BottomSheetScaffold(
    sheetContent: @Composable ColumnScope.() -> Unit,
    modifier: Modifier = Modifier,
    scaffoldState: BottomSheetScaffoldState = rememberBottomSheetScaffoldState(),
    sheetPeekHeight: Dp = BottomSheetDefaults.SheetPeekHeight,
    sheetMaxWidth: Dp = BottomSheetDefaults.SheetMaxWidth,
    sheetShape: Shape = BottomSheetDefaults.ExpandedShape,
    sheetContainerColor: Color = BottomSheetDefaults.ContainerColor,
    sheetContentColor: Color = contentColorFor(sheetContainerColor),
    sheetTonalElevation: Dp = 0.dp,
    sheetShadowElevation: Dp = BottomSheetDefaults.Elevation,
    sheetDragHandle: @Composable (() -> Unit)? = { BottomSheetDefaults.DragHandle() },
    sheetSwipeEnabled: Boolean = true,
    topBar: @Composable (() -> Unit)? = null,
    snackbarHost: @Composable (SnackbarHostState) -> Unit = { SnackbarHost(it) },
    containerColor: Color = MaterialTheme.colorScheme.surface,
    contentColor: Color = contentColorFor(containerColor),
    content: @Composable (PaddingValues) -> Unit
) {
    BottomSheetScaffoldLayout(
        modifier = modifier,
        topBar = topBar,
        body = { content(PaddingValues(bottom = sheetPeekHeight)) },
        snackbarHost = { snackbarHost(scaffoldState.snackbarHostState) },
        sheetOffset = { scaffoldState.bottomSheetState.requireOffset() },
        sheetState = scaffoldState.bottomSheetState,
        containerColor = containerColor,
        contentColor = contentColor,
        bottomSheet = {
            StandardBottomSheet(
                state = scaffoldState.bottomSheetState,
                peekHeight = sheetPeekHeight,
                sheetMaxWidth = sheetMaxWidth,
                sheetSwipeEnabled = sheetSwipeEnabled,
                shape = sheetShape,
                containerColor = sheetContainerColor,
                contentColor = sheetContentColor,
                tonalElevation = sheetTonalElevation,
                shadowElevation = sheetShadowElevation,
                dragHandle = sheetDragHandle,
                content = sheetContent
            )
        }
    )
}
```

컴포즈에서 제공하는 스탠다드 버전 바텀 시트입니다. 위에서 설명했듯이 바텀 시트 영역을 담당하는 컴포저블과 시트 뒤의 배경 영역을 담당하는 컴포저블까지 두 개의 컴포저블 구현이 필요합니다. 배경 영역은 `content` , 바텀 시트 영역은 `sheetContent` 에 정의합니다.

​		

## 주요 파라미터

### sheetContent

![1](/assets/img/post/branch10/4.png)

바텀 시트에 표시될 콘텐츠를 정의하는 컴포저블입니다. 위 사진처럼 지도와 마커가 배경을 차지하고 해당 영역과 상호작용(마커 필터링)하기 위한 컴포저블을 이곳에 정의합니다.

​		

### scaffoldState

바텀 시트의 상태를 제어하는 객체입니다. 확장 및 닫힘 상태를 제어하거나 조회할 수 있습니다. 또한 확장이나 닫힘 상태로 바뀜에 따라 변화하는 위치 값도 수신 가능합니다. 만약 특정 상태나 조건에 따라 바텀 시트 상태를 제어하려면 외부에서 `rememberBottomSheetScaffoldState()` 를 만들어 관리하면 됩니다.

​		

### sheetPeekHeight

바텀 시트가 닫힌 상태에서의 높이를 의미합니다. 모달 버전과 다르게 스탠다드 버전은 바텀 시트를 일부 노출시켜 사용자가 이를 끌어올려 확인할 수 있도록 기본 높이 값 지정을 지원합니다. 기능 요구사항에 따라 음식점 검색 옵션 텍스트 헤더가 보일만큼 기본 높이 값을 조정하면 됩니다.

```kotlin
sheetPeekHeight: Dp = BottomSheetDefaults.SheetPeekHeight
// BottomSheetDefaults.SheetPeekHeight
val SheetPeekHeight = 56.dp
```

개발자가 따로 높이 값을 지정하지 않으면 파라미터 기본 값에 따라 `56.dp` 로 초기 설정됩니다.

​				

### sheetShape

바텀 시트 모양을 설정합니다. 바텀 시트 기본 형태는 `RoundCornerShape` 에 상단 `28.dp` 입니다. 디자인에 따라 기본 형태와 Dp 값을 설정하시면 되겠습니다.

​		

### sheetDragHandle



