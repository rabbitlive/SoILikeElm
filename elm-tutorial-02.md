# 信号 Signals

这一章关于信号，他包括:

* signal 介绍
* 使用`foldp`函数保持状态
* mailboxs

## signal 介绍

在Elm 中，**信号Signals**是创建应用程序的一个基本构建块。可以把一个 signal 想象成一个流，他的值每时每刻都在变化。

信号可以合并，转换以及过滤。让我们来看一个基本的 signal：

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

在浏览器中运行这段代码，移动鼠标就会看到`x`坐标值在变化。

首先我们导入了`Html`模块，用他将`x`坐标显示为 HTML。紧接着又导入了`Mouse`模块，他提供了用于鼠标操作的工具。

### view

`view`函数取一个整数作为参数并返回一段HTML（`Int -> Html`）。

不过，`Html.text`需要的是一个字符串，所以我们必须将`x`转为字符串。这就要用到`toString`函数。

### main

Elm 程序的 main 函数比较特殊，他要求返回一个**静态元素**或是一个**信号**。这里的`main`返回的是携带`Html`的信号（`Signal.Signal Html`）。换句话说，我们的HTML现在可以随信号的变化而变化。

要理解这是怎么做到的，让我们先来分析一下代码的最后一行（`Signal.map view Mouse.x`）。

### Mouse.x

`Mouse.x`会在`x`改变时给我们一个signal 。这个信号的签名是`Signal.Signal Int`，他表示这个信号会携带一个整数。

### Signal.map

`Signal.map`的作用是将一个信号转换为另一个信号，类型签名是`Signal.map : (a -> result) -> Signal a -> Signal result`。让我们来分析一下：

* 第一个参数是一个函数，他接收`a`类型的值，返回`result`类型的值。这个函数会将原信号携带的值转换为一个新值用作输出信号。
* 第二个参数就是原信号，他的类型变量是`a`，和第一个参数中的`a`相同。
* 返回类型是`Signal result`，表示输出的 signal 携带的类型是`result`。

> **图示**：signal`A`通过`Signal.map`产生了signal`B`。输出信号中的值是原信号中的两倍。

回到我们的例子中：

```elm
Signal.map view Mouse.x
```

`view`是用来转换的函数，他取一个`Int`并返回`Html`。

第二个参数（`Mouse.x`）是一个带有`Int`值的信号。

而结果正是我们想要的`Html`值的信号，用作`main`函数的输出。

`Singal.map`用转换函数将原信号转换成为一个新信号。当原信号发生变化时，信号中的值统统会被转换到新信号中。


这里是 map 的另一个例子：

```elm
double x =
  x * 2

doubleSignal =
  Signal.map double Mouse.x
```

`double`是一个将输入值加倍的函数，这样看来，`doubleSignal`就是一个将当前 x 坐标乘 2 的信号。

## 保持状态

上一节中，我们创建了一个简单的应用来显示当前鼠标的`x`坐标。获取到x坐标这个状态之后就被丢弃掉了。但是在大多数应用中，我们希望储存一些用户交互的状态值。

现在我们来创建一个记录鼠标单击的程序。


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

现在这个程序会显示出`1`，让我们来依次介绍一下。

`view`函数用来显示数量，但目前是1，不会改变。

`countSignal`函数返回一个带有整数的信号。`Mouse.clicks`信号作为输入并被转换为`(always 1)`。

`main`函数用`view`来转换`countSignal`。

### Signal.foldp

`Signal.foldp`用来“折叠过去”，他取上一个状态和当前输入进行组合，产生一个新的信号。很像JavaScript的`reduce`。

我们来使用`foldp`改写：

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

让我们看下`Signal.foldp`那行发生了些什么：

`Signal.foldp`有三个参数：

* 积累函数：`(\_ state -> state + 1)`
* 初始状态：`0`
* 输入信号：`Mouse.clicks`

语法`\x y -> x + y`表示这是一个匿名函数，他等同于 ES6 中的`(x, y) => x + y`。

### foldp是如何工作的

* `foldp`每次接到信号就会调用积累函数。
* **积累函数**接受**上一个状态**和**输入信号**用于输出。
* 首次调用`foldp`会将**输入信号**和一个**初始状态**传递给积累函数。
* 积累函数计算并返回一个新的状态。
* `foldp`保持这个新状态，下一次时将作为前一状态传入积累函数
* 最后，`foldp`会产生一个输出信号

### 积累函数

```elm
\_ state -> state + 1
```

**积累函数**取`Mouse.clicks`信号中得到值和上一状态作为参数，返回一个新的状态。

不过，来自`Mouse.clicks`信号的值是`()`，在 ELm 中这个叫做单位类型。现在我们并不需要他，所以用`_`忽略掉了。

## 邮箱 Mailboxes

`foldp`是构建程序非常重要的构建块，还有一个东东也非常重要，他就是邮箱 Mailboxes。

到目前为止，我们一直在监听像`Mouse.x`这样的"原始"信号并且用它来在页面上显示信息。但在一个大型应用中，我们想要的是能够与用户交互。比如单击一个链接或按钮。

在 Elm 中我们使用**邮箱**来做这件事。一个邮箱就是一个从UI接收消息和任务，并发出信号的中转站。

想要更好的理解邮箱，让我们先来创建一个按钮：

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

`view`函数将显示一条消息和一个用于点击的按钮，这个按钮现在还木有做任何事。

`messageSignal`是一个用“Hello”字符串构造的信号，意味着他的值将不会改变。

`main`函数从`messageSignal`得到字符串并交给`view`函数转换。在浏览器中运行这段代码，就会看到来自`messageSignal`的`Hello`。

### 添加交互

我们想要点击按钮来改变消息。这需要用到单击事件`Html.Events.onClick`。如果你看过了`Html.Events`模块的文档，就会发现所有的事件都有相同的函数签名：`Signal.Address a -> a -> Html.Attribute`

任意类型变量的`Signal.Address`作为第一个参数，第二个和第一个参数具备相同的类型变量，然后返回了一个`Html.Attribute`。

### 地址 Address

**地址**就是一个辨识信号的标识。他允许我们给那个信号发送**消息**。

想要获取**地址**发来的消息，我们需要使用一个`Mailbox`。

可以像这样创建一个**邮箱**：

```elm
mb : Signal.Mailbox String
mb =
  Signal.mailbox ""
```

`mb`是一个返回`Mailbox`的函数。这个信箱用于处理字符串，他接受`String`类型的消息并发出一个`String`类型的信号。空字符串是信箱提供给信号的默认值。

这个函数返回的信箱其实是一个两个属性的记录：

```elm
{ address : Signal.Address a
, signal : Signal.Signal a
}
```

记录的`address`让我们可以发消息给他，并且可以监听他的`signal`。

## 使用 Mailbox

下一步是在我们的程序中使用信箱，这样我们就可以发送和刷新消息了。

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

`view`函数现在改为取`Signal.Address`作为第一个参数。

`Events.onClick address "Hello"`用于在Html元素上监听点击事件。在每次点击元素时`onClick`将向给定的地址发送消息（这里是“Hello”）。

### main

```elm
main : Signal Html
main =
  Signal.map (view mb.address) mb.signal
```

这里的`main`函数做了两件事情：

* 使用了一个偏应用的view`(view mb.address)`函数，这样view的第一个参数就是邮箱的`address`。
* 在邮箱的导出的信号上应用偏应用函数view。所以`view`会将邮箱的信号作为他的第二个参数。

这里的图帮助我们理解发生了什么：

1. 最开始时，`main`用信箱的初始值传递给地址然后渲染视图(空字符串)
2. 当视图渲染完成，我们在按钮上使用`onClick`来监听事件
3. 当按钮点击时，一个消息发送给邮箱的地址
4. 在接到消息后，邮箱会发出一个信号，`main`会接到这个信号
5. 此时，`main`函数再次通过邮箱的地址来渲染视图，消息就来自于邮箱("Hello")
