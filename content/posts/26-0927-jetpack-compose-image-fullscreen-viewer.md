---
title: "Jetpack Compose 高性能全屏大图预览：多层手势联动与无缝退出动画架构演进 [BY Gemini 3.8 Flash]"
date: 2026-09-27T11:01:53+08:00
slug: "jetpack-compose-image-fullscreen-viewer"
draft: false
image: 
math: false
license: CC BY-SA 4.0
comments: true
hidden: false

tags:
  - 'jetpack compose'
  - image-viewer
categories:
  - Android
---

{{< callout icon="!" type="warning" border="true" >}}

本文整理自开发中与 Gemini 3.8 Flash 的对话，并完全由其生成。

{{< /callout >}}

在 Android 应用开发中，全屏大图预览（涵盖列表缩略图共享元素飞入飞出、双指捏合平移缩放、下拉阻尼退场、系统预测性返回手势适配）是最考验 Compose 渲染管线、手势调度与数学抽象能力的场景之一。本文将由浅入深，首先拆解一个生产级全屏覆盖层（Overlay）的**基础分层架构与几何底座**，随后重点复盘多层手势叠加时触发的**双重形变（Double Transform）与黑屏闪变难题**，最终推导出基于函数复合的 **变换权力交接协议（TransformHandover）** 与现代 `Modifier.Node` 的高内聚解法。

<!--more-->


整篇文档以完整的软件工程视角展开，结构为：**系统分层架构 $\rightarrow$ 基础底座实现（坐标探测、几何居中拟合、手势互斥底座、入场动效） $\rightarrow$ 进阶手势交接与踩坑复盘 $\rightarrow$ 完整落地代码**。

---

## 1. 系统整体分层架构

要实现不依赖 Activity 切换、丝滑无黑屏的全屏预览，最佳实践是将预览层挂载在根视图的独立 `OverlayLayout` 中。系统逻辑自下而上分为三层：

```
┌─────────────────────────────────────────────────────────────┐
│ 3. 手势与内容消费层 (Gesture & Content Layer)               │
│    DismissibleBox (下拉位移与缩放) + ZoomableBox (双指缩放平移) │
├─────────────────────────────────────────────────────────────┤
│ 2. 动效与几何协调层 (Overlay Layer / Scope)                  │
│    OverlaySceneState (管理 animatedRect, bgAlpha, cornerRadius) │
├─────────────────────────────────────────────────────────────┤
│ 1. 宿主感知层 (Host Screen Layer)                            │
│    缩略图列表通过 Modifier 注册实时屏幕物理坐标 (Bounds & Radius)│
└─────────────────────────────────────────────────────────────┘
```

* **宿主感知层**：宿主列表中的每张图片通过 Modifier 探测并在全局注册自身屏幕绝对坐标（`Rect`）和圆角大小。
* **动效协调层**：全局单例控制器。负责管理黑底渐变（`bgAlpha`）、圆角收敛（`cornerRadius`）以及当前视口几何边界（`animatedRect`）。
* **手势消费层**：承载高清大图，负责处理内层的双指缩放及外层的下拉退出手势。

---

## 2. 基础实现：几何底座与核心流转管道

在处理复杂的退场交接之前，必须先搭建好稳定可靠的基础底座。

### 2.1 缩略图坐标探测器（Thumbnail Registry）
为了在点击缩略图时精准定位飞入起点，宿主图片需通过 `onGloballyPositioned` 捕获视口坐标，注册到全局单例池中：

```kotlin
data class ThumbnailTarget(
    val bounds: Rect,         // 屏幕视口内的绝对物理位置
    val cornerRadius: Float,  // 缩略图圆角
    val aspectRatio: Float,   // 图片宽高比 (width / height)
)

class OverlayLayoutThumbnailRegistry {
    private val targets = mutableMapOf<Any, () -> ThumbnailTarget?>()

    fun register(key: Any, provider: () -> ThumbnailTarget?) {
        targets[key] = provider
    }

    fun unregister(key: Any) {
        targets.remove(key)
    }

    /** 查询当前可见的缩略图位置，若已划出屏幕则返回 null */
    fun queryVisibleThumbnail(key: Any, screenBounds: Rect): ThumbnailTarget? {
        val target = targets[key]?.invoke() ?: return null
        // 校验缩略图是否在当前可视窗口内相交
        return if (screenBounds.overlaps(target.bounds)) target else null
    }
}
```

### 2.2 保持宽高比的全屏拟合矩形（FitRect 计算）
大图在大图浏览器中通常以 `ContentScale.Fit` 的方式居中充满屏幕。外层的 `animatedRect` 入场终点必须精确等于**图片在全屏空间居中后的实际几何边界（`fitRect`）**，否则在动画结束的瞬间会出现长宽拉伸突变。

```kotlin
object GeometryUtils {
    /** 根据屏幕宽高和图片宽高比，计算出居中 Fit 后的物理 Rect */
    fun calculateFitRect(screenBounds: Rect, aspectRatio: Float?): Rect {
        val screenW = screenBounds.width
        val screenH = screenBounds.height
        if (screenW <= 0f || screenH <= 0f) return Rect.Zero

        // 若无宽高比，默认占满视口
        val targetRatio = aspectRatio ?: (screenW / screenH)
        val containerRatio = screenW / screenH

        val fitW: Float
        val fitH: Float
        if (targetRatio > containerRatio) {
            // 宽对齐：以屏幕宽度为基准
            fitW = screenW
            fitH = screenW / targetRatio
        } else {
            // 高对齐：以屏幕高度为基准
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

---

### 2.3 双层手势容器的交互底座（Zoomable 与 Dismissible 互斥协同）

在大图交互中，**双指放大** 与 **单指下拉退场** 必须进行严格的手势互斥，否则用户在双指放大平移时会误触发下滑退场。

#### 1. 内层 `ZoomableState`：平移、缩放与边界约束
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

    /** 双指实时变换更新，同步约束 Offset 越界 */
    fun updatePanZoom(zoomChange: Float, panChange: Offset, centroid: Offset) {
        val newScale = (scale * zoomChange).coerceIn(minScale * 0.8f, maxScale * 1.5f)
        val center = Offset(containerSize.width / 2f, containerSize.height / 2f)
        val newOffset = if (newScale > 1f) {
            offset + panChange - (centroid - center) * (zoomChange - 1f)
        } else Offset.Zero

        scale = newScale
        offset = clampOffset(newOffset, newScale)
    }

    /** 边界碰撞检测：限制平移不能超出放大后的图像边缘 */
    private fun clampOffset(target: Offset, currentScale: Float): Offset {
        if (currentScale <= 1f) return Offset.Zero
        val maxX = containerSize.width * (currentScale - 1f) / 2f
        val maxY = containerSize.height * (currentScale - 1f) / 2f
        return Offset(target.x.coerceIn(-maxX, maxX), target.y.coerceIn(-maxY, maxY))
    }
}
```

#### 2. 外层 `DismissibleBox`：手势互斥驱动
通过 `enabled = { !zoomState.isZoomed }` 保证只有当图片处于原始比例（$1.0\times$）时，下拉手势才会被激活：

```kotlin
@Composable
fun DismissibleBox(
    state: DismissState,
    enabled: () -> Boolean,
    onDismissRequest: () -> Unit,
    modifier: Modifier = Modifier,
    content: @Composable BoxScope.() -> Unit,
) {
    Box(
        modifier = modifier
            .pointerInput(enabled) {
                if (!enabled()) return@pointerInput
                detectVerticalDragGestures(
                    onDragStart = { state.onSwipeStart() },
                    onDrag = { _, dragAmount -> state.updateSwipeDrag(Offset(0f, dragAmount)) },
                    onDragEnd = { state.settleSwipe(onDismissRequest) },
                )
            }
            .graphicsLayer {
                // 拖拽过程中直接改变图形矩阵
                translationX = state.contentOffset.x
                translationY = state.contentOffset.y
                scaleX = state.contentScale
                scaleY = state.contentScale
            },
        content = content
    )
}
```

---

### 2.4 全局转场状态机与入场动效（Entering $\rightarrow$ Settled）

整个全屏预览的生命周期由严密的状态机驱动：
```
[Entering: 缩略图 -> fitRect] ──> [Settled: 用户手势掌控中] ──> [Dismissing: 接管飞回缩略图] ──> [Dismissed]
```

#### 入场动画执行机制：
在初次挂载时，`animatedRect` 以缩略图物理位置为起点，同步启动透明度渐变与圆角展平：

```kotlin
fun runEnterAnimation() {
    coroutineScope.launch {
        coroutineScope {
            // 背景从全透明渐变至纯黑
            launch { bgAlpha.animateTo(1f, tween(260, easing = FastOutSlowInEasing)) }
            if (isGeometryMode) {
                // 圆角从缩略图的圆角平滑展开至直角 0dp
                launch { cornerRadius.animateTo(0f, tween(260, easing = FastOutSlowInEasing)) }
                // 几何矩形从缩略图位置扩展至全屏 fitRect
                launch { animatedRect.animateTo(fitRect, tween(260, easing = FastOutSlowInEasing)) }
            }
        }
        // 标记入场完毕，正式移交控制权给手势层
        transitionPhase = OverlayTransitionPhase.Settled
    }
}
```

---

## 3. 进阶痛点：经典缺陷与成因分析

当基础底座跑通后，一旦将手势拖拽退出与共享元素退场结合，就会暴露两个极其隐蔽的经典 Bug。

### 3.1 松手瞬间画面“暴缩沉降”（双重变换 / Double Transformation）

```
[拖拽松手瞬间]
1. DismissState (内层): contentScale = 0.8f, swipeOffset = 300px
2. Overlay (外层):   读取当前视觉矩形 (内含 0.8 缩放与 300px 偏移) -> animatedRect.snapTo(...)
────────────────────────────────────────────────────────────
❌ 叠加结果: Scale = 0.8 * 0.8 = 0.64f, OffsetY = 300 + 300 = 600px! (瞬间暴缩暴跌)
```

* **本质原因**：外层的 `animatedRect` 已经代理了手势发生后的几何大小与屏幕绝对坐标，但内层的 `DismissibleBox` 矩阵变换仍在持续生效，导致内外层进行了二次叠乘计算。

### 3.2 松手瞬间屏幕“先黑再亮”（Alpha 时序倒挂）

* 拖拽期间，背景随着下拉距离减小（比如淡化为 `0.5f`）。
* 松手时为了解决双重变换，调用了 `dismissState.reset()`。
* 但 `reset()` 把 `activeSource` 变为了 `None`，导致 `dismissState.backgroundAlpha` **瞬间跳回了 `1.0f`（纯黑）**。
* 退场动效在 `reset()` 之后才去读取当前 Alpha，读到了 `1.0f` 并执行 `bgAlpha.snapTo(1.0f)`。
* **现象**：半透明背景瞬间被盖上一层纯黑幕，下一毫秒才从纯黑慢慢淡化退出。

---

## 4. 核心架构设计：变换权力交接协议（TransformHandover）

针对双重变换问题，如果采用硬编码扁平计算：
$$\text{TotalOffset} = \text{DismissOffset} + (\text{ZoomOffset} \times \text{DismissScale})$$
外层必须知道内层的嵌套顺序，且无法扩展多图滑动或多层级手势。

### 4.1 数学本质：函数复合（Function Composition）
图形学中，多层嵌套的本质就是**函数的串联复合**：
$$\text{ScreenRect} = \text{Layer}_{外}(\ \text{Layer}_{内}(\text{fitRect})\ )$$

只要每一层只对自己的一亩三分地负责（将输入的 `Rect` 映射为经过本层形变后的输出 `Rect`，并在交接后重置自身），无论嵌套多少层、顺序如何调整，都能通过链式折叠（Fold）自动完成精确计算。

### 4.2 协议定义：各 State 自治与链式折叠

#### 1. 定义形变层协议与通用扩展
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

#### 2. 各 State 实现协议（互不知道对方存在）
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

#### 3. 链式交接协调器（Chain Coordinator）
通过函数折叠与批量重置，一键完成“采样快照 + 瞬间洗净”：
```kotlin
class TransformHandoverChain(
    private val layers: List<TransformHandoverLayer>,
) {
    /** 核心：由内到外链式流转计算，并同步重置整条链上的内部状态 */
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

## 5. 渲染时序与调度保障

即便数学计算正确，若在 Compose 线程和调度微任务中有微小延迟，依然会造成 1 帧跳动。

### 5.1 快照采样顺序治理
在退出触发函数中，必须保证**先采样当前的 Alpha 快照，再执行状态归一重置**：

```kotlin
fun triggerDismiss(...) {
    if (transitionPhase == OverlayTransitionPhase.Dismissing) return

    // 🌟 步骤 1：先采样当前的真实透明度快照（此时 dismissState 尚未 reset）
    val currentVisualAlpha = contentAlphaProvider?.invoke() ?: bgAlpha.value

    // 🌟 步骤 2：标记状态机进入退出阶段
    transitionPhase = OverlayTransitionPhase.Dismissing

    // 🌟 步骤 3：执行几何计算（内部会触发 captureVisualRectAndReset 抹除内层状态）
    val startVRect = customTransform?.invoke(fitRect)
        ?: dismissTransformHandoverProvider?.invoke()?.captureVisualRectAndReset(fitRect)
        ?: fitRect

    // 🌟 步骤 4：UNDISPATCHED 同步启动动效
    coroutineScope.launch(start = CoroutineStart.UNDISPATCHED) {
        bgAlpha.snapTo(currentVisualAlpha) // 衔接在当前透明度，绝不弹回 1.0f
        animatedRect.snapTo(startVRect)

        coroutineScope {
            launch { bgAlpha.animateTo(0f, tween(260)) }
            launch { animatedRect.animateTo(targetThumbnail.bounds, tween(260)) }
        }
    }
}
```

### 5.2 为什么必须使用 `CoroutineStart.UNDISPATCHED`？

Compose 动画中的 `Animatable.snapTo()` 是一个挂起函数（`suspend`），因为它内部依赖 `MutatorMutex` 互斥锁来抢占并取消此前正在运行的旧动画。

如果使用默认的 `coroutineScope.launch`：
* 协程任务会被推入 Android 主线程的 `Handler/Looper` 消息队列做微任务排队。
* 此时在当前调用栈中，内层的 `reset()` 已经把状态归零了。
* 如果刚好赶上屏幕渲染周期（VSYNC 信号），用户会看到**整整 1 帧“画面弹回原位”，下一帧才开始退场动效**的跳变。

**`CoroutineStart.UNDISPATCHED` 的作用**：
它打破默认队列排队机制，**强制协程体内的第一段代码（直到第一个真正的挂起点 `animateTo` 之前）直接在当前主线程调用栈中同步执行**。
它使得：
$$\text{dismissState.reset()} \quad \text{与} \quad \text{animatedRect.snapTo(startVRect)}$$
在**同一个主线程事件周期内完成提交**，Compose 快照系统（Snapshot System）在下一帧 VSYNC 打包渲染，彻底消灭 1 帧闪动。

### 5.3 背景遮罩状态机设计
外层遮罩渲染必须与状态机对齐：
```kotlin
val currentBackgroundAlpha: Float
    get() = when (transitionPhase) {
        OverlayTransitionPhase.Entering -> bgAlpha.value
        // 手势进行期：实时跟随手指
        OverlayTransitionPhase.Settled -> contentAlphaProvider?.invoke() ?: bgAlpha.value
        // 退场进行期：完全由 bgAlpha 动画驱动接管（内层已被 reset，不得再读）
        OverlayTransitionPhase.Dismissing -> bgAlpha.value
    }
```

---

## 6. 深度踩坑：Compose 引用陈旧与 Modifier 演进

### 6.1 `by rememberUpdatedState` 解包陷阱

曾尝试如下代码向外层绑定数据：
```kotlin
val currentTransformHandover by rememberUpdatedState(dismissTransformHandover)

DisposableEffect(targetKey) {
    // ⚠️ 致命 Bug：直接解包赋值
    state.dismissTransformHandover = currentTransformHandover
}
```
* `by` 关键字仅是一个属性委托，它会在访问该变量的瞬间调用 `.value` 取值。
* `DisposableEffect(targetKey)` 只在初次挂载时执行一次。
* 这行代码**在第 1 帧把解包后的裸对象传给 state** 后，后续重组哪怕 `dismissTransformHandover` 变了，`state` 里引用的依然是首帧的过期快照！
* **原则**：向外部对象提供数据时，必须提供**可延迟计算的 Lambda（`() -> T`）**，让取值发生在被调用的瞬间；或者使用 `SideEffect` 每帧同步。

### 6.2 多图画廊（HorizontalPager）的 `onDispose` 误杀
多图滑动时，页面 0 和页面 1 同时驻留内存。若页面 0 退出屏幕触发 `onDispose`，直接无脑清空 `state.providers = null`，会导致刚进入的页面 1 数据被破坏。
**防御策略**：
```kotlin
onDispose {
    // 只有全局激活的对象依旧是我时，才允许清理
    if (state.currentKeyProvider?.invoke() == targetKey) {
        state.clearAllProviders()
    }
}
```

### 6.3 Modifier 三代演进对比

```
[第一代: composed + 双层 Lambda] ──> [第二代: DisposableEffect + SideEffect] ──> [第三代: Modifier.Node]
   (过度设计、套娃严重)                     (业务代码首选: 精简无 Bug)             (底层库首选: 零重组开销)
```

#### 实用首选（第二代：`SideEffect` + `DisposableEffect`）
仅 15 行代码，职责彻底解耦：
```kotlin
override fun Modifier.overlayInteractiveTarget(...): Modifier = this.composed {
    val targetKey = itemKey ?: state.initialKey

    // 1. 只负责销毁期清理与防误杀
    DisposableEffect(targetKey) {
        onDispose {
            if (state.currentKeyProvider?.invoke() == targetKey) {
                state.clearProviders()
            }
        }
    }

    // 2. 只负责重组期参数同步（每帧无脑赋值最新 Lambda，零套娃）
    SideEffect {
        state.currentKeyProvider = { targetKey }
        state.contentAlphaProvider = backgroundAlpha
        state.dismissTransformHandoverProvider = dismissTransformHandover
    }
    this
}
```

#### 底层极致性能（第三代：`Modifier.Node`）
如果编写公共基础组件，使用 `Modifier.Node` 能够消除 `composed` 的隐藏重组开销：
* 虽然内联 Lambda 会导致 `ModifierNodeElement.equals()` 为 `false`。
* 但 Compose 框架仅仅是在 Layout 树上调用了一次普通的 Java 成员函数 `node.update()`，**耗时极低（几个纳秒），绝对不会反向引发 Composable 重组**。

---

## 7. 完整生产级实现参考

### 7.1 全局转场控制器
```kotlin
class OverlaySceneState(
    val initialKey: Any,
    initialWindowBounds: Rect,
    val registry: OverlayLayoutThumbnailRegistry,
    private val coroutineScope: CoroutineScope,
    private val onDismissFinished: () -> Unit,
) {
    var transitionPhase by mutableStateOf(OverlayTransitionPhase.Entering)
        private set

    var contentAlphaProvider by mutableStateOf<(() -> Float)?>(null)
    var currentKeyProvider by mutableStateOf<(() -> Any)?>(null)
    var dismissTransformHandoverProvider by mutableStateOf<(() -> TransformHandoverChain)?>(null)

    private val initialTarget = registry.queryVisibleThumbnail(initialKey, initialWindowBounds)
    val isGeometryMode = initialTarget != null

    var fitRect by mutableStateOf(GeometryUtils.calculateFitRect(initialWindowBounds, initialTarget?.aspectRatio))
        private set

    val animatedRect = Animatable(initialTarget?.bounds ?: fitRect, RectVectorConverter)
    val bgAlpha = Animatable(0f)
    val cornerRadius = Animatable(initialTarget?.cornerRadius ?: 0f)

    val currentBackgroundAlpha: Float
        get() = when (transitionPhase) {
            OverlayTransitionPhase.Entering -> bgAlpha.value
            OverlayTransitionPhase.Settled -> contentAlphaProvider?.invoke() ?: bgAlpha.value
            OverlayTransitionPhase.Dismissing -> bgAlpha.value
        }

    fun runEnterAnimation() {
        coroutineScope.launch {
            coroutineScope {
                launch { bgAlpha.animateTo(1f, tween(260, easing = FastOutSlowInEasing)) }
                if (isGeometryMode) {
                    launch { cornerRadius.animateTo(0f, tween(260, easing = FastOutSlowInEasing)) }
                    launch { animatedRect.animateTo(fitRect, tween(260, easing = FastOutSlowInEasing)) }
                }
            }
            transitionPhase = OverlayTransitionPhase.Settled
        }
    }

    fun triggerDismiss(
        key: Any? = null,
        customTransform: ((baseRect: Rect) -> Rect)? = null,
    ) {
        if (transitionPhase == OverlayTransitionPhase.Dismissing) return

        // 1. 采样 Alpha 快照
        val currentVisualAlpha = contentAlphaProvider?.invoke() ?: bgAlpha.value
        transitionPhase = OverlayTransitionPhase.Dismissing

        val targetKey = key ?: currentKeyProvider?.invoke() ?: initialKey

        // 2. 原子化捕获形变并洗净内层状态
        val startVRect = customTransform?.invoke(fitRect)
            ?: dismissTransformHandoverProvider?.invoke()?.captureVisualRectAndReset(fitRect)
            ?: fitRect

        // 3. UNDISPATCHED 同步接管，消除 1 帧调度延迟
        coroutineScope.launch(start = CoroutineStart.UNDISPATCHED) {
            bgAlpha.snapTo(currentVisualAlpha)
            animatedRect.snapTo(startVRect)

            val targetThumbnail = registry.queryVisibleThumbnail(targetKey, fitRect)
            val finalRect = targetThumbnail?.bounds ?: GeometryUtils.getCenterPointRect(fitRect)
            val finalRadius = targetThumbnail?.cornerRadius ?: 0f

            coroutineScope {
                launch { bgAlpha.animateTo(0f, tween(260, easing = FastOutSlowInEasing)) }
                if (isGeometryMode || targetThumbnail != null) {
                    launch { cornerRadius.animateTo(finalRadius, tween(260, easing = FastOutSlowInEasing)) }
                    launch { animatedRect.animateTo(finalRect, tween(260, easing = FastOutSlowInEasing)) }
                }
            }
            onDismissFinished()
        }
    }
}
```

### 7.2 UI 侧全屏大图消费组件
```kotlin
@Composable
fun OverlayLayoutScope.ImageFullScreenViewer(
    overlayTransitionElementId: String,
    imageUrl: Any,
    modifier: Modifier = Modifier,
) {
    val zoomState = rememberZoomableState()
    val dismissState = rememberDismissState()

    // 组合形变链：内层 Zoomable，外层 Dismissible
    val handoverChain = remember(zoomState, dismissState) {
        TransformHandoverChain(listOf(zoomState, dismissState))
    }

    val dismissAction = remember(overlayTransitionElementId) {
        { this.dismiss(overlayTransitionElementId) }
    }

    // 适配 Android 物理 Back / Predictive Back 返回手势
    BackHandler(enabled = true, onBack = dismissAction)

    DismissibleBox(
        state = dismissState,
        enabled = { !zoomState.isZoomed }, // 手势互斥
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

## 8. 设计哲学总结

1. **权力移交定律（Hand-off Invariant）**：
   从“手势驱动”切换为“动效驱动”时，外层接管全部几何形变的同时，内层本地矩阵必须在同一帧归零（Identity）。
2. **函数复合解耦层级**：
   摒弃硬编码扁平缩放偏移算式，利用 `TransformHandoverLayer` 协议和链式折叠（Fold）流转变换，让任意多层手势天然解耦。
3. **时序与快照严明**：
   在抹除内层状态之前建立快照缓存，借助 `CoroutineStart.UNDISPATCHED` 消除主线程执行栈与微任务队列的延迟空档，杜绝黑屏与 1 帧跳动。
4. **合理选用 Modifier 机制**：
   * 业务开发遵循简洁至上：优先使用 **`SideEffect` + `DisposableEffect`**。
   * 基础设施追求极限吞吐：选用 **`Modifier.Node`**，将重组开销降维为纳秒级的对象属性赋值。