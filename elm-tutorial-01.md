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

这个代码很难读，因为只能从内到外求解。管道操作允许我们使用更具可读性的方式来写他：

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

思考一个带有类型签名的函数：

```elm
indexOf : String -> Array String -> Int
```

这个假想的函数取一个字符串和一个字符串数组并返回给定字符串的索引，没有找到的话会返回-1。

但如果往数组中传入的是整数呢？我们就不能再用这个函数了。然而，我们可以使用类型变量而不是特定类型来使得函数变得通用。

```elm
indexOf : a -> Array a -> Int
```

把`String`替换成`a`，函数签名现在表现为，`indexOf`取一个任意类型的`a`和一个具有相同类型`a`的数组并返回一个整数。只要类型匹配，编译器就会满意。你可以一个`String`和一个`String`数组或是一个`Int`和一个`Int`数组调用`indexOf`，他都将正常工作。

这种方式的函数可能更通用。你可以有几个类型变量：

```elm
switch : (a, b) -> (b, a)
switch (x, y) =
  (y, x)
```

这个函数娶一个类型为`a`，`b`的元组返回一个类型为`b`，`a`的 元组。所以这些都是有效的：

```elm
switch(1, 2)
switch("A", 2)
switch(1, ["B"])
```

### 函数作为参数

思考一个像这样的签名：

```elm
map : (Int -> String) -> List Int -> List String
```

这个函数：

* 取一个函数：`(Int -> String)`部分
* 一个整数列表
* 返回一个字符串列表

有趣的部分是`(Int -> String)`。他表示一个函数必须符合`(Int -> String)`签名。

例如，标准库中的`toString`就是这样一个函数。所以你可以这样调用`map`：

```elm
map toString [1, 2, 3]
```

但是，`Int`和`String`类型太典型了。所以大多数情况下你会看到签名像下面这样：

```elm
map : (a -> b) -> List a -> List b
```

这个函数把`a`列表映射为`b`列表。我们不需要太关心`a`和`b`是什么，只要和第一个参数中使用的类型相同就可以了。

例如下面这些给定签名的函数:

```elm
convertStringToInt : String -> Int
convertIntToString : Int -> String
convertBoolToInt : Bool -> Int
```

我们可以以通用的方式调用map：

```elm
map convertStringToInt ["Hello", "1"]
map convertIntToString [1, 2]
map convertBoolToInt [True, False]
```

## 导入和模块

在 Elm 中导入一个模块要使用`import`关键字，如：

```elm
import Html
```

这会从库中导入`Html`模块。然后你可以用完全限定路径使用模块中的函数和类型：

```elm
Html.div [] []
```

你也可以导入模块并暴露某些函数和类型：

```elm
import Html exposing (div)
```

`div`混入当前范围。所以你可以这样使用：

```elm
div [] []
```

也可以暴露模块的所有东西：

```elm
import Html exposing (..)
```

然后你就可以使用模块中的每一个函数和类型。但是并不是很建议这样用，因为模块之间命名可能会彼此冲突。

### 模块和类型使用相同的名字

许多模块导出类型时使用了和模块相同的名字。例如`Html`模块包含一个`Html`类型，`Signal`模块有一个`Signal`类型。

所以这个返回一个`Html`元素的函数：

```elm
import Html

myFunction : Html.Html
myFunction = 
  ...
```

等同于：

```elm
import Html exposing (Html)

myFunction : Html
myFunction =
  ...
```

第一个仅导入了`Html`模块并使用了完全限定的`Html.Html`。

后面的从`Html`模块导出并直接使用了`Html`类型。

### 模块申明

当你在 Elm 中创建一个模块时，你需要在顶部增加一个`module`声明：

```elm
module Main (..) where
```

`Main`就是模块的名字。`(..)`表示你要暴露全部的函数和类型。Elm 会在一个叫做`Main.elm`的文件中寻找这个模块。所以文件名要和模块名称相同。

你可以在程序中使用更深的文件结构，比如`Players/Utils.elm`应该被声明为：

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

`Answer`可以是`Yes`或`No`。联合类型可以使我们的代码变得更通用。例如一个函数可以像这样声明：

```elm
respond : Answer -> String
respond answer =
  ...
```

无论`Yes`或`No`都可以作为第一个参数，如`respond Yes`会被调用。

在 Elm 中联合类型也常常被称为**tags**。

### 负载

联合类型可以关联信息@todo：

```elm
type Answer = Yes | No | Other String
```

这里的`Other`有一个关联的字符串。你可以像这样调用`respond`：

```elm
respond (Other "Hello")
```

你需要括号，否则 Elm 会解析成两个参数的函数调用。

### 像函数一样调用

现在我们给`Other`增加一个负载：

```elm
Other "Hello"
```

这非常像调用`Other`函数。联合类型的行为就像函数。例如下面的类型：

```elm
type Answer = Message Int String
```

你可以创建一个`Message`标签像这样：

```elm
Message 1 "Hello"
```

还可以像其他函数那样做偏应用。


### 嵌套

你可以嵌套联合类型，这是非常常见的：

```elm
type OtherAnswer = DontKnow | Perhaps | Undecided

type Answer = Yes | No | Other OtherAnswer
```

然后你可以像这样传递给`respond`函数（需要`Answer`）：

```elm
respond (Other Perhaps)
```

### 类型别名

也可能会用类型别名：

```elm
type Answer a = Yes | No | Other a
```

这样`Answer`就可以不同的类型，如 Int，String。

例如`respond`可以像这样使用：

```elm
respond : Answer Int -> String
respond answer =
  ...
```

这里我们说`a`应该是`Int`类型的，被使用在`Answer Int`签名中@todo。

所以之后我们这样调用`respond`：

```elm
respond (Other 123)
```

但是 respond `(Other "Hello")`会报错，原因是`respond`需要一个整数来代替`a`。

### 惯用法

在我们的程序中，一个联合类型典型的用法是某个值是已知一组值中的一个。

例如在典型的 web 应用程序中，我们有一些动作可能被执行，例如加载用户，新增用户，删除等等。其中一些可能还有负载。

这是使用联合类型常见的地方：

```elm
type Action
  = LoadUsers
  | AddUser
  | EditUser UserId
```

还有更多的关于联合类型，有兴趣可以阅读[这里](http://elm-lang.org/guide/model-the-problem)

### 单位类型

空元组`()`被称作**单位类型**,在 Elm 中被广泛使用，所以需要解释一下。

思考一个带有类型变量（用`a`表示）的类型别名。

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

但是如果函数的`body`并不需要任何值呢？我们使用单位类型传入body表示空：

```elm
readMessage : Message () -> String
readMessage message =
  ...
```

这个函数取的是空`body`的`Message`。他不同于**任意类型**，仅仅表示一个空值。所以，单位类型又常常被用于空值的占位符。

### Task

关于单位类型真实的例子就是`Task`类型。当使用`Task`时，你将会常常见到单位类型。

一个典型的task含有一个 **`error`** 和一个 **`result`**：

```elm
Task error result
```

* 有时我们希望错误能被安全忽略掉：`Task () result`
* 或者结果被忽略：`Task error ()`
* 又或者全部：`Task () ()`

