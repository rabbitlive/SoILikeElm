# 基础

这一章是Elm的基础部分，其涵盖了：

* 运行一个基本的 Elm 程序
* Elm 中的基本函数

## Hello World

### 安装 Elm

在 [http://elm-lang.org/install](http://elm-lang.org/install) 下载符合你系统的安装程序，并安装他。

### 我们的第一个 Elm 程序

让我们来写第一个 Elm 程序。首先创建一个文件夹。在文件夹处打开终端，并运行下面的命令：

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

接着在终端中输入：

```sh
elm reactor
```

你将会看到：

```sh
elm reactor 0.16.0
Listening on http://0.0.0.0:8000/
```

在浏览器中打开`http://0.0.0.0:8000/`，并且点击`Hello.elm`。你会在浏览器中看到`Hello`。让我们来看下这些东西。

### import

在 Elm 中你需要明确的导入你需要使用的**模块**。这里我们要用到**Html**。这个模块里有许多工作于 html 的函数。我们用到了`.text`。

### main

Elm 中的每个程序都有一个`main`函数作为开始程序。`main`是一个返回静态的元素或是元素 signal （signals 会在稍后讨论）的函数。这里的`main`仅仅返回了`Html.Html`（Html 元素来自于 Html 模块）。

### elm reactor

Elm **reactor** 创建了一个服务器用来编译 Elm 代码。reactor 是一个非常有用的开发工具，使用他就不必担心麻烦的构建过程。但是 reactor 有一些局限性，所以我们会在之后切换到构建程序当中。

## 函数基础

这一节包括了 Elm 几个非常重要的基础语法概念：函数，函数签名，偏应用和管道操作。

### 函数

先来让我们看一下 Elm 中的函数：

```elm
add : Int -> Int -> Int
add x y =
  x + y
```

第一行是函数签名。这里的签名在 Elm 中是可选的，但非常推荐把他加上，因为可以使你的函数更加清晰明确。

这个叫做`add`的函数取两个整数（`Int -> Int`）并返回另外一个整数（第三个`->`）。

第二行是函数定义。参数是`x`和`y`。

我们函数的主体是`x + y`，返回了参数的和。

调用函数可以写为：

```elm
add 1 2
```

### 用括号分组

当你想要调用一个函数并作为另一个函数的结果，你需要使用括号对其进行分组：

```elm
add 1 (divide 12 3)
```

这里`divide 12 3`的结果给了`add`作为他的第二个参数。

相比之下，其他语言中可能会写成：

```elm
add(1, divide(12, 3))
```

### 偏应用

上面的`add`函数中，你可以调用一个函数，并只给他一个参数，如`add 2`。

这将返回一个用2作为第一个参数的函数。用第另一个数去调用这个函数就会返回`2+`这个数的和。

```elm
add2 = add 2
add2 3 ==> 5
```

让我们换种思路来理解函数签名。`add : Int -> Int -> Int`，是取第一个整数作为参数并且返回一个函数。这个返回的函数给他另一个整数会返回一个整数。

偏应用在 Elm 中对于提高代码可读性和在函数和程序间传递状态是非常有用的。

## 管道操作

你可以这样嵌套函数：

```elm
add 1 (multiply 2 3)
```

这只是一个简单的例子，来考虑一个复杂些的：

```elm
sum (filter (isOver 100) (map getCost records))
```

这个代码很难读，因为只能从内到外求解。而管道操作允许我们使用更具可读性的方式来写他：

```elm
3
  |> multipy 2
  |> add 1
```

这很大程度上依赖偏应用。这个例子中，`3`传递给偏应用函数`multiply 2`。他的结果再传递给下一个偏应用函数`add 1`。

使用管道操作来完成上面的例子：

```elm
records
  |> map getCost
  |> filter (isOver 100)
  |> sum
```

## 更多函数

### 类型别名

想想一个带有类型签名的函数：

```elm
indexOf : String -> Array String -> Int
```

这个假想的函数取一个字符串和一个字符串数组，并返回给定字符串的索引，没有找到的话会返回-1。

但如果往数组中传入的是整数呢？我们就不能再使用这个函数了。然而，我们可以使用类型变量而不是特定类型来使得函数变得通用。

```elm
indexOf : a -> Array a -> Int
```

把`String`替换成`a`，函数签名表现为，`indexOf`取一个任意类型的`a`和一个具有相同类型`a`的数组并返回一个整数。只要类型匹配，编译器就不会报错。你可以用一个`String`和一个`String`数组或是一个`Int`和一个`Int`数组调用`indexOf`，他都将正常工作。

这种方式的函数可能更加通用。你也可以有好几个类型变量：

```elm
switch : (a, b) -> (b, a)
switch (x, y) =
  (y, x)
```

这个函数娶一个类型为`a`，`b`的元组并返回一个`b`，`a`类型的元组。下面所有这些都是有效的：

```elm
switch(1, 2)
switch("A", 2)
switch(1, ["B"])
```

### 函数作为参数

一个像这样的签名：

```elm
map : (Int -> String) -> List Int -> List String
```

这里的函数：

* 取一个函数（`(Int -> String)`部分）
* 一个整数列表
* 返回一个字符串列表

有趣的部分是`(Int -> String)`。他表示一个函数而且要符合`(Int -> String)`的签名。

例如，标准库中的`toString`就是这样一个函数。所以你可以这样调用`map`：

```elm
map toString [1, 2, 3]
```

但是，`Int`和`String`这样类型太典型了。所以大多数情况下你会看到签名是像下面这样：

```elm
map : (a -> b) -> List a -> List b
```

函数把列表`a`映射为列表`b`。我们不需要太关心`a`和`b`是什么，只要和第一个参数中使用的类型相同就可以了。

例如下面这些给定签名的函数:

```elm
convertStringToInt : String -> Int
convertIntToString : Int -> String
convertBoolToInt : Bool -> Int
```

我们以通用的方式来调用map：

```elm
map convertStringToInt ["Hello", "1"]
map convertIntToString [1, 2]
map convertBoolToInt [True, False]
```

## 导入和模块

在 Elm 中导入一个模块要用到`import`关键字，如：

```elm
import Html
```

这会从库中导入`Html`模块。然后就可以用完全限定去使用模块中的函数和类型：

```elm
Html.div [] []
```

你也可以暴露某些函数和类型：

```elm
import Html exposing (div)
```

`div`混入到当前范围。所以就能够这样使用：

```elm
div [] []
```

也可以暴露模块的所有东西：

```elm
import Html exposing (..)
```

这样就可以使用模块中的每一个函数和类型了。但是并不是很建议这样做，因为模块之间的命名可能会彼此冲突。

### 模块和类型使用相同的名字

许多模块导出类型时使用了和模块相同的名字。例如`Html`模块包含一个`Html`类型，`Signal`模块有一个`Signal`类型。

这里有一个返回`Html`元素的函数：

```elm
import Html

myFunction : Html.Html
myFunction = 
  ...
```

他等同于：

```elm
import Html exposing (Html)

myFunction : Html
myFunction =
  ...
```

前面的例子仅导入了`Html`模块并使用了完全限定的`Html.Html`。

之后的从`Html`模块导出并直接使用了`Html`类型。

### 模块申明

当你在 Elm 中创建一个模块时，你需要在顶部增加一个`module`声明：

```elm
module Main (..) where
```

`Main`就是模块的名字。`(..)`表示你要暴露全部的函数和类型。Elm 会在一个叫做`Main.elm`的文件中寻找这个模块。所以，文件名要和模块名相同。

也可以在程序中使用深一层的文件结构，比如`Players/Utils.elm`应该被声明为：

```elm
module Players.Utils (..) where
```

这样就可以在程序各处这样导入:

```elm
import Players.Utils
```

### 联合类型

Elm 中联合类型被用来做许多事情，因为他们非常灵活。一个联合类型看起来像这样：

```elm
type Answer = Yes | No
```

`Answer`可以是`Yes`或`No`。联合类型可以使我们的代码变得更加通用。例如一个函数可以像这样声明：

```elm
respond : Answer -> String
respond answer =
  ...
```

无论`Yes`或`No`都可以作为第一个参数，如`respond Yes`。

在 Elm 中联合类型也常常被称为**tags**。

### 负载

联合类型可以附带一个关联信息：

```elm
type Answer = Yes | No | Other String
```

这里的`Other`有一个关联的字符串。你可以像这样调用`respond`：

```elm
respond (Other "Hello")
```

这里需要括号，否则 Elm 会解析为两个参数的函数调用。

### 像函数一样调用

现在我们给`Other`增加一个负载：

```elm
Other "Hello"
```

这非常像调用`Other`函数。联合类型的行为就像函数。例如下面的类型：

```elm
type Answer = Message Int String
```

你可以像下面这样创建一个`Message`标签：

```elm
Message 1 "Hello"
```

还可以像其他函数那样做偏应用。


### 嵌套

联合类型还可以嵌套，这是非常常见的：

```elm
type OtherAnswer = DontKnow | Perhaps | Undecided

type Answer = Yes | No | Other OtherAnswer
```

然后就可以像这样传递给`respond`函数（需要`一个Answer`）：

```elm
respond (Other Perhaps)
```

### 类型别名

你也可能会用到类型别名：

```elm
type Answer a = Yes | No | Other a
```

这样`Answer`就可以有不同的类型，如 Int，String。

例如`respond`可以这样使用：

```elm
respond : Answer Int -> String
respond answer =
  ...
```

这里我们说签名中的`a`应该是`Int`类型的。

所以之后我们这样调用`respond`：

```elm
respond (Other 123)
```

所以`respond (Other "Hello")`会报错，原因是`respond`需要一个整数来代替`a`。

### 惯用法

在我们的程序中，一个联合类型典型的用法是某个值是已知的一组值中的一个。

例如在典型的 web 应用程序中，我们常常会执行一些操作，如加载用户，新增用户，删除等等。其中一些类型可能会有负载。

这是使用联合类型常见的地方：

```elm
type Action
  = LoadUsers
  | AddUser
  | EditUser UserId
```

更多的关于联合类型，有兴趣可以阅读[这里](http://elm-lang.org/guide/model-the-problem)

### 单位类型

空元组`()`被称作**单位类型**，在 Elm 中被广泛使用，所以需要解释一下。

一个带有类型变量（用`a`表示）的类型别名：

```elm
type alias Message a = {
  code : String,
  body : a
}
```

你可以写一个使用`Message`并用`Sting`表示`body`的函数，像这样：

```elm
readMessage : Message String -> String
readMessage message =
  ...
```

或者是用一个整数列表表示`Body`的函数:

```elm
readMessage : Message (List Int) -> String
readMessage message =
  ...
```

但是如果函数的`body`并不需要任何值呢？这时我们使用单位类型来传入body表示空：

```elm
readMessage : Message () -> String
readMessage message =
  ...
```

这个函数取的是空`body`的`Message`。不同于**任意类型**，他仅仅表示一个空值。所以，单位类型又常常被用于空值占位符。

### Task

关于单位类型一个真实的用例就是`Task`类型。当使用`Task`时，你将会常常见到单位类型。

一个典型的 task 含有一个 **`error`** 和一个 **`result`**：

```elm
Task error result
```

* 有时我们希望错误能够被安全忽略掉：`Task () result`
* 或者结果被忽略掉：`Task error ()`
* 又或者全部都忽略：`Task () ()`

