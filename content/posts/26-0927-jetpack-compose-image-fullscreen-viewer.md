---
title: "Jetpack Compose 高性能全屏大图预览：多层手势联动、双轨视口裁剪与 Modifier.Node 极致演进 [BY Gemini 3.8 Flash]"
date: 2026-09-27T11:01:53+08:00
slug: "jetpack-compose-image-fullscreen-viewer"
description: "在 Android 应用开发中，实现一个全屏大图预览功能，涵盖列表缩略图共享元素飞入飞出、双指捏合平移缩放、下拉阻尼退场、系统预测性返回手势适配。基于传统的 Chat 方式解决了开发过程中遇到的问题，最后由 Gemini 整理得到此文。"
draft: false
image: 
math: true
license: CC BY-SA 4.0
comments: true
hidden: false

tags:
  - 'jetpack compose'
  - image-viewer
  - architecture
  - performance
categories:
  - Android
---

在 Android 应用开发中，全屏大图预览（涵盖列表缩略图共享元素飞入飞出、双指捏合平移缩放、下拉阻尼退场、系统预测性返回手势适配）是最考验 Compose 渲染管线、手势调度与数学抽象能力的场景之一。

本文由浅入深，首先拆解一个生产级全屏覆盖层（Overlay）的**基础分层架构与几何底座**；随后重点复盘开发过程中暴露的**双重形变（Double Transform）、黑屏闪变（Alpha Timing）、非等比压扁（Squashing Bug）、视口穿模遮挡（Viewport Collision）以及圆角丢失**等五大经典缺陷；最终推导出基于函数复合的 **变换权力交接协议（TransformHandover）** 与彻底告别 `composed` 的现代 **`Modifier.Node` 零开销架构**。

<!--more-->


{{< callout icon="!" type="warning" border="true" >}}

本文整理自开发中与 Gemini 3.8 Flash 的对话，并完全由其生成。

{{< /callout >}}


---

## 1. 系统整体分层架构

要实现不依赖 Activity 切换、丝滑无黑屏的全屏预览，最佳实践是将预览层挂载在根视图的独立 `OverlayLayout` 中。系统逻辑自下而上分为三层：

```
┌─────────────────────────────────────────────────────────────┐
│ 3. 手势与内容消费层 (Gesture & Content Layer)               │
│    DismissibleBox (下拉位移与缩放) + ZoomableBox (双指缩放平移) │
├─────────────────────────────────────────────────────────────┤
│ 2. 动效与几何协调层 (Overlay Layer / Host)                   │
│    OverlaySceneState (管理 animatedRect, animatedClipRect)  │
│    OverlayContentTransitionNode (视口裁剪 + 矩阵变换)        │
├─────────────────────────────────────────────────────────────┤
│ 1. 宿主感知层 (Host Screen Layer)                            │
│    列表视口注册 (overlayViewport) + 缩略图物理探测 (Node)      │
└─────────────────────────────────────────────────────────────┘
```

* **宿主感知层**：宿主列表中的图片通过 `Modifier.Node` 探测并向全局注册自身真实的屏幕物理坐标（`Rect`）、等比宽高比与物理圆角；列表容器则广播自身可视视口（排除 TopBar 与 Composer）。
* **动效协调层**：全局控制器。负责驱动背景渐变（`bgAlpha`）、圆角收敛（`cornerRadius`）、内容几何矩形（`animatedRect`）以及视口安全遮罩（`animatedClipRect`）。
* **手势消费层**：承载高清大图，负责处理内层的双指缩放及外层的下拉退出手势，并在退出时完成向外层动效的无缝权力交接。

---

## 2. 基础实现：几何底座与核心流转管道

### 2.1 保持宽高比的全屏拟合矩形（FitRect 计算）
大图在大图浏览器中通常以 `ContentScale.Fit` 居中充满屏幕。外层的 `animatedRect` 入场终点必须精确等于**图片在全屏空间居中后的实际几何边界（`fitRect`）**，否则在动画结束的瞬间会出现长宽拉伸突变。

```kotlin
object GeometryUtils {
    /** 根据屏幕视口与图片宽高比，计算出居中 Fit 后的物理 Rect */
    fun calculateFitRect(screenBounds: Rect, aspectRatio: Float?): Rect {
        val screenW = screenBounds.width
        val screenH = screenBounds.height
        if (screenW <= 0f || screenH <= 0f) return Rect.Zero

        val targetRatio = aspectRatio ?: (screenW / screenH)
        val containerRatio = screenW / screenH

        val fitW: Float
        val fitH: Float
        if (targetRatio > containerRatio) {
            fitW = screenW
            fitH = screenW / targetRatio
        } else {
            fitW = screenH * targetRatio
            fitH = screenH
        }

        val center = Offset(screenW / 2f, screenH / 2f)
        return Rect(
            left = center.x - fitW / 2f,
            top = center.y - fitH / 2f,
            right = center.x + fitW / 2f,
            bottom = center.y + fitH / 2f,
        )
    }
}
```

### 2.2 双层手势容器的交互底座（Zoomable 与 Dismissible 互斥协同）

在大图交互中，**双指放大** 与 **单指下拉退场** 必须进行严格的手势互斥，否则用户在双指放大平移时会误触发下滑退场。

```kotlin
@Stable
class ZoomableState(
    val minScale: Float = 1f,
    val maxScale: Float = 4f,
) {
    var scale by mutableFloatStateOf(1f)
    var offset by mutableStateOf(Offset.Zero)
    var containerSize by mutableStateOf(IntSize.Zero)

    val isZoomed: Boolean by derivedStateOf { scale > 1.05f }

    fun updatePanZoom(zoomChange: Float, panChange: Offset, centroid: Offset) {
        val newScale = (scale * zoomChange).coerceIn(minScale * 0.8f, maxScale * 1.5f)
        val center = Offset(containerSize.width / 2f, containerSize.height / 2f)
        val newOffset = if (newScale > 1f) {
            offset + panChange - (centroid - center) * (zoomChange - 1f)
        } else Offset.Zero

        scale = newScale
        offset = clampOffset(newOffset, newScale)
    }

    private fun clampOffset(target: Offset, currentScale: Float): Offset {
        if (currentScale <= 1f) return Offset.Zero
        val maxX = containerSize.width * (currentScale - 1f) / 2f
        val maxY = containerSize.height * (currentScale - 1f) / 2f
        return Offset(target.x.coerceIn(-maxX, maxX), target.y.coerceIn(-maxY, maxY))
    }
}
```

---

## 3. 进阶痛点：经典缺陷与成因深潜

当基础底座跑通后，一旦将手势拖拽退出与共享元素退场结合，就会暴露出五大环环相扣的暗坑。

### 3.1 松手瞬间画面“暴缩沉降”（双重变换 / Double Transformation）

```
[拖拽松手瞬间]
1. DismissState (内层): contentScale = 0.8f, swipeOffset = 300px
2. Overlay (外层):   读取当前视觉矩形 (内含 0.8 缩放与 300px 偏移) -> animatedRect.snapTo(...)
────────────────────────────────────────────────────────────
❌ 叠加结果: Scale = 0.8 * 0.8 = 0.64f, OffsetY = 300 + 300 = 600px! (瞬间暴缩暴跌)
```

* **本质原因**：外层的 `animatedRect` 已经代理了手势发生后的几何大小与屏幕绝对坐标，但内层 `DismissibleBox` 矩阵变换仍在持续生效，导致内外层进行了二次叠乘计算。

### 3.2 松手瞬间屏幕“先黑再亮”（Alpha 时序倒挂）

* 拖拽期间，背景随着下拉距离减小（比如淡化为 `0.5f`）。
* 松手时为了解决双重变换，调用了 `dismissState.reset()`。
* 但 `reset()` 把 `activeSource` 变为了 `None`，导致 `dismissState.backgroundAlpha` **瞬间跳回了 `1.0f`（纯黑）**。
* 退场动效在 `reset()` 之后才去读取当前 Alpha，读到了 `1.0f` 并执行 `bgAlpha.snapTo(1.0f)`。
* **现象**：半透明背景瞬间被盖上一层纯黑幕，下一毫秒才从纯黑慢慢淡化退出。

### 3.3 边缘遮挡的“穿模”与“压扁”困局（The Squashing & Viewport Bug）

当缩略图位于列表边缘（例如一部分滚到了 TopBar 标题栏或底部 Composer 输入框下方）：
* **微信的做法（粗糙穿模）**：以完整图片坐标起飞，起飞和落下的瞬间，**图片会直接骑在 TopBar 或 Composer 脸上**。
* **错误优化引发的“压扁”惨案**：
  若试图用相交矩形（`visibleRect`）来做动画起点，由于高度被 TopBar 削去了一截，导致：
  $$ScaleX = \frac{W_{visible}}{W_{fit}}, \quad ScaleY = \frac{H_{visible}}{H_{fit}} \quad (ScaleX \neq ScaleY)$$
  整张图片在开始和结束时被严重压缩压扁（Squash）。
* **Compose API 隐藏暗坑**：
  直接调用 `coords.boundsInWindow()`，其底层声明是：
  ```kotlin
  fun LayoutCoordinates.boundsInWindow(clipBounds: Boolean = true): Rect
  ```
  `clipBounds` 默认是 `true`！这意味着 Compose 会自动把它被父级滚动容器裁切后的残缺矩形返回，导致你误以为拿到了完整大小，直接引发非等比形变。

### 3.4 圆角动画“全程直角”的单位陷阱（Dp vs Px）

如果在注册缩略图时直接调用：
```kotlin
cornerRadius = BUBBLE_CARD_ROUNDED_CORNER_RADIUS_IN_DP.toFloat() // 传入了 16f
```
在 Canvas 中，`CornerRadius(radius)` 接收的是**物理像素（px）**。在 `density = 3.0` 的现代屏幕上，`16.dp` 应当是 `48px`。直接传 `16f` 相当于只有 `5.3dp`；叠加 `FastOutSlowInEasing` 急剧下落曲线，起飞 50ms 后圆角就衰减到了 `1dp` 以下，肉眼观察全程全是直角。

---

## 4. 核心架构：变换权力交接协议（TransformHandover）

针对双重变换问题，如果采用硬编码扁平计算：
$$\text{TotalOffset} = \text{DismissOffset} + (\text{ZoomOffset} \times \text{DismissScale})$$
外层必须知道内层的嵌套顺序，且无法扩展多图滑动或多层级手势。

### 4.1 数学本质：函数复合（Function Composition）
图形学中，多层嵌套的本质就是**函数的串联复合**：
$$\text{ScreenRect} = \text{Layer}_{外}(\ \text{Layer}_{内}(\text{fitRect})\ )$$

只要每一层只对自己负责（将输入的 `Rect` 映射为形变后的输出 `Rect`，并在交接后重置自身），无论嵌套多少层、顺序如何调整，都能通过链式折叠（Fold）自动完成精确计算。

```kotlin
interface TransformHandoverLayer {
    /** 将上层输入的 Rect 映射成本层的视觉 Rect */
    fun mapRect(input: Rect): Rect
    /** 归一化重置本层手势状态（擦除脏数据，交出控制权） */
    fun reset()
}

fun Rect.applyTransform(scale: Float, offset: Offset): Rect {
    if (scale == 1f && offset == Offset.Zero) return this
    val newCenter = this.center + offset
    val newW = this.width * scale
    val newH = this.height * scale
    return Rect(
        left = newCenter.x - newW / 2f,
        top = newCenter.y - newH / 2f,
        right = newCenter.x + newW / 2f,
        bottom = newCenter.y + newH / 2f,
    )
}
```

```kotlin
@Stable
class ZoomableState(...) : TransformHandoverLayer {
    override fun mapRect(input: Rect): Rect = input.applyTransform(scale, offset)
    override fun reset() {
        scale = 1f
        offset = Offset.Zero
    }
}

@Stable
class DismissState(...) : TransformHandoverLayer {
    override fun mapRect(input: Rect): Rect = input.applyTransform(contentScale, contentOffset)
    override fun reset() {
        activeSource = DismissSource.None
        swipeOffset = Offset.Zero
    }
}
```

### 4.2 链式交接协调器（Chain Coordinator）
通过函数折叠与批量重置，一键完成“采样快照 + 瞬间洗净”：
```kotlin
class TransformHandoverChain(
    private val layers: List<TransformHandoverLayer>,
) {
    fun captureVisualRectAndReset(baseRect: Rect): Rect {
        // 1. 由内向外 fold 变换：Zoom -> Dismiss
        val finalVisualRect = layers.fold(baseRect) { currentRect, layer ->
            layer.mapRect(currentRect)
        }
        // 2. 采样完成后，原子级洗净所有内层状态
        layers.forEach { it.reset() }
        return finalVisualRect
    }
}
```

---

## 5. 双轨几何动效与视口安全裁剪（Dual-Geometry Pipeline）

要达到 Telegram / iOS 原生相册般毫无破绽的退场效果，核心思想是**将“内容几何变换”与“视口遮罩裁剪”物理剥离**。

```
              animatedRect (图片完整排版矩形: 200x300, 等比缩放)
            ┌────────────────────────┐
 TopBar ─── ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ (TopBar 遮挡线)
            │                        │
            │  animatedClipRect      │
            │  (露头可见窗口: 200x200)│
            │                        │
            └────────────────────────┘
```

1. **`bounds`（真实物理矩形）**：
   必须通过 `coords.boundsInWindow(clipBounds = false)` 或 `coords.localToWindow(Offset.Zero) + coords.size` 取得。即使有一半在 TopBar 背后，高度依然是完好无损的 `300`，保证 $ScaleX == ScaleY$，**内容绝对不被压扁**。
2. **`clipBounds`（视口安全遮罩）**：
   通过真实列表视口求交集 `fullRect.intersect(viewportBounds)`。遮罩在起飞时卡在 TopBar 下沿（切平），飞向全屏时遮罩展开至 `windowBounds`（完整展现）。

---

## 6. 渲染时序与调度保障

### 6.1 先采样快照，后状态重置
在退出触发函数中，必须严格保证**先采样当前的 Alpha 快照，再执行状态归一重置**：

```kotlin
fun triggerDismiss(...) {
    if (transitionPhase == OverlayTransitionPhase.Dismissing) return

    // 🌟 1. 先采样当前的真实透明度快照（此时 dismissState 尚未 reset）
    val currentVisualAlpha = contentAlphaProvider?.invoke() ?: bgAlpha.value
    transitionPhase = OverlayTransitionPhase.Dismissing

    // 🌟 2. 执行几何计算（内部会触发 captureVisualRectAndReset 抹除内层状态）
    val startVRect = customTransform?.invoke(fitRect)
        ?: dismissTransformHandoverProvider?.invoke()?.captureVisualRectAndReset(fitRect)
        ?: fitRect

    // 🌟 3. UNDISPATCHED 同步启动动效
    coroutineScope.launch(start = CoroutineStart.UNDISPATCHED) {
        bgAlpha.snapTo(currentVisualAlpha) // 衔接当前透明度，绝不弹回 1.0f
        animatedRect.snapTo(startVRect)

        coroutineScope {
            launch { bgAlpha.animateTo(0f, tween(260)) }
            launch { animatedRect.animateTo(targetThumbnail.bounds, tween(260)) }
            launch { animatedClipRect.animateTo(targetThumbnail.clipBounds, tween(260)) }
        }
        onDismissFinished()
    }
}
```

### 6.2 为什么必须使用 `CoroutineStart.UNDISPATCHED`？

Compose 动画中的 `Animatable.snapTo()` 是一个挂起函数（`suspend`），它内部依赖 `MutatorMutex` 互斥锁。如果使用默认的 `coroutineScope.launch`，任务会被推入 Android 主线程的 `Handler/Looper` 队列做微任务排队。

此时在当前调用栈中，内层的 `reset()` 已经把状态归零了。若恰好赶上屏幕 VSYNC 刷新，用户会看到**整整 1 帧“画面弹回原位”，下一帧才开始缩回**的严重跳变。

**`UNDISPATCHED` 的作用**：
强制协程体内的第一段代码（直到第一个真正的挂起点 `animateTo` 之前）直接在当前主线程调用栈中同步执行。它保证了：
$$\text{dismissState.reset()} \quad \text{与} \quad \text{animatedRect.snapTo(startVRect)}$$
在**同一个主线程事件周期内完成提交**，Compose 快照系统在下一帧 VSYNC 打包渲染，彻底消灭 1 帧闪动。

---

## 7. 极致性能收敛：全面迁移 `Modifier.Node`

为了消灭 `Modifier.composed` 带来的隐式重组槽位与 GC 开销，我们将所有高频交互封装升级为 Compose 1.5+ 原生 Node。

### 7.1 宿主视口捕获：`Modifier.overlayViewport`
列表容器通过单例 Node 广播自身位置，自动解决兄弟子组件跨层传递痛点：

```kotlin
fun Modifier.overlayViewport(): Modifier = this.then(OverlayViewportElement)

private object OverlayViewportElement : ModifierNodeElement<OverlayViewportNode>() {
    override fun create(): OverlayViewportNode = OverlayViewportNode()
    override fun update(node: OverlayViewportNode) {}
    override fun hashCode(): Int = "OverlayViewportElement".hashCode()
    override fun equals(other: Any?): Boolean = other === this
}

private class OverlayViewportNode :
    Modifier.Node(),
    CompositionLocalConsumerModifierNode,
    GlobalPositionAwareModifierNode {

    override fun onGloballyPositioned(coordinates: LayoutCoordinates) {
        val registry = currentValueOf(LocalOverlayLayoutThumbnailRegistry)
        registry?.updateViewport(coordinates)
    }
}
```

### 7.2 缩略图零开销物理注册：`RecordThumbnailBoundsNode`
**核心黑魔法**：让 Node 自身直接实现 `ThumbnailMetadataProvider` 接口（`this` 即 Provider），属性变更原地修改，实现绝对的**零堆内存分配（Zero GC）**：

```kotlin
interface OverlayLayoutThumbnailMetadataProvider {
    val coordinates: LayoutCoordinates?
    val cornerRadius: Float
    val aspectRatio: Float?
}

fun Modifier.recordThumbnailBounds(
    sharedElementId: Any,
    cornerRadius: Dp = 0.dp,
    aspectRatio: Float?,
): Modifier = this.then(
    RecordThumbnailBoundsElement(sharedElementId, cornerRadius, aspectRatio)
)

private data class RecordThumbnailBoundsElement(
    val sharedElementId: Any,
    val cornerRadius: Dp,
    val aspectRatio: Float?,
) : ModifierNodeElement<RecordThumbnailBoundsNode>() {
    override fun create() = RecordThumbnailBoundsNode(sharedElementId, cornerRadius, aspectRatio)
    override fun update(node: RecordThumbnailBoundsNode) {
        node.update(sharedElementId, cornerRadius, aspectRatio)
    }
}

private class RecordThumbnailBoundsNode(
    var sharedElementId: Any,
    var cornerRadiusDp: Dp,
    override var aspectRatio: Float?,
) : Modifier.Node(),
    OverlayLayoutThumbnailMetadataProvider,
    GlobalPositionAwareModifierNode,
    CompositionLocalConsumerModifierNode {

    override var coordinates: LayoutCoordinates? = null
        private set

    override var cornerRadius: Float = 0f
        private set

    override fun onAttach() {
        updateCornerRadiusPx()
        registerToRegistry(sharedElementId)
    }

    override fun onGloballyPositioned(coordinates: LayoutCoordinates) {
        this.coordinates = coordinates
        registerToRegistry(sharedElementId)
    }

    fun update(sharedElementId: Any, cornerRadiusDp: Dp, aspectRatio: Float?) {
        val oldKey = this.sharedElementId
        this.sharedElementId = sharedElementId
        this.cornerRadiusDp = cornerRadiusDp
        this.aspectRatio = aspectRatio

        if (oldKey != sharedElementId) {
            unregisterFromRegistry(oldKey)
            registerToRegistry(sharedElementId)
        }
        updateCornerRadiusPx()
    }

    override fun onDetach() {
        unregisterFromRegistry(sharedElementId)
        this.coordinates = null
    }

    override fun onReset() {
        // 响应 LazyColumn Item 节点池复用
        unregisterFromRegistry(sharedElementId)
        this.coordinates = null
    }

    private fun updateCornerRadiusPx() {
        if (isAttached) {
            val density = currentValueOf(LocalDensity)
            // 🌟 自动完成 Dp -> Px 计算 (16.dp -> 48px)
            this.cornerRadius = with(density) { cornerRadiusDp.toPx() }
        }
    }

    private fun registerToRegistry(key: Any) {
        val registry = currentValueOf(LocalOverlayLayoutThumbnailRegistry) ?: return
        registry.register(key, this) // 传自身，零对象构建
    }

    private fun unregisterFromRegistry(key: Any) {
        val registry = currentValueOf(LocalOverlayLayoutThumbnailRegistry) ?: return
        registry.unregister(key)
    }
}
```

### 7.3 全屏内容统一转场驱动：`OverlayContentTransitionNode`
采用 `DrawModifierNode`，一条管线按序串联视口遮罩、圆角及矩阵变换。并处理 `@DrawScopeMarker` 的显式 Receiver 调用：

```kotlin
fun Modifier.overlayContentTransition(state: OverlaySceneState): Modifier =
    this.then(OverlayContentTransitionElement(state))

private data class OverlayContentTransitionElement(
    val state: OverlaySceneState,
) : ModifierNodeElement<OverlayContentTransitionNode>() {
    override fun create() = OverlayContentTransitionNode(state)
    override fun update(node: OverlayContentTransitionNode) = node.update(state)
}

private class OverlayContentTransitionNode(
    var state: OverlaySceneState,
) : Modifier.Node(), DrawModifierNode {

    // 🌟 Node 级别常驻 Path，避免 Draw 阶段重复 new 对象
    private val roundPath = Path()

    fun update(state: OverlaySceneState) {
        if (this.state != state) {
            this.state = state
            invalidateDraw()
        }
    }

    override fun ContentDrawScope.draw() {
        if (state.transitionPhase.isTransitioning && state.isGeometryMode) {
            val clip = state.animatedClipRect.value
            val currentBounds = state.animatedRect.value
            val radius = state.cornerRadius.value
            val fitRect = state.fitRect

            // 1. 视口安全裁剪（绝不入侵 TopBar / Composer）
            clipRect(
                left = clip.left,
                top = clip.top,
                right = clip.right,
                bottom = clip.bottom,
            ) {
                // 2. 缩略图物理圆角裁剪
                if (radius > 0f) {
                    roundPath.rewind()
                    roundPath.addRoundRect(
                        RoundRect(
                            rect = currentBounds,
                            cornerRadius = CornerRadius(radius),
                        )
                    )
                    clipPath(roundPath) {
                        drawTransformedContent(fitRect, currentBounds)
                    }
                } else {
                    drawTransformedContent(fitRect, currentBounds)
                }
            }
        } else {
            // Settled 阶段：完全放开，双指放大与手势拖拽无任何遮挡
            drawContent()
        }
    }

    private fun ContentDrawScope.drawTransformedContent(fitRect: Rect, currentBounds: Rect) {
        if (fitRect.width > 0f && fitRect.height > 0f) {
            withTransform({
                translate(
                    left = currentBounds.center.x - fitRect.center.x,
                    top = currentBounds.center.y - fitRect.center.y,
                )
                scale(
                    scaleX = currentBounds.width / fitRect.width,
                    scaleY = currentBounds.height / fitRect.height,
                    pivot = fitRect.center,
                )
            }) {
                // 🌟 消除 @DrawScopeMarker 作用域警告
                this@drawTransformedContent.drawContent()
            }
        } else {
            drawContent()
        }
    }
}
```

---

## 8. 完整生产级实现参考

### 8.1 Registry 探测与视口求交
```kotlin
class OverlayLayoutThumbnailRegistry {
    private val activeProviders = mutableMapOf<Any, OverlayLayoutThumbnailMetadataProvider>()
    var viewportCoordinates by mutableStateOf<LayoutCoordinates?>(null)

    fun updateViewport(coordinates: LayoutCoordinates) {
        this.viewportCoordinates = coordinates
    }

    fun register(key: Any, provider: OverlayLayoutThumbnailMetadataProvider) {
        activeProviders[key] = provider
    }

    fun unregister(key: Any) {
        activeProviders.remove(key)
    }

    fun queryVisibleThumbnail(itemKey: Any, windowBounds: Rect): ThumbnailTarget? {
        val provider = activeProviders[itemKey] ?: return null
        val coords = provider.coordinates ?: return null
        if (!coords.isAttached) return null

        // 1. 真实视口边界（扣除 TopBar/Composer）
        val actualViewport = viewportCoordinates
            ?.takeIf { it.isAttached }
            ?.boundsInWindow()
            ?: windowBounds

        // 🌟 2. 传 clipBounds = false 取得未被父级削减的真实物理尺寸
        val fullRect = coords.boundsInWindow(clipBounds = false)
        if (fullRect.width <= 0f || fullRect.height <= 0f) return null

        if (!fullRect.overlaps(actualViewport)) return null

        // 3. 计算实际可视交集
        val visibleRect = fullRect.intersect(actualViewport)

        return ThumbnailTarget(
            bounds = fullRect,        // 200x300 完好尺寸 (scaleX == scaleY)
            clipBounds = visibleRect, // 200x200 露头视口 (由 Canvas 裁切)
            cornerRadius = provider.cornerRadius,
            aspectRatio = provider.aspectRatio,
        )
    }
}
```

### 8.2 顶层 OverlayLayout 容器
```kotlin
@Composable
fun OverlayLayout(
    state: OverlaySceneState,
    modifier: Modifier = Modifier,
    content: @Composable OverlayLayoutScope.() -> Unit,
) {
    val scope = remember(state) { OverlayLayoutScopeImpl(state) }

    Box(modifier = modifier.fillMaxSize()) {
        // 背景黑色遮罩
        Box(
            modifier = Modifier
                .fillMaxSize()
                .graphicsLayer { alpha = state.currentBackgroundAlpha }
                .background(Color.Black)
                .pointerInput(Unit) {
                    detectTapGestures(
                        onTap = { state.onBgTap?.invoke() },
                        onDoubleTap = { offset -> state.onBgDoubleTap?.invoke(offset) },
                    )
                }
        )

        // 🌟 内容承载容器：单个 Node 修饰符统一闭环
        Box(
            modifier = Modifier
                .fillMaxSize()
                .overlayContentTransition(state)
        ) {
            scope.content()
        }
    }
}
```

### 8.3 UI 消费层实现（零污染）
```kotlin
@Composable
fun OverlayLayoutScope.ImageFullScreenViewer(
    overlayTransitionElementId: String,
    imageUrl: Any,
    modifier: Modifier = Modifier,
) {
    val zoomState = rememberZoomableState()
    val dismissState = rememberDismissState()

    val handoverChain = remember(zoomState, dismissState) {
        TransformHandoverChain(listOf(zoomState, dismissState))
    }

    val dismissAction = remember(overlayTransitionElementId) {
        { this.dismiss(overlayTransitionElementId) }
    }

    BackHandler(enabled = true, onBack = dismissAction)

    DismissibleBox(
        state = dismissState,
        enabled = { !zoomState.isZoomed },
        onDismissRequest = dismissAction,
        modifier = modifier
            .fillMaxSize()
            .overlayInteractiveTarget(
                itemKey = overlayTransitionElementId,
                backgroundAlpha = { dismissState.backgroundAlpha },
                dismissTransformHandover = { handoverChain },
            )
    ) {
        ZoomableBox(
            state = zoomState,
            onSingleTap = dismissAction,
            modifier = Modifier.fillMaxSize(),
        ) {
            AsyncImage(
                model = imageUrl,
                contentDescription = null,
                contentScale = ContentScale.Fit,
                modifier = Modifier.fillMaxSize(),
            )
        }
    }
}
```

---

## 9. 核心工程哲学总结

1. **权力移交定律（Hand-off Invariant）**：
   从“手势驱动”切换为“动效驱动”时，外层接管全部几何形变的同时，内层本地矩阵必须在同一帧归零（Identity）。
2. **函数复合解耦层级**：
   摒弃硬编码扁平缩放偏移算式，利用 `TransformHandoverLayer` 协议和链式折叠（Fold）流转变换，让任意多层手势天然解耦。
3. **内容形变与视口遮罩物理剥离**：
   图片自身的等比缩放矩阵（Bounds）永远尊重真实原始物理尺寸，外界的一切遮挡（TopBar、Composer、键盘）纯粹作为几何遮罩（ClipBounds）施加，从根源上杜绝非等比压扁。
4. **时序与快照严明**：
   在抹除内层状态之前建立快照缓存，借助 `CoroutineStart.UNDISPATCHED` 消除主线程执行栈与微任务队列的延迟空档，杜绝黑屏与 1 帧跳动。
5. **全面奔赴 `Modifier.Node` 零开销架构**：
   * 让 Node 自身直接充当数据 Provider，杜绝任何中间包装对象分配。
   * 列表滚动项中严禁使用 `Modifier.composed`，将参数同步降维为纳秒级的对象字段赋值。
