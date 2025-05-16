---
title: Android Compose - Material Design을 이용한 Bottom Sheet 구현하기
author: yoonmin
date: 2025-05-16 00:00:00 +0900
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

![1](/assets/img/post/branch10/4.png){:.normal}

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

![1](/assets/img/post/branch10/5.png)

사용자가 바텀 시트를 끌어 올릴 수 있는 시각적 요소입니다. 위 사진은 기본 형태의 드래그 핸들이며 커스텀이 가능합니다. 드래그 핸들의 사이즈는 `32X4`, 버티컬 패딩 값은 `22Dp` 입니다. 피그마 디자인 페이지에는 `16Dp` 로 설정되어 있는데 최근에 수치를 조정한 것 같습니다.

​		

### content

![1](/assets/img/post/branch10/6.png){:.normal}

바텀 시트의 배경을 차지하는 메인 콘텐츠 영역입니다. 이곳에 바텀 시트와 상호작용할 수 있는 내용을 정의합니다. 바텀 시트가 지도상에 존재하는 마커를 필터링하는 역할을 하므로 배경 영역에는 지도와 마커를 표시합니다.

​		

## PeekHeight 세부조정하기

`Content` 및 `SheetContent` 영역의 컴포저블 제작은 완료됐다고 가정하고 다음 작업으로 넘어 가겠습니다. 이제 바텀 시트의 초기 높이 값을 지정해야 합니다. 기능 요구사항과 같이 음식점 검색 옵션 텍스트 헤더만 보이게 높이를 정확하게 설정해야 합니다. 

따라서 먼저 텍스트 헤더의 높이 값을 구해 봅시다. 텍스트 헤더를 지닌 로우 컴포저블의 높이는 `Modifier` 의 `onGloballyPositioned` 메서드를 통해 얻을 수 있습니다. 컴포저블 레이아웃 `size` 에 접근하여 `height` 에 접근합니다.

​		

### onGloballyPositioned

```kotlin
Row(
    Modifier.onGloballyPositioned { coordinates ->
    	coordinates.size.height // Row 레이아웃의 height 값 (pixel)
    }
)
```

하지만 이 높이 값은 `Dp` 가 아닌 픽셀 값이기 때문에 그대로 사용하면 실제 높이 값과는 전혀 다른 값이 사용될 겁니다. `Dp` 로 변환하기 위해서는 픽셀 값을 모바일 디바이스가 가진 디스플레이 `Density` 값으로 나누어야 합니다. (운영체제가 동일한 안드로이드여도 기기마다 디스플레이 밀도가 다르기 때문)

​		

### Density

```kotlin
val density = LocalDensity.current.density

@OptIn(InternalComposeApi::class)
inline val current: T
    @ReadOnlyComposable @Composable get() = currentComposer.consume(this)
```

디스플레이 밀도는 위와 같이 쉽게 구할 수 있습니다. 대신 `current` 는 컴포저블 특성을 가지기 때문에 밀도 값을 구할 때는 컴포저블 내에서 호출해야 합니다.

```kotlin
Row(
    Modifier.onGloballyPositioned { coordinates ->
    	coordinates.size.height / density
    }
)
```

구한 밀도 값을 위와 같이 레이아웃 높이 값에 나누면 실제 로우 컴포저블 레이아웃의 높이 `Dp` 값을 얻습니다.

​		

### DrageHandle

```kotlin
@Composable
fun DragHandle(
    modifier: Modifier = Modifier,
    width: Dp = SheetBottomTokens.DockedDragHandleWidth,
    height: Dp = SheetBottomTokens.DockedDragHandleHeight,
    shape: Shape = MaterialTheme.shapes.extraLarge,
    color: Color = SheetBottomTokens.DockedDragHandleColor.value,
) {
    val dragHandleDescription = getString(Strings.BottomSheetDragHandleDescription)
    Surface(
        modifier =
            modifier.padding(vertical = DragHandleVerticalPadding).semantics {
                contentDescription = dragHandleDescription
            },
        color = color,
        shape = shape
    ) {
        Box(Modifier.size(width = width, height = height))
    }
}

private val DragHandleVerticalPadding = 22.dp

internal object SheetBottomTokens {
    val DockedContainerColor = ColorSchemeKeyTokens.SurfaceContainerLow
    val DockedContainerShape = ShapeKeyTokens.CornerExtraLargeTop
    val DockedDragHandleColor = ColorSchemeKeyTokens.OnSurfaceVariant
    val DockedDragHandleHeight = 4.0.dp
    val DockedDragHandleWidth = 32.0.dp
    val DockedMinimizedContainerShape = ShapeKeyTokens.CornerNone
    val DockedModalContainerElevation = ElevationTokens.Level1
    val DockedStandardContainerElevation = ElevationTokens.Level1
    val FocusIndicatorColor = ColorSchemeKeyTokens.Secondary
}
```

텍스트 헤더의 높이 값을 구했으니 다음은 드래그 핸들 컴포저블의 높이 값을 구해야 합니다. 이 값은 위에서 설명한 대로 `32 X 4` 크기의 드래그 핸들 박스에 `vertical` 패딩이 `22Dp` 로 기본 설정되어 있습니다. 내부 코드에서 확인 가능합니다. 따라서 드래그 핸들의 높이 값은 `22 + 4 + 22 = 48Dp` 입니다.

​		

### Bottom Navigation Padding

![1](/assets/img/post/branch10/10.png){: width="300" .normal}![1](/assets/img/post/branch10/9.png){: width="300" .normal}

하지만 문제가 아직 남아 있습니다. 앞에서 구한 값을 합쳐서 드래그 핸들과 텍스트 헤더까지 화면 영역에 노출시키는 것은 성공했으나 바텀 네비게이션 영역과 겹칩니다. 이를 통해 기존 설정 높이(드래그 핸들 높이 값 + 텍스트 헤더 높이 값)에 바텀 네비게이션 높이 값까지 추가해야 하는 것을 알았습니다.

![1](/assets/img/post/branch10/7.png){: width="300" .normal}![1](/assets/img/post/branch10/8.png){: width="300" .normal}

사용자가 사용하는 디바이스 내 바텀 네비게이션 타입에 따라 필요한 높이 값이 다른 변수가 존재합니다. 동작 탐색 모드 기반 바텀 네비게이션은 높이가 매우 낮은 반면에 N버튼 탐색 모드는 높이가 매우 큽니다.

```kotlin
WindowInsets.Companion.navigationBars.asPaddingValues().calculateBottomPadding()
```

바텀 네비게이션의 높이는 패딩 값을 의미합니다. 따라서 위 코드를 통해 높이 값을 구할 수 있습니다. 디스플레이 밀도 값과 마찬가지로 `asPaddingValues()` 메서드도 컴포저블이기 때문에 컴포저블 내에서 네비게이션 패딩 값을 요청해야 합니다.

​		

### 높이 조정 완료

```kotlin
BottomSheetScaffold(
    sheetPeekHeight = /* (바텀 네비게이션 높이 + 타이틀 헤더 높이 + 드래그 핸들 높이).Dp */
)
```

위에서 구한 세 가지 값을 바텀 시트 기본 높이로 설정합니다.

![1](/assets/img/post/branch10/11.png){: width="300" .normal}![1](/assets/img/post/branch10/12.png){: width="300" .normal}

바텀 시트의 높이 조정이 완료되었습니다! 이제 모바일 디바이스의 종류, 바텀 네비게이션 모드 상관없이 정확히 타이틀 헤더까지 사용자에게 노출시킬 수 있습니다.

​		

### 버튼 그룹 높이 조정

```kotlin
@Composable
fun MapButtonGroup(
		...
) {
    Box(
        modifier = Modifier.padding(bottom = /* 위에서 설정한 바텀 시트 기본 높이 값 + 10.dp */)
        ...
    ) {
    	...
    }
    ...
}
```

 바텀 시트의 높이를 완벽히 조정했으니, 이제 `content` 영역에 존재하는 버튼 그룹의 높이를 설정할 수 있습니다. 버튼 그룹의 위치 설정은 바텀 패딩 값을 설정해서 해결할 수 있습니다. 바텀 시트의 높이 값 + 간격 값을 설정하면 됩니다. 저는 바텀 시트와 버튼 그룹 사이의 간격을 `10.dp` 로 설정하겠습니다.

![1](/assets/img/post/branch10/13.png){: width="300" .normal}![1](/assets/img/post/branch10/14.png){: width="300" .normal}

​		

# ModalBottomSheet

