# 基础

这一章涵盖了：

* 运行一个基本的 Elm 程序
* Elm 中的基本函数

## Hello World

### 安装 Elm

在 [http://elm-lang.org/install](http://elm-lang.org/install) 可以找到符合你系统的安装程序，下载并安装他。

## 我们的第一个 Elm 程序

让我们来写我们的第一个 Elm 程序。创建一个文件夹。文件夹处在终端中运行下面的命令：

```sh
elm package install evancz/elm-html
```

这将会安装 _elm-html_ 模块。然后新建一个`Hello.elm`文件，输入下面的代码：

```elm
import Html

main : Html.Html
main =
  Html.text "Hello"
```

[https://github.com/sporto/elm-tutorial-assets/blob/master/code/010_foundations/Hello.elm](https://github.com/sporto/elm-tutorial-assets/blob/master/code/010_foundations/Hello.elm)

接着在终端中键入：

```sh
elm reactor
```

你将会看到：

```sh
elm reactor 0.16.0
Listening on http://0.0.0.0:8000/
```

在浏览器中打开`http://0.0.0.0:8000/`，并且点击`Hello.elm`。你会在你的浏览器中看到`Hello`

### import

在 Elm 中你需要明确的导入你需要使用的**模块**。这里我们需要使用**Html**。这个模块里有许多工作于html的函数。我们将使用`.text`。

### main

Elm 中的每个程序都开始于一个`main`函数。`main`应该是一个返回一个静态的元素或是一个元素 signal （signals 将在稍后讨论）的函数。这里`main`仅仅返回了`Html.Html`元素（Html 元素来自 Html 模块）。

### elm reactor

Elm reactor 创建一个服务器来编译 Elm 代码。reactor 是一个非常有用的开发工具，使用他不必担心麻烦的构建过程。然而 reactor 有一些局限性，所以我们会在之后切换到程序构建当中。

## 函数基础

这一章包括了 Elm 非常重要的基础语法：函数，函数签名，偏应用和管道操作。

### 函数

然我们看一下 Elm 中的函数：

```elm
add : Int -> Int -> Int
add x y =
  x + y
```

例子的第一行是函数签名。这个签名在 Elm 中是可选的，但是非常推荐加上，因为他可以是你的函数更加清晰明确。

这个叫做`add`的函数取两个整数（`Int -> Int`）并且返回一个其他的整数（第三个`->`）。

第二行是函数定义。参数是`x`和`y`。

然后我们函数的主体是`x + y`，仅仅返回了参数的和。

调用函数可以写为：

```elm
add 1 2
```

### 用括号分组

当你想要调用一个函数并且作为另一个函数的结果，你需要使用括号进行分组：

```elm
add 1 (divide 12 3)
```

这里`divide 12 3`的结果给了`add`作为第二个参数。

相比之下，其他语言中会被写为：

```elm
add(1, divide(12, 3))
```

### 偏应用

在 ELm 中你可以取一个函数，如上面的`add`，并只给他一个参数调用他，如`add 2`。

这将返回另外一个用2作为第一个参数的函数。用第二个数调用这个函数会返回`2+`这个数的值。

```elm
add2 = add 2
add2 3 ==> 5
```

换种思路来理解函数签名`add : Int -> Int -> Int`，他是一个函数取第一个整数作为参数并且返回另一个函数。这个返回的函数取另一个整数会返回一个整数。

偏应用在 Elm 中对于提高代码可读性和在函数和程序之间传递状态是非常有用的。

## 管道操作

下面你可以这样嵌套函数：

```elm
add 1 (multiply 2 3)
```

这是一个简单的例子，来考虑一个复杂一些的：

```elm
sum (filter (isOver 100) (map getCost records))
```

这个代码很难读，因为@todo。管道操作允许我们使用更具可读性的方式来写他：

```elm
3
  |> multipy 2
  |> add 1
```

他很大程度依赖于偏应用。这个例子中，`3`传递给偏应用函数`multiply 2`。他的结果再传递给另一个偏应用函数`add 1`。

使用管道操作来完成上面的例子可以写为：

```elm
records
  |> map getCost
  |> filter (isOver 100)
  |> sum
```

## 更多函数

### 类型别名
