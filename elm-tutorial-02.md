# 信号 Signals

这一章关于信号，他包括:

* signal 介绍
* 使用`foldp`函数保持状态
* mailboxs

## signal 介绍

在Elm 中，**Signals**是创建应用程序的一个基本构建块。可以把一个 signal 想象成一个流，他的值每时每刻都在改变。

Signals 可以被合并，转换以及过滤。


先我们来看一个基本的 signal：

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

[https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/BasicMouse.elm](https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/BasicMouse.elm)

在浏览器中运行这段代码，移动鼠标时就会看到`x`坐标值的变化。

首先我们导入了`Html`模块，用他来将`x`坐标显示为 HTML。

紧接着导入了`Mouse`模块。他提供了用于鼠标操作的工具。

### view

`view`函数取一个整数作为参数并返回一段HTML（`Int -> Html`）。

不过，`Html.text`需要的是一个字符串，所以我们必须将`x`转为字符串。这要用到toString。

### main

Elm 程序的 main 函数比较特殊，他要求返回一个**静态元素**或是一个**信号**。这里的`main`返回的是挂载了`Html`的信号（`Signal.Signal Html`）。换句话说，我们的HTML可以随时间的变化而变化。

要理解这是怎么做到的，让我们分析一下代码的最后一行（`Signal.map view Mouse.x`）。

### Mouse.x

`Mouse.x`会在`x`改变时给我们一个signal 。这个信号的签名是`Signal.Signal Int`，他表示这个信号会携带一个整数。

### Signal.map

`Signal.map`是将一个信号转换为另一个信号的函数，类型签名是`Signal.map : (a -> result) -> Signal a -> Signal result`。让我们来分析一下：

* 第一个参数是一个函数，他接收`a`类型的值，返回`result`类型的值。这个函数会将原信号携带的值转换为一个新值用作输出信号。
* 第二个参数就是原信号，他的类型变量是`a`，和第一个参数中的`a`相同。
* 返回类型是`Signal result`，表示输出的signal携带的类型是`result`。

> **图示**：signal`A`通过`Signal.map`产生了signal`B`。产生信号中的值是原信号中值的两倍。

回到我们的例子中：

```elm
Signal.map view Mouse.x
```

`view`是用来转换的函数，他取一个`Int`并返回`Html`。

第二个参数（`Mouse.x`）是一个有`Int`值的信号。

结果正是我们想要的`Html`值的信号，作为`main`函数的输出。

`Singal.map`用转换函数将原信号转换成为一个新信号。当原信号发生变化时，信号中的值都会被转换到新信号中。


这里是 map 的另一个例子：

```elm
double x =
  x * 2

doubleSignal =
  Signal.map double Mouse.x
```

`double`是一个将输入值加倍的函数，这样看来，`doubleSignal`就是一个将当前 x 坐标值乘 2 的信号。

## 保持状态

上一章中，我们创建了一个简单的应用来显示当前鼠标`x`的坐标。当前状态作为输入值接受之后就被丢弃了。但是在大多数应用中，我们希望在用户交互时储存一些状态。

让我们来创建一个应用跟踪鼠标单击。


```elm
module Main (..) where


import Html exposing (Html)
import Mouse



view : Int -> Html
view count =
  Html.text (toString count)



countSignal : Signal Int
countSignal =
  Signal.map (always 1) Mouse.clicks



main : Signal.Signal Html
main =
  Signal.map view countSignal
```

[https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/Clicks.elm](https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/Clicks.elm)

这个程序总是显示`1`，让我们来依次介绍一下。

`view`函数显示给定的数量，但这里他总是1。

`countSignal`函数产生一个整数 signal。他取`Mouse.clicks`signal作为输入并将他映射为`(always 1)`。

`main`取`countSignal`并用`view`函数转换他。

### Signal.foldp

`Signal.foldp`取前一个状态和当前的输入进行组合，产生一个新的signal。他很像JavaScript数组的`reduce`操作。

这里使用了`foldp`：

```elm
module Main (..) where


import Html exposing (Html)
import Mouse



view : Int -> Html
view count =
  Html.text (toString count)
  
  

countSignal : Signal Int
countSignal =
  Signal.foldp (\_ state -> state + 1) 0 Mouse.clicks
  
  
main : Signal.Signal Html
main =
  Signal.map view countSignal
```

[https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/ClicksWithFoldp.elm](https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/ClicksWithFoldp.elm)

让我们看下`Signal.foldp`那行发生了什么：

`Signal.foldp`有三个参数：

* 一个累加器函数：`(\_ state -> state + 1)`
* 初始状态，这里是`0`
* 输入信号 signal：`Mouse.clicks`

`\x y -> x + y`是一个匿名函数，他等同于 ES6 中的`(x, y) => x + y`。

### foldp是如何工作的

* `foldp`每次接到输入信号就会调用累加器函数
* 累加器函数接受上一个状态和输入信号
* 第一次调用`foldp`会接受一个输入信号和初始状态传递给累加器
* 累加器函数计算并返回新的状态
* `foldp`保持这次的新状态，下一次时将作为上一个状态传入累加器
* `foldp`产生一个输出信号

### 累加器函数

```elm
\_ state -> state + 1
```

**累加器函数**从`Mouse.clicks`取输入和前一状态，并返回一个新的状态。

`Mouse.clicks`的输入总是`()`，这在 ELm 中叫做单位类型。我们不需要这个值，所以用`_`忽略他。

## Mailboxes

`foldp`是构建程序非常重要的块，另一个重要的是 Mailboxes。

目前为止，我们一直在监听一个像`Mouse.x`一样的"原始"signal并且用它来在页面上显示信息。在一个大型应用中，我们要的是能够用户与UI元素交互。如单击一个链接或按钮。

在ELm中我们使用**mailboxs**来做这件事。一个mailbox是一个从UI接收message，task和广播信号的中央枢纽。

想要更好的理解这些，让我们开始创建一个按钮：

```elm
module Main (..) where

import Html exposing (Html)


view : String -> Html
view message =
  Html.div
    []
    [ Html.div [] [ Html.text message ]
    , Html.button [] [ Html.text "Click" ]
    ]


messageSignal : Signal String
messageSignal =
  Signal.constant "Hello"


main : Signal Html
main =
  Signal.map view messageSignal
```

[https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/Mailbox01.elm](https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/Mailbox01.elm)

`view`来显示一个信息和一个按钮为我们点击，按钮木有做任何事。

`messageSignal`是一个用“Hello”构造的字符串 signal，意味着他不会改变。

`main`从`messageSignal`取出字符串交给`view`映射。如果你在浏览器中跑这段代码，你将会看到从构造`messageSignal`的值`Hello`。

### 添加交互

我们想要点击按钮改变 message。做这个需要用到`Html.Events.onClick`。如果你读过`Html.Events`模块的文档，你会看到所有的事件都有相同的签名：`Signal.Address a -> a -> Html.Attribute`

他取任意类型变量的`Signal.Address`作为第一个参数，和一个具备相同类型变量的第二个参数，返回一个`Html.Attribute`。

### 地址 Address

一个**address**是一个指向特殊 signal 指针。他允许我们发送**messages**给那个singal。

获取一个**address**发来的 message，我们需要使用一个`Mailbox`。

创建一个**mailbox**像这样：

```elm
mb : Signal.Mailbox String
mb =
  Signal.mailbox ""
```

`mb`是一个返回`Mailbox`函数。这个特殊的 mailbox 用于处理字符串@todos，例如，他接收`String`类型的message并返回一个`String`类型的 signal。这个参数是 mailbox 提供给 signal 的默认值，这里他是个空字符串。

这个函数返回的 mailbox 是一个两个属性的 record：

```elm
{ address : Signal.Address a
, signal : Signal.Signal a
}
```

记录的`address`我们可以发消息给他，并且可以监听他的`signal`。

## 使用 Mailbox

下一步是在我们的程序中使用mailbox，这样我们就可以发送和刷新messages。

```elm
module Main (..) where

import Html exposing (Html)
import Html.Events as Events


view : Signal.Address String -> String -> Html
view address message =
  Html.div
    []
    [ Html.div [] [ Html.text message ]
    , Html.button
        [ Events.onClick address "Hello"
        ]
        [ Html.text "Click" ]
    ]


mb : Signal.Mailbox String
mb =
  Signal.mailbox ""


main : Signal Html
main =
  Signal.map (view mb.address) mb.signal
```

[https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/Mailbox02.elm](https://github.com/sporto/elm-tutorial-assets/blob/master/code/020_signals/Mailbox02.elm)

### view

```elm
view : Signal.Address String -> String -> Html
view address message =
  Html.div
    []
    [ Html.div [] [ Html.text message ]
    , Html.button
        [ Events.onClick address "Hello"
        ]
        [ Html.text "Click" ]
    ]
```

`view`函数现在取`Signal.Address`作为第一个参数。

`Events.onClick address "Hello"`设置一个事件监听器监听Html元素的点击事件。`onClick`发送给定的message（这里是“Hello”）给给定的地址在每次元素点击时。

### main

```elm
main : Signal Html
main =
  Signal.map (view mb.address) mb.signal
```

`main`函数做了两件事：

* 创建了一个偏应用的view`(view mb.address)`，这样view总是将mailbox的`address`作为第一个参数。
* 在mailbox的导出的signal上映射偏应用view。`view`将mailbox signal作为第二个参数。

这里的图帮助我们理解发生了什么：

1. 最开始，`main`用mailbox的初始值传递给地址然后渲染view(空字符串)
2. 当视图渲染完成，我们在按钮上用`onClick`监听事件
3. 当按钮点击时，一个信息发送给mailbox的地址
4. 在接到信息时，mailbox发射一个信号，`main`捡起了他
5. `main`再次通过mailbox的地址来渲染视图，信息来自mailbox("Hello")

### 总结

Mailboxs 是通信枢纽，他们从我们的UI得到信息并且传播他们到我们程序的其他地方。当创建复杂web应用时，他们是一个整体功能块。

