---
title: Android Compose - Material Design을 이용한 Bottom Sheet 구현하기
author: yoonmin
date: 2025-05-17 00:00:00 +0900
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

## 완성된 바텀 시트 동작 최종 확인하기

![1](/assets/img/post/branch10/15.gif){: width="300" .normal}

​		

# ModalBottomSheet

```kotlin
@Composable
@ExperimentalMaterial3Api
fun ModalBottomSheet(
    onDismissRequest: () -> Unit,
    modifier: Modifier = Modifier,
    sheetState: SheetState = rememberModalBottomSheetState(),
    sheetMaxWidth: Dp = BottomSheetDefaults.SheetMaxWidth,
    shape: Shape = BottomSheetDefaults.ExpandedShape,
    containerColor: Color = BottomSheetDefaults.ContainerColor,
    contentColor: Color = contentColorFor(containerColor),
    tonalElevation: Dp = 0.dp,
    scrimColor: Color = BottomSheetDefaults.ScrimColor,
    dragHandle: @Composable (() -> Unit)? = { BottomSheetDefaults.DragHandle() },
    contentWindowInsets: @Composable () -> WindowInsets = { BottomSheetDefaults.windowInsets },
    properties: ModalBottomSheetProperties = ModalBottomSheetDefaults.properties,
    content: @Composable ColumnScope.() -> Unit,
) {
    val scope = rememberCoroutineScope()
    val animateToDismiss: () -> Unit = {
        if (sheetState.anchoredDraggableState.confirmValueChange(Hidden)) {
            scope
                .launch { sheetState.hide() }
                .invokeOnCompletion {
                    if (!sheetState.isVisible) {
                        onDismissRequest()
                    }
                }
        }
    }
    val settleToDismiss: (velocity: Float) -> Unit = {
        scope
            .launch { sheetState.settle(it) }
            .invokeOnCompletion { if (!sheetState.isVisible) onDismissRequest() }
    }

    val predictiveBackProgress = remember { Animatable(initialValue = 0f) }

    ModalBottomSheetDialog(
        properties = properties,
        onDismissRequest = {
            if (sheetState.currentValue == Expanded && sheetState.hasPartiallyExpandedState) {
                // Smoothly animate away predictive back transformations since we are not fully
                // dismissing. We don't need to do this in the else below because we want to
                // preserve the predictive back transformations (scale) during the hide animation.
                scope.launch { predictiveBackProgress.animateTo(0f) }
                scope.launch { sheetState.partialExpand() }
            } else { // Is expanded without collapsed state or is collapsed.
                scope.launch { sheetState.hide() }.invokeOnCompletion { onDismissRequest() }
            }
        },
        predictiveBackProgress = predictiveBackProgress,
    ) {
        Box(modifier = Modifier.fillMaxSize().imePadding().semantics { isTraversalGroup = true }) {
            Scrim(
                color = scrimColor,
                onDismissRequest = animateToDismiss,
                visible = sheetState.targetValue != Hidden,
            )
            ModalBottomSheetContent(
                predictiveBackProgress,
                scope,
                animateToDismiss,
                settleToDismiss,
                modifier,
                sheetState,
                sheetMaxWidth,
                shape,
                containerColor,
                contentColor,
                tonalElevation,
                dragHandle,
                contentWindowInsets,
                content
            )
        }
    }
    if (sheetState.hasExpandedState) {
        LaunchedEffect(sheetState) { sheetState.show() }
    }
}
```

다음으로 모달 버전 바텀 시트입니다. 스탠다드 버전의 바텀 시트와는 다르게 배경 영역이 필요하지 않습니다. 그래서 배경 영역을 채울 수 있는 컴포저블 파라미터가 아닌 단순 배경 색상을 정할 수 있는 파라미터만 존재합니다. 그것이 `scrimColor` 입니다.

​		

## 주요 파라미터

### onDismissRequest

유저가 바텀시트 외부를 클릭했을 때 실행됩니다. 시트가 숨겨진 상태로 애니메이션된 후 호출됩니다. 보통 바텀 시트는 특정 조건이 충족되면 `show()` 혹은 `hide()` 됩니다. 기능 요구사항에 따르면 특정 지도 아이템의 오른쪽 상단 옵션 버튼에 의해 메뉴 바텀 시트가 보여야 합니다.

따라서 옵션 버튼이 클릭됐을 때를 구분하는 논리형 상태 값을 하나 추가하여 `true` 라면 바텀 시트 컴포저블을 생성, `false` 라면 바텀 시트를 닫게 설계합니다. 설계 방식은 밑에서 자세히 풀어 보겠습니다.

​		

### sheetState

스탠다드 버전 바텀 시트와 마찬가지로 모달 버전도 시트 상태 객체를 지니고 있습니다. 이 상태 값으로 바텀 시트를 조건에 따라 열고 닫을 수 있습니다.

```kotlin
@ExperimentalMaterial3Api
enum class SheetValue {
    /** The sheet is not visible. */
    Hidden,

    /** The sheet is visible at full height. */
    Expanded,

    /** The sheet is partially visible. */
    PartiallyExpanded,
}
```

바텀 시트 상태는 세 가지로 정의되어 있습니다. 각각 완전히 보이지 않는 상태, 열린 상태, 부분적으로 열린 상태입니다. 그래서 바텀 시트를 열고 닫을 때 위 세 가지 상태가 순환합니다.

​		

### scrimColor

바텀 시트 배경 영역의 색상을 정하는 파라미터입니다. 다이얼로그의 배경이 어두운 것처럼 모달 바텀 시트도 배경 영역을 건들 수 없기 때문에 배경이 어둡습니다. 배경을 어둡게 처리해야 사용자가 배경 영역은 건들 수 없음을 인지하니까요.

​		

## 바텀 시트 열고 닫는 동작 구체화

```kotlin
@Composable
fun HomeScreen(
    mapViewModel: MapViewModel = hiltViewModel(),
) {
      Box(
        modifier = Modifier
            .background(AppServiceColors.white)
            .fillMaxSize()
    ) {
        ...
        
        MapList(
            mapList = mapFilterUiState.value.mapList,
            onClickMenuOption = { map -> mapViewModel.initMenuBottomSheet(map) }
				)
        
        ...
    }
}
```

최상위 컴포저블(스크린 컨테이너)에 지도 목록을 리스트로 보여줄 컴포저블을 호출합니다. 최상위 컴포저블 영역은 뷰 모델 참조 변수를 가지고 있기 때문에 이곳에 뷰 모델에 특정 작업을 요청하는 동작을 정의해서 하위 컴포저블에 전달합니다.

지도 아이템 옵션 버튼을 클릭하면 해당 지도 아이템 정보가 담긴 메뉴 바텀 시트를 출력하는 트리거(상태 값)는 뷰 모델이 관리하고 있습니다.

​		

### MapList

```		kotlin
LazyColumn(
    modifier = modifier,
    verticalArrangement = Arrangement.spacedBy(12.dp),
    contentPadding = PaddingValues(bottom = navigationPadding)
) {
    items(
        items = mapList,
        key = { it.id }
    ) { map ->
        MapItem(
            modifier = Modifier.fillMaxWidth(),
            item = map,
            onClickOption = { onClickMenuOption(map) }
        )
    }
}
```

`MapList` 컴포저블은 지도 목록을 `LazyColumn` 으로 표시하고 각 아이템마다 위와 같이 단일 지도 정보와 옵션 버튼 클릭 이벤트를 인자로 전달합니다. 

​		

### MapItem

```kotlin
MapItem(
	mapItem: MapModel,
	onClickOption: () -> Unit,
	...
)
```

실제 지도 목록의 아이템을 이 컴포저블에 정의합니다. 클릭할 수 있는 옵션 버튼 하나를 제외하고 모두 지도 아이템에 대한 정보를 표시하기 때문에 필수 파라미터는 위 두 개로 충분합니다. 최상위 컴포저블에서 정의한 옵션 클릭 이벤트를 옵션 아이콘의 `onClick` 인자로 전달합니다.

​		

### In ViewModel

```kotlin
fun initMenuBottomSheet(selectedMapItem: MapModel? = null) {
    viewModelScope.launch {
        _mapMenuUiState.update { it.copy(mapItem = selectedMapItem) }
    }
}
```

뷰 모델은 트리거 역할을 하는 논리형 상태 값을 초기화하여 이를 최상위 컴포저블에 알립니다. 

```kotlin
@Composable
fun HomeScreen(
    mapViewModel: MapViewModel = hiltViewModel(),
) {
  	val mapMenuUiState = mapViewModel.mapMenuUiState.collectAsStateWithLifecycle()
  	
  	...
  
    mapMenuUiState.value.mapItem?.let {
        MapItemOptionBottomSheet(
            ...
        )
    }
  
  	...
}
```

컴포저블은 업데이트된 뷰 모델의 트리거(지도 아이템 정보) 값을 받습니다. 이제 `mapItem` 이 `null` 이 아니기 때문에 지도 메뉴 바텀 시트 역할을 하는 `MapItemOptionBottomSheet` 컴포저블이 생성됩니다.

```kotlin
MapItemOptionBottomSheet(
    onDismissRequest = { mapViewModel.initMenuBottomSheet() }
    ...
)
```

바텀 시트를 닫을 때는 생성할 때와 마찬가지로 뷰 모델 내 트리거(상태 값)를 업데이트 해줘야 합니다. 닫을 때는 `null` 로 설정하여 사라지게 합니다.

​		

### 바텀 시트가 없어질 때 문제점

```kotlin
// 바텀 시트를 생성할 때
if (sheetState.hasExpandedState) {
    LaunchedEffect(sheetState) { sheetState.show() }
}

// sheetState의 show() 내부 코드
suspend fun show() {
    val targetValue =
        when {
            hasPartiallyExpandedState -> PartiallyExpanded
            else -> Expanded
        }
    animateTo(targetValue)
}
```

Material Design에서 제공하는 바텀 시트를 사용하면 한 가지 디테일을 신경써야 합니다. 생성할 때는 아래에서 위로 올라오는 슬라이드 애니메이션이 연출됩니다. 그러나 바텀 시트를 **제거할 때 주의할 점**이 있습니다.

모달 바텀 시트를 제거하는 방법은 **배경 영역을 터치하여 포커스를 바텀 시트로부터 뺏어서 닫게 만드는 방법**과 직접 **바텀 시트 내부에 버튼을 구현하여 닫게 만드는 방법**이 있습니다.

```kotlin
// 포커스가 배경으로 옮겨질 때
val animateToDismiss: () -> Unit = {
    if (sheetState.anchoredDraggableState.confirmValueChange(Hidden)) {
        scope
            .launch { sheetState.hide() }
            .invokeOnCompletion {
                if (!sheetState.isVisible) {
                    onDismissRequest()
                }
            }
    }
}

// sheetState의 hide() 내부 코드
suspend fun hide() {
    check(!skipHiddenState) {
        "Attempted to animate to hidden when skipHiddenState was enabled. Set skipHiddenState" +
            " to false to use this function."
    }
    animateTo(Hidden)
}
```

전자는 내부적으로 이미 구현되어 있어서 확장때와 마찬가지로 슬라이드 애니메이션이 적용됩니다. 그래서 이 방법은 문제가 되지 않습니다.

![1](/assets/img/post/branch10/16.gif){: width="300" .normal}

그러나 후자는 애니메이션이 적용되지 않아서 위 이미지처럼 제거될 때 굉장히 부자연스럽습니다. 바텀 시트 내용물의 잔상이 보이는 것과 생성할 때와 제거할 때 동작의 통일성이 없는 것이 마음에 들지 않습니다.

​		

## 직접 애니메이션 적용하기

이것을 해결하려면 직접 애니메이션을 추가해야 합니다. 다행이도 직접 애니메이션 관련 객체나 컴포저블을 가져올 필요는 없고 `SheetState` 객체에서 `hide` 를 직접 호출하면 됩니다. 하이드 메서드 자체에 애니메이션 코드가 있으니까요.

```kotlin
val animateToDismiss: () -> Unit = {
    if (sheetState.anchoredDraggableState.confirmValueChange(Hidden)) {
        scope
            .launch { sheetState.hide() } // 1. 애니메이션 효과와 함께 바텀 시트를 숨김
            .invokeOnCompletion { // 2. hide() 실행이 완료되면
                if (!sheetState.isVisible) { // 3. 바텀 시트가 현재 숨겨진 상태인지 확인
                    onDismissRequest() // 4. 직접 필요한 작업 진행
                }
            }
    }
}
```

그리고 내부 코드에 이미 힌트가 존재합니다. 위의 모달 바텀 시트 내부 코드를 보면 바텀 시트를 애니메이션 효과와 함께 하이드한 뒤, 필요한 작업을 수행합니다. 이런 식으로 바텀 시트 내 취소 버튼 클릭 이벤트 동작을 정의하면 되겠습니다.

```kotlin
suspend fun hide() {
    check(!skipHiddenState) {
        "Attempted to animate to hidden when skipHiddenState was enabled. Set skipHiddenState" +
            " to false to use this function."
    }
    animateTo(Hidden)
}
```

주의할 점은 `hide` 동작은 `suspend` 함수이기 때문에 코루틴 영역에서 실행되어야 합니다. 그래서 코루틴 영역을 불러올 수 있는 영역에서 해당 동작을 정의하시기 바랍니다.

![1](/assets/img/post/branch10/17.gif){: width="300" .normal}

이제 바텀 시트의 취소 버튼을 눌러서 닫아도 자연스럽게 닫힙니다. 움직임이 굉장히 자연스러워졌네요. 위처럼 특정 조건에서 바텀 시트를 직접 닫아야 하는 경우 참고하시면 좋을 것 같습니다.

​		

## 완성된 바텀 시트 동작 최종 확인하기

![1](/assets/img/post/branch10/18.gif){: width="300" .normal}

​		

# 정리

지금까지 Material Design에서 제공하는 두 가지 바텀 시트 컴포저블을 알아보고 응용해 봤습니다. 스탠다드 바텀 시트는 배경 영역과 바텀 시트 영역을 동시에 정의하고 서로 상호작용할 때 사용합니다. 반면에 모달 바텀 시트는 바텀 시트만 포커스를 고정합니다. 배경은 다이얼로그처럼 사용하지 않습니다.

단순한 구조의 바텀 시트를 구현해야 한다면 이 두 가지 바텀 시트면 충분하다고 생각합니다. 내부 구조도 파악하기 쉽고 기본적인 커스텀도 어렵지 않게 할 수 있습니다.

하지만 조금 복잡한 디자인 요구사항을 따라야 한다면 직접 바텀 시트를 구현하는 것이 좋을수도 있습니다. 기능 요구사항과 마테리얼 디자인이 제공하는 두 바텀 시트의 변경 가능한 요소를 잘 비교하여 판단하시면 될 것 같습니다.

