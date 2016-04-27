# 信号 Signals

这一章关于Signals，他包括:

* 介绍 Elm 中的信号
* 用`foldp`保持程序的状态
* 信箱

## 信号介绍

Elm 中，**Signals**是一个创建程序基本的构建块。你可以把signal想象成每个时刻都在改变的一个流。

Signals 可以被合并，转换和过滤。


让我们来看一个基本的 signal：

```elm
module Main (..) where

import Html exposing (Html)
import Mouse

view : Int -> Html
view x =
  Html.text (toString x)

main : Signal.Signal Html
main =
  Signal.map view Mouse.x
```

[]()

如果在浏览器中运行这个例子，你将会看到易懂鼠标时`x`坐标值的变化。第一行我们从core导入了Html模块用来将x坐标值显示为Html。第二行导入了Mouse模块。他提供了用于鼠标工作的工具。

### view

view是一个函数，取一个整数返回一个HTML片段（`Int -> Html`）。

然而，`Html.text`需要的是一个字符串，所以我们必须将`x`转换成一个 string 。这就是toString函数要做的事情。

### main

一个 Elm 的 main 函数可以返回一个**静态element**或是一个**signal**。这里的 main 返回一个`Html`的 signal （`Signal.Signal Html`）。这就是说，HTML可以随时间的变化而变化！

要理解这是如何实现的，让我们分析一下上面程序的最后一行。

### Mouse.x

`Mouse.x`会在`x`坐标改变时给我们一个 signal 。这个 singal 有一个签名 `Signal.Signal Int`，表示这个 signal 携带一个整数。

### Signal.map

`Signal.map`是一个转换和遍历一个 signal 为不同 signal 的函数。

他的类型签名是`Signal.map : (a -> result) -> Signal a -> Signal result`。

* 第一个参数是一个函数，他接收一个`a`类型的值，返回一个`result`类型的值。这个函数将原signal的值转换为输出signal的值@todo

* 第二个参数`Signal.map`是原signal，也就是说，要想转换signal，这个signal必须有一个`a`类型。

* 输出的类型是`Signal.result`。表示输出的signal类型是`result`。

> **图示**：我们有一个signal`A`。Signal.map产生第二个signal`B`。第二个 signal 产生的值是原signal的两倍。

回到我们的例子：

```elm
Signal.map view Mouse.x
```

上面的例子中，`view`函数用来转换或遍历函数，他取一个`Int`值并返回`Html`。

第二个参数（`Mouse.x`）是一个`Int`值得 signal。

结果是一个`Html`值的 signal，作为`main`函数输出，这正是我们想要的。

`Singal.map`返回一个新的 signal 并带有一个从原 singal 用转换函数转换来的值。作为原 signal 的变化，每个新值都被转换或映射到目标 signal 类型。


这是map的另一个列子：

```elm
double x =
  x * 2

doubleSignal =
  Singal.map double Mouse.x
```

`double`是一个将输入值加倍的函数，这样看来，`doubleSignal`是一个将当前鼠标 x 坐标乘以 2 的 singal。

### 练习

试着组合鼠标`x`坐标和上面的`doubleSignal`，当你移动鼠标时，你将看到 x 乘 2 的值。


## 保持 state




