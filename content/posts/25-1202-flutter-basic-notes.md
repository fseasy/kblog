---
title: "Flutter Basic Notes"
date: 2025-12-02T17:24:41+08:00
slug: "flutter-baisc-notes"
draft: false
image: 
math: false
license: CC BY-SA 4.0
comments: true
hidden: false

tags:
  - 
categories:
  - 
---

## Layout

1. layout 是由自己和 parent 共同决定的：Constraints go down. Sizes go up. Parent sets the position.
   
   In the simplest example, the layout conversation looks like this:

   - A widget receives its constraints from its parent.
   - A constraint is just a set of 4 doubles: a minimum and maximum width, and a minimum and maximum height.
   - The widget determines what size it should be within those constraints, and passes its width and height back to the parent.
   - The parent looks at the size it wants to be and how it should be aligned, and sets the widget's position accordingly. Alignment can be set explicitly, using a variety of widgets like Center, and the alignment properties on Row and Column.

2. ListViews are commonly used when you have an unknown or very large (or infinite) number of list items. When this is the case, **it's best to use the ListView.builder constructor**. The builder constructor only builds the children that are currently visible on screen.

## State Management

[原文档](https://docs.flutter.cn/get-started/fundamentals/state-management).

这里都是讲的内建 (primitives)方式，实际开发中常用第三方库，如 riverpod, provider 等；

1. 继承 StatefulWidget, 本身不存储任何状态，而是包含一个 `State<WidgetT>` 类成员；这个成员由一个继承自 `Stage<WidgetT>` 的子类构造。
   - Encapsulation: widget 本身不能看到/访问 State 里的内容 (后面看到，其实也可以定义成员；然后在 State 里可以通过 widget 变量来访问！)
   - Object lifecycle: State object 和 它的成员在 Widget 初始化时构建，并直到它从屏幕中移除才销毁。这是 **ephemeral state** (暂时状态).

2. 状态的共享/依赖：如果一个状态改变，如何把状态传递给需要的 widgets? 3 种方法
   1. widget constructor: 把依赖内容暴露为 contructor 变量——这样上层改了，重建时下游自然重建。
      问题：似乎这只适合依赖的对象是当前类的子元素的情况？
      另外，如果层次很深，就需要每层开洞（透传），也即 prob-drilling.
   2. InherientWidget: 继承自 InherimentWidget, 就可以使用 Of 方法，把变量注入到 Context 里。
      但这个 State 本身还需要实现一个是否相等的判断的函数，方便框架确定是否需要重绘。
      provider 就是这个机制。
   3. callback: widget 暴露一个 callback 函数给外部；外部可以自定义这个 callback 操作

      Flutter provides the `ValueChanged` type, which declares a function callback with a single parameter:
      `typedef ValueChanged<T> = void Function(T value);`

3. 状态共享让别的 widget 获取到状态，但是还需要一个额外的触发机制来更行UI，从而让状态更新真的被应用（绘制）。 2 种触发更新 UI 的方式：
   1. ChangeNotifier: 自定义一个类继承自 `ChangeNotifier`，可以自定义需要的成员变量或函数；然后实例化成一个变量：
      - 注册监听：需要根据状态更新 UI 的 widget，就在构建 Widget 时，用 `ListenableBuilder` 包裹这个 Widget：将此实例变量传给其 `listenable` 参数，然后在 builder 参数中访问此变量，构建 Widget.
      - 需要触发更新 UI 时，就调用实例的 `notifyListeners` 接口（继承自 ChangeNotifier）.
   2. ValueNotifier: 是 ChangeNotifier 的简化版。直接绑定一个值，这样就不用去创建一个继承自 ChangeNotifier 的类了。
      - 通过 `ValueListenableBuilder` 或者 `ListenableBuilder` 来创建监听；前者的区别是 builder 函数直接将 value unpack 出来作为参数传递给 callback 了。
      - 通过 `ValueNotifier.value` 来访问值；

4. MVVM 组织代码： Model - ViewModel - View.
   
   - Model: server 侧的数据结构；可以与 Flutter 无任何关系。
   - ViewModel: 与 Model 交互，同时暴露接口给 View. 一般继承自 ChangeNotifier, 这样 View 就可以使用它来注册监听，实时渲染改动。
   - View: Widget (这里的话，都是用 ListenableBuilder 封装，这样方便更新)。注意它只会访问 ViewModel 实例，看不见 Model 的。

## User Input


### 按钮

Buttons allow a user to initiate an action in the UI by clicking or tapping. The Material library provides a variety of button types that are functionally similar, but styled differently for various use cases, including:

ElevatedButton: A button with some depth. Use elevated buttons to add dimension to otherwise mostly flat layouts. 在整体偏扁平的界面中，使用带立体感（有阴影）的按钮，可以增加层次和深度感。 dimension 在这里不是数学维度，指的是：立体感，层次感，深度（depth）

FilledButton: A filled button that should be used for important, final actions that complete a flow, like Save, Join now, or Confirm. 实色按钮（primary color, 很显眼）
Tonal Button: A middle ground button between FilledButton and OutlinedButton. They're useful in contexts where a lower-priority button requires more emphasis than an outline, like Next. 色调按钮（同色系但柔和，次要引导）
OutlinedButton: A button with text and a visible border. These buttons contain actions that are important, but aren't the primary action in an app. 描边按钮（几乎无填充，低调）
TextButton: Clickable text, without a border. Since text buttons don't have visible borders, they must rely on their position relative to other content for context.
IconButton: A button with an icon.
FloatingActionButton: An icon button that hovers over content to promote a primary action.


Rich Text: Rich Text Editor code https://github.com/flutter/samples/tree/main/simplistic_editor


Form App demo: https://github.com/flutter/samples/tree/main/form_app/

## 语法

`<Calendar>{calendarView},`: 泛型字面量构造函数； = Calendar<T>(child: calendarView) where T is the 推断的类型。



## Tips


SizedBox(width: 18, child: SizedBox(child: Text('$_favoriteCount'))),
当 Text 在 40 和 41 之间变化时，将文本放在 SizedBox 中并设置其宽度可防止出现明显的「跳跃」，因为这些值具有不同的宽度。

## Examples


根据 size / 列表来动态创建元素。

```
 inputs = 3
 children: List<Widget>.generate(inputs, (int index) {
                return InputChip(
                  label: Text('Person ${index + 1}'),
                  selected: selectedIndex == index,
                  onSelected: (bool selected) {
                    setState(() {
                      if (selectedIndex == index) {
                        selectedIndex = null;
                      } else {
                        selectedIndex = index;
                      }
                    });
                  },
                  onDeleted: () {
                    setState(() {
                      inputs = inputs - 1;
                    });
                  },
                );
              }).toList(),
```