---
title: "Dart 碎片记录"
date: 2025-11-28T23:08:21+08:00
slug: ""
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

## Basic 1

```Dart
import 'dart:math' show Random;

void main() async {
  print('Compute π using the Monte Carlo method.');
  await for (final estimate in computePi().take(100)) {
    print('π ≅ $estimate');
  }
}

/// Generates a stream of increasingly accurate estimates of π.
Stream<double> computePi({int batch = 100000}) async* {
  var total = 0; // Inferred to be of type int
  var count = 0;
  while (true) {
    final points = generateRandom().take(batch);
    final inside = points.where((p) => p.isInsideUnitCircle);

    total += batch;
    count += inside.length;
    final ratio = count / total;

    // Area of a circle is A = π⋅r², therefore π = A/r².
    // We consider only non-negative x and y (that is, the
    // first quadrant), which doesn't change the ratio.
    // So, when given random points with x ∈ [0, 1],
    // y ∈ [0, 1], the ratio of those inside the unit circle
    // should approach π / 4. Therefore, the value of π
    // should be:
    yield ratio * 4;
  }
}

Iterable<Point> generateRandom([int? seed]) sync* {
  final random = Random(seed);
  while (true) {
    yield Point(random.nextDouble(), random.nextDouble());
  }
}

class Point {
  final double x;
  final double y;

  const Point(this.x, this.y);

  bool get isInsideUnitCircle => x * x + y * y <= 1;
}
```

- Stream: with `async*` modifier, 异步生成器，返回这个 Stream
- Iterable: with `sync*` modifer, 同步生成器，返回 Iterable
- async 异步， sync 同步，`*` 表示是一个生成器；函数默认都是 sync 的，所以单独的 sync 不写
- async 返回的永远都是 `Feature<T>`；但是对 main 函数，因为不会检查返回值，所以写成 `void` 也没事 (正常该是 `Feature<void>`)
- main 可以是 sync, 也可以是 async.
- 同步方法里，可以调用异步函数，但是没法用 await 关键字来等待异步返回；只用用 `.then` 来串联其后续动作。
  - 为什么不能去等异步的返回？程序层面不行。理论上可以 while + sleep + 检测标志位 来实现等待，但这失去了异步的价值？不过感觉还是不应该有这个限制的。
  - 但从 ChatGPT 描述，Dart 是单线程的，这么做会卡死 UI 或者死锁——如果是单线程，你去 while 检测肯定就锁死了。
  - 所以，要想用 await，就用 async. async 函数有传递性啊。
  - 同步函数就是去吊起一个操作的。如果有先后依赖关系，就得靠 then 来解决
  - 如果有多个依赖，还有 Future.wait([]).then((xxx) {xxx}) 来等待多个返回后再执行 then.
- 类定义：字段、构造函数、方法；默认所有类都是 public；私有成员用前缀 `_` 标识
- 构造函数有几类：
  - 普通构造函数：类同名的方法 `C`；可以用 `this.xxx` 来按**位置对应关系**赋值字段； 只有 1 个
  - 命名构造函数：类名+`.`+名字，`C.name`. 可以有多个
  - 工厂函数
- 方法：特殊的 get 方法，使用箭头函数直接返回，有 `get` 关键字
- 单继承，使用 extends 关键字；
- mixin 混入，类似 interface，可以实现多继承

## Basic 2

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class Package {
  final String name;
  final String latestVersion;
  final String? description;

  Package(this.name, this.latestVersion, {this.description});

  @override
  String toString() {
    return 'Package{name: $name, latestVersion: $latestVersion, description: $description}';
  }
}

void main() async {
  final httpPackageUrl = Uri.https('dart.dev', '/f/packages/http.json');
  final httpPackageResponse = await http.get(httpPackageUrl);
  if (httpPackageResponse.statusCode != 200) {
    print('Failed to retrieve the http package!');
    return;
  }
  final json = jsonDecode(httpPackageResponse.body);
  final package = Package(
    json['name'],
    json['latestVersion'],
    description: json['description'],
  );
  print(package);
}
```

1. final 关键字：表示第一次赋值后不能再更改；无需要求静态赋值（可以运行时才知道值）；const 则表示值需要在编译时就是定值。
   与 final 对应的是 `var`，默认就是 var, 所以一般不写。
2. toString 是从顶层 object 隐式继承的函数
3. `String?` 里的 ? 表示可空 (null) 类型.
   `!` 非空断言;
   `??` 空合并操作符： `final b = a ?? "null-default"`
   `??=` 空合并赋值： `b ??= "null-default"`: 只有 b 是 null 时才执行这个赋值。
4. Dart 的核心内建类型可以分成 7 类：
   - 基本值类型：int, double, num, bool, String, Null
   - 对象体系：Object, Object?, dynamic
   - 集合：List, Set, Map, Iterable
   - 异步：Future, Stream, FutureOr
   - 运行时：Type, Function, Never
   - 时间：DateTime, Duration
   - 大数/实用类：BigInt, RegExp, Uri

## Basic 3

```
final container = Container(
  // grey box
  width: 320,
  height: 240,
  color: Colors.grey[300],
  child: Center(
    child: Transform(
      alignment: Alignment.center,
      transform: Matrix4.identity()..rotateZ(15 * 3.1415927 / 180),
      child: Container(
        // red box
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: Colors.red[400],
        ),
        child: Text(
          'Lorem ipsum',
          style: bold24Roboto,
          textAlign: TextAlign.center,
        ),
      ),
    ),
  ),
);
```

完成旋转操作；

`Matrix4.identity()..rotateZ(15 * 3.1415927 / 180)`:

1. `..` 是级联运算符：不管函数返回啥，通过这个运算符，总是返回执行操作后得原对象。
2. 要想选择一个元素，使用这个方法！ Transform widget + Matrix.

